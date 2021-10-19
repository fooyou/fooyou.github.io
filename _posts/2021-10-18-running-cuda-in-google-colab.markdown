---
layout: post
title: 在 Google Colab 里运行 CUDA
category: Document
tags: cuda deeplearning
date: 2021-10-18 19:10:18
published: true
summary: 如何在 Colab 里运行多版本的 CUDA
image: pirates.svg
comment: true
---

在 Colab 里使用 CUDA 之前要先在「代码执行程序」-> 「更改运行时类型」里选择使用「GPU」。

Google Colab 里已经内置了 CUDA，可简单通过 nvcc 或 nvidis-smi 检查其版本

```bash
!nvcc --version
```

在当前的 Colab 版本中，使用的是 CUDA 11.1，如下：

```bash
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2020 NVIDIA Corporation
Built on Mon_Oct_12_20:09:46_PDT_2020
Cuda compilation tools, release 11.1, V11.1.105
Build cuda_11.1.TC455_06.29190527_0
```

在 Colab 里开启 CUDA 编程并执行，可以安装 [nvcc4jupyter]() 插件，如下：

```bash
!pip install git+git://github.com/andreinechaev/nvcc4jupyter.git
```

然后，加载插件：

```bash
%load_ext nvcc_plugin
```

编写 CUDA 代码的时候代码透添加：

```bash
%%cu
```

然后就可以像编写 Python 代码一样编写 CUDA C 代码了，例子如下：

```c
%%cu

#include <stdio.h>

#define gpuErrchk(ans) { gpuAssert((ans), __FILE__, __LINE__); }

inline void gpuAssert(cudaError_t code, const char *file, int line, bool abort=true) { 
    if (code != cudaSuccess) {
        fprintf(stderr, "GPUassert: %s %s %d\n", cudaGetErrorString(code), file, line);
        if (abort) exit(code);
    }
}

__global__ void add(int a, int b, int *c) {
    *c = a + b;
}

int main() {
    // - - Host declarations and initializations
    int a, b, c;
    a = 2;
    b = 6;
    // - - Device allocations
    int *d_c; gpuErrchk(cudaMalloc(&d_c, sizeof(int)));
    // - - Kernel execution
    add<<<1, 1>>>(a, b, d_c);
    // gpuErrchk(cudaPeekAtLastError());
    // gpuErrchk(cudaDeviceSynchronize());
    // - - Moving the results from device to host
    gpuErrchk(cudaMemcpy(&c, d_c, sizeof(int), cudaMemcpyDeviceToHost));
    // - - Result printout
    printf(“%d + %d is %d\n”, a, b, c);
    gpuErrchk(cudaFree(d_c));
    return 0;
}
```

请注意如果其他 GPU 不可用，Colab 当前提供的是新的 T4 或 P100 GPU 或者是老点的 K80。CUDA 11 对 3.x 架构只部分支持。

Google Colab 已安装了多个版本的 CUDA，试着输入如下命令：

```bash
%cd /usr/local
```

然后检查当前目录：

```bash
!pwd
```

接着查看所有 CUDA 版本：

```bash
!ls -l | grep cuda
```

当前的 Colab 显示如下：

```bash
lrwxrwxrwx  1 root root   22 Oct  8 13:38 cuda -> /etc/alternatives/cuda
drwxr-xr-x 16 root root 4096 Oct  8 13:29 cuda-10.0
drwxr-xr-x 15 root root 4096 Oct  8 13:32 cuda-10.1
lrwxrwxrwx  1 root root   25 Oct  8 13:38 cuda-11 -> /etc/alternatives/cuda-11
drwxr-xr-x 15 root root 4096 Oct  8 13:34 cuda-11.0
drwxr-xr-x  1 root root 4096 Oct  8 13:36 cuda-11.1
```

可见，cuda 10.0, 10.1, 11.0, 11.1 都有安装。

如果你想激活某个 CUDA 版本（比如在老的显卡，或你使用 TF/Torch 支持较低的版本），只需要删除当前的 cuda 软链接，并重新建立软链接即可，例如我要激活 10.0 版本的 cuda：

```bash
!rm -rf cuda
!ln -s /usr/local/cuda-10.0 /usr/local/cuda
```

或者你也可以使用 update-alternatives 命令来更改。

好了，检查一下 cuda 的软链接

```bash
!stat cuda
```

显示如下：

```bash
  File: cuda -> cuda-10.0
  Size: 9             Blocks: 0          IO Block: 4096   symbolic link
Device: 33h/51d    Inode: 4325381     Links: 1
Access: (0777/lrwxrwxrwx)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2021-10-18 12:15:37.178152339 +0000
Modify: 2021-10-18 12:15:32.440158181 +0000
Change: 2021-10-18 12:15:32.440158181 +0000
 Birth: -
```

当然你也可以下载和安装其他版本的 CUDA，比如安装 CUDA 9.2:

```bash
!wget https://developer.nvidia.com/compute/cuda/9.2/Prod/local_installers/cuda-repo-ubuntu1604-9-2-local_9.2.88-1_amd64 -O cuda-repo-ubuntu1604–9–2-local_9.2.88–1_amd64.deb
!dpkg -i cuda-repo-ubuntu1604–9–2-local_9.2.88–1_amd64.deb
!apt-key add /var/cuda-repo-9–2-local/7fa2af80.pub
!apt-get update
!apt-get install cuda-9.2
```

> https://vitalitylearning.medium.com/running-cuda-in-google-colab-525a92efcf75
