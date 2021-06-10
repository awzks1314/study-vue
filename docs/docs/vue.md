

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


# reactive

`reactive()`函数接受一个普通对象，返回一个响应式的数据对象
```js
import { reactive } from '@vue/composition-api'

// 创建响应式数据对象，得到的 state 类似于 vue 2.x 中 data() 返回的响应式对象
const state = reactive({ count: 0 })
```

```js
setup() {
     // 创建响应式数据对象
    const state = reactive({count: 0})

     // setup 函数中将响应式数据对象 return 出去，供 template 使用
    return state
}
```

##### isReactive

检查一个对象是否是由`reactive`创建的响应式代理

如果这个代理是由`readonly`创建的，但是又被`reactive`创建的另一个代理包裹了一层，那么同样
也会返回true

# ref

`ref()`函数用来根据给定的值创建一个**响应式的数据对象**，
`ref()`函数调用的返回值是一个对象，这个对象上只包含一个`.value`属性：

```js
import { ref } from '@vue/composition-api'

// 创建响应式数据对象 count，初始值为 0
const count = ref(0)

// 如果要访问 ref() 创建出来的响应式数据对象的值，必须通过 .value 属性才可以
console.log(count.value) // 输出 0
// 让 count 的值 +1
count.value++
// 再次打印 count 的值
console.log(count.value) // 输出 1
```

##### 在reactive对象中访问ref创建的响应式数据

当把`ref()`创建出来的响应式数据对象，挂载到`reactive()`上时，
会自动把响应式数据对象**展开为原始的值**，不需要通过`.value`就可以
直接被访问，例如：

```js
const count = ref(0)
const state = reactive({
  count
})

console.log(state.count) // 输出 0
state.count++ // 此处不需要通过 .value 就能直接访问原始值
console.log(count) // 输出 1

```

新的ref会覆盖旧的ref：

```js
// 创建 ref 并挂载到 reactive 中
const c1 = ref(0)
const state = reactive({
  c1
})

// 再次创建 ref，命名为 c2
const c2 = ref(9)
// 将 旧 ref c1 替换为 新 ref c2
state.c1 = c2
state.c1++

console.log(state.c1) // 输出 10
console.log(c2.value) // 输出 10
console.log(c1.value) // 输出 0
```

# isRef

`isRef()`用来判断某个值是否为`ref()`创建出来的对象；应用场景：
当需要展开某个可能为`ref()`创建出来的值的时候：

```js
	import { isRef } from '@vue/composition-api'
	
	const unwrapped = isRef(foo) ? foo.value : foo
```

# toRefs

`toRefs()` 函数可以将`reactive()`创建出来的响应式对象，转换为普通的对象
，只不过，这个对象上的每一个属性节点，都是`ref()`类型的响应式数据：

```js
import { toRefs } from '@vue/composition-api'

setup() {
    // 定义响应式数据对象
    const state = reactive({
      count: 0
    })

    // 定义页面上可用的事件处理函数
    const increment = () => {
      state.count++
    }

    // 在 setup 中返回一个对象供页面使用
    // 这个对象中可以包含响应式的数据，也可以包含事件处理函数
    return {
      // 将 state 上的每个属性，都转化为 ref 形式的响应式数据
      ...toRefs(state),
      // 自增的事件处理函数
      increment
    }
}
```

页面上可以直接访问`setup()`中return出来的响应式数据：

```js
<template>
  <div>
    <p>当前的count值为：{{count}}</p>
    <button @click="increment">+1</button>
  </div>
</template>
```

# computed

`computed`用来创建计算属性，`computed()`函数的返回值是一个`ref`
的实例。

```js
import { computed } from '@vue/composition-api'
```

##### 创建只读的计算属性

在调用`computed`函数期间，传入一个`function`函数，可以得到一个
只读的计算属性：

```js
// 创建一个 ref 响应式数据
const count = ref(1)

// 根据 count 的值，创建一个响应式的计算属性 plusOne
// 它会根据依赖的 ref 自动计算并返回一个新的 ref
const plusOne = computed(() => count.value + 1)

console.log(plusOne.value) // 输出 2
plusOne.value++ // error
```

##### 创建可读可写的计算属性

在调用`computed`函数期间，传入一个包含`get`和`set`函数的对象，
可以得到一个可读可写的计算属性：

