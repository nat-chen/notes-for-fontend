# Vue-Router

## 实例

* this.$router 访问路由器，this.$route 访问当前路由

```js
window.history.length > 1 ? this.$router.go(-1) : this.$router.push('/')
```

* 当 router-link 对应的路由匹配成功，将自动设置 class 属性值 .router-link-active

* 例子

```html
<!-- 使用 router-link 组件来导航. -->
<!-- 通过传入 `to` 属性指定链接. -->
<!-- <router-link> 默认会被渲染成一个 `<a>` 标签 -->
<p>
  <router-link to="/foo">Go to Foo</router-link>
  <router-link to="/bar">Go to Bar</router-link>
</p>
<!-- 路由出口 -->
<!-- 路由匹配到的组件将渲染在这里 -->
<router-view></router-view>
```

```js
const Foo = { template: '<div>foo</div>' };
const Bar = { template: '<div>bar</div>' };
// 定义路由
// 每个路由应该映射一个组件。 其中"component" 可以是
// 通过 Vue.extend() 创建的组件构造器，
// 或者，只是一个组件配置对象。
const routes = [
  { path: '/foo', component: Foo },
  { path: '/bar', component: Bar },
];

//创建 router 实例，然后传 `routes` 配置
const router = new VueRouter({
  routes
})

// 创建和挂载根实例。
// 记得要通过 router 配置参数注入路由，
// 从而让整个应用都有路由功能
const app = new Vue({
  router
}).$mount('#app');
```

## 动态路由匹配

* 一个“路径参数”使用冒号 : 标记；通过 this.$route.params 获取 （$route.query 获取查询参数）
* 一个路由中设置多个路径参数：
模式 | 匹配路径 | $route.params
=====
/user/:username	| /user/evan |	{ username: 'evan' }
/user/:username/post/:post_id	| /user/evan/post/123	 | { username: 'evan', post_id: '123' }

```js
///user/foo 和 /user/bar 都将映射到相同的路由。
const User = {
  template: '<div>User</div>'
};

const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User }
  ]
})
```

* 从 /user/foo 导航到 /user/bar，原来的组件实例会被复用。因为两个路由都渲染同个组件，比起销毁再创建，复用则显得更加高效。意味着组件的生命周期钩子不会再被调用。复用组件时，想对路由参数的变化作出响应的话，简单地 watch (监测变化) $route 对象 或是 beforeRouteUpdate 导航守卫

```js
const Use = {
  template: '',
  watch: {
    '$route' (to, from) {}
  }
}

const User = {
  beforeRouteUpdate(to, from, next) {
    // react to route changes...
    // don't forget to call next()
  }
}
```

*　同一个路径可以匹配多个路由，此时，匹配的优先级就按照路由的定义顺序：谁先定义的，谁的优先级就最高。


## 嵌套路由

* 以 / 开头的嵌套路径会被当作根路径。

```js
const router = new VueRouter({
  routes: [
    {
      path: '/user/:id', component: User,
      children: [
        {
          // 当 /user/:id/profile 匹配成功，
          // UserProfile 会被渲染在 User 的 <router-view> 中
          path: 'profile',
          compoennt: UserProfile
        }, {
          // 当 /user/:id/posts 匹配成功
          // UserPosts 会被渲染在 User 的 <router-view> 中
          path: 'posts',
          component: UserPosts
        }, {
          path: '', //空的子路由
          component: UserHome
        }
      ]
    }
  ]
})
```

## 编程式的导航

* 导航到不同的 URL，则使用 router.push 方法。这个方法会向 history 栈添加一个新的记录，所以，当用户点击浏览器后退按钮时，则回到之前的 URL。
* 使用：
  * 声明式：`<router-link :to=""></router-link>`
  * 编程式：`router.push(字符串路径/描述地址的对象)`
* 提供了 path 后 params 会被忽略。因此需提供路由的 name 或手写完整的带有参数的 path （同样的规则也适用于 router-link 组件的 to 属性。）

```js
router.push('home');
router.push({
  path: 'register',
  query: { plan: 'private' }
});
router.push({
  path: 'register',
  params: { userId: 123 }
})

router.push({ name: 'user', params: { userId }}) // -> /user/123
router.push({ path: `/user/${userId}` }) // -> /user/123
// 这里的 params 不生效
router.push({ path: '/user', params: { userId }}) // -> /user
```

* router.push(location, onComplete?, onAbort?)
  * onComplete: 在导航成功完成 (在所有的异步钩子被解析之后)
  * onAbort: 终止 (导航到相同的路由、或在当前导航完成之前导航到另一个不同的路由) 

