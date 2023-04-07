---
title: cookie 在nextjs 中的使用
date: 2023-03-06 19:30
tags: [js-cookie]
categories: react
excerpt: 在无需后端帮忙种植 token 情况下，为了完成业务前端自行处理缓存token。
---

<font face="STCAIYUN" size="2">&ensp; &ensp; &ensp;项目优化项利用 nextjs 的服务端渲染，做一些请求，如 userInfo 信息、权限、主体等一些业务逻辑接口，但是由于这些接口需要 token 验证，在不借助后端帮忙的情况下，前端需要处理在服务端拿到 token , 就只能借助 cookie 传递。</font>

> <font face="STCAIYUN" size="2">实现方式：
借助 js-cookie npm 包，进行设置、删除；
key: 字符串,这里指要设置的 token；
token: 项目使用 token 鉴权，登录后会返回 token；
expires: cookie 失效时间，在 js-cookie 中时间是以天为单位开始；</font>

<font face="STCAIYUN" size="2">1. 页面登录成功后，执行
Cookies.set('token',token,{expires:1})
在\_app.js 的 getInitialProps 中派发 ajax 是，需要使用 token 进行鉴权，此时则可以在
appContext.ctx?.req?.cookies?.token 中获取，然后正常使用。</font>

<font face="STCAIYUN" size="2">2. 注意事项：退出登录、token 失效时执行退出登录逻辑时，需要将 token 给手动清除掉
Cookies.remove('token')</font>

<font face="STCAIYUN" size="2">3.expires 是需要设置的，如果不设置会出现一个问题：服务端返回的 token 的失效时间是 24 小时，那么在用户登录后，使用完，不点击退出登录直接关闭浏览器，隔几个小时再次访问页面，此时是直接处于登录态，可以正常使用的，但是手动刷新或者因某些操作触发了页面刷新导致了 getInitialProps 的触发，这个时候 token 是获取不到的，这里涉及一个知识点，就是 cookie 如果不设置失效时间，它的类型为：session，会随浏览器的关闭而清空，如果设置了时间，则会按照时间进行倒计时清空，这里只需要将时间设置为大于等于服务器的时间，因为失效后去访问接口，会提示 token 失效，引导登录。</font>
