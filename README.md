# jQuery源码学习

[TOC]

## 1. 版本及源码

选择的jQuery版本是v3.4.1，[源码地址](https://unpkg.com/jquery@3.4.1/dist/jquery.js)

## 2. 文件目录

js目录下：
- jQueryOriginal.js是下载过来的源码

- jQuery.js是删除原来的英文注释，增加了自己的中文注释的源码
  - 这里的注释会逐渐完善，其中的内容也会总结在这个文档里

index.html：

- 用于一些测试以及debug

## 3. 学习心得

### 3.1 立即执行函数

```javascript
( function( global, factory ) {
	...
} )( typeof window !== "undefined" ? window : this, function( window, noGlobal ) {
	...
}
```

代码开头就是一个**立即执行函数**，可以**防止代码冲突和污染全局环境**

两个参数：global为window对象，factory为需要执行的回调函数

### 3.2 判断执行环境

#### 3.2.1 CommonJS

```java
( function( global, factory ) {
    "use strict";
    if ( typeof module === "object" && typeof module.exports === "object" ) {
        module.exports = global.document ?
            factory( global, true ) :
            function( w ) {
                if ( !w.document ) {
                    throw new Error( "jQuery requires a window with a document" );
                }
                return factory( w );
            };
    } else {
        factory( global );
    }
} )( typeof window !== "undefined" ? window : this, function( window, noGlobal ) {
	...
}
```

看立即执行函数的内部，主要是判断执行环境是否为CommonJS或类似环境

**module和module.exports**是CommonJS独有的对象，表示**当前模块和对外的接口**

##### 3.2.1.1 noGlobal

- 可以看到，判断CommonJS，主要影响的是factory的第二个参数noGlobal

```javascript
if ( !noGlobal ) {
    window.jQuery = window.$ = jQuery;
}
```

noGlobal只在factory函数的最后用于判断

如果是CommonJS环境，会用require引用，如果不是，就需要挂载在浏览器的window对象下

这里把**jQuery对象和$对象**挂载到window对象下，就是我们通常使用的两个对象

#### 3.2.2 AMD

代码最后除了判断是否CommonJS环境外，还判断了是否遵守AMD规范

```javascript
if ( typeof define === "function" && define.amd ) {
    define( "jquery", [], function() {
        return jQuery;
    } );
}
```

**define函数**是AMD规范定义的唯一一个API

使用方法：define([module-name?], [array-of-dependencies?], [module-factory-or-object]);

### 3.3 jQuery整体结构

factory函数最终返回的是jQuery对象，我们就来看看jQuery对象的定义

```javascript
var jQuery = function( selector, context ) {
    return new jQuery.fn.init( selector, context );
}
```

这里可以看出，jQuery是一个函数（后面还定义了很多属性，因此jQuery既是函数，又是对象）

两个参数：选择器，上下文，返回一个jQuery.fn.init类型的对象（即以此为构造函数的对象）

**参数**就体现了我们平时的用法：$(selector)

那么其**返回值**又有什么意义呢？

#### 3.3.1 jQuery作为函数的返回值

##### 3.3.1.1 jQuery.fn

先看jQuery.fn是什么

```javascript
jQuery.fn = jQuery.prototype = {
	...
    constructor: jQuery,
    ...
}
```

> prototype 和 \_\_proto\_\_简介 ：
>    1.prototype是函数的属性，\_\_proto\_\_是对象的属性
>    2.\_\_proto\_\_指向对象的原型对象
>        作用：当访问一个对象的属性时，如果没有，就会去原型里找
>    3.prototype指向由这个函数创建的实例的原型对象，即constructor.prototype === \_\_proto\_\_
>        作用：由此函数构造的对象，可以有公用的属性和方法，都在prototype里

这里有两个关注点：

1. `jQuery.fn = jQuery.prototype`，jQuery.fn定义为jQuery类型对象（这里指以jQuery为构造器的实例对象，而非jQuery本身）的原型，也就是说，jQuery.fn中定义的所有属性和方法，都是jQuery类型的实例对象所共有的
2. `constructor: jQuery`，jQuery.fn的构造器时jQuery，也就是说jQuery.fn本身也是jQuery类型的对象

##### 3.3.1.2 jQuery.fn.init

从上一节，我们知道jQuery.fn就是jQuery类型的对象，那么它的init函数又构造了怎样一个对象呢

先看看init本身作为一个普通的函数，是什么样的：

```javascript
var init = jQuery.fn.init = function( selector, context, root ){
    ...
}
```

说实话看不出什么，就比jQuery本身多了一个参数

再看看后面对init有什么操作吧

```javascript
init.prototype = jQuery.fn;
```

就在init的定义之后，出现了这一句话

回顾一下，`jQuery.fn = jQuery.prototype`说明，由jQuery构造的对象，其原型为jQuery.fn

这里`init.prototype = jQuery.fn`，说明由init构造的对象，其原型也为jQuery.fn

而jQuery.fn是一个jQuery类型的实例对象，所以**init类型和jQuery类型的对象有共同的原型jQuery.fn**

##### 3.3.1.3 结论

1. **jQuery().\_\_proto\_\_ = jQuery.fn**

2. jQuery作为函数所返回的实例对象，可以使用**jQuery.fn中的属性和方法（共有）**，也可以使用其**构造函数init中定义的属性和方法（私有）**