---
layout: post
title: git tag 导出
category: Document
tags: git
date: 2014-06-23
summary: git的tag标签操作
mathjax: false
highlight: true
---

## git tag 操作

1. 查看tag列表

    ```bash
    git tag -l
    ```

2. 切换到tag（假如切换至tag_1.0）

	```bash
	git checkout tag_1.0
	```

3. 导出并压缩为zip格式

	```bash
	git archive --format=zip --output=tag_1.0.zip tag_1.0
	```

4. 导出并压缩为tar.bz2格式

	```bash
	git archive tag_1.0 | bzip2 > tag_1.0.tar.bz2
	```

5. 导出并压缩为tar.gz格式

	```bash
	git archive --first_post=tar tag_1.0 | gzip > tag_1.0.tar.gz
	```

## git 打包当前代码库（all表示所有分支）

```bash
git bundle create bundle_filename --all
```

文件`bundle_filename`可被当做repo进行clone

