* 多使用 v-for 生成 DOM 模板

```js
data: {
  branches: ['master', 'dev'],
  currentBranch: 'master',
}

<template v-for="branch in branches">
  <input type="radio"
    :id="branch"
    :value="branch"
    name="branch"
    v-model="currentBranch">
  <label :for="branch">{{ branch }}</label>
</template>
```

* watch 选项：直接使用 methods 中的字符串 

```js
watch: {
  currentBranch: 'fetchData',
}
```

* 对后台返回的数据加工时，多使用 filters 特性

* computed 的 set 函数，只能用于设置新的依赖值，不可直接将 value 返回

* 父组件传 prop 给子组件时，将 prop 中的属性作为子组件 data 中初始化的值：
  * 基本类型值：后续父组件更新 prop，子组件已声明的 data 值不受影响
    * 如果子组件要求响应父组件的改变，使用 computed 缓存 prop 值
  * 对象类型：子组件 data 中属性值同 prop 是同一个引用，子组件能访问更新后的 prop 值。
    * Object.assign(vm.object, {a: 1}) 由于原对象上做修改，新的数据可以在子组件访问
    * vm.object = Object.assign({}, vm.object, {a: 1}); 父组件的 prop 已指向另一个对象，而子组件仍指向旧的 prop；解决办法：使用 computed 缓存 prop 值，或是监听 prop 改变后修改子组件的 data 值

* 不要在 computed 或 watch 中，去修改所依赖的数据的值，尤其是 computed；如果这样做，可能导致一个无线循环的触发。
