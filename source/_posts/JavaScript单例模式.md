---
title: JavaScript设计模式 - 单例模式
tags: [设计模式, javascript]
---

## JavaScript单例模式

> 参考自汤姆大叔博客：<http://www.cnblogs.com/TomXu/archive/2012/02/20/2352817.html>

在一些场景中，可能只允许一个类只实例化一次。在第一个实例创建之后，再次实例化这个类时，只返回已经已有实例。具体实现思路，在实例化时，先判断是否已经有实例，有的话直接返回，没有的话再进行实例化。以下是一些实现的思路总结。

<!-- more -->

### 1 最简单的实现，是对象字面量

```js
var singleton = {
    property1: 'name',
    property2: 'age',
    method: function() {
        // do something...
    }
}
```

优点和缺点都比较明显。优点是实现简单，维护方便，思路也较为简单清晰。缺点也显而易见，私有属性（方法）和共有属性（方法）都会暴露出去，封装不彻底；属性或方法很容易会被覆盖重写；对象的引用赋值问题，可能会导致一些一想之外的问题产生。

### 2 闭包封装
在上面例子的基础上，使用闭包封装一下，把只想暴露出去的方法暴露出去，私有属性和方法只在内部生效。
```js
var singleton = function() {
    var previteA = 'sth';
    var publicB = 'test';
    function a() {
        // method a 
    }
    function b() {
        // method b
    }
    return {
        public: publicB,
        a: a,
        b: b
    }
}
// 需要时调用
var single = singleton();
single.a();
single.b();
single.public; // 'sth'
```
上面的代码，本质上仍旧是个静态方法，并没有实例化类的操作。如果想在使用的时候再实例化，那么上面的方法需要做下改造了。

### 3 实例化改造
直接看代码。
```js
var Singleton = (function() {
    function SingletonConstructor(args){
        this.args = args || [];
        // 一些初始化的属性
        this.name = 'Singleton';
        this.pointX = args.pointX || 6;
        this.pointY = args.pointY || 10;
    }
    var instance;
    var _static = {
        name: 'test',
        getInstance: function(args) {
            if(instance === void 0) {
                instance = new SingletonConstructor(args);
            }
            return instance;
        }
    };
    return _static;
})()
var single = Singleton.getInstance({pointX: 5});
single.pointX; // 5
```

