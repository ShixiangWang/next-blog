---
title: Linux结构化命令
date: 2017-08-11
categories:
- Linux
tags:
- bash shell
- shell笔记

---

<!-- more -->

## 条件控制



> **内容**
>
> - 使用if-then语句
> - 嵌套if语句
> - test命令
> - 复合条件测试
> - 使用双方括号和双括号
> - case命令

许多程序要求对shell脚本中的命令施加一些逻辑流程控制。而某些命令会根据条件判断执行相应的命令，这样的命令通常叫做**结构化命令**。从概念上理解，结构化命令是shell脚本的逻辑结构，不像顺序执行shell脚本，而是有组织地执行命令以应对复杂任务需求。

### if-then语句

最基本的结构化命令是if-then语句，它的格式如下：

```shell
if command
then 
	commands
fi
```

**注意**，在其他编程语言中，`if`语句之后的对象是一个等式，等式的结果为`TRUE`或者`FALSE`，但是bash shell中的`if`语句是运行`if`后面的命令，如果该命令的退出状态码是0（命令成功执行），则运行`then`语句后面的命令。`fi`表示`if`语句到此结束。

下面是一个简单的例子：

```shell
wsx@wsx-ubuntu:~/script_learn$ cat test1.sh 
#! /bin/bash
# testing the if statement
if pwd
then 
    echo "It worked"
fi

wsx@wsx-ubuntu:~/script_learn$ chmod u+x test1.sh 
wsx@wsx-ubuntu:~/script_learn$ ./test1.sh 
/home/wsx/script_learn
It worked
```

这个例子中在判断成功执行`pwd`命令后，执行输出文本字符串。

大家可以尝试把`pwd`命令改成随便乱打的字符试试结果。它会显示报错信息，`then`后面的语句也不会执行。

if-then语句的另一种形式：

```shell
if command; then
commands
fi
```

在then部分，我们可以使用多个命令（从格式中command结尾有没有s也可以看出）。

我们再来一个例子：在`if`语句中用`grep`命令在`/etc/passwd`文件中查找某个用户名当前是否在系统上使用。如果有用户使用了哪个登录名，脚本会显示一些文本信息并列出该用户HOME目录的bash文件。

```shell
wsx@wsx-ubuntu:~/script_learn$ cat test3.sh 
#!/bin/bash
# testing multiple commands in the then section
# 
testuser=wsx
#
if grep $testuser /etc/passwd
then
  echo "This is my first command"
  echo "This is my second command"
  echo "I can even put in other commands besides echo:"
  ls -a /home/$testuser/.b*
fi

wsx@wsx-ubuntu:~/script_learn$ chmod u+x test3.sh 
wsx@wsx-ubuntu:~/script_learn$ ./test3.sh 
wsx:x:1000:1000:wsx,,,:/home/wsx:/bin/bash
This is my first command
This is my second command
I can even put in other commands besides echo:
/home/wsx/.bash_history  /home/wsx/.bashrc
/home/wsx/.bash_logout	 /home/wsx/.bashrc-anaconda3.bak

```

如果设置的用户名不存在，那么就没有输出。那么如果在这里显示的一些消息可以说明用户名在系统中未找到，这样可能就会显得更友好。所以接下来看看`if-then-else`语句。

### if-then-else语句

我相信意思非常容易理解，这里较之前我们添加了一个`else`块来处理`if`中命令没有成功执行的步骤。格式为：

```shell
if command
then 
  commands
else commands
fi
```

### 嵌套if

有时我们需要检查脚本代码中的多种条件，可以是用嵌套的`if-then`语句。

处理一个例子：检查`/etc/passwd`文件中是否存在某个用户名以及该用户名的目录是否存在。

```shell
wsx@wsx-ubuntu:~/script_learn$ cat test5.sh 
#!/bin/bash

# Testing nested ifs
#
testuser=NoSuchUser
#
if grep $testuser /etc/passwd
then 
  echo "The user $testuser exits on this system."
else 
  echo "The user $testuser does not exit on this system."
  if ls -d /home/$testuser/
  then
     echo "However, $testuser has a directory."
  fi
fi

wsx@wsx-ubuntu:~/script_learn$ chmod u+x test5.sh 
wsx@wsx-ubuntu:~/script_learn$ ./test5.sh 
The user NoSuchUser does not exit on this system.
ls: 无法访问'/home/NoSuchUser/': 没有那个文件或目录
```

