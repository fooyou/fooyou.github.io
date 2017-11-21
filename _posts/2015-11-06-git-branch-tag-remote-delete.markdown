---
layout: post
title: git 删除远程tag和branch
category: Document
tags: git
date: 2015-11-06 14:15:10
published: true
summary: git删除远程tag和branch如何做。
mathjax: false
highlight: true
---

删除远程分支：

```bash
$ git branch -r -d origin/branch-name
$ git push orign :branch-name
```

删除远程tag

```bash
$ git tag -d tag_name
$ git push origin :refs/tags/tag_name
```

