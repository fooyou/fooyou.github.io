---
layout: post
title: 使用ffmpeg修改mp3的ID3 tag（添加歌词、专辑等信息）
category: Document
tags: ffmpeg
date: 2015-12-03 12:12:42
published: true
summary: 有很多修改mp3 ID3 tag的工具，著名的有GNU的easytag，puddletag等，但万能的ffmpeg可不可以呢？当然可以。
image: pirates.svg
comment: true
---

## 基本用法

1. 使用 `-metadata` 后面跟 <key>=<value> 就可以修改相应的tag了。

    ```
    $ ffmpeg -i track12.mp3 -metadata album="Closer" You\ Raise\ Me\ Up.mp3
    ```

2. 重复使用`-metadata`加 key/value 修改多个tag

    ```
    $ ffmpeg -i track12.mp3 -metadata album="Closer" -metadata artist="Josh Groban" You\ Raise\ Me\ Up.mp3
    ```

3. 把value置空来删除某个标签，比如删掉 genre 标签

    ```
    $ ffmpeg -i track12.mp3 -metadata genre="" You\ Raise\ Me\ Up.mp3
    ```

可见使用 ffmpeg 修改tag是很简单的事，但难的是不知道音乐文件都有什么标签，或者我的音乐文件已经存在哪些标签，Windows上在音乐文件右键查看属性在详细页里有很多属性，iTunes里右键查看也能看到，那么标准的音乐文件都有哪些tag呢：

  Windows  |  iTunes(Info tab)  |  id3v2.3  |  ffmpeg key  | ffmpeg 示例
-----------|--------------------|-----------|--------------|---------------
 Title     | Title              | TIT2      | title        | -metadata title="海阔天空"
 Subtitle  | Description(Video tab)| TIT3   | TIT3         | -metadata TIT3="beyond 20周年纪念版"
 Rating    | n/a                | n/a       | n/a          | n/a
 Comments  | Comments           | COMM      | n/a          | n/a
 Contributing artists| Artist   | TPE1      | artist       | -metadata artist="黄家驹"
 Album artist| Album artist     | TPE2      | album_artist | -metadata album_artist="Josh Groban"
 Album     | Album              | TALB      | album        | -metadata album="Closer"
 Year      | Year               | TYER      | date         | -metadata date="2009"
 #         | Track Number       | TRCK      | track        | -metadata track="3/12"(12首歌中的第3个)
 Genre     | Genre              | TCON      | genre        | -metadata genre="Vocal"
 Publisher | n/a                | TPUB      | publisher    | -metadata publisher="Heaven Church"
 Encoded by| n/a                | TENC      | encoded_by   | -metadata encoded_by="Joshua"
 Aythor URL| n/a                | WOAR      | n/a          | n/a
 CopyRight(不可编辑)| n/a       | TCOP      | copyright    | -metadata copyright="℗  lqsoft"
 Composers | n/a                | TCOM      | composer     | -metadata composer="Joshua"
 Conductors| n/a                | TPE3      | performer    | -metadata performer="Joshua"
 Group description| Grouping    | TIT1      | TIT1         | -metadata TIT1="The Classics"
 Mood      | n/a                | n/a       | n/a          | n/a
 Part of set| Disc Number       | TPOS      | disc         | -metadata disc="1/2"
 Initial key| n/a               | TKEY      | TKEY         | -metadata TKEY="G"
 Beats-per-minute| BOM          | TBPM      | TBPM         | -metadata TBPM="120"
 Part of a compilation| Part of a compilation| TCMP | n/a  | n/a
 n/a       | n/a                | TLAN      | language     | -metadata language="eng"
 n/a       | n/a                | TSSE      | encoder      | -metadata encoder="iTunes v10"

## 进阶使用

1. 清除音乐文件所有的tag信息(`-map_metadata -1`可清除所有metadata信息)：

    ```
    $ ffmpeg -i track12.mp3 -map_metadata -1 out.mp3
    ```

2. 把音乐文件的metadata信息导出到文件里：

    ```
    $ ffmpeg -i track12.mp3 -f ffmetadata metadata.txt
     ```
    可对输出的 metadata.txt 进行修改，完事后重新写入到文件里。ffmetadata文件的格式如下：

    ```
    ;FFMETADATA1
    title=You Raise Me Up
    ;逗号可以添加注释
    artist=Josh Groban

    [CHAPTER]
    TIMEBASE=1/1000
    START=0
    #chapter ends at 0:01:00
    END=60000
    title=chapter \#1
    [STREAM]
    title=多行\
    多行
    ```

3. 把 metadata.txt 文件写入音乐文件

    ```
    $ ffmpeg -i track12.mp3 -i metadata.txt -map_metadata 1 -c:a copy -id3v2_version 3 -write_id3v1 1 out.mp3
    ```

    `-map_metadata 1`：代表使用输入顺序为1的文件，作为metadata，也就是 metadata.txt，这个数字和 `-i`的重复的次数有关，从0开始计数。

    `-c:a` codec audio的意思，这里用的是 copy，不写也行，默认值就是。

    `-id3v2_version 3`和`-write_id3v1 1`是为了对Windows兼容加上的，否则在Windows上会出现问题。

-------------

除了ffmpeg还有诸多的工具，`lame`还可以向音乐文件里插入图片：

```
$ lame --ai <path/to/*.[jpg|png|gif]> <*.mp3> [out.mp3]
```

参考：

> http://jonhall.info/how_to/create_id3_tags_using_ffmpeg

