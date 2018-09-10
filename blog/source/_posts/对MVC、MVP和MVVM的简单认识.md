---
title: 对MVC、MVP和MVVM的简单认识
date: 2018-08-27 19:00:30
tags: 设计模式
categories: 其它
---

### 1. 先聊一下 MVC

MVC：Model（模型） View（视图） Controller（控制器），主要是基于分层的目的，让彼此的职责分开。

View 通过 Controller 和 Model 联系，Controller 是 View 和 Model 的协调者，view 和 Model 不直接联系，基本联系都是单向的。

<!-- more -->

### 2. 顺带提下 MVP

MVP：是从 MVC 模式演变而来的，都是通过 Controller/Presenter 负责逻辑的处理+Model 提供数据+View 负责显示。

在 MVP 中，Presenter 完全把 View 和 Model 进行分离，主要的程序逻辑在 Presenter 里实现。并且，Presenter 和 View 是没有直接关联的，是通过定义好的接口进行交互，从而使得在变更 View 的时候可以保持 Presenter 不变。

### 3. 再聊聊 MVVN

MVVM：Model + View + ViewModel，把 MVC 中的 Controller 和 MVP 中的 Presenter 改成 ViewModel

view 的变化会自动更新到 ViewModel，ViewModel 的变化也会自动同步到 View 上显示。这种自动同步是因为 ViewModel 中的属性实现了 Observer，当属性变更时都能触发对应操作。

- View 是 HTML 文本的 js 模板；
- ViewModel 是业务逻辑层（一切 js 可视业务逻辑，比如表单按钮提交，自定义事件的注册和处理逻辑都在 viewmodel 里面负责监控俩边的数据）；
- Model 数据层，对数据的处理（与后台数据交互的增删改查）

提一下我熟悉的 MVVM 框架：vue.js，Vue.js 是一个构建数据驱动的 web 界面的渐进式框架。Vue.js 的目标是通过尽可能简单的 API 实现响应的数据绑定和组合的视图组件。核心是一个响应的数据绑定系统。

### 4. 一句话总结下不同之处

MVC 中联系是单向的，MVP 中 P 和 V 通过接口交互，MVVP 的联系是双向的
