---
layout: post
title: linux下ssh离线执行任务的工具-tmux
category: Document
tags: linux
date: 2015-09-30 15:52:28
published: true
summary: ssh断开时目标机继续执行任务，通常使用nohup和screen，screen很强大貌似不在维护了，所以找到了一个后续版本tmux。
image: pirates.svg
comment: true
latex: false
---

ssh离线任务工具一般用的是nohup和screen，screen更为强大，而tmux是其后续，他们两个操作类似，下面只介绍tmux的使用。

screen和tmux可以轻松的管理ssh远程任务，session、windows的管理直观方便，无需像使用fg、bg命令那么麻烦。

##功能##

- 提供强劲的、易于使用的命令行界面
- 可横向和纵向分割窗口，窗格可以自由移动和调整大小，或直接利用四个预设布局之一。
- 支持utf-8编码及256色终端。
- 可在多个缓冲区进行复制和粘贴。
- 可通过交互式菜单来选择窗口、会话及客户端。
- 支持跨窗口搜索
- 支持自动及手动锁定窗口

## 安装

```
$ sudo apt-get install tmux
```

## 用法

- tmux      # 运行tmux -2 以256终端运行
- C-b d     # 返回主shell，tmux依旧在后台运行，里面的命令也保持运行状态
- tmux ls   # 显示已有tmux会话（C-b s）
- tmux attach-session -t number         # 切入到tmux的某会话
- tmux new-session -s session-name      # 新建某名称的会话
- tmux kill-session -t session-name     # 终止某名称的会话

## 快捷键

   快捷键   |    功能
------------|-----------------
 C-b ?      | 显示快捷键帮助
 C-b C-o    | 调换窗口位置
 C-b space  | 采用下一个内置布局
 C-b !      | 把当前窗口变为新窗口
 C-b "      | 横向分割窗口
 C-b %      | 纵向分割窗口
 C-b q      | 显示分割窗口编号
 C-b o      | 跳到下一个分割窗口
 C-b UP DOWN| 上一个及下一个分割窗口
 C-b C-方向键| 调整窗口大小
 C-b &      | 确认后退出tmux
 C-b c      | 创建新窗口
 C-b 0-9    | 选择n号窗口
 C-b n      | 选择下一个窗口
 C-b l      | 最后使用的窗口
 C-b p      | 前一个窗口
 C-b w      | 菜单显示及选择窗口
 C-b s      | 菜单显示及选择会话
 C-b t      | 显示时钟
 C-b [      | 复制（space开始）
 C-b ]      | 粘贴（Enter结束）
 C-b ,      | 给当前窗口改名


## 配置

配置文件：~/.tmux.conf

```
##############################
#  _
# | |_ _ __ ___  _   ___  __
# | __| '_ ` _ \| | | \ \/ /
# | |_| | | | | | |_| |>  < 
#  \__|_| |_| |_|\__,_/_/\_\
#
#############################
#
# COPY AND PASTE
# http://robots.thoughtbot.com/tmux-copy-paste-on-os-x-a-better-future
#
# Use vim keybindings in copy mode
setw -g mode-keys vi

# Setup 'v' to begin selection as in Vim
bind-key -t vi-copy v begin-selection
bind-key -t vi-copy y copy-pipe "reattach-to-user-namespace pbcopy"

# Update default binding of `Enter` to also use copy-pipe
# unbind -t vi-copy Enter
# bind-key -t vi-copy Enter copy-pipe "reattach-to-user-namespace pbcopy"
#
############################################################################
############################################################################
# Reset Prefix
############################################################################
set -g prefix C-a
bind-key a send-prefix # for nested tmux sessions

############################################################################
# Global options
############################################################################
# large history
set-option -g history-limit 10000

# colors
setw -g mode-bg black
set-option -g default-terminal "screen-256color" #"xterm-256color" # "screen-256color"
set-option -g pane-active-border-fg green

# utf8 support
set-window-option -g utf8 on


