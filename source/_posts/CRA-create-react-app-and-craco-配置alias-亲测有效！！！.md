---
title: CRA(create-react-app) and craco 配置alias 亲测有效！！！
date: 2021-08-13 22:14:15
tags:
---

# CRA(create-react-app) and craco 配置alias 亲测有效！！！

> 本文跳过，项目初步搭建过程，默认你已经有了 `craco.config` ，如果想要完整的项目教程，请跳转到 [官方文档](https://ant.design/docs/react/use-with-create-react-app-cn#%E9%AB%98%E7%BA%A7%E9%85%8D%E7%BD%AE) 

## 直接上干货！

<!-- more -->

### `craco.config.js`

```javascript
//只写了跟alias有关的配置，防止干扰视线！！！
//craco.config.js 

const { resolve } = require("path");
const resolveToStaticPath = (relativePath) => resolve(__dirname, relativePath);

module.exports = {
    // ...
    webpack: {
        alias: {
            '@': resolveToStaticPath("./src"),
        }
    }
}
```

---

只配置 `craco.config.js` 是远远不够的，还要配置 `tsconfig.json` 

### `tsconfig.json`

```javascript
//同样省略其他配置,只写跟alias有关的配置

{
  "compilerOptions": {
    ...
    // 添加 Alias 支持
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  },
}

到这重启项目其实就可以了，如果你重启后不可以正常使用，请继续看我遇到的问题...
```

---

> 问题：在命令行执行 `yarn start` 后，刚刚在 `tsconfig.json` 写的配置神奇得消失了，不明所以，所以我单独写了一个 `tsconfig.path.json` 解决了这个问题...

### `tsconfig.path.json` 

```javascript
{
  "compilerOptions": {
    // 添加 Alias 支持
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}

然后引入进 tsconfig.json 就好了,像这样：

{
  "extends": "./tsconfig.path.json",
  "compilerOptions": {
    //...
  }
}

然后再重启项目。
```

---

到这里，你的项目基本可以启动没有问题了。

但是，你的编译器可能会出现一些路径信息报错问题。

so，最后一步（以vscode为例）。

### .vscode文件夹

```javascript
//settings.json

{
  "typescript.locale": "zh-CN"
}

至此，齐活！

```

---



> 更多答案可以参考 [Source](https://stackoverflow.com/questions/56387849/how-to-set-alias-path-via-webpack-in-cra-create-react-app-and-craco) 

---



>  `tsconfig` 消失问题后续：
>
>  CRA warns about `compilerOptions.baseUrl` and `compilerOptions.paths` not being supported but everything works as expected. 
>
>  这句话是在CRA issues 上找到的，意思就是CRA，不支持`baseUrl` 和 `paths` 
>
>  所以只能用 `tsconfig.json` 的扩展功能：
>
>  `"extends": "./tsconfig.paths.json", // or jsconfig.paths.json` 
>
>  [For more Information](https://github.com/facebook/create-react-app/issues/5118) 

---

>[参考资料](https://blog.csdn.net/chrislincp/article/details/97312235) 