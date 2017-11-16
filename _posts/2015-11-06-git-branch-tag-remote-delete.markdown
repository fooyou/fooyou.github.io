---
layout: post
title: git 删除远程tag和branch
category: Document
tags: git
date: 2015-11-06 14:15:10
published: true
summary: git删除远程tag和branch如何做。
image: pirates.svg
comment: true
latex: false
---

删除远程分支：

```
$ git branch -r -d origin/branch-name
$ git push orign :branch-name
```

删除远程tag

```
$ git tag -d tag_name
$ git push origin :refs/tags/tag_name
```

