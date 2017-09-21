---
title: "快速探索数据"
date: 2017-09-21
status: publish
output: html_notebook
categories: 
- R
- R-Cookbook
tags:
- R
- ggplot2
---
 
 
# 绘制散点图
 
 
## 方法
 
直接调用`plot()`函数，运行命令时传入向量x和向量y。
 

{% highlight r %}
plot(mtcars$wt, mtcars$mpg)
{% endhighlight %}

![plot of chunk unnamed-chunk-1](/figures/unnamed-chunk-1-1.png)
 
 
对于ggplot2系统，可以用`qplot()`函数得到相同的绘图结果：

{% highlight r %}
library(ggplot2)
qplot(mtcars$wt, mtcars$mpg)
{% endhighlight %}

![plot of chunk unnamed-chunk-2](/figures/unnamed-chunk-2-1.png)
 
如果两个参数向量包含在一个数据框内，使用下面命令：
 

{% highlight r %}
qplot(wt, mpg, data=mtcars)
{% endhighlight %}

![plot of chunk unnamed-chunk-3](/figures/unnamed-chunk-3-1.png)

{% highlight r %}
# 这与下面等价
# ggplot(mtcars, aes(x=wt, y=mpg)) + geom_point()
{% endhighlight %}
 
<!-- more --> 
 
# 绘制折线图
 
## 方法
 
使用`plot()`函数绘制时需要传入坐标参量，以及使用参数`type="l"`:

{% highlight r %}
plot(pressure$temperature, pressure$pressure, type="l")
{% endhighlight %}

![plot of chunk unnamed-chunk-4](/figures/unnamed-chunk-4-1.png)
 
如果想要在此基础之上添加点或者折线，需要通过`point()`函数与`lines()`函数实现。

{% highlight r %}
plot(pressure$temperature, pressure$pressure, type="l")
points(pressure$temperature, pressure$pressure)
 
lines(pressure$temperature, pressure$pressure/2, col="red")
points(pressure$temperature, pressure$pressure/2, col="red")
{% endhighlight %}

![plot of chunk unnamed-chunk-5](/figures/unnamed-chunk-5-1.png)
 
可以使用`ggplot`包达到类似的效果。

{% highlight r %}
library(ggplot2)
qplot(pressure$temperature, pressure$pressure, geom=c("line", "point"))
{% endhighlight %}

![plot of chunk unnamed-chunk-6](/figures/unnamed-chunk-6-1.png)

{% highlight r %}
# 或者
 
ggplot(pressure, aes(x=temperature, y=pressure)) + geom_line() + geom_point()
{% endhighlight %}

![plot of chunk unnamed-chunk-6](/figures/unnamed-chunk-6-2.png)
 
 
 
# 绘制条形图
 
## 方法
 
向`barplot()`传入两个参数，第一个设定条形的高度，第二个设定对应的标签（可选）。
 

{% highlight r %}
barplot(BOD$demand, names.arg = BOD$Time)
{% endhighlight %}

![plot of chunk unnamed-chunk-7](/figures/unnamed-chunk-7-1.png)
 
有时候，条形图表示分组中各元素的频数，这跟直方图类似。不过x轴不再是上图看到的连续值，而是离散的。可以使用`table()`函数计算类别的频数。
 

{% highlight r %}
barplot(table(mtcars$cyl))
{% endhighlight %}

![plot of chunk unnamed-chunk-8](/figures/unnamed-chunk-8-1.png)
 
可以使用`ggplot2`系统函数，注意需要将作为横坐标的变量转化为因子型以及参数设定。

{% highlight r %}
library(ggplot2)
 
ggplot(BOD, aes(x=factor(Time), y=demand)) + geom_bar(stat="identity")
{% endhighlight %}

![plot of chunk unnamed-chunk-9](/figures/unnamed-chunk-9-1.png)

{% highlight r %}
ggplot(mtcars, aes(x=factor(cyl))) + geom_bar()
{% endhighlight %}

![plot of chunk unnamed-chunk-9](/figures/unnamed-chunk-9-2.png)
 
 
# 绘制直方图
 
同样地，我们用两者方法来绘制

{% highlight r %}
hist(mtcars$mpg)
{% endhighlight %}

![plot of chunk unnamed-chunk-10](/figures/unnamed-chunk-10-1.png)

{% highlight r %}
# 通过breaks参数指定大致组距
hist(mtcars$mpg, breaks = 10)
{% endhighlight %}

![plot of chunk unnamed-chunk-10](/figures/unnamed-chunk-10-2.png)

{% highlight r %}
library(ggplot2)
ggplot(mtcars, aes(x=mpg)) + geom_histogram()
{% endhighlight %}



{% highlight text %}
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
{% endhighlight %}

![plot of chunk unnamed-chunk-10](/figures/unnamed-chunk-10-3.png)

{% highlight r %}
ggplot(mtcars, aes(x=mpg)) + geom_histogram(binwidth = 5)
{% endhighlight %}

![plot of chunk unnamed-chunk-10](/figures/unnamed-chunk-10-4.png)
 
 
# 绘制箱线图
 
使用`plot()`函数绘制箱线图时向其传递两个向量：x,y。当x为因子型变量时，它会默认绘制箱线图。

{% highlight r %}
plot(ToothGrowth$supp, ToothGrowth$len)
{% endhighlight %}

![plot of chunk unnamed-chunk-11](/figures/unnamed-chunk-11-1.png)
 
 
当这两个参数变量包含在同一个数据框，可以使用公式语法。

{% highlight r %}
# 公式语法
boxplot(len ~ supp, data=ToothGrowth)
{% endhighlight %}

![plot of chunk unnamed-chunk-12](/figures/unnamed-chunk-12-1.png)

{% highlight r %}
# 在x轴引入两变量交互
boxplot(len ~ supp + dose, data = ToothGrowth)
{% endhighlight %}

![plot of chunk unnamed-chunk-12](/figures/unnamed-chunk-12-2.png)
 
下面使用`ggplot2`绘制

{% highlight r %}
library(ggplot2)
qplot(ToothGrowth$supp, ToothGrowth$len, geom="boxplot")
{% endhighlight %}

![plot of chunk unnamed-chunk-13](/figures/unnamed-chunk-13-1.png)

{% highlight r %}
ggplot(ToothGrowth, aes(x=supp, y=len)) + geom_boxplot()
{% endhighlight %}

![plot of chunk unnamed-chunk-13](/figures/unnamed-chunk-13-2.png)
 
使用`interaction()`函数将分组变量组合在一起可以绘制基于多分组变量的箱线图。
 

{% highlight r %}
ggplot(ToothGrowth, aes(x=interaction(supp, dose), y=len)) + geom_boxplot()
{% endhighlight %}

![plot of chunk unnamed-chunk-14](/figures/unnamed-chunk-14-1.png)
 
