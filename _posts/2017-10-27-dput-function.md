---
title: "dput函数 - 保存R的数据结构"
date: 2017-10-27
categories: 
- R
tags:
- R
- 提问
---

**记住这个函数！记住这个函数！记住这个函数！**
重要的事情说三遍，为了让你能更清醒地记住，我再说一遍，记住这个函数！

`dput`是导出数据结构供重复使用的函数，这是跟别人交流或者请教问题的一个重要函数。比如你在用R分析的时候遇到问题了，有问题的是对某个列表或者数据框进行操作，该怎么让别人可以重复你的错误然后帮你找错呢？这就是这个函数存在的意义。

<!-- more -->

举个例子，假如你有这样一个数据框：
```R
> staff
      name salary years_old
1 zhangsan   1200        20
2     lisi   2200        30
3   wangwu   3200        40
```

然后你想给每个人涨200块并更新表格，想做个简单计算：
```R
> staff$salary <- staff$salary + 200
Error in staff$salary + 200 : non-numeric argument to binary operator
```

我擦，什么鬼，英文我不懂，想请教下别人怎么帮你解决，你这时候就用的上`dput`函数了。

于是你使用这个函数把数据存取，发布到给专家：
```R
> dput(staff)
structure(list(name = structure(c(3L, 1L, 2L), .Label = c("lisi", 
"wangwu", "zhangsan"), class = "factor"), salary = c("1200", 
"2200", "3200"), years_old = c(20, 30, 40)), .Names = c("name", 
"salary", "years_old"), row.names = c(NA, -3L), class = "data.frame")
```

为了区别，专家用a表示`staff`把你的数据结构在他的电脑上重现：

```R
> a <- structure(list(name = structure(c(3L, 1L, 2L), .Label = c("lisi", "wangwu", "zhangsan"), class = "factor"), salary = c("1200", 
+                 "2200", "3200"), years_old = c(20, 30, 40)), .Names = c("name", "salary", "years_old"), row.names = c(NA, -3L), class = "data.frame")
```
然后就可以帮你检查了：

```R
> str(a)
'data.frame':	3 obs. of  3 variables:
 $ name     : Factor w/ 3 levels "lisi","wangwu",..: 3 1 2
 $ salary   : chr  "1200" "2200" "3200"
 $ years_old: num  20 30 40
```

喔~原来`salary`变量是字符向量，不能做数学运算。帮你修改：

```R
> a$salary <- as.integer(a$salary)
> a$salary <- a$salary + 200 
> a
      name salary years_old
1 zhangsan   1400        20
2     lisi   2400        30
3   wangwu   3400        40
```
导出：
```R
> dput(a)
structure(list(name = structure(c(3L, 1L, 2L), .Label = c("lisi", 
"wangwu", "zhangsan"), class = "factor"), salary = c(1400, 2400, 
3400), years_old = c(20, 30, 40)), .Names = c("name", "salary", 
"years_old"), row.names = c(NA, -3L), class = "data.frame")
```

然后你把结果拷贝并导回，工作完成：
```R
> salary <- structure(list(name = structure(c(3L, 1L, 2L), .Label = c("lisi", 
+                                                                     "wangwu", "zhangsan"), class = "factor"), salary = c(1400, 2400, 
+                                                                                                                          3400), years_old = c(20, 30, 40)), .Names = c("name", "salary", 
+                                                                                                                                                                        "years_old"), row.names = c(NA, -3L), class = "data.frame")
> salary
      name salary years_old
1 zhangsan   1400        20
2     lisi   2400        30
3   wangwu   3400        40
```
