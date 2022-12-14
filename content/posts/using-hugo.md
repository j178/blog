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

### 自定义域名
1. 创建一个 `static/CNAME` 文件，Hugo 会自动将其复制到 `public` 目录下。
2. 在 settings - pages 中设置 custom domain
3. 将 cloudflare SSL 设置为 Full，不然会出现无限 302 的问题。因为 cloudflare 尝试用 http 连接 github pages，被 github pages 重定向到 https。
4. 等待 GitHub 为你的自定义域名申请 Let's Encrypt 证书，[这个过程可能需要几个小时](https://docs.github.com/en/pages/getting-started-with-github-pages/securing-your-github-pages-site-with-https)。

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
3. 参考 PaperMod 的[配置文档](https://github.com/adityatelange/hugo-PaperMod/wiki/Features#comments)，在 `config.toml` 中添加如下配置：
```toml
[params]
  comments = true
```

### Hexo tags 不规范
### 文件名格式