* router.replace(location, onComplete?, onAbort?)
  * 它不会向 history 添加新记录，而替换掉当前的 history 记录。
  * 声明式：`<router-link :to="..." replace>`	编程式 `router.replace()`

* router.go(n): 在 history 记录中向前或者后退多少步

* router.push/replace/go 效仿了 window.history API: window.history.pushState、 window.history.replaceState 和 window.history.go

## 命名路由

* 设置路由名称

```js
const router = new VueRouter({
  routes: [
    {
      path: '/user/:userId',
      name: 'user',
      component: User
    }
  ]
})

<router-link :to="{ name: 'user', params: { userId: 123 }}">User</router-link>

router.push({ name: 'user', params: { userId: 123 }
```

## 命名视图

```html
<router-view class="view one"></router-view>
<router-view class="view two" name="a"></router-view>
<router-view class="view three" name="b"></router-view>
```

```js
const router = new VueRouter({
  routes: [
    {
      path: '/',
      components: {
        default: Foo,
        a: Bar,
        b: Baz
      }
    }
  ]
})
```

## 重定向和别名

```js
//重定向
const router = new VueRouter({
  routes: [
    { path: '/a', redirect: '/b' }
    //{ path: '/a', redirect: { name: 'foo' }}
    //{ path: '/a', redirect: to => {}} 接收 目标路由 作为参数，return 重定向的 字符串路径/路径对象
  ]
})

//别名
// /a 的别名是 /b，意味着，当用户访问 /b 时，URL 会保持为 /b，但是路由匹配则为 /a，就像用户访问 /a 一样。
const router = new VueRouter({
  routes: [
    { path: '/a', component: A, alias: '/b' }
  ]
})
```

## 路由组件传参

* 取代 $route 的耦合，通过 props 解耦

```js
const User = {
  props: ['id'],
  template: '<div>User {{ id }}</div>'
};

const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User, props: true },
    { // 对于包含命名视图的路由，你必须分别为每个命名视图添加 `props` 选项：
      path: '/user/:id',
      component: { default: User, sidebar: Sidebar },
      props: { default: true, sidebar: false }
    }
  ]
})
```

## 导航守卫

* “导航”表示路由正在发生改变。
* 参数或查询的改变并不会触发进入/离开的导航守卫。可以通过 watch 对象 $route 或是 beforeRouteUpdate 组件内守卫
* 全局前置守卫：router.beforeEach(to, from, next)：
  * 当一个导航触发时，全局前置守卫按照创建顺序调用。守卫是异步解析执行，此时导航在所有守卫 resolve 完之前一直处于 等待中。
  * to: 即将要进入的目标 路由对象
  * from: 当前导航正要离开的路由
  * next: resolve 钩子 (确保要调用 next 方法，否则钩子就不会被 resolved)
    * next(): 进行管道中的下一个钩子。如果全部钩子执行完了，则导航的状态就是 confirmed (确认的)。
    * next(false): 中断当前的导航。
    * next('/') 或是 next({path: ''/}): 跳转到一个不同的地址
    * next(error): 导航会被终止且该错误会被传递给 router.onError() 注册过的回调。
* 全家解析守卫： router.beforeResolve：在导航被确认之前，同时在所有组件内守卫和异步路由组件被解析之后，解析守卫就被调用。
* 全局后置钩子：router.afterEach((to, from) => {})
  * 子不会接受 next 函数也不会改变导航本身：
* 路由独享的守卫: 在路由配置上直接定义 beforeEnter 守卫，与全局前置守卫的方法参数一致

```js
const router = new VueRouter({
  routes: [
    {
      path: '/foo',
      component: Foo,
      beforeEnter: (to, from, next) => {}
    }
  ]
})
```

* 组件内的守卫
  * beforeRouteEnter: 
    * beforeRouteEnter 是支持给 next 传递回调的唯一守卫。对于 beforeRouteUpdate 和 beforeRouteLeave 来说，this 已经可用了，所以不支持传递回调，因为没有必要了。
  * beforeRouteUpdate
  * beforeRouteLeave

```js
const Foo = {
  template: `...`,
  beforeRouteEnter (to, from, next) {
    next(vm => {}) //在导航被确认的时候执行回调，并且把组件实例作为回调方法的参数。
    // 在渲染该组件的对应路由被 confirm 前调用
    // 不！能！获取组件实例 `this`
    // 因为当守卫执行前，组件实例还没被创建
  },
  beforeRouteUpdate (to, from, next) {
    // 在当前路由改变，但是该组件被复用时调用
    // 举例来说，对于一个带有动态参数的路径 /foo/:id，在 /foo/1 和 /foo/2 之间跳转的时候，
    // 由于会渲染同样的 Foo 组件，因此组件实例会被复用。而这个钩子就会在这个情况下被调用。
    // 可以访问组件实例 `this`
  },
  beforeRouteLeave (to, from, next) {
    // 导航离开该组件的对应路由时调用
    // 可以访问组件实例 `this`
    //next(false) 来禁止用户在还未保存修改前突然离开
  }
}
```

