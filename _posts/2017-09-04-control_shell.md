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

## 在非控制台下运行脚本

如果我们不想出现终端退出后台程序退出的情况，可以使用`nohup`命令来实现。

**`nohup`命令运行了另外一个命令来阻断所有发送给该进程的`SIGHUP`信号。这会在退出终端会话时阻止进程退出。**

其格式如下：

```shell
wangsx@SC-201708020022:~$ nohup ./test4.sh  &
[1] 156
wangsx@SC-201708020022:~$ nohup: 忽略输入并把输出追加到'nohup.out'
```

由于`nohup`命令会解除终端与进程的关联，进程也就不再同`STDOUT`和`STDERR`联系在一起。它会自动将这两者重定向到名为`nohup.out`的文件中。



## 作业控制

启动、停止终止以及恢复作业统称为**作业控制**。我们可以通过这几种方式完全控制shell环境中所有的进程的运行方式。

### 查看作业

作业控制的**关键命令**是`jobs`命令。它允许查看shell当前正在处理的作业。

```shell
wangsx@SC-201708020022:~$ cat test10.sh
#!/bin/bash
# Test job control
#
echo "Script Process ID: $$"
#
count=1
while [ $count -le 10 ]
do
    echo "Loop #$count"
    sleep 10
    count=$[ $count + 1 ]
done
#
echo "End of script..."
#
wangsx@SC-201708020022:~$ ./test10.sh
Script Process ID: 27
Loop #1
Loop #2
^Z^C # 我的ctrl+z好像不起作用
```

脚本用`$$`变量来显示系统分配给脚本的PID。使用Ctrl+Z组合键来停止脚本（我的在这不起作用~之前好像也是）。

我们使用同样的脚本，利用`&`将另外一个作业作为后台进程启动。我们通过`jobs -l`命令查看作业的PID。

```shell
wangsx@SC-201708020022:~/tmp$ ./test10.sh > test10.out &
[1] 121
wangsx@SC-201708020022:~/tmp$ jobs -l
[1]+   121 运行中               ./test10.sh > test10.out &
```

下面看`jobs`命令的一些参数：

| 参数   | 语法                       |
| ---- | ------------------------ |
| -l   | 列出进程的PID以及作业号            |
| -n   | 只列出上次shell发出的通知后改变了状态的作业 |
| -p   | 只列出作业的PID                |
| -r   | 只列出运行中的作业                |
| -s   | 只列出已停止的作业                |

如果仔细注意的话，我们发现作业号后面有`+`号。带加号的作业会被当成默认作业。当前的默认作业完成处理后，带减号的作业成为下一个默认作业。任何时候只有一个带加号的作业和一个带减号的作业。

```shell
wangsx@SC-201708020022:~/tmp$ jobs -l
[1]    132 运行中               ./test10.sh > test10.out &
[2]-   134 运行中               ./test10.sh > test10.out &
[3]+   136 运行中               ./test10.sh > test10.out &
```

可以发现最好运行的脚本输出排在最前面。

我们调用了`kill`命令向默认进程发送了一个`SIGHUP`信号，终止了该作业。

```shell
wangsx@SC-201708020022:~/tmp$ ./test10.sh > test10.out &
[1] 165
wangsx@SC-201708020022:~/tmp$ ./test10.sh > test10.out &
[2] 167
wangsx@SC-201708020022:~/tmp$ ./test10.sh > test10.out &
[3] 169
wangsx@SC-201708020022:~/tmp$ jobs -l
[1]    165 运行中               ./test10.sh > test10.out &
[2]-   167 运行中               ./test10.sh > test10.out &
[3]+   169 运行中               ./test10.sh > test10.out &
wangsx@SC-201708020022:~/tmp$ kill 169
wangsx@SC-201708020022:~/tmp$ jobs -l
[1]    165 运行中               ./test10.sh > test10.out &
[2]-   167 运行中               ./test10.sh > test10.out &
[3]+   169 已终止               ./test10.sh > test10.out
wangsx@SC-201708020022:~/tmp$ jobs -l
[1]-   165 运行中               ./test10.sh > test10.out &
[2]+   167 运行中               ./test10.sh > test10.out &
```

### 重启停止的作业

我们可以将已经停止的作业作为后台进程或者前台进程重启。前台进程会接管当前工作的终端，所以使用时需要注意。

