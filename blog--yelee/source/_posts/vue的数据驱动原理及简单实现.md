---
title: vue的数据驱动原理及简单实现
date: 2017-09-10 09:00:30
tags: vue
categories: vue
---
  ## Vue双向数据绑定原理
### 1、目标实现
1. 理解双向数据绑定原理；
2. 实现 &#123;&#123; &#125;&#125;、v-model和基本事件指令v-bind（:）、v-on（@）；
3. 新增属性的双向绑定处理；
<!-- more -->

### 2、双向数据绑定原理

vue实现对数据的双向绑定，通过对数据劫持结合发布者-订阅者模式实现的。

#### 2.1 Object.defineProperty

vue通过Object.defineProperty来实现数据劫持，会对数据对象每个属性添加对应的get和set方法，对数据进行读取和赋值操作就分别调用get和set方法。

```javascript
Object.defineProperty(data, key, {
    enumerable: true,
    configurable: true,
    get: function() {

        // do something

        return val;
    },
    set: function(newVal) {

        // do something
    }
});
```

我们可以将一些方法放到里面，从而完成对数据的监听（劫持）和视图的同步更新。

![image](https://note.youdao.com/yws/public/resource/b9cdada69234d36736d09235b516171c/xmlnote/F8D35A0DDA0C438990651233F3954DD3/6460)

#### 2.2 过程说明

实现双向数据绑定，首先要对数据进行数据监听，需要一个监听器Observer，监听所有属性。如果属性发生变化，会调用setter和getter，再去告诉订阅者Watcher是否需要更新。由于订阅者有很多个，我们需要一个消息订阅器Dep来专门收集这些订阅者，然后在监听器Observer和订阅者Watcher之间进行统一管理。还有，我们需要一个指令解析器Complie，对每个元素进行扫描和解析，将相关指令对应初始化成一个订阅者Watcher，并替换模板数据或绑定相应函数。当订阅者Watcher接收到相应属性的变化，就会执行对应的更新函数，从而更新视图。

![image](https://note.youdao.com/yws/public/resource/b9cdada69234d36736d09235b516171c/xmlnote/A6333A71A8E243A38EAD196BB13F9EE7/6463)

### 3、实现Observer

Observer是一个数据监听器，核心方法是我们提到过的Object.defineProperty。如果要监听所有属性的话，则需要通过递归遍历，对每个子属性都defineProperty。

```javascript
/**
 * 监听器构造函数
 * @param {Object} data 被监听数据
 */
function Observer(data) {

    if (!data || typeof data !== "object") {
        return;
    }

    this.data = data;
    this.walk(data);

}

Observer.prototype = {
    /**
     * 属性遍历
     */
    walk: function(data) {
        var self = this;
        Object.keys(data).forEach(function(key) {
            self.defineReactive(data, key, data[key]);
        });
    },

    /**
     * 监听函数
     */
    defineReactive: function(data, key, val) {

        observe(val);

        Object.defineProperty(data, key, {
            enumerable: true,
            configurable: true,
            get: function() {
                return val;
            },
            set: function(newVal) {
                if (newVal === val) {
                    return;
                }

                val = newVal;

                console.log("属性：" + key + "被监听了，现在值为：" + newVal);

                updateView(newVal);
            }
        });

        updateView(val);
    }
}

/**
 * 监听器
 * @param {Object} data 被监听对象
 */
function observe(data) {

    return new Observer(data);
}

/**
 * vue构造函数
 * @param {Object} options 所有入参
 */
function MyVue(options) {

    this.vm = this;

    this.data = options.data;

    // 监听数据
    observe(this.data);

    return this;
}

/**
 * 更新视图
 * @param {*} val
 */
function updateView(val) {
    var $name = document.querySelector("#name");
    $name.innerHTML = val;
}

var myvm = new MyVue({
    el: "#demo",
    data: {
        name: "hello word"
    }
});
```

![image](https://note.youdao.com/yws/public/resource/b9cdada69234d36736d09235b516171c/xmlnote/93FC8BA149764033996B07C9031FBE2D/6483)

### 4、实现Dep

在流程介绍中，我们需要创建一个可以订阅者的订阅器Dep，主要负责手机订阅者，属性变化的时候执行相应的订阅者，更新函数。下面稍加改造Observer，就可以插入我们的订阅器。

```javascript
Observer.prototype = {
    // ...

    /**
     * 监听函数
     */
    defineReactive: function(data, key, val) {
        var dep = new Dep();

        observe(val);

        Object.defineProperty(data, key, {
            enumerable: true,
            configurable: true,
            get: function() {

                // 判断是否需要添加订阅者 什么时候添加订阅者呢？ 与实际页面DOM有关联的data属性才添加相应的订阅者
                if (Dep.target) {
                    // 添加一个订阅者
                    dep.addSub(Dep.target);
                }

                return val;
            },
            set: function(newVal) {
                if (newVal === val) {
                    return;
                }

                val = newVal;

                console.log("属性：" + key + "被监听了，现在值为：" + newVal);

                // 通知所有订阅者
                dep.notify(newVal);
            }
        });

        updateView(val);

        // 订阅器标识本身实例
        Dep.target = dep;
        // 强行执行getter，往订阅器中添加订阅者
        var v = data[key];
        // 释放自己
        Dep.target = null;
    }
}

/**
 * 监听器
 * @param {Object} data 被监听对象
 */
function observe(data) {

    return new Observer(data);
}

/**
 * 订阅器
 */
function Dep() {
    this.subs = [];
    this.target = null;
}

Dep.prototype = {
    addSub: function(sub) {
        this.subs.push(sub);
        console.log("this.subs:", this.subs);
    },
    notify: function(data) {
        this.subs.forEach(function(sub) {
            sub.update(data);
        });
    },
    update: function(val) {
        updateView(val)
    }
};

// ...

```
PS:将订阅器Dep添加到一个订阅者设计到getter里面，是为了让Watcher初始化进行触发。

### 5、实现Watcher
订阅者Watcher在初始化的时候需要将自己添加到订阅器Dep中，那该如何添加呢？我们已经知道监听器Observer是在get函数执行添加了订阅者Watcher的操作，所以我们只要在订阅者Watcher初始化的时候触发对应的get函数去执行添加订阅者操作。那么，怎样去触发get函数？很简单，只需获取对应的属性值就可以触发了，因为我们已经用Object.defineProperty监听了所有属性。vue在这里做了个技巧处理，就是咋我们添加订阅者的时候，做一个判断，判断是否是事先缓存好的Dep.target，在订阅者添加成功后，把target重置null即可。

```javascript
// ...

/**
 * 订阅者
 * @param {Object} vm vue对象
 * @param {String} exp 属性值
 * @param {Function} cb 回调函数
 */
function Watcher(vm, exp, cb) {
    this.vm = vm;
    this.exp = exp;
    this.cb = cb;
    // 将自己添加到订阅器
    this.value = this.get();
}

Watcher.prototype = {
    update: function() {
        this.run();
    },
    run: function() {
        var value = this.vm.data[this.exp];
        var oldVal = this.value;

        if (value !== oldVal) {
            this.value = value;
            this.cb.call(this.vm, value, oldVal);
        }
    },
    get: function() {
        // 缓存自己 做个标记
        Dep.target = this;

        // 强制执行监听器里的get函数
        // this.vm.data[this.exp] 调用getter，添加一个订阅者sub，存入到全局变量subs
        var value = this.vm.data[this.exp];

        // 释放自己
        Dep.target = null;

        return value;
    }
};

/**
 * vue构造函数
 * @param {Object} options 所有入参
 */
function MyVue(options) {

    this.vm = this;

    this.data = options.data;

    observe(this.data);

    var $name = document.querySelector("#name");

    // 给name属性添加一个订阅者到订阅器中，当属性发生变化后，触发回调
    var w = new Watcher(this, "name", function(val) {
        $name.innerHTML = val;
    });

    return this;
}
```

到这里，其实已经实现了我们的双向数据绑定：能够根据初始数据初始化页面特定元素，同时当数据改变也能更新视图。

### 5、实现Compile

步骤4整个过程都能有去解析DOM节点，而是直接固定节点进行替换。接下来我们就来实现一个解析器，完成一些解析和绑定工作。

1. 获取页面的DOM节点，遍历存入到文档碎片对象中；
2. 解析出文本节点，匹配&#123;&#123;&#125;&#125;（暂时只做"&#123;&#123;&#125;&#125;"的解析），用初始化数据替换，并添加相应订阅者；
3. 分离出节点的指令v-on、v-bind和v-model，绑定相应的事件和函数；


![image](https://note.youdao.com/yws/public/resource/b9cdada69234d36736d09235b516171c/xmlnote/BD96748FED95467A8B44B1AFFB499D44/6764)

```javascript
// ...

/**
 * 编译器构造函数
 * @param {String} el 根元素
 * @param {Object} vm vue对象
 */
function Compile(el, vm) {
    this.vm = vm;
    this.el = document.querySelector(el);
    this.fragment = null;
    this.init();
}

Compile.prototype = {
    /**
     * 初始
     */
    init: function() {
        if (this.el) {
            console.log("this.el:", this.el);
            // 移除页面元素生成文档碎片
            this.fragment = this.nodeToFragment(this.el);
            // 编译文档碎片
            this.compileElement(this.fragment);
            this.el.appendChild(this.fragment);
        } else {
            console.log("DOM Selector is not exist");
        }
    },

    /**
     * 页面DOM节点转化成文档碎片
     */
    nodeToFragment: function(el) {
        var fragment = document.createDocumentFragment();
        var child = el.firstChild;

        // 此处添加打印，出来的不是页面中原始的DOM，而是编译后的？
        // NodeList是引用关系，在编译后相应的值被替换了，这里打印出来的NodeList是后来被引用更新了的
        console.log("el:", el);
        // console.log("el.firstChild:", el.firstChild.nodeValue);
        while (child) {
            // append后，原el上的子节点被删除了，挂载在文档碎片上
            fragment.appendChild(child);
            child = el.firstChild;
        }

        return fragment;
    },
    /**
     * 编译文档碎片，遍历到当前是文本节点则去编译文本节点，如果当前是元素节点，并且存在子节点，则继续递归遍历
     */
    compileElement: function(fragment) {
        var childNodes = fragment.childNodes;
        var self = this;
        [].slice.call(childNodes).forEach(function(node) {
            // var reg = /\{\{\s*(.+)\s*\}\}/g;
            var reg = /\{\{\s*((?:.|\n)+?)\s*\}\}/g;
            var text = node.textContent;

            if (self.isElementNode(node)) {
                self.compileAttr(node);
            } else if (self.isTextNode(node) && reg.test(text)) {
                reg.lastIndex = 0

                /* var match;
                while(match = reg.exec(text)) {
                    self.compileText(node, match[1]);
                } */

                self.compileText(node, reg.exec(text)[1]);
            }

            if (node.childNodes && node.childNodes.length) {
                self.compileElement(node);
            }
        });
    },

    /**
     * 编译属性
     */
    compileAttr: function(node) {
        var nodeAttrs = node.attributes;
        var self = this;

        Array.prototype.forEach.call(nodeAttrs, function(attr) {
            var attrName = attr.name;

            // 只对vue本身指令进行操作
            if (self.isDirective(attrName)) {
                var exp = attr.value;

                // v-on指令
                if (self.isOnDirective(attrName)) {
                    self.compileOn(node, self.vm, exp, attrName);
                }
                // v-bind指令
                if(self.isBindDirective(attrName)) {
                    self.compileBind(node, self.vm, exp, attrName);
                }
                // v-model
                else if (self.isModelDirective(attrName)) {
                    self.compileModel(node, self.vm, exp, attrName);
                }

                node.removeAttribute(attrName);
            }
        })
    },

    /**
     * 编译文档碎片节点文本，即对标记替换
     */
    compileText: function(node, exp) {
        var self = this;
        var exps = exp.split(".");
        var initText = this.vm.data[exp];

        // 初始化视图
        this.updateText(node, initText);

        // 添加一个订阅者到订阅器
        var w = new Watcher(this.vm, exp, function(val) {
            self.updateText(node, val);
        });
    },

    /**
     * 编译v-on指令
     */
    compileOn: function(node, vm, exp, attrName) {
        // @xxx v-on:xxx
        var onRE = /^@|^v-on:/;
        var eventType = attrName.replace(onRE, "");

        var cb = vm.methods[exp];

        if (eventType && cb) {
            node.addEventListener(eventType, cb.bind(vm), false);
        }
    },

    /**
     * 编译v-bind指令
     */
    compileBind: function (node, vm, exp, attrName) {
        // :xxx v-bind:xxx
        var bindRE = /^:|^v-bind:/;
        var attr = attrName.replace(bindRE, "");

        var val = vm.data[exp];

        node.setAttribute(attr, val);
    },

    /**
     * 编译v-model指令
     */
    compileModel: function(node, vm, exp, attrName) {
        var self = this;
        var val = this.vm.data[exp];

        // 初始化视图
        this.modelUpdater(node, val);

        // 添加一个订阅者到订阅器
        new Watcher(this.vm, exp, function(value) {
            self.modelUpdater(node, value);
        });

        // 绑定input事件
        node.addEventListener("input", function(e) {
            var newVal = e.target.value;
            if (val === newVal) {
                return;
            }
            self.vm.data[exp] = newVal;
            // val = newVal;
        });
    },

    /**
     * 更新文档碎片相应的文本节点
     */
    updateText: function(node, val) {
        node.textContent = typeof val === "undefined" ? "" : val;
    },

    /**
     * model更新节点
     */
    modelUpdater: function(node, val, oldVal) {
        node.value = typeof val == "undefined" ? "" : val;
    },

    /**
     * 属性是否是vue指令，包括v-xxx:,:xxx,@xxx
     */
    isDirective: function(attrName) {
        var dirRE = /^v-|^@|^:/;
        return dirRE.test(attrName);
    },

    /**
     * 属性是否是v-on指令
     */
    isOnDirective: function(attrName) {
        var onRE = /^v-on:|^@/;
        return onRE.test(attrName);
    },

    /**
     * 属性是否是v-bind指令
     */
    isBindDirective: function (attrName) {
        var bindRE = /^v-bind:|^:/;
        return bindRE.test(attrName);
    },

    /**
     * 属性是否是v-model指令
     */
    isModelDirective: function(attrName) {
        var mdRE = /^v-model/;
        return mdRE.test(attrName);
    },

    /**
     * 判断元素节点
     */
    isElementNode: function(node) {
        return node.nodeType == 1;
    },

    /**
     * 判断文本节点
     */
    isTextNode: function(node) {
        return node.nodeType == 3;
    }
};

/**
 * vue构造函数
 * @param {Object} options 所有入参
 */
function MyVue(options) {

    this.vm = this;

    this.data = options.data;

    this.methods = options.methods;

    observe(this.data);

    new Compile(options.el, this.vm);

    return this;
}

```

这样我们就可以调用指令v-bind、v-on和v-model。


```html
<head>
    <meta charset="UTF-8">
    <style>
        .red {
            color: red;
        }
    </style>
</head>

<body>
    <div id="demo">
        <h2 v-bind:class="myColor">&#123;&#123; name &#125;&#125;</h2>
        <input type="text" name="" v-model="name">
        <button @click="clickOk">Ok</button>
    </div>
</body>

<script>
var myvm = new MyVue({
    el: "#demo",
    data: {
        name: "hello word",
        myColor: "red"
    },
    methods: {
        clickOk: function() {
            alert("I am OK");
        }
    }
});

setTimeout(function() {
    myvm.data.name = "wawawa...vue was born";
}, 2000);
</script>
```
![image](https://note.youdao.com/yws/public/resource/b9cdada69234d36736d09235b516171c/xmlnote/0442953EE78A48C3B26FFE48B1731582/6629)

### 5、其他

#### 5.1 proxy代理data

可能注意到了，我们不管是在赋值还是取值，都是在myvm.data.someAttr上操作的，而在vue上我们习惯直接myvm.someAttr这种形式。怎样实现呢？同样，我们可以用Object.defineProperty对data所有属性做一个代理，即访问vue实例属性时，代理到data上。很简单，实现如下：

```
/**
 * 将数据拓展到vue的根，方便读取和设置
 */
MyVue.prototype.proxy = function(key) {
    var self = this;

    Object.defineProperty(this, key, {
        enumerable: true,
        configurable: true,
        get: function proxyGetter() {
            return self.data[key];
        },
        set: function proxySetter(newVal) {
            self.data[key] = newVal;
        }
    });
}
```

![image](https://note.youdao.com/yws/public/resource/b9cdada69234d36736d09235b516171c/xmlnote/71B50262A5944F928CDA8CFD8944A538/6670)

#### 5.2 parsePath
上面对于data的操作只是到对于简单的基本类型属性，对于对象属性的改变该怎么更新到位呢？其实，只要深度遍历对象属性路径，就可以找到要访问属性值。

```javascript
/**
 * 根据对象属性路径，最终获取值
 * @param {Object} obj 对象
 * @param {String} path 路径
 * return 值
 */
function parsePath(obj, path) {
    var bailRE = /[^\w.$]/;
    if (bailRE.test(path)) {
        return
    }
    var segments = path.split('.');

    for (var i = 0; i < segments.length; i++) {
        if (!obj) { return }
        obj = obj[segments[i]];
    }
    return obj;
}
```
用这个方法替换我们的所有取值操作
vm[exp] => parsePath(vm, exp)

![image](https://note.youdao.com/yws/public/resource/b9cdada69234d36736d09235b516171c/xmlnote/BB5F59CFD0244D4991E13B5D35F88AF3/6709)

### 6、新增属性的双向数据绑定

#### 6.1 给对象添加属性

Vue 不允许在已经创建的实例上动态添加新的根级响应式属性 (root-level reactive property)。然而它可以使用 Vue.set(object, key, value) 方法将响应属性添加到嵌套的对象上。
也就是我们需要在Vue原型上添加一个set方法去设置新添加的属性，新属性同样要进行监听和添加订阅者。


```javascript
/**
 * vue的set方法，用于外部新增属性 Vue.$set(target, key, val)
 * @param {Object} target 数据
 * @param {String} key 属性
 * @param {*} val 值
 */
function set(target, key, val) {
    if (Array.isArray(target)) {
        target.length = Math.max(target.length, key);
        target.splice(key, 1, val);
        return val;
    }

    if (target.hasOwnProperty(key)) {
        target[key] = val;
        return val
    }
    var ob = (target).$Observer;

    if (!ob) {
        target[key] = val;
        return val
    }

    // 对新增属性定义监听
    ob.defineReactive(target, key, val);

    ob.dep.notify();

    return val;
}

MyVue.prototype.$set = set;
```

#### 6.1 给数组对象添加属性

把数组看成一个特殊的对象，就很容易理解了，对于unshift、push和splice变异方法是添加了对象的属性的，需要对新加的属性进行监听和添加订阅者。

```javascript
var arrKeys = ["push", "pop", "shift", "unshift", "splice", "sort", "reverse"];
var extendArr = [];

arrKeys.forEach(function(key) {
    def(extendArr, key, function() {
        var result,
            arrProto = Array.prototype,
            ob = this.$Observer,
            arr = arrProto.slice.call(arguments),
            inserted,
            index;

        switch (key) {
            case "push":
                inserted = arr;
                index = this.length;
                break;
            case "unshift":
                inserted = arr;
                index = 0;
                break;
            case "splice":
                inserted = arr.slice(2);
                index = arr[0];
                break;
        }

        result = arrProto[key].apply(this, arguments);

        // 监听新增数组对象属性
        if (inserted) {
            ob.observeArray(inserted);
        }

        ob.dep.notify();

        return result;
    });
});

var arrayKeys = Object.getOwnPropertyNames(extendArr);

/**
 * 监听器构造函数
 * @param {Object} data 被监听数据
 */
function Observer(data) {

    this.dep = new Dep();

    if (!data || typeof data !== "object") {
        return;
    }

    // 在每个object上添加一个observer
    def(data, "$Observer", this);

    // 继承变异方法
    if (Array.isArray(data)) {

        // 把数组变异方法的处理，添加到原型链上
        data.__proto__ = extendArr;

        // 监听数组对象属性
        this.observeArray(data);
    } else {
        this.data = data;
        this.walk(data);
    }
}

Observer.prototype = {
    // ...

    /**
     * 监听数组
     */
    observeArray: function(items) {
        console.log("items:", items);
        for (var i = 0, l = items.length; i < l; i++) {
            observe(items[i]);
        }
    }
};
```
