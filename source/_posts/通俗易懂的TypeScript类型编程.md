---
title: 通俗易懂的TypeScript类型编程
date: 2021-08-08 17:40:28
tags:
  - TypeScript
categories:
  - TypeScript
---

# TypeScript 的另一面：类型编程

> 正文包括：
>
> - 基础泛型
> - 索引类型 & 映射类型
> - 条件类型 & 分布式条件类型
> - infer 关键字
> - 类型守卫 is in 关键字
> - 内置工具类型机能与原理
> - 内置工具类型增强
> - 更多通用工具类型

接下来进入正文

<!-- more -->

## 泛型 Generic Type

直接上个例子

```bash
    function foo<T>(arg: T): T {
        return arg;
    }
```

很多初学者不理解 `T` 到底是什么意思，其实它表示的就是一种未知类型，它是入参与返回值的类型。

**解释**：上面例子表示未，函数 `foo` 可接受的参数类型为 `T` , `arg` 的类型为 `T` ,函数返回值的类型为 `T` 。

```bash
    foo<string>("dylan")

    const [count, setCount] = useState<number>(1)
```

另外，你可能曾经见过 `Array<number>`, `Map<string, ValueType>` 这样的使用方式，通常我们将上面例子中 `T` 这样的未赋值形式成为 **类型参数变量** 或者说 **泛型类型**，而将 `Array<number>` 这样已经实例化完毕的称为 **实际类型参数** 或者是 **参数化类型**。

当然也可以不指定，因为 TS 会自动推导出泛型的实际类型。

```bash
    ⚠️ 注意 TS 在 泛型在箭头函数下的书写

    const foo = <T>(arg: T) => arg

    如果在TSX中这么书写, <T>可能会被识别为JSX标签,因为需要显示告诉编译器

    const foo = <T extends {}>(arg: T) => arg
```

除了在函数中泛型也可以在类中使用：

```bash
class Foo<T, U> {
  constructor(public arg1: T, public arg2: U) {}

  public method(): T {
    return this.arg1;
  }
}
```

介绍完泛型的基础概念，接下来讲讲泛型的应用。

## 类型守卫、is in 关键字

- **is**

举个栗子

比如现在有个字段

```
  numOrStrProp: number | string;
```

使用时，想将这个联合类型缩小范围，比如精确到`string`,你可能会加个函数去做判断

```
  export const isString = (arg: unknown): boolean => typeof arg === "string";
```

然后使用这个方法

```
function useIt(numOrStr: number | string) {
  if (isString(numOrStr)) {
    console.log(numOrStr.length);
  }
}
```

会出现如下错误

```
类型“string | number”上不存在属性“length”。
  类型“number”上不存在属性“length”。
```

说明参数依然是联合类型，这个时候就该使用 is 关键字了：

```
export const isString = (arg: unknown): arg is string =>
  typeof arg === "string";
```

这只是以原始类型为成员的联合类型，再看一个简单的假值判断

```
export type Falsy = false | "" | 0 | null | undefined;

export const isFalsy = (val: unknown): val is Falsy => !val;
```

类似的，还有 `isPrimitive` 、`isFunction`这样的类型守卫。

- **in**

然后来看下 **in** 关键字

```
class A {
  public a() {}

  public useA() {
    return "A";
  }
}

class B {
  public b() {}

  public useB() {
    return "B";
  }
}

function useIt(arg: A | B): void {
  'a' in arg ? arg.useA() : arg.useB();
}
```

类似 `for...in` 中的 `in` 它能判断一个属性是否为一个对象所拥有，通过这种方式可以进行类型收窄。

- **字面量类型(literal types)**

然后要说的是字面量类型`(literal types)`，例如：

你的状态码可能是 `0/1/2`

```
你就可以写成 status: 0 | 1 | 2 , 而不是用一个number来表示
```

字面量类型包括 **字符串字面量**、**数字字面量**、**布尔值字面量**

