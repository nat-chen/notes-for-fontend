# Vue 笔记

## 实例

* 当实例被创建时 data 中存在的属性才是响应式。`Object.freeze()` 会阻止修改现有的属性，也意味着响应系统无法再追踪变化。
* Vue 暴露了的实例属性与方法都以 $ 为前缀，以便与用户定义的属性区分开来。(`vm.$data, vm.$watch`)
* 不要在选项属性中使用箭头函数，箭头函数是和父级上下文绑定在一起的。

```js
created: () => console.log(this.a)
vm.$watch('a', newValue => this.myMethod())
```

## 模板语法

* 在底层的实现上，Vue 将模板编译成虚拟 DOM 渲染函数。结合响应系统，Vue 能够智能地计算出最少需要重新渲染多少组件，并把 DOM 操作次数减到最少。（不写模板，可直接写渲染 render 函数，使用可选的 JSX 语法）
* 双大括号：普通文本插值
* 指令特性的值预期是单个 JS 表达式 (v-for 是例外情况)。指令的职责是，当表达式的值改变时，将其产生的连带影响，响应式地作用于 DOM
* v-once 指令：执行一次插值，后续数据的更新不会影响。留心将影响到该节点上的其它数据绑定。
* v-html 指令：站点上动态渲染的任意 HTML 可能会非常危险，因为它很容易导致 XSS 攻击. 绝不要对用户提供的内容使用插值。
* v-bind 指令：动态绑定值；在布尔特性的情况下，存在即为 true。 但值为 null/undefined/false 时，disabled 特性甚至不会被包含在渲染出来的元素中。
* 动态参数：
  1. 用方括号括起来的 JS 表达式作为一个指令的参数; 
  2. 动态参数预期会求出一个字符串，异常情况下值为 null。这个特殊的 null 值可以被显性地用于移除绑定。任何其它非字符串类型的值都将会触发一个警告。
  3. 空格和引号字符无效。

```html
<a v-bind:[attributeName]="url">hello</a>

<a v-on:[eventName]="doSomething"></a>
```

* 修饰符：是以半角句号 . 指明的特殊后缀，用于指出一个指令应该以特殊方式绑定。

```js
<form v-on:submit.prevent="onSumbmit"></form>
```

## 计算属性和监听器

* 模板应只放简单的声明式逻辑，任何复杂的逻辑，应该改用计算属性
* 计算属性
  1. 基于它们的依赖进行缓存的，只在相关依赖发生改变时它们才会重新求值。
  2. 默认只有 getter, 可同时设置 setter；它的 getter 函数是没有副作用 (side effect) 的，这使它更易于测试和理解。
* 侦听属性：观察和响应 Vue 实例上的数据变动；当需要在数据变化时执行异步或开销较大的操作时。
* data/methods 可在代码的执行过程动态添加新的属性

## Class 与 Style 绑定

* Class 绑定：
  * 动态地切换 class。当类名较多时，可使用一个计算属性
  * 在自定义组件上使用 class 属性时，将添加到该组件的根元素上面，同时不会覆盖已经存在的类

```html
<!-- isActive 的真假值决定 -->
<div v-bind:class="{active: isActive }"></div>
<div v-bind:class="[activeClass, errorClass]"></div>
<div v-bind:class="[{ active: isActive }, errorClass ]"></div>
```

* Style 绑定：
  * CSS 属性名可以用驼峰式 (camelCase) 或短横线分隔 (kebab-case，记得用单引号括起来) 来命名
  * 常常结合返回对象的计算属性使用
  * 在使用 v-bind:style 时，Vue 会自动侦测并添加相应的前缀。

```html
<div v-bind:style="{color: activeColor, fontSize: fontSize + 'px' }"></div>
<div v-bind:style="[baseStyles, overridingStyles]"></div>
```

## 条件渲染

