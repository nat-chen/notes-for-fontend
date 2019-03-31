# Vue API

## 全局配置

### Vue.config

* 一个对象，包含 Vue 的全局配置
  
#### silent

* 取消 Vue 所有的日志与警告，默认值为 false。
* 使用：Vue.config.silent = true

#### optionMergeStrategies

* 自定义合并策略的选项，默认值为 {}。
* 合并策略选项分别接收在父实例和子实例上定义的该选项的值作为第一个和第二个参数，Vue 实例上下文被作为第三个参数传入。

```js
Vue.config.optionMergeStrategies._my_option = function(parent, child, vm) {
  return child + 1;
};

const Profile = Vue.extend({
  _my_option: 1
});

//Profile.options._my_option = 2
```

#### devtools

* 配置是否允许 vue-devtools 检查代码。开发版本默认为 true，生产版本默认为 false。
* 使用：Vue.config.devtools = true

#### keyCodes

* 给 v-on 自定义键位别名

```js
Vue.config.keyCodes = {
  f1: 112,
  mediaPlayPause: 179,
}
```

## 全局 API

### Vue.extend(options)

* 使用基础 Vue 构造函数，创建一个子类。参数是一个包含组件选项的对象

```js
// 创建构造器
var Profile = Vue.extend({
  template: '<p>{{firstName}} {{lastName}} aka {{alias}}</p>',
  data: function () {
    return {
      firstName: 'Walter',
      lastName: 'White',
      alias: 'Heisenberg'
    }
  }
});
// 创建 Profile 实例，并挂载到一个元素上
new Profile().$mount('#mount-point');
```

### Vue.nextTick(callback, context)

* 在下次 DOM 更新循环结束之后执行延迟回调。在修改数据之后立即使用这个方法，获取更新后的 DOM
* 返回值为 Promise 类型

### Vue.set(target, key, value)

* 向响应式对象中添加一个属性，并确保这个新属性同样是响应式的，且触发视图更新。
* 返回值为设置的值

### Vue.delete(target, key)

* 删除对象的属性。如果对象是响应式的，确保删除能触发更新视图。主要用于避开 Vue 不能检测到属性被删除的限制，但是你应该很少会使用它。

### Vue.directive(id, definition)

* 注册或获取全局指令

```js
// 注册
Vue.directive('my-directive', {
  bind: function() {},
  inserted: function() {},
  update: function() {},
  componentUpdated: function() {},
  unbind: function() {},
});
// 注册 (指令函数)，被 `bind` 和 `update` 调用
Vue.directive('my-directive', function() {});
// getter，返回已注册的指令
var myDirective = Vue.directive('my-directive');s
```

### Vue.filter(id, definition)

* 注册或获取全局过滤器

```js
//返回已注册的过滤器
var myFilter = Vue.filter('my-filter')
```

### Vue.component(id, definition)

### Vue.use(plugin)

* 安装 Vue.js 插件。
* 如果插件是一个对象，必须提供 install 方法。如果插件是一个函数，它会被作为 install 方法。install 方法调用时，会将 Vue 作为参数传入。
* 要在调用 new Vue() 之前被调用。
* 当 install 方法被同一个插件多次调用，插件将只会被安装一次。

### Vue.mixin(mixin)

* 全局注册一个混入，影响注册之后所有创建的每个 Vue 实例（不推荐）

### Vue.compile(template)

* 在 render 函数中编译模板字符串。只在独立构建时有效

```js
var res = Vue.compile('<div><span>{{ msg }}</span></div>');

new Vue({
  data: {
    msg: 'hello'
  },
  render: res.render,
  staticRenderFns: res.staticRenderFns
})
```

### Vue.observable(object)

* 让一个对象可响应。Vue 内部会用它来处理 data 函数返回的对象
* 返回的对象可以直接用于渲染函数和计算属性内，并且会在发生改变时触发相应的更新。也可以作为最小化的跨组件状态存储器，用于简单的场景：

```js
const state = Vue.observable({ 
  count: 0,
});

const Demo = {
  render(h) {
    reutrn h('button', {
      on: { click: () => { state.count++ }}
    }, `count is: ${state.count}`)
  }
}
```

### Vue.version

* 提供字符串形式的 Vue 安装版本号

## 选项/数据

### data

* 递归将 data 的属性转换为 getter/setter，从而让 data 的属性能够响应数据变化。对象必须是纯粹的对象 (含有零个或多个的 key/value 对)：浏览器 API 创建的原生对象，原型上的属性会被忽略。

### propsData

* 创建实例时传递 props。主要作用是方便测试。

## 选项 / DOM

### el

* 将 Vue 实例挂载到已存在的 DOM 元素上。或者通过 vm.$mount() 手动挂载

### render

* 字符串模板的代替方案，允许你发挥 JavaScript 最大的编程能力。
* createElement()

## 选项 / 生命周期钩子

