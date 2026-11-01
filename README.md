# 理砚集 · 个人博客

Hugo + [Stack](https://github.com/CaiJimmy/hugo-theme-stack) 搭建，发布在 GitHub Pages。

- 线上地址：https://adamsly05.github.io/blog/
- 源码在 `main` 分支，构建产物推到 `gh-pages` 分支

## 本地构建

```sh
hugo --minify
```

预览时注意 `baseURL`：站点挂在 `/blog/` 下，本地预览要用

```sh
hugo server -b http://localhost:1313/
```

## 图片来源

- `static/img/bg.jpg`、`road.jpg`、`desk.jpg`：来自 [Lorem Picsum](https://picsum.photos)（图片源为 Unsplash，遵循 Unsplash License，可免费使用）
- 文章封面 `cover.svg`：自绘 SVG
