---
title: element-ui更换主题和按需引入组件
date: 2018-09-04 19:40:30
tags: [vue, element-ui]
categories: vue
---

## element-ui 使用初体验

### 前言

手上有些项目用的`element-ui`，刚好有空琢磨一下怎么减小打包文件大小和打包速度方面，为了演示实验，用 vue-cli 生成初始项目，在这仅对 element-ui 主题和组件方面来优化。

<!-- more -->

```javascript
vue init webpack vuecli
```

### 完整引入

完整地将 ui 和样式引入。

```javascript
import ElementUI from 'element-ui'
import 'element-ui/lib/theme-chalk/index.css'
```

在页面简单使用三个组件，看看效果。

```html
<div class="block">
  <span class="demonstration">区分颜色</span>
  <el-rate v-model="value1" :colors="['#99A9BF', '#F7BA2A', '#FF9900']">
  </el-rate>
</div>
<el-button type="primary" icon="el-icon-edit"></el-button>
<el-row>
  <el-button icon="el-icon-search" circle></el-button>
  <el-button type="primary" icon="el-icon-edit" circle></el-button>
  <el-button type="success" icon="el-icon-check" circle></el-button>
  <el-button type="info" icon="el-icon-message" circle></el-button>
  <el-button type="warning" icon="el-icon-star-off" circle></el-button>
  <el-button type="danger" icon="el-icon-delete" circle></el-button>
</el-row>
```

![](/gb/element-ui更换主题和按需引入组件/0.bmp)

再看一下打包后的资源大小情况`npm run build --report`。

```
Hash: fa47514a97341329a7c0
Version: webpack 3.11.0
Time: 20363ms
                                                  Asset       Size  Chunks                    Chunk Names
                 static/fonts/element-icons.6f0a763.ttf      11 kB          [emitted]
               static/js/vendor.5efcf828140d5dbedda9.js     714 kB       0  [emitted]  [big]  vendor
                  static/js/app.a4a31db472f653b911e7.js      12 kB       1  [emitted]         app
             static/js/manifest.2ae2e69a05c33dfc65f8.js  857 bytes       2  [emitted]         manifest
    static/css/app.f24bb0ae3686720fe2e00c5a2024b8f1.css     185 kB       1  [emitted]         app
static/css/app.f24bb0ae3686720fe2e00c5a2024b8f1.css.map     267 kB          [emitted]
           static/js/vendor.5efcf828140d5dbedda9.js.map    2.73 MB       0  [emitted]         vendor
              static/js/app.a4a31db472f653b911e7.js.map    22.8 kB       1  [emitted]         app
         static/js/manifest.2ae2e69a05c33dfc65f8.js.map    4.97 kB       2  [emitted]         manifest
                                             index.html  508 bytes          [emitted]
```

发现打包后提取公共模块`static/js/vendor.js`有 714kb

再看一下各个模块占用情况：

![](/gb/element-ui更换主题和按需引入组件/1.bmp)

发现`elment-ui.common.js`占用最大。所有模块资源总共有 708kb。怎么才能减小打包后的大小呢？很容易就会想到 ui 的引入和样式的引入中，实际我们只使用了三个组件，却整体都被打包了，在这里引入这三个组件即可。

### 按需引入组件样式

新建一个`element-variables.scss`文件（为什么是 SCSS 文件，后面自定义主题会用到）。

```
/*icon字体路径变量*/
$--font-path: "~element-ui/lib/theme-chalk/fonts";

/*按需引入用到的组件的scss文件和基础scss文件*/
@import "~element-ui/packages/theme-chalk/src/base.scss";
@import "~element-ui/packages/theme-chalk/src/rate.scss";
@import "~element-ui/packages/theme-chalk/src/button.scss";
@import "~element-ui/packages/theme-chalk/src/row.scss";
```

### 按需引入组件

新建一个`elementConfig.js`文件，将项目用到的 element 组件引入。

```javascript
import { Rate, Row, Button } from 'element-ui'

export default {
  install(V) {
    V.use(Rate)
    V.use(Button)
    V.use(Row)
  }
}
```

### 第一次优化后打包分析

将以上`element-variables.scss`和`elementConfig.js`引入到`main.js`中。

```javascript
import ElementUI from '@/assets/js/elementConfig'
import '@/assets/css/element-variables.scss'

Vue.use(ElementUI)
```

貌似上面一切都很顺理成章，打包后大小会减小。

