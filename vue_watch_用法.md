# Vue 中 watch 选项的使用

## immediate 属性

* 监听数据时，实例初始化时并不会立即执行 watch 的函数，需等到 data 属性的值发生初次改变。
* immediate 默认为 false, 改为 true 后表立即执行。

## deep 属性

* Vue 不能检测到对象属性的添加或删除。在初始化实例时对属性执行 getter/setter 转化过程，属性必须在 data 对象上存在才能让它是响应的。
* 监听的属性为对象类型时，默认情况下，handler 函数值监听它的引用是否发生变化。
* deep: true 表深入观察，监听器会一层层的往下遍历，给对象的所有属性都加上这个监听器，但是性能开销非常大。
* 优化：使用字符串监听；字符串将一层层解析下去，直到遇到属性 a 并添加监听函数。

```js
watch: {
  'obj.a': {
    handler(newName, oldName) {},
    immediate: true,
  }
}
```

## watch 注销

* watch 写在组件选项中，会随着组件的生命周期销毁而销毁。当使用全局时，从一个页面调到另个页面时，需要手动注销：app.$watch 返回的值便是 unWatch 函数，直接调用即可: unWatch()