* v-if 指令用于条件性地渲染一块内容。
* 当同时需要切换多个元素时，使用一个 template 元素 包裹，而最终渲染的结果将不包含此元素
* 用 key 管理可复用的元素: Vue 会尽可能高效地渲染元素，通常会复用已有元素而不是从头开始渲染。

```html
<template v-if="ok">
  <h1>title</h1>
  <p>Paragraph 1</p>
</tempalte>

<!-- 添加 key 后，表示不要复用 -->
<!-- input 不复用，但 label 仍然会被高地复用 -->
<template v-if="loginType === 'username'">
  <label>Username</label>
  <input placeholder="Enter your username" key="username-input">
</template>
<template v-else>
  <label>Email</label>
  <input placeholder="Enter your email address" key="email-input">
</template>
```

* v-show：简单地切换元素的 CSS 属性 display; v-show 不支持 template 元素，也不支持 v-else
* 比较：
  * v-if 是“真正”的条件渲染，因为它会确保在切换过程中条件块内的事件监听器和子组件适当地被销毁和重建。也是惰性的：如果在初始渲染时条件为假，则什么也不做
  * v-show: 简单地基于 CSS 进行切换。
  * v-if 更高的切换开销。v-show 更高的初始渲染开销

* 当 v-if 与 v-for 一起使用时，v-for 具有比 v-if 更高的优先级。


## 列表渲染

* v-for 指令根据一组数组/对象的选项列表进行渲染。可以用 of 替代 in 作为分隔符，最接近迭代器的语法。
* 在遍历对象时，是按 Object.keys() 的结果遍历，但是不能保证它的结果在不同的 JavaScript 引擎下是一致的。
* 当 Vue.js 用 v-for 正在更新已渲染过的元素列表时，它默认用“就地复用”策略。如果数据项的顺序被改变，Vue 将不会移动 DOM 元素来匹配数据项的顺序， 而是简单复用此处每个元素，并且确保它在特定索引下显示已被渲染过的每个元素。
* 建议尽可能在使用 v-for 时提供 key，除非刻意依赖默认行为以获取性能上的提升。
* 数组更新 (mutation method：改变原数组)，非变异方法 filter()/concat()/slice(); 用一个含有相同元素的数组去替换原来的数组是非常高效的操作。
  * push()
  * pop()
  * shift()
  * unshift()
  * splice()
  * sort()
  * reverse()
* Vue 不能检测变动的数组： 1. 利用索引直接设置一个项。2. 修改数组的长度；应使用如下

```js
Vue.set(vm.items, indexOfItem, newValue);
vm.$set(vm.items, indexOfItem, newValue)
vm.items.splice(indexOfItem, 1, newValue);
```

* Vue 不能检测对象属性的添加或删除，对于已经创建的实例，不能动态添加根级别的响应式属性。使用 Vue.set(object, key, vaue) 向嵌套对象添加响应式属性
* 为已有对象赋予多个新的属性时，应该用两个对象的属性创建一个新的对象 (响应式属性)。

```js
Object.assign({}, a, b);
```

* v-for 取整数范围；搭配 template 渲染多个元素
* 当 v-for 与 v-if 使用时，v-for 比 v-if 有更高的优先级
* v-for 组件，key 为必要项。任何数据都不会被自动传递到组件里，因为组件有自己独立的作用域。为了把迭代数据传递到组件里，要用 props 

## 事件

* 事件修饰符
  * .stop
  * .prevent
  * .capture （使用事件捕获模式）
  * .self （event.target 是当前元素自身时触发）
  * .once （只有once 可作用于自定义的组件事件）
  * .passive (提升移动端的性能)
* 使用修饰符时，顺序很重要；v-on:click.prevent.self 会阻止所有的点击，而 v-on:click.self.prevent 只会阻止对元素自身的点击
* 不要把 .passive 和 .prevent 一起使用，因为 .prevent 将会被忽略，同时浏览器可能会向你展示一个警告。请记住，.passive 会告诉浏览器你不想阻止事件的默认行为。

