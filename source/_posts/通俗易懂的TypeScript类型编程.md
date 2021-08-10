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

当然也可以不指定，因为 TS 会自动推导出泛型的实际类型。

```bash
    ⚠️ 注意 TS 在 泛型在箭头函数下的书写

    const foo = <T>(arg: T) => arg

    如果在TSX中这么书写, <T>可能会被识别为JSX标签,因为需要显示告诉编译器

    const foo = <T extends {}>(arg: T) => arg
```

> 未完待续。。。
