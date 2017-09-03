---
title: "Basics-创造填满值的向量"
date: 2017-09-03
status: publish
output: html_notebook
categories: 
- R
- R-Cookbook
tags:
- R
- vector
---
 
# 问题
 
你想要创建一个填满值的列表。
 
<!-- more -->
 
# 方案
 

{% highlight r %}
rep(1:50)
{% endhighlight %}



{% highlight text %}
##  [1]  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 21 22 23
## [24] 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46
## [47] 47 48 49 50
{% endhighlight %}
 
 

{% highlight r %}
rep(FALSE, 20)
{% endhighlight %}



{% highlight text %}
##  [1] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
## [12] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
{% endhighlight %}
 

{% highlight r %}
rep(1:5, 4)
{% endhighlight %}



{% highlight text %}
##  [1] 1 2 3 4 5 1 2 3 4 5 1 2 3 4 5 1 2 3 4 5
{% endhighlight %}
 

{% highlight r %}
rep(1:5, each=4)
{% endhighlight %}



{% highlight text %}
##  [1] 1 1 1 1 2 2 2 2 3 3 3 3 4 4 4 4 5 5 5 5
{% endhighlight %}
 
在一个因子变量上使用
 

{% highlight r %}
rep(factor(LETTERS[1:3]), 5)
{% endhighlight %}



{% highlight text %}
##  [1] A B C A B C A B C A B C A B C
## Levels: A B C
{% endhighlight %}
