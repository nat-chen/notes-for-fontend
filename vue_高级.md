# Vue 高级

## 添加实例属性

### 基本用法

* 在原型上定义它们使其在每个 Vue 的实例中可用
* $ 符号是作为 Vue 所有实例中都可用的属性的约定，避免局部方法/属性产生冲突

```js
Vue.prototype.$appName = 'my App'
```

### 实例

* axiox 请求：`Vue.prototype.$http = axios`

```js
//此处不要使用箭头函数，因上下文同父级绑定到一起
Vue.prototype.$reverseText = function(propName) { 
  this[propertyName] = this[propertyName].
    split('')
    .reverse()
    .join('');
}

new Vue({
  data: {
    message: 'Hello',
  },
  created: function() {
    this.$reverseText('message');
  }
})
```