# basic settings
set-window-option -g xterm-keys on # for vim
set-window-option -g mode-keys vi # vi key
set-window-option -g monitor-activity on
set-window-option -g window-status-current-fg white
setw -g window-status-current-attr reverse

# Automatically set window title
setw -g automatic-rename

# use mouse # More on mouse support http://floriancrouzat.net/2010/07/run-tmux-with-mouse-support-in-mac-os-x-terminal-app/
setw -g mode-mouse on
#setw -g mouse-resize-pane on
#set -g mouse-select-window on
set -g mouse-select-pane on
set -g terminal-overrides 'xterm*:smcup@:rmcup@'

# vi movement keys
# set-option -g status-keys vi

############################################################################
# Status Bar
############################################################################
set-option -g status-utf8 on
set-option -g status-justify right
set-option -g status-bg black # colour213 # pink
set-option -g status-fg cyan
set-option -g status-interval 5
set-option -g status-left-length 30
set-option -g status-left '#[fg=magenta]» #[fg=blue,bold]#T#[default]'
set-option -g status-right '#[fg=red,bold][[ #(git branch) branch ]] #[fg=cyan]»» #[fg=blue,bold]###S #[fg=magenta]%R %m-%d#(acpi | cut -d ',' -f 2)#[default]'
set-option -g visual-activity on

# Titles (window number, program name, active (or not)
set-option -g set-titles on
set-option -g set-titles-string '#H:#S.#I.#P #W #T'


############################################################################
# Unbindings
############################################################################
#unbind [ # copy mode bound to escape key
unbind j
unbind C-b # unbind default leader key
unbind '"' # unbind horizontal split
unbind %   # unbind vertical split


############################################################################
# Bindings
############################################################################
# reload tmux conf
bind-key r source-file ~/.tmux.conf

#bind Escape copy-mode

# new split in current pane (horizontal / vertical)
bind-key - split-window -v # split pane horizontally
bind-key \ split-window -h # split pane vertically

# list panes
bind-key Space list-panes

# break-pane
bind-key Enter break-pane

# join-pane [-dhv] [-l size | -p percentage] [-s src-pane]
# [-t:dst-window.dst-pane] (destination window (dot) destination pane
#                (alias: joinp)
#
#bind C-j command-prompt "joinp"
#bind C-j command-prompt "join-pane"
#bind-key j command-prompt "join-pane -s '%%'"
#bind-key j command-prompt "joinp -t:0"
bind-key Space command-prompt "joinp -t:%%" # %% = prompt for window.pane [-V|H] # vert|hor split

#previous pane
bind-key -n C-up prev
bind-key -n C-left prev

#next pane
bind-key -n C-right next
bind-key -n C-down next

############################################################################
# windows
############################################################################
set-window-option -g window-status-current-bg red
bind C-j previous-window
bind C-k next-window
bind-key C-a last-window # C-a C-a for last active window
bind A command-prompt "rename-window %%"
# By default, all windows in a session are constrained to the size of the 
# smallest client connected to that session, 
# even if both clients are looking at different windows. 
# It seems that in this particular case, Screen has the better default 
# where a window is only constrained in size if a smaller client 
# is actively looking at it.
setw -g aggressive-resize on

############################################################################
# panes
############################################################################
# Navigation ---------------------------------------------------------------
# use the vim motion keys to move between panes
bind-key h select-pane -L
bind-key j select-pane -D
bind-key k select-pane -U
bind-key l select-pane -R

# Resizing ---------------------------------------------------------------
bind-key C-h resize-pane -L
bind-key C-j resize-pane -D
bind-key C-k resize-pane -U
bind-key C-l resize-pane -R

# use vim motion keys while in copy mode
setw -g mode-keys vi

############################################################################
# layouts
############################################################################
bind o select-layout "active-only"
bind M-- select-layout "even-vertical"
bind M-| select-layout "even-horizontal"
bind M-r rotate-window


# focus on first window
# select-window -t 0
```

> 参见：https://github.com/tmux/tmux/

> 参见：https://tmux.github.io/
