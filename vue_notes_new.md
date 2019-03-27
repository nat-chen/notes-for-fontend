* 所有的 Vue 组件都是 Vue 实例，并且接受相同的选项对象 (一些根实例特有的选项除外)。
* 只有当实例被创建时 data 中存在的属性才是响应式的。使用 Object.freeze() 阻止更新
* 不要在选项属性或回调上使用箭头函数，比如 created: () => console.log(this.a) 或 vm.$watch('a', newValue => this.myMethod())。因为箭头函数是和父级上下文绑定在一起的。
* 双大括号文本渲染，v-once 一次性插值，v-html 原始 HTML (易造成 XSS 攻击，对可行内容插值)
* HTML 特性的值为布尔时，存在即为 true。值为 null/undefined/false 时特性为 false
* 动态数据的绑定只能是单个 JS 表达式而非语句
* 动态参数：如下所示。预期为一个字符串，异常情况下值为 null，将示为显性移除绑定
  * 动态参数不能出现空格和引号，考虑用计算属性替代
```js
//eventName = "focus"
<a v-on:[eventName]="doSomething"></a>
```
* 计算属性
  * getter 与 setter
  * 基于依赖进行缓存，只在相关依赖发生 改变时才重新求值 (响应式依赖)
```js
computed: {
  fullName: {
    get: function() {},
    set: function(newValue) {} //vm.fullName = 'nat' 时触发
  }
}
```
* 方法
  * 每当触发重新渲染时，调用方法将总会再次执行函数
* watch
  * 在数据发生变化时执行异步或开销比较大的操作
* class
```js
<div :class="{active: isActive, 'text-danger': hasError }"></div>//对应 true/false
<div :class="classObject"></div>
//数组语法
<div v-bind:class="[activeClass, errorClass]"></div>//对应 class 值
//数组与对象
<div v-bind:class="[{ active: isActive }, errorClass]"></div>
```
* style 同 class
* v-if 
  * 真正的条件渲染，在切换过程中条件块内的事件监听器和子组件适当地被销毁和重建。在初始值为真时，才开始渲染条件块。更高的切换开销。
  * v-for 具有比 v-if 更高的优先级。
  * 切换多个元素时，使用 <template> 元素包裹，而最终的结果不包含 template 元素
  * key 管理可复用的元素: vue 复用已有的元素来高效地渲染。
```js
//label 复用：仅替换文本节点
//input 复用：替换 placeholder
<template v-if="loginType === 'username'">
  <label>Username</label>
  <input placeholder="Enter your username">
</template>
<template v-else>
  <label>Email</label>
  <input placeholder="Enter your email address">
</template>
```
* v-show: 切换 css 的 display。更高的初始渲染开销。
* v-for 
  * 迭代数组和对象: 遍历对象时，按 Object.keys() 的结果遍历。
  * v-for 取整数 / template 元素
  * 接受两个参数，(item, index) / (item, key)
  * v-for 的优先级比 v-if 更高，解决：将 v-if 置于外层元素 (或 <template>)
  * 使用 of 替代 in，最接近 JS 的迭代器语法
  * 在使用 v-for 时提供 key，它是 Vue 识别节点的一个通用机制，key 并不与 v-for 特别关联，
  * 不要使用对象或数组之类的非原始类型值作为 v-for 的 key。用字符串或数类型的值取而代之。
  * v-for 渲染组件时，在特定元素中，只能某些才能作为子元素，比如：ul => li; 解决的办法 is="todo-item"
```js
<template>
  <div id="todo-list-example">
    <form v-on:submit.prevent="addNewTodo">
      <label for="new-todo">Add a todo</label>
      <input
        v-model="newTodoText"
        id="new-todo"
      >
      <button>Add</button>
    </form>
    <ul>
      <li
        is="todo-item"
        v-for="(todo, index) in todos"
        v-bind:key="todo.id"
        v-bind:title="todo.title"
        v-on:remove="todos.splice(index, 1)"
      ></li>
    </ul>
  </div>
</template>
```

