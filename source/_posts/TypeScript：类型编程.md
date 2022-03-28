---
title: TypeScript：类型编程(基础类型)
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

<!-- more -->

#### 下载安装及使用

- 使用`npm install -g typescript` 下载即可
- `tsc -v` 查看版本号
- 建议安装 TSlint 插件规范代码

---

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

6.枚举

是一种数据结构

```bash
    //第一个字母大写
    enum Color{
        Red,  //0
        Blue, //1
        Black //2
    }
    //默认从0开始
    let color : Color
    color = Color.Blue  // 1
```

再举个栗子

```bash
    // 对比常用的 JS 代码
    // const Status = {
    //     OFFLINE : 0,
    //     ONLINE: 1,
    //     OTHERS: 2
    // }

    // 下面代码直接打印出 enum 的下标值
    console.log(Status.OFFLINE)  //0
    console.log(Status.ONLINE)   //1
    console.log(Status.OTHERS)   //2

    // 打印下标对应的属性
    console.log(Status[0])
    console.log(Status[1])
    console.log(Status[2])
```

7.任意值

```bash
    //对数组内的类型不确定时
    let list: any[] = [1, true, "free"];
```

或者我们在编程阶段还不清楚类型的变量。 它们可能来源于 **动态**的内容。

- 例如用户输入
- 第三方代码库

这种情况下，我们可以使用 `any` 来标记这些变量

8.空值

当一个函数没有返回值的时候，通常是 `void`

```bash
    //在函数中的写法
    function warnUser(): void {
        alert("This is my warning message");
    }

    //在接口中的写法
    interface Props{
        callback: ()=> void
    }
```

9.Null 和 Undefined

- 它们本身类型用处不大

- 默认情况`null`和`undefined`是所有类型的子类型，就是说，你可以把`null`和`undefined`赋值给`number`类型

- 然而，当你指定了 `--strictNullChecks` 标记，`null` 和 `undefined` 只能赋值给 `void`

- 你也可以使用联合类型 `string | null | undefined`

> 注意：我们鼓励尽可能地使用--strictNullChecks，但在本手册里我们假设这个标记是关闭的。

10.Never

- 表示那些永远不存在的值的类型

- `never` 类型是任何类型的子类

举个具体点的栗子，当你有个 union type

```bash
    interface Foo {
        type: 'foo'
    }

    interface Bar {
        type: 'bar'
    }

    type All = Foo | Bar
```

在 switch 当中判断 type，TS 是可以收窄类型的 (discriminated union)：

```bash
    function handleValue(val: All) {
        switch (val.type) {
            case 'foo':
            // 这里 val 被收窄为 Foo
            break
            case 'bar':
            // val 在这里是 Bar
            break
            default:
            // val 在这里是 never
            const exhaustiveCheck: never = val
            break
        }
    }
```

注意在 default 里面我们把被收窄为 never 的 val 赋值给一个显式声明为 never 的变量。如果一切逻辑正确，那么这里应该能够编译通过。但是假如后来有一天你的同事改了 All 的类型：

```bash
    type All = Foo | Bar | Baz
```

这里就会编译报错，通过这个办法就可以穷尽所有 All 的类型

详细可以看[尤大的知乎回答](https://www.zhihu.com/search?type=content&q=ts%20never)

11.类型断言

告诉编译器，"相信我，我知道自己在干什么"。

类型断言的两个形式

- 尖括号语法

```bash
    let someValue: any = "this is a string";

    let strLength: number = (<string>someValue).length;
```

- `as` 语法

```bash
    let someValue: any = "this is a string";

    let strLength: number = (someValue as string).length;
```

两种形式是等价的。

> 注意：当你在 TypeScript 里使用 JSX 时，只有 `as` 语法断言是被允许的。

---

更多介绍请访问 [TypeScript 中文手册](https://typescript.bootcss.com/variable-declarations.html)
