---
title: Websocket介绍
tags: 
  - websocket
  - Javascript
---

# Websocket介绍

任何一门技术的产生必然对应一个特定的时代背景。在HTML5规范之前的时代。如果想要实现一种在线聊天的web应用，一个绕不过的门槛就是 __轮询__ 。就是需要在浏览器脚本中定时发送HTTP请求到服务端获取数据。

这种技术具有很明显的缺点，例如HTTP请求包含有很长的HEADER信息，其中有用的信息可能只是很小的一部分，显然这样会浪费大量的带宽等资源。另外，轮询的方式也会浪费大量的服务端资源。

<!-- more -->


## 优点

- 基础协议依然是TCP，服务端实现简单
- 与HTTP兼容性良好端口都是80和403，服务端接口可同时支持两种协议
- 数据格式轻量，通信高效
- 可发送文本和二进制
- __没有同源限制__
- 与HTTP相比只是标识符改变了，http/https - > ws/wss，url不变


## 浏览器端实现

```js
var ws = new WebSocket('ws://www.websocket.org')
wx.addEventListener('open', function(env) {
  console.log('websocket is open')
  console.log(wx.readyState)
  ws.send('Hello server.')
})
wx.addEventListener('message', function(env) {
  console.log('Received data from server:')
  console.log(wx.readyState)
  console.log(env.data)
})
wx.addEventListener('close', function() {
  console.log('Connection closed.')
  console.log(wx.readyState)
})
```
