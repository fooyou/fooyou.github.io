---
layout: post
title: brew 更新为国内源
category: Document
tags: brew
date: 2020-05-15 15:05:30
published: true
summary: 使用 ustc 大学的镜像替换官方镜像
image: pirates.svg
comment: true
---

```bash
# 替换 Homebrew
git -C "$(brew --repo)" remote set-url origin https://mirrors.ustc.edu.cn/brew.git

# 替换 Homebrew Core
git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git

# 替换 Homebrew Cask
git -C "$(brew --repo homebrew/cask)" remote set-url origin https://mirrors.ustc.edu.cn/homebrew-cask.git

# 替换 Homebrew-bottles
# 对于 bash 用户：
# echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.bash_profile
# source ~/.bash_profile
# 对于 zsh 用户：
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.zshrc
source ~/.zshrc
```


但有点尴尬的是，二进制文件下载替换后，反而变的更慢了！
