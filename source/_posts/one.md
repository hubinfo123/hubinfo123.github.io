---
title: JSON.stringify的魔法
date: 2020-08-25 23:27:46
tags:
---
JSON.stringify 是将javascript对象转换成JSON字符串;对应的还有一个JSON.parse,它是将JSON字符串转换为javascript对象，取到互补的作用；

##### 日常工作中常用的场合如下：

###### 1.在使用浏览器缓存的时候，先要将缓存的对象转换成JSON字符串，取出的时候再将字符串转换为对象

```
const auth = {
	id:001,
  create_time:'Wed Dec 11 2019 23:19:57 GMT+0800 (中国标准时间)',
  name:'yy'
}
//存入
localStorage.setItem('auth',JSON.stringify(auth));

//调用
const currentAuth = JSON.parse(localStorage.getItem('auth'));

```

###### 2.判断数组中是否包含某个对象
```
const data = [
  {type:'javascript'},
  {type:'html'},
  {type:'css'}
];
JSON.stringify(data).includes(JSON.stringify({type:'css'}));//true

```

###### 3.判断数组/对象是否相等
```
const a = [1,2,3];
const b = [1,2,3];
JSON.stringify(a) == JSON.stringify(b);//true

```

###### 4.数据深拷贝
```
const a = [1,2,3];
const b = JSON.parse(JSON.stringify(a));
b[0] = 0;
console.log(a,b);//[1,2,3] [0,2,3]

```

###### 5.数据处理
```
//处理前的对象
const todayILearn = {
  _id: 1,
  content: '今天学习 JSON.stringify()，我很开心！',
  created_at: 'Mon Nov 25 2019 14:03:55 GMT+0800 (中国标准时间)',
  updated_at: 'Mon Nov 25 2019 16:03:55 GMT+0800 (中国标准时间)'
};
//处理后的对象
const todayILearn = {
  id: 1,
  content: '今天学习 JSON.stringify()，我很开心！',
  createdAt: 'Mon Nov 25 2019 14:03:55 GMT+0800 (中国标准时间)',
  updatedAt: 'Mon Nov 25 2019 16:03:55 GMT+0800 (中国标准时间)'
}

//实现方法
1.创建新的对象 + 遍历实现
// 多一个变量存储
const todayILearnTemp = {};
for (const [key, value] of Object.entries(todayILearn)) {
  if (key === "_id") todayILearnTemp["id"] = value;
  else if (key === "created_at") todayILearnTemp["createdAt"] = value;
  else if (key === "updatedAt") todayILearnTemp["updatedAt"] = value;
  else todayILearnTemp[key] = value;
}
console.log(todayILearnTemp);
// 结果：
// { id: 1,
//  content: '今天学习 JSON.stringify()，我很开心！',
//  createdAt: 'Mon Nov 25 2019 14:03:55 GMT+0800 (中国标准时间)',
//  updated_at: 'Mon Nov 25 2019 16:03:55 GMT+0800 (中国标准时间)'
// }
缺点：多声明了一个变量，加了层循环，多个if else 判断

2.先delete 属性，再增加属性
todayILearn.id = todayILearn._id;
todayILearn.createdAt = todayILearn.created_at;
todayILearn.updatedAt = todayILearn.updated_at;
delete todayILearn._id;
delete todayILearn.created_at;
delete todayILearn.updated_at;
console.log(todayILearn);
//     太暴力😢
//{
//  content: '今天学习 JSON.stringify()，我很开心！',
//  id: 1,
//  createdAt: 'Mon Nov 25 2019 14:03:55 GMT+0800 (中国标准时间)',
//  updatedAt: 'Mon Nov 25 2019 16:03:55 GMT+0800 (中国标准时间)'
//}
缺点：属性的顺序发生了改变

3.序列化 + replace
const mapObj = {
  _id: "id",
  created_at: "createdAt",
  updated_at: "updatedAt"
};
JSON.parse(
  JSON.stringify(todayILearn).replace(
    /_id|created_at|updated_at/gi,
    matched => mapObj[matched])
    )

// {
// id: 1,
//  content: '今天学习 JSON.stringify()，我很开心！',
//  createdAt: 'Mon Nov 25 2019 14:03:55 GMT+0800 (中国标准时间)',
//  updatedAt: 'Mon Nov 25 2019 16:03:55 GMT+0800 (中国标准时间)'
// }

```

##### JSON.stringify 与 toString 的区别
```
const a = [1,2,3];
console.log(JSON.stringify(a));//"[1,2,3]"
console.log(a.toString());//"1,2,3"  

```

>二者虽然都是转成了字符串，但是本质上还是有区别的，JSON.stringify 主要是将javascript 对象转换成字符串，偏向于操作对象类型，而toString 更偏向于数字类型转换为字符串类型

>下面来看看JSON.stringify 的几大特性

###### 特性1：对于undefined,任意类型函数，symbol类型的数据，分别作为对象的属性，数组中的元素，单独的值时，JSON.stringify 将返回不同的结果

```
a.作为对象属性值存在，JSON.stringify 将跳过（忽略）他们，然后进行序列化
  const data = {
    a: "aaa",
    b: undefined,
    c: Symbol("dd"),
    fn: function() {
      return true;
    }
  };
  JSON.stringify(data); // 输出：？

  // "{"a":"aaa"}"

b.作为数组元素存在，JSON.stringify 将他们序列化为null
  JSON.stringify(["aaa", undefined, function aa() {
      return true
    }, Symbol('dd')])  // 输出：？

  // "["aaa",null,null,null]"

c.单独序列化的转换结果？？？
JSON.stringify(function a (){console.log('a')})
// undefined
JSON.stringify(undefined)
// undefined
JSON.stringify(Symbol('dd'))
// undefined

```