* 数组
  * 新数组替换旧数组：用一个含有相同元素的数组去替换原来的数组是非常高效的操作，vue 做了特别的处理
  * 检测限制
    * 索引直接设置一个项
    * 修改数组的长度
```js
Vue.set(vm.items, indexOfItem, newValue);
vm.$set(vm.items, indexOfItem, newValue)
```
* 对象
  * 不能检测对象属性的添加或删除，解决方法用上。
```js
vm.userProfile = Object.assign({}, vm.userProfile, {
  age: 27,
  favoriteColor: 'Vue Green'
})
```
* 事件：
  * 事件修饰符 (方法只有纯粹的数据逻辑，而不是去处理 DOM 事件细节)
    * stop
    * prevent
    * capture
    * self：只当在 event.target 是当前元素自身时触发处理函数
    * once: 可作用与自定义的组件上
    * passive：提升移动端性能 （告诉浏览器不阻止事件的默认行为）
> 修饰符顺序重要，v-on:click.prevent.self 阻止所有的点击，而 v-on:click.self.prevent 只会阻止对元素自身的点击。
  * 按键修饰符：
    * 在监听键盘事件时添加按键修饰符
```js
<input v-on:keyup.enter="submit">
<input v-on:keyup.page-down="onPageDown">
```
  * 按键码
    * enter/tab/delete/esc/space/up/down/left/right
    * keycode 事件已被移除
    * config.keyCodes 自定义按键别名
  * 系统修饰符
    * 仅在按下相应按键时才触发鼠标或键盘事件的监听器
    * ctrl/alt/shift/meta
    * 在和 keyup 事件一起用时，事件触发时修饰键必须处于按下状态，而释放其他按键
  * exact 修饰符
    * 精确的控制系统修饰符组合触发的事件
```js
<!-- 有且只有 Ctrl 被按下的时候才触发 -->
<button @click.ctrl.exact="onCtrlClick">A</button>
```
  * 鼠标按钮修饰符
    * left/right/middle
  * 在 HTML 监听事件
    * 快速定位代码和维护
    * vm 销毁时，所有的事件监听器都会自动删除
* 表单
  * 元素：input, select, textarea
  * v-model 忽略所有表单元素的 value/checked/selected 的初始值
  * text/textarea 使用 value 属性和 input 事件
  * checkbox/radio 使用 checked 属性和 change 事件
  * select 使用 value 作为 prop 属性和 change 事件
  * 复选框
    * 单个复选框：布尔值
    * 多个复选框：同一个数组
  * 单选按钮
    * 选中的值
  * 选择框 
    * 单选：选中的值
    * 多选：同一个数组
  * 修饰符：
    * .lazy: v-model 在每次 input 事件触发后将输入框的值与数据进行同步，从而改用 change 事件进行同步：
    * .number: 数值类型 
    * .trim: 过滤用户输入的首尾空白字符
  
# 组件
* 当传入的 prop 过多时，可组合对象类型。不论何时为对象添加新的属性，都会自动在组件有效
* v-on 监听子组件，在子组件中 $emit 触发。
```js
//通过 event 访问
<button v-on:click="$emit('enlarge-text', 0.1)"></button>
<blog-post v-on:enlarge-text="postFontSize += $event"></blog-post>

<input v-model="searchText">
<input
  :value="searchText"
  @input="searchText = $event.target.value"
/>
```
* v-model
  * 子元素内部 value 的特性绑定到名同为 value 的 prop 上
  * 内部绑定 input 事件，事件触发时新的值抛出
```js
Vue.component('custom-input', {
  props: ['value'],
  template: `
    <input 
      :value="value"
      @input="$emit('input', $event.target.value)"
    >
  `
});
```
* 动态组件: 在不同组件之间进行动态切换