```js
// 创建一个 ref 响应式数据
const count = ref(1)

// 创建一个 computed 计算属性
const plusOne = computed({
  // 取值函数
  get: () => count.value + 1,
  // 赋值函数
  set: val => {
    count.value = val - 1
  }
})

// 为计算属性赋值的操作，会触发 set 函数
plusOne.value = 9
// 触发 set 函数后，count 的值会被更新
console.log(count.value) // 输出 8
```

# watch

对比`watchEffect`，`watch`允许我们：
- 懒执行副作用
- 更具体地说明什么状态的改变会触发侦听器重新运行
- 访问侦听状态变化前后的值

`watch()`函数用来监视某些数据项的变化，从而触发某些特定的操作：

```js
	import { watch } from '@vue/composition-api'
```
```js
const count = ref(0)

// 定义 watch，只要 count 值变化，就会触发 watch 回调
// watch 会在创建时会自动调用一次
watch(() => console.log(count.value))
// 输出 0

setTimeout(() => {
  count.value++
  // 输出 1
}, 1000)
```
##### 监视指定的数据源

监视`reactive`类型的数据源：

```js
// 定义数据源
const state = reactive({ count: 0 })
// 监视 state.count 这个数据节点的变化
watch(
  () => state.count,
  (count, prevCount) => {
    /* ... */
  }
)
```

监视`ref`类型的数据源：

```js
// 定义数据源
const count = ref(0)
// 指定要监视的数据源
watch(count, (count, prevCount) => {
  /* ... */
})
```
### 监视多个数据源

监视`reactive`类型的数据源：

```js
const state = reactive({ count: 0, name: 'zs' })

watch(
  [() => state.count, () => state.name], // Object.values(toRefs(state)),
  ([count, name], [prevCount, prevName]) => {
    console.log(count) // 新的 count 值
    console.log(name) // 新的 name 值
    console.log('------------')
    console.log(prevCount) // 旧的 count 值
    console.log(prevName) // 新的 name 值
  },
  {
    lazy: true // 在 watch 被创建的时候，不执行回调函数中的代码
  }
)

setTimeout(() => {
  state.count++
  state.name = 'ls'
}, 1000)
```

监视`ref`类型的数据源：

```js
const count = ref(0)
const name = ref('zs')

watch(
  [count, name], // 需要被监视的多个 ref 数据源
  ([count, name], [prevCount, prevName]) => {
	console.log(count)
	console.log(name)
	console.log('-------------')
	console.log(prevCount)
	console.log(prevName)
  },
  {
	lazy: true
  }
)

setTimeout(() => {
  count.value++
  name.value = 'xiaomaolv'
}, 1000)
```

##### 清除监视

在`setup()`函数内创建的`watch`监视，会在当前组件被销毁的时候
自动停止。如果想要明确地停止某个监视，可以调用`watch()`函数的返回值即可

```js
// 创建监视，并得到 停止函数
const stop = watch(() => {
  /* ... */
})

// 调用停止函数，清除对应的监视
stop()
```

##### 在watch中清除无效的异步任务

有时候，当被`watch`监视的值发生变化时，或`watch`本身被`stop`之后，
我们期望能够清除那些无效的异步任务，此时，`watch`回调函数中提供了一个
`cleanup registrator function`来执行清除的工作。这个清除函数会在如下
情况下被调用：
- watch被重复执行了
- watch被强制`stop`了

```html
/* template 中的代码 */ <input type="text" v-model="keywords" />
```
```js
// 定义响应式数据 keywords
const keywords = ref('')

// 异步任务：打印用户输入的关键词
const asyncPrint = val => {
  // 延时 1 秒后打印
  return setTimeout(() => {
    console.log(val)
  }, 1000)
}

// 定义 watch 监听
watch(
  keywords,
  (keywords, prevKeywords, onCleanup) => {
    // 执行异步任务，并得到关闭异步任务的 timerId
    const timerId = asyncPrint(keywords)

    // 如果 watch 监听被重复执行了，则会先清除上次未完成的异步任务
    onCleanup(() => clearTimeout(timerId))
  },
  // watch 刚被创建的时候不执行
  { lazy: true }
)

// 把 template 中需要的数据 return 出去
return {
  keywords
}
```
# LifeCycle Hooks(生命周期函数)

新版的生命周期函数，可以按需导入到组件中，且只能在`setup()`函数中
使用，代码示例如下：