可以使用`else`部分的另一种形式:`elif`。这样我们就不再用书写多个`if-then`语句了。在其他语言中，有的是用`elif`的形式，有的使用`else if`等形式。面对相同内含在不同语言中不同的表示方式，我们需要有意识地区别，以免接触的东西多了可能各种语言代码串写喔。

```shell
if command1
then
	commands
elif command2
then 
	more commands
fi
```

这种表示方式逻辑更为清晰，但是也有点容易让写的人搞混。其实可以看到一个`if`对应一个`fi`。这是一个大的嵌套`if`结构。

**记住**，在`elif`语句中，紧跟其后的`else`语句属于`elif`代码块，而不是属于`if-then`代码块。

### test命令

到此为止，我们很清楚`if`后面跟着的是普通的shell命令，那么我们需要测试其他条件怎么办呢？

`test`命令提供了在`if-then`语句中测试不同条件的途径。如果`test`命令中列出的条件成立，`test`命令就会退出并返回状态码0。这样`if-then`语句就与其他编程语言中的`if-then`语句以类似的方式工作了。

test命令格式：

```
test condition	
```

`condition`是`test`命令要测试的一系列参数和值。如果不写这个`condition`，`test`返回非0，`if`语句跳转到`else`进行执行。

bash shell提供了一种条件测试方法，无需在`if-then`语句中声明`test`命令。

```shell
if [ condition ]
then commands
fi
```

这跟我们其他的编程习惯非常接近。建议使用这种方式。

如果使用`test`命令，需要记住的是各种条件参数。

**数值比较**

| 比较        | 描述         |
| --------- | ---------- |
| n1 -eq n2 | (n1)等于(n2) |
| n1 -ge n2 | 大于或等于      |
| n1 -gt n2 | 大于         |
| n1 -le n2 | 小于或等于      |
| n1 -lt n2 | 小于         |
| n1 -ne n2 | 不等于        |

**字符串比较**

| 比较           | 描述               |
| ------------ | ---------------- |
| str1 = str2  | （str1与str2比较）相同  |
| str1 != str2 | 不同               |
| str1 < str2  | 小                |
| str1 > str2  | 大                |
| -n str1      | 检查string1的长度非0   |
| -z str1      | 检查string1的长度是否为0 |

注意，大于和小于号必须转义；大于和小于顺序和sort命令所采用的不同。

**文件比较**

| 比较              | 描述                |
| --------------- | ----------------- |
| -d file         | 检查file是否存在并是一个目录  |
| -e file         | ～是否存在             |
| -f file         | ～是否存在并是一个文件       |
| -r file         | ～是否存在并可读          |
| -s file         | ～是否存在并非空          |
| -w file         | ～是否存在并可写          |
|                 |                   |
| -x file         | ～是否存在并可执行         |
| -O file         | ～是否存在并属当前用户所有     |
| -G file         | ～是否存在并且默认组与当前用户相同 |
| file1 -nt file2 | 检查file1是否比file2新  |
| file1 -ot file2 | 检查file1是否比file2旧  |

### 复合条件测试

`if-then`语句允许我们使用布尔逻辑来组合测试。可用

- [ condition1] && [ condition2]
- `[ condition1] || [ condition2]`

### if-then的高级特性

- 用于数学表达式的双括号
- 用于高级字符串处理功能的双方括号

**双括号**

命令格式：

```
(( expresiion ))
```

`expression`可以是任意的数学赋值或比较表达式。除了`test`命令使用的标准数学运算符，下面列出了一些其他的：

| 符号     | 描述   |
| ------ | ---- |
| val ++ | 后增   |
| val -- | 后减   |
| ++ val | 先增   |
| -- val | 先减   |
| !      | 逻辑取反 |
| ~      | 位求反  |
| **     | 幂运算  |
| <<     | 左位移  |
| >>     | 右位移  |
| &      | 位布尔和 |
| \|     | 位布尔或 |
| &&     | 逻辑和  |
| \|\|   | 逻辑或  |

