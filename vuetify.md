# Vuetify

## Grid system

### 简介

* 底层应用弹性布局：flex-box
* 12 点的栅格系统 + 5 种类型的媒体断点

### 用途

* v-container 常应用于整个页面，搭配 fluid 属性占满整个宽度。
* v-layout 应用于页面各个版块间，包含 v-flex
* v-flex 自动应用于子元素 flex: 1 1 auto;
* `<v-layout pa-3 mb-2></v-layout>` 简写于 `<v-layout class="pa-2 mb-2"><v-layout>`

### 特性

* Offset: 设置偏移
* Order: 设置排列顺序
* Direction and Align：指定方向和分布
* 动态布局：v-bind 动态绑定 v-layout
* Grow and Shrink: `grow` 和 `shrink` 特性
* v-flex 与 v-layout 互相嵌套布局
* v-spacer: 占满两个组件的剩余空间
* 指定 HTML 标签名 （三种布局都适用） `<v-container tag="section">`


## Grid List

### 简介

* 栅格列表是v-container组件的一个插件，在各元素间添加沟槽控件

### 用途

* 5 个变量：动态改变 xs 到 xl （`grid-list-sm`）
```js
<v-container v-bind="{[`grid-list-${size}`]: true}">
```

## Icons

* 默认使用 Google 的 Material Icon

## Default Markup

* v-app 为所有应用的必要项，也是入口点。
* 它决定了 grid 的断点布局，可置于 body 标签任何位置，但必须是所有 Vuetify 组件的的父组件；v-content 必须为 v-app 的直接后代
* app 属性设定为固定位置。可用 absolute 属性覆写 app 属性

```js
<v-app>
  <v-navigation-drawer app></v-navigation-drawer>
  <v-toolbar app></v-toolbar>
  <v-content>
    <v-container fluid>
      <router-view></router-view>
    </v-container>
  </v-content>
  <v-footer app></v-footer>
</v-app>
```

## Aspect Ratios

* v-responsive 组件用于指定样式比例

```js
<v-responsive :aspect-ratio="16/9"></v-responsive>
```

## Breakpoints

* 尺寸
  * Extra small - xs - < 600px
  * Small - sm - 600px > & < 960px
  * Medium - md - 960px > & < 1264px
  * Large - lg - 1264px > & < 1904px
  * Extra large - xl - > 1904px

* Breakpoint 对象：`$vuetify.breakpoint` 基于当前的视口尺码

```js
export defaul {
  mounted() {
    console.log(this.$vuetify.breakpoint);
  },
  computed: {
    imageHeight() {
      switch(this.$vuetify.breakpoint.name) {
        case 'xs': return '220px';
        case 'sm': return '400px';
        case 'md': return '500px';
        case 'lg': return '600px';
        case 'xl': return '800px';
      }
    }
  }
}

<v-dialog :fullscreen="$vuetify.breakpoint.xsOnly"></v-dialog>

//自定义 breakpoints
Vue.use(Vuetify, {
  breakpoint: {
    thresholds: {
      xs: 360,
    },
    scrollbarWidth: 10
  }
})
```

## Colors

* 设置背景和字体颜色
* text--{lighten|darken}-{n}

```js
<div class="red"></div>
<span class="red--text"></span>

Vue.use(Vuetify, {
  theme: {
    primary: colors.red.darken1,
    secondary: colors.rend.lighten4,
  }
})
```

## Content

* `blockquote` 引用
* `p` 段落
* `code` 代码
* `var` 变量
* `kdb` 键盘输入

## Theme

* 默认使用 ligh 主题，通过 dark 属性覆写
* 自定义组件的主题

```js
<v-app dark>
</v-app>

import colors from 'vuetify/es5/util/colors';
Vue.use(Vuetify, {
  theme: {
    primary: '#3f51b5', //预定义颜色 colors.grey.darken1
  }
});

//同上
this.$vuetify.theme.primary = '#4caf50';

export default {
  primary: {
    base: colors.purple.base,
    darken1: colos.purple.darken2,
  },
  secondary: colors.indigo,
  tertiary: colors.pink.base //自定义颜色
}
```

