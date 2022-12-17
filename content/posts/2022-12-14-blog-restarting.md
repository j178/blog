---
title: "Blog 重启之路"
slug: blog-restarting
date: 2022-12-14T00:32:47+08:00
draft: true
---

无意中看到了自己的 GitHub 上的 [blog source](https://github.com/j178/blog)，发现上一次折腾博客已经是 5 年多以前了。
那会儿还在上学，刚接触编程其实也没多久，喜欢折腾各种没有技术含量的东西(从那时候就开始走上了歪路:(
![image](https://user-images.githubusercontent.com/10510431/208245025-7720f0db-7056-4223-8c30-77dd881dab18.png)

依稀记得之前是用 Hexo 本地构建出静态文件后，push 到 github pages 部署的。并且由 cloudflare 做 CDN 和 https 支持。
不过由于长时间没有维护，连域名都弄丢了。而且 5 年之后，想让基于 Node 的 Hexo 再成功跑起来，也是相当不容易。所以这一次我决定重头开始，尝试用 Hugo 来构建博客，并用上(白嫖) GitHub 的一些新特性。

5 年过去了，github 也做了很多改进，可以让我们的部署过程更加方便。比如说：

1. 由于有了 github actions，我们可以自定义 action 来自动 build 静态文件。
2. github pages 支持了从 github artifacts 中部署，所以我们也不再需要将静态文件保存在 `gh-pages` 分支中嘞。
3. github pages 也支持自动帮我们申请 let's encrypt 的 SSL 证书，所以自定义域名也不再需要用 cloudflare 的 CDN 了。

## 自定义域名

1. 创建一个 `static/CNAME` 文件，Hugo 会自动将其复制到 `public` 目录下。
2. 在 settings - pages 中设置 custom domain
3. 在 cloudflare 中添加 CNAME 解析，将自定义域名 CNAME 到 <your-name>.github.io，注意关闭 proxy，因为我们要使用 github pages 自带的证书。
4. 等待 GitHub 为你的自定义域名申请 Let's Encrypt 证书，[这个过程可能需要几个小时](https://docs.github.com/en/pages/getting-started-with-github-pages/securing-your-github-pages-site-with-https)。

cloudflare 开启了 proxied 之后的 CNAME 并不是普通的 CNAME，而是由 cloudflare 服务器接管了。以下是关闭 proxied 前后的 dig 的结果：

![image](https://user-images.githubusercontent.com/10510431/207522605-cb76812b-f69a-42f3-a7fd-c7930125baaf.png)

关闭 proxied 之后，github pages 页面可以正常申请证书了：

![image](https://user-images.githubusercontent.com/10510431/207522618-00ab0cb3-f3de-4703-bea4-7f050e07f35f.png)

## 迁移记录

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

### 配置 Syntax Highlighting

Hugo 内置了 [chroma](https://github.com/alecthomas/chroma) 来实现代码的语法高亮，有两种实现方式：

1. 通过元素的 `style` 属性直接控制，对应 `highlight.noclasses = false`，是默认情况。
2. 使用 CSS Classes 来控制，对应 `highlight.noclasses = false`。
   这种情况下需要自己提供 CSS，可以通过 `hugo gen chromastyles --style=github` 来生成。

Chroma 支持在 code fence 的 info string 后提供参数，比如

````md
```go {linenos=table,hl_lines=[7,"13-14"],linenostart=199}
...
```
````

就可以生成这样的效果：

![image](/img/2022-12-18-03-46-17.png)

但是，一些主题(比如 PaperMod)会使用 highlight.js 来实现代码高亮，而 highlight.js 是不支持高亮行的。
这种情况下要么放弃高亮行的功能，直接使用主题提供的开箱即用的方案，要么关闭主题的 HLJS，配置 Hugo 的 Chroma 来实现代码高亮。
不过要想调出一个与主题风格一致的代码高亮，还是需要花一些时间的。（有的主题可能还包括 dark mode，这就更麻烦了）

### 配置调整

- [x] 选择主题
- [ ] 修复不规范的 Hexo tags
- [ ] 修复文件名格式，统一 slug
- [x] 添加 favicon
- [x] 配置目录、social
- [x] 配置 suggest changes
- [ ] 配置 twitter card

## Ref

- https://www.brycewray.com/posts/2022/06/get-good-git-info-hugo/
- https://dennislee.xyz/2020/hugo-jekyll-style-date-and-slug-from-filename/
- https://docspring.com/blog/posts/adding-a-timestamp-to-hugo-post-filenames/
