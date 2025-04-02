我们以前都没有想过main()的返回值代表什么，如何使用它们。经过研究，我们得到了下面的结论:

### main()函数的`return`返回值代表什么?

这个返回值遵循一定的**协议**。在一般的协议中，我们一般使用`0`表示正常运行结束退出，这也是我们大多数时候会使用的返回值:

```c++
return 0;
```

但是，除了`0`以外，我们也可以使用其他的数字作为返回值。下面分为`Windows`和`Linux`两个系统进行说明:

#### Windows

Windows的名义规范(很多应用都在使用嘛?) 退出代码可以在下面的链接查询:

[Win32 System Error Codes](https://learn.microsoft.com/en-us/windows/win32/debug/system-error-codes--0-499-)

这些常量在`WinError.h`头文件中定义，使用时，我们有两种方式，一种是直接`return <错误码>`, 另一种如下:

```c++
#include <cstdlib>

int main(){
	exit(1);
}
```

通过包含`cstdlib`头文件来使用`exit(<错误码>)`函数退出。

除此以外，`Windows`还有一套`NTSTATUS VALUES`。举例说明, 如果我们使用`g++`编译下面的程序:
```c++
int main(){  
    int a = 1;  
    int c = a / 0;  
    return 0;  
}
```

我们将会得到一个运行时错误:
![[Pasted image 20250402160045.png]]
在对应的网站中, 我们可以找到`0xC0000094`错误代码对应的是**除零错误**。链接:

[NT Status Values](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-erref/596a1078-e883-4972-9bbc-49e60bebca55)

`NTSTATUS VALUES`也可通过`return`的方式或者`exit`方式来使用。

以上就是`Windows`平台的两种主要错误返回机制。不过，有一点值得注意的是，上面所说的两套机制似乎是相互兼容的。下面的链接说明了两者的联系和区别(网页中还提到了另一种返回码`hresult`，这里我们暂不涉及): 

https://jpassing.com/2007/08/20/error-codes-win32-vs-hresult-vs-ntstatus/

简单来说，两套机制的混合是`NT`内核发展过程中的历史和兼容问题(我们都知道Windows对于老程序的兼容性非常好)，我们也可以通过同样的方式来取的这两套机制返回的错误码。(一种机制取得两套协议下的代码，是否会导致混淆?) 具体获取方式将会在下面说明。在说明之前，先看看`Linux`的错误码定义。

#### Linux

在`Linux`中，类似地，我们也有一套机制来处理返回值, 参考下面的链接:

[Exit Codes](https://tldp.org/LDP/abs/html/exitcodes.html)

同时，`Linux`也在`sysexits.h`头文件中定义了一套返回码:

```
$ find /usr -name sysexits.h
/usr/include/sysexits.h
$ cat /usr/include/sysexits.h

/*
 * Copyright (c) 1987, 1993
 *  The Regents of the University of California.  All rights reserved.

 (A whole bunch of text left out.)

#define EX_OK           0       /* successful termination */
#define EX__BASE        64      /* base value for error messages */
#define EX_USAGE        64      /* command line usage error */
#define EX_DATAERR      65      /* data format error */
#define EX_NOINPUT      66      /* cannot open input */    
#define EX_NOUSER       67      /* addressee unknown */    
#define EX_NOHOST       68      /* host name unknown */
#define EX_UNAVAILABLE  69      /* service unavailable */
#define EX_SOFTWARE     70      /* internal software error */
#define EX_OSERR        71      /* system error (e.g., can't fork) */
#define EX_OSFILE       72      /* critical OS file missing */
#define EX_CANTCREAT    73      /* can't create (user) output file */
#define EX_IOERR        74      /* input/output error */
#define EX_TEMPFAIL     75      /* temp failure; user is invited to retry */
#define EX_PROTOCOL     76      /* remote error in protocol */
#define EX_NOPERM       77      /* permission denied */
#define EX_CONFIG       78      /* configuration error */

#define EX__MAX 78      /* maximum listed value */
```

那么，这些代码除了给我们人使用以外，在程序的自动化执行过程中(例如这个返回错误码的进程是另一个进程的子进程，另一个进程获得了这个错误码)，有哪些用途呢？


