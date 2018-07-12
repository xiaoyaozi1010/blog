---
title: JSONP浏览器端原理及实现
tags: Javascript, JSONP
---

# Jsonp浏览器端原理及实现
前端开发中会常用Ajax来向服务端请求数据。由于浏览器同源策略限制，Ajax不能请求当前域名之外的资源。然而在一些业务场景里，不得不需要从外部域名获取资源，目前比较成熟的方案有：

- 服务端代理转发请求；
- 服务端CORS；
- Jsonp；

本文介绍Jsonp在浏览器端的实现。

<!-- more -->

## 原理
如果你从事过前端开发，应该遇到过以下场景：

> 在`html`页面里通过`<script src="https://www.somecdn.com/jquery.min.js"></script>`的方式引入了位于某外部cdn上的`jquery`文件。
> 此后你便可以愉快的使用jQuery提供的一系列便利方法。

Jsonp正是利用了`src`不具有跨域限制的这个特性来获取当前域之外的数据。 当`jquery.min.js`在下载、解析完成后，将$()方法挂载到当前的`window`对象上。

那么，想一下，如果我们修改一下`$`方法：

```js
function $(data, callback) {
    callback(data) 
}
```

那么，当调用`$`方法时传入一组数据

```js
$({ 
    "code": '1',
    "message": "success",
    "data": {}
}, (data) => {})
```

如果这里把`$`的参数换成服务动态生成的数据，那么`data`就是我们要请求的数据了。只不过这一步需要在服务端处理。

## 实现

```js
function jsonp(url, [parameter, ]callback) {}
```

假定方法名叫做 `jsonp`。

`jsonp`需要接受至少两个参数才能满足我们的使用需求:
- `url` 表示我们要请求的地址
- `callback` 表示请求成功后回去数据的回调（也可以设计为Promise形式）
- `parameter` 是需要传递给服务端的参数，这个不是必须的

### 拼装url

如果我们需要携带参数请求，首要的任务是把参数解析拼装到`url`里。

```js
// 这里用了es6的特性
function jsonp(url, parameter = {}, callback) {
    url += '?';
    let paramArr = Object.keys(parameter).map((key) => `${key}=${parameter[key]}`);
    url += `?${paramArr.join('&')}`;
    // 其他逻辑
}
```

如果你和服务端同事只是口头约定好了将来要执行的方法名，恐怕还不够严谨。所以需要把前端定好的函数名称以参数形式传递。

```js
function jsonp(url, parameter = {}, callback) {
    // ...
    paramArr.push('jsonp=getJsonp');
    url += `?${paramArr.join('&')}`;
}
```

### 创建`script`

这一步创建依赖的`script`标签，之后将拼装好的url设置在`script`元素的`src`属性上。

```js
function jsonp(url, parameter = {}, callback) {
    // ...
    url += `?${paramArr.join('&')}`;
    let script = document.createElement('script');
    script.src = url;
}
```

### 追加至DOM

设置`src`之后浏览器就已经开始加载脚本了。要拿到脚本返回的数据，还需要把脚本执行了才行。执行的方法需要我们提前定义好。

```js
function jsonp(url, parameter = {}, callback) {
    // ...
    paramArr.push('jsonp=getJsonp');
    // ...
    url += `?${paramArr.join('&')}`;
    // ...
    window.getJsonp = function(data) {
        callback(data)
    }
}
```
### 后端处理

后端收到`url`的get请求，从参数中解析出方法名称是`getJsonp`，处理数据之后，以`文本`形式将数据包装成：

```js
getJsonp({
    "code": 1,
    "message": "success",
    "data": {
        // 数据...
    }
})
```

以express为例：

```js
// ...
const app = express();
app.get('/some/path', (req, res) => {
    // 取到jsonp的方法名
    const responseFn = req.params.jsonp;
    const data = {}; 
    // 处理数据 ...code
    data = getData();
    res.status(200).send(`${responseFn}(${JSON.stringify(data)})`)
})
```

## 优化
如果这个方法提供给别人使用的话，需要再做一些优化，比如增加超时检测、中断请求、支持Promise、支持自定义回调名称之类的配置。实际开发中使用，建议将回调名称增加前缀且最好自增，否则同时使用时会出现意想不到的结果。

## Jsonp缺点
由原理得知，jsonp只能支持get请求。某些场景需要使用post时，还是要考虑其他解决方案。