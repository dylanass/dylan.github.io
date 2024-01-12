---
title: 你在项目里是如何使用 Lodash ？
date: 2024-01-12 15:04:15
tags: lodash
---

## What
> Lodash 使用频率较高的 JavaScript 工具库
## Why 
> 它提供了很多工具函数，操作数组，数字，对象，字符串等等，还有比如深、浅拷贝，防抖，节流，空值校验等等，帮我们节约了很多开发时间。
> 但是，Lodash 一直有一个令人诟病的问题：如果我们稍不注意，在项目中直接引入 Lodash ，会导致体积变得 巨大无比 ，甚至构建产物中，有些代码始终都没有用到。 体积问题对于中后台应用还好，如果是移动端应用，尤其在弱网的条件下，可能导致页面性能变得非常差。
> 那我们该如何使用呢？

<!-- more -->

## How
> Lodash 官网 提供了多种模块方式给开发者选择使用 里面也有能让 Lodash 体积减小的方式
![](https://raw.githubusercontent.com/dylanass/blog-image/main/1.png?10)
接下来这些接入方式究竟哪种是最好的：
> lodash/fp 和 lodash-amd 使用频率不高，并且与lodash 模块对项目体积影响类似，所以在这就不做对比了

### 前置准备
随便准备一段代码使用：merge 和 mergeWith 两个常用函数，并且使用 Webpack 构建工具打包，最后借助 [Webpack Bundle Analyzer](https://www.npmjs.com/package/webpack-bundle-analyzer?spm=ata.21736010.0.0.190f51dcWg6axf) 对最终产物体积进行分析。

**示例代码（lodash官网栗子）** 
```
const object = {
  'a': [{ 'b': 2 }, { 'd': 4 }]
};

const other = {
  'a': [{ 'c': 3 }, { 'e': 5 }]
};

function customizer(objValue, srcValue) {
  if (Array.isArray(objValue)) {
    return objValue.concat(srcValue);
  }
}

console.log(merge(object, other));
console.log(mergeWith(object, other, customizer));
```

### Lodash 使用方式对比
#### 1.Lodash 
首先我们使用最原始的方式：在源码中直接倒入 Lodash 模块
```
import _ from 'lodash'
const merge = _.merge
cosnt mergeWith = _.mergeWith
```
然后我们使用 webpack-bundler-analyzer  分析最终打进去的 bundle 中的体积
发现整个 lodash 压缩后的体积有24.1KB：
![](https://raw.githubusercontent.com/dylanass/blog-image/main/2.png)


大家都知道 Webpack 提供了 Tree Shaking 把没用的工具函数都移除掉，但是为啥，我们只用了两个，体积会这么大？
因为 lodash 源码使用的 CommonJS 模块规范（一部分源码：）
![](https://raw.githubusercontent.com/dylanass/blog-image/main/3.png)

而 Tree Shaking 生效的前提是 ES Modules

#### 2.Lodash 独立方法包：
```
import merge from 'lodash.merge'
import mergeWith from 'lodash.mergeWith'
```
执行构建：
![](https://raw.githubusercontent.com/dylanass/blog-image/main/4.png)
可以发现两个模块加起来 Gzipped 后的体积是接近6.78k了，虽然减少了 17.32（70%），但这里还有个问题是：merge 和 mergeWith 都引用了相同的模块：
![](https://raw.githubusercontent.com/dylanass/blog-image/main/5.png)
由于 lodash.merge 和 lodash.mergewith也都是 CommonJS 模块，发布 npm 包之前是分别以 merge.js 和 mergeWith.js文件为入口单独打包出产物，导致无法通过 Tree Shaking 移除掉相同重复的模块。
简单来说就是：`_baseMerge` 和 `_createAssigner` 被打进去两次了。
> ⚠️ 这种方式官方将会在 lodash V5 版本中废弃
#### 3.babel-plugin-lodash
Lodash 官方推出了一个 babel 插件：babel-plugin-lodash，它可以将我们的源码进行 transfrom 操作，
![](https://github.com/dylanass/blog-image/blob/main/6.png)

可以发现 Gzipped 后的体积是 4.1K，非常可观了，相比于全量导入 lodash 模块的方式体积减小了 20K（83%）。但是这种方式需要 额外配置 babel-loader 和 babel plugin。
```
 options: {plugins: ['lodash'],presets: ['@babel/preset-env']}
```
#### 4.lodash 子模块
Lodash 官方还提供了一种方式，用过 sub-path 的方式引入对应的 Lodash 子模块，比如：
```
import merge from 'lodash/merge'
import mergeWith from 'lodash/mergewith'
```
> 这种方式与上面使用的 babel-plugin-lodash 按需引入是类似的，只不过，这个按需引入是我们自己处理而已。体积也是 4.1K。
#### 5.lodash-es
官方提供的一个 ES Module 版本的 lodash：lodash-es
import {merge,mergeWith} from 'lodash-es';
既然是 ES Module ,那这样 Tree Shaking 就可以生效了，其他没用的代码就可以移除掉了：
![](https://raw.githubusercontent.com/dylanass/blog-image/main/7.png)

可以看到，Gzipped 后的代码体积是 3.76K，相比于全量导入 lodash CommonJS 格式的产物，减小了 20.34K（85%）。
#### 6.lodash-webpack-plugin
最后，官方还提供了一个 lodash-webpack-plugin 配合上面的 babel-plugin-lodash 能减小更多的体积，看下打包后的体积，Gzipped 后的体积只有 2.27K 了，体积减小了 21.76K（90%）
![](https://github.com/dylanass/blog-image/blob/main/8.png)


> 注意⚠️
但是查看lodash-webpack-plugin 文档我们可以发现是，lodash-webpack-plugin 会把一些函数用体积较小的函数代替（比如使用 [_nativeKeysIn]()代替 [keysIn](https://www.npmjs.com/package/lodash?activeTab=code) 函数）。虽然产物体积大大减小了，但是一些功能直接就失效，如果你的三方依赖使用了 lodash 并且使用了对应的功能，由于 lodash-webpack-plugin 替换了部分方法，导致三方依赖无法正常工作，出现意想不到的表现，排查问题的效率也会大大降低的，因此十分不推荐使用这种方式减小 Lodash 的体积。

总结

|  Lodash 使用方式   | 体积大小  | 备注 | 推荐指数| 
|        ----      | ----  |  ----  | ----  |  
| lodash  | 24.1K | 产物体积大，因为是 CommonJS 模块，无用模块无法 Tree Shaking | ⭐️ |
| lodash 独立方法包（如 lodash.merge）| 6.78K（↓70%）|	体积有所减少，但因为还是 CommonJS 模块，重复的模块也无法 Tree Shaking	| ⭐️⭐️ |
| lodash + babel-plugin-lodash |	4.1K（↓83%）|	体积减小了很多，不过要额外配置 babel-loader 和 babel plugin |	⭐️⭐️⭐️ |
|lodash 子模块（如：import merge from 'lodash/merge'）|	4.1K（↓83%）|	体积也减小了很多，但需要手动按需引入对应的子模块	|⭐️⭐️⭐️|
|loadsh-es	|3.76K（↓85%）|	产物是 ES Module，Tree Shaking 能生效，体积进一步减小，但一定程度上会影响构建速度	|⭐️⭐️⭐️|
|lodash-webpack-plugin|	2.26K（↓90%）|	虽然体积最小，但可能导致某些模块无法正常工作|	⭐️|

> 综上所述，如果在意产物体积或者性能，顺应前端社区使用 ESM 规范的趋势，建议使用 lodash-es；如果在意构建速度，建议使用 babel-plugin-lodash 或者 lodash 子模块的方式导入对应的工具函数。

[测试demo 仓库地址](https://github.com/dylanass/lodash-test)

FAQ
如何解决项目中已引用 lodash-es，但是三方依赖中引用了 lodash常规版本，导致两个包都打进了最终构建产物，导致不必要的代码冗余？

1. webpack alias 
2. babel-plugin-lodash 
3. 手动修改
4. PR
5. 接受现状？
