---
layout: post
title: 黑客大曝光 Linux 第一章
category: Document
tags: 
date: 2016-04-22 14:04:12
published: true
summary: 
image: pirates.svg
comment: true
---

案例研究：

西蒙是 Linux 系统的铁粉，但他所在的公司却只有他一人在使用桌面系统

hardcore：铁杆
malware: 恶意软件
worm：蠕虫病毒
grab：截获
audit：审查

恶意权限提升

## 重要工具

1. tcpdump 命令抓包
2. Systrace: 系统访问控制工具
3. Metasploit: 软件和驱动漏洞发现工具
4. AppArmor: 对应用程序做概要分析（profile），可探查App文件权限，并可使用描述文件限制应用程序。
5. umask / chmod: umask 可更改系统创建文件和文件夹时的默认权限，chmod可更改单独的文件或文件夹的权限。
6. chattr: 不可变文件(immutabel)设置，设置为不可便文件，连 root 都不可以对其进行修改或者删除，除非移除该标志位。
    
    ```
    $ chattr +i /var/test_file
    ```

    设置后，可使用 lsattr 进行查看

    ```
    $ lsattr /var/test_file
    ----i-------- test_file
    ```
7. strace: 可跟踪所有系统调用和可执行的编译。给定一个可执行文件，它将枚举所有与该文件相关的文件，包括配置文件、库依赖关系、打开的文件、输出的文件。它能显示系统单步调试本身所执行的一个二进制文件时大量的输出。

    ```
    $ strace touch
    open("/usr/lib/locale/locale-archive", O_RDONLY|O_CLOEXEC) = 3
    fstat(3, {st_mode=S_IFREG|0644, st_size=7216496, ...}) = 0
    mmap(NULL, 7216496, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f25eed40000
    close(3)                                = 0
    ```
8. ldd: 可枚举可执行文件的库依赖关系，但它不列举配置文件或打开的文件

    ```
    $ ldd touch
    linux-vdso.so.1 =>  (0x00007fffbe9f1000)
    libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fd8d2fcd000)
    /lib64/ld-linux-x86-64.so.2 (0x00007fd8d3392000)
    ```

9. lsof: 列举指定 daemon 程序所使用的所有打开文件

    ```
    $ lsof | grep sshd
    ...
    ```

