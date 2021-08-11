---
layout: post
title: 使用 youtube-dl 只下载字幕
category: Document
tags: 
date: 2021-08-11 10:08:39
published: true
summary: 怎么用 youtube-dl 只下载字幕呢
image: pirates.svg
comment: true
---

看大佬的技术分享视频，获益良多，有了 ppt，但视频中有诸多的细节是 ppt 里没有的，所以想把字幕下下来温习一下。

## 首先查看跟字幕相关的功能

```bash
$ youtube-dl --help | grep sub
                                         caseless sub-string)
                                         (regex or caseless sub-string)
    --write-sub                          Write subtitle file
    --write-auto-sub                     Write automatically generated subtitle
    --all-subs                           Download all the available subtitles of
    --list-subs                          List all available subtitles for the
    --sub-format FORMAT                  Subtitle format, accepts formats
    --sub-lang LANGS                     Languages of the subtitles to download
                                         --list-subs for available language tags
    --embed-subs                         Embed subtitles in the video (only for
    --convert-subs FORMAT                Convert the subtitles to other format
```

这下就很清晰了。

## 查看字幕

```bash
$ youtube-dl --proxy 0.0.0.0:1087 --list-subs https://youtu.be/WQY85vaQfTI
[youtube] WQY85vaQfTI: Downloading webpage
Available subtitles for WQY85vaQfTI:
Language formats
zh-TW    vtt, ttml, srv3, srv2, srv1
```

## 下载字幕

```bash
$ youtube-dl --proxy 0.0.0.0:1087 --sub-lang zh-TW --skip-download https://youtu.be/WQY85vaQfTI
[youtube] WQY85vaQfTI: Downloading webpage
WARNING: Unable to download webpage: <urlopen error EOF occurred in violation of protocol (_ssl.c:1129)>
[youtube] WQY85vaQfTI: Downloading API JSON
[youtube] WQY85vaQfTI: Downloading API JSON
[info] Writing video subtitles to: 【機器學習2021】機器學習模型的可解釋性 (Explainable ML) (上) – 為什麼類神經網路可以正確分辨寶可夢和數碼寶貝呢？-WQY85vaQfTI.zh-TW.vtt
```

DONE!
