---
title: TypeScript：类型编程
date: 2021-07-19 21:06:46
tags:
    - TypeScript
categories: 
	- TypeScript
---

#### Typescript 是什么？

- Typescript 是 JavaScript 的超集，两者是所属关系。
- Typescript 是 JavaScript 的增强，包含 JavaScript 的最新特性，非常适合创建大型项目
- Typescript 是静态语言与动态语言 JavaScript 不同，静态语言在编写代码的时候就能发现潜在的错误，编写代码时静态语言能识别到可能使用到的属性等，像 data 参数静态语言能直接读出里面的属性 x, y

#### 下载安装及使用

- 使用`npm install -g typescript` 下载即可
- `tsc -v` 查看版本号
- 建议安装 TSlint 插件规范代码

---

### 下面进入正题

#### 基础类型

1.布尔值

```bash
let isDone: boolean = false;
```

2.数字

```bash
let decLiteral: number = 6;
```

3.字符串

```bash
let name: string = "bob";

//还可以使用字符串模版
let sentence: string = `Hello, my name is ${ name }`
```

4.数组

```bash
let list: number[] = [1, 2, 3];
```

第二种方式是使用数组泛型，`Array<元素类型>`

```bash
let list: Array<number> = [1, 2, 3];
```

5.元组 Tuple

元组类型允许表示一个已知元素数量和类型的数组

```bash
let x: [string, number];

x = [ 'hello', 6] // OK
x = [ 10 , 'hello'] //Error
```

当访问一个已知索引的元素，会得到正确的类型：

```bash
console.log(x[0].substr(1)); // OK
console.log(x[1].substr(1)); // Error, 'number' does not have 'substr'
```

当访问一个越界的元素，会使用联合类型替代（已经定义的）：

```bash
x[3] = 'world'; // OK, 字符串可以赋值给(string | number)类型

console.log(x[5].toString()); // OK, 'string' 和 'number' 都有 toString

x[6] = true; // Error, 布尔不是(string | number)类型
```
