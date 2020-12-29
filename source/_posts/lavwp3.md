---
title: Vue基于html2canvas实现HTML页面生成为图片并下载的功能及清晰度优化
tags:
  - CSS
categories:
  - - 前端
    - 前端笔记
description: 在Vue中基于html2canvas和Canvas2Image实现HTML页面生成为图片并下载的功能，并解决了一些文本丢失或者图片加载不全的问题，跨域配置，以及清晰度优化等等。
keywords: html2canvas，HTML页面生成为图片，下载，图片加载不全，文本丢失
top_img: 'https://cdn.jsdelivr.net/gh/constown/HexoStaticFile/img/20200717233539.jpg'
cover: 'https://cdn.jsdelivr.net/gh/constown/HexoStaticFile/img/20201113115021.png'
abbrlink: 558885cd
date: 2020-11-13 11:36:33
updated: 2020-11-13 11:36:33
---

## 前言

hexo 博客目前最常用的加速方案应该就是使用 `jsdelivr` + `github` 的方式了，免费还好用，我的博客的图片也基本上是使用了这种方式 。

关于如何使用 `jsdelivr` 来加速 `hexo` 博客我已经不想再赘述了。

如果你对此不太了解，你可以参考：[优雅使用 jsdeliver 加速文件](https://www.antmoe.com/posts/e33d1c55/index.html)

## 缓存问题

因为 CSS 样式，随着博客的更新，可能经常调整，所以我需要经常改动和发布后实时预览线上环境是否和我测试环境一致，所以我会选择 `完全忽略该版本或使用“最新”加载最新版本` 这样的方式

```
https://cdn.jsdelivr.net/gh/jquery/jquery@latest/dist/jquery.min.js
https://cdn.jsdelivr.net/gh/jquery/jquery/dist/jquery.min.js
```

我在尝试使用 `jsdelivr` 来加速我博客引入的自定义的 CSS 文件时，遇到了这样的问题：

- 第一次上传可以正常引入文件
- 随后我对 CSS 的样式再次进行了修改，上传到 github 仓库，引入失败（确切的说，引入的文件并没有更新，仍然是我第一次提交的内容）
- 此后，通过 `jsdelivr` 加速的文件，一直没有刷新成为我的最新版本。
- 这让我的博客无法立即更新线上环境的样式。

在使用这种方法的时候，我们使用的 `latest` 这个 tags，官网上说，在引用 CDN 的时候版本号可以省略，但经过尝试，发现不带版本号并不会指向正确的版本，甚至有些资源文件会报 404，因为这部分资源文件回滚以后发现还是没有，所以就会出现上面的情况，我最先想到的还是把这个版本号固定下来，这样就不用每次都去修改配置文件，这样又引入一个新问题，即：每次部署的时候都要先删除远程的 latest，这实在是让人有点烦躁。

## 解决办法

### 本地引入

以下教程只针对 `butterfly` 主题，其他主题使用对应的本地引入 css 文件的方式应该即可。

在 `source` 文件夹下新建 `style` 文件夹用于存放我们的样式(不推荐使用 `CSS` 作为文件名 )，然后在配置文件中使用本地引入的方式即可：

```yaml
inject:
  head:
    - <link rel="stylesheet" href="/style/xxx.css">
  bottom:
    # - <script src="xxxx"></script>
```

当然这种方法与我们的初衷似乎有些违背。

### 启用又拍云或者其他云储存服务

我目前使用的是 又拍云，把文件上传到 又拍云的 云储存 服务就好。实测是上传了可以实时刷新。

关于如何使用 云储存 服务，这里暂时先不赘述。

### jsdelivr 缓存

`jsdelive` 的缓存机制，我目前仍然不清楚，`jsdelivr` 官网是这么叙述的：

> We use a permanent S3 storage to ensure all files remain available even if GitHub goes down, or a repository or a release is deleted by its author. Files are fetched directly from GitHub only the first time, or when S3 goes down.
> 大概意思：我们使用永久性的 S3 存储，以确保即使 GitHub 发生故障，或者其作者删除了存储库或发行版，所有文件仍然可用。仅在第一次或 S3 故障时才直接从 GitHub 提取文件。

> If you use this feature and a file you requested is not available in the newest release, the link will keep working thanks to our version-fallback feature. We'll continue to serve the file from older release instead of failing with a 404 error.
> 大概意思：如果您使用此功能，但您所请求的文件在最新版本中不可用，则由于我们的版本回退功能，该链接将继续有效。我们将继续提供较旧版本的文件，而不是因为 404 错误而失败。

我想这大概率就是在 `push` 了代码后，但是 `jsdelivr` 加速的文件没有更新的原因。

当然 **第一种解决办法** 就是，你每改动一次代码，就 `releases` 一次，用更换版本号的形式来解决：

```
https://cdn.jsdelivr.net/gh/jquery/jquery@3.2.1/dist/jquery.min.js
```

但是或许还是有一些麻烦？因为你或许还要修改配置文件中对应的 CDN 地址。

我想，既然 `jsdelivr` 提供了一个 `完全忽略该版本或使用“最新”加载最新版本` 这样的引入方式，那应该就是支持 `实时刷新` 的，如果请求的文件一直未获得刷新，我个人觉得可能是因为 CDN 节点上的缓存资源并没有刷新，就和浏览器的缓存同一个道理，在我们本地调试的时候，修改了样式却没有变化的时候，我们总是想到的是清理一下浏览器的缓存。

如何清理 `jsdelivr` 缓存？

把链接地址中的 `cdn` 换成 `purge` 即可清除指定文件的缓存，（经过测试，这个方法也是有时候好用有时候不好用）

```
https://purge.jsdelivr.net/gh/jquery/jquery/dist/jquery.min.js
```

但是官方好像并没有提到这一点，我找了很久的文档，官方是这么说的：

> jsDelivr 具有易于使用的 API，可以从缓存中清除文件并强制更新文件。当您发布新版本并要强制更新所有版本别名用户时，此功能很有用。
> 为避免滥用，在发出电子邮件请求后（现在为[-dak@prospectone.io](mailto:dak@prospectone.io)）可以访问清除功能。

## jsDelivr API

查看 CDN 上的 tags 和 versisons 列表。

用法：

```
/package/npm/:name
 - name: npm package name

/package/gh/:user/:repo
 - user: GitHub username
 - repo: GitHub repository name
```

示例：

```
https://data.jsdelivr.com/v1/package/npm/jquery

//=>
{
    "tags": {
        "beta": "3.2.1",
        "latest": "3.2.1"
    },
    "versions": [
        "3.2.1",
        "3.2.0",
        "3.1.1",
        ...
    ]
}
```

更多的接口请直接访问：[jsDelivr API](https://github.com/jsdelivr/data.jsdelivr.com)

## 结语

以上，就是目前我能想到的应对 `jsdelivr` 缓存问题的所有方法了，总结如下：

- 直接使用本地引入（简单粗暴）
- 使用其他 云储存服务 来加速（可能会收费）
- 使用版本号的方式来引入（直接好用）
- `purge` 刷新缓存（似乎也不太稳定）