```html
<button @click="say">Say Hi</button>
<button @click="say('hi')">Say Hi</button>
<button @click="say('hi', $event)">Say Hi</button>
```

* 按键修饰符：在监听键盘事件时。
* 系统修饰键：.ctrl/.alt/.shift/.meta
* .exact 修饰符：许你控制由精确的【系统修饰符】组合触发的事件。

```html
<!-- 即使 Alt 或 Shift 被一同按下时也会触发 -->
<button @click.ctrl="onClick">A</button>

<!-- 有且只有 Ctrl 被按下的时候才触发 -->
<button @click.ctrl.exact="onCtrlClick">A</button>

<!-- 没有任何系统修饰符被按下的时候才触发 -->
<button @click.exact="onClick">A</button>
```

* 鼠标按钮修饰符: .left/.right/.middle

## 表单输入绑定

* v-model 指令在表单 input/textarea/select 元素上创建双向数据绑定。它负责监听用户的输入事件以更新数据，并对一些极端场景进行一些特殊处理。
* v-model 会忽略所有表单元素的 value、checked、selected 特性的初始值而总是将 Vue 实例的数据作为数据来源。
* v-model 在内部使用不同的属性为不同的输入元素并抛出不同的事件：
  * text 和 textarea 元素使用 value 属性和 input 事件；
  * checkbox 和 radio 使用 checked 属性和 change 事件；
  * select 字段将 value 作为 prop 并将 change 作为事件。
* 单个复选框，绑定到布尔值：
* 多个复选框，绑定到同一个数组：
* 单选按钮，选中的值
* 单选选择框，选中的值
* 多选选择框，绑定到一个数组

```html
<input type="checkbox" id="checkbox" v-model="checked">
<label for="checkbox">{{ checked }}</label>

<div id='example-3'>
  <input type="checkbox" id="jack" value="Jack" v-model="checkedNames">
  <label for="jack">Jack</label>
  <input type="checkbox" id="john" value="John" v-model="checkedNames">
  <label for="john">John</label>
  <input type="checkbox" id="mike" value="Mike" v-model="checkedNames">
  <label for="mike">Mike</label>
  <br>
  <span>Checked names: {{ checkedNames }}</span>
</div>

<div id="example-4">
  <input type="radio" id="one" value="One" v-model="picked">
  <label for="one">One</label>
  <br>
  <input type="radio" id="two" value="Two" v-model="picked">
  <label for="two">Two</label>
  <br>
  <span>Picked: {{ picked }}</span>
</div>

<div id="example-5">
  <select v-model="selected">
    <option disabled value="">请选择</option>
    <option>A</option>
    <option>B</option>
    <option>C</option>
  </select>
  <span>Selected: {{ selected }}</span>
</div>

<div id="example-6">
  <select v-model="selected" multiple style="width: 50px;">
    <option>A</option>
    <option>B</option>
    <option>C</option>
  </select>
  <br>
  <span>Selected: {{ selected }}</span>
</div>
```

* 如果 v-model 表达式的初始值未能匹配任何选项，select 元素将被渲染为“未选中”状态。在 iOS 中，这会使用户无法选择第一个选项。因为这样的情况下，iOS 不会触发 change 事件。因此，更推荐像上面这样提供一个值为空的禁用选项。
* .lazy 修饰符：从 input 事件到 change 事件
* .number 修饰符：自动将用户的输入值转为数值类型
* .trim 修饰符：自动过滤用户输入的首尾空白字符

```html
<input v-model.number="age" type="number">
<input v-model.trim="msg">
```

## 组件基础

