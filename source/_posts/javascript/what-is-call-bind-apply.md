---
title: JavaScript中的 call、apply、bind
date: 2020-11-13 18:10:00
categories: 前端
tags: ['javascript']
---
本文主要探讨JavaScript中的 call、apply、bind是什么？有什么作用？并尝试手写call、apply、bind。
<!--more-->
要理解这三个方法，首先要理解js中`this`的指向。

## 一、this指向的五种情况
### 1.全局上下文中的this
全局上下文中this指向window，这种情况较为简单。
```javascript
    console.log(this === window) // true
```

### 2.事件绑定方法中的this
给元素的某个事件行为绑定方法，当事件行为触发，方法执行，方法中的this指向当前元素本身。
```javascript
  var a = 'window.a'
  var body = document.body
  function func(){
    console.log(this) // this指向window
    console.log(this.a) // window.a
  }
  body.onclick=function(){
    console.log(this) // body
    func() // window.a
  }
```

特殊情况：在IE6-8中，基于attachEvent实现的DOM2事件绑定，事件触发，方法中的this指向window
```javascript
  body.attachEvent("onclick", function(){
    console.log(this)  // window
  })
```

### 3.普通函数执行中的this
函数执行（包含自执行函数执行、普通函数执行、对象成员访问调取方法执行等），只需要看函数执行的时候方法名前面是否有`.`，有的话，`.`前边是谁this就是谁，没有的话this指向window（严格模式下是undefined）
```javascript
  // 1.自执行函数执行
  (function(){
    console.log(this)	// window，开启严格模式的话是undefined
  })()

  let obj = {
    f: (function(){
        console.log(this)   // window
          return function(){}
    })()
  }
  obj.f()

  // 2.普通函数执行
  function func(){
    console.log(this) // this指向window
  }
  func()

  // 3.对象成员访问调取方法执行
  function func(){
    console.log(this) // this指向调用它的对象-> obj
  }
  let obj = {
    f:func
  }
  obj.f()
```

### 4.构造函数中this指向
构造函数体中的this是当前类的实例
```javascript
  function Fun(){
    console.log(this) // this指向实例化Fun的实例-> f
  }
  let f = new Fun()
```

### 5.箭头函数中this指向
箭头函数中没有this，它的this是继承自己所在上下文中的this
```javascript
  var a = 'window.a'
  let obj = {
    a:'obj.a',
    fun: function(){
      console.log(this) // this指向调用它的对象-> obj
      console.log(this.a) // obj.a
    },
    arrowFun: ()=>{
      console.log(this) // 箭头函数中没有this，继承所在上下文中的this
      console.log(this.a) // window.a
    }
  }

  // obj,此处输出的a为 obj.a
  console.log(obj.fun()) // obj, obj.a

  // arrowFun中的this继承obj的this，在此处 obj 的this指向window,所以arrowFun的this指向window
  console.log(obj.arrowFun()) // window, window.a

  // 强制使用之后提到的call去改变this也不行，this还是指向window
  obj.arrowFun.call(obj) // window, window.a
```

## 二、call、apply、bind是什么？有什么用？

每个函数都是`Function`这个类的实例，这三个方法都是`Function`的`prototype(原型)`上的方法，它们可以改变this的指向。

### 1.call 和 apply
[`call`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call) 和 [`apply`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/apply) 方法功能是一样的，只是传入的参数形式不一样，作用都是绑定（劫持）一个特定的执行环境,它们的使用方法是：
```javascript
  func.call(thisArg, arg1, arg2, ...)
  func.apply(thisArg, [argsArray])
```
能够看出它们唯一的不同就是 `call` 方法接受的是一个参数列表，而 `apply` 方法接受的是一个包含多个参数的数组。

```javascript
  // call 方法
  const o_call = {
    name: 'chenwuai'
  }
  function fn(a,b) {
      console.log(this);
      console.log(a + b);
  };
  fn.call(o_call,1,2); // {name:"chenwuai"}  3

  // apply 方法
  var o_apply = {
    name: 'chenwuai',
  }
  function fn(arr) {
      console.log(this);
      console.log(arr);
  }
  fn.apply(o_apply, [1,2]); // {name:chenwuai}  [1,2]
```

一些具体的应用
```javascript
  // 1.求数组最大值
  let arr = [1,2,3]
  Math.max.apply(Math,arr) // 巧妙地利用了第二个参数为数组特征

  // 2.判断数组
  function isArray(obj){ 
    return Object.prototype.toString.call(obj) === '[object Array]' ;
  }
  isArray([1,2,3]) // true
```