要以后台模式重启一个作业，可以用`bg`命令加上作业号（我的Window10子系统好像确实不能使用Ctrl+Z的功能，有兴趣可以自己测试一下）。

```shell
wangsx@SC-201708020022:~/tmp$ ./test10.sh
Script Process ID: 13
Loop #1
^ZLoop #2
^C
wangsx@SC-201708020022:~/tmp$ bg
bash: bg: 当前: 无此任务
wangsx@SC-201708020022:~/tmp$ jobs
```

如果是默认作业，只需要使用`bg`命令。如果有多个作业，你得在`bg`命令后加上作业号。



## 调整谦让度

在Linux中，内核负责将CPU时间分配给系统上运行的每一个进程。**调度优先级**是内核分配给进程的CPU时间。在Linux系统中，由shell启动的所有进程的调度优先级默认都是相同的。

调度优先级是一个整数值，从-20（最高）到+19（最低）。默认bash shell以优先级0来启动所有进程。

我们可以使用`nice`命令来改变shell脚本的优先级。

### nice命令

要让命令以更低的优先级运行，只要用`nice`的`-n`命令行来指定新的优先级级别。

```shell
wsx@ubuntu:~/tmp$ nice -n 10 ./test10.sh > test10.out &
[5] 18953
wsx@ubuntu:~/tmp$ ps -p 18953 -o pid,ppid,ni,cmd
   PID   PPID  NI CMD
 18953  18782  10 /bin/bash ./test10.sh

```

如果想要提高优先级，需要使用超级用户权限。`nice`命令的`-n`选项不是必须的，只需要在破折号后面跟上优先级就行了。

```shell
wsx@ubuntu:~/tmp$ nice -10 ./test10.sh > test10.out &
[6] 18999
[5]   Done                    nice -n 10 ./test10.sh > test10.out
wsx@ubuntu:~/tmp$ ps -p 18999 -o pid,ppid,ni,cmd
   PID   PPID  NI CMD
 18999  18782  10 /bin/bash ./test10.sh
```

### renice命令

有时候我们想要改变系统上已经运行命令的优先级，这是`renice`命令可以做到的。它允许我们指定PID来改变它的优先级。

```shell
wsx@ubuntu:~/tmp$ ./test10.sh & 
[3] 19086
wsx@ubuntu:~$ ps -p 19086 -o pid,ppid,ni,cmd
   PID   PPID  NI CMD
 19086  19070   0 /bin/bash ./test10.sh
wsx@ubuntu:~$ renice -n 10 -p 19086
19086 (process ID) old priority 0, new priority 10
wsx@ubuntu:~$ ps -p 19086 -o pid,ppid,ni,cmd
   PID   PPID  NI CMD
 19086  19070  10 /bin/bash ./test10.sh
```

## 定时运行脚本

Linux系统提供了多个在预定时间运行脚本的方法：`at`命令和`cron`表。

### 用at命令来计划执行任务

`at`命令允许指定Linux系统何时运行脚本。`at`命令会将作业提交到队列中，指定shell何时运行该作业。`at`的守护进程`atd`会以后台模式运行，检查作业队列来运行作业。

`atd`守护进程会检查系统上的一个特殊目录（通常位于`/var/spool/at`）来获取用`at`命令提交的作业。

#### at命令的格式

```shell
at [-f filename] time
```

默认，`at`命令会将`STDIN`的输入放入队列中。我们可以用`-f`参数来指定用于读取命令的文件名。`time`参数指定了Linux系统何时运行该脚本。

`at`命令能识别多种不同的时间格式。

- 标准的小时和分钟格式，比如10:15
- AM/PM指示符，比如10:15 PM
- 特定可命令的时间，比如now, noon, midnight或teatime （4 PM）

除了指定运行时间，还可以指定运行的日期。

- 标准日期格式，比如MMDDYY, MM/DD/YY或DD.MM.YY
- 文本日期，比如Jul 4或Dec 25，加不加年份都可以
- 还可以指定时间增量
  - 当前时间+25min
  - 明天10:15PM
  - 10:15+7天

针对不同的优先级，存在26种不同的作业队列。作业队列通常用小写字母a-z和大写字母A-Z来指代。

作业队列的字母排序越高，作业运行的优先级就越低（更高的nice值）。可以用`-q`参数指定不同的队列字母。