看一个例子：

```shell
wsx@wsx-ubuntu:~/script_learn$ cat test23.sh 
#!/bin/bash
# using doble parenthesis
#
val1=10
#
if (( $val1 ** 2 > 90 ))
then
  (( val2 = $val1 ** 2 ))
  echo "The square of $val1 is $val2"
fi

wsx@wsx-ubuntu:~/script_learn$ chmod u+x test23.sh 
wsx@wsx-ubuntu:~/script_learn$ ./test23.sh 
The square of 10 is 100

```

**双方括号**

双方括号命令提供了针对字符串比较的高级特性。命令格式如下：

```
[[ expression ]]
```

双方括号里的`expression`使用了`test`命令中采用的标准字符串比较。但它提供了`test`没有提供的一个特性——模式匹配。

在模式匹配中，可以定义一个正则表达式来匹配字符串值。

```shell
wsx@wsx-ubuntu:~/script_learn$ cat test24.sh 
#! /bin/bash
# using pattern matching
# 
if [[ $USER == r* ]]
then
  echo "Hello $USER"
else 
  echo "Sorry, I do not know you"
fi

wsx@wsx-ubuntu:~/script_learn$ chmod u+x test24.sh
wsx@wsx-ubuntu:~/script_learn$ ./test24.sh 
Sorry, I do not know you
```

上面一个脚本中，我们使用了双等号。双等号将右边的字符串视为一个模式，并将其应用模式匹配规则。

### case命令

有了`case`命令，就不需要写出所有的`elif`语句来不停地检查同一个变量的值了。`case`命令会采用列表格式来检查单个变量的多值。

下面是两个脚本实现相同功能进行对比：

if语句：

```shell
wsx@wsx-ubuntu:~/script_learn$ cat test25.sh
#!/bin/bash
# looking for a possible value
# 
if [ $USER = "rich" ]
then
  echo "Welcome $USER"
  echo "Please enjoy you visit"
elif [ $USER = "barbara" ]
then
  echo "Welcome $USER"
  echo "Please enjoy you visit"
elif [ $USER = "testing" ]
then
  echo "Special testing account"
elif [ $USER = "jessica" ]
then
  echo "Do not forget to logout when you're done"
```

case语句：

```
case variable in
pattern1 | pattern2) commands1;;
pattern3) commands2;;
*) default commands;;
esac
```

上面的实例可以用`case`语句表示为：

```shell
wsx@wsx-ubuntu:~/script_learn$ cat test26.sh
#!/bin/bash
# using the case command
#
case $USER in
rich | barbara)
  echo "Welcome, $USER"
  echo "Please enjoy your visits";;
testing)
  echo "Special testing account";;
jessica)
  echo "Do not forget to log off whe you're done";;
*)
  echo "Sorry, you are not allowed here";;
esac

wsx@wsx-ubuntu:~/script_learn$ chmod u+x test26.sh 
wsx@wsx-ubuntu:~/script_learn$ ./test26.sh
Sorry, you are not allowed here

```

`case`命令会将指定的变量与不同模式进行比较。如果变量和模式是匹配的，那么shell会执行为该模式指定的命令。可以通过竖线操作符在一行中分隔出多个模式。星号会捕获所有与已知模式不匹配的值。注意双分号的使用。

### 小结

> 最基本的命令是`if-then`语句；
>
> 可以拓展`if-then`语句为`if-then-else`语句；
>
> 可以将`if-then-else`语句通过`elif`语句连接起来；
>
> 在脚本中，我们需要测试一种条件而不是命令时，比如数值、字符串内容、文件或目录的状态，`test`命令提供了简单方法；
>
> 方括号是`test`命令统一的特殊bash命令；
>
> 双括号使用另一种操作符进行高级数学运算双方括号允许高级字符串模式匹配运算；
>
> `case`命令是执行多个`if-then-else`命令的简便方式，它会参照一个值列表来检查单个变量的值。

关于结构化命令中循环，将在下次整理的笔记中阐述。



## 循环控制



> **内容**
>
> - for循环语句
> - until迭代语句使用while语句
> - 循环
> - 重定向循环的输出

