---
title: "Migrate to Hugo"
date: 2022-12-14T00:32:47+08:00
draft: true
---
# 重启 Blog

无意中看到了自己的 GitHub 上的 blog source，发现上一次折腾博客已经是 5 年多以前了。
依稀记得之前是用 Hexo 本地构建出静态文件后，push 到 github pages 部署的。并且由 cloudflare 做 CDN 和 https 支持。
不过由于长时间没有维护，之前的域名已经不属于我了，而且 hexo 基于 node 这一套依赖太容易出问题，部署过程也很慢，所以这一次我决定重头开始用 Hugo + github pages 来部署博客。

5 年过去了，github pages 也做了很多改进，可以让我们的部署过程更加方便。比如说：
1. 由于有了 github actions，我们可以自定义 action 来自动 build 静态文件。
2. github pages 支持了从 github artifacts 中部署，所以我们也不再需要将静态文件保存在 `gh-pages` 分支中嘞。
3. github pages 也支持自动帮我们申请 let's encrypt 的 SSL 证书，所以自定义域名也不再需要用 cloudflare 的 CDN 了。

## 迁移问题

### 自定义域名
1. 创建一个 `static/CNAME` 文件，Hugo 会自动将其复制到 `public` 目录下。
2. 在 settings - pages 中设置 custom domain
3. 在 cloudflare 中添加 CNAME 解析，将自定义域名 CNAME 到 <your-name>.github.io，注意关闭 proxy，因为我们要使用 github pages 自带的证书。
4. 等待 GitHub 为你的自定义域名申请 Let's Encrypt 证书，[这个过程可能需要几个小时](https://docs.github.com/en/pages/getting-started-with-github-pages/securing-your-github-pages-site-with-https)。

cloudflare 开启了 proxied 之后的 CNAME 并不是普通的 CNAME，而是由 cloudflare 服务器接管了。以下是关闭 proxied 前后的 dig 的结果：
![image](https://user-images.githubusercontent.com/10510431/207522605-cb76812b-f69a-42f3-a7fd-c7930125baaf.png)
关闭 proxied 之后，github pages 页面可以正常申请证书了：
![image](https://user-images.githubusercontent.com/10510431/207522618-00ab0cb3-f3de-4703-bea4-7f050e07f35f.png)

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

### 配置调整

#### 主题
随便翻了下，发现 PaperMod 非常对我的胃口，就用它了。

#### Hexo tags 不规范
#### 文件名格式
#### 添加 favicon