```javascript
Hash: 3ba9b74482f121efd3aa
Version: webpack 3.11.0
Time: 18854ms
                                                  Asset       Size  Chunks                    Chunk Names
                 static/fonts/element-icons.6f0a763.ttf      11 kB          [emitted]
               static/js/vendor.11c71f168a2d61b547a0.js     714 kB       0  [emitted]  [big]  vendor
                  static/js/app.dbb5b49dad2d42b3598c.js    11.2 kB       1  [emitted]         app
             static/js/manifest.2ae2e69a05c33dfc65f8.js  857 bytes       2  [emitted]         manifest
    static/css/app.bf52525d6279e7fb87b4db770d119a8d.css    25.7 kB       1  [emitted]         app
static/css/app.bf52525d6279e7fb87b4db770d119a8d.css.map      63 kB          [emitted]
           static/js/vendor.11c71f168a2d61b547a0.js.map    2.73 MB       0  [emitted]         vendor
              static/js/app.dbb5b49dad2d42b3598c.js.map      21 kB       1  [emitted]         app
         static/js/manifest.2ae2e69a05c33dfc65f8.js.map    4.97 kB       2  [emitted]         manifest
                                             index.html  508 bytes          [emitted]
```

结果可知，`static/js/vendor.js`还是 714kb！

再看各个模块占用情况：

![](/gb/element-ui更换主题和按需引入组件/2.bmp)

WHAT?竟然模块都没什么变化，岂不是竹篮打水，事与愿违。

### 再次打包优化尝试

后来查到有人同样遇到这个问题，提出一个[issues#6362](https://github.com/ElemeFE/element/issues/6362#issuecomment-323517538)，原来只引入需要的`element-ui`组件，`webpack`还是把整体的 ui 库和样式都打包了，需要一个`webpack`的`babel`插件`babel-plugin-component`，这样才能真正按需引入打包。这块其实被写到官方文档更换 [自定义主题](http://element-cn.eleme.io/#/zh-CN/component/custom-theme) 的配置了。

于是`npm i babel-pugin-componet -D`安装后，在增加`.babelrc`文件插件配置

```javascript
{
  "presets": [
    ["env", {
      "modules": false,
      "targets": {
        "browsers": ["> 1%", "last 2 versions", "not ie <= 8"]
      }
    }],
    "stage-2"
  ],
  "plugins": ["transform-vue-jsx", "transform-runtime", ["component", {
    "libraryName": "element-ui",
    "styleLibraryName": "theme-chalk"
  }]]
}
```

页面运行正常，再次打包。

```javascript
Hash: 9cc71dead6d7646c9ed4
Version: webpack 3.11.0
Time: 9963ms
                                                  Asset       Size  Chunks             Chunk Names
                 static/fonts/element-icons.6f0a763.ttf      11 kB          [emitted]
                  static/js/app.77c0883f4f0fc0bf5cbc.js    11.4 kB       0  [emitted]  app
               static/js/vendor.942130fd13274b901889.js     126 kB       1  [emitted]  vendor
             static/js/manifest.2ae2e69a05c33dfc65f8.js  857 bytes       2  [emitted]  manifest
    static/css/app.b140020e5dbee406ae70780b43ba7ddc.css    27.8 kB       0  [emitted]  app
static/css/app.b140020e5dbee406ae70780b43ba7ddc.css.map    91.4 kB          [emitted]
              static/js/app.77c0883f4f0fc0bf5cbc.js.map    21.1 kB       0  [emitted]  app
           static/js/vendor.942130fd13274b901889.js.map     613 kB       1  [emitted]  vendor
         static/js/manifest.2ae2e69a05c33dfc65f8.js.map    4.97 kB       2  [emitted]  manifest
                                             index.html  508 bytes          [emitted]
```

`static/js/vendor.js`确实变小了，126kB。再来看各个模块分析图。

![](/gb/element-ui更换主题和按需引入组件/3.bmp)

模块总共 135.03KB，少了 5 倍！

### 更换主题

`element-ui`的`theme-chalk`使用`SCSS`编写，如果在自己的项目中也是用`SCSS`，那么可以直接在项目中改变样式变量，因此可以在前面新建的`element-variables.scss`文件用新的主题颜色变量覆盖即可。

```
/*主题颜色变量*/
$--color-primary: #f0f;

/*icon字体路径变量*/
$--font-path: "~element-ui/lib/theme-chalk/fonts";
```

可能你已经注意到了，这里没有分别引入用到的组件样式了，是因为`babel-plugin-component`帮我们按需引入了对应的样式。

现在我们的主题就变成了

![](/gb/element-ui更换主题和按需引入组件/4.bmp)

如果你没有用到`SCSS`，可以用`element-theme`主题编译插件，生成自定义主题文件引入。

完~ps：个人见解有限，欢迎指正。
