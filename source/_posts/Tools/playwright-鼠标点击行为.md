---
title: Playwright 鼠标点击行为
date: 2026-08-04 18:00:00
updated: 2026-08-04 18:00:00
tags: [Playwright, 测试, 自动化, 鼠标, 点击]
categories: 工具
keywords: Playwright, 鼠标点击, locator.click, page.mouse, dblclick, 自动化测试, 右键点击
description: Playwright 鼠标点击行为全面参考 — locator.click()、page.mouse、dblclick()、dispatchEvent 及常见使用模式
cover: https://picsum.photos/id/160/800/450
comments: true
toc: true
toc_number: true
toc_style_simple: false
copyright: true
copyright_author: iehtian
copyright_author_href:
copyright_url:
copyright_info:
mathjax: false
katex: false
aplayer: false
highlight_shrink: false
aside: true
abcjs: false
noticeOutdate: false
---

# Playwright 鼠标点击行为

`locator.click()` 是 Playwright 中最常用的鼠标点击方法，支持右键、双击、修饰键、强制点击等全部点击行为。本文覆盖所有点击相关 API 及常见使用模式。

## 1. locator.click() 核心用法

`locator.click()` 是 Playwright 推荐的点击方式，支持所有高级选项。

### 参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `button` | `"left"` \| `"right"` \| `"middle"` | `"left"` | 鼠标按钮 |
| `clickCount` | `number` | `1` | 点击次数 |
| `delay` | `number` | `0` | mousedown 与 mouseup 之间的延迟（ms） |
| `position` | `{ x: number, y: number }` | — | 相对于元素左上角的点击偏移坐标 |
| `modifiers` | `Modifier[]` | `[]` | 修饰键：`"Alt"`、`"Control"`、`"Meta"`、`"Shift"` |
| `force` | `boolean` | `false` | 跳过 actionability 检查直接点击 |
| `noWaitAfter` | `boolean` | `false` | 不等待后续导航完成 |
| `timeout` | `number` | `30000` | 等待元素可交互的超时时间（ms） |
| `trial` | `boolean` | `false` | 仅执行 actionability 检查，不实际点击 |

### 返回值

`Promise<void>` — 无返回值；操作失败时抛出异常。

### 示例

```ts
// 基础点击
await page.getByRole('button', { name: '提交' }).click();

// 右键点击（触发 contextmenu 事件）
await page.getByText('列表项').click({ button: 'right' });

// 双击
await page.getByText('可编辑文本').click({ clickCount: 2 });

// Ctrl + 点击（Windows/Linux 多选）
await page.getByRole('option').click({ modifiers: ['Control'] });

// 点击元素内指定坐标（左上角偏移 10,10 像素处）
await page.getByTestId('canvas').click({ position: { x: 10, y: 10 } });

// 跳过可见性检查强制点击
await page.getByRole('button', { name: '确认' }).click({ force: true });

// 仅检查是否可点击，不实际执行
await page.getByRole('button').click({ trial: true });
```

- `button: "right"` 触发 `contextmenu` 事件，需配合 `page.on('dialog', …)` 阻止默认上下文菜单来测试自定义右键菜单
- `position` 适用于 Canvas、SVG 等需要精确定位的元素，坐标原点为元素内容区左上角
- `force: true` 跳过可见、启用、稳定等所有可交互性检查，仅应在确认元素已存在于 DOM 且无其他手段接近时使用

## 2. page.click()

`page.click(selector)` 基于 CSS/XPath 选择器点击，参数与 `locator.click()` 完全一致。

```ts
await page.click('#submit-btn');
await page.click('.item', { button: 'right' });
await page.click('text=删除', { force: true });
```

Playwright 内部会将 `page.click()` 转为 locator 执行。**优先使用 `locator.click()`** — 支持链式调用、自动等待、更好的错误信息，且与现代 Playwright API 风格一致。

## 3. dblclick()

`dblclick()` 等价于 `click({ clickCount: 2 })`，参数一致。

```ts
// 两种等价写法
await page.locator('.cell').dblclick();
await page.locator('.cell').click({ clickCount: 2 });
```

- `dblclick()` 内部触发 `click → dblclick` DOM 事件序列
- 如需自定义两次点击间隔，用 `click({ clickCount: 2, delay: 100 })` 替代

## 4. page.mouse 底层 API

