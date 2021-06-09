

# vue 3.0新特性

Vue 3中需要关注的一些新功能包括：
- 组合式 API
- Teleport
- 片段
- 触发组件选项
- `createRenderer` API 来自 `@vue/runtime-core` 创建自定义渲染器
- 单文件组件组合式 API 语法糖 (`<script setup>`) 实验性
- 单文件组件状态驱动的 CSS 变量 (`<style vars>`) 实验性
- 单文件组件 `<style scoped>` 现在可以包含全局规则或只针对插槽内容的规则

# setup

### 执行时机

### 参数

使用`setup`函数时，它将接受两个参数：
- `props`
- `context`

**props**

`setup`函数中第一个参数是`props`。`props`是响应式的，当传入新的prop时，它将被更新。

```js
// MyBook.vue

export default {
  props: {
    title: String
  },
  setup(props) {
    console.log(props.title)
  }
}
```
注：`props`是响应式的，不能使用`ES6解构`，因为它会消除prop的响应性。

如果需要解构prop，可以通过`setup`函数中的`toRefs`来安全的完成此操作。

```js
// MyBook.vue

import { toRefs } from 'vue'

setup(props) {
	const { title } = toRefs(props)

	console.log(title.value)
}
```

**上下文context**

传递给`setup`函数的第二个参数是`context`。`context`是一个普通JavaScript对象，它暴露三个组件的property：

```js
// MyBook.vue

export default {
  setup(props, context) {
    // Attribute (非响应式对象)
    console.log(context.attrs)

    // 插槽 (非响应式对象)
    console.log(context.slots)

    // 触发事件 (方法)
    console.log(context.emit)
  }
}
```

`context`是一个普通的JavaScript对象，不是响应式的，可以安全对着`context`使用es6解构。

```js
// MyBook.vue
export default {
  setup(props, { attrs, slots, emit }) {
    ...
  }
}
```
`attrs` 和 `slots` 是有状态的对象，
它们总是会随组件本身的更新而更新。
这意味着你应该避免对它们进行解构，
并始终以 `attrs.x` 或 `slots.x` 的方式引用 `property`。
请注意，与 `props` 不同，`attrs` 和 `slots` 是非响应式的。
如果你打算根据 `attrs` 或 `slots` 更改应用副作用，
那么应该在 `onUpdated` 生命周期钩子中执行此操作

# ref响应式变量

# setup内生命周期勾子函数
 - onMounted
 - watch