这一节我们来了解如何重复一些过程和命令，也就是循环执行一组命令直到达到了某个特定条件。

### for命令

基本格式：

```shell
for var in list
do
	commands
done
```

也可以

```shell
for var in list; do
```

分号只用来分隔命令的，让代码更简约。

来个简单例子：

```shell
wsx@wsx-ubuntu:~/script_learn$ cat test1
#!/bin/bash
# basic for command

for test in Alabama Alaska Arizona Arkansas California Colorado
do 
    echo The next state is $test
done

wsx@wsx-ubuntu:~/script_learn$ ./test1
The next state is Alabama
The next state is Alaska
The next state is Arizona
The next state is Arkansas
The next state is California
The next state is Colorado
```

这里操作基本和其他语言一致（格式不同），不多讲啦。

**在读取列表中的复杂值时**，我们可能会遇到问题。比如下面这个例子：

```shell
wsx@wsx-ubuntu:~/script_learn$ cat badtest1 
#!/bin/bash
# another example of how not to use the for command

for test in I don't know if this'll work
do
    echo "word:$test"
done

wsx@wsx-ubuntu:~/script_learn$ ./badtest1 
word:I
word:dont know if thisll
word:work
```

我们可以看到shell看到了列表值中的单引号尝试使用它们来定义一个单独的数据值。

这里有两种解决办法：

- 使用转义字符将单引号转义
- 使用双引号来定义用到单引号的值

我们将这两种解决办法同时用到上个例子：

```shell
wsx@wsx-ubuntu:~/script_learn$ cat test2
#! /bin/bash
# another example of how not to use the for command

for test in I don\'t know if "this'll" work; do
echo  "word:$test"
done
wsx@wsx-ubuntu:~/script_learn$ ./test2
word:I
word:don't
word:know
word:if
word:this'll
word:work
```

我们可能明白了`for`循环是假定每个值是用空格分隔的，所以当有包含空格的数据时，我们需要用双引号括起来。

**通常我们会将列表值存储在一个变量中**，然后通过遍历变量的方式遍历了其内容的的列表。

看看怎么完成这个任务：

```shell
wsx@wsx-ubuntu:~/script_learn$ cat test3
#!/bin/bash

# using a variable to hold the list
list="Alabama Alaska Arizona Arkansas Colorado"
list=$list" Connecticut" # 在尾部拼接文本

for state in $list; do
  echo "Have you ever visited $state?"
done

wsx@wsx-ubuntu:~/script_learn$ ./test3
Have you ever visited Alabama?
Have you ever visited Alaska?
Have you ever visited Arizona?
Have you ever visited Arkansas?
Have you ever visited Colorado?
Have you ever visited Connecticut?
```

注意，代码中还用了另一个赋值语句向`$list`变量包含的已有列表中添加了一个值。这是在已有文本字符串尾部添加文本的一种常用方法。

我们还可以**用命令来输出我们需要的列表内容**：

```shell
wsx@wsx-ubuntu:~/script_learn$ cat test4
#!/bin/bash
# reading values from a file

file="states"

for state in $(cat $file)
do
    echo "Visit beautiful $state"
done

wsx@wsx-ubuntu:~/script_learn$ cat states
Alabama
Alaska
Arizona
Arkansas
Colorado
Connecticut
Delaware
Florida
Georgia
wsx@wsx-ubuntu:~/script_learn$ ./test4
Visit beautiful Alabama
Visit beautiful Alaska
Visit beautiful Arizona
Visit beautiful Arkansas
Visit beautiful Colorado
Visit beautiful Connecticut
Visit beautiful Delaware
Visit beautiful Florida
Visit beautiful Georgia
```



**更改字段分隔符**

环境变量`IFS`，也叫作字段分隔符。它定义了bash shell用作字段分隔符的一系列字符。默认情况下，bash shell会将**空格、制表符和换行符**当作字段分隔符。

如果想修改`IFS`的值，比如使其只能识别换行符，我们可以将下面这行代码加入脚本：

```shell
IFS=$'\n'
```

在处理大量脚本时，我们可能只在某一部分使用其他的分隔符，这时候可以先保存原有的`IFS`值，然后修改，最后恢复：