```js
import { onMounted, onUpdated, onUnmounted } from '@vue/composition-api'

const MyComponent = {
  setup() {
    onMounted(() => {
      console.log('mounted!')
    })
    onUpdated(() => {
      console.log('updated!')
    })
    onUnmounted(() => {
      console.log('unmounted!')
    })
  }
}
```

##### setup内生命周期勾子函数
- beforeMount -> onBeforeMount
- mounted -> onMounted
- beforeUpdate -> onBeforeUpdate
- updated -> onUpdated
- beforeDestroy -> onBeforeUnmount
- destroyed -> onUnmounted
- errorCaptured -> onErrorCaptured

tip:`setup`是围绕`beforeCreate`和`created`生命周期勾子运行的


# provider & inject

`provide()`和`inject()`可以实现嵌套组件之间的数据传递。这两个函数
只能在`setup()`函数中使用。父级组件中使用`provide()`函数向下传递数据；
子组件中使用`inject()`获取上层传递过来的数据。

##### provider
```js
setup() {
    provide('location', 'North Pole')
    provide('geolocation', {
      longitude: 90,
      latitude: 135
    })
  }
```

确保通过`provide`传递的数据不会被注入的组件更改，我们建议对提供者的property使用`readonly`

```js
import { provide, reactive, readonly, ref } from 'vue'


export default{
	setup() {
	    const location = ref('North Pole')
	    const geolocation = reactive({
	      longitude: 90,
	      latitude: 135
	    })
	
	    const updateLocation = () => {
	      location.value = 'South Pole'
	    }
	
	    provide('location', readonly(location))
	    provide('geolocation', readonly(geolocation))
	    provide('updateLocation', updateLocation)
	  }
}
```

##### 注入inject

```js
setup() {
    const userLocation = inject('location', 'The Universe')
    const userGeolocation = inject('geolocation')

    return {
      userLocation,
      userGeolocation
    }
  }
```
# 共享普通数据

`App.vue`根组件：

```js
<template>
  <div id="app">
    <h1>App 根组件</h1>
    <hr />
    <LevelOne />
  </div>
</template>

<script>
import LevelOne from './components/LevelOne'
// 1. 按需导入 provide
import { provide } from '@vue/composition-api'

export default {
  name: 'app',
  setup() {
    // 2. App 根组件作为父级组件，通过 provide 函数向子级组件共享数据（不限层级）
    //    provide('要共享的数据名称', 被共享的数据)
    provide('globalColor', 'red')
  },
  components: {
    LevelOne
  }
}
</script>
```
`LevelOne.vue`组件：

```js
<template>
  <div>
    <!-- 4. 通过属性绑定，为标签设置字体颜色 -->
    <h3 :style="{color: themeColor}">Level One</h3>
    <hr />
    <LevelTwo />
  </div>
</template>

<script>
import LevelTwo from './LevelTwo'
// 1. 按需导入 inject
import { inject } from '@vue/composition-api'

export default {
  setup() {
    // 2. 调用 inject 函数时，通过指定的数据名称，获取到父级共享的数据
    const themeColor = inject('globalColor')

    // 3. 把接收到的共享数据 return 给 Template 使用
    return {
      themeColor
    }
  },
  components: {
    LevelTwo
  }
}
</script>
```

`LevelTwo.vue`组件：
```js
<template>
  <div>
    <!-- 4. 通过属性绑定，为标签设置字体颜色 -->
    <h5 :style="{color: themeColor}">Level Two</h5>
  </div>
</template>

<script>
// 1. 按需导入 inject
import { inject } from '@vue/composition-api'

export default {
  setup() {
    // 2. 调用 inject 函数时，通过指定的数据名称，获取到父级共享的数据
    const themeColor = inject('globalColor')

    // 3. 把接收到的共享数据 return 给 Template 使用
    return {
      themeColor
    }
  }
}
</script>
```

# 共享ref响应式数据

如下代码实现了点按钮切换主题颜色的功能，主要修改了`App.vue`组件中
的代码，`LevelOne.vue`和`LevelTwo.vue`中的代码不改变

