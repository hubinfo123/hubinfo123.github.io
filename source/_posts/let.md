---
title: 从Babel 编译理解let const 
date: 2020-08-28 23:59:46
tags: [JS]
categories: js 基础
excerpt: 学习let 变量。
---


<font face="STCAIYUN" size="3">首先let 和 const 是es6 出的，这里就要了解一下，为什么要出新语法，一般是新事物带替旧事物，必然是旧事物满足不了当前，梳理一下es5 里面的变量声明会存在哪些不足，然后再从let 和 const 上去了解他们的优点。</font>

> <font face="STCAIYUN" size="2">es5 变量声明，也就是var 会存在的问题</font>

<font face="STCAIYUN" size="2">1.内层变量可能会覆盖外层变量</font>
<font face="STCAIYUN" size="2">2.循环中的计数变量可能会泄漏成为全局变量</font>

> <font face="STCAIYUN" size="2">那么立足es5 本身有没有什么办法去解决呢？或者是做个块级作用域，想到作用域，js 里面只有函数存在作用域，那么就可以利用函数来实现</font>

```
var a = 1;
(function IIFE(){
  var a = 2;
  console.log(a); //2
})()
console.log(a);//1
```

> <font face="STCAIYUN" size="2">当在es6 中引入了块级作用域，也就是在花括号里面存在let 或者是const 时，花括号所在的区域内就是属于块级作用域了，它的优点是：</font>

<font face="STCAIYUN" size="2">1.外层作用域无法读取内层作用的变量，也就是es5 的缺点1就不复存在了</font>
<font face="STCAIYUN" size="2">2.外层声明的变量名可以与内层的相同，互不干扰，不用当心会被修改</font>
<font face="STCAIYUN" size="2">3.用函数来模拟或者是立即执行函数就不在需要了</font>


<font face="STCAIYUN" size="2">下面来看一组源码示例：</font>
```
var x = 1;
let y = 1;
if (true) {
  var x = 2;
  let y = 2;
}
console.log(x); // 2
console.log(y); // 1
```

<font face="STCAIYUN" size="2">被babel 编译解析后的示例：</font>
```
"use strict";
var x = 1;
var y = 1;
 
if (true) {
  var x = 2;
  var _y = 2;
}
console.log(x); // 2
```

<font face="STCAIYUN" size="2">上面的示例可以看出2个问题：</font>
<font face="STCAIYUN" size="2">1.由于 let 使花括号提升为块级作用域，使得即使声明了相同的变量名 y 也互不干扰。  
2.为了实现此效果，Babel 重命名了块级作用域内 let 声明的变量名，也就是可以理解为babel 在解析let 语法糖的时候就是将变量名重命名了。</font>


<font face="STCAIYUN" size="2">参照:
https://blog.csdn.net/weixin_34290390/article/details/87986287?utm_medium=distribute.pc_relevant.none-task-blog-title-1&spm=1001.2101.3001.4242</font>



