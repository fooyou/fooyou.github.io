---
layout: post
title: python的定时input
category: Document
tags: python
date: 2015-11-05 10:38:10
published: true
summary: 使用signal的signal和alarm方法产生异常来中断input()，ffplay命令的使用
image: pirates.svg
comment: true
latex: false
---

VLC、Media Player、etc，but I love ffmpeg the best.

15分钟拼装了个linux下听歌的脚本，使用ffplay。

```python
#!/usr/bin/env python
#! coding: utf8

import os
import random
import config
import signal
import re

def play(path):
    print('播放：', path)
    # os.system('ffplay ' + path.replace(' ', '\ ') + ' -autoexit -hide_banner')
    os.system('ffplay ' + re.escape(path) + ' -autoexit -hide_banner')

song_list = []
play_list = []
song_type = ['mp3', 'wav']

def gen_songlist(folders):
    # 遍历目录下的音乐文件
    for folder in folders:
        for root, dirs, files in os.walk(folder):
            for fl in files:
                for st in song_type:
                    if fl.endswith(st):
                        song_list.append(os.path.join(root, fl))
                        break

    for song in song_list:
        play_list.append(song)

def show():
    # 打印音乐列表
    print('序号\t\t歌名')
    for i, song in enumerate(song_list):
        print(i, song)

def interrupt(signum, frame):
    raise('Time out')

def timer_input(sec):
    signal.signal(signal.SIGALRM, interrupt)
    signal.alarm(sec)
    try:
        istr = input('序号播放，"ls"查看列表，"q"退出：')
    except:
        istr = None
    finally:
        signal.alarm(0)
    return istr

def parse():
    istr = timer_input(5)
    if None == istr:
        if len(play_list) > 0:
            no = random.randint(0, len(song_list) - 1)
            play(play_list[no])
            play_list.remove(play_list[0])
            
    elif 'l' == istr or 'ls' == istr:
        show()
    elif 'q' == istr:
        exit(0)
    else:
        try:
            no = int(istr)
            if no < 0 or no >= len(song_list):
                no = random.randint(0, len(song_list) - 1)
        except ValueError as e:
            no = random.randint(0, len(song_list) - 1)

        play(song_list[no])
    parse()


if __name__ == '__main__':
    gen_songlist(config.music_folder)
    show()
    parse()
```

配置脚本，音乐库目录

```
#!/usr/bin/env python
#! codeing: utf8

music_folder = [
        '/home/joshua/Music/',
        '/media/joshua/My_Resource/Entertainment/music/'
    ]
```

如何设置定时input，这种方法有点怪异。

