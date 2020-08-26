---
title: React Hooks
date: 2019-08-20 10:27:46
tags:
---

###### 1.什么是 Hooks

###### • React 一直都提倡使用函数组件，但是有时候需要使用 state 或者其他一些功能时，只能使用类组件，因为函数组件没有实例，没有生命周期函数，只有类组件才有
###### • Hooks 是 React 16.8 新增的特性，它可以让你在不编写 class 的情况下使用 state 以及其他的 React 特性
###### • 如果你在编写函数组件并意识到需要向其添加一些 state，以前的做法是必须将其它转化为 class。现在你可以直接在现有的函数组件中使用 Hooks 
###### • 凡是 use 开头的 React API  都是 Hooks


###### 2.Hooks 解决的问题
>1. 类组件的不足
###### • 状态逻辑难复用：在组件之间复用状态逻辑很难，可能要用到 render props （渲染属性）或者 HOC（高阶组件），但无论是渲染属性，还是高阶组件，都会在原先的组件外包裹一层父容器（一般都是 div 元素），导致层级冗余、趋向复杂难以维护
###### • 在生命周期函数中混杂不相干的逻辑（如：在 componentDidMount 中注册事件以及其他的逻辑，在 componentWillUnmount 中卸载事件，这样分散不集中的写法，很容易写出 bug ）
###### • 类组件中到处都是对状态的访问和处理，导致组件难以拆分成更小的组件
###### • this 指向问题：父组件给子组件传递函数时，必须绑定 this 
###### • react 中的组件四种绑定 this 方法的区别

```

class App extends React.Component<any, any> {
    handleClick2;

    constructor(props) {
        super(props);
        this.state = {
            num: 1,
            title: ' react study'
        };
        this.handleClick2 = this.handleClick1.bind(this);
    }

    handleClick1() {
        this.setState({
            num: this.state.num + 1,
        })
    }

    handleClick3 = () => {
        this.setState({
            num: this.state.num + 1,
        })
    };

    render() {
        return (<div>
            <h2>Ann, {this.state.num}</h2>
            <button onClick={this.handleClick2}>btn1</button>
            <button onClick={this.handleClick1.bind(this)}>btn2</button>
            <button onClick={() => this.handleClick1()}>btn3</button>
            <button onClick={this.handleClick3}>btn4</button>
        </div>)
    }
}
```

>子组件内部做了性能优化，如（React.PureComponent）
###### • 第一种是在构造函数中绑定 this：那么每次父组件刷新的时候，如果传递给子组件其他的 props 值不变，那么子组件就不会刷新；
###### • 第二种是在 render() 函数里面绑定 this：因为 bind 函数会返回一个新的函数，所以每次父组件刷新时，都会重新生成一个函数，即使父组件传递给子组件其他的 props 值不变，子组件每次都会刷新；
###### • 第三种是使用箭头函数：父组件刷新的时候，即使两个箭头函数的函数体是一样的，都会生成一个新的箭头函数，所以子组件每次都会刷新；
###### • 第四种是使用类的静态属性：原理和第一种方法差不多，比第一种更简洁

>综上所述，如果不注意的话，很容易写成第三种写法，导致性能上有所损耗。

>2. Hooks 优势
###### • 能优化类组件的三大问题
###### • 能在无需修改组件结构的情况下复用状态逻辑（自定义 Hooks ）
###### • 能将组件中相互关联的部分拆分成更小的函数（比如设置订阅或请求数据）
###### • 副作用的关注点分离：副作用指那些没有发生在数据向视图转换过程中的逻辑，如 ajax 请求、访问原生 dom 元素、本地持久化缓存、绑定/解绑事件、添加订阅、设置定时器、记录日志等。以往这些副作用都是写在类组件生命周期函数中的。而 useEffect 在全部渲染完毕后才会执行，useLayoutEffect 会在浏览器 layout 之后，painting 之前执行。