```shell
IFS.OLD=$IFS
IFS=$'\n'
<在代码中使用新的IFS值>
IFS=$IFS.OLD
```

假如我们要遍历一个文件中用冒号分隔的值：

```shell
IFS=:
```

假如要指定多个`IFS`字符，只要将它们的赋值行串起来：

```shell
IFS=$'\n':;"
```

这个赋值会将换行符、冒号、分号以及双引号作为字段分隔符。



**用通配符读取目录**

我们可以用`for`命令来自动遍历目录中的文件。进行此操作时，必须在文件名或路径名中使用通配符。它会强制shell使用**文件扩展匹配**。文件扩展匹配是生成匹配指定通配符的文件名或路径名的过程。

我拿我的一个目录来尝试一下：

```shell
wsx@wsx-ubuntu:~/script_learn$ cat test5
#!/bin/bash

# iterate through all the files in a directory

for file in /home/wsx/python_learn/*
do
  if [ -d "$file" ]
  then
    echo "$file is a directory"
  elif [ -f "$file" ]
  then
    echo "$file is a file"
  fi
done
wsx@wsx-ubuntu:~/script_learn$ ./test5
/home/wsx/python_learn/athletelist.py is a file
/home/wsx/python_learn/athletemodel.py is a file
/home/wsx/python_learn/ch2_data_input.py is a file
/home/wsx/python_learn/chapter5_first.py is a file
/home/wsx/python_learn/chapter6_first.py is a file
/home/wsx/python_learn/chapter6_second.py is a file
/home/wsx/python_learn/chapter6_third.py is a file
/home/wsx/python_learn/coinFlips.py is a file
/home/wsx/python_learn/Dive_into_python is a directory
```

**注意：**第一个方括号之后和第二个方括号之前必须加上一个空格，否则会报错。

在Linux中，目录名和文件名中包含空格是合法的，所以将`$file`变量用双引号圈起来。当然，大家尽量不要让文件或目录包含空格，不然很容易出问题（命令会把空格当做文件的分隔符）。

### C语言风格的for命令

C语言风格的`for`命令看起来如下：

```shell
for (( a = 1; a < 10; a++ ))
```

值得注意的是，这里有些部分没有遵循bash shell标准的`for`命令：

- 变量赋值可以有空格；
- 条件中的变量不以美元符开头；
- 迭代过程的算式未用`expr`命令格式。

在使用这种格式时要小心，不同的格式不注意就会出错。

下面举个例子：

```shell
wsx@wsx-ubuntu:~/script_learn$ cat test6
#!/bin/bash

# testing the C-style for loop

for (( i=1; i <= 10; i++ ))
do
  echo "The next number is $i"
done

wsx@wsx-ubuntu:~/script_learn$ ./test6
The next number is 1
The next number is 2
The next number is 3
The next number is 4
The next number is 5
The next number is 6
The next number is 7
The next number is 8
The next number is 9
The next number is 10
```



### while命令
`while`命令的格式为：
```shell
while test command
do 
  other commands
done
```

`while`命令某种意义上是`if-then`语句和`for`循环的混杂体。注意，这里`while`后面接的也是命令。`while`命令允许定义一个要测试的命令，然后循环执行一组命令，只要定义的测试命令返回的是退出状态码是0（类似一般语言中 的TRUE）。直到非0时退出循环。

`while`命令中定义的`test command`和`if-then`语句中的格式一模一样。可以使用任何普通的bash shell命令，或者用`test`命令进行条件测试，比如测试变量值。

最常见的用法是用方括号来检查循环命令中用到的`shell`变量的值。

```shell
wangsx@SC-201708020022:~/tmp$ cat test
#/bin/bash
# while command test

var1=10
while [ $var1 -gt 0 ]
do
        echo $var1
        var1=$[ $var1 - 1 ]
done

wangsx@SC-201708020022:~/tmp$ ./test
10
9
8
7
6
5
4
3
2
1
```

**使用多个测试命令**
`while`命令允许我们在`while`语句行中定义多个测试命令。只有最后一个测试命令的退出状态码会被用来决定什么时候结束循环。
比如` while echo $var1 [ $var1 -ge 0 ] `检测的就是后面方括号命令的退出状态码。

