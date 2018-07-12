---
title: JSBridge设计
tags: JSBridge, Javascript, Bridge设计
---

# 混合开发中的JSBridge设计

包含了常用的JSBridge功能特性。如UA检测、硬件调用、错误约定等。

<!-- more -->

## 环境检测

**`navigator.userAgent`**

以`smartoneapp(待定)/x.x(app 版本)`为关键字标识为SOA内嵌

** iOS **

```bash
mozilla/5.0 (iphone; cpu iphone os 5_1_1 like mac os x) applewebkit/534.46 (khtml, like gecko) mobile/9b206 smartoneapp/1.0
```

** Android **

```bash
mozilla/5.0 (linux; u; android 4.1.2; zh-cn; mi-one plus build/jzo54k) applewebkit/534.30 (khtml, like gecko) version/4.0 mobile safari/534.30 smartoneapp/1.0
```

---

## 初始化

机制原因，JavaScript注入api、与native端交互均为异步API注入后，会在全局对象(window)注入属性: `JSJSBridge`。下文代码示例/语法均以缩写形式展示。

```js
// 属性挂载
window.JSJSBridge
```

---

## 错误约定

__任何的API调用，都是异步的__。在调用回调中，如果发生任何错误，会在参数中传入错误信息，信息如下：

错误种类待定。

```js
{
	error: 1,
	errorMessage: '接口不存在'
}
```

示例：

| error | errorMessage |
|---|---|
|1 | 接口不存在
|2 | 参数无效
|3 | 发生未知错误
|4 | 接口无权限（用户未授权）

---

## <font color="gray">事件</font>

### on _[function]_

通过`on`函数来监听事件。

#### 语法

```js
JSBridge.on(type: String, listener: Function)
```

##### *type*

事件类型字符串。不同的事件类型，对应不同的事件通知(listener参数)，事件类型见下方

##### *listener*

事件侦听函数，当监听的事件触发时，会接受到一个事件通知，触发此函数。函数参数为通用事件对象。

##### *事件类型*

| type | 描述 | 备注
|--- | --- | --- |
|menuChange | 监听Hamburger展开状态 | 需要从事件对象中获取展开状态（关闭/展开）|
|optionClick | 监听titleBar右上角option按钮点击事件 | --- |
|orientationChange | 手机方向改变 | 如果app禁用了横屏，需要透此时间给js调用，使用`JSBridge.on`绑定|


##### *事件对象*

默认情况下，事件对象只包含timeStamp这一固定属性，其余属性视事件种类而定。

事件示例：

```js
JSBridge.on('menuChange', (e) => {
	const { timeStamp, status } = e
})
```

| 属性 | 事件 | 说明 | 备注 |
|---|---|---|---|
| timeStamp | * | 事件创建时的时间戳 | 此时间戳表示页面加载完成，距离事件发生时的时间，单位：毫秒
| menuStatus | menuChange | Hamburger的展开状态 | 'SHOW' 或者 'HIDE'
| orientation | orientationChange | native端禁用横向模式，通过此事件监测手机横向模式 | 取值：0\90\180\270



### off  *[function]*

解除由addEventListener绑定事件，listener必须与addEventListener listener具有相同引用

```js
JSBridge.on(type: String, listener: Function)
```

---

## 注意：

__实现机制原因，所有回调方法必须传递回调方法名或者方法体（string类型），且必须挂载在全局之下。__

示例如下：

```js
window.setStorageCallback = (data) => {
	console.log(data)
}

// 传入函数名称
JSBridge.setStorage('setStorageCallback')

// 传入函数体字符串
JSBridge.setStorage(setStorage.toString())
```

本质上，Native在window注入JSBridge下的所有方法，只接受字符串做为参数。具体原理是Native端会将回调方法拼入参数之后，再传入eval执行。所以callback在没有bind到其他作用域的情况下，默认是在全局执行的。`this`指向window。


以下demo均一致。

## Native功能

### 媒体相关

#### *photo*

唤起系统拍照/选择照片界面

*JSBridge.photo(callback: [string])*

示例:

```js
global.callbackName = (res) => {
	const {path, size} = res
}
JSBridge.photo('callbackName')
```

__参数说明：__

| 参数 | 类型 | 含义 |
|---|---|---|
| callback |  string  | 选择图片成功回调 |

__回调参数：__

| 属性 | 类型 | 含义 | 备注
|---|---|---|---|
| path  | string  | 图片上传路径 | 包含文件名、文件后缀 |
| size  |  number | 文件大小 | 单位`kb` |


#### <font color="gray"> *imageViewer* </font>

唤起系统图片预览界面


#### \* *saveImage*

保存图片至系统相册，需用户授权


#### \* *compressImage*

压缩图片，用于上传前的压缩


#### \* *playAudio*

播放声音、音乐


#### \* *pauseAudio*

暂停声音、音乐


#### \* *stopAudio*

停止播放声音、音乐


