# Vue 风格指南

## 优先级 A

* 组件名应该始终是多个单词的，根组件 App 除外。避免跟现有的以及未来的 HTML 元素相冲突。
* Prop 定义：尽量详细，至少需指定类型。
* v-for 总是用 key
* 避免 v-if 与 v-for 用在同一个元素
  * v-for 比 v-if 具有更高的优先级，哪怕只需渲染小部分数据，但每次都会渲染整个列表。解决办法是：1. 在一个已做筛选的计算属性上遍历。
* 单文件组件的样式设定作用域：scoped 特性
* 私有属性名：始终为自定义的私有属性使用 $_ 前缀。并附带一个命名空间以回避和其它作者的冲突。

```js
var myGreetMixin = {
  methods: {
    $_myGreetMixin_update: function() {}
  }
}
```

## 优先级 B

* 单文件组件的命名大写开头
* 基础组件，即展示类的、无逻辑的或无状态的组件，应以一个特定的前缀开头，比如 Base/App/V
* 单例组件，即每个页面只使用一次，不接受任何 prop。使用 The 前缀表唯一性
* 与父组件紧密耦合的子组件应以父组件名作为前缀命名

```js
components
  |- ToList.vue
  |- TodoListItem.vue
  |- TodoListItemButton.vue
```

* 组件名应该以高级别的单词开头，以描述性的修饰词结尾：因编辑器通常会按字母顺序组织文件，组件间的关系一目了然。

```js
components
  |- SearchButtonClear.vue
```

* 自闭合组件：表没有内容。

```js
// 在单文件组件，字符串模式和 JSX
<MyComponent />

// 在 DOM 模板中
<my-component></my-component>
```

* 组件命名：在单文件中使用 PascalCase，DOM 模板中使用 kebab-case，组件 name 选项使用大小写
* Prop 命名：声明 prop 时：camelCase，而在 DOM 模板中使用 kebab-case
* 多个特性的元素应该分多行撰写，每个特性一行
* 模板中应该只包含简单的表达式，复杂的表达式用计算属性或方法
* 把复杂计算属性分割为尽可能多的更简单的属性

```js
computed: {
  price: function () {
    var basePrice = this.manufactureCost / (1 - this.profitMargin)
    return (
      basePrice -
      basePrice * (this.discountPercent || 0)
    )
  }
}

computed: {
  basePrice: function () {
    return this.manufactureCost / (1 - this.profitMargin)
  },
  discount: function () {
    return this.basePrice * (this.discountPercent || 0)
  },
  finalPrice: function () {
    return this.basePrice - this.discount
  }
}
```

* 非空 HTML 特性值应该始终带引号 (单引号或双引号，选你 JS 里不用的那个)
* 指令缩写 (用 : 表示 v-bind: 和用 @ 表示 v-on:) 应该要么都用要么都不用

## 优先级 C

* 组件/实例的选项应该有统一的顺序
  * el
  * name/parent
  * functional
  * delimiters
  * comments
  * components
  * directives
  * filters
  * extends
  * mixins
  * inheribAttrs
  * model
  * props / propsData
  * data
  * computed
  * watch
  * 生命周期钩子
    * beforeCreate
    * created
    * beforeMount
    * mounted
    * beforeUpdate
    * updated
    * activated
    * deactivated
    * beforeDestroy
    * destroyed
  * methods
  * template / render
  * renderError

* 元素 (包括组件) 的特性应该有统一的顺序
  * is
  * v-for
  * v-if
  * v-else-if
  * v-else
  * v-show
  * v-cloak
  * v-pre
  * v-once
  * id
  * ref
  * key
  * slot
  * v-model
  * v-on
  * v-html
  * v-text

* 组件/实例选项中不要有空行

## 优先级 D

* v-if / v-else-if / v-else 中使用 key：默认情况下，Vue 尽可能高效的更新 DOM。意味着在相同的元素之间切换时，会修补已存在的元素，而不是将旧的元素移除然后在同一个位置添加一个新元素，如果本不相同的元素被识别为相同，则可能不按预期显示。
* 单文件中 style 标签含 scoped 特性时，避免出现元素选择器。元素选择器相对于类选择器性能更差
* 优先通过 prop 和事件进行父子组件之间的通信，而不是 this.$parent 或改变 prop
* 优先通过 Vuex 管理全局状态，而不是通过 this.$root 或一个全局事件总线。


