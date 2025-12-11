---
title: vue3 defineProps的使用
tags:
  - Vue3
  - defineProps
  - Composition API
  - TypeScript
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
date: 2025-12-11 22:55:16
updated: 2025-12-11 23:37:40
categories:
  - 前端
  - Vue
keywords:
  - Vue3
  - defineProps
  - Props 校验
  - Composition API
description: "Vue 3 中 defineProps 的定义、类型声明、默认值与常见注意事项总结，包含 TS 示例与调试技巧。"
top_img:
cover:
copyright_author_href:
copyright_url:
copyright_info:
---

## 1. 基础用法

```vue
<script setup>
// 数组语法（简单声明）
const props = defineProps(['title', 'content', 'count'])

// 对象语法（带类型验证与默认值）
const detailProps = defineProps({
  title: {
    type: String,
    required: true,
    default: '默认标题'
  },
  count: {
    type: Number,
    default: 0,
    validator: (value) => value >= 0
  },
  status: {
    type: String,
    default: 'pending',
    validator: (value) => ['pending', 'success', 'error'].includes(value)
  }
})
</script>

<template>
  <div>
    <h1>{{ detailProps.title }}</h1>
    <p>计数: {{ detailProps.count }}</p>
  </div>
</template>
```

## 2. TypeScript 支持

```ts
<script setup lang="ts">
interface Props {
  title: string
  count?: number
  isActive?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  count: 0,
  isActive: false
})

type UserInfo = {
  name: string
  email: string
}

const emitProps = defineProps<{
  user: UserInfo
  onUpdate?: (data: UserInfo) => void
}>()
</script>
```

## 3. 使用 props

```vue
<script setup>
import { computed } from 'vue'

const props = defineProps({
  message: String,
  count: Number
})

const doubleCount = computed(() => props.count * 2)

const handleClick = () => {
  console.log('当前消息:', props.message)
}
</script>

<template>
  <div>
    <p>{{ message }}</p>
    <p>{{ count }}</p>
    <p>{{ doubleCount }}</p>
    <button @click="handleClick">输出消息</button>
  </div>
</template>
```

## 4. 解构与响应性

```vue
<script setup>
import { toRefs } from 'vue'

const props = defineProps({
  title: String,
  count: Number
})

// 保持响应性，避免直接解构丢失响应
const { title, count } = toRefs(props)
</script>

<template>
  <div>
    <h2>{{ title }}</h2>
    <p>当前计数: {{ count }}</p>
  </div>
</template>
```

## 5. 结合 defineEmits / withDefaults

```ts
<script setup lang="ts">
type CardType = 'primary' | 'info' | 'warning'

const props = withDefaults(defineProps<{ label: string; type?: CardType }>(), {
  type: 'info'
})

const emit = defineEmits<{ (e: 'toggle', value: boolean): void }>()

const handleToggle = () => emit('toggle', props.type === 'primary')
</script>

<template>
  <button @click="handleToggle">
    {{ props.label }} - {{ props.type }}
  </button>
</template>
```

## 6. 常见调试与注意事项

- defineProps 是编译器宏，无需从 Vue 导入
- 避免直接解构 props，使用 toRefs 保持响应性
- 只能在 <script setup> 中使用，属于 Composition API 语法糖
- props 为只读，修改需要 emit 事件或使用父组件数据源
- TS 下优先使用类型参数 + withDefaults，提升可读性和提示
- 校验失败或类型不匹配时，检查控制台警告并确认默认值、必填项