```js
//已注册组件的名字或是组件的选项对象
<component :is="currentTabComponent"></component>
```
* 组件注册
  * 组件名写法：
    * kebab-case (推荐，适用更广)
    * PascalCase
  * 注册
    * 全局：在注册之后可用在任何新创建的 Vue 根实例 (new Vue) 
    * 局部
  * 基础组件：定义通用组件，会被频繁用到。搭配使用 webpack 的 require.context 方法
* Prop
  * 写法：DOM 中使用 kebab-case，组件使用 camelCase 
  * 单向数据流：父级 prop 的更新会向下流动到子组件中，但是反过来则不行。防止意外改变父级组件的状态。每次父组件发生更新时，子组件的所有 prop 将会自动刷新。
    * prop 作为本地 data 数据
    * prop 值需要转换时，定义计算属性 
  * Prop 验证：指定数据类型，在开发别人用到的组件尤为重要
    * 不需要验证时，直接传字符串数组
    * prop 在组件实例创建之前验证，无法引用组件中的数据。
  * 非 Prop 特性：传给组件，但没有相应的 prop 定义的特性
    * 组件接受任意的特性，因组件库的作者不能预料组件应用的场景。特性将被添加到组件的根元素上
  * 对于绝大部分特性，从外部提供的值会替换到组件内部的值。type='text' 替换 type='data'。但 class/style 会被合并。
  * 禁用特性继承：根元素不继承，在选项对象中设置 inheritAttrs: false；可配合实例 $attr 属性使用 (注意 inheritAttrs: false 选项不会影响 style 和 class 的绑定。)
```js
//包含该 prop 没有值的情况在内，都意味着 `true`。
<blog-post is-published></blog-post>

Vue.component('base-input', {
  inheritAttrs: false,
  props: ['lable', 'value'],
  template: `
    <label>
      {{ label }}
      <input
        v-bind="$attrs"
        v-bind:value="value"
        v-on:input="$emit('input', $event.target.value)"
      >
    </label>
  `
});
```
* 自定义事件
  * 始终使用 kebab-case 的事件名
  * 原生事件绑定到组件中 .native 修饰符。但根元素为特定元素不支持该事件，监听器将静默失败；解决办法是：$listeners 属性，包含作用这个组件上所有的监听器
  * v-model: 默认用 value 的 prop 和 input 事件，使用 model 选项避免覆写内部元素的 value 特性
```js
Vue.component('base-checkbox', {
  model: {
    prop:'checked',
    event: 'change',
  },
  props: {
    checked: Boolean, //需要在 props 选项中声明
  },
  template: `
    <input 
      type="checkbox"
      v-bind:checked="checked"
      v-on:change="$emit('change', $event.target.checked)">
  `
});

Vue.component('base-input', {
  inheritAttrs: false,
  props: ['label', 'value'],
  computed: {
    inputListeners: function() { //事件监听器使用计算属性
      var vm = this;
      return Object.assign({},
        this.$listeners, //从父级添加所有的监听器
        {
          input: function(event) { //覆写一些监听器的行为
            vm.$emit('input', event.target.value);
          }
        }
      )
    }
  },
  template: `
    <label>
      {{ label }}
      <input
        v-bind="$attr"
        v-bind:value="value"
        v-on="inputListeners"
      >
    </label>
  `
})
```
* .sync 修饰符
  * 对 prop 进行双向绑定，会难以维护。推荐使用 `update:propName`
  * .sync 不能和表达式使用：v-bind:title.sync=”doc.title + ‘!’” 无效，与 v-model 类似
```js
//父组件
<text-document
  v-bind:title="doc.title"
  v-on:update:title="doc.title = $event"
></text-document>

//子组件
this.$emit('update:title', newTitle);

//缩写
<text-document v-bind:title.sync="doc.title"></text-document>

//同时设置多个 prop 时，使用对象
//格式无法解析：v-bind.sync=”{ title: doc.title }”
<text-document v-bind.sync="doc"></text-document>
```
* 插槽
  * slot 元素作为承载分发内容的出口
  * 后备内容：在没有提供内容时被渲染