* 完整的导航解析流程
  * 导航被触发。
  * 在失活的组件里调用离开守卫。
  * 调用全局的 beforeEach 守卫。
  * 在重用的组件里调用 beforeRouteUpdate 守卫 (2.2+)。
  * 在路由配置里调用 beforeEnter。
  * 解析异步路由组件。
  * 在被激活的组件里调用 beforeRouteEnter。
  * 调用全局的 beforeResolve 守卫 (2.5+)。
  * 导航被确认。
  * 调用全局的 afterEach 钩子。
  * 触发 DOM 更新。
  * 用创建好的实例调用 beforeRouteEnter 守卫中传给 next 的回调函数。

## 路由元信息

* 配置 meta 字段
  * routes 配置中的每个路由对象为 路由记录。路由记录可以是嵌套的，因此，当一个路由匹配成功后，他可能匹配多个路由记录
  * 一个路由匹配到的所有路由记录会暴露为 $route 对象的 $route.matched 数组；遍历 $route.matched 来检查路由记录中的 meta 字段。

```js
router.boforeEach((to, from, next) => {
  if (to.matched.some(record => record.meta.requireAuth)) {
    if (!auth.loggedIn()) {
      next({
        path: '/login',
        query: { redirect: to.fullPath }
      })
    } else {
      next();
    }
  } else {
    next()  // 确保一定要调用 next()
  }
})
```

## 过渡动效

```html
<transition>
  <router-view></router-view>
</transition>
```

## 数据获取

* 两种方式：
  * 导航完成之后获取：先完成导航，然后在接下来的组件生命周期钩子中获取数据。在数据获取期间显示“加载中”之类的指示。
  * 导航完成之前获取：导航完成前，在路由进入的守卫中获取数据，在数据获取成功后执行导航。

```js
export default {
  data () {
    return {
      loading: false,
      post: null,
      error: null
    }
  },
  created () {
    // 组件创建完后获取数据，
    // 此时 data 已经被 observed 了
    this.fetchData()
  },
  watch: {
    // 如果路由有变化，会再次执行该方法
    '$route': 'fetchData'
  },
  methods: {
    fetchData () {
      this.error = this.post = null
      this.loading = true
      // replace getPost with your data fetching util / API wrapper
      getPost(this.$route.params.id, (err, post) => {
        this.loading = false
        if (err) {
          this.error = err.toString()
        } else {
          this.post = post
        }
      })
    }
  }
}

export default {
  data () {
    return {
      post: null,
      error: null
    }
  },
  beforeRouteEnter (to, from, next) {
    getPost(to.params.id, (err, post) => {
      next(vm => vm.setData(err, post))
    })
  },
  // 路由改变前，组件就已经渲染完了
  // 逻辑稍稍不同
  beforeRouteUpdate (to, from, next) {
    this.post = null
    getPost(to.params.id, (err, post) => {
      this.setData(err, post)
      next()
    })
  },
  methods: {
    setData (err, post) {
      if (err) {
        this.error = err.toString()
      } else {
        this.post = post
      }
    }
  }
}
```

## 滚动行为

* 当切换到新路由时，想要页面滚到顶部，或者是保持原先的滚动位置，就像重新加载页面那样。
* 接收 to 和 from 路由对象。第三个参数 savedPosition 当且仅当 popstate 导航 (通过浏览器的 前进/后退 按钮触发) 时才可用。

```js
const router = new VueRouter({
  routes: [...],
  scrollBehavior(to, from, savedPosition) {
    // return 期望滚动到哪个的位置
  }
})
```

## 路由懒加载

* 把不同路由对应的组件分割成不同的代码块，然后当路由被访问的时候才加载对应组件

```js
const Foo = () => import('./Foo.vue');
const router = new VueRouter({
  routes: [{
    path: '/foo', 
    component: Foo
  }]
})
```

* 把组件按组分块，把某个路由下的所有组件都打包在同个异步块 (chunk) 中。使用命名 chunk。

```js
const Foo = () => import(/* webpackChunkName: "group-foo" */ './Foo.vue')
const Bar = () => import(/* webpackChunkName: "group-foo" */ './Bar.vue')
```