```
mode: 'dev' | 'prod'

open: 'true' | 'none' | 'chrome'
```

布尔值字面量通常与其他字面量类型混用.

### 基于字段区分接口

即登录与未登录下的用户信息是完全不同的接口

```
interface ILogInUserProps {
  isLogin: boolean;
  name: string;
}

interface IUnLoginUserProps {
  isLogin: boolean;
  from: string;
}

type UserProps = ILogInUserProps | IUnLoginUserProps;

function getUserInfo(user: ILogInUserProps | IUnLoginUserProps): string {
  return 'name' in user ? user.name : user.from;
}
```

或者通过字面量类型:

```
interface ICommonUserProps {
  type: "common",
  accountLevel: string
}

interface IVIPUserProps {
  type: "vip";
  vipLevel: string;
}

type UserProps = ICommonUserProps | IVIPUserProps;

function getUserInfo(user: ICommonUserProps | IVIPUserProps): string {
  return user.type === "common" ? user.accountLevel : user.vipLevel;
}
```

## 索引类型与映射类型

- **索引类型**

先介绍一个语法 **索引类型查询** 语法 `keyof` 它会返回后面跟着的类型参数的键值组成的字面量联合类型，举个例子：

```
interface foo {
  a: number;
  b: string;
}

type A = keyof foo; // "a" | "b"
```

是不是就像 Object.keys() 一样？区别就在于它返回的是联合类型.

接下来，要深刻认识到类型编程也是编程，带着这种思想去阅读接下来的部分。

首先来实现一个简单的函数

```
// 假设key是obj键名
function pickSingleValue(obj, key) {
  return obj[key];
}
```

接下来定义类型:

1. 参数`obj`
2. 参数`key`
3. 返回值

```
function pickSingleValue<T>(obj: T, key: keyof T): T[keyof T] {
  return obj[key];
}
```

解释一下，首先定义 `obj` 为泛型 `T` ,`key` 也就是 `obj` 键值是 `keyof` 返回的联合类型，返回值自然就是 `T[keyof T]`。

这种写法明显又可以改进的地方：

1. `keyof` 出现了两次
2. `T` 应该被限制为对象类型

```
function pickSingleValue<T extends object, U extends keyof T>(
  obj: T,
  key: U
): T[U] {
  return obj[key];
}
```

用一个变量把多处出现的存起来，记得，**在类型编程里，泛型就是变量**。

再看一个概念 **索引签名** 在类型编程中，索引签名用于快速建立一个内部字段类型相同的接口.

```
interface Foo {
  [keys: string]: string;
}
```

那么接口 Foo 实际上等价于一个键值全部为 string 类型，不限制成员的接口。

> 等同于 `Record<string, string>`

- **映射类型 Mapped Types**

类似于数组 map 方法,在类型编程中我们会从一个类型的定义映射的一个新的类型定义,并在旧的基础上进行一些改造:

1. 修改原有接口的键值类型
2. 为原有接口的键值类型新增修饰符,例如 `readonly` 、 可选 `?`

从一个简单的场景入手

```
interface A {
  a: boolean;
  b: string;
  c: number;
  d: () => void;
}
```

需求: 实现一个接口跟 A 中的字段完全相同,但类型全部为 `string` 类型

相当于写一个 `utils` 里的工具,然后去用就完事儿了

```
type StringifyA<T> = {
  [K in keyof T]: string;
};

type B = StringifyA<A>

//相当于
type B = {
  a: string;
  b: string;
  c: string;
  d: string;
}
```

重要的就是这个 `in` 操作符, 你可以把它理解为 `for...in` / `for...of` 这种遍历思路

获取到键名之后，键值就简单了,再来写一个拷贝新的类型别名

```
type CloneA<T> = {
    [K in keyof T]:T[K]
}
```

`Partial` 来个常用的: 将接口下的字段全部变成可选的

