---
title: 控制脚本
date: 2017-09-04
categories: Linux
tags:
- linux
- shell

---



> **内容**
>
> - 处理信号
> - 以后台模式运行脚本
> - 禁止挂起
> - 作业控制
> - 修改脚本优先级
> - 脚本执行自动化

<!-- more -->

除了在命令行界面世界运行脚本，还存在一些方法：**向脚本发送信号、修改脚本的优先级以及在脚本运行时切换到运行模式**。

下面逐一讲述。

## 处理信号

Linux利用信号与运行在系统中的进程进行通信。我们可以通过对脚本编程，使其在收到特定信号时执行某些命令，从而实现对脚本运行的控制。

### Linux信号

Linux和应用程序可以生成超过30个信号。下面列出最常见的系统信号。

|  信号  |    值    |       描述        |
| :--: | :-----: | :-------------: |
|  1   | SIGHUP  |      挂起进程       |
|  2   | SIGINT  |      终止进程       |
|  3   | SIGQUIT |      停止进程       |
|  9   | SIGKILL |     无条件终止进程     |
|  15  | SIGTERM |     尽可能终止进程     |
|  17  | SIGSTOP | 无条件停止进程，但不是终止进程 |
|  18  | SIGTSTP | 停止或暂停进程，但不是终止进程 |
|  19  | SIGCONT |    继续运行停止的进程    |



默认情况下，bash shell会忽略收到的任何`SIGQUIT`和`SIGTERM`信号（所以交互式shell不会被终止）。但是bash shell会处理收到的`SIGHUP`和`SIGINT`信号。

Shell会将这些信号传给shell脚本程序来处理。而shell脚本默认是忽略这些信号的，为了避免它，我们可以在脚本中加入识别信号的代码，并执行命令来处理信号。



### 生成信号

键盘上的组合可以生成两种基本的Linux信号。它在停止或暂停失控程序时非常有用。

1. 中断程序： 使用`Ctrl`+`C`，它会发送`SIGINT`信号。

   ~~（测试没起作用，尴尬了～）~~

2. 暂停进程：使用`Ctrl`+`Z`，它会发送`SIGTSTP`信号。

   ```shell
   wsx@wsx-ubuntu:~$ sleep 1000
   ^Z
   [2]+  已停止               sleep 1000

   ```

**注意**：停止进程会让程序继续保留在内存中，并能从上次停止的位置继续运行。

方括号中的数字是shell自动为程序分配的*作业号*。shell将shell中运行的每个进程成为*作业*，并为其分配唯一的作业号。

退出shell时发现有停止的进程，用`ps`命令查看

```shell
wsx@wsx-ubuntu:~$ exit
exit
有停止的任务。
wsx@wsx-ubuntu:~$ ps -l
F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
0 S  1000  5438  5433  0  80   0 -  6153 wait   pts/4    00:00:00 bash
0 T  1000  5452  5438  0  80   0 -  2258 signal pts/4    00:00:00 sleep
0 T  1000  5456  5438  0  80   0 -  2258 signal pts/4    00:00:00 sleep
4 R  1000  5525  5438  0  80   0 -  7665 -      pts/4    00:00:00 ps

```

在表示进程状态的S列中，`ps`命令将已经停止作业的状态显示为`T`。这说明命令要么被跟踪，要么被停止了。

如果你仍想退出shell，只需要再输入一遍`exit`。也可以用`kill`生成`SIGKILL`信号标识上`PID`杀死进程。

```shell
wsx@wsx-ubuntu:~$ kill 5456
wsx@wsx-ubuntu:~$ kill -9 5456
wsx@wsx-ubuntu:~$ kill -9 5452
[1]-  已杀死               sleep 1000
[2]+  已杀死               sleep 1000

```


待续