#### 获取作业的输出

Linux系统会将提交作业的用户的电子邮件地址作为STDOUT和STDERR。任何发到STDOUT或STDERR的输出都会通过邮件系统发送给用户。

```shell
# 解决atd没启动的问题
wangsx@SC-201708020022:~/tmp$ sudo /etc/init.d/atd start
```



```shell
wangsx@SC-201708020022:~/tmp$ cat test13.sh
#!/bin/bash
# Test using at command
#
echo "This script ran at $(date +%B%d,%T)"
echo
sleep 5
echo "This is the script's end..."
#
wangsx@SC-201708020022:~/tmp$ at -f test13.sh now
warning: commands will be executed using /bin/sh
job 4 at Tue Sep 26 12:12:00 2017
```

`at`命令会显示分配给作业的作业号以及为作业安排的运行时间。`at`命令利用`sendmail`应用程序来发送邮件。如果没有安装这个工具就无法获得输出，因此在使用`at`命令时，最好在脚本中对STDOUT和STDERR进行重定向。

```shell
wangsx@SC-201708020022:~/tmp$ cat test13b.sh
#!/bin/bash
# Test using at command
#
echo "This script ran at $(date +%B%d,%T)" > test13b.out
echo >> test13b.out
sleep 5
echo "This is the script's end..." >> test13b.out
#
wangsx@SC-201708020022:~/tmp$ at -M -f test13b.sh now
warning: commands will be executed using /bin/sh
job 7 at Tue Sep 26 12:16:00 2017
wangsx@SC-201708020022:~/tmp$ cat test13b.out
This script ran at 九月26,12:16:24

This is the script's end...
```

这里使用了`-M`选项来屏蔽作业产生的输出信息。

#### 列出等待的作业

`atq`命令可以查看系统中有哪些作业再等待。

```shell
wangsx@SC-201708020022:~/tmp$ at -M -f test13b.sh teatime
warning: commands will be executed using /bin/sh
job 11 at Wed Sep 27 16:00:00 2017
Can't open /var/run/atd.pid to signal atd. No atd running?
wangsx@SC-201708020022:~/tmp$ sudo /etc/init.d/atd start
[sudo] wangsx 的密码：
 * Starting deferred execution scheduler atd                                                                     [ OK ]
wangsx@SC-201708020022:~/tmp$ at -M -f test13b.sh teatime
warning: commands will be executed using /bin/sh
job 12 at Wed Sep 27 16:00:00 2017
wangsx@SC-201708020022:~/tmp$ at -M -f test13b.sh tomorrow
warning: commands will be executed using /bin/sh
job 13 at Wed Sep 27 21:44:00 2017
wangsx@SC-201708020022:~/tmp$ at -M -f test13b.sh 13:30
warning: commands will be executed using /bin/sh
job 14 at Wed Sep 27 13:30:00 2017
wangsx@SC-201708020022:~/tmp$ atq
11      Wed Sep 27 16:00:00 2017 a wangsx
12      Wed Sep 27 16:00:00 2017 a wangsx
13      Wed Sep 27 21:44:00 2017 a wangsx
14      Wed Sep 27 13:30:00 2017 a wangsx
```

作业列表中显示了作业号、系统运行该作业的日期和时间以及它所在的队列位置。

#### 删除作业

使用`atrm`命令删除等待中的作业。

```shell
wangsx@SC-201708020022:~/tmp$ atq
11      Wed Sep 27 16:00:00 2017 a wangsx
12      Wed Sep 27 16:00:00 2017 a wangsx
13      Wed Sep 27 21:44:00 2017 a wangsx
14      Wed Sep 27 13:30:00 2017 a wangsx
wangsx@SC-201708020022:~/tmp$ atrm 11
wangsx@SC-201708020022:~/tmp$ atq
12      Wed Sep 27 16:00:00 2017 a wangsx
13      Wed Sep 27 21:44:00 2017 a wangsx
14      Wed Sep 27 13:30:00 2017 a wangsx
```

只能删除自己提交的作业，不能删除其他人的。



### 安排需要定期执行的脚本

如果是需要定期执行的脚本，我们不需要使用`at`不断提交作业，而是可以利用Linux系统的另一个功能。

