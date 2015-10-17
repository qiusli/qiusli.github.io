---
layout: post
title: "如何使用Octopress写博客"
date: 2015-10-17 03:12:48 -0600
comments: true
categories: 杂记
---
具体的步骤是按照[这个地址](http://shengmingzhiqing.com/blog/setup-octopress-with-github-pages.html/)来的。配置什么的都在里面，这里简单说说配置完成后到写博客的步骤

1. rake new_post['blog title']创建新的文章
2. 完成编辑后运行rake generate
3. 然后rake preview在本地查看效果(http://localhost:4000/)
4. rake deploy到github上
5. git add . 来把本地文件并入git管理
6. git commit －m ‘whatever’
7. git push origin source