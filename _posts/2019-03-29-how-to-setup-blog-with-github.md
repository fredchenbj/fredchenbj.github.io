---
layout: post
title:  "How to Setup Blog with Github"
date: 2019-03-29 21:06:26 +0800
categories: blog
---

使用 [GitHub Pages](https://pages.github.com) 和 [Jekyll](https://jekyllrb.com) 搭建个人博客是非常简单和轻松的。下面介绍我的搭建过程。

<!--description-->

## GitHub Pages
- 从上图 Github Pages 主页上可以看出，它主要用于个人网站（blog）和项目网站
  - 直接由 Github 来提供 Web 服务的，而不用购买 vps 和域名
  - 只需要做：编辑、上传
![image]({{site.baseurl}}/assets/img/github-pages.jpg)
- 个人网站只需要创建一个 Github repository 就可以了
![image]({{site.baseurl}}/assets/img/create-blog-repo.jpg)
- 在 Settings 可以看到 Github Pages 的相关配置选项
  - 其中显示了 site url
  - "Theme Chooser" 可以选择 Github 提供的 theme，主要用于 project website
  - "Custom domain" 可以选用自定义的域名
![image]({{site.baseurl}}/assets/img/pages-in-settings.jpg)

## Jekyll
- Jekyll 是一种 static site generator，并且是 Github Pages 官方指定的（默认），可见 [相关文档](https://help.github.com/en/articles/using-jekyll-as-a-static-site-generator-with-github-pages)
- 选择 Jekyll theme
  1. 从网站 <http://jekyllthemes.org> 上选择一个自己喜欢的主题。这里，我选的是 [jekyll-simple](http://jekyllthemes.org/themes/jekyll-simple/) 主题
![image]({{site.baseurl}}/assets/img/jekyll-simple.jpg)
  2. 从 Download 下载 zip 文件，解压缩后复制到博客仓库的本地目录下
  3. **修改 _config.yml**。特别注意 baseurl 的配置。如果是 ***.github.io 项目，不修改为空 '' 的话，会导致 JS, CSS 等静态资源无法找到的错误
  4. `git add --all & git commit -s -a -m & git push`
  5. 稍等一会，就可以从 url site 访问博客最新内容了。
  6. 在 Github 项目主页里的 Deployments 栏目下可以看到 github-pages 的构建活动。
![image]({{site.baseurl}}/assets/img/deployment-logs.jpg)
- 本地运行网站来 preview 网页
  1. 参照 [jekyll Quickstart](https://jekyllrb.com/docs/)

## 撰写博客
- 使用 markdown 撰写 blog 放入 _posts 目录，其中 blog 按 `YYYY-MM-DD-name-of-post.md` 格式命名
  - quick reference of [jekyll markdown](https://gist.github.com/roachhd/779fa77e9b90fe945b0c)
- 参照 [style](https://fredchenbj.github.io/jekyll/2016/07/03/styles-for-simple.html) 和 [中文排版指南](https://zhuanlan.zhihu.com/p/20506092)
