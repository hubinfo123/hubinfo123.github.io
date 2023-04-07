---
title: CSS盒模型
date: 2018-07-24 23:27:46
tags: [CSS]
categories: css 基础
excerpt: 盒模型学习。
---

##### 1.盒模型的概念

css盒模型是指所有HTML元素都可以做看作是一个盒子，从内到外由内容（content），内边距（padding），边框（border），外边距（margin）组成


##### 2.盒模型的种类

```
box-sizing=conten-box; //标准盒子模型
box-sizing=border-box; //IE盒子模型
```

![](https://cdn.nlark.com/yuque/0/2020/jpeg/2311579/1596783142768-cc4728ce-8829-419a-a18c-db9199d608ea.jpeg)
• 标准盒模型，设置的width或height是对实际内容（content）的width或height进行设置，内容周围的border和padding另外设置，即盒子模型的width（height）=设置的content的宽高+padding+border+margin
注：除内容content外，其他为上下左右都有
• IE盒子模型，设置的width或height是对 实际内容（content）+内边距（padding）+边框（border）之和的width和height进行设置的，其盒模型的width（height）=设置的width（height）+外边距margin


##### 3、box-sizing的应用

box-sizing 属性允许您以特定的方式定义匹配某个区域的特定元素（个人认为可以理解为指定盒模型的类型，即上述两种类型）
• box-sizing值为content-box时：宽度和高度分别应用到元素的内容框，在宽度和高度之外绘制元素的内边距和边框（即标准盒模型）
• box-sizing值为border-box时：为元素设定的宽度和高度决定了元素的边框盒，就是说，为元素指定的任何内边距和边框都将在已设定的宽度和高度内进行绘制，通过从已设定的宽度和高度分别减去边框和内边距才能得到内容的宽度和高度（即 IE盒模型）
• box-sizing值为inherit时：规定应从父元素继承 box-sizing 属性的值
