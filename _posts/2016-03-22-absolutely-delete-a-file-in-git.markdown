---
layout: post
title: git 仓库如何彻底删除某个文件？
category: Document
tags: git
date: 2016-03-22 16:03:56
published: true
summary: 回退一次 commit 很容易，但彻底删除一次 commit 或者彻底删除一个误传的文件需要……
image: pirates.svg
comment: true
---

git 仓库的设计是比较保守的，这个结论来自 git 文件提交管理里，所以对于误操作 git 的态度是非常严谨的，想轻易的删除一次提交或者某个已经提交的文件是很困难的，尤其是已经上传到版本库的提交，利用远程版本控制貌似删除无法彻底清除，除非你是管理员。


不小心提交了个训练模型，异常的大，又 push 到了 remote 里，不仅暴跳如雷……×%￥#……×&×（

删除方法如下：


```
$ git filter-branch --tree-filter 'rm -f path/to/file' HEAD
$ git ls-remote .
$ git update-ref -d refs/original/refs/heads/master
$ rm -rf .git/logs/
$ git reflog --all
$ git prune
$ git gc
```


```
$ git filter-branch --index-filter 'git rm -f --cached --ignore-unmatch path/to/file' HEAD
$ git push origin master --force
$ rm -rf .git/refs/original/
$ git reflog expire --expire=now --all
$ git gc --prune=now
$ git gc --aggressive --prune=now
```

这时，本地的 .git/ 目录下的二进制数据已经删除，origin 中的并没有删除，另外如果是多人协同的话，其他人的 git 仓库里也没有删除，为避免其他人再次 push 到 origin 里，需要其他人也执行一边除了第二行 git push 以外的其他脚本。或者可以重新 clone。

那么，测试一下吧：


```
joshua@joshua:~/Documents$ mkdir gdt
joshua@joshua:~/Documents$ cd gdt/
joshua@joshua:~/Documents/gdt$ git init
初始化空的 Git 版本库于 /home/joshua/Documents/gdt/.git/
joshua@joshua:~/Documents/gdt$ dd if=/dev/urandom of=test.txt bs=10240 count=1024
记录了1024+0 的读入
记录了1024+0 的写出
10485760字节(10 MB)已复制，0.762128 秒，13.8 MB/秒
joshua@joshua:~/Documents/gdt$ git add test.txt
joshua@joshua:~/Documents/gdt$ git commit -m 'Initial commit'
[master （根提交） 74ef95d] Initial commit
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 test.txt
joshua@joshua:~/Documents/gdt$ du -sh
11M     ./.git
21M     .
joshua@joshua:~/Documents/gdt$ git rm test.txt
rm 'test.txt'
joshua@joshua:~/Documents/gdt$ git commit -m 'delete txt'
[master c914919] delete txt
 1 file changed, 0 insertions(+), 0 deletions(-)
 delete mode 100644 test.txt
joshua@joshua:~/Documents/gdt$ du --max-depth=1 -h
 11M     ./.git
 11M     .
```

删除了，还有11M的 .git/ 缓存中，为能恢复代码，彻底删除吧！！这是个失误的操作。


```
joshua@joshua:~/Documents/gdt$ git filter-branch --tree-filter 'rm -f test.txt' HEAD
Rewrite c914919c47716d4fb13df4b0768a0e01e3776c3f (2/2)
Ref 'refs/heads/master' was rewritten
joshua@joshua:~/Documents/gdt$ git ls-remote .
9df643968a6b41bb1a76d46231c88733118ae81e        HEAD
9df643968a6b41bb1a76d46231c88733118ae81e        refs/heads/master
c914919c47716d4fb13df4b0768a0e01e3776c3f        refs/original/refs/heads/master
joshua@joshua:~/Documents/gdt$ git update-ref -d refs/original/refs/heads/master
joshua@joshua:~/Documents/gdt$ git ls-remote .
9df643968a6b41bb1a76d46231c88733118ae81e        HEAD
9df643968a6b41bb1a76d46231c88733118ae81e        refs/heads/master
joshua@joshua:~/Documents/gdt$ git update-ref -d refs/original/refs/heads/master
joshua@joshua:~/Documents/gdt$ git ls-remote .
9df643968a6b41bb1a76d46231c88733118ae81e        HEAD
9df643968a6b41bb1a76d46231c88733118ae81e        refs/heads/master
joshua@joshua:~/Documents/gdt$ rm -rf .git/logs/
joshua@joshua:~/Documents/gdt$ git reflog --all
joshua@joshua:~/Documents/gdt$ git prune
joshua@joshua:~/Documents/gdt$ git gc
Counting objects: 3, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), done.
Total 3 (delta 1), reused 0 (delta 0)
joshua@joshua:~/Documents/gdt$ du -sh
140K    .
```

OK!


> 参考：http://blog.csdn.net/king_sundi/article/details/7751906