## Typography

* font size
* font weight
* text transforms
  * text-capitalize
  * text-lowercase
  * text-none
  * text-uppercase
* text wrapping
  * text-no-wrap
  * text-truncate

## Display

* 可见性：hidden-{breakpoint}-{condition}
* breakpoint: 
  * xs - extra small
  * sm - small
  * md - medium
  * lg - large
  * xl - extra large
* condition: 
  * only: 隐藏元素从 xs 到 xl
  * and-down: 
  * and-up
* hidden-screen-only 和 hidden-print-only
* css 属性 display:
  * d-inline-flex:
  * d-flex
  * d-inline-block
  * d-block
  * d-inline
  * d-none

## Elevation

* 控制沿着z-轴的两个曲面之间的相对深度或距离，共有25个海拔层级，使用类elevation-{n}去设置一个元素的海拔，n 介于 0-24 的整数

## Spacing

* 控制元素的 padding 与 margin
* {property}{direction}-{size} 
  * property: m 为 margin, p 为 padding
  * direction: t 为 top, b 为 bottom, l 为 left, r 为 right, x 为 left and right, y 为 top and bottom, a 为所有方向
  * size: auto/0/1/2/3/4/5

## Text alignment

* 定位你的文本基于视口大小
* text-lg-right/text-md-center/text-sm-left/text-xs-center/text-xs-right

## Scroll

* 编程式的触发滑动，使用 $vuetify 对象的 goTo 方法
* goTo(target, options)
  * target：相对于顶部的偏移 / css 选择器 / DOM 元素
  * options: 包含 duration, easing 或是 offset 的对象
* 路由

```js
import Router from 'vue-router';
import goTo from 'vuetify/lib/components/Vuetify/goTo'

export default new Router({
  scrollBehavior: (to, from, savedPosition) => {
    let scrollTo = 0;
    if (to.hash) {
      scrollTo = to.hash;
    } else if (savedPosition) {
      scrollTo = savedPosition.y;
    }
    return goTo(scrollTo);
  },
  routes: [...]
})
```

## Motion

* transition 属性：
  * 水平方向: slide-x-transtion / slide-x-reverse-transtion
  * 垂直方向：slide-y-transition / "slide-y-reverse-transition
  * 比例：scale-transition
  * 淡入/淡出：fade-transition
  * 展开：v-expand-x-transition / x-expand-transition
  * 过渡的中心：origin

* 过渡的辅助函数

```js
import { createSimpleTranstion } from 'vuetify/es5/util/helpers'
const myTransition = createSimpleTransition('my-transition'); //过渡组件的名字
Vue.component('my-transition', myTransition);
```

```css
.fade-transition
  &-leave-active
    position: absolute

  &-enter-active, &-leave, &-leave-to
    transition: $primary-transition

  &-enter, &-leave-to
    opacity: 0
```

## Resize directive

* v-resize: 指定特定的函数，当窗口大小变化时调用

```js
<v-layout v-resize="onResize" column align-center justify-center>
  <v-subheader>window size</v-subheader>
<v-layout>
```

## Ripple directive

* v-ripple：只适用块级元素和组件
* 自定义颜色：v-ripple="{class: 'primary--text'}"
* 居中：v-ripple="{ center: true }" 
* 禁用：`<v-btn :ripple="false">`

## Scrolling directive

* v-scroll: 当窗口或特定元素滚动时，调用函数

```js
<v-layout v-scroll:#scroll-target="onScroll">
// #scoll-target 为目标元素
//onScroll 为调用的函数
```

## Touch Support

* v-touch: 捕获手势时的回调

```js
<v-layout
  v-touch="{
    left: () => swipe('Left'),
    right: () => swipe('Right'),
    up: () => swipe('Up'),
    down: () => swipe('Down'),
  }"
>
```