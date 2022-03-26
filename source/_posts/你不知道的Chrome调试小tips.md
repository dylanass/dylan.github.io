---
title: 你不知道的Chrome调试小tips
date: 2021-08-10 22:17:37

tags:
    - Chrome
categories: 
	- 浏览器
---

# 正文开始

### 1.$0

在 `Chrome` 的 `Elements` 面板中，`$0` 是我选择的当前 `html` 节点的引用。

so ，`$1` 就是我们上一次选择的节点的引用，`$2` 是在 `$1` 之前选的节点的引用，

<!-- more -->

以此类推，一直到 `$4`

```javascript
<h1>1111</h1>
<span>2222</span>

先点h1,再点span,然后执行
$1.appendChild($0)

->>>
<h1>
  1111
  <span>2222</span>
</h1>
```

---

### 2.`$` 和 `?`

- 如果你在 App 中没有定义 `$` ，例如 `jQuery` 。它在 `console` 中就是 `document.querySelector` 的别名。

- `?` 则是先执行了 `document.QuerySelectorAll` 并且返回了：一个节点的**数组** ，而不是一个 `Node list`

  > `Array.from(document.querySelectorAll('div')) === ?('div')`

---

### 3. `$_`

是对上次执行结果的引用

```javascript
> Math.random()
< 0.15546175976148358
> $_
< 0.15546175976148358
```

---

### 4. `$i`

安装 [Chrome 插件: Console Importer](https://chrome.google.com/webstore/detail/console-importer/hgajpakhafplebkdljleajgbpdmplhie/related) ，可以在 console 中引入一些 npm 库了。

直接运行例如 `$i('lodash')` 或者 `$i('moment')` 几秒钟之后，就可以获取到 `lodash / momentjs` 了。

> 实操结果，在百度的 `console` 可以，掘金的 `console` 就不可以。