### 2.bind
[`bind`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind) 方法创建一个新的函数，在 `bind` 被调用时，这个新函数的 `this` 被指定为 `bind` 的第一个参数，而其余参数将作为新函数的参数，供调用时使用。它的使用方法是：
```javascript
  func.bind(thisArg[, arg1[, arg2[, ...]]])
```

例子：
```javascript
  this.x = 9;    // this 指向 window
  var module = {
    x: 81,
    getX: function() { return this.x; }
  };

  module.getX(); // 81

  var retrieveX = module.getX;
  retrieveX();   
  // 返回 9 - 因为函数是在全局作用域中调用的

  // 创建一个新函数，把 'this' 绑定到 module 对象
  // 新手可能会将全局变量 x 与 module 的属性 x 混淆
  var boundGetX = retrieveX.bind(module);
  boundGetX(); // 81
```
例如Vue2.X中的 mounted生命周期中的匿名函数
```javascript
  let vm = new Vue({
    data(){
      return {
        height:768
      }
    },
    mounted(){
      window.addEventListener("resize",(function(){
        this.height = document.body.clientHeight
      }).bind(this)) // this指向vm
    }
  })
```
偏函数例子：
```javascript
  function list() {
    return Array.prototype.slice.call(arguments);
  }

  function addArguments(arg1, arg2) {
      return arg1 + arg2
  }

  var list1 = list(1, 2, 3); // [1, 2, 3]

  var result1 = addArguments(1, 2); // 3

  // 创建一个函数，它拥有预设参数列表。
  var leadingThirtysevenList = list.bind(null, 37);

  // 创建一个函数，它拥有预设的第一个参数
  var addThirtySeven = addArguments.bind(null, 37); 

  var list2 = leadingThirtysevenList(); 
  // [37]

  var list3 = leadingThirtysevenList(1, 2, 3); 
  // [37, 1, 2, 3]

  var result2 = addThirtySeven(5); 
  // 37 + 5 = 42 

  var result3 = addThirtySeven(5, 10);
  // 37 + 5 = 42 ，第二个参数被忽略
```

## 三、手写简单的call、apply、bind
仅为简单实现，未做边界判断等容错处理。

### 1.实现call
```javascript
  Function.prototype.mycall = function(context,...args){
    // this 为调用 myCall 的函数
    const func = this;
    
    // call方法只能用于function
    if (typeof this !== "function") {
      throw new TypeError("not a function")
    }

    //这里默认不传就是给window(非严格模式下),也可以用es6给参数设置默认参数
    context = context || window
    args = args ? args : []

    //给context新增一个独一无二的属性以免覆盖原有属性
    const key = Symbol()
    context[key] = this

    //通过隐式绑定的方式调用函数
    const result = context[key](...args)

    //删除添加的属性
    delete context[key]

    //返回函数调用的返回值
    return result
  }
```

### 2.实现apply
apply和call的区别只有参数的区别，所以实现方式几乎一样
```javascript
  //只有args参数有变化
  Function.prototype.myapply = function(context,args){
    const func = this;
    if (typeof this !== "function") {
      throw new TypeError("not a function")
    }
    context = context || window
    args = args ? args : []
    const key = Symbol()
    context[key] = this
    const result = context[key](...args)
    delete context[key]
    return result
  }
```

### 3.实现bind
以下是两种实现bind的方法
```javascript
  // 第一种
  Function.prototype.bind = function(context,...args){
    const func = this
    if(typeof func !== 'function'){
      throw new TypeError('Not A Function!')
    }
    
    args = args || []

    return function newFn(...newArgs){
      if(this instanceof newFn){
        return new func(...args, ...newArgs)
      }
      return func.apply(context,[...args,...newArgs])
    }
  }
```
```javascript
  // 第二种
  var ArrayPrototypeSlice = Array.prototype.slice;
  Function.prototype.bind = function(otherThis) {
    if (typeof this !== 'function') {
      // closest thing possible to the ECMAScript 5
      // internal IsCallable function
      throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable');
    }

    var baseArgs= ArrayPrototypeSlice.call(arguments, 1),
        baseArgsLength = baseArgs.length,
        fToBind = this,
        fNOP    = function() {},
        fBound  = function() {
          baseArgs.length = baseArgsLength; // reset to default base arguments
          baseArgs.push.apply(baseArgs, arguments);
          return fToBind.apply(
                 fNOP.prototype.isPrototypeOf(this) ? this : otherThis, baseArgs
          );
        };

    if (this.prototype) {
      // Function.prototype doesn't have a prototype property
      fNOP.prototype = this.prototype; 
    }
    fBound.prototype = new fNOP();

    return fBound;
  };
```