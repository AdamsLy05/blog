---
title: 在一部 Android 手机上跑 Hugo 和 Git
slug: hugo-on-android
image: "cover.svg"
date: 2026-11-01T15:00:00+08:00
description: 这个博客是怎么在一部手机上构建、发布、上线 GitHub Pages 的。
categories:
  - 手机
tags:
  - Hugo
  - GitHub
  - 博客
series:
  - from-phone-to-server
---

这个博客没有用任何"博客平台"。它是一堆静态文件，在一部手机上生成，推到一个 GitHub 仓库，由 GitHub Pages 提供服务。

整个过程比想象的简单，坑也比想象的多。

---

## 装 Hugo

Hugo 是单个二进制文件，arm64 版本直接下载解压就能用：

```sh
curl -LO https://github.com/gohugoio/hugo/releases/download/v0.169.0/hugo_extended_0.169.0_linux-arm64.tar.gz
tar xzf hugo_extended_0.169.0_linux-arm64.tar.gz hugo
chmod +x hugo
./hugo version
```

注意要 **extended** 版本——很多主题用 SCSS，普通版编译会报错。

构建速度：这个站 20 个页面，**不到 1 秒**。

---

## 授权 GitHub

不想把 token 交出去，所以用 `gh` 的设备码登录：

```sh
gh auth login --web
```

它给一个 8 位验证码和网址，在手机浏览器里输入、确认，凭据就存在本机了。之后再配一次 `gh auth setup-git`，`git push` 就能直接走。

---

## 发布

Hugo 生成的静态文件推到 `gh-pages` 分支，在仓库设置里把 Pages 指向它，就上线了。

```sh
hugo --minify
cd public && git push -f origin gh-pages
```

也可以写 GitHub Actions 自动做，但那需要给授权额外的 `workflow` 权限。我图省事，直接推构建结果。

---

## 三个把我坑住的细节

**一、`baseURL` 和子路径。**

站点地址是 `https://用户名.github.io/blog/`，不是在根目录。如果 `baseURL` 配错，生成的 CSS 路径会指向 `/scss/...`，页面加载出来的样子**完全没有样式**——你会以为是主题不适配手机，其实是样式表 404。

本地预览更要注意：用 `hugo -b http://127.0.0.1:8000/` 临时覆盖，别用生产地址。

**二、图片路径同理。**

写 `image: "/img/cover.jpg"`，上线后就是 404。正确做法是把图片放进文章的目录（page bundle），用相对路径 `cover.jpg`，Hugo 会自动给出带子路径的正确地址。

**三、浏览器缓存会骗你。**

改了 CSS，手机上一看没变化，很可能是缓存。用 `?v=2` 这样的查询参数强制刷新再判断。

---

## 一句话结论

**手机不缺能力，缺的是顺手。**

构建、发布、授权这些"一次性"的事，手机都能干，而且不慢。真正折磨人的是每天都要做的部分：改样式、看效果、反复微调——那需要一块大屏。

但如果你只是想"有个地方写东西"，一部手机确实够了。