```
type Partial<T> = {
    [key in keyof T]?:T[k]
}
```

> key?: value 意为这一字段是可选的，在大部分情况下等同于 key: value | undefined

## 条件类型 Conditional Types

在编程中的条件判断我们常用 `if` 语句与 `三元表达式` 实现:

```
if(condition){
    execute()
}
//or
condition ? execute() : void 0
```

而 条件类型 的语法，实际上就是三元表达式，看一个最简单的例子:

```
T extends U ? X : Y
```

> 理解为 T 继承 U ,也就是说, T 中包含 U 中所有属性
> 也就是说 T 中必须包含 U 中的所有属性,则是 true ,反之就是 false

为了更好理解上面的例子

```
type A = {a:number}
type B = {a:number , b:number}
type C = {a:number , b:number , c:number}

type R1 = A extend B ? true : false // false
type R2 = C extend B ? true : false // true
```

然后我们先来看一个概念叫: **泛型约束**

首先我们要知道,泛型本身是来者不拒的,所有类型都能被**显示传入**(如`Array<number>`)和**隐式推导**(如`foo(1)`)

这个例子

```
T extends object 与 U extends keyof T
```

都是泛型约束,分别将 **T 约束为对象类型** 和 将 **U 约束为 T 键名的字面量联合类型**

所以在 TS 中我们通过泛型约束,要求传入的泛型只能是固定类型

例如 `T extends {}` 被约束为对象类型 和 `T extend string | number` 被约束为字符串和数字类型

看一个以条件类型做为返回值类型的例子

```
declare function strOrNum<T extends boolean>(
  x: T
): T extends true ? string : number;


const strReturnType = strOrNum(true);  //string
const numReturnType = strOrNum(false);  //number
```

在这种情况下，条件类型的推导就会被延迟，因为此时类型系统没有足够的信息来完成判断。
只有给出了所需信息（在这里是入参 x 的类型），才可以完成推导。

同样的，就像三元表达式可以嵌套，条件类型也可以嵌套，如果你看过一些框架源码，也会发现其中存在着许多嵌套的条件类型，无他，条件类型可以将类型约束收拢到非常窄的范围内，提供精确的条件类型，如：

```
type TypeName<T> = T extends string
  ? "string"
  : T extends number
  ? "number"
  : T extends boolean
  ? "boolean"
  : T extends undefined
  ? "undefined"
  : T extends Function
  ? "function"
  : "object";
```

- **分布式条件类型 Distributive Conditional Types**

分布式条件类型实际上不是一种特殊的条件类型，而是其特性之一（所以说条件类型的分布式特性更为准确）。我们直接先上概念： 对于属于裸类型参数的检查类型，条件类型会在实例化时期自动分发到联合类型上。

> 原文: Conditional types in which the checked type is a naked type parameter are called distributive conditional types. Distributive conditional types are automatically distributed over union types during instantiation

先提取几个关键词，然后我们再通过例子理清这个概念：

1. **裸类型参数（类型参数即泛型，见文章开头的泛型章节介绍**
2. **实例化**
3. **分发到联合类型**

使用上面的 `TypeName` 举几个例子

```
type T1 = TypeName<string | (() => void)>;  // string | function

type T2 = TypeName<string | string[]>;  // string | object

type T3 = TypeName<string[] | number[]>; // object
```

上面的例子推导结果都是联合类型, 实际上就依次推导结果用 | 连起来.

T3 也是,只不过两次结果相同,合并了.

上面的例子都是泛型裸露的情况,如果被包裹呢?

再看另外一个例子:

```
type A<T> = T extends boolean ? 'Y' : 'N'
type WrapperA<T> = [T] extends [boolean] ? 'Y' : 'N'


type B = A<boolean | number>                 //'Y' | 'N'
type WrapperB = WrapperA<boolean | number>   //'N'
```

