---
title: Electron 开发第一篇：启动不同环境及打包不同环境
date: 2021-01-20 11:29
tags:
---


<font face="STCAIYUN" size="2">最近在开发桌面端应用，昨天第一个版本上线完，今天没有什么紧急的事要做，把桌面端项目的调试环境与打包环境修改了一番，更方便，灵活性更高，相应的提高了一些开发能效。</font>

> <font face="STCAIYUN" size="2">未修改时可能存在的问题：
1.多人协作时，改了BASE_URL 在提交时就要一直记住再改回去，不小心提交了别人合并时得看半天，担心合并错误；
  2.打包时，如果测试需要测试不同环境的包时，要来回的改BASE_URL，这个还是一个变量，如果出现多个变量，那就呵呵了🙃；
  3.太不智能,不够自动化。</font>

<font face="STCAIYUN" size="2">1.解决办法就是在package.json 里面去设置不同环境的命令，来针对不同的环境来执行不同的命令，如下：</font>


```
"scripts": {
    "build": "concurrently \"yarn build:main\" \"yarn build:renderer\"",
    "build:main": "cross-env NODE_ENV=production webpack --config ./.erb/configs/webpack.config.main.prod.babel.js",
    "build:renderer": "cross-env NODE_ENV=production webpack --config ./.erb/configs/webpack.config.renderer.prod.babel.js",
    "rebuild": "electron-rebuild --parallel --types prod,dev,optional --module-dir src",
    "lint": "cross-env NODE_ENV=development eslint . --cache --ext .js,.jsx,.ts,.tsx",
    "package": "yarn build && electron-builder build --publish never",
    "package:dev": "cross-env ENV=dev yarn package",
    "package:test": "cross-env ENV=test yarn package",
    "package:test2": "cross-env ENV=test2 yarn package",
    "package:pre": "cross-env ENV=pre yarn package",
    "package:product": "cross-env ENV=product yarn package",
    "postinstall": "node -r @babel/register .erb/scripts/CheckNativeDep.js && electron-builder install-app-deps && yarn cross-env NODE_ENV=development webpack --config ./.erb/configs/webpack.config.renderer.dev.dll.babel.js && opencollective-postinstall && yarn-deduplicate yarn.lock",
    "base": "node -r @babel/register ./.erb/scripts/CheckPortInUse.js && cross-env yarn start:renderer",
    "start:dev": "cross-env ENV=dev yarn base",
    "start:test": "cross-env ENV=test yarn base",
    "start:test2": "cross-env ENV=test2 yarn base",
    "start:pre": "cross-env ENV=pre yarn base",
    "start:product": "cross-env ENV=product yarn base",
    "start:main": "cross-env NODE_ENV=development electron -r ./.erb/scripts/BabelRegister ./src/main.dev.ts",
    "start:renderer": "cross-env NODE_ENV=development webpack serve --config ./.erb/configs/webpack.config.renderer.dev.babel.js",
    "test": "jest"
  },
```



<font face="STCAIYUN" size="2">在修改过程中发生的问题：
1.node 环境下可以正常拿到环境变量，在electron 中是拿不到的</font>

![](/images/electron1.png)

<font face="STCAIYUN" size="2">2.方法1可以看出写了很多相同的代码，下面就借助inquirer 命令行工具进行简化，可以针对不同的环境进行选择不同的启动命令，如下：</font>

> <font face="STCAIYUN" size="2">
1.启动或者打包；   <br/>
2."inquirer":"TYPE=development node ./scripts/inquirer.js",<br/>
3."inquirer:package":"TYPE=production node ./scripts/inquirer.js",</font>


<br/>
```
//inquirer.js
const inquirer = require('inquirer');
const spawn = require('cross-spawn');
const promptList = [
  {
    type: 'list',
    message: '请选择一种环境:',
    name: 'fruit',
    choices: ['dev', 'test', 'test2', 'pre', 'product'],
    filter: function (val) {
      // 使用filter将回答变为小写
      return val.toLowerCase();
    },
  },
  {
    type: 'list',
    message: '请选择打包平台:',
    name: 'platform',
    choices: ['--win', '--mac'],
    filter: function (val) {
      // 使用filter将回答变为小写
      return val.toLowerCase();
    },
  },
];

if(process.env.TYPE === 'development'){
  promptList.pop();
}
inquirer.prompt(promptList).then((answers) => {
  const spawnType = process.env.TYPE === 'development'?'start':'package';
  const args = process.env.TYPE ==='development'?[spawnType]:[spawnType,answers.platform];
  console.log(args,'args')
  spawn(`yarn`,args, {
    stdio: 'inherit',
    // 仅在当前运行环境为 Windows 时，才使用 shell
    shell: ['win32','win64'].includes(process.platform) ,
    env:{
      ...process.env,
      ENV:answers.fruit
    }
  });
  console.log(answers); // 返回的结果
});

```
