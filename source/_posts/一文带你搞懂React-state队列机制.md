---
title: 一文带你搞懂React state队列机制
date: 2021-08-13 22:01:49
tags:
---


# 背景：

> 前提摘要：很多面试官都会问React中的 `setState` 到底异步还是同步？
>
> 那么他到底是同步还是异步呢？
>
> setState的不同现象大致分为两种情况
>
> - React 事件系统中的 合成事件 （例如: `onClick`  , `onChange` ）和 React的生命周期中，它是''异步''的，注意我说的是带引号的异步。
> - 在原生事件，或者 `setTimeout` 中(基于 `event loop` 模型) ，可以理解为setState是同步的，因为你能拿到最新的state。
> 

<!-- more -->

## 我们先来说说第一种情况

通过代码来印证

```javascript
// 为了方便同学们验证这里展示完整代码，文章末尾也会附上codesandbox链接

import React from "react";

class Home extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
  }

  componentDidMount() {
    
    this.setState({count: this.state.count + 1});
    
    console.log("步骤1>>>>" + this.state.count);    

    this.setState({ count: this.state.count + 1 }, () => {
      console.log("步骤3>>>>" + this.state.count); 
    });

    this.setState(
      (prevState) => {
        console.log("步骤2>>>>" + prevState.count); 
        return {
          count: prevState.count + 1
        };
      },
      () => {
        console.log("步骤4>>>>" + this.state.count); 
      }
    );
  }

  render() {
    return <div></div>;
  }
}

export default Home;

```



>在console上的执行顺序是 
>
>步骤1>>>>0 
>
>步骤2>>>>1 
>
>步骤3>>>>2 
>
>步骤4>>>>2
>
>注意我写的步骤顺序



如果跟你想的有差异，来解释一波（以下思路纯个人理解）

首先，React在每次渲染之前会维护一个state队列，将每次setState的值进行合并，最后进行一次渲染。

我们把代码拿过来：

```javascript
// 没执行 componentDidMount 之前，我们的 state 虚拟队列是 [0]

componentDidMount() {
    
    this.setState({count: this.state.count + 1});  //到这虚拟队列是 [0 , 1]
    
    console.log("步骤1>>>>" + this.state.count);    //当前组件state的值: 0 

    this.setState({ count: this.state.count + 1 }, () => {  //第二个回调，我理解为类似 微任务队列：[步骤3] 待执行
      console.log("步骤3>>>>" + this.state.count); 
    });

    this.setState(
      (prevState) => {
        console.log("步骤2>>>>" + prevState.count);  //第一个回调preState拿最新的值，毋庸置疑，所以是: 1
        return {
          count: prevState.count + 1 // 到这里虚拟队列是 [0 , 1 , 2]
        };
      },
      () => {
        console.log("步骤4>>>>" + this.state.count); //这里也类似 微任务队列：[步骤3 ， 步骤4] 待执行
      }
    );
    
  	//开始执行 [步骤3 ， 步骤4] 
  	// 步骤3 >>>> 2
    // 步骤4 >>>> 2
    // 因为理解为微任务队列 理解为'异步'，所以拿到最新的值
 }
```



有没有清晰一点，如果没有那等会再看，先来看看第二种情况



## 第二种情况

也就是setTimeout和原生事件，这里就不展示原生事件了。

先来看一波简单直接的代码:

```javascript
//还是计算 虚拟队列帮助理解 初始值还是 0 
 
// 虚拟队列[0]
componentDidMount() {

    // 生命周期中调用
    this.setState({ count: this.state.count + 1 });  // 虚拟队列[0,1]
    console.log("生命周期中", this.state.count);       // 拿当前的state : 0 
    
    setTimeout(() => {
      // setTimeout中调用
      console.log("setTimeout1", this.state.count);    // 因为在setTimeout中，异步，所以拿到最新的state : 1
      this.setState({ count: this.state.count + 1 });  // 这里为 1+1 虚拟队列[0,1,2]
      this.setState({ count: this.state.count + 1 });  // 这里为 2+1 虚拟队列[0,1,2,3]
      this.setState({ count: this.state.count + 1 });  // 这里为 3+1 虚拟队列[0,1,2,3,4]
      this.setState({ count: this.state.count + 1 });  // 这里为 4+1 虚拟队列[0,1,2,3,4,5]
      this.setState({ count: this.state.count + 1 });  // 这里为 5+1 虚拟队列[0,1,2,3,4,5,6]
      console.log("setTimeout2:", this.state.count);   // 6
    }, 0);
 }


//有没有豁然开朗的感觉！

```



本文代码 ：[codesandbox](https://codesandbox.io/s/crazy-varahamihira-oxh7f?file=/src/Home.jsx)

>如果你还想知道React内部实现原理，可以精读这篇文章 [你真的理解setState吗?](https://juejin.cn/post/6844903636749778958)
>
>我个人感觉讲解非常清晰了。
>
>感谢阅读。













