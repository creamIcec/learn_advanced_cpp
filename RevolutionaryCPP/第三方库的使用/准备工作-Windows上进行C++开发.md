> 众所周知, `Windows`并不是非常适合用于原生开发。解决方案是, 将`Unix`环境移植到`Windows`上, 例如下面的`MingW`。

*注: 本篇用于记录环境配置, 便于日后使用*

"原生"C++开发需要针对不同平台编译不同的第三方库。在`Windows`上, 我们使用`MingW`作为构建系统。开始使用第三方库之前, 如果之前没有安装过`MingW`或`CMake`, 我们需要进行下面的工作。

### 安装`Mingw`构建系统

从下面的链接下载安装器:

[MSYS2](https://github.com/msys2/msys2-installer/releases/download/2025-02-21/msys2-x86_64-20250221.exe)

运行安装器。

### 配置`Mingw`构建系统

运行安装路径根目录下的`ucrt64.exe`, 在其中的命令行内执行包管理指令安装附加组件:

```bash
pacman -S mingw-w64-ucrt-x86_64-gcc
pacman -S mingw-w64-x86_64-make
pacman -S mingw-w64-x86_64-gdb
```

在这里，我们安装的分别是:
- 用于编译源代码的`gcc`编译器
- 用于生成输出程序的`make`生成器
- 用于调试的`gdb`调试器

没错, 就是`pacman`, 和我们在ArchLinux上使用的包管理器是同一个。(没想到能在Arch以外的地方见到`pacman`，好亲切🥰)

执行完毕后, 将下面的目录添加到环境变量, 便于后续程序使用:

`<我们的mingW安装路径根目录/ucrt64/bin`
`<我们的mingW安装路径根目录>/mingw64/bin`

### 安装`cmake`

通过下面的链接安装`cmake`:

[CMake](https://github.com/Kitware/CMake/releases/download/v4.0.1/cmake-4.0.1-windows-x86_64.msi)

运行安装程序, 一路Next直到安装结束。