* 组件是可复用的 Vue 实例，它们与 new Vue 接收相同的选项，例如 data、computed、watch、methods 以及生命周期钩子等。仅有的例外是像 el 这样根实例特有的选项。
* 一个组件的 data 选项必须是一个函数，因此每个实例可以维护一份被返回对象的独立的拷贝：
* 通过 Prop 向子组件传递数据
* 每个组件必须只有一个根元素
* 通过在父组件 v-on 监听子组件实例的事件，在子组件内部调用 $emit 方法并传入事件名称来触发事件
* 在不同组件之间进行动态切换: `<component :is="componentName"></component>`, componentName 可以是已注册组件的名字或是组件的选项对象
* 有些 HTML 元素，诸如 <ul>、<ol>、<table> 和 <select>，对于哪些元素可以出现在其内部是有严格限制，使用特性的 is 特性变通。但此条限制不存在：
  * 字符串 (例如：template: '...')
  * 单文件组件 (.vue)
  * <script type="text/x-template">

```html
<blog-post
  v-on:enlarge-text="postFontSize += 0.1"
></blog-post>

<button v-on:click="$emit('enlarge-text')">Enlarge text</button>
<button v-on:click="$emit('enlarge-text', 0.1)">Enlarge text</button>


<input v-model="searchText">
<input 
  v-bind:value="searchText"
  v-on:input="searchText = $event.target.value"
>

<!-- 将其 value 特性绑定到一个名叫 value 的 prop 上 -->
<!-- 在其 input 事件被触发时，将新的值通过自定义的 input 事件抛出 -->
<custom-input
  v-bind:value="searchText"
  v-on:input="searchText = $event"
></custom-input>

Vue.component('custom-input', {
  props: ['value'],
  template: `
    <input
      v-bind:value="value"
      v-on:input="$emit('input', $event.target.value)"
    >`
})
```

## 组件注册

* 定义组件名的方式有两种：使用 kebab-case / PascalCase；但直接在 DOM (即非字符串的模板) 中使用时只有 kebab-case 是有效的。推荐字母全小写且必须包含一个连字符

```js
Vue.component('my-component-name', {});
Vue.component('MyComponentName', {});
```

* 注册类型：
  * 全局注册: 用在其被注册之后的任何 (通过 new Vue) 新创建的 Vue 根实例，也包括其组件树中的所有子组件的模板中。全局注册的行为必须在根 Vue 实例 (通过 new Vue) 创建之前发生。
  * 局部注册

```js
// 全局注册
Vue.component('my-component-name', {});

//局部注册
new Vue({
  el: '#app',
  components: {
    'component-a': ComponentA, //组件名：选项对象
    'component-b': ComponentB,
  }
})
```

* 基础组件的自动化全局注册：require.context

## Prop

* 单向数据流：父级 prop 的更新会向下流动到子组件中，但是反过来则不行。 防止从子组件意外改变父级组件的状态，从而导致你的应用的数据流向难以理解。
* prop 用来传递一个初始值，作为本地的数据使用；对于一个数组或对象类型的 prop 来说，在子组件中改变这个对象或数组本身将会影响到父组件的状态。

```js
Vue.component('blog-post', {
  props: ['postTitle'],
  template: '<h3>{{ postTitle }}</h3>'
});
<blog-post post-title="hello!"></blog-post>

<!-- 包含该 prop 没有值的情况在内，都意味着 `true`。-->
<blog-post is-published></blog-post> 
```

* Prop 验证

* prop 会在一个组件实例创建之前进行验证，实例的属性 (如 data、computed 等) 在 default 或 validator 函数中是不可用的。
* 非 Props 的特性会被添加到这个组件的根元素上。
* 绝大多数特性来说，从外部提供给组件的值会替换掉组件内部设置好的值。如 type="text" 替换掉 type="date"；但 class / style 会合并。
* 不希望组件的根元素继承特性，在组件的选项中设置 inheritAttrs: false；在撰写基础组件的时候是常会用到的：注意 inheritAttrs: false 选项不会影响 style 和 class 的绑定。

