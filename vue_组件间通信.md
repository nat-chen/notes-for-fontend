# 组件的通信

## 父对子

* 父组件通过 Prop 向下传递给子组件（单向数据流）
* 当父组件改变 Prop 时，新的值将会自动传递给子组件 （但子组件未必响应式更新）
* 子组件不能改变 Prop 的值:
  * 当传递的值为对象，父组件改变某个属性时，子组件不会响应式更新。应放到 computed 或者是 deep watch
  * 计算属性中不推荐任何数据的改变，最好只进行计算。应 deep watch 或者 computed 的 setter/getter
  * 在子组件中改变父组件传递的 props，因传递的是引用，先执行一次深拷贝 JSON.parse(JSON.stringify())


## 子对父

* 父组件通过 v-on 监听子组件触发的 $emit 事件

## 兄弟间

* EventBus
  * 根实例：this.$root.$on / this.$root.$emit
  * 注册一个全局的 Vue 实例：Vue.prototype.$eventBus = new Vue();
* 使用
  * mounted：事件监听
  * beforeDestory：事件销毁

## 层级嵌套深的组件

* Vuex

* refs：原生 HTML 标签上，访问 DOM 对象；子组件则引用指向组件实例。要在 Dom 加载完成后使用，否则可能获取不到。解决办法是 $nextTick
* ref、parent、children 它们几个的一个缺点就是无法处理跨级组件和兄弟组件；children 为数组

```js
//this.$refs.editor 访问子组件的整个内容 （不推荐，易造成逻辑的混乱，$refs 只在组件渲染完成后才会被赋值，而且它是非响应式的。）
<editor ref="editor" @editorEmit="editorEmit"></editor>
```

* this.$parent/ this.$children

* 声明组件另一种方式：Vue.extend()

```js
let Child = Vue.extedn({
  template: '<h2>{{ content }}</h2>',
  props: {
    content: {
      type: String,
      default: () => { return 'from child' }
    }
  }
});

new Vue({
  el: '#app',
  data: {},
  components: { Child }
});
```

* .sync 语法糖: 在某些情况下，需要对 prop 进行 “双向绑定”。但这将带来维护的问题，因没有明显的改动来源。

```js
<text-document
  v-bind-title="doc.title"
  v-on:update:title="doc.title = $event"
></text-document>

//简写
<text-document
  :title.sync="doc.title"
></text-document>

//子组件触发
this.$emit('update:title', newVal);
```

* $attr 和 $listeners 多久组件的传递

