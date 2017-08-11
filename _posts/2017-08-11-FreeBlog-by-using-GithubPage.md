---
title: 用Github搭建自己免费的个人博客
date: 2017-08-11
categories:
- Documentation
tags:
- Github
- Blog
---



咳咳，如果你想要公开你的博客。买域名还是要钱滴，我不会告诉你我花了一块钱。

网上关于这方面的素材已然不少，但是我还是想跟大家讲讲，一是谈谈感受，二是推荐下NexT主题。我的个人博客也是最近几天搭建好的，你可以通过[点击](http://www.flypeom.site/)查看主题效果。

<!-- more -->

## 搭建博客

首先需要创建一个`github`账号，官方网址在[这里](https://github.com/)。
![图片转自http://blog.csdn.net/renfufei/article/details/37725057/](http://upload-images.jianshu.io/upload_images/3884693-673464d8910d5dc6?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

操作很简单，相信你没问题。

第二步是`fork`我的主题，地址点击[这里](https://github.com/ShixiangWang/ShixiangWang.github.io)。右上角有个大大的`Fork`按钮，点击它，没错，点它。这样等一下，`github`嘟嘟嘟就把我的所有内容传到你的资源包里去了。

第三步，把资源包的名字改掉。改成根据创建账号用户名+github.io。像我的用户名是`ShixiangWang`，名字就该是`ShxiangWang.github.io`。如下图，在`Settings`里面改变然后点击`Rename`即可。
![rename.png](http://upload-images.jianshu.io/upload_images/3884693-28cb769bdb90186c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这样你的博客框架已经搭好了，可以在浏览器输入`用户名.github.io`查看。

如果你觉得这个主题不行，那我无话可说。你可以上[Jekyll Themes](http://jekyllthemes.org/)
找个你喜欢的，找到它的`github`地址`Fork`之后改名；或者下载所有的文件把`ShixiangWang.github.io`下的文件全部删掉，把你喜欢的资源包拷贝进去。这个过程如果你在浏览器上操作比较麻烦，如果是`Windows`用户或者`Mac`用户可以下载[Github桌面版](https://desktop.github.com/)进行操作：先把你的资源库Fork到本地，然后修改（删掉所有的内容，把你下载的主题资源包内容拷贝进去），然后上传到仓库（`Commit`后`Push`）。

具体软件的使用可以参考这个[知乎链接](https://www.zhihu.com/question/20070065)。使用Linux的朋友多少对git有些概念，直接使用git对github仓库进行操作并不困难，相信你们能够搞定，不会的话网上也有一大堆开源博文等着你。我之前也整理过一篇[git使用手册](http://www.jianshu.com/p/e32a8e7ca93b)。

## 修改主题配置

关于NexT主题的使用，[README文档](http://www.jianshu.com/p/e32a8e7ca93b)里有详细的介绍和配置文档链接。我是从https://github.com/Simpleyyt/jekyll-theme-next那里Fork过来的，如果有问题你可以建立`issue`进行交流。当然你也可以Fork他的，因为我的主题已经参考[使用文档](http://theme-next.simpleyyt.com/)做了一些自定义修改，所以有一些不同。

如果Fork我的主题，你看到的就是我[个人博客](http://www.flypeom.site/)显示的那样。你只需要改些跟自己有关爱好、涉及账号的东西就可以了。当然你可以根据[使用文档](http://theme-next.simpleyyt.com/)一步一步的修改和调试，跟着做就行了。

如果是Fork我的，请修改以下内容，具体操作参考[使用文档](http://theme-next.simpleyyt.com/)：
>**开始使用**专栏里：
>
> 设置 头像
>
> 设置 作者昵称
>
> 站点描述
>
>**主题配置**专栏里：
>
>侧边栏社交链接
>
>开启打赏功能
>
>**第三方服务**专栏里：
>
>来必力
>
>百度统计

## 公开博客
这一部分就是为你的`Github`个人博客绑定域名。就像我的域名是flypeom.site，你在浏览器输入它，浏览器能够找到它的ip地址，从而打开你的博客主页。

>虽然在Internet上可以访问我们的网站，但是网址是GitHub提供的:http://xxxx.github.io ，而我们想使用我们自己的个性化域名，这就需要绑定我们自己的域名。这里演示的是在阿里云万网的域名绑定，在国内主流的域名代理厂商也就阿里云和腾讯云。登录到阿里云，进入管理控制台的域名列表，找到你的个性化域名，进入解析
>![](http://upload-images.jianshu.io/upload_images/3884693-e4dc8d7fa29a242e.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
>然后添加解析
>![](http://upload-images.jianshu.io/upload_images/3884693-ba44d23fc51bb01f.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
>包括添加三条解析记录，192.30.255.112是GitHub的地址，你也可以ping你的 http://xxxx.github.io 的ip地址，填入进去。第三个记录类型是CNAME，CNAME的记录值是：你的用户名这里千万别弄错了。第二步，登录GitHub，进入之前创建的仓库，点击settings，设置Custom domain，输入你的域名。

以上内容引自：[GitHub+Hexo 搭建个人网站详细教程](https://zhuanlan.zhihu.com/p/26625249?utm_source=weibo&utm_medium=social)

所谓的Ping地址，Windows可以用菜单键+R键入cmd进入终端/Linux直接打开终端，键入：
```shell
ping xxx.github.io #把xxx改成你的用户名
```
可以看到返回结果（比如下面我的）里面包含ip地址，这就是你在添加解析时需要填入的。
```shell
wsx@wsx-ubuntu:~$ ping ShixiangWang.github.io
PING sni.github.map.fastly.net (151.101.193.147) 56(84) bytes of data.
64 bytes from 151.101.193.147: icmp_seq=1 ttl=48 time=60.7 ms
64 bytes from 151.101.193.147: icmp_seq=2 ttl=48 time=60.3 ms
64 bytes from 151.101.193.147: icmp_seq=3 ttl=48 time=60.2 ms
64 bytes from 151.101.193.147: icmp_seq=4 ttl=48 time=60.1 ms
64 bytes from 151.101.193.147: icmp_seq=5 ttl=48 time=60.1 ms
64 bytes from 151.101.193.147: icmp_seq=6 ttl=48 time=60.1 ms
```

最后在资源的最外层创建一个`CNAME`文本文件，记住不要后缀，在里面填入域名即可。

下图可以看到我的资源里有这个文件：
![cname.png](http://upload-images.jianshu.io/upload_images/3884693-58f0df24548b8dd5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

看看里面的内容吧：

里面就一行字符，写的就是域名。
![cname_content.png](http://upload-images.jianshu.io/upload_images/3884693-ce7bdb19d983510d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



## 写博客

上述配置完后，剩下你需要关心的只有写博文了。博文的书写需要遵循一定的要求，包括3方面：

- 头信息
- 博文内容
- 文件名

头信息是需要遵循YAML语法的一些信息，书写在文件的头部。内容可以有很多，它的格式如下：

```yaml
---
layout: post
title: This is my first blog
---
```

这是符合YAML语法的头信息，它用来让Jekyll正确解析该文件的内容。比如说`layout`指定了这是一篇博文，`title`指定了题目。

常用的有以下几个键值对：

```yaml
---
title: My blog title
date: 2017-08-11
categories:
- life
- more
tags:
- blog
- post
---
```

`date`指定了文章写作日期；`categories`指定了文章放置的目录；`tags`指定文章标签。这些信息书写后，Jekyll会自动将你的文章按时间顺序收录和生成标签云。是不是很赞~关于Jekyll以及YAML的相关知识，可以查看[官方中文文档](http://jekyllcn.com/docs/home/)喔。

博文的内容需要服从Markdown语法。正文的话就直接打就行了，但是像标题，斜体，下划线等等的实现符合使用Markdown语法。Markdown非常简单易学，也非常流行，各大编程相关的网站（像Biostar, Stack overflow）都基本支持，[简书](http://www.jianshu.com/)也支持。想要了解的朋友可以查看https://github.com/ShixiangWang/README，或者依赖搜索引擎查阅相关资料。

博文存档时的文件名需要符合特定的格式要求：可以是`.md`文件和`.html`文件。如果是前者，Jekyll会自动将它解析成网页。命名则是`xxxx-xx-xx-*.md`，其中`xxxx-xx-xx`需要填入书写博文的时间，比如今天应该书写为`2017-08-11`，`*`指代可以填入任意内容，用以区分文件。`.md`表示是Markdown文件。



-----------------

文章内容已经写完了，有什么疑问欢迎和我交流。能力所限，难免有所遗漏，大家多多包涵。

感觉NexT主题很不错，非常简约，本人十分喜欢。如果你也喜欢它，也想搭建这样的博客，就一起来吧。你可以看到我是两天前开始弄的，现在就已经在这里给你们写经验了。其实非常简单， 我自己摸索还走了不少弯路。Come on.
