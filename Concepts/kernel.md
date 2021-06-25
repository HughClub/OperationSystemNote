## kernel

### 系统结构

![MS-DOS-structure](kernel.assets/image-20210219114855144.png)

![traditional-UNIX](kernel.assets/image-20210219114949261.png)

![classical-microkernel](kernel.assets/image-20210219115031924.png)

#### 混合系统

##### MacOS

![MacOSX](kernel.assets/image-20210219121818823.png)

1.  Mach 微内核
    1.  内存管理
    2.  远程过程调用 RPC
    3.  进程间通信 IPC
    4.  线程调度
2.  BSD Unix内核
    1.  CLI
    2.  Net/FileSystem
    3.  POSIX API
3.  I/O Kit
    1.  kernel extension

##### IOS

![Apple-IOS](kernel.assets/image-20210219122146111.png)

>   Cocoa 规定用于 Objective-C 的 API

##### Android

![Android](kernel.assets/image-20210219122325242.png)



