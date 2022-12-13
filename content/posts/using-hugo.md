---
title: "Using Hugo"
date: 2022-12-14T00:32:47+08:00
draft: true
---
# 重启 Blog

无意中看到了自己的 GitHub 上的 blog source，发现上一次折腾博客已经是 5 年多以前了。
我已经完全忘记了之前的博客是怎么搭建、域名了。这一次再重新尝试用 Hugo + GitHub pages 写点东西吧。

## 迁移问题

### 主题
PaperMod 非常对我的胃口。

### 添加 CNAME
创建一个 `static/CNAME` 文件，Hugo 会自动将其复制到 `public` 目录下。

### 添加评论系统
使用基于 GitHub Issue 的 [utteranc.es](https://utteranc.es/) 评论系统。
1. 在 Repo 中安装 https://github.com/apps/utterances
2. 创建 `layouts/partials/comments.html` 文件，内容如下：
```html
<script src="https://utteranc.es/client.js"
        repo="j178/blog"
        issue-term="title"
        label="blog"
        theme="github-light"
        crossorigin="anonymous"
        async>
</script>
```
2. 在 `config.toml` 中添加如下配置：
```toml
[params]
  comments = true
```

### Hexo tags 不规范
### 文件名格式
