---
layout: post
title: linux终端邮件客户端 mutt msmtp fetchmail procmail
category: Document
tags: mail linux mutt fetchmail msmtp
date: 2016-03-29 10:03:35
published: true
summary: 使用 mutt 搭建 linux 邮件客户端
image: pirates.svg
comment: true
---

*喜欢它，爱屋及乌，发现乌鸦也很棒。*

需要安装的软件工具：

1. mutt 管理邮件
2. msmtp 发邮件
3. fetchmail 收邮件
4. procmail 邮件处理
5. ca_root_nss 认证（ubuntu 14.04+ 已默认安装）
6. 为浏览邮件里的 doc, pdf, html, audio, video等文件，而安装的 wv, xpdf, w3m, mplayer等

在 $HOME 下新建 .mail/ 目录存放邮件（名字可随意起）inbox 是收件，以下所有的 rc 文件的访问权限都是 600，这样别人不会轻易的获取你的邮箱密码。

## 发送邮件配置： .msmtprc

```
## Description: ~/.msmtprc of msmtp
## Author: Joshua Liu
## Settings: chmod 600 .msmtprc
## LastUpdate: 24-03-2016

# Set default values
defaults
  logfile ~/.mail/log/msmtp.log
  tls on
  tls_starttls on
  tls_certcheck on

# smtp of gmail server
account gmail
  host smtp.gmail.com
  port 587
  user xxx
  from xxx@gmail.com
  password xxxxxx
  auth login
  tls_fingerprint 31:E8:68:CB:58:1C:01:7C:07:1A:25:39:4A:E6:9A:73:2B:25:B4:47

## Multi-accounts specify default
# account default : gmail
```

**auth on/off/login**

这个选项

**获取 tls_fingerprint 或者 tls_certfile**

```
$ msmtp --serverinfo --host=smtp.gmail.com --tls=on --tls-certcheck=off
SMTP server at smtp.gmail.com (ig-in-x6c.1e100.net [2607:f8b0:4001:c05::6c]), po
    mx.google.com ESMTP ga10sm76109igd.0 - gsmtp
TLS certificate information:
    Owner:
        Common Name: smtp.gmail.com
        Organization: Google Inc
        Locality: Mountain View
        State or Province: California
        Country: US
    Issuer:
        Common Name: Google Internet Authority G2
        Organization: Google Inc
        Country: US
    Validity:
        Activation time: Tue 15 Jul 2014 08:40:38 AM UTC
        Expiration time: Sat 04 Apr 2015 03:15:55 PM UTC
    Fingerprints:
        SHA1: 9C:0A:CC:93:1D:E7:51:37:90:61:6B:A1:18:28:67:95:54:C5:69:A8
        MD5: E7:48:1D:0B:99:4A:C3:A8:31:86:E5:8F:E5:EE:4F:2A
Capabilities:
    SIZE 35882577:
        Maximum message size is 35882577 bytes = 34.22 MiB
    PIPELINING:
        Support for command grouping for faster transmission
    STARTTLS:
        Support for TLS encryption via the STARTTLS command
    AUTH:
        Supported authentication methods:
        PLAIN LOGIN
```