```js
<template>
  <div id="app">
    <h1>App 根组件</h1>

    <!-- 点击 App.vue 中的按钮，切换子组件中文字的颜色 -->
    <button @click="themeColor='red'">红色</button>
    <button @click="themeColor='blue'">蓝色</button>
    <button @click="themeColor='orange'">橘黄色</button>

    <hr />
    <LevelOne />
  </div>
</template>

<script>
import LevelOne from './components/LevelOne'
import { provide, ref } from '@vue/composition-api'

export default {
  name: 'app',
  setup() {
    // 定义 ref 响应式数据
    const themeColor = ref('red')

    // 把 ref 数据通过 provide 提供的子组件使用
    provide('globalColor', themeColor)

    // setup 中 return 数据供当前组件的 Template 使用
    return {
      themeColor
    }
  },
  components: {
    LevelOne
  }
}
</script>
```

# template refs

通过`ref()`还可以引用页面上的元素或组件

##### 元素的引用

```js
<template>
  <div>
    <h3 ref="h3Ref">TemplateRefOne</h3>
  </div>
</template>

<script>
import { ref, onMounted } from '@vue/composition-api'

export default {
  setup() {
    // 创建一个 DOM 引用
    const h3Ref = ref(null)

    // 在 DOM 首次加载完毕之后，才能获取到元素的引用
    onMounted(() => {
      // 为 dom 元素设置字体颜色
      // h3Ref.value 是原生DOM对象
      h3Ref.value.style.color = 'red'
    })

    // 把创建的引用 return 出去
    return {
      h3Ref
    }
  }
}
</script>
```

##### 组件的引用
`TemplateRefOne.vue`中的示例代码如下：
```js
<template>
  <div>
    <h3>TemplateRefOne</h3>

    <!-- 4. 点击按钮展示子组件的 count 值 -->
    <button @click="showNumber">获取TemplateRefTwo中的count值</button>

    <hr />
    <!-- 3. 为组件添加 ref 引用 -->
    <TemplateRefTwo ref="comRef" />
  </div>
</template>

<script>
import { ref } from '@vue/composition-api'
import TemplateRefTwo from './TemplateRefTwo'

export default {
  setup() {
    // 1. 创建一个组件的 ref 引用
    const comRef = ref(null)

    // 5. 展示子组件中 count 的值
    const showNumber = () => {
      console.log(comRef.value.count)
    }

    // 2. 把创建的引用 return 出去
    return {
      comRef,
      showNumber
    }
  },
  components: {
    TemplateRefTwo
  }
}
</script>
```
`TemplateRefTwo.vue`中的示例代码：
```js
<template>
  <div>
    <h5>TemplateRefTwo --- {{count}}</h5>
    <!-- 3. 点击按钮，让 count 值自增 +1 -->
    <button @click="count+=1">+1</button>
  </div>
</template>

<script>
import { ref } from '@vue/composition-api'

export default {
  setup() {
    // 1. 定义响应式的数据
    const count = ref(0)

    // 2. 把响应式数据 return 给 Template 使用
    return {
      count
    }
  }
}
</script>
```

# 响应性

##### 添加响性

为了增加提供值和注入值之间的响应性，我们可以在提供值时使用`ref`或`reactive`

```js
setup() {
    const location = ref('North Pole')
    const geolocation = reactive({
      longitude: 90,
      latitude: 135
    })

    provide('location', location)
    provide('geolocation', geolocation)
  }
```

##### 修改响应式property

当使用响应式提供/注入值时，**建议尽可能，在提供者内保持响应式 property 的任何更改。**


# 渲染函数

### h()参数

h()函数是一个用于创建vnode的使用程序。也许可以更准确地将其命名为`createVNode`，但是由于频繁使用和简洁，
它被称为`h()`。它接受三个参数：

```js
// @returns {VNode}
h(
  // {String | Object | Function | null} tag
  // 一个 HTML 标签名、一个组件、一个异步组件，或者 null。
  // 使用 null 将会渲染一个注释。
  //
  // 必需的。
  'div',

  // {Object} props
  // 与 attribute、prop 和事件相对应的对象。
  // 我们会在模板中使用。
  //
  // 可选的。
  {},

  // {String | Array | Object} children
  // 子 VNodes, 使用 `h()` 构建,
  // 或使用字符串获取 "文本 Vnode" 或者
  // 有 slot 的对象。
  //
  // 可选的。
  [
    'Some text comes first.',
    h('h1', 'A headline'),
    h(MyComponent, {
      someProp: 'foobar'
    })
  ]
)
```

# Proxy

**Proxy是一个包含另一个对象或函数并允许你对其进行拦截的对象**

`new Proxy(target,handler)`

##### isProxy

检查一个对象是否是由`reactive`或者`readonly`方法创建的代理

# 声明响应式状态

