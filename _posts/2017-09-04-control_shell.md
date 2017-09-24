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



### 捕获信号

`trap`命令允许我们来指定shell脚本要监看并从shell中拦截的Linux信号。

格式为：

```shell
trap commands signals
```

下面展示一个简单的例子，看如何使用`trap`命令忽略`SIGINT`信号，并控制脚本的行为。

```shell
wangsx@SC-201708020022:~/tmp$ cat test1.sh
#!/bin/bash
# Testing signal trapping
#
trap "echo ' Sorry! I have trapped Ctrl-C'" SIGINT
#
echo This is a test script
#
count=1
while [ $count -le 10 ]
do
    echo "Loop #$count"
    sleep 1
    count=$[ $count + 1 ]
done
#
echo "This is the end of the test script"
#
```

来运行测试一下：

```shell
wangsx@SC-201708020022:~/tmp$ ./test1.sh
This is a test script
Loop #1
Loop #2
Loop #3
Loop #4
Loop #5
Loop #6
^C Sorry! I have trapped Ctrl-C
Loop #7
Loop #8
Loop #9
Loop #10
This is the end of the test script
```



### 捕获信号

我们也可以在shell脚本退出时进行捕获。这是**在shell完成任务时执行命令的一种简便方法**。

```shell
wangsx@SC-201708020022:~/tmp$ cat test2.sh
#!/bin/bash
# Trapping the script exit
#
trap "echo Goodbye..." EXIT
#
count=1
while [ $count -le 5 ]
do
    echo "Loop #$count"
    sleep 1
    count=$[ $count + 1 ]
done
#

wangsx@SC-201708020022:~/tmp$ ./test2.sh
Loop #1
Loop #2
Loop #3
Loop #4
Loop #5
Goodbye...
```

当该脚本运行到退出位置，捕获就触发了，shell会执行在`trap`命令行指定的命令。就算提取退出，也能够成功捕获。



### 修改或移除捕获

想在不同的位置进行不同的捕获处理，只需要重新使用带新选项的`trap`命令。

```shell
wangsx@SC-201708020022:~/tmp$ cat test3.sh
#!/bin/bash
# Modifying a set trap
#
trap "echo ' Sorry... Ctrc-C is trapped.'" SIGINT # SIGINT是退出信号
#
count=1
while [ $count -le 5 ]  # 当count<5的时候
do
    echo "Loop #$count"
    sleep 1    # 睡1秒
    count=$[ $count + 1 ]
done

#
trap "echo ' I modified the trap!'" SIGINT
#
count=1
while [ $count -le 5 ]  # 当count<5的时候
do
    echo "Loop #$count"
    sleep 1    # 睡1秒
    count=$[ $count + 1 ]
done
wangsx@SC-201708020022:~/tmp$ ./test3.sh
Loop #1
Loop #2
Loop #3
^C Sorry... Ctrc-C is trapped.
Loop #4
Loop #5
Loop #1
Loop #2
Loop #3
^C I modified the trap!
Loop #4
Loop #5
```

相当于两次不同的捕获。

我们也可以删除已经设置好的捕获。

```shell
wangsx@SC-201708020022:~/tmp$ cat test3.sh
#!/bin/bash
# Modifying a set trap
#
trap "echo ' Sorry... Ctrc-C is trapped.'" SIGINT # SIGINT是退出信号  在这里设置捕获
#
count=1
while [ $count -le 5 ]  # 当count<5的时候
do
    echo "Loop #$count"
    sleep 1    # 睡1秒
    count=$[ $count + 1 ]
done

#
trap -- SIGINT # 在这里删除捕获
echo "I modified the trap!"
#
count=1
while [ $count -le 5 ]  # 当count<5的时候
do
    echo "Loop #$count"
    sleep 1    # 睡1秒
    count=$[ $count + 1 ]
done
wangsx@SC-201708020022:~/tmp$ ./test3.sh
Loop #1
Loop #2
Loop #3
Loop #4
Loop #5
I modified the trap!
Loop #1
Loop #2
Loop #3
^C
```

信号捕获被移除之后，脚本会按照原来的方式处理`SIGINT`信号。所以使用`Ctrl+C`键时，脚本运行会退出。当然，如果是在这个信号捕获移除前接受到`SIGINT`信号，那么脚本还是会捕获。（因为shell脚本运行是按步的，前面没有接收到信号捕获的移除，自然不会实现信号捕获的移除）



## 以后台模式运行脚本

### 后台运行脚本

以后台模式运行shell脚本非常简单，只要再命令后加`&`符就可以了。

```shell
wangsx@SC-201708020022:~$ cat test4.sh
#!/bin/bash
# Testing running in the background
#
count=1
while [ $count -le 10 ]
do
    sleep 1
    count=$[ $count + 1 ]
done
wangsx@SC-201708020022:~$ ./test4.sh &
[1] 69
```

当添加`&`符号后，命令和bash shell会分离而作为一个独立的后台进行运行。并返回作业号（方括号内）和进程ID（PID），Linux系统上运行的每一个进程都必须有一个唯一的PID。

当后台进程结束后，它会在终端显示出消息：

```shell
[1]   已完成               ./test4.sh
```

需要注意的是，当后台程序运行时，它仍然会使用终端显示器显示`STDOUT`和`STDERR`消息。最好是将`STDOUT`和`STDERR`进行重定向。



### 运行多个后台作业

我们可以在命令提示符中同时启动多个后台作用，然后用`ps`命令查看。

```shell
wangsx@SC-201708020022:~$ ./test4.sh &
[1] 117
wangsx@SC-201708020022:~$ ./test5.sh &
[2] 122
wangsx@SC-201708020022:~$ ./test6.sh &
[3] 128
wangsx@SC-201708020022:~$ ps
  PID TTY          TIME CMD
    2 tty1     00:00:00 bash
  117 tty1     00:00:00 test4.sh
  122 tty1     00:00:00 test5.sh
  128 tty1     00:00:00 test6.sh
  135 tty1     00:00:00 sleep
  136 tty1     00:00:00 sleep
  137 tty1     00:00:00 ps
  138 tty1     00:00:00 sleep
```

我们特别需要注意，如果终端退出，后台程序也会随之退出。

### 在非控制台下运行脚本

如果我们不想出现终端退出后台程序退出的情况，可以使用`nohup`命令来实现。

**`nohup`命令运行了另外一个命令来阻断所有发送给该进程的`SIGHUP`信号。这会在退出终端会话时阻止进程退出。**

其格式如下：

```shell
wangsx@SC-201708020022:~$ nohup ./test4.sh  &
[1] 156
wangsx@SC-201708020022:~$ nohup: 忽略输入并把输出追加到'nohup.out'
```

由于`nohup`命令会解除终端与进程的关联，进程也就不再同`STDOUT`和`STDERR`联系在一起。它会自动将这两者重定向到名为`nohup.out`的文件中。

