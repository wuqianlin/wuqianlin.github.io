---
title: linux namespace 资源隔离
date: 2021-01-25 18:08:00 +08:00
modified: 2021-01-25 18:08:00 +08:00
tags: [linux, namespace]
---
# Linux 操作系统原理 — Namespace 资源隔离

Linux Namespace (命名空间) 是一种操作系统层级的资源隔离技术，能够将 Linux 全局资源，划分为 namespace 范围内的资源，而且不同 namespace 间的资源彼此透明，不同 namespace 里的进程无法感知到其它 namespace  里面的进程和资源。



Linux namespace 实现了6项资源隔离，基本上涵盖了一个小型操作系统的运行要素，包括主机名、用户权限、文件系统、网络、进程号、进程间通信。Linux namespace 是容器技术（e.g. docker, podman）的底层实现基础。



| Namespace | 系统调用参数  | 隔离内容                   | 内核版本 |
| --------- | ------------- | -------------------------- | -------- |
| UTS       | CLONE_NEWUTS  | 主机名和域名               | 2.6.9    |
| IPC       | CLONE-NEWIPC  | 信号量、消息队列和共享内存 | 2.6.9    |
| PID       | CLONE_NEWIPD  | 进程编号                   | 2.6.24   |
| Network   | CLONE_NEWNET  | 网络设备、网络栈、端口等   | 2.6.29   |
| Mount     | CLONE_NEWNS   | 挂载点（文件系统）         | 2.4.9    |
| User      | CLONE_NEWUSER | 用户和用户组               | 3.8      |

这 6 项资源隔离分别对应 6 种系统调用，将这些系统调用传输传入 clone 函数就可以建立一个容器了。一个容器进程还可以再 clone 出一个容器进程，完成容器的嵌套。

```c
int clone(int (*child_func)(void *), void *child_stack, int flags, void *arg);
```

通过文件 文件 /proc/[pid]/ns 可以查看指定进程运行在哪些 namespace 中：

```
root@node75 ~]# ls -l /proc/$$/ns
总用量 0
lrwxrwxrwx 1 root root 0 1月  25 16:35 ipc -> ipc:[4026531839]
lrwxrwxrwx 1 root root 0 1月  25 16:35 mnt -> mnt:[4026531840]
lrwxrwxrwx 1 root root 0 1月  25 16:35 net -> net:[4026531968]
lrwxrwxrwx 1 root root 0 1月  25 16:35 pid -> pid:[4026531836]
lrwxrwxrwx 1 root root 0 1月  25 16:35 user -> user:[4026531837]
lrwxrwxrwx 1 root root 0 1月  25 16:35 uts -> uts:[4026531838]
[root@node75 ~]# 
```



## Linux Namespace 的类型

- **UTS namespace** 提供了 hostname 和 domain 的隔离，这样每个容器就拥有独立的主机名和域名了，在网络上就可以被视为一个独立的节点，在容器中对 hostname 的命名不会对宿主机造成任何影响。
- **PID namespace** 完成的是进程号的隔离，保证了容器的 init 进程是以 1 号进程来启动的。
- **IPC namespace** 实现了进程间通信的隔离，包括常见的几种进程间通信机制，例如：信号量，消息队列和共享内存。我们知道，要完成 IPC，需要申请一个全局唯一的标识符，即 IPC 标识符，所以 IPC 资源隔离主要完成的就是隔离 IPC 标识符。
- **Mount namespace** 通过隔离文件系统的挂载点来达到对文件系统的隔离。保证了容器看到的文件系统的视图，是容器镜像提供的一个文件系统，也就是说它看不见宿主机上的其它文件，除了通过 -v 参数 bound 的那种模式，是可以把宿主机上面的一些目录和文件，让它在容器里面可见的；
- **Network namespace** 实现了操作系统层面的网络资源隔离，包括网络设备接口、IPv4 和 IPv6 协议栈，IP 路由表，防火墙，/proc/net 目录，/sys/class/net 目录，Sockets 套接字等资源。同一个网络设备只能位于一个 Network namespace 中，不同 namespace 中的网络设备可以利用 veth pair 进行桥接。
- **User namespace** 主要隔离了安全相关的标识符和属性，包括 User ID、User Group ID、root 目录、key 以及特殊权限。