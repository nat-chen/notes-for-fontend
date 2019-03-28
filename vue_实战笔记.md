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