```js
<button type="submit">
  <slot>Submit</slot>
</button>
```
  * 具名插槽：name 特性；一个不带 name 的 <slot> 出口会带有隐含的名字“default”。
  * v-slot 只能添加在一个 <template> 上 
```js
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

<base-layout>
  <template v-slot:header>
    <h1>Here might be a page title</h1>
  </template>

  <p>A paragraph for the main content.</p>
  <p>And another one.</p>

  <template v-slot:footer>
    <p>Here's some contact info</p>
  </template>
</base-layout>

//显示定义默认插槽
<template v-slot:default>
  <p>A paragraph for the main content.</p>
  <p>And another one.</p>
</template>
```
  * 作用域插槽
    * 绑定在 <slot> 元素上的特性被称为插槽 prop
    * 默认插槽的缩写语法不能和具名插槽混用，将报错
```js
//user 无法访问到
<current-user>
  {{ user.firstName }}
</current-user>
//解决办法
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

//以上可简写为
<current-user v-slot="slotProps">
  {{ slotProps.user.firstName }}
</current-user>
```
  * 解构插槽 Prop
    * 作用域插槽的内部工作原理是将你的插槽内容包括在一个传入单个参数的函数里，因此 v-slot 值可为任何 JS 表达式
  * 动态插槽名
```js
<base-layout>
  <template v-slot:[dynamicSlotName]></template>
</base-layout>
```
  * v-slot 缩写为 # 符，v-slot:header 缩写为 #header
```js
<current-user #default="{ user }">
  {{ user.firstName }}
</current-user>

//无效，触发警告 
<current-user #="{ user }">
  {{ user.firstName }}
</current-user>
```
  * 插槽 prop 允许我们将插槽转换为可复用的模板，模板可以基于输入的 prop 渲染出不同的内容。在设计封装数据逻辑同时允许父级组件自定义部分布局的可复用组件时是最有用的。

```js
<ul>
  <li
    v-for="todo in filteredTodos"
    v-bind:key="todo.id"
  >
    <slot name="todo" v-bind:todo="todo">
      {{ todo.text }}
    </slot>
  </li>
</ul>

<todo-list v-bind:todos="todos">
  <template v-slot:todo="{ todo }">
    <span if="todo.isComplete">✓</span>
    {{ todo.text }}
  </template>
</todo-list>
```

* 动态组件
  * 在动态组件中使用 keep-alive。将组件缓存而非销毁再重建
  * keep-alive 要求被切换到的组件都有自己的名字，不论是通过组件的 name 选项还是局部/全局注册。
* 异步组件
  * 以工厂函数的方式定义组件并异步解析。组件在被需要渲染时才触发工厂函数，且把结果缓存
```js
//全局
Vue.component(
  'async-webpack-exmaple',
  () => import('./my-async-component')
);
//局部
new Vue({
  //...
  components: {
    'my-component': () => import('./my-async-component')
  }
});

//异步组件工厂函数返回如下格式对象
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
* 处理边界
  * 访问组件和元素
    * 最好不要手动操作 DOM 元素
    * 在子组件中通过 $root 访问根实例
    * 访问父级组件实例：从子组件中访问父组件：$parent；弊端：难以维护
    * 访问子组件实例：ref 特性给子组件赋予一个 ID 引用
      * ref 与 v-for 使用时，得到的引用将会是一个包含了对应数据源的这些子组件的数组。
      * $refs 只会在组件渲染完成之后生效，并且它们不是响应式的。
```js
this.$root.foo //访问 data 数据
this.$root.bar //访问计算属性
this.$root.baz() //组件方法