**Linux系统使用`cron`程序来安排要定期执行的作业。它会在后台运行并检查一个特殊的表（成为cron时间表），以获得已安排执行的作业。**

#### cron时间表

`cron`时间表的格式如下：

```shell
min hour dayofmonth month dayofweek command
```

`cron`时间表允许我们用特定值、取值范围（比如1-5）或者通配符（星号）来指定条目。

例如，我们想在每天的10:15运行一个命令，可以使用：

```shell
15 10 * * * command
```

在其中三个字段使用了通配符，表明`cron`会在每个月的每天的10:15执行该命令。

要指定在每周一4:15PM运行命令，可以使用：

```shell
15 16 * * 1 command
```

可以用三字符的文本值mon,tue,wed,thu,fri,sat,sum或数值（0为周日，6为周六）来指定dayofweek的表项。

`dayofmonth`可以用1-31表示。

```shell
# 怎么在每个月的最后一天执行命令？
00 12 * * * if [`date +%d -d tomorrow` = 01 ]; then ; command
```

命令列表必须指定要运行的命令或脚本的全路径名。

```shell
# 例如
15 10 * * * /home/rich/test4.sh > test4out
```

注意：`corn`会用提交作业的用户账户运行脚本，所以我们在操作指定文件时必须有相应权限。

#### 构建cron时间表

Linux提供了`crontab`命令来处理`cron`时间表。我们可以使用`-l`选项列出时间表。

```shell
wangsx@SC-201708020022:~/tmp$ crontab -l
no crontab for wangsx
```

要添加条目，使用`-e`选项。在添加条目时，`crontab`命令会启动一个文本编辑器，使用已有的`cron`时间表作为文件内容。

#### 浏览cron目录

如果对时间精确性要求不高，用预配置的`cron`脚本目录会更方便。有4个基本目录：hourly, daily, monthly和weekly。

```shell
wangsx@SC-201708020022:~/tmp$ ls /etc/cron.*ly
/etc/cron.daily:
apport      bsdmainutils  man-db   passwd                  upstart
apt-compat  dpkg          mdadm    popularity-contest
aptitude    logrotate     mlocate  update-notifier-common

/etc/cron.hourly:

/etc/cron.monthly:

/etc/cron.weekly:
fstrim  man-db  update-notifier-common
```

如果脚本需要每天运行一次，只要将脚本复制到daily目录，cron每天会执行它。

#### anacron程序

如果提交的作业需要运行时系统处于关机状态，`cron`不会运行那些错过的脚本。为了解决这个问题，`anacron`程序诞生了。

如果`anacron`知道某个作业错过了执行时间，它会尽快运行该作业。这个功能常用于进行常规日志维护的脚本。

`anacron`程序只会处理位于`cron`目录下的程序，比如`/etc/cron.monthly`。它使用时间戳来决定作业是否在正确的计划间隔内运行了，每个`cron`目录都有个时间戳文件，该文件位于`/var/spool/anacron`。

`anacron`程序使用自己的时间表来检查作业目录。

```shell
wsx@ubuntu:~$ sudo cat /var/spool/anacron/cron.monthly
[sudo] password for wsx: 
20170926
wsx@ubuntu:~$ sudo cat /etc/anacrontab
# /etc/anacrontab: configuration file for anacron

# See anacron(8) and anacrontab(5) for details.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
HOME=/root
LOGNAME=root

# These replace cron's entries
1       5       cron.daily      run-parts --report /etc/cron.daily
7       10      cron.weekly     run-parts --report /etc/cron.weekly
@monthly        15      cron.monthly    run-parts --report /etc/cron.monthly

```

`anacron`的时间表的基本格式和`cron`时间表略有不同：

```shell
period delay identifier command
```

`period`定义了作业多久运行一次，以天为单位。`anacron`用此条目来检查作业的时间戳文件。`delay`条目会指定系统启动后`anacron`程序需要等待多少分钟再开始运行错过的脚本。`command`条目包含了`run-parts`程序和一个`cron`脚本目录名。`run-parts`程序负责运行目录中传给它的任何脚本。

注意了，`anacron`不会处理执行时间需求小于一天的脚本，所以它是不会运行`/etc/cron.hourly`的脚本。

`identifier`条目是一种特别的非空字符串，如`cron-weekly`。它用于唯一标识日志消息和错误邮件中的作业。