#### playVideo

调起native播放器播放给定URL的视频，只能播放网络视频，不支持本地视频播放。

*JSBridge.playVideo(url: [String], callback: [String])*

示例:

```js
global.playCallback = (res) => {
	// 播放开始回调
}
JSBridge.playVideo('http://path/to/video/video.mp4', 'playCallback')
```

__参数说明：__

| 参数 | 类型 | 含义 |
|---|---|---|
| url |  string  | 视频URL，只支持网络视频 |
| callback   |  string  |  开始播放的回调名称  |


#### <font color="gray">\* pauseVideo</font>

暂停视频


#### \* stopVideo

停止播放视频


### 地理位置

#### getCurrentLocation

获取设备地理位置

*JSBridge.getCurrentLocation(callback: [Function])*

__参数说明：__


| 参数 | 类型 | 含义 |
|---|---|---|
| callback   |  function  |  获取位置信息回调  |

__回调参数：__

回调参数是一个[Coordinate对象](https://developer.mozilla.org/zh-CN/docs/Web/API/Coordinates)

| 属性 | 类型 | 含义 | 备注 |
|---|---|---|---|
| latitude   |  double  |  当前位置纬度坐标  | - |
| longitude    | double  |  当前位置经度坐标 | - |
| altitude    |  double | 相对于海平面的位置高度(米)  | 未实现 |
|speed   | double  |  设备当前移动速度（米/秒） | 未实现 |

示例:

```js
global.locationCb = (coord) => {
	const {latitude, longitude} = coord
	console.log(`latitude=${latitude}, longitude=${longitude}`) //
}

JSBridge.getCurrentLocation('locationCb')
```

#### <font color="gray">\* *startContinuousLocation*</font>

开始持续获取当前地理位置，1次/500ms


#### \* stopContinuousLocation

停止持续获取当前地理位置


### 分享

#### shareTo

分享当前页面到微信（朋友圈、好友）

*JSBridge.shareTo(option, callback)*

__参数说明：__

| 参数 | 类型 | 含义 | 备注 |
|---|---|---|---|
| option   |  object  |  分享配置  | 配置见下方
|  callback   | function  | 分享结束回调  |  用户取消、分享失败、成功都会回调

__配置说明：option__

| 属性 | 类型 | 说明 | 必选 | 备注 |
|---|---|---|---|---|
| url   |  string | web页面url | 是 |  -  |
| title   | string  | 分享标题  | 是 | -  |
| description   | string  | 描述文字 | 是 | 分享平台不同，超长截断字数也不同  |
| image   |  string | 图标url  |  否 | 网络地址 |
| target   | array  |  分享的目标平台数组  |  否  |  接受`["wechat", "timeline", "sina"]`，默认全部分享渠道 |

__回调参数说明：__

| 属性 | 类型 | 含义 | 备注 |
|---|---|---|---|
| status   |  boolean  |  分享结果：true、false  | `true` or `false` |
| target    | array[string]  |  用户分享渠道 | - |

示例:

```js
const shareConfig = JSON.stringify({
	url: 'http://estore-dev.china.smart.com/pdp?xxxx=xxx',
	title: '奔驰GLE 中国红',
	description: '描述文字',
	image: '/static/share/smart-share.jpg',
	target: ['wechat', 'timeline']
})
global.shareCb = (res) => {
	const { status, target } = res
	console.log(`分享结果：${status ? '成功' : '失败'}, 分享至：${target[0]}`)
}

JSBridge.shareTo(shareConfig, 'shareCb')
```

### <font color="gray">\* 网络</font>

#### *getNetworkType*

获取网络类型



### *界面、组件*

#### <font color="gray">\* toast</font>

标准化UI 提示框

#### <font color="gray">\* confirm</font>

标准化UI 确认对话框


#### <font color="gray">\* datePicker</font>

系统日期选择器


#### <font color="gray">\* dateTimePicker</font>

系统日期时间选择器


#### <font color="gray">\* showLoading</font>

显示`加载中...`提示窗


#### <font color="gray">\* hideLoading</font>

隐藏`加载中...`提示窗


#### setTitle

设置titleBar文字内容

*JSBridge.setTitle(title: [String], callback: [String])*

__参数说明：__

| 参数 | 类型 | 含义 | 备注 |
|---|---|---|---|
| title   |  string  |  title内容  | - |
| callback | 回调  |  -  | - |

示例:

```js
global.setTitleCb = () => {
	console.log('新的标题已经生效了')
}

JSBridge.setTitle('新的标题文字', 'setTitleCb')
```

#### setTitleBgColor

设置titleBar背景色

*JSBridge.setTitleBgColor(color: [String], callback: [Function])*

__参数说明：__

| 参数 | 类型 | 含义 | 备注 |
|---|---|---|---|
| color   |  string  |  16进制颜色  | - |
| callback | 回调  |  -  | - |

示例:

```js
global.setTitleBgColorCb = () => {
	console.log('新标题背景色生效了')
}

JSBridge.setTitleBgColor('#FF3390', 'setTitleBgColorCb')
```

#### \* setTitleColor

设置titleBar文字颜色

*JSBridge.setTitleColor(color: [String], callback: [Function])*

__参数说明：__

| 参数 | 类型 | 含义 | 备注 |
|---|---|---|---|
| color   |  string  |  16进制颜色  | - |
| callback | 回调  |  -  | - |

示例:

```js
JSBridge.setTitleColor('#FF3390', () => {
	console.log('新标题色生效了')
})
```

#### \* hideTitlebar

隐藏title栏

*JSBridge.hideTitlebar(callback: [Function])*

__参数说明：__

| 参数 | 类型 | 含义 | 备注 |
|---|---|---|---|
| callback | 回调  |  -  | - |

示例:

```js
JSBridge.hideTitlebar(() => {
	console.log('标题栏隐藏了')
})
```


#### \* showTitlebar

显示title栏

*JSBridge.showTitlebar(callback: [Function])*

__参数说明：__

| 参数 | 类型 | 含义 | 备注 |
|---|---|---|---|
| callback | 回调  |  -  | - |

示例:

```js
JSBridge.showTitlebar(() => {
	console.log('标题栏显示出来了')
})
```

#### \* hideTab

隐藏tab栏

*JSBridge.hideTab(callback: [Function])*

__参数说明：__

| 参数 | 类型 | 含义 | 备注 |
|---|---|---|---|
| callback | 回调  |  -  | - |

示例:

```js
JSBridge.hideTab(() => {
	console.log('tab栏被隐藏了')
})
```

#### \* showTab

显示tab栏

*JSBridge.showTab(callback: [Function])*

__参数说明：__

| 参数 | 类型 | 含义 | 备注 |
|---|---|---|---|
| callback | 回调  |  -  | - |

示例:

```js
JSBridge.showTab(() => {
	console.log('tab栏显示出来了')
})
```

#### <font color="gray">\* setLandscape</font>

设置横屏或竖屏


### 通信

#### \* navigateTo

跳转至native页面

*JSBridge.navigateTo(option: [Object], callback: [Function])*

__参数说明：__

| 参数 | 类型 | 含义 | 备注 |
|---|---|---|---|
| option   |  object  |  跳转配置  | 配置见下方

__配置说明：option__

| 属性 | 类型 | 说明 | 必选 | 备注 |
|---|---|---|---|---|
| name   |  string | native页面名称 | 是 |  现支持`["login", "discovery"]` |
| param   |  object  |  传递数据  | 否 |  -  |

示例:

```js
JSBridge.navigateTo({
	name: 'login',
	param: {
		redirectUrl: '/dealers?xxx=xxx',
		someDataForNative: {
			a: 'a',
			b: 'b'
		}
	}
}, () => {
	console.log('跳转至native页面了')
})
```

#### setStorage

向客户端存储数据，登录json、cookie、全局共用数据等

*JSBridge.setStorage(option: [Object], callback: [Function])*

__参数说明：__

| 参数 | 类型 | 含义 | 备注 |
|---|---|---|---|
| option   |  object  |  storage配置  | 配置
| callback   | function  |  回调  |  -  |

__配置说明：option__

| 属性 | 类型 | 说明 | 必选 | 备注 |
|---|---|---|---|---|
| namespace   |  string | 存储namespace | 是 | 如登录：'CIAM' |
| data   |  object  |  json数据  | 是 |  需做`JSON.stringify`处理  |

示例:

```js
const storage = JSON.stringify({
	namespace: 'CIAM',
	data: {
		responseFromCIAM: {
			a: 'a',
			b: 'b'
		}
	}
})

global.setStorageCb = () => {
	console.log('设置数据成功')
}
JSBridge.setStorage(storage, 'setStorageCb')
```

#### getStorage

向客户端存储数据，登录json、cookie、全局共用数据等

*JSBridge.getStorage(option: [Object], callback: [Function])*

__参数说明：__

| 参数 | 类型 | 含义 | 备注 |
|---|---|---|---|
| option   |  object  |  storage配置  | - |
| callback   | function  |  回调  |  -  |

__配置说明：option__

| 属性 | 类型 | 说明 | 必选 | 备注 |
|---|---|---|---|---|
| namespace   |  string | 存储namespace | 是 | 如登录：'CIAM' |

__回调参数说明：__

| 参数 | 类型 | 说明  | 备注 |
|---|---|---|---|
| data   |  object  |  json数据  | 是 |  需做`JSON.parse`处理  |

示例:

```js
JSBridge.getStorage({
	namespace: 'CIAM'
}, (res) => {
	const data = JSON.parse(res)
	console.log(data)
	/**
	=> 输出：
	responseFromCIAM: {
		a: 'a',
		b: 'b'
	}
	*/
})
```