```js
Vue.component('my-component', {
  props: {
    propA: Number,
    propB: [String, Number],
    propC: {
      type: String,
      required: true,
    },
    propD: {
      type: Number,
      default: 100,
    },
    propE: {
      type: Object,
      // 对象或数组默认值必须从一个工厂函数获取
      default: function () {
        return { message: 'hello' }
      }
    },
    // 自定义验证函数
    propF: {
      validator: function (value) {
        // 这个值必须匹配下列字符串中的一个
        return ['success', 'warning', 'danger'].indexOf(value) !== -1
      }
    }
  }
});

Vue.component('base-input', {
  inheritAttrs: false,
  props: ['label', 'value'],
  template: `
    <label>
      {{ label }}
      <input
        v-bind="$attrs"
        v-bind:value="value"
        v-on:input="$emit('input', $event.target.value)"
    ></label>
  `
})
```

## 自定义事件

* 不同于组件和 prop，事件名不存在任何自动化的大小写转换。推荐始终使用 kebab-case 的事件名。
* 自定义组件的 v-model: 一个组件上的 v-model 默认会利用名为 value 的 prop 和名为 input 的事件，但是像单选框、复选框等类型的输入控件可能会将 value 特性用于不同的目的。model 选项可以用来避免这样的冲突：

```js
Vue.component('base-checkbox', {
  model: {
    prop: 'checked',
    event: 'change',
  },
  props: {
    checked: Boolean,
  },
  template: `
    <input
      type="checkbox"
      v-bind:checked="checked"
      v-on:change="$emit('change', $event.target.checked)"
    >
  `
});
<base-checkbox v-model="lovingVue"></base-checkbox>
```

* v-on 的 .native 修饰符: 在一个组件的根元素上直接监听一个原生事件。但当 input 事件作用的根元素为 input 标签时，将静默失败。
* Vue 提供了一个 $listeners 属性，它是一个对象，里面包含了作用在这个组件上的所有监听器。

```js
Vue.component('base-input', {
  inheritAttrs: false,
  props: ['label', 'value'],
  computed: {
    inputListeners: function() {
      var vm = this;
      return Object.assign({},
        //从父级添加所有的监听器
        this.$listeners,
        {
          //添加自定义监听器，
          //或覆写一些监听器的行为
          input: function(event) {
            vm.$emit('input', event.target.value);
          }
        }
      )
    }
  },
  template: `
    <label>
      {{ lable }}
      <input
        v-bind="$attrs"
        v-bind:value="value"
        v-on="inputListeners"
      >
    </label>
  `
})

* .sync 修饰符
子组件：this.$emit('update:title', newTitle)
父组件：
```js
<text-document
  v-bind:title="doc.title"
  v-on:update:title="doc.title=$event"
></text-document>

//简写
<text-document v-bind:title.sync="doc.title"></text-document>
```

## 插槽内容

* slot 元素作为承载分发内容的出口，否则该组件起始标签和结束标签之间的任何内容都会被抛弃。
* 父级模板里的所有内容都是在父级作用域中编译的；子模板里的所有内容都是在子作用域中编译的。

```js
<navigation-link url="/profile">
  Clicking here will send you to: {{ url }}
  <!--
  这里的 `url` 会是 undefined，因为 "/profile" 是
  _传递给_ <navigation-link> 的而不是
  在 <navigation-link> 组件*内部*定义的。
  -->
</navigation-link>
```

* 默认内容

```js
<button type="submit">
  <slot>Submit</slot>
</button>

<submit-button></submit-button>

//当 submit-button 不提供任何插槽时，将渲染为
<button type="submit">
  Submit
</button>
```

* 具名插槽：一个不带 name 的 slot 出口会带有隐含的名字“default”。
* 在向具名插槽提供内容的时候，在一个 template 元素上使用 v-slot 指令，并以 v-slot 的参数的形式提供其名称
* v-slot 只能添加在一个 template 上

```js
<div class="container">
  <header></header>
  <main></main>
  <footer></footer>
</div>

<div class="container">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>

