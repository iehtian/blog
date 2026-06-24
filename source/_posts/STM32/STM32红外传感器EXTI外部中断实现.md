---
title: STM32F103 红外传感器 EXTI 外部中断实现
tags: [STM32, EXTI, 中断, HAL库]
comments: true
toc: true
toc_number: true
toc_style_simple: false
copyright: true
copyright_author: iehtian
mathjax: false
katex: false
aplayer: false
highlight_shrink: false
aside: true
abcjs: false
noticeOutdate: false
date: 2026-06-08
categories: 嵌入式
keywords: STM32, EXTI, 外部中断, 红外传感器, NVIC, HAL回调
description: 将 STM32F103 红外对射传感器从 GPIO 轮询改为 EXTI 下降沿中断模式，详解中断链路、NVIC、__weak 回调机制及 EXTI 挂起寄存器清除原理。
top_img:
cover: https://picsum.photos/id/114/800/450
copyright_author_href:
copyright_url:
copyright_info:
---

## 背景

在 STM32F103C8T6 项目中，PB14 连接了一个红外对射传感器，用于检测遮挡事件并在 OLED 上显示计数。最初的实现使用 GPIO 轮询方式读取引脚状态，但存在两个问题：

1. **累加错误**：`counter += HAL_GPIO_ReadPin()` 在引脚为高电平时每次循环都会累加，并不是边沿检测
2. **引脚未配置为输入**：`MX_GPIO_Init()` 中只开启了 GPIO 时钟，PB14 从未被配置为输入模式

最终决定放弃轮询方案，改用 **EXTI 外部中断**的下降沿触发模式，实现可靠的边沿检测。

## 中断链路

当红外光束被遮挡时，PB14 产生下降沿，整个中断响应流程如下：

```
PB14 下降沿（光束被遮挡）
  → EXTI15_10_IRQHandler()           [stm32f1xx_it.c：中断入口]
    → HAL_GPIO_EXTI_IRQHandler()     [HAL 库：清除 PR 标志 + 调用回调]
      → HAL_GPIO_EXTI_Callback()     [main.c：ir_counter++]
```

整个链路涉及三个层级：硬件中断入口、HAL 中间处理、用户回调逻辑。

## 代码修改

### main.c

在 `main.c` 中需要做三处修改：

**1. 添加全局计数变量**

```c
volatile int ir_counter = 0;
```

注意：`volatile` 关键字必不可少，因为该变量会在中断中修改、在主循环中读取，防止编译器优化掉对它的访问。

**2. 修改 PB14 的 GPIO 配置**

将原来的普通输入模式改为下降沿中断模式，并启用 NVIC：

```c
GPIO_InitTypeDef GPIO_InitStruct = {0};

// PB14 配置为下降沿中断 + 上拉
GPIO_InitStruct.Pin = GPIO_PIN_14;
GPIO_InitStruct.Mode = GPIO_MODE_IT_FALLING;
GPIO_InitStruct.Pull = GPIO_PULLUP;
GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
HAL_GPIO_Init(GPIOB, &GPIO_InitStruct);

// 启用 EXTI15_10 中断并设置优先级
HAL_NVIC_SetPriority(EXTI15_10_IRQn, 0, 0);
HAL_NVIC_EnableIRQ(EXTI15_10_IRQn);
```

**3. 重写回调函数并简化主循环**

```c
// 重写 HAL 弱回调
void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
{
    if (GPIO_Pin == GPIO_PIN_14)
    {
        ir_counter++;
    }
}
```

主循环只需负责显示，不再读取引脚状态：

```c
while (1)
{
    // 仅显示 ir_counter 到 OLED
    display_counter(ir_counter);
}
```

### stm32f1xx_it.c

在 `stm32f1xx_it.c` 中添加中断服务函数：

```c
void EXTI15_10_IRQHandler(void)
{
    HAL_GPIO_EXTI_IRQHandler(GPIO_PIN_14);
}
```

注意：Pins 10–15 共享同一个中断向量 `EXTI15_10_IRQn`，因此中断入口函数名为 `EXTI15_10_IRQHandler`。

## 关键概念

### NVIC（嵌套向量中断控制器）

NVIC 是 ARM Cortex-M3 内置的中断管理器，负责中断的优先级排序与开关控制：

- `HAL_NVIC_SetPriority()`：设置中断优先级（抢占优先级 + 子优先级）
- `HAL_NVIC_EnableIRQ()`：使能指定中断向量

可以类比为"门卫 + 调度员"：NVIC 决定哪个中断可以进、谁先执行、谁能打断谁。

### `__weak` 回调模式

HAL 库中定义了一个空的弱回调函数：

```c
__weak void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
{
    UNUSED(GPIO_Pin);
}
```

用户在自己的代码中定义同名函数即为"强定义"，链接时会自动替换弱定义。这是一种**无需注册回调**的机制——只要函数名对上，链接器帮你完成绑定。

### EXTI 挂起标志清除

中断触发后，`EXTI->PR` 寄存器中对应位会被硬件置 1。**必须在退出 ISR 前清除该标志**，否则中断会无限重入。

`HAL_GPIO_EXTI_IRQHandler()` 内部自动执行清除操作：

```c
// HAL 内部实现（简化）
__HAL_GPIO_EXTI_CLEAR_IT(GPIO_Pin);
```

写 1 清除（Write-1-to-Clear）是 ARM 寄存器的常见设计：向对应位写 1 表示清除，写 0 无效。

### EXTI15_10 共享中断

STM32F1 的 EXTI 设计中，Pins 10–15 共享一个中断向量 `EXTI15_10_IRQn`。这意味着 PB10、PB11……PB15 中任意引脚触发中断，都会进入同一个 `EXTI15_10_IRQHandler()`。

`HAL_GPIO_EXTI_Callback(pin)` 作为统一入口，通过 `GPIO_Pin` 参数区分具体是哪个引脚触发的，用户在回调中用 `if` 判断即可。

## 注意事项

- **传感器极性**：本文假设未遮挡时输出高电平、遮挡时输出低电平（下降沿触发）。如果实际相反，需将 `GPIO_MODE_IT_FALLING` 改为 `GPIO_MODE_IT_RISING`，`GPIO_PULLUP` 改为 `GPIO_PULLDOWN`
- **硬件消抖**：当前实现没有软件消抖，若传感器信号存在抖动，计数可能偏多。可在回调中加入时间门控：检查 `HAL_GetTick()` 的时间差，短时间内只计一次
- **volatile 变量**：中断中修改的全局变量必须声明为 `volatile`，否则主循环可能读到缓存旧值

### 消抖示例

```c
void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
{
    if (GPIO_Pin == GPIO_PIN_14)
    {
        static uint32_t last_tick = 0;
        uint32_t now = HAL_GetTick();
        if (now - last_tick > 50)  // 50 ms 消抖
        {
            ir_counter++;
            last_tick = now;
        }
    }
}
```

## 小结

从轮询切换到 EXTI 中断后，红外检测逻辑更可靠：只在边沿时刻响应，主循环不再被引脚读取占用。核心改动集中在 `main.c`（GPIO 配置 + 回调）和 `stm32f1xx_it.c`（中断入口），涉及 NVIC、`__weak` 回调、挂起标志清除等关键概念。

建议在硬件上验证传感器极性和消抖需求后，再最终确定触发模式与时间门控参数。