`page.mouse` 提供细粒度的鼠标控制，在页面坐标系下直接操作，适合拖拽、画布绘制等场景。

### 方法

| 方法 | 说明 |
|------|------|
| `mouse.click(x, y[, options])` | 在页面坐标 (x, y) 处执行完整点击序列（move → down → up） |
| `mouse.dblclick(x, y[, options])` | 页面坐标双击 |
| `mouse.down([options])` | 按下鼠标按钮（不释放） |
| `mouse.up([options])` | 释放鼠标按钮 |
| `mouse.move(x, y[, options])` | 移动鼠标到页面坐标 (x, y) |

`options` 包含 `button`、`clickCount` 和 `steps`（移动步数，仅 `move` 有效）。

```ts
// 页面坐标点击
await page.mouse.click(200, 300);
await page.mouse.click(200, 300, { button: 'right' });

// 拖拽：按下 → 平滑移动 → 释放
await page.mouse.move(100, 200);
await page.mouse.down();
await page.mouse.move(300, 200, { steps: 10 });
await page.mouse.up();

// Canvas 手绘路径
await page.mouse.move(50, 50);
await page.mouse.down();
await page.mouse.move(150, 50, { steps: 20 });
await page.mouse.move(150, 150, { steps: 20 });
await page.mouse.move(50, 50, { steps: 20 });
await page.mouse.up();
```

- `mouse.click()` 在页面坐标系下工作，与滚动位置无关
- 拖拽时 `steps` 控制移动平滑度：`steps: 1` 瞬间跳转（默认），值越大过渡越平滑
- 定位器点击已满足大多数场景，仅在需要精确坐标控制或轨迹模拟时使用 `page.mouse`

## 5. dispatchEvent("click")

`dispatchEvent()` 直接在元素上触发 DOM 事件，完全绕过 Playwright 的 actionability 检查。

```ts
await page.locator('.hidden-btn').dispatchEvent('click');
```

- 不等待元素可见、不滚动到视口、不执行 hover 等前置操作
- 适用场景：元素被 CSS 隐藏（`opacity: 0`、`visibility: hidden`、`display: none`）但仍需触发其绑定的 click 事件
- **优先使用 `click({ force: true })`** — 至少执行滚动到视口等基本操作，比 `dispatchEvent` 更接近真实用户行为

## 6. 常见模式

### 6.1 右键菜单

```ts
// 触发右键
await page.getByText('项目').click({ button: 'right' });

// 断言菜单项可见
await expect(page.getByRole('menuitem', { name: '删除' })).toBeVisible();

// 点击菜单项
await page.getByRole('menuitem', { name: '删除' }).click();
```

### 6.2 配合修饰键多选

```ts
// Windows/Linux: Ctrl + 点击
await page.getByText('Alice').click({ modifiers: ['Control'] });
await page.getByText('Charlie').click({ modifiers: ['Control'] });

// macOS: Meta + 点击
await page.getByText('Bob').click({ modifiers: ['Meta'] });
```

### 6.3 绕过遮挡强制点击

```ts
await page.getByRole('button', { name: '确认' }).click({ force: true });
```

适用于按钮被 loading 遮罩、tooltip 等临时元素覆盖的场景。

### 6.4 元素内指定位置点击

```ts
// Canvas 或大区域内精确点击
await page.getByTestId('chart').click({ position: { x: 50, y: 100 } });
```

### 6.5 批处理逐条点击

```ts
const items = page.locator('.list-item');
for (let i = 0; i < await items.count(); i++) {
  await items.nth(i).click();
}
```

循环中避免 `forEach` + `await` 或 `map` + `Promise.all` — `click()` 可能触发导航，并发点击会导致上下文丢失。

### 6.6 轮询等待元素出现后点击

```ts
const btn = page.locator('#dynamic-btn');
await btn.waitFor({ state: 'visible' });
await btn.click();
```

## 7. 方法速查

| 方法 | 适用场景 |
|------|---------|
| `locator.click()` | 绝大多数点击场景（推荐） |
| `locator.dblclick()` | 双击编辑、双击展开 |
| `page.mouse.click()` | 页面坐标定位、Canvas 操作 |
| `page.mouse.down/up/move()` | 拖拽、绘图 |
| `dispatchEvent('click')` | 绕过 actionability 强制触发事件 |