//<template v-slot:default><p>main content</p></tempalte>
<base-layout>
  <template v-slot:header>
    <h1>title</h1>
  </template>
  <p>main content</p>
  <template v-slot:footer>
    <p>footer</p>
  </tempalte>
</base-layout>
```

* 作用域插槽：绑定在 slot 元素上的特性被称为插槽 prop

```js
//代码不会执行成功
<current-user>
  {{ user.firstName }}
</current-user>

//user 只能在组件 current-user 组件上使用，为了让 user 在父级的插槽内容可用，将 user 作为一个 <slot> 元素的特性绑定
<span>
  <slot v-bind:user="user">
   {{ user.lastName }}
  </slot>
</span>

//将包含所有插槽 prop 的对象命名为 slotProps
<current-user>
  <template v-slot:default="slotProps">
    {{ slotProps.user.firstName }}
  </template>
</current-user>
```

## 动态组件和异步组件

* 使用 component is 特性时，在组件之间切换的时候，Vue 都会创建一个新的组件实例。若想保持这些组件的状态，以避免反复重渲染导致的性能问题，使用 keep-alive 元素将其动态组件包裹起来。
* keep-alive 要求被切换到的组件都有自己的名字，不论是通过组件的 name 选项还是局部/全局注册。

```js
<keep-alive>
  <component v-bind:is="currentTabComponent"></component>
</keep-alive>
```

* 异步组件：Vue 允许你以一个工厂函数的方式定义你的组件，这个工厂函数会异步解析你的组件定义。只有在这个组件需要被渲染的时候才会触发该工厂函数，且会把结果缓存起来供未来重渲染

```js
Vue.component('async-example', function(resolve, reject) {
  // 向 `resolve` 回调传递组件定义
  resolve({
    template: '<div>I am async!</div>'
  })
});

//推荐做法是将异步组件和 webpack 的 code-splitting 功能一起配合使用：
Vue.component('async-webpack-example', function(resolve, reject) {
  require(['./my-async-component'], resolve);
});
//或是
Vue.component(
  'async-webpack-example',
  () => import('./my-async-component')
)
//使用局部注册的时候，直接提供一个返回 Promise 的函数
new Vue({
  //...
  components: {
    'my-compnent': () => import('./my-async-component')
  }
})
```

* 处理加载状态

```js
const AsyncComponent = () => ({
  // 需要加载的组件 (应该是一个 `Promise` 对象)
  component: import('./MyComponent.vue'),
  // 异步组件加载时使用的组件
  loading: LoadingComponent,
  // 加载失败时使用的组件
  error: ErrorComponent,
  // 展示加载时组件的延时时间。默认值是 200 (毫秒)
  delay: 200,
  // 如果提供了超时时间且组件加载也超时了，
  // 则使用加载失败时使用的组件。默认值是：`Infinity`
  timeout: 3000
})
```

## 处理边界

* 访问根实例：通过 $root 属性进行访问。

```js
//所有的子组件都可以将这个实例作为一个全局 store 来访问或使用
this.$root.foo
this.$root.baz()
```

* 访问父级组件实例: $parent 属性可以用来从一个子组件访问父组件的实例。虽说可以随时触达父级组件，以替代将数据以 prop 的方式传入子组件的方式。在绝大多数情况下，触达父级组件会使得你的应用更难调试和理解。尽量不要是 this.$parent.$parent 访问，需要向任意更深层级的组件提供上下文信息时推荐依赖注入的原因。

* 访问子组件实例和子元素: $refs 只会在组件渲染完成之后生效，并且它们不是响应式的

```js
<base-input ref="usernameInput"></base-input>; 当 ref 和 v-for 一起使用的时候，得到的引用将会是一个包含了对应数据源的这些子组件的数组。
this.$refs.usernameInput

