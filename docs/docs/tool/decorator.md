
使vue支持ts写法呢,我们需要用到[vue-property-decorator](https://github.com/kaorun343/vue-property-decorator),这个组件完全依赖于vue-class-component.

### 安装
```js
npm i -S vue-property-decorator
```
### 使用
这是一些装饰器和Minxin函数
- @Prop
- @PropSync
- @Model
- @Watch
- @Provide
- @Inject
- @ProvideReactive
- @InjectReactive
- @Emit
- @Ref
- @Component (由 vue-class-component提供)
- Mixins (名为mixins的辅助函数, 由 vue-class-component提供)

# @Component

`@Component`装饰器可以接受一个对象作为参数，可以在对象中声明`components`，`filters`，`directives`等
未提供装饰器的选项，也可以声明`computed`，`watch`等

```js
import { Vue, Component } from 'vue-property-decorator'

@Component({
  filters: {
    toFixed: (num: number, fix: number = 2) => {
      return num.toFixed(fix)
    }
  }
})
export default class MyComponent extends Vue {
  public list: number[] = [0, 1, 2, 3, 4]
  get evenList() {
    return this.list.filter((item: number) => item % 2 === 0)
  }
}

```
# @Prop

### Prop(options: (PropOptions | Constructor[] | Constructor) = {})

@Prop装饰器接收一个参数，这个参数可以有三种写法：

- Constructor，例如String，Number，Boolean等，指定 prop 的类型；
- Constructor[]，指定 prop 的可选类型；
- PropOptions，可以使用以下选项：type，default，required，validator。

```js
// ts写法
import { Vue, Component, Prop } from 'vue-property-decorator'

@Component
export default class YourComponent extends Vue {
  @Prop(Number) readonly propA: number | undefined
  @Prop({ default: 'default value' }) readonly propB!: string
  @Prop([String, Boolean]) readonly propC: string | boolean | undefined
}

```
等同于

```js
export default {
  props: {
    propA: {
      type: Number,
    },
    propB: {
      default: 'default value',
    },
    propC: {
      type: [String, Boolean],
    },
  },
}
```

注：
- 属性的ts类型后面需要加上undefined类型；或者在属性名后面加上!，表示非null和非undefined的断言，否则编译器会给出错误提示；
- 指定默认值必须使用上面例子中的写法，如果直接在属性名后面赋值，会重写这个属性，并且会报错 

### 示例

```html
	// 父组件:
	<template>
	  <div class="Props">
	    <PropComponent :name="name" :age="age" :sex="sex"></PropComponent>
	  </div>
	</template>
	 
	<script lang="ts">
	import {Component, Vue,} from 'vue-property-decorator';
	import PropComponent from '@/components/PropComponent.vue';
	 
	@Component({
	  components: {PropComponent,},
	})
	export default class PropsPage extends Vue {
	  private name = '张三';
	  private age = 1;
	  private sex = 'nan';
	}
	</script>
	 
	// 子组件:
	<template>
	  <div class="hello">
	    name: {{name}} | age: {{age}} | sex: {{sex}}
	  </div>
	</template>
	 
	<script lang="ts">
	import {Component, Vue, Prop} from 'vue-property-decorator';
	 
	@Component
	export default class PropComponent extends Vue {
	   @Prop(String) readonly name!: string | undefined;
	   @Prop({ default: 30, type: Number }) private age!: number;
	   @Prop([String, Boolean]) private sex!: string | boolean;
	}
	</script>
```
# @PropSync
### @PropSync(propName: string, options: (PropOptions | Constructor[] | Constructor) = {})

`@PropSync`装饰器与`@Prop`用法类似，二者的区别在于:
- `@PropSync`装饰器接受两个参数：

	`propName:string`表示父组件传递过来的属性名
	
	`options: Constructor | Constructor[] | PropOptions `与`@Prop`的第一个参数一致；
- `@PropSync`会生成一个新的计算属性 

```js
import { Vue, Component, PropSync } from 'vue-property-decorator'

@Component
export default class MyComponent extends Vue {
  @PropSync('propA', { type: String, default: 'abc' }) public syncedPropA!: string
}

```
等同于

```js
	export default {
	  props: {
	    propA: {
	      type: String,
	      default: 'abc'
	    }
	  },
	  computed: {
	    syncedPropA: {
	      get() {
	        return this.propA
	      },
	      set(value) {
	        this.$emit('update:propA', value)
	      }
	    }
	  }
	}

```
**注：** `@PropSync`**需要配合父组件的**`.sync`**修饰符使用**

### 示例

```html
// 父组件
<template>
  <div class="PropSync">
    <h1>父组件</h1>
    like:{{like}}
    <hr/>
    <PropSyncComponent :like.sync="like"></PropSyncComponent>
  </div>
</template>
 
<script lang='ts'>
import { Vue, Component } from 'vue-property-decorator';
import PropSyncComponent from '@/components/PropSyncComponent.vue';
 
@Component({components: { PropSyncComponent },})
export default class PropSyncPage extends Vue {
  private like = '父组件的like';
}
</script>
 
// 子组件
<template>
  <div class="hello">
    <h1>子组件:</h1>
    <h2>syncedlike:{{ syncedlike }}</h2>
    <button @click="editLike()">修改like</button>
  </div>
</template>
 
<script lang="ts">
import { Component, Prop, Vue, PropSync,} from 'vue-property-decorator';
 
@Component
export default class PropSyncComponent extends Vue {
  @PropSync('like', { type: String }) syncedlike!: string; // 用来实现组件的双向绑定,子组件可以更改父组件穿过来的值
 
  editLike(): void {
    this.syncedlike = '子组件修改过后的syncedlike!'; // 双向绑定,更改syncedlike会更改父组件的like
  }
}
</script>
```
# @Model

### @Model(event?: string, options: (PropOptions | Constructor[] | Constructor) = {})

`@Model`装饰器允许我们在一个组件上自定义`v-model`，接受两个参数：

- `event:string`事件名
- `options: Constructor | Constructor[] | PropOptions` 与`@Prop`的第一个参数一致。

```js
import { Vue, Component, Model } from 'vue-property-decorator'

@Component
export default class MyInput extends Vue {
  @Model('change', { type: String, default: '123' }) public value!: string
}
```

等同于

```js
export default {
  model: {
    prop: 'value',
    event: 'change'
  },
  props: {
    value: {
      type: String,
      default: '123'
    }
  }
}
```
上面例子中指定的是`change`事件，所以我们还需要在`template`中加上相应的时间:

```html
<template>
  <input
    type="text"
    :value="value"
    @change="$emit('change', $event.target.value)"
  />
</template>
```

### 示例

```html
// 父组件
<template>
  <div class="Model">
    <ModelComponent v-model="fooTs" value="some value"></ModelComponent>
    <div>父组件 app : {{fooTs}}</div>
  </div>
</template>
<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import ModelComponent from '@/components/ModelComponent.vue';
 
@Component({ components: {ModelComponent} })
export default class ModelPage extends Vue {
  private fooTs = 'App Foo!';
}
</script>
 
// 子组件
<template>
  <div class="hello">
    子组件:<input type="text" :value="checked" @input="inputHandle($event)"/>
  </div>
</template>
 
<script lang="ts">
import {Component, Vue, Model,} from 'vue-property-decorator';
 
@Component
export default class ModelComponent extends Vue {
   @Model('change', { type: String }) readonly checked!: string
 
   public inputHandle(that: any): void {
     this.$emit('change', that.target.value); // 后面会讲到@Emit,此处就先使用this.$emit代替
   }
}
</script>
```

# @Watch

### @Watch(path: string, options: WatchOptions = {})

`@Wacth`装饰器接受两个参数：

- `path:string`被侦听的属性名；
- `options?:WatchOptions = {} options`可以包含两个属性：

	`immediate?:boolean`侦听开始之后是否立即调用该属性回调函数；
	
	`depp?:boolean`被侦听的对象的属性被改变时，是否调用该回调函数；

**侦听开始，发生在**`beforeCraete`**勾子之后，**`created`**勾子之前**

```js
import { Vue, Component, Watch } from 'vue-property-decorator'

@Component
export default class MyInput extends Vue {
  @Watch('msg')
  public onMsgChanged(newValue: string, oldValue: string) {}

  @Watch('arr', { immediate: true, deep: true })
  public onArrChanged1(newValue: number[], oldValue: number[]) {}

  @Watch('arr')
  public onArrChanged2(newValue: number[], oldValue: number[]) {}
}

```

等同于

```js
export default {
  watch: {
    msg: [
      {
        handler: 'onMsgChanged',
        immediate: false,
        deep: false
      }
    ],
    arr: [
      {
        handler: 'onArrChanged1',
        immediate: true,
        deep: true
      },
      {
        handler: 'onArrChanged2',
        immediate: false,
        deep: false
      }
    ]
  },
  methods: {
    onMsgVhanged(newValue, oldValue) {},
    onArrChange1(newValue, oldValue) {},
    onArrChange2(newValue, oldValue) {}
  }
}

```
### 示例

```html
<template>
  <div class="PropSync">
    <h1>child:{{child}}</h1>
    <input type="text" v-model="child"/>
  </div>
</template>
 
<script lang="ts">
import { Vue, Watch, Component } from 'vue-property-decorator';
 
@Component
export default class WatchPage extends Vue {
  private child = '';
 
  @Watch('child')
  onChildChanged(newValue: string, oldValue: string) {
    console.log(newValue);
    console.log(oldValue);
  }
}
</script>
```

# @Emit

### @Emit(event?:string)

- `@Emit`装饰器接受一个可选参数，该参数是`$Emit`的第一个参数，充当事件名。如果没有提供这个参数，
`$Emit`会将回调函数名的`camelCase`转为`kebab-case`，并将其作为事件名;
- `@Emit`会将回调函数的返回值作为第二个参数，如果返回值是一个`Promise`对象，`$emit`会在`Promise`对象被标记为`resolved`之后触发；
- `@Emit`的回调函数的参数，会放在其返回值之后，一起被`$emit`当做参数使用。 

```js
import { Vue, Component, Emit } from 'vue-property-decorator'

@Component
export default class MyComponent extends Vue {
  count = 0
  @Emit()
  public addToCount(n: number) {
    this.count += n
  }
  @Emit('reset')
  public resetCount() {
    this.count = 0
  }
  @Emit()
  public returnValue() {
    return 10
  }
  @Emit()
  public onInputChange(e) {
    return e.target.value
  }
  @Emit()
  public promise() {
    return new Promise(resolve => {
      setTimeout(() => {
        resolve(20)
      }, 0)
    })
  }
}

```
等同于

```js
export default {
  data() {
    return {
      count: 0
    }
  },
  methods: {
    addToCount(n) {
      this.count += n
      this.$emit('add-to-count', n)
    },
    resetCount() {
      this.count = 0
      this.$emit('reset')
    },
    returnValue() {
      this.$emit('return-value', 10)
    },
    onInputChange(e) {
      this.$emit('on-input-change', e.target.value, e)
    },
    promise() {
      const promise = new Promise(resolve => {
        setTimeout(() => {
          resolve(20)
        }, 0)
      })
      promise.then(value => {
        this.$emit('promise', value)
      })
    }
  }
}

```
### 示例

```html
// 父组件
<template>
  <div class="">
    点击emit获取子组件的名字<br/>
    姓名:{{emitData.name}}
    <hr/>
    <EmitComponent sex='女' @add-to-count="returnPersons" @delemit="delemit"></EmitComponent>
  </div>
</template>
 
<script lang="ts">
import { Vue, Component } from 'vue-property-decorator';
import EmitComponent from '@/components/EmitComponent.vue';
 
@Component({
  components: { EmitComponent },
})
export default class EmitPage extends Vue {
  private emitData = { name: '我还没有名字' };
 
  returnPersons(data: any) {
    this.emitData = data;
  }
 
  delemit(event: MouseEvent) {
    console.log(this.emitData);
    console.log(event);
  }
}
</script>
 
// 子组件
<template>
  <div class="hello">
    子组件:
    <div v-if="person">
      姓名:{{person.name}}<br/>
      年龄:{{person.age}}<br/>
      性别:{{person.sex}}<br/>
    </div>
    <button @click="addToCount(person)">点击emit</button>
    <button @click="delToCount($event)">点击del emit</button>
  </div>
</template>
 
<script lang="ts">
import {
  Component, Vue, Prop, Emit,
} from 'vue-property-decorator';
 
type Person = {name: string; age: number; sex: string };
 
@Component
export default class PropComponent extends Vue {
  private name: string | undefined;
 
  private age: number | undefined;
 
  private person: Person = { name: '我是子组件的张三', age: 1, sex: '男' };
 
  @Prop(String) readonly sex: string | undefined;
 
  @Emit('delemit') private delEmitClick(event: MouseEvent) {}
 
  @Emit() // 如果此处不设置别名字,则默认使用下面的函数命名
  addToCount(p: Person) { // 此处命名如果有大写字母则需要用横线隔开  @add-to-count
    return this.person; // 此处不return,则会默认使用括号里的参数p;
  }
 
  delToCount(event: MouseEvent) {
    this.delEmitClick(event);
  }
}
</script>
```

# @Ref
### @Ref(refKey?:string)

`@Ref`装饰器接收一个可选参数，用来指向元素或子组件的引用信息。如果没有提供这个参数，会使用装饰器后面的属性名充当参数

```js
import { Vue, Component, Ref } from 'vue-property-decorator'
import { Form } from 'element-ui'

@Component
export default class MyComponent extends Vue {
  @Ref() readonly loginForm!: Form
  @Ref('changePasswordForm') readonly passwordForm!: Form

  public handleLogin() {
    this.loginForm.validate(valide => {
      if (valide) {
        // login...
      } else {
        // error tips
      }
    })
  }
}

```
等同于

```js
export default {
  computed: {
    loginForm: {
      cache: false,
      get() {
        return this.$refs.loginForm
      }
    },
    passwordForm: {
      cache: false,
      get() {
        return this.$refs.changePasswordForm
      }
    }
  }
}

```

### 示例

```html
<template>
  <div class="PropSync">
    <button @click="getRef()" ref="aButton">获取ref</button>
    <RefComponent name="names" ref="RefComponent"></RefComponent>
  </div>
</template>
 
<script lang="ts">
import { Vue, Component, Ref } from 'vue-property-decorator';
import RefComponent from '@/components/RefComponent.vue';
 
@Component({
  components: { RefComponent },
})
export default class RefPage extends Vue {
  @Ref('RefComponent') readonly RefC!: RefComponent;
  @Ref('aButton') readonly ref!: HTMLButtonElement;
  getRef() {
    console.log(this.RefC);
    console.log(this.ref);
  }
}
</script>
```

# @Provide+@Inject

```js
// js写法
// 组件A
export default {
    provide() {
        return {
            reload: this.reload;
        }
    }
    methods: {
        reload() {
            // 刷新页面的代码。。。
        }
    }
}
// 组件BCDEFG都可以使用
export default {
    inject: ['reload']
    created() {
        this.reload(); // 这个时候会调用组件A的reload方法
    }
}
```

```js
// ts写法
// 组件A
import { Vue, Component, Provide } from 'vue-property-decorator';
@Component({
    name: 'ComponentA',
    components: {}
})
export default class ComponentA extends Vue {
    @Provide('reload')
    reload = this.reload;
    reload(): void {
        // 刷新页面的代码。。。
    }
}
// 组件BCDE
import { Vue, Component, Inject } from 'vue-property-decorator';
@Component({
    name: 'ComponentB',
    components: {}
})
export default class ComponentB extends Vue {
    @Inject('reload')
    created() {
        this.reload(); // 此时会调用组件A的reload方法
    }
}
```