---
title: vue-cli中遇到的坑
date: 2017-09-02 09:00:30
tags: vue-cli
categories: vue
---

项目构建自动化，错误查起来越来越不知所措，坑很多，踩过后要记录，防止踩第二遍

#### vue 单文件@import css 文件，不加`~`会报错

<!-- more -->

```
<style lang="stylus" scoped>
@import 'assets/css/variable'
</style>
```

报错：

```
[HMR] bundle has 1 errors
172:176 ./~/css-loader?{"minimize":false,"sourceMap":false}!./~/vue-loader/lib/style-compiler?{"vue":true,"id":"data-v-be4708e4","scoped":true,"hasInlineConfig":false}!./~/stylus-loader?{"sourceMap":false}!./~/vue-loader/lib/selector.js?type=styles&index=0!./src/components/views/programs/Programs.vue
Module build failed: Error: D:\appSoft\wampserver\wamp64\www\iHomed_vue\src\components\views\programs\Programs.vue:200:9
   196| }
   197| </script>
   198|
   199| <style lang="stylus" scoped>
   200| @import 'assets/css/variable'
----------------^
   201|
```

正确写法`@import '~assets/css/variable'`

#### vue-cli 中 config/index.js 配置说明

```
module.exports = {
  build: {
    env: require('./prod.env'), // 使用 config/prod.env.js 中定义的编译环境
    index: path.resolve(__dirname, '../dist/index.html'), // 编译输入的 index.html 文件
    assetsRoot: path.resolve(__dirname, '../dist'), // 编译输出的静态资源路径
    assetsSubDirectory: 'static', // 编译输出的二级目录
    assetsPublicPath: '/', // 编译发布的根目录，可配置为资源服务器域名或 CDN 域名
    productionSourceMap: true, // 是否开启 cssSourceMap
    // Gzip off by default as many popular static hosts such as
    // Surge or Netlify already gzip all static assets for you.
    // Before setting to `true`, make sure to:
    // npm install --save-dev compression-webpack-plugin
    productionGzip: false, // 是否开启 gzip
    productionGzipExtensions: ['js', 'css'], // 需要使用 gzip 压缩的文件扩展名
    // Run the build command with an extra argument to
    // View the bundle analyzer report after build finishes:
    // `npm run build --report`
    // Set to `true` or `false` to always turn it on or off
    bundleAnalyzerReport: process.env.npm_config_report
  },
  dev: { // dev 环境
    env: require('./dev.env'), // 使用 config/dev.env.js 中定义的编译环境
    port: 8084, // 运行测试页面的端口
    autoOpenBrowser: true, // 自动在浏览器中打开
    assetsSubDirectory: 'static', // 编译输出的二级目录
    assetsPublicPath: '/', // 编译发布的根目录，可配置为资源服务器域名或 CDN 域名
    proxyTable: {}, // 需要 proxyTable 代理的接口（可跨域）
    // CSS Sourcemaps off by default because relative paths are "buggy"
    // with this option, according to the CSS-Loader README
    // (https://github.com/webpack/css-loader#sourcemaps)
    // In our experience, they generally work as expected,
    // just be aware of this issue when enabling this option.
    cssSourceMap: false  // 是否开启 cssSourceMap
  }
}
```

曾经不易理解的两点`assetsSubDirectory`和`assetsPublicPath`

1. assetsSubDirectory
   被 webpack 编译处理过的资源文件都会在这个 build.assetsRoot 目录下，如果 assetsRoot 值是`"/web/app"`，assetsSubDirectory 值为`"static"`，那么，webpack 将把所有资源编译到`web/app/static`目录下

2. assetsPublicPath
   这个是通过 http 服务器运行的 url 路径，大多数情况下，这个是根目录（/）。如果你的后台框架对静态资源 url 前缀有要求，你仅需改变这个参数。比如不用本地的，而用线上的 CDN。

#### 父子组件嵌套，各个钩子函数触发顺序