<input ref="input">
this.$ref.input
```

* 依赖注入: provide 和 inject
  * provide 选项允许我们指定我们想要提供给后代组件的数据/方法。
  * 任何后代组件里，使用 inject 选项来接收指定的想要添加在这个实例上的属性
* 把依赖注入看作一部分“大范围有效的 prop”，除了：
  * 祖先组件不需要知道哪些后代组件使用它提供的属性
  * 后代组件不需要知道被注入的属性来自哪里

```js
provide: function() {
  return {
    getMap: this.getMap,
  }
}

inject: ['getMap']
```

* 程序化的事件侦听器: 
通过 $on(eventName, eventHandler) 侦听一个事件
通过 $once(eventName, eventHandler) 一次性侦听一个事件
通过 $off(eventName, eventHandler) 停止侦听一个事件

> 以上事件监听器不是 dispatchEvent, addEventListener, removeEventListener 的别名

```js
mounted: function() {
  this.attachDatepicker('startDateInput');
  this.attachDatepicker('endDateInput');
},
methods: {
  attachDatepicker: function(refName) {
    var picker = new Pikaday({
      field: this.$refs[refName],
      format: 'YYYY-MM-DD',
    });
    this.$once('hook:beforeDestroy', function() {
      picker.destroy();
    })
  }
}
```

* 循环引用
  * 递归组件可在自己的模板中调用自身，通过 name 选项。
  * 使用 Vue.component 全局注册一个组件时，这个全局的 ID 会自动设置为该组件的 name 选项。

```js
Vue.component('unique-name-of-my-component', {});
```

* 控制更新：强制更新 `$forceUpdate` (慎用)
* 组件包含大量静态内容，可在根元素上添加 `v-once` 确保只计算一次然后缓存下来

## 混入

* 混入 (mixins) 是一种分发 Vue 组件中可复用功能的非常灵活的方式。混入对象可以包含任意组件选项。当组件使用混入对象时，所有混入对象的选项将被混入该组件本身的选项。
* 当组件和混入对象含有同名选项时:
  * 数据对象在内部会进行递归合并，在和组件的数据发生冲突时以组件数据优先。
  * 同名钩子函数将混合为一个数组，因此都将被调用。另外，混入对象的钩子将在组件自身钩子之前调用。
  * 值为对象的选项，例如 methods, components 和 directives，将被混合为同一个对象。两个对象键名冲突时，取组件对象的键值对。
**Vue.extend() 也使用同样的策略进行合并。**
* 全局混入： 一旦使用全局混入对象，将会影响到 所有 之后创建的 Vue 实例
* 自定义选项合并策略：Vue.config.optionMergeStrategies 添加一个函数

```js
//定义一个混入对象
var myMixin = {
  created: function() {
    this.hello();
  },
  methods: {
    hello: function() {
      console.log('hello from mixin');
    }
  }
}
//定义一个使用混入对象的组件
var Component = Vue.extend({
  mixins: [myMixin],
});
var component = new Component();

Vue.mixin({
  created: function() {
    var myOption = this.$options.myOption;
    if (myOption) {
      console.log(myOption);
    }
  }
});
new Vue({
  myOption: 'hello'
})

```

## 自定义指令

* 一个指令定义对象可以提供如下几个钩子函数 (均为可选)
  * bind: 只调用一次，指令第一次绑定到元素时调用。进行一次性的初始化设置
  * inserted: 被绑定元素插入父节点时调用 (仅保证父节点存在，但不一定已被插入文档中)。
  * update: 所在组件的 VNode 更新时调用，但是可能发生在其子 VNode 更新之前。'
  * componentUpdated：指令所在组件的 VNode 及其子 VNode 全部更新后调用。
  * unbind：只调用一次，指令与元素解绑时调用。
* 钩子函数参数 
  * el: 指令所绑定的元素，可以用来直接操作 DOM 
  * binding：一个对象: name, value, oldValue, expression, arg, modifiers
  * vnode：Vue 编译生成的虚拟节点
  * oldVnode：上一个虚拟节点，仅在 update 和 componentUpdated 钩子中可用。
> 除了 el 之外，其它参数都应该是只读的，切勿进行修改。如果需要在钩子之间共享数据，建议通过元素的 dataset 来进行。

```js
//全局注册
Vue.directive('focus', {
  inserted: function(el) {
    el.focus();
  }
});
//注册局部指令：组件中接受一个 directives 的选项：
directives: {
  focus: {
    inserted: function(el) {
      el.focus();
    }
  }
}
//在模板中任何元素使用新的 v-focus 属性
<input v-focus>

