---
layout: post
title: Ubuntu VirtualBox Windows 共享文件夹
category: Document
tags: windows virtualbox ubuntu
year: 2014
month: 11
day: 28
published: true
summary: Ubuntu下，和VirtualBox下的Windows系统共享文件夹 
image: pirates.svg
comment: true
---

有些时候，我们可能需要在Ubuntu中虚拟一个Windows操作系统，如下是我的环境配置： 

- 母操作系统：Ubuntu 14.04 + VirtualBox 4.2 
- 子操作系统：Window XP 

至于如何安装VirtualBox及虚拟Windows XP，这里不再做详细的说明，只是简要写一下步骤： 

1. 在终端中运行：sudo apt-get install virtualbox 
2. 输入您的登录口令，然后就是一步步的确认安装； 
3. 创建一个Windows XP虚拟系统，插入光盘，然后就是一步步安装XP的过程。 

下面着重要讲的是母操作系统Ubuntu如何与子操作系统Windows XP进行文件共享的问题。 

1. 安装VirtualBox的**增强功能包**。
2. 在子操作系统Windows XP 中出现安装提示，一路确认安装下去即可。 
3. 重新启动Windows XP后，打开VirtualBox的菜单：设备(D)-->分配数据空间(S)... 
4. 点击新增按钮，添加一个共享目录，您可以根据需要确定共享的目录是否只读，及是否仅共享于当前传话。 
5. 确定之后，回到子操作系统Windows XP，此时Windows XP操作系统内并没有任何变更（不会出现发现新硬件或者多出一个共享空间等），下面还需要您的手动配置才行。 
6. 打Windows XP中"我的电脑"，然后可以查看您的操作系统中各磁盘的盘符信息，VirtualBox中文件共享的机制是将共享文件夹作为一个单独的硬件，因此我们可以将它视作一个网络共享硬件或者是移动设施。在Windows XP中，我们需要为刚才的共享文件分配一个盘符才行，在命令提示符中运行如下命令：（命令是windos下的cmd） 

```
net use x: \\vboxsvr\share 
```

*说明：x:为Windows XP操作系统中可分配的盘符信息，不能与已有的盘符重复；//vboxsvr 为 VirtualBox 标识； share即为刚才您为共享文件夹取的共享名称。*

如我要在我的Windows XP中创建盘符为e:的网络驱动，则需要执行如下的语句： 

```
net use e: \\vboxsvr\Data 
```

然后您就可以像本地文件一样存取共享文件夹中的内容，实现母子操作系统中的数据共享了。 

*附：如果您的子操作系统是Linux操作系统，您可以通过如下方式实现共享：*

```
mount -t vboxsf share mount_point 
```

*说明：share为您的共享文件夹别名，与上面相同；mount_point为您想加载到文件夹路径，可以设置到您的当前文件夹下，如/home/amon/share/。*