偶然看到这个问题：vue 中父子组件各个钩子函数触发顺序是怎样的？一时还真背问到了，在项目中添加打印才发现是这样子的

![](https://note.youdao.com/yws/public/resource/bb7792e904a30442f11cb6c88c33cce8/xmlnote/E40E6B0ED1F5479F909A69469C83D99A/12039)

顺序是：先依次触发父级组件`beforeCreate`、`create`和`beforeMounte`，再依次触发子级组件`beforeCreate`、`create`、`beforeMounte`和`mounted`，最后父级组件`mounted`

#### 父子组件之间通信，兄弟组件之间通信

这个问题基本清晰，在这归纳一下

##### 1. 父组件数据传给子组件

通过`props`属性传递

```html
<!--父组件-->
<parent-component :parent-data="pdata"></parent-component>
```

```javascript
// 子组件
export default {
  props: {
    parentData: {
      type: String,
      default: ''
    }
  }
}
```

##### 2. 子组件传数据给父组件

使用`$emit`派发

```html
<!--父组件-->
<parent-component :parent-data:sync="pdata" @handle-callback="handlerCallback"></parent-component>
```

```javascript
// 父组件
export default {
  methods: {
    handlerCallback(params) {
      // do something
    }
  }
}
```

```javascript
// 子组件
export default {
  created() {
    // ...
    this.$emit('handleCallback', params)

    // ...
    this.$emit('update:parentData', someData)
  }
}
```

##### 3. 兄弟组件数据传递

1. 对于大型项目，用 vue 官方推荐的 vuex
2. EventBus

   - 提取 bus.js
     ```javascript
     import Vue from 'vue'
     const bus = new Vue()
     export default bus
     ```
   - 兄弟组件 1 发送数据

     ```javascript
     import bus from './bus'

     export default {
       created() {
         // ...
         this.$emit('busEvent1', someData)
       }
     }
     ```

   - 兄弟组件 2 接收数据

     ```javascript
     import bus from './bus'

     export default {
       created() {
         // ...
         this.$on('busEvent1', function(data) {
           console.log(data)
         })
       }
     }
     ```

3. 子组件 A `$emit`派发某个事件，再由父组件`@handle-callback="handlerCallback"`监听获取数据，然后，父组件`$refs`直接访问到子组件 B 的方法，从而间接实现从子组件 A 到子组件 B 的数据传递

#### props 在子组件中被重写报错

```javascript
vue.esm.js?06e7:591 [Vue warn]: Avoid mutating a prop directly since the value will be overwritten whenever the parent component re-renders. Instead, use a data or computed property based on the prop's value. Prop being mutated: "isShow"
```

**解决方案**
`props`传过来的参数通过`Vue.$emit`提交修改

#### 在`props`和`data`中使用`this`

在 Vue2.2.2 或更高版本才能这样使用，低于这个版本时，注入的值会在`props`和`data`初始化之后得到。

#### 对于高阶插件/组件库，解决组件与其子孙组件数据传输问题

1. 解决方案一：`$attrs`和`$listener`

   使用属性和方法不通过`props`传递，在子孙组件中直接用`$attrs`和`$listeners`接收。不过支持 v2.4.0+。

   ```javascript
   // 父级组件
   <ul id="app6">
   	<item
   		class="item"
   		:model="treeData"
   		:count="123"
   		@abc="function(){}"
   	></item>
   </ul>

   // 子孙组件
   inheritAttrs: false,
   created () {
       let attrs = this.$attrs;
       console.log('mode:', attrs.mode);
       console.log('count:', attrs.count);

       let listeners = this.$listeners;
       console.log('bac:', listeners.abc);
   }
   ```

2. 解决方案二：`provide/inject`

   父级组件传入`provide`数据选项，子孙组件注入`inject`数据。

   ```javascript
   // 父组件
   provide: {
     foo: 'bar'
   }

   // 子孙组件
   inject: ['foo'] // or inject: { name: 'foo', defult: '' }
   ```
