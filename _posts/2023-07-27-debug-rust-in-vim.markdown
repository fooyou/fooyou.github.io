---
layout: post
title: 在 vim 中调试 Rust 代码：使用 TermDebug 和 vimspector
category: Document
tags: rust vim debug
date: 2023-07-27 17:07:48
published: true
summary: 
cover: 
comment: true
---

## 概述

在 Vim/Neovim 中，为喜欢的编程语言设置 DAP（Debug Adapter Protocol）来完成代码调式并不是非常直接的。像 vimspector 和 nvim-dap 这样用于调式的插件大家都能搜到，但设置起来还是需要下一些功夫的，不过对于真正喜欢用 vim 编程的人来说，总会把它弄好的。

本文将讲述如何在 vim/Neovim 中设置 vimspector 和 Termdebug 来对 Rust 代码进行调试。

## cargo.toml

本文中的代码使用 cargo 来管理项目和包。为确保调式 symbols 能正确生成，可在 cargo.toml 里添加几行，如下：

```toml
[profile.dev]
opt-level = 0
debug = true

[profile.release]
opt-level = 3
debug = false
```

## vimspector

它是 vim 的插件，可以使用 vim-plug Vundle 等管理工具进行安装，在 .vimrc/init.vim 中添加如下行进行安装：

```vim
Plug 'puremourning/vimspector'
```

Rust 支持 GDB 和 CodeLLDB，目前推荐的是 CodeLLDB，使用如下命令进行安装（或者，在调试时会自动安装）：

```vim
:VimspectorInstall CodeLLDB
```

## lldb-vscode.json

```json
{
  "adapters": {
    "lldb-vscode": {
      "variables": {
        "LLVM": {
          "shell": "brew --prefix llvm"
        }
      },
      "attach": {
        "pidProperty": "pid",
        "pidSelect": "none"
      },
      "command": [
        "${LLVM}/bin/lldb-vscode"
      ],
      "env": {
        "LLDB_LAUNCH_FLAG_LAUNCH_IN_TTY": "YES"
      },
      "name": "lldb"
    }
  }
}
```

## .vimrc/init.vim

```vim
let g:vimspector_enable_mappings = 'HUMAN'
" nmap <leader>dd :call vimspector#Launch()<CR>
" nmap <leader>dx :VimspectorReset<CR>
" nmap <leader>de :VimspectorEval
" nmap <leader>dw :VimspectorWatch
" nmap <leader>do :VimspectorShowOutput
```

在 Rust 项目根目录（和 Cargo.toml 同级）创建 .vimspector.json 来配置 adapter:

## .vimspector.json

```
{
  "configurations": {
    "launch": {
      "adapter": "CodeLLDB",
      "configuration": {
        "request": "launch",
        "program": "${workspaceRoot}/target/debug/${binaryName}",
        "args": ["*${ProgramArgs}"]
      }
    }
  }
}
```

按 F5 即可进入调试

> https://alpha2phi.medium.com/setting-up-neovim-for-rust-debugging-termdebug-and-vimspector-df749e1ba47c