<div id="hook-arguments-example" v-demo:foo.a.b="message"></div>
Vue.directive('demo', {
  bind: function (el, binding, vnode) {
    var s = JSON.stringify
    el.innerHTML =
      'name: '       + s(binding.name) + '<br>' +
      'value: '      + s(binding.value) + '<br>' +
      'expression: ' + s(binding.expression) + '<br>' +
      'argument: '   + s(binding.arg) + '<br>' +
      'modifiers: '  + s(binding.modifiers) + '<br>' +
      'vnode keys: ' + Object.keys(vnode).join(', ')
  }
})

new Vue({
  el: '#hook-arguments-example',
  data: {
    message: 'hello!'
  }
})

//简写：在 bind 和 update 时触发相同行为，而不关心其它的钩子
Vue.directive('color-swatch', function(el, binding) {
  el.style.backgroundColor = binding.value;
})

//对象字面量
<div v-demo="{ color: 'white', text: 'hello' }"></div>
Vue.directive('demo', function(el, binding) {
  console.log(binding.value.color, binding.value.text)
})
```

## 渲染函数 & JSX

* render 函数
* 虚拟 DOM （Virtual Node/ VNode）
  * createElement 函数返回一个对象，描述需要渲染什么样的节点，及其子节点。 

```js
Vue.component('anchored-heading', {
  render: function(createElement) {
    return createElement(
      'h' + this.level, //标签名称
      this.$slots.default //子元素数组
    )
  },
  props: {
    level: {
      type: Number,
      required: true
    }
  }
})
```

## 插件

* 通过全局方法 Vue.use() 使用插件。它需要在你调用 new Vue() 启动应用之前完成
* Vue.use 会自动阻止多次注册相同插件，届时只会注册一次该插件

```js
Vue.use(MyPlugin);
//传入选项对象
Vue.use(MyPlugin, { someOption: true });
```

* 开发插件：install(Vue, 可选的选项对象)

```js
MyPlugin.install = function(Vue, options) {
  Vue.myGlobalMethod = function() {};
  Vue.directive('my-directive', {
    bind(el,  binding, vnode, oldVnode) {}
  });
  Vue.mixin({ created: function() {} })
}
```

## 过滤器

* 场景：双花括号插值和 v-bind 表达式
* 串联：message | filterA | filterB
* 过滤器函数接收管道符前面的值作为第一个参数 （message）。message | filterA('arg1', arg2)

```js
//本地
filters: {
  capitalize: function(value) {}
}
//全局, 在 new Vue() 之前
Vue.filter('capitalize', function(value) {})
```


## 生命周期

* new Vue()
* 初始化事件和生命周期
* beforeCreate
* 初始化插入和响应
* created
* 是否有 `el` 选项？ 如果没有，vm.$mount(el) 将会被调用
* 是否有 `template` 选项？
  * 有：编译模板到渲染函数
  * 无：编译 `el` 选项的 outerHTML 作为模板
* beforeMount
* 创建 vm.$el，用 el 代替
* mounted
  * 当 data 改变时，boforeUpdate => 虚拟 DOM 的重新渲染 => updated
* beforeDestory (当 vm.$destroy() 调用)
* 销毁监听器（watchers），子组件（childComponents），事件监听器 （event listeners）
* destoryed









## 递归组件

* 在组件内容自己调用自己，需要设置 name，同时要有跳出递归的条件