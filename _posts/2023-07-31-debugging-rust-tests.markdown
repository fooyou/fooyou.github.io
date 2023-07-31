---
layout: post
title: 调试 Rust Test
category: Document
tags: rust test debug
date: 2023-07-31 13:07:33
published: true
summary: 
cover: 
comment: true
---

如果你的 Rust 在 cargo test 有失败的用例，那么可能你需要 Debug 你的测试用例，尤其是当使用 C 级别绑定时，你可能会想能跳到 gdb session 里取调试你的代码。

本文将告诉你怎么使用 rust-gdb 来调试测试用例。

## Step 1

使用 `cargo test` 来找出带调试测试用例的执行文件，如下：

```bash
$ cargo test
   Compiling cutr v0.1.0 (/home/joshua/workspace/rust-tutorials/command_line/08.cutr)
     Running unittests src/lib.rs (target/debug/deps/cutr-9da6f31d175dd591)

running 3 tests
test unit_test::test_extract_chars ... ok
test unit_test::test_extract_fields ... ok
test unit_test::test_parse_pos ... ok

test result: ok. 3 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.01s

     Running unittests src/main.rs (target/debug/deps/cutr-3c640cc4436b9d40)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests cutr

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

如上，找到测试用例执行文件：`target/debug/deps/cutr-9da6f31d175dd591`

## Step 2

使用前文设置好的 TermDebug，或是直接使用 rust-gdb 开开启调试

```bash
$ rust-gdb target/debug/deps/cutr-9da6f31d175dd591
```

打上断点，并运行：

```gdb
(gdb) b lib.rs:173
Breakpoint 1 at 0x7460a: file src/lib.rs, line 173.
(gdb) run --test
```

## Step 3

然后使用 gdb 的操作命令进行 debug 就行了。
