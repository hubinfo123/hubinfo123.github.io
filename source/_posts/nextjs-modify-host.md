---
title: NextJS 项目配置host、浏览器自动访问链接
date: 2023-01-29 19:30
tags: [node-open]
categories: react
excerpt: nextjs 项目中手动实现本地开发环境打开浏览器。
---

> <font face="STCAIYUN" size="2">项目使用 nextjs 框架，一直以来项目启动后，都是手动输入访问链接，而且项目启动后，终端输出的访问链接还是默认的 localhost， 这里就将这 2 个问题一并解决掉，为开发提效...</font>

<font face="STCAIYUN" size="2">
1. 配置正确的host想到该项目里面有部分webpack 的配置，就以为按照webpack 的配置应该就很方便的解决上面的问题，在devServer 处进行了如下修改：<br/>
  devServer：{host: 'local.meet.cn',open: true}<br/>
2. 重新启动，无任何反应，以为是配置参数写错误了，打开webpack 文档，核实了参数，重新运行发现无任何反应，通过webpack 这样去修改，是行不通的，于是在各种找.....<br/></font>

<font face="STCAIYUN" size="2">打开 nextjs 的开发文档，上面也是没有详细提到这样的配置，转到 github nextjs 仓库查看有无类似的 examples ， 几乎翻完了所有自己认为有的 next.config.js 配置文件的目录，还是没有找到，没有办法了，只能自己搞。</font>

 <font face="STCAIYUN" size="2">
1. 进入node_moduels 下的.bin 目录，找到next 启动文件，查看里面的运行方式，找到这一条：<br/>
// dev: ()=>Promise.resolve(require('../cli/next-dev').nextDev)<br/>
2. 进入node_modules 先的next 目录的dist/cli/next-dev.js 文件， 看到了startServer 这个函数在执行:<br/>
<p>
```
const host = args['--hostname'];
    (0, _startServer).startServer({
        allowRetry,
        dev: true,
        dir,
        hostname: host,
        isNextDevCommand: true,
        port
    }).then(async (app)=>{
        const appUrl = `http://${app.hostname}:${app.port}`;
        (0, _output).startedDevelopmentServer(appUrl, `${host || '0.0.0.0'}:${app.port}`); // 这里是可以传host 的，那么在需要在哪里传呢？
```
</p>
</font>

> <font face="STCAIYUN" size="2">接着再看这个文件，上面验证的是的以下 3 个命令：</font>

<font face="STCAIYUN" size="2">
<p>
 ```
 const validArgs = {
   // Types
   '--help': Boolean,
   '--port': Number,
   '--hostname': String,
   // Aliases
   '-h': '--help',
   '-p': '--port',
   '-H': '--hostname'
  };
   这几个命令可以在启动命令进行测试，将 package.json 里面的启动命令改成如下测试：
   "start": "next start -p 8000 -h",
   此时重新启动，终端不会执行，执行到 help 后就结束了，打印如下：

Options
--port, -p A port number on which to start the application
--hostname, -H Hostname on which to start the application (default: 0.0.0.0)
--help, -h Displays this message

从这里可以看出，增加 host 在这里可以用--hostname 的方式，实际项目启动脚本修改如下：
"next": "node scripts/setEnv.js && next -p 8000 --hostname=local.meet.cn"
再次启动, 发现终端的打印不在是 0.0.0.0 和 localhost 了， 变成 started server on local.meet.cn:8000 和
url333: http://local.meet.cn:8000
// url333 是为了测试后面的自动打开浏览器的断点调试，原本打印的是 url:
至此，第一个问题修改 hostname 已经完成了，至少在项目启动后，手动去点击这个地方，也总比去地址栏手动输入要快一些。

```
</p>
</font>


> <font face="STCAIYUN" size="2">接下来解决第二个问题：</font>


<font face="STCAIYUN" size="2">
<p>
```
 刚开始在 node_moduels 下面去搜 url: 没有搜到，于是就想在自定义 server 那块看能不能实现， 
 上面的启动是走的 next 内置的 server, 当时为了使用 https 项目里面也进行了自定义 server 的支持，里面是手动进行输出：
   console.log(`> Ready on https://local.meet.cn:${port}`) // 在终端输出就是这个，很明显打开浏览器是需要手动实现
```
</p>
</font>

<font face="STCAIYUN" size="2">
<p>
```
 在自定义 server 的基础上手动实现打开浏览器
   按照 node 环境打开浏览器的第三方库 yarn add open --save-dev
   增加命令：open(`https://local.meet.cn:${port}`, {app: {name: 'firefox'}}) // 第二个参数去掉，就使用电脑默认设置的浏览器进行打开
   // 在自定义 server 的启动命令上执行，项目启动后，浏览器会自动新开 tab 自动进行链接的访问

```
</p>
</font>


> <font face="STCAIYUN" size="2">继续解决不使用自定义server 启动命令，就是next 自带启动命令，如何打开呢？在 node_modules/next/dist/server/lib/start-server.js 下面是next 维护的一份启动配置：	</font>
![](/images/nextjs-modify-host-1.png)

> <font face="STCAIYUN" size="2">于是再将url: 进行了搜索，发现vscode 上有一个筛选条件被使用，去掉，就搜索出来了，有多个，333 是为了验证执行的文件：	</font>
![](/images/nextjs-modify-host-2.png)

 <font face="STCAIYUN" size="2">其实，到这里就可以看出，通过next内置的启动方式，是无法进行浏览器自动自动打开的，因为nextjs 框架没有去做这个事，而我们的项目里面也一直都是采用https 的方式（nextjs 开启需自定义server.js ）,那就在server.js 里面手动去打开浏览器吧。	
</font>

<font face="STCAIYUN" size="4"><p>~~ 完毕！！！</p></font>