//在父组件中通过 this.$refs.usernameInput 访问
<base-input ref="usernameInput"></base-input>
//或者，通过在子组件定义 ref，在子组件中访问。
<input ref="input" />
//或者，通过其父级组件定义方法
methods: {
  // 用来从父级组件聚焦输入框
  focus: function() {
    this.$refs.input.focus();
  }
}
this.$refs.usernameInput.focus()
```
  * 依赖注入
    * 用途：访问更深层次的嵌套组件时
    * 提供两个实例选项：provide 和 inject
    * provide: 指定提供给后代组件的数据/方法。
    * inject: 在任何后代组件中接受
    * 负面：
      * 提供的属性是非响应式的
      * 组件耦合，难以维护
```js
provide: function() {
  return {
    getMap: this.getMap, 
  }
}

inject: ['getMap']
```
* 事件监听器
  * $on(eventName, eventHandler) 侦听一个事件
  * $emit(eventName, eventHandler) 触发一个事件
  * $once(eventName, eventHandler) 一次性侦听一个事件
  * $off(eventName, eventHandler) 停止侦听一个事件
```js
mounted: function() {
  this.attachDatePicker('startDateInput');
  this.attachDatePicker('endDateInput');
},
methods: {
  attachDatePicker: function(refName) {
    var picker = new Pikaday({
      field: this.$refs[refName],
      format: 'YYYY-MM-DD'
    });
    //避免将销毁 picker 实例的函数单独放到 beforeDesctroy 钩子函数中 
    this.$once('hook:beforeDestroy', function() {
      picker.destory();
    });
  }
}
//在单个组件中创建过多的建立和清理工作，最好还是创建更多的模块化组件
```
* 循环引用
  * 递归组件
    * 组件是可以在它们自己的模板中调用自身的。当你使用 Vue.component 全局注册一个组件时，这个全局的 ID 会自动设置为该组件的 name 选项。
    * 易造成 max stack size exceeded，请确保递归调用是条件性的 (例如使用一个最终会得到 false 的 v-if)。
  * 组件间的循环引用
    * A 组件内部引用 B 组件，反之亦然
    * 在 Vue.component 全局注册时没有问题，但通过模块系统依赖/导入时（webpack），将报错 ‘Failed to mount component: template or render function not defined.’
```js
//解决办法：全局注册： beforeCrate 注册 B
beforeCreate: function () {
  this.$options.components.TreeFolderContents = require('./tree-folder-contents.vue').default
};
//本地注册：异步 import
components: {
  TreeFolderContents: () => import('./tree-folder-contents.vue')
}
```
* 控制更新
  * $forceUpdate 手动强制更新
  * v-once 创建低开销的静态组件: 将静态内容只计算一次然后缓存起来 (**不要过度使用**)
```js
Vue.component('terms-of-service', {
  template: `
    <div v-once>
      <h1>Terms of Service</h1>
      ... a lot of static content ...
    </div>
  `
})
```
* 混入
  * 分发组件中可复用的功能，混入对象可以包含任意组件选项。
  * 选项合并：（Vue.extend() 采用相同的策略）
    * 数据对象在内部会进行**递归合并**，在和组件的数据发生冲突时以组件数据优先。
    * 同名钩子函数将混合为一个数组，混入对象的钩子将在组件自身钩子之前调用。
    * 值为对象的选项，使用 methods/components/directives 被混合为同一对象。两个对象键名冲突时，取组件对象的键值对。
```js
//mixin.js
var myMixin = {
  create: function() {
    this.hello();
  },
  methods: {
    hello: function() {
      console.log('hello from mixin');
    }
  }
}
// 定义一个使用混入对象的组件
var Component = Vue.extend({
  mixins: [myMixin]
});

var component = new Component();
```
  * 全局混入：Vue.mixin({}) 
    * 谨慎使用全局混入对象，会影响到每个单独创建的 Vue 实例 (包括第三方模板)
  * 自定义选项合并策略：Vue.config.optionMergeStrategies.myOption 添加函数

* 自定义指令
  * 常用于对普通 DOM 元素进行底层操作
```js
//全局注册
Vue.directive('focus', {
  // 当被绑定的元素插入到 DOM 中时，聚焦元素
  inserted: function(el) {
    el.focus();
  }
});

