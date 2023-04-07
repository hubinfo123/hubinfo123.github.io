---
title: Element-UI Rate 评分使用自定义icon
date: 2020-11-19 22:20
tags: [Element]
categories: vue
excerpt: 抹平Element 评分组件与Antd 评分组件样式差异。
---


<font face="STCAIYUN" size="3">项目里面使用了React 与 Vue 进行同时开发，React 对应的使用的是Antd,Vue 则是用Element 这些组件库进行开发，在同一个项目下使用这2种框架，本身就依赖了vuera 来进行相互协作，无论是React 嵌入到Vue 里面，还是Vue 嵌入到React 里面，本来Vue 写得不多，所以这边就没有使用vuera 来完成，一个查看评分功能，在React 模块下用antd 的Rate 完成，在Vue 下用Element 的Rate 进行开发，想加深一下Vue 的开发。但是最终导致的结果是，查看评分的弹框，里面的星星大小颜色都不一样，需要统一一下。</font>

> <font face="STCAIYUN" size="2">解决思路：
1.通过自定义icon ,让星星大小、样色一样 ,
  2.antd 的 Rate 使用character、element 的 Rate 使用icon-classes，
  3.使用iconfont 配合完成</font>


  ```
antd Rate：
  // 非自定义：<Rate value={subItem.value} disabled count={item.name === '增值评分'?3:5}/>
  <Rate value={subItem.value} disabled count={item.name === '增值评分'?3:5} character={()=><IconSvg type="iconstar" style={{ color: '#ffc300' }} />} />

  // IconSvg 解析 阿里iconfont 
  const IconSvg = (props:{ type: string, className?:string, style?:any}) => {
    return (
      <svg className={`icon ${props.className || ''}`} aria-hidden="true" style={props.style}>
        <use xlinkHref={`#${props.type}`}></use>
      </svg>
    )
  }
  export default IconSvg

element Rate:
  // 非自定义：<el-rate v-model="subItem.value" disabled :max="item.name === '增值评分'?3:5"></el-rate>
  <el-rate v-model="subItem.value" disabled :max="item.name === '增值评分'?3:5" :icon-classes="iconClasses" void-icon-class="icon-rate-face-off" :colors="colors">
  </el-rate>

  // javascript
  name: 'check-rate',
  props: ['rate', 'mark'],
  data () {
    return {
      iconClasses: ['icon iconfont iconstar','icon iconfont iconstar','icon iconfont iconstar'],
      colors:['#FFC300','#FFC300','#FFC300']
    };
  }
```

<font face="STCAIYUN" size="3">antd 的改造很顺利就完成了，但是element 的星星一直不出来，遇到的问题就是iconfont icon 引入后，没有展示出来：</font>

<font face="STCAIYUN" size="3">主要是在iconfont 的使用上，先进行引入，如下图，将css js 文件分别在模板html 中进行引入</font>


![](/images/iconfont.png)


<font face="STCAIYUN" size="3">注意的点是iconClasses 这个里面填入的name 不要是在iconfont 网站上复制得到的，这是显示不出来的,它得到的是：iconstar  ，与at.alicnd.com 引用的对应有欠缺，所以导致显示不出来，它里面实际对应的是 icon iconfont iconstar ，类型+name 的一个组合</font>

![](/images/star.png)

![](/images/name.png)

<font face="STCAIYUN" size="3">对比方式是在iconfont 网站上点击下载本地，打开demo_index.html 后就可以看到span 标签里面的class 就是实际可显示的。</font>


