---
title: 【深一丢丢】let和const的理解补充
date: 2021-12-13 14:15:01
tags: [ES6, 笔记]
categories: 
    - [深一丢丢]
---

## 前言
最近在掘金上看到很多人在作用域上的理解。但是发现每个人描述的都有些区别并且和自己理解中的有些不同。
在查找相关资料后，整理了一下let和const的理解补充

## let，const到底有没有声明变量提升

**结论：** 有

## 说明

#### 执行上下文

首先需要理解执行上下文，词法环境，变量环境的抽象概念。（此文章不做说明）  

**重点：**在 ES6 中，**词法环境**和**变量环境**的一个不同就是前者被用来**存储函数声明和变量（let 和 const）绑定**，而后者只用来**存储 var 变量绑定**。
```
let a = 20;
const b = 30;
var c;

function multiply(e, f) {
 var g = 20;
 return e * f * g;
}

c = multiply(20, 30);

```
执行上下文看起来像这样：
```
GlobalExectionContext = {

  ThisBinding: <Global Object>,

  LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Object",
      // 在这里绑定标识符
      a: < uninitialized >,
      b: < uninitialized >,
      multiply: < func >
    }
    outer: <null>
  },

  VariableEnvironment: {
    EnvironmentRecord: {
      Type: "Object",
      // 在这里绑定标识符
      c: undefined,
    }
    outer: <null>
  }
}

FunctionExectionContext = {
  ThisBinding: <Global Object>,

  LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      // 在这里绑定标识符
      Arguments: {0: 20, 1: 30, length: 2},
    },
    outer: <GlobalLexicalEnvironment>
  },

VariableEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      // 在这里绑定标识符
      g: undefined
    },
    outer: <GlobalLexicalEnvironment>
  }
}

```
<code>let</code>  和 <code>const</code> 定义的变量并没有关联任何值，但 <code>var</code> 定义的变量被设成了 <code>undefined</code>。  

这是因为在创建阶段时，引擎检查代码找出变量和函数声明，虽然函数声明完全存储在环境中，但是变量最初设置为<code> undefined（var 情况下）</code>，或者<code>未初始化（let 和 const 情况下）</code>。  

这就是为什么可以在声明之前访问 <code>var</code> 定义的变量（虽然是 undefined），但是在声明之前访问 <code>let</code> 和 <code>const</code> 的变量会得到一个引用错误。  

这就是变量声明提升。

#### 总结
一个变量有三个操作：
1. <code>声明(提到作用域顶部)</code>
2. <code>初始化(赋默认值)</code>
3. <code>赋值(继续赋值)</code>。   

**var**：一开始变量声明提升，然后初始化成undefined，代码执行到那行的时候赋值。   
**let**：一开始变量声明提升，然后没有初始化分配内存，代码执行到那行初始化，之后对变量继续操作是赋值。**因为没有初始化分配内存，所以会报错，这是暂时性死区。**   
**const：**只有声明和初始化，没有赋值操作，所以不可变。