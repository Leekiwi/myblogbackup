---
title: 搞定css三列布局
date: 2018-07-22 19:32:30
tags: [css, 布局]
categories: css
---

> 谈到布局，首先就要想到定位，当别人问我，css 的 position 定位有哪些取值，分别表示什么意思？呃.....

### 定位

position 有六个属性值：static、relative、absolute、fixed、sticky 和 inherit。

<!-- more -->

- static（默认）：元素框正常生成。块级元素生成一个矩形框，作为文档流的一部分；行内元素则会创建一个或多个行框，置于父级元素中。
- relative：元素框相对于之前正常文档流中的位置发生偏移，并且原先的位置仍然被占据。发生偏移的时候，可能会覆盖其他元素。
- absolute：元素框不再占有文档位置，并且相对于包含块进行偏移（所谓包含块就是最近一级外层元素 position 不为 static 的元素）。
- fixed：元素框不再占有文档流位置，并且相对于视窗进行定位。
- sticky：css3 新增属性值，粘性定位，相当于 relative 和 fixed 的混合。最初会被当作是 relative，相对原来位置进行偏移；一旦超过一定的阈值，会被当成 fixed 定位，相对于视口定位。

### 三列布局

三列布局，其中一列宽度自适应，在 PC 端最常用之一，搞定了三列布局，其他一样的套路。

##### 方式一：浮动布局

缺点：html 结构不正确,当包含区域宽度小于左右框之和，右边框会被挤下来

```
<style>
    .tree-columns-layout.float .left {
        float: left;
        width: 300px;
        background-color: #a00;
    }

    .tree-columns-layout.float .right {
        float: right;
        width: 300px;
        background-color: #0aa;
    }

    .tree-columns-layout.float .center {
        /* left: 300px;
        right: 300px; */
        margin: 0 300px;
        background-color: #aa0;
        overflow: auto;
    }
</style>
<section class="tree-columns-layout float">
    <article class="left">
        <h1>我是浮动布局左框</h1>
    </article>
    <article class="right">
        <h1>我是浮动布局右框</h1>
    </article>
    <article class="center">
        <h1>我是浮动布局中间框</h1>
    </article>
</section>
```

#### 方式二：定位布局

缺点：要求父级要有非 static 定位，如果没有，左右框容易偏移出去

```
<style>
    .tree-columns-layout.position {
        position: relative;
    }

    .tree-columns-layout.position .left {
        position: absolute;
        left: 0;
        top: 0;
        width: 300px;
        background-color: #a00;
    }

    .tree-columns-layout.position .right {
        position: absolute;
        right: 0;
        top: 0;
        width: 300px;
        background-color: #0aa;
    }

    .tree-columns-layout.position .center {
        margin: 0 300px;
        background-color: #aa0;
        overflow: auto;
    }
</style>
<section class="tree-columns-layout position">
    <article class="left">
        <h1>我是浮动定位左框</h1>
    </article>
    <article class="center">
        <h1>我是浮动定位中间框</h1>
    </article>
    <article class="right">
        <h1>我是浮动定位右框</h1>
    </article>
</section>
```

#### 方式三：表格布局

缺点：没什么缺点，恐惧 table

```
<style>
    .tree-columns-layout.table {
        display: table;
        width: 100%;
    }

    .tree-columns-layout.table > article {
        display: table-cell;
    }

    .tree-columns-layout.table .left {
        width: 300px;
        background-color: #a00;
    }

    .tree-columns-layout.table .center {
        background-color: #aa0;
    }

    .tree-columns-layout.table .right {
        width: 300px;
        background-color: #0aa;
    }

</style>
<section class="tree-columns-layout table">
    <article class="left">
        <h1>我是表格布局左框</h1>
    </article>
    <article class="center">
        <h1>我是表格布局中间框</h1>
    </article>
    <article class="right">
        <h1>我是表格布局右框</h1>
    </article>
</section>
```

#### 方式四：flex 弹性布局

缺点：兼容性

```
<style>
    .tree-columns-layout.flex {
        display: flex;
    }

    .tree-columns-layout.flex .left {
        width: 300px;
        flex-shrink: 0; /* 不缩小 */
        background-color: #a00;
    }

    .tree-columns-layout.flex .center {
        flex-grow: 1; /* 增大 */
        background-color: #aa0;
    }

    .tree-columns-layout.flex .right {
        flex-shrink: 0; /* 不缩小 */
        width: 300px;
        background-color: #0aa;
    }
</style>
<section class="tree-columns-layout flex">
    <article class="left">
        <h1>我是flex弹性布局左框</h1>
    </article>
    <article class="center">
        <h1>我是flex弹性布局中间框</h1>
    </article>
    <article class="right">
        <h1>我是flex弹性布局右框</h1>
    </article>
</section>
```

#### 方式五：grid 栅格布局

缺点：兼容性 Firefox 52, Safari 10.1, Chrome 57, Opera 44

```
<style>
    .tree-columns-layout.grid {
        display: grid;
        grid-template-columns: 300px 1fr 300px;
    }

    .tree-columns-layout.grid .left {
        background-color: #a00;
    }

    .tree-columns-layout.grid .center {
        background-color: #aa0;
    }

    .tree-columns-layout.grid .right {
        background-color: #0aa;
    }
</style>
<section class="tree-columns-layout grid">
    <article class="left">
        <h1>我是grid栅格布局左框</h1>
    </article>
    <article class="center">
        <h1>我是grid栅格布局中间框</h1>
    </article>
    <article class="right">
        <h1>我是grid栅格布局右框</h1>
    </article>
</section>
```

#### 方式六：圣杯布局

缺点：需要多加一层标签，html 顺序不对，占用了布局框的 margin 属性

```
<style>
    .tree-columns-layout.cup:after {
        clear: both;
        content: "";
        display: table;
    }

    .tree-columns-layout.cup .center {
        width: 100%;
        float: left;
    }

    .tree-columns-layout.cup .center > div {
        margin: 0 300px;
        overflow: auto;
        background-color: #aa0;
    }

    .tree-columns-layout.cup .left {
        width: 300px;
        float: left;
        margin-left: -100%;
        background-color: #a00;
    }

    .tree-columns-layout.cup .right {
        width: 300px;
        float: left;
        margin-left: -300px;
        background-color: #0aa;
    }
</style>
<section class="tree-columns-layout cup">
    <article class="center">
        <div>
            <h1>我是圣杯布局中间框</h1>
        </div>
    </article>
    <article class="left">
        <h1>我是圣杯布局左框</h1>
    </article>
    <article class="right">
        <h1>我是圣杯布局右框</h1>
    </article>
</section>
```