* beforeCreate: 在实例初始化之后，数据观测 (data observer) 和 event/watcher 事件配置之前被调用
* created: 在实例创建完成后被立即调用。已完成数据观测 (data observer)，属性和方法的运算，watch/event 事件回调，但还未挂载。
* beforeMount: 在挂载开始之前被调用：相关的 render 函数首次被调用。
* mounted: el 被新创建的 vm.$el 替换，并挂载到实例上去之后调用该钩子。mounted 不会承诺所有的子组件也都一起被挂载。希望等到整个视图都渲染完毕，在 mounted 中使用 vm.$nextTick；
* beforeUpdate: 数据更新时调用，发生在虚拟 DOM 打补丁之前。在更新之前访问现有的 DOM，比如手动移除已添加的事件监听器。该钩子在服务器端渲染期间不被调用，因为只有初次渲染会在服务端进行
* updated: 由于数据更改导致的虚拟 DOM 重新渲染和打补丁，在这之后会调用该钩子。updated 不会承诺所有的子组件也都一起被重绘。希望等到整个视图都重绘完毕，在 updated 里用 vm.$nextTick。
* activated: keep-alive 组件激活时调用。
* deactivated: keep-alive 组件停用时调用。
* beforeDestroy: 实例销毁之前调用。在这一步，实例仍然完全可用。
* destroyed: Vue 实例销毁后调用。调用后，Vue 实例指示的所有东西都会解绑定，所有的事件监听器会被移除，所有的子实例也会被销毁。

## 选项 / 组合

### parent / children

* 节制地使用 $parent 和 $children，仅作为访问组件的应急方法。更推荐用 props 和 events 实现父子组件通信

### mixins

* 接受一个混入对象的数组

### extends

* 允许声明扩展另一个组件，无需使用 Vue.extend。便于扩展单文件组件

```js
var CompA = {};
var CompB = {
  extends: CompA,
};
```

### provide / inject

* 主要为高阶插件/组件库提供用例，并不推荐直接用于应用程序代码中。
* 允许一个祖先组件向所有子孙后代注入一个依赖，不论组件层次有多深，并在起上下游关系成立的时间里始终生效
* provide 选项应该是一个对象或返回一个对象的函数。包含可注入其子孙的属性。
* inject: 一个字符串数组或是一个对象（key，value）

```js
const s = Symbol();

const Provider = {
  provide() {
    return {
      [s]: 'foo',
    }
  }
};

const Child = {
  inject: { s }
}

//通过设置默认值使其变成可选项
const Child = {
  inject: {
    foo: {
      from: 'bar',
      default: () => [1, 2, 3]
    }
  }
}
```

## 选项 / 其它

### name

* 允许组件模块递归地调用自身。组件在全局用 Vue.component() 注册时，全局 ID 自动作为组件的 name
* 便于调试，有名字的组件有更友好的警告信息。

## 实例属性

### vm.$options

* 用于当前 Vue 实例的初始化选项。需要在选项中包含自定义属性时会有用处

```js
new Vue({
  customOption: 'foo',
  created: function() {
    console.log(this.$options.customOption);
  }
})
```

### vm.$parent

* 父实例，如果当前实例有的话

### vm.$children

* 当前实例的直接子组件。结果不能保证顺序，也不是响应式的

### vm.$slots

* 用来访问被插槽分发的内容。每个具名插槽有其相应的属性（如 v-slot:foo 通过 vm.$slots.foo 找到，默认插槽使用 v-slot:default）

```js
Vue.component('blog-post', {
  render: function(createElement) {
    var header = this.$slots.header;
    var body = this.$slots.default;
    var footer = this.$slots.footer;
    return createElement('div', [
      createElement('header', header),
      createElement('main', body),
      createElement('footer', footer)
    ]);
  }
})
```

### vm.$scopedSlots

* 用来访问作用域插槽。在使用渲染函数开发一个组件时非常有用。

### vm.$refs

* 一个对象，持有注册过 ref 特性的所有 DOM 元素和组件实例

## 实例方法 / 数据

* 观察 Vue 实例变化的一个表达式或计算属性函数。回调函数得到的参数为新值和旧值。表达式只接受监督的键路径。对于更复杂的表达式，用一个函数取代。
* 注意：在变异 (不是替换) 对象或数组时，旧值将与新值相同，因为它们的引用指向同一个对象/数组。Vue 不会保留变异之前值的副本。
* vm.$watch 返回一个取消观察函数，用来停止触发回调

```js
// 键路径
vm.$watch('a.b.c', function(newVal, oldVal) {});

// 函数
vm.$watch(
  function() {
    // 表达式 `this.a + this.b` 每次得出一个不同的结果时
    // 处理函数都会被调用。
    // 这就像监听一个未被定义的计算属性
    return this.a + this.b
  },
  function(newVal, oldVal) {}
)
```

## 指令

* v-pre：跳过这个元素和它的子元素的编译过程。可以用来显示原始 Mustache 标签。跳过大量没有指令的节点会加快编译。
* v-cloak: 这个指令保持在元素上直到关联实例结束编译。和 CSS 规则如 [v-cloak] { display: none } 一起用时，这个指令可以隐藏未编译的 Mustache 标签直到实例准备完毕。

## 特殊特性

### key

* key 的特殊属性主要用在 Vue 的虚拟 DOM 算法，在新旧 nodes 对比时辨识 VNodes。如果不使用 key，Vue 会使用一种最大限度减少动态元素并且尽可能的尝试修复/再利用相同类型元素的算法。使用 key，它会基于 key 的变化重新排列元素顺序，并且会移除 key 不存在的元素。
* 有相同父元素的子元素必须有独特的 key。重复的 key 会造成渲染错误。

## 内置的组件

* component: 渲染一个“元组件”为动态组件。依 is 的值，来决定哪个组件被渲染。
* transition / transition-group
* keep-alive
  * 包裹动态组件时，会缓存不活动的组件实例，而不是销毁它们
  * include: 字符串或正则表达式。只有名称匹配的组件会被缓存。
  * exclude: 字符串或正则表达式。任何名称匹配的组件都不会被缓存。
  * max: 数字，最多可以缓存多少组件实例。
  * 当组件在 keep alive 被切换，它的 activated 和 deactivated 这两个生命周期钩子函数将会被对应执行。
