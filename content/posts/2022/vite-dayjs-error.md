---
title: "React使用Day.js通过Vite发布后报错"
date: 2022-01-12T13:59:21+08:00
draft: false
tags: ["TypeScript", "React", "Vite"]
categories: ["开发记录"]
series: [""]
---

### 问题描述

最近在React项目中使用了 Day.js 来处理需要在 Ant Design 列表中显示的一个时间数据，在本地开发过程中一直没出问题，`vite build`发布以后，列表每次加载数据都会报错：`TypeError: $ is not a function`以及`Error: Minified React error #31;`.

后面发现将使用 Day.js 相关方法注释后，加载数据就恢复正常了。经过搜索，发现[这篇博文](https://www.127m.work/article/31)，虽然文中的报错信息和我遇到的不一样，但是解决方式是相同的。

### 解决方式

不要使用 `import dayjs from 'dayjs'`或者 `import * as dayjs from 'dayjs'` 方式引入 Day.js ，应当改用：

```typescript
import dayjs from 'dayjs/esm/index.js'
```

这样修改后，就可以正常使用了。