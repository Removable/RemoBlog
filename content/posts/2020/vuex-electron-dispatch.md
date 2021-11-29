---
title: "vuex-electron使用的小坑"
date: 2020-11-09T12:50:02+08:00
draft: false
tags: ["Vue", "跨平台"]
categories: ["开发记录"]
series: [""]
---

最近使用的electron-vue框架中，自带了vuex-electron模块。本来照着vuex官方文档写的，结果发现this.$store.dispatch('example')无效，根本无法调用成功。

经过反复被坑与Google+Baidu，终于发现vuex-electron这个模块引入了两个插件：

```js
plugins: [ createPersistedState(), createSharedMutations() ]
```

其中createSharedMutations这个插件用于进程间共享数据，因此还需要经过一点小小的调整。目前大概有两种方案：

1.推荐：在electron的主进程的index.js中引入store：

```js
import store from '../renderer/store'
```

（以上import路径仅针对electron-vue的默认结构，具体路径请自行调整）

2.如果不需要共享数据，也可以放弃这个插件，在store/index.js中把createSharedMutations插件的引用给去掉即可。