//局部
directives: {
  focus: {
    inserted: function(el) {
      el.focus();
    }
  }
}
```
  * 钩子函数
    * bind: 只调用一次，指令第一次绑定到元素时调用。常用于一次性的初始化设置
    * inserted: 被绑定元素插入父节点时调用 (仅保证父节点存在，但不一定已被插入文档中)。
    * update: 所在组件的 VNode 更新时调用，但是可能发生在其子 VNode 更新之前。指令的值可能发生了改变，也可能没有。通过比较更新前后的值来忽略不必要的模板更新
    * componentUpdated: 指令所在组件的 VNode 及其子 VNode 全部更新后调用。
    * unbind：只调用一次，指令与元素解绑时调用
  * 钩子函数参数
    * el: 指令所绑定的元素，可以用来直接操作 DOM 
  *  binding：一个对象，包含以下属性：
    * name：指令名，不包括 v- 前缀。
    * value：指令的绑定值，例如：v-my-directive="1 + 1" 中，绑定值为 2。
    * oldValue：指令绑定的前一个值，仅在 update 和 componentUpdated 钩子中可用。无论值是否改变都可用。
    * expression：字符串形式的指令表达式。例如 v-my-directive="1 + 1" 中，表达式为 "1 + 1"。
    * arg：传给指令的参数，可选。例如 v-my-directive:foo 中，参数为 "foo"。
    * modifiers：一个包含修饰符的对象。例如：v-my-directive.foo.bar 中，修饰符对象为 { foo: true, bar: true }。
  * vnode：Vue 编译生成的虚拟节点。移步 VNode API 来了解更多详情。
  * oldVnode：上一个虚拟节点，仅在 update 和 componentUpdated 钩子中可用。

* 插件 
  * 
  * 通常会为 Vue 添加全局功能
  * 使用
    *  Vue.use()，需在 new Vue() 启动应用之前完成
  * Vue.use 会自动阻止多次注册相同插件，届时只会注册一次该插件。
```js
// 调用 `MyPlugin.install(Vue)`
Vue.use(MyPlugin);
//或是
Vue.use(MyPlugin, { someOption: true })
```
  * 开发插件: 方法 install，接受第一个参数 Vue 构造器，第二个可选的选项对象
```js
MyPlugin.install = function(Vue, options) {
  // 1. 添加全局方法或属性
  Vue.myGlobalMethod = function() {

  };
  // 2. 添加全局资源
  Vue.derective('my-directive', {
    bind(el, binding, vnode, oldVnode) {

    }
  });
  // 3. 注入组件
  Vue.mixin({
    created: function() {

    }
  });
  // 4. 添加实例方法
  Vue.prototype.$myMethod = function(methodOptions) {

  }
} 
```
* 过滤器
  * 双花括号插值和 v-bind 表达式；添加在 JS 表达式尾部，由管道符号指示。
  * 常用于文本格式化
  * 接收管道符前面的值作为参数，可串联。同时可接收额外的参数 `{{ message | filter('arg1', 'arg2')}}`
```js
//在双花括号中
{{ message | capitalize }}

//v-bind 中
<div v-bind:id="rawId | formatId"></div>

//本地
filters: {
  capitalize: function(value) {}
}

//全局
Vue.filter('capitalize', function(value) {})
```

v-model VS sync
.sync was added after we had added v-model for components and found that people often could use that v-model logic for more than one prop.
So they essentially do the same thing.

<comp :value="bar" @update:value="val => bar = val"></comp>
<comp :value.sync="val => bar = val"></comp>

<compo :value="bar" @input="val => bar = val"></comp>
<comp v-model="bar"></comp>

如果把<comp v-model="total">理解成<comp @input="total=arguments[0]" :value="total">就容易懂了。
区别：1. 自定义事件的名称 2. sync 可以传多个 prop, v-model 只能一个