---
title: nextjs SSG 使用
date: 2023-03-17 18:00
tags:
---

<font face="STCAIYUN" size="2">&nbsp;&nbsp;&nbsp;&nbsp;在当前需求开发进入测试阶段，新需求还未进入评审阶段，如是想优化一下当前系统，如预渲染机制使用起来，于是就用到了 nextjs 的 SSG（静态生成），使用方式也还算比较简单，参照文档既可进行将现有页面进行改造，这里主要是将过程中遇到的问题进行记录。</font>


<font face="STCAIYUN" size="2">
<p>
```
  # 在首页上进行改造，改页面数据几乎不会随意变动，符合SSG 要求;

  # 1. 在首页增加getStaticProps 方法
  export const getStaticProps = async () => {
    let hotList = []
    try {
      const {data} = await getTemplateList({
        ...params // 接口需要的参数
      })
      hotList = get(data, 'result.items', []) || []
    } catch (error) {
      console.log(error,'打印error')
    }
    return {
      props: {
        hotList,
      },
    }
  }

  # 2. 去掉之前在CSR 端获取数据方法

  # 3. 启动本地开发环境，首页可以正常访问，打开调试器，浏览器ajax 中是没有请求了，在node 端打印，可以看到请求派发，
       本地调整完毕，发版测试验证；

  //本地开发环境
```
</p>
</font>

> <font face="STCAIYUN" size="2">发版报错：</font>
<font face="STCAIYUN" size="2">
<p>
```
# 1. 带有ajax 请求的代码提交后，部署报错：
  Error occurred prerendering page "/". Read more: https://nextjs.org/docs/messages/prerender-error
  Error: connect EAFNOSUPPORT ::1:80 - Local (undefined:undefined)
      at internalConnect (net.js:934:16)
      at defaultTriggerAsyncIdScope (internal/async_hooks.js:452:18)
      at net.js:1022:9
      at processTicksAndRejections (internal/process/task_queues.js:77:11)
  [0minfo  - Generating static pages (1/1)
  [91m
  [0m[91m> Build error occurred
  [0m[91mError: Export encountered errors on following paths:
    /
    at /app/node_modules/next/dist/export/index.js:423:19

  出现这个错误，以为是写入方式有问题，搜索了很多文档，没有类似问题，尝试将其他顶部文件里面的生命周期如getInitialProps 替换getServerSideProps 、
  是否需要配合getStaticPaths （这个是没有仔细阅读文档的猜测，这个是动态路由需要使用的）均未解决

# 2. 尝试其他方法，不进行 ajax ，直接返回，如下是可以的，部署能成功
  export const getStaticProps = async () => {
    return { props: { initialState: { lastUpdate: Date.now() } } }
  }

# 3. 接着断点调试，在ajax 处打印config、response 处进行打印，发现是代码里面写了一处throw error 导致部署终端（问题不在这），于是调整了ajax 请求，直接打印错误，报如下问题：
info  - Linting and checking validity of types...
[91mwarn  - The Next.js plugin was not detected in your ESLint configuration. See https://nextjs.org/docs/basic-features/eslint#migrating-existing-config
[0m[91mFailed to compile.

./src/pages/index.tsx:53:10
Type error: Property 'forEach' does not exist on type 'void | [unknown, unknown]'.
  Property 'forEach' does not exist on type 'void'.

[0m [90m 51 | [39m  console[33m.[39mlog(values[33m,[39m [32m'values'[39m[33m,[39m hotResult[33m,[39m clickResult)[0m
[0m [90m 52 | [39m  [90m// 处理promiseAll 的401[39m[0m

这个报错是因为拿不到结果，直接对结果进行forEach 导致的，主要问题还是开发环境请求可以成功，build 时发现请求失败，接着排查错误...

# 4. 验证一下线上的请求配置同本地的请求配置是否一样，对比一下差异点，结果发现baseUrl 在构建的时候丢失，这个同项目搭建有关系，
     项目是打包后再注入环境变量，打包是是没有环境变量的，所以构建时获取不到环境对应的后端接口地址，如是在请求的地方兼容了一下：
// SSG 先build 发送请求是取不到baseUrl
  if(!config.baseURL){
    config.baseURL = envToHost[process.env.APP_ENV]
  }
// envToHost 为各个环境的枚举
```
</p>
</font>

<font face="STCAIYUN" size="2">&nbsp;&nbsp;&nbsp;&nbsp;重新发版，请求成功，数据正常显示，调试查看浏览器ajax 未发，在node 端打包编译时进行了请求，页面切换不会等待数据，提升了部分渲染速度。</font>