第二种情况 `WrapperB` 是没有进行分发的过程的,会直接进行 `[boolean | number] extend [boolean]` 的判断,所以结果是 `'N'`
**(这块的结果可以理解成:第一种情况裸露泛型就会进行类型分发是 `|` ;第二种情况不会进行 l 类型分发是 `&&`)**

现在来理解下 上面的三个概念

1. 裸类型参数:实际上就是没有`[]`包裹过的
2. 实例化:其实就是条件类型的判断过程，就像我们前面说的，条件类型需要在收集到足够的推断信息之后才能进行这个过程。在这里两个例子的实例化过程实际上是不同的，具体会在下一点中介绍
3. 分发到联合类型: `typeB = A<boolean> | A<number>` 实际上就是这个的结果

一句话概括：**没有被 [] 额外包装的联合类型参数，在条件类型进行判定时会将联合类型分发，分别进行判断。**

> 这两种行为没有好坏之分，区别只在于是否进行联合类型的分发，如果你需要走分布式条件类型，那么注意保持你的类型参数为裸类型参数。如果你想避免这种行为，那么使用 `[]` 包裹你的类型参数即可（注意在 extends 关键字的两侧都需要)。

## infer 关键字

## 工具类型 Tool Type

这一章应该是本文“性价比”最高的一部分了，因为即使你在阅读完这部分后，还是不太懂这些工具类型是如何实现的，也不影响你把它用的恰到好处，就像 Lodash 不会要求你对每个使用的函数都熟知原理一样。

- **内置工具类型**

1. `Partial` 它用于将一个接口中的字段全部变为可选

```
//可选 (内置)
type Partial<T> = {
  [K in keyof T]?: T[k];
};

//必填 (不属于内置)
type Required<T> = {
  [K in keyof T]-?: T[K];
};

//只读 (不属于内置)
type Readonly<T> = {
  readonly [K in keyof T]: T[K];
};
```

2. `Pick` 从一个接口中挑选一些字段

```
type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};

// type Part = Pick<A, "a" | "b">
// 从 A 中挑出字段是 'a' 和 'b'
```

既然有了 `Pick` 就要有 `Omit` : 移除传入的键值

在看 `Omit` 之前要知道一个类型叫 `never`: 表示永远不会出现的类型 (通常被用来收窄联合类型或接口)

3. `Exclude` 排除

```
type Exclude<T, U> = T extends U ? never : T;

//原理就是 T 中包含 U吗? 包含就剔除,赋值 never

type LeftFields = Exclude<"1" | "2" | "3" | "4" | "5", "1" | "2">;  // "3" | "4" | "5"
```

4. `Omit` 从一个对象中排除部分

```
type Omit<T,K extends keyof any> = Pick<T, Exclude<keyof T, K>>

// 原理就是 挑选出 从 T 的键值中 排除掉 K的
```

- `Exclude` 用在排除联合类型
- `Omit` 用来排出对象中的键值

5. `Record` 生成一个新接口 第一个参数做为键 , 第二个参数作为每个键的值

先写源码:

```
// K extends keyof any 约束K必须为联合类型

type Record<K extends keyof any , T> = {
  [P in K] : T
}
```

使用例子:

```
type Key = 'a' | 'b' | 'c'

interface Value {
  widgets: string[];
  title?: string;
  keepAlive?: boolean;
}

type routerProps = Record<Key, Value>;

相当于...
// type routerProps = {
//    a: Value;
//    b: Value;
// }

const router: Record<Key, Value> = {
  a: { widgets: [""] , title:'123' , keepAlive:true},
  b: { widgets: [""] , title:'456'},
  c: { widgets: [""] },
};
```

- **社区工具类型**
- **递归的工具类型**
- **返回键名的工具类型**
- **基于值类型的 Pick 与 Omit**
- **工具类型一览**

## TypeScript 4.x 中的部分新特性

- **模板字面量类型**
- **重映射**

## 结束语