可以得到证书指纹和证书的发布者名称 “Google Internet Authority G2” 可检索[下载](https://pki.google.com/GIAG2.crt)到证书，然后转换一下证书成可读格式：

```
$ openssl x509 -inform DER -in GIAG2.crt -outform PEM -out gmail-smtp.crt
```

下面的指令能成功登录：

```
$ msmtp --serverinfo --host=smtp.gmail.com --tls=on --tls-trust-file=gmail-smtp.crt
```


## 收取邮件：.fetchmailrc

```
## Description: ~/.fetchmailrc for fetchmail
##      More info: man fetchmail
## Author: Joshua Liu
## Settings: chmod 600 .fetchmailrc
## LastUpdate: 24-03-2016

# 设置接收间隔为 300s
set daemon 300
# 设置日志文件
set logfile "~/.mail/log/fetchmail.log"

#
# gmail service
poll smtp.gmail.com with proto POP3
  port 995
  user "username" there with password "xxxxxxx" is "cp-user" here
  mda "/usr/bin/procmail -d %T"
  keep
  no fetchall
  no rewrite
  options
  ssl
  sslproto tls1
  sslcertpath "/etc/ssl/certs/"
  sslfingerprint "68:35:95:7D:41:3A:57:67:B1:34:75:A4:CD:B9:35:8C"
  # sslcertck

```

**获取发件证书**

```
$ openssl s_client -connect pop.gmail.com:995
or
$ openssl s_client -host pop.gmail.com -port 995
```

可查看 PEM 格式的发件证书


## 邮件处理：.procmail

```
## Description: ~/.procmailrc of procmail
##              处理邮件
## Author: Joshua Liu
## Setting: chmod 600 .procmailrc
## LastUpdate: 24-03-16

SHELL=/bin/bash
PATH=/bin:/usr/bin:/usr/local/bin
MAILDIR=$HOME/.mail
DEFAULT=$MAILDIR/inbox
LOGFILE=$MAILDIR/log/procmail.log
LOG=""
VERBOSE=yes

## 丢弃垃圾邮件
#reason to return these
#:0:
#*^From:.*(noreply\@tcl\.com)
#/dev/null

# 所有经过查看后的邮件存储在 $MAILDIR/inbox 信箱中
:0
* .* inbox
default

```


## 邮件管理：.muttrc

```
## Description: ~/.muttrc of mutt
##              管理邮件
## Author: Joshua Liu
## Setting: chmod 600 .muttrc
## LastUpdate: 24-03-2016

## 必需的全局设置
set hostname="ikuduku.com"  # localhost
set hidden_host             # 隐藏 host 细节

#
# 信箱设置
#
set folder=~/.mail                  # (+ / =) 存储目录
set mbox=+inbox                     # 缺省邮箱 (~/.mail/inbox)
set mbox_type=mbox                  # 共有4个可选项，指定如何存储已读邮件
set spoolfile=+inbox                # 收件箱
set postponed=+postponed            # 推迟发送
set alias_file=+aliases             # 别名
set record="+sent-`date +%Y-%m`"    # 已发邮件目录 ($folder/sent-2016-03)
set header_cache=+mutt_cache        # cache，可以快速打开 mailboxes

#
# 收信设置
#
macro index G "!fetchmail -a -m 'procmail -d %T'\r"
macro pager G "!fetchmail -a -m 'procmail -d %T'\r"
set check_new=yes                   # 检查新邮件
set mail_check=60                   # 每分钟检查一次
set timeout=30

#
# 发信设置
#
set sendmail="/usr/bin/msmtp"       # 用 msmtp 发信
set use_from=yes                    # 让 msmtp 知道用哪个账号
set from="xxxx@gmail.com"           # 缺省从此账号发信
set realname="xxxxxx"
set envelope_from=yes               # 让 mutt 使用 msmtp 的 -f 选项
set include                         # 发信包含原文
set indent_str=">"                  # 回信的引文插入的符号
set ispell="/usr/bin/aspell"        # 英文拼写检查

#
# 邮件格式
#
my_hdr From: xxxx@gmail.com                 # 缺省发件地址
my_hdr Reply-To: xxxx@gmail.com             # 缺省回件地址
set index_format="| %4C | %Z | %{ %b %d } | %-15.15L | %s"
set folder_format="| %2C | %t %N | %8s | %d | %f"


##
## 多个账户切换
##
# macro generic "<esc>1" ":set from=xxxxx@gmail.com"
# macro generic "<esc>2" ":set from=xxxxx@outlook.com"
# macro generic "<esc>3" ":set from=xxxxx@yahoo.com"

#
# 邮件打分
#
set sort=reverse-date-received  # 按日期由近及远排列
#set sort_aux=score              # 按评分排序
#score "~N" +4                   # 新信件 +4
#score "~s 会议" +2              # 主题包含“会议” +2
#score "~s 通知" +2              # 主题包含“通知” +2
#score "~s meeting" +2           # 主题包含“meeting” +2
#score "~D" -5                   # 已经标记删除的 -5
#score "~O" +1                   # 上次没读的 +1

#
# 地址簿设置（使用 mutt-aliases 和 abook）
#
source $alias_file
set query_command="abook --mutt-query '%s'"
macro editor ";" \Ct

#
# 颜色设置：前景色 + 背景色
#
color normal        white           default     # 正常：白色（default 透明）
color attachment    yellow          default     # 附件：黄色
color bold          brightwhite     default     # 粗体：亮白
color underline     default         blue        # 下划线：蓝色
color error         brightwhite     default     # 错误：亮白
color indicator     default         magenta     # 光标所在行
color message       brightblue      default 
color status        brightgreen     default
color header        brightgreen     default ^From:
color header        brightcyan      default ^To:
color header        brightcyan      default ^Reply-To:
color header        brightcyan      default ^Cc:
color header        brightred       default ^Subject:
color body          brightwhite     default [\-\.+_a-zA-Z0-9]+@[\-\.a-zA-Z0-9]+
color body          brightblue      default (https?|ftp)://[\-\.,/%~_:?&=\#a-zA-Z0-9]+
color index         brightblue      default ~N
color index         brightwhite     default ~O

#
# 个人使用习惯
#
set editor="vim"                   # vim 编写邮件
set edit_headers=yes               # 允许编辑邮件头
set nomark_old                     # 未读新邮件别标注为旧邮件
set copy                           # 保留已发邮件的备份
set beep_new=yes                   # 来新邮件时，蜂鸣
set smart_wrap                     # 禁止从单词中间断行
set nomarkers                      # 禁止换行标记
set mime_forward                   # 转发的邮件 MIME 附件
set pager_index_lines=4            # 看信时在 index 留出多少行显示邮件列表
set pager_context=3                # Display 3 lines of context in pager
set nostrict_threads               # Lets have some fuzzy threading
set wait_key=no                    # 外部程序退出时，要求用户按键返回
set sendmail_wait=-1               # Don't wait around for sendmail
set fcc_clear                      # Keep fcc's clear of signatues and encryption
set nopipe_decode                  # Don't decode messages when piping
set tilde                          # 过滤带 ~ 的邮件
set read_inc=100                   # Read counter ticks every 100 msgs
set write_inc=100                  # Write counter ticks every 100 msgs
set noconfirmappend                # Just append, don't hassle me
set pager_stop                     # Don't skip msgs on next page
set resolve=yes                    # 按 "t" 或 "D" 时，自动移动光标到下封
set fast_reply                     # 按 "r" 回信时，直接进入编辑模式
set quit=yes                       # 退出时，直接退出
set postpone=ask-no                # 推迟发送
set move=yes                       # 邮件移至 mbox
set delete=ask-yes                 # 删除前询问
ignore x-mailer                    # 忽略 x-mailer 邮件头
auto_view text/html application/msword    # 自动浏览邮件中 text/html 
alternative_order text/enriched text/plain text/html

#
# 编码及中文设置
#
set locale="zh_CN.UTF-8"            # 使用中文
set ascii_chars=yes                 # ascii 表示树状列表
set charset="utf-8"                 # 编码及发件编码
set send_charset="us-ascii:iso-8859-1:utf-8:gb2312"
charset-hook ^us-ascii$ gb2312      # 把 us-ascii 编码映射到 gb2312
charset-hook ^iso-8859-1$ gb2312    # 把 iso-8859-1 编码映射到 gb2312
charset-hook !utf-8 gb2312          # 把非 utf-8 编码映射到 gb2312
```

## 附件浏览：.mailcap

```
## Description: View the attachments of mutt.
## Functions: (1) Convert *.doc to *.html by wvHtml;
##            (2) Use w3m to read *.doc and *.html in Mutt;
##            (3) Use xpdf to view *.pdf;
##            (4) Use display to view images;
##            (5) Use mplayer to enjoy the audio and video.
## Author: Joshua Liu
## Last Update: 26-03-2016

# Microsoft word
application/msword; wvHtml %s - | w3m -dump; nametemplate=%s.html; copiousoutput
# pdf
application/pdf; xpdf %s > /dev/null
# html
text/html; w3m -dump -I %{charset} -T text/html %s; copiousoutput
# image
image/*; display %s; /dev/null
# audio
audio/*; mplayer %s > /dev/null
# video
video/*; mplayer %s > /dev/null
```


## 如何使用

安装设置 rc 完毕后，终端或 tty 输入 mutt 回车即可进入 mutt 邮件管理系统

- 输入 G 来收邮件
- V 查看附件
- s 保存附件
- d 删除邮件

其他操作查看帮助文档。


## 参考

> https://wiki.freebsdchina.org/doc/m/mutt
> https://wjianz.wordpress.com/2014/09/03/howto-installconfigure-msmtp-and-mutt-on-ubuntu/
