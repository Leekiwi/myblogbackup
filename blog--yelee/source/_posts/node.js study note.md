---
title: nodejs学习笔记
date: 2018-08-28 20:42:30
tags: nodejs
categories: nodejs
---

## Nodejs 学习笔记

**node 内部对模块输出 module.exports 的实现**

变量 module 是 Node 在加载 js 文件前准备的一个变量，并将其传入加载函数

<!-- more -->

```
// 准备module对象
var module = {
    id: 'hello',
    exports: {}
};

var load = function(module) {
    // 读取的hello.js代码
    function greet(name) {
        console.log('Hello, ' + name + '!');
    }

    module.exports = greet;

    // hello.js代码结束
    return module.exports;
};

var exported = load(module);

// 保存module
save(module, exported);
```

默认情况下，Node 准备的 exports 变量和 module.exports 变量实际上是同一个变量，所以一下两种写法都支持

```
// method 1
module.exports = {
    foo: foo,
    bar: bar
};
or
module.exports.foo = foo;
module.exports.bar = bar;

// method 2
exports.foo = foo;
exports.bar = bar;
```

**process**

下一轮事件循环 回调

```
process.nextTick(function() {
    console.log('nextTick callback');
});
```

程序即将退出 回调

```
process.on('exit', function(code) {
    console.log('about to exit with code' + code);
});

console.log('nextTick set');
```

**readFile/readFileSync writeFile/writeFileSync stat**

异步读取一个文本文件

```
fs.readFile('./hello.js', 'utf-8', function(err, data) {
    console.log('read file start...');
    if(err) {
        console.log(err);
    }
    else {
        console.log(data);
    }
});
```

异步读取一个二进制文件

```
fs.readFile('1.jpg', function(err, data) {
    if(err) {
        console.log(err);
    }
    else {
        // 返回一个buffer对象
        console.log(data);
        // Buffer对象转化成字符串
        console.log(data.toString('utf-8'));
        // 文件大小
        console.log(data.length + ' bytes');
    }
});
```

同步读取一个文件直接返回，读取错误用 try...catch 捕获

```
try {
    var data = fs.readFileSync('./1.jpg');
    console.log(data);
}
catch(err) {
    console.log(err);
}

console.log('readFileSync ended');
```

异步写入一个文件，默认是以 UTF-8 编码写入文本文件

```
var data = 'Hello,Node.js';
// var data = fs.readFileSync('./1.jpg');
fs.writeFile('output.txt', data, function(err) {
    if(err) {
        console.log(err);
    }
    else {
        console.log('write file finished');
    }
});
```

同步写入文本到一个文件

```
var data = 'Hello,Node.js,I am sync data';
fs.writeFileSync('output.txt', data);
console.log('writeFileSync ended');
```

获取文件信息

```
fs.stat('./1.jpg', function(err, stat) {
    if(err) {
        console.log(err);
    }
    else {
        // 是否是文件
        console.log('isFile:' + stat.isFile());
        // 是否是目录
        console.log('isDirectory:' + stat.isDirectory());

        if(stat.isFile()) {
            // 文件大小
            console.log('size:' + stat.size);
            // 创建时间，Date对象
            console.log('birth time:' + stat.birthtime);
            // 修改时间，Date对象
            console.log('modified time:' + stat.mtime);
        }
    }
});
```

PS:绝大部分需要在服务器运行期反复执行业务逻辑，必须使用异步代码服务器启动时如果需要读取配置文件，或者结束时需要写入到状态文件时，可以使用同步代码

**createReadStream createWriteStream pipe**

在 node.js 中，流也是一个对象，我们只需要响应流的事件就可以了。
data 事件表示流的数据已经可以读取了，end 事件表示这个流已经到末尾了，没有数据可以读取了，error 事件表示出错。

```
var rs = fs.createReadStream('./data.txt', 'utf-8');

// data事件可能有多次，每次传递的chunk是流的一部分数据
rs.on('data', function(chunk) {
    console.log('data event:', chunk);
});

rs.on('end', function (chunk) {
    console.log('end event:');
});

rs.on('error', function (chunk) {
    console.log('error event:');
});
```

以流的形式写入文件，只需要不断调用 write()方法，最后以 end()结束

```
var ws = fs.createWriteStream('./data.txt');
ws.write('user stream write data\n');
ws.write('loading...\n');
ws.write('END');
ws.end();
```

pipe()把一个文件流和另一个文件流串起来，这样源文件的所有数据就自动写入到目标文件里

```
var rs = fs.createReadStream('./data.txt');
var ors = fs.createReadStream('./output.txt');
var ws = fs.createWriteStream('./output.txt');
rs.pipe(ws);
```

创建一个服务器

```
var server = http.createServer(function(request, response) {
    // http请求头的method和url
    console.log('header meathod:', request.method);
    console.log('header url:', request.url);

    // 将http响应200写入response，同时设置content-type
    response.writeHead(200, {'Content-Type': 'text/html'});

    // 将http响应的html内容写入response
    response.end('<h1>Hello world!</h1>');
});

server.listen(8080);

console.log('Server is running at http://localhost:8080');
```

实现一个文件服务器，拼接访问路径读取本地文件，从命令参数获取 root 目录，默认是当前目录

```
var root = path.resolve('.');
console.log('Static root dir:' + root);

// 创建服务器
 var server = http.createServer(function (request, response) {
    // node提供url模块解析url字符串 获取url的path

    var pathname = url.parse(request.url).pathname;

    if (pathname === '/favicon.ico') {
        return;
    }

    console.log('url:', url.parse(request.url));

    // 获取对应本地文件路径
    var filepath = path.join(root, pathname);
    // 读取文件状态
    fs.stat(filepath, function (err, stats) {
        // 文件出错
        if (err) {
            console.log('file error!');
            response.end('<h1>file error!</h1>');
            return;
        }

        // 是文件
        if (stats.isFile()) {
            console.log('200 ' + request.url);
            response.writeHead(200);
            // 将文件流导入response
            fs.createReadStream(filepath).pipe(response);
        }
        // 文件不存在
        else {
            console.log('404 ' + request.url);
            response.writeHead(404);
            // 将文件流导入response
            response.end('<h1>404 not found!</h1>');
        }
    });
});

server.listen(8080);

console.info('Server is runing at http://localhost:8080/');
```

express 是第一代流行的 web 框架，它对 Node.js 的 HTTP 进行封装，语法基于 ES5，要实现异步代码，只有一个方法：回调。

koa2 完全基于 ES7 开发，使用 Promise 配合 async 实现异步

```
// 创建一个Koa对象
const app = new Koa();

// 对于任何请求，app将调用该异步函数处理请求
// ctx是koa封装request和response变量
// next是koa传入的将要处理下一个异步函数
// 每个async函数称为middleware
// app.use()顺序决定了middleware的顺序

app.use(async(ctx, next) => {

    fs.readFile('./data.txt', 'utf-8', function(err, data) {
        console.log(data);
    });

    // 调用下一个middleware，如果没有调用，则下一个middleware不会执行
    await next();
});

app.use(async(ctx, next) => {
    ctx.response.type = 'text/html';
    ctx.response.body = '<h1>Hello, koa!</h1>';
    console.log('response end');
    // 调用下一个middleware
    await next();
});

app.listen(3000);
console.log('app started at http://localhost:3000/');
```
