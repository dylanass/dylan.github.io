---
title: hexo写作指令
date: 2021-07-19 15:23:00
tags: 
    - hexo
---

#### 1.创建一个 md 文件

```bash
$ hexo new <title>
$ hexo new "我的页面"
```

#### 2.布局（layout）

```bash
$ hexo new [layout] <title>
$ hexo new page "我的页面"
```

- 布局有三种：post（文章）、draft（草稿）、page（页面）

#### 3.草稿（draft）

- 我们可在启动服务器时加上 --draft 参数来查看草稿。

```bash
$ hexo server --draft
```

- 还可以在站点配置文件中把 render_drafts 参数设为 true 来预览草稿。
- 我们可以通过 publish 命令将草稿发布文章或者页面，它将会被移动到指定的文件夹。

```bash
$ hexo publish [layout] <title>
```

#### 4.Front-matter

```bash
---
title: Hello World # 标题就是我们上面创建的时候指定的名字
date: 2013/7/13 20:46:25 # 文件创建的时间
---
```

> 在 Typora 中我们在 md 文件的首行（必须是第一行）输入--- ，然后按回车就可以插入 Front-matter 了。

- Front-matter 预定义参数

```bash
  layout  布局  默认为true，如果你不想你的文章被处理，可以设置为false
  title  标题  标题会显示在最上方居中位置
  date  建立日期    如果不指定则为默认值-文件创建日期，可以自定义。
  update  更新日期  如果不指定则为默认值-文件修改后重新生成静态文件的日期。
  comments  是否开启文章的评论功能 默认值为true
  tags  标签（不适用于页面page布局）
  categoreies  分类（不适用于页面page布局）
  permalink  覆盖文章网址
  keywords  仅用于 meta 标签和 Open Graph 的关键词（不推荐使用）
```

##### 为文章添加分类与标签

```bash
categories:
    - 个人博客（第一层级）
    - Hexo博客（第二层级）
tags:
    - Hexo
    - 博客
```

##### 基本操作

- **清除缓存**：`hexo clean`
- **生成静态文件**：`hexo generate` 可简写为 `hexo g`
- **启动服务器**：`hexo server` 或者 `hexo s` 常用参数：-p（--port）重设端口
- **部署**：`hexo deploy`可简写为`hexo d`，用于将网站部署到服务器上。（暂时用不到，目前都是在本地，后面我们将博客托管到 GitHub Pages 或 Gitee Pages 时才会用到此命令）
  常用参数：-g（--generate），`hexo d -g`部署前预先生成静态文件，等同于 `hexo g -d`

**一般发布文章或者修改博客后需要这些操作**：清除缓存>生成静态文件>启动服务器，测试没问题后再部署。

```bash
// 我们可以写成一条命令
$ hexo clean && hexo g && hexo s
$ hexo d
```

> 更多细节请查看:[官方文档](https://hexo.bootcss.com/docs/)