### until命令
`until`命令和`while`命令工作的方式完全相反。只有测试命令的退出状态码不为0，bash shell才会执行循环中列出的命令。一旦测试命令返回了退出状态码0，循环就结束了。

```{shell}
until test command
do
  other commands
done
```
一个例子：
```shell
wangsx@SC-201708020022:~/tmp$ cat test12  
#!/bin/bash                               
# using the until command                 
                                          
var1=100                                  
                                          
until [ $var1 -eq 0 ]                     
do                                        
        echo $var1                        
        var1=$[ $var1 - 25 ]              
done                                      
                                          
wangsx@SC-201708020022:~/tmp$ ./test12    
100                                       
75                                        
50                                        
25                                        
```
同样地，在`until`命令中放入多个测试命令时也要注意（类似`while`）。

### 嵌套循环

在循环语句内使用任意类型的命令，包括其他循环命令，叫做嵌套循环。因为是在迭代中迭代，需要注意变量的使用以及程序的效率问题。

下面举一个`for`循环嵌套`for`循环的例子：

```shell
wangsx@SC-201708020022:~/tmp$ cat test14
#!/bin/bash
# nesting for loops

for (( a = 1; a <= 3; a++ ))
do
        echo "Starting loop $a:"
        for (( b = 1; b <= 3; b++ ))
        do
                echo "    Inside loop: $b"
        done
done

wangsx@SC-201708020022:~/tmp$ . test14
Starting loop 1:
    Inside loop: 1
    Inside loop: 2
    Inside loop: 3
Starting loop 2:
    Inside loop: 1
    Inside loop: 2
    Inside loop: 3
Starting loop 3:
    Inside loop: 1
    Inside loop: 2
    Inside loop: 3
```

shell能够自动识别匹配的`do`和`done`字符。这种模式很常见，比如通常的小括号（`(`与`)`）、中括号、花括号匹配等等。它们的本质都是字符匹配。

在混用循环命令时也一样，比如在`while`循环中内嵌一个`for`循环：

```shell
wangsx@SC-201708020022:~/tmp$ cat test15
#!/bin/bash
# placing a for loop inside a while loop

var1=5

while [ $var1 -ge 0 ]
do
        echo "Outer loop: $var1"
        for (( var2 = 1; $var2 < 3; var2++))
        do
                var3=$[ $var1 * $var2 ]
                echo "  Inner loop: $var1 * $var2 = $var3"
        done
        var1=$[ $var1 - 1 ]
done

wangsx@SC-201708020022:~/tmp$ . test15
Outer loop: 5
  Inner loop: 5 * 1 = 5
  Inner loop: 5 * 2 = 10
Outer loop: 4
  Inner loop: 4 * 1 = 4
  Inner loop: 4 * 2 = 8
Outer loop: 3
  Inner loop: 3 * 1 = 3
  Inner loop: 3 * 2 = 6
Outer loop: 2
  Inner loop: 2 * 1 = 2
  Inner loop: 2 * 2 = 4
Outer loop: 1
  Inner loop: 1 * 1 = 1
  Inner loop: 1 * 2 = 2
Outer loop: 0
  Inner loop: 0 * 1 = 0
  Inner loop: 0 * 2 = 0
```

如果想要挑战脑力，可以混用`until`和`while`循环。

```shell
wangsx@SC-201708020022:~/tmp$ cat test16
#!/bin/bash
# using until and while loop

var1=3

until [ $var1 -eq 0 ]
do
        echo "Outer loop: $var1"
        var2=1
        while [ $var2 -lt 5 ]
        do
                var3=$(echo "scale=4; $var1 / $var2" | bc)
                echo "  Inner loop: $var1 / $var2 = $var3"
                var2=$[ $var2 + 1 ]
        done
        var1=$[ $var1 - 1 ]
done

wangsx@SC-201708020022:~/tmp$ . test16
Outer loop: 3
  Inner loop: 3 / 1 = 3.0000
  Inner loop: 3 / 2 = 1.5000
  Inner loop: 3 / 3 = 1.0000
  Inner loop: 3 / 4 = .7500
Outer loop: 2
  Inner loop: 2 / 1 = 2.0000
  Inner loop: 2 / 2 = 1.0000
  Inner loop: 2 / 3 = .6666
  Inner loop: 2 / 4 = .5000
Outer loop: 1
  Inner loop: 1 / 1 = 1.0000
  Inner loop: 1 / 2 = .5000
  Inner loop: 1 / 3 = .3333
  Inner loop: 1 / 4 = .2500
```