要为JavaScript对象创建响应式状态，可以使用`reactive`方法：

```js
import { reactive } from 'vue'

// 响应式状态
const state = reactive({
  count: 0
})
```
创建独立的响应式值为refs

想象一下，我们有一个独立的原始值 (例如，一个字符串)，我们想让它变成响应式的。
当然，我们可以创建一个拥有相同字符串 property 的对象，并将其传递给 reactive。
Vue 为我们提供了一个可以做相同事情的方法 ——ref：

```js
import { ref } from 'vue'

const count = ref(0)	
```

ref 会返回一个可变的响应式对象，该对象作为它的内部值——一个响应式的引用，这就是名称的来源。
此对象只包含一个名为 value 的 property ：

```js
import { ref } from 'vue'

const count = ref(0)
console.log(count.value) // 0

count.value++
console.log(count.value) // 1
```

##### Ref展开

当 ref 作为渲染上下文 (从 setup() 中返回的对象) 上的 property 
返回并可以在模板中被访问时，它将自动展开为内部值。
不需要在模板中追加 .value：

```js
<template>
  <div>
    <span>{{ count }}</span>
    <button @click="count ++">Increment count</button>
  </div>
</template>

<script>
  import { ref } from 'vue'
  export default {
    setup() {
      const count = ref(0)
      return {
        count
      }
    }
  }
</script>
```

# readonly

##### 使用`readonly`防止更改响应式对象

例如，当我们有一个被 provide 的响应式对象时，我们不想让它在注入的时候被改变。
为此，我们可以基于原始对象创建一个只读的 Proxy 对象：

```js
const original = reactive({ count: 0 })

const copy = readonly(original)

watchEffect(() => {
  // 依赖追踪
  console.log(copy.count)
})

// original 上的修改会触发 copy 上的侦听
original.count++

// 无法修改 copy 并会被警告
copy.count++ // warning!
```
##### sReadonly
检查一个对象是否是由`readonly`，创建的只读代理。


# watchEffect

立即执行传入的一个函数，并响应式追踪其依赖，并在其依赖变更时重新运行该函数。

```js
const count = ref(0)

watchEffect(() => console.log(count.value))
// -> 打印出 0

setTimeout(() => {
  count.value++
  // -> 打印出 1
}, 100)
```

##### 停止侦听

当`watchEffect`在组件的`setup()`函数或生命周期勾子被调用时，侦听器会被链接到该组件
的生命周期，并在组件卸载时自动停止。

在一些情况下，也可以显示调用返回值以停止侦听：

```js
const stop = watchEffect(() => {
  /* ... */
})

// 之后
stop()
```

##### 清除副作用

有时副作用函数会执行一些异步的副作用，这些响应需要在其失效时清除(即完成之前状态已经改变了)。
所以侦听副作用传入的函数可以接受一个`onInvalidate`函数作入参，用来注册清除失效时的回调。
当一下情况发生时，这个**失效回调**会被触发：
- 副作用即将重新执行时
- 侦听器被停止(如果在`setup()`或生命周期勾子函数中使用了`watchEffext`，则在卸载组件时)

```js
watchEffect((onInvalidate) => {
  const token = performAsyncOperation(id.value)
  onInvalidate(() => {
    // id 改变时 或 停止侦听时
    // 取消之前的异步操作
    token.cancel()
  })
})
```

我们之所以是通过传入一个函数去注册失效回调，而不是从回调返回它，是因为返回值对于异步错误处理
很重要。

在执行数据请求时，副作用函数往往是一个异步函数。

```js
const data = ref(null)
watchEffect(async () => {
  data.value = await fetchData(props.id)
})
```

# v-model

默认情况下，组件上的`v-model`使用`modelValue`作为prop和`update:modelValue`作为事件。
我们可以通过向`v-model`传递参数来修改这些名词：

```js
<my-component v-model:foo="bar"></my-component>
```

```js
const app = Vue.createApp({})

app.component('my-component', {
  props: {
    foo: String
  },
  template: `
    <input 
      type="text"
      :value="foo"
      @input="$emit('update:foo', $event.target.value)">
  `
})
```

# v-model 修饰符
在 2.x 中，我们对组件 v-model 上的 .trim 等修饰符提供了硬编码支持。
但是，如果组件可以支持自定义修饰符，则会更有用。
在 3.x 中，添加到组件 v-model 的修饰符将通过 `modelModifiers` prop 提供给组件：


























































