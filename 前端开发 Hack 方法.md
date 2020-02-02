# 前端开发中的 Hack 方法

## iOS 浏览器 HTMl 内元素点击闪烁的解决办法

这个属性只用于iOS (iPhone和iPad)。
当你点击一个链接或者通过Javascript定义的可点击元素的时候，它就会出现一个半透明的灰色背景。
要重设这个表现，你可以设置-webkit-tap-highlight-color为任何颜色。
想要禁用这个高亮，设置颜色的alpha值为0即可。

```css
-webkit-tap-highlight-color: rgba(0,0,0,0);
```



## 安装 node-sass 失败的解决方法

安装 node-sass 的时候总是会各种不成功，原因就是安装 node-sass 时在 node scripts/install 阶段会从 github.com 上下载一个 .node 文件，但是 GitHub Releases 里的文件都托管在 s3.amazonaws.com 上面，而这个网址在国内总是网络不稳定。
解决方案就是为 node-sass 设置独立的国内仓库源。具体操作如下：

1. 为需要使用 sass 的项目添加独立的 npm 配置文件 .npmrc
2. .npmrc 内写入如下内容：

```
sass_binary_site=https://npm.taobao.org/mirrors/node-sass/
phantomjs_cdnurl=https://npm.taobao.org/mirrors/phantomjs/
electron_mirror=https://npm.taobao.org/mirrors/electron/
puppeteer_download_host = https://npm.taobao.org/mirrors
```

我顺便加入了 phantomjs_cdnurl 和 electron_mirror ， phantomjs 和 electron 这两个库也会因为同样的原因导致安装失败。同理，其他库如果托管到 s3.amazonaws.com 或其他国内访问不稳定的服务器上的都可以使用这种方案解决。
另一个方法是使用 cnpm 。



## 禁止 Safari 将数字识别成电话号码

### 问题描述：

某些版本的 safari 访问网站时，网站上的一串数字颜色会变成了蓝色，点击还会弹出菜单呼叫的选项。

### 原因：

Safari识别电话号码功能会自动将数字识别成电话号码。

### 解决方案：

Safari 有个私有 meta 属性可以禁止 safari 自动识别电话号码。代码如下：

```html
<meta name="format-detection" content="telephone=no" />
```



## 移动端视频自动播放的方案

现在移动端无论android，还是 ios 都不支持网页视频自动播放，以免偷跑用户流量。但是支持在用户有点击行为以后允许视频播放，因此这个方案探索的就是如何在引导用户点击行为以后触发视频的播放。

一般来讲，引导用户点击的事件不一定绑定在视频元素上面，所以，有些元素的点击仍然无法触发视频播放。目前尝试的一种的方案的是，触发和视频 video 元素同级的元素可以触发视频播放事件。那么一种完全的方案就变成触发和视频video 同级元素的点击事件即可。

示例如下(示例使用 vue 和 chimee )：

```js
// 触发播放的元素
// 触发播放
        const dom = document.getElementById('test_header');
        // Native
        let event = null;
        if (window.CustomEvent) {
          event = new CustomEvent('click');
        } else {
          event = document.createEvent('CustomEvent');
          event.initCustomEvent('click', true, true);
        }
        dom.dispatchEvent(event);
```

```vue
// 视频组件 chimee.vue
<script>
import ChimeeMobilePlayer from 'chimee-mobile-player';
import 'chimee-mobile-player/lib/chimee-mobile-player.browser.css';

export default {
  props: {
    isLive: { // 资源类型：直播流、视频
      type: Boolean,
      required: true,
    },
    controls: { // 控制条
      type: Boolean,
      required: false,
    },
    autoplay: {
      type: Boolean,
      required: false,
    },
    src: { // 资源地址
      type: String,
      required: true,
    },
    poster: { // 视频封面图
      type: String,
      required: false,
      default: '',
    },
    state: { // 控制播放的状态值
      type: Boolean,
      required: false,
    },
  },
  watch: {
    src() {
      console.log('这个值改变了', this.src);
      this.init();
    },
    state(val) {
      if (!val) {
        this.chimee.pause();
      }
    },
  },
  mounted() {
    // this.init();
  },
  methods: {
    init() {
      this.chimee = new ChimeeMobilePlayer({
        wrapper: '.video',
        src: this.src,
        box: 'native',
        controls: true,
        autoplay: false,
        playsInline: true,
        poster: this.poster,
        preload: 'auto',
      });
      if (this.poster) {
        this.$refs.chimee.querySelector('video').style.backgroundImage = `url(${this.poster})`;
      }
    },
    play() {
      setTimeout(() => {
        this.chimee.play();
      }, 500);
    },
  }
};
</script>

<template>
  <div class="chimee_box" ref="chimee">
    <header id="test_header" @click="play"></header>
    <div class="video"></div>
  </div>
</template>
```

preload: 'auto' 该属性在 ios 的浏览器上可能无效(微信内浏览器实测无效)



## import/require 模块查找规则

1.node中模块执行的顺序是先加载node自带的核心模块,再加载用户模块(就是我们写的),最后是加载第三方模块;

2.用户模块的查找规则:require("./index"),不写后缀名,查找规则为:index--- index.js --- index.json ---index.node。

3.第三方查找规则分为四种情况:

- 1、node_modules===>和文件一致的文件夹===>查找package.json===>查看是否有main属性,指向路径是否存在
- 2、无main:如果package.json中无main属性,或main的路径不存在,或物package.json文件,node加载index相关的文件(index.js,index.json,index.node)
- 3、无node_modules:如果node_modules找不到对应的模块文件夹,或无该文件夹,则向上一层文件夹查找index
- 4、如果上一级没找到继续上一级,查找到盘符未找到报错;cannot find modules XXX,如果没有npm init -y则会出现node_modules路径在c/用户/用户名



## flex-direction: column 宽度自动变为100%的解决办法

### 描述

当给 container 添加 `flex-direction:column` 以后，所有 item 的 宽度都会变成 100%。

### 解决方案

1. 给 item 设置固定的 width。
2. 给需要自适应宽度 的 item 设置 `align-self:baseline` 。

or

1. 添加 flex-grow: 0

### 解释

只要给 align-self:设置一个值，就能实现这个目的，那大概率是因为 默认值为 `align-self: stretch`
修改这个默认值，就能解决宽度 100% 的问题。



## CSS: flex 布局实现每行固定 div 个数

### 描述

flex 布局要实现每行固定显示子元素的个数

### 解决方案

```html
<div class="parent">
    <div class="child"></div>
    <div class="child"></div>
    <div class="child"></div>
    <div class="child"></div>
    <div class="child"></div>
    <div class="child"></div>
    <div class="child"></div>
    <div class="child"></div>
    <div class="child"></div>
  </div>
<style >
  .parent {
     width: 100%;
     height: 150px;
     display: flex;
     flex-flow: row wrap;
     align-content: flex-start;
   }

   .child {
     box-sizing: border-box;
     background-color: white;
     flex: 0 0 33%; // 50% = 2个；33% = 3个；25% = 4个； 20% = 5个 ...
     height: 50px;
     border: 1px solid red;
   }
</style>
```