外部的`until`循环以值3开始，并继续执行到值等于0。内部`while`循环以值1开始一直执行，只要值小于5。需要注意循环条件的设置，我跑的几次都没写完整，然后无限循环只好重开终端。

### 控制循环

之前的学的命令已经可以让我们写循环程序了，设定好以后等待命令开始执行和等待循环结束。但是很多情况下，在循环中我们设定的某个（多个）变量达到某种条件时，我们就想要停止循环，然后运行循环下面的命令。这时候我们需要用到`break`和`continue`命令来帮我们控制住循环。

这两个命令在其他语言中基本都时关键字，特别是`C`，用法差不多。我也就不具体介绍了，只点出它们的功能。

**break**

> 在shell执行break命令时，它会尝试跳出当前正在执行的循环。
>
> 在处理多个循环时，break命令会自动终止你所在的最内层循环。
>
> break命令接受单个命令行参数值：
>
> ​	break n
>
> ​	其中n制订了要跳出的循环层级（层数）

**continue**

>continue命令可以提前终止某次循环的命令，但并不会完全终止整个循环。可以在循环内部设置shell不执行命令的条件。
>
>也就是说使用continue命令时，它会自动跳过本次循环中接下来的运行步骤，跳转到下一次循环。但注意不是跳出，跳出时break的功能。
>
>同样的可以使用continue n         n制定要继续执行哪一级循环



### 处理循环的输出

在shell脚本中，我们可以对循环的输出使用管道或进行重定向。这是通过在`done`命令之后添加一个处理命令来实现的。

```shell
wangsx@SC-201708020022:~/tmp$ cat test
#!/bin/bash
for file in /home/*
do
        if [ -d "$file" ]
        then
                echo "$file is a directory"
        else
                echo "$file is a file"
        fi
done > output.txt
wangsx@SC-201708020022:~/tmp$ cat output.txt
/home/wangsx is a directory
```

shell将`for`命令的结果重定向到文件`output.txt`中，而不是显示在屏幕上。



### 实例

下面两个例子演示如何用简单循环来处理数据。

**查找可执行文件**

Linux运行程序时通过环境变量`$PATH`提供的目录搜索可执行文件。如果徒手找的话，比较费时间，我们可以写个脚本来搞定它。

```shell
wangsx@SC-201708020022:~$ cat test25
#!/bin/bash
# finding files in the PATH

IFS=:
for folder in $PATH
do
        echo "$folder:"
        for file in $folder/*
        do
                if [ -x $file ]
                then
                        echo "  $file"
                fi
        done
done

# 输出结果太多，我就不拷贝结果了
```

先设定`IFS`分隔符以便于能正确分隔目录，然后将目录存放在`$folder`中，用`for`循环来迭代特定的目录中所有文件，然后用`if-then`命令检查文件的可执行权限。

Linux有一个`tree`工具，非常方便输出目录结构，推荐使用下。



**创建多个用户账号**

如果你是管理员，需要创建大量账号时。不必每次都有`useradd`命令添加用户。将用户信息存放在指定文件，然后用脚本进行处理就可以了。

用户信息的格式如下：

```
userid, user name
```

第一个是你为用户选择的id，第二个是用户的全名。这是`csv`文件格式。

为了能够读取它，我们使用以下命令：

```shell
while IFS=',' read -r userid name
```

`read`命令会自动获取`.csv`文本文件的下一行内容，所以不用再写一个循环来处理。当`read`命令返回`FALSE`时（也就是读完了），`while`命令就会退出。

为了把数据从文件导向`while`命令，只要再`while`命令尾部加一个重定向符号。

处理过程写成脚本如下：

```shell
#!/bin/bash
# process new user accounts

input="users.csv"
while IFS=',', read -r userid name
do
	echo "adding $userid"
	useradd -c "$name" -m $userid
done < "$input"
```
