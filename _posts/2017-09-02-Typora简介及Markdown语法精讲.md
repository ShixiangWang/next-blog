---
title: Typora简介及Markdown语法精讲
date: 2017-09-02
categories: Documentation
tags:
- Markdown
- Typora
---

这篇文章简单介绍Typora这款Markdown语法编辑器，以及精讲它支持的Markdown语法。

<!-- more -->

# Typora

Typora的官网是<https://www.typora.io/>，里面有详尽的介绍和展示，下面我摘取一些片段介绍一下该编辑器的特性。

用它进行Markdown文件的阅读和写作可以用“美”字来形容，具体体现在三大体验感中：

- 自由随心
- 实时无缝预览
- 所见即所得

喜欢它，因为它简约又强大。下面看下它的一些特性——

### 插入图片

![Images](https://www.typora.io/img/new/image.png)

### 生成标题

![Headers](https://www.typora.io/img/new/toc.png)

### 生成列表

![Lists](https://www.typora.io/img/new/lists.png)

### 表格

![Tables](https://www.typora.io/img/new/table.png)

### 代码块

![Code Fences](https://www.typora.io/img/new/fences.png)

### 数学公式

![Math](https://www.typora.io/img/new/math.png)

### 图解

![Diagrams](https://www.typora.io/img/new/diagram.png)

### 行内样式

![Inline Style](https://www.typora.io/img/new/inline.png)

除此之外，它可以方便地管理md文件，在文章标题间快速跳转，支持YAML头信息，导入和输出成通用的文件格式。

![管理文件](https://www.typora.io/img/new/files.png)

![导入和导出](https://www.typora.io/img/new/import-export.png)

还有大量我完全没用过的特性，像支持其他插件进行扩展、修改主题、自动匹配等等。关键是开发团队还在不断地维护和开发这个软件，并且我们可以免费使用它。我之前也用过不少的Markdown语法编辑器，这个是我一用上就离不开的，无论是在Windows上还是Ubuntu上，我都可以用它进行快速流畅的文档创作（当然它也支持Mac）。

它对英文字体支持非常棒，看起来很舒服。可惜在Windows上对中文字体的显示不太友好，看起来有点丑。好像可以更换字体，不过没有尝试过。



# Markdown



之前推荐过Markdwon的语法介绍，像我在Github上fork的<https://github.com/ShixiangWang/README>。当然百度也能搜出大量的该语法介绍。这里我想根据Typora官方提供的Markdown参考精讲一下它支持的语法特性。

## 块元素

### 段落和换行符

在Markdown源码中需要多个空行才能重启一段，Typora中只需要按下`Enter`键即可。

按下`Shift`+`Enter`键会创建一个单个换行符，但是大多数Markdown语法解析器都会忽略它。我们可以通过在一行的末尾加两个空格键，或者插入`<br/>`。



### 标题

在一行的开始加1-6个井号键，对应着文章的1-6级标题。

```markdown
# 这是一级标题

## 这是二级标题

###### 这是六级标题
```

一般建议标题不要超过三级，不然可能层级太多影响阅读。

### 引用

Markdown用`>`代表引用，格式看起来如下：

```markdown
> This is a blockquote with two paragraphs. This is first paragraph.
>
> This is second pragraph.Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus.
```



> This is a blockquote with two paragraphs. This is first paragraph.
>
> This is second pragraph.Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus.



### 列表

`*`和`+`，`-`三种字符都能用来创建无序列表，但最好不要混用。有序列表用数字加`.`号即可创建。

```markdown
## un-ordered list
*   Red
*   Green
*   Blue

## ordered list
1.  Red
2. 	Green
3.	Blue
```

un-ordered list

*   Red
*   Green
*   Blue

ordered list

1.  Red
2.  Green
3.  3.Blue



### 任务列表

任务列表用[]或[x]标记（未完成或完成）。、

```markdown
- [ ] a task list item
- [ ] list syntax required
- [ ] normal **formatting**, @mentions, #1234 refs
- [ ] incomplete
- [x] completed
```

- [ ] a task list item
- [ ] list syntax required
- [ ] normal **formatting**, @mentions, #1234 refs
- [ ] incomplete
- [x] completed

在Typora中直接可以点击来修改任务完成的状态。



### 代码块

Typora只支持Github的Markdown语法，对源代码块（Markdown有不同的种类，所以有些地方不太一样）不支持。

Typora中只需要键入\```并按下Enter键就可以输入代码块了。在\```后面可以加上你使用的语言名。Typora会自动进行语法高亮。

```markdown
Here's an example:

​```
function test() {
  console.log("notice the blank line before this function?");
}
​```

syntax highlighting:
​```ruby
require 'redcarpet'
markdown = Redcarpet.new("Hello World!")
puts markdown.to_html
​```
```



```
function test() {
  console.log("notice the blank line before this function?");
}
```

语法高亮:
```ruby
require 'redcarpet'
markdown = Redcarpet.new("Hello World!")
puts markdown.to_html
```


### 数学公式块

我们可以使用**MathJax**语法输入*LaTex*数学公式。

用一个`$`开始和结尾会生成行内公式。而用`$$`开始和结尾并单独生成公式块。

  
$$
\mathbf{V}_1 \times \mathbf{V}_2 =  \begin{vmatrix} 
\mathbf{i} & \mathbf{j} & \mathbf{k} \\
\frac{\partial X}{\partial u} &  \frac{\partial Y}{\partial u} & 0 \\
\frac{\partial X}{\partial v} &  \frac{\partial Y}{\partial v} & 0 \\
\end{vmatrix}
$$


比如我们如果要生成如上公式，需要输入

```markdown
$$
\mathbf{V}_1 \times \mathbf{V}_2 =  \begin{vmatrix} 
\mathbf{i} & \mathbf{j} & \mathbf{k} \\
\frac{\partial X}{\partial u} &  \frac{\partial Y}{\partial u} & 0 \\
\frac{\partial X}{\partial v} &  \frac{\partial Y}{\partial v} & 0 \\
\end{vmatrix}
$$
```



### 表格

输入 `| First Header  | Second Header |` 并按下 `Enter` 键会返回两列的表。

一旦表格生成，我们可以像使用Excel一样对表格进行对齐、添加、修改、删除操作。

本身的源代码是这样的：

```markdown
| First Header  | Second Header |
| ------------- | ------------- |
| Content Cell  | Content Cell  |
| Content Cell  | Content Cell  |
```

看看效果:

| First Header | Second Header |
| ------------ | ------------- |
| Content Cell | Content Cell  |
| Content Cell | Content Cell  |

可以用`:`符号进行对齐操作。

比如：

```markdown
| Left-Aligned  | Center Aligned  | Right Aligned |
| :------------ |:---------------:| -----:|
| col 3 is      | some wordy text | $1600 |
| col 2 is      | centered        |   $12 |
| zebra stripes | are neat        |    $1 |
```

它将显示成：

| Left-Aligned  | Center Aligned  | Right Aligned |
| :------------ | :-------------: | ------------: |
| col 3 is      | some wordy text |         $1600 |
| col 2 is      |    centered     |           $12 |
| zebra stripes |    are neat     |            $1 |



### 脚注

```markdown
You can create footnotes like this[^footnote].

[^footnote]: Here is the *text* of the **footnote**.
```

第一行是文本，第二行是对标记字符进行解释。当把鼠标指向脚注时，会看到解释内容。

You can create footnotes like this[^footnote].

[^footnote]: Here is the *text* of the **footnote**.



### 水平线

有时候我们想要水平线来进行文本内容分割，可以在空白行上使用`***`或者`---`然后按下回车键即可。



### YAML头信息

Typora支持YAML头信息，一般网页需要写这个。在文档的开始键入`---`然后按下回车键就会生成一个独立的区块可以写入YAML头信息。



### 目录

输入`[toc]`并按下回车键会自动生成文章目录（这个在Github和简书上好像都不支持）。



### 图解

不翻译了，拷贝。

Typora supports, [sequence](https://bramp.github.io/js-sequence-diagrams/), [flowchart](http://flowchart.js.org/) and [mermaid](https://knsv.github.io/mermaid/#mermaid), after this feature is enabled from preference panel.

See this [document](http://support.typora.io/Draw-Diagrams-With-Markdown/) for detail.



## Span元素

### 链接

Markdown支持两种格式的链接：行内链接和参考链接。

两种链接中，文本信息都用`[]`限定。

常用的就是行内链接了，平常我们点击网址，然后浏览器会自动跳转并打开就属于这种。

它又分为有标题的和没标题的：

```markdown
This is [an example](http://example.com/ "Title") inline link.

[This link](http://example.net/) has no title attribute.
```

This is [an example](http://example.com/ "Title") inline link.

[This link](http://example.net/) has no title attribute.

#### 内部链接

我们可以把链接指向文本的某个标题，这样就能够实现文章内部的跳转，像书签一样。

这个功能我没太摸清楚，参考的markdown文章中是能用的。有兴趣的可以看下，在下方留评。



#### 参考链接

两个`[]`，第一个写文本信息，第二个写链接ID。

```markdown
This is [an example][id] reference-style link.

Then, anywhere in the document, you define your link label like this, on a line by itself:

[id]: http://example.com/  "Optional Title Here"
```

This is [an example][id] reference-style link.

Then, anywhere in the document, you define your link label like this, on a line by itself:

[id]: http://example.com/  "Optional Title Here"

这就像写文章的参考文献一样。

有趣的是第二个方括号内内容可以省略，第一个方括号内内容会当作ID使用。

```markdown
[Google][]
And then define the link:

[Google]: http://google.com/
```

[Google][]
And then define the link:

[Google]: http://google.com/



### URLs

直接用`<>`可以插入网址。

`<i@typora.io>` becomes <i@typora.io>.



### 图片

插入图片与插入链接相似，不过要在最前面加`!`做区别。

```markdown
![Alt text](/path/to/img.jpg)
# 左边是图片标题，右边是地址，可以是url，也可以是本地的图片地址（注意绝对路径和相对路径的区别，可以在Typora中设定）
![Alt text](/path/to/img.jpg "Optional title")
```

比如我在文章最前面插的那些图片：

```markdown
![Images](https://www.typora.io/img/new/image.png)
```

效果就是

![Images](https://www.typora.io/img/new/image.png)

### 强调

有时候我们需要对文本进行强调，方式是斜体。可以用`*`或`_`分隔要强调的文本。

```markdown
*single asterisks*

_single underscores_
```

输出: 

*single asterisks*

_single underscores_

它们是可以和引用，以及后面讲的加粗连用的。

如果我们需要输出`*`键，可以用`\`进行转义。



### 加粗

强调用一个符号，加粗用两个。

```markdown
**double asterisks**

__double underscores__
```

**double asterisks**

__double underscores__



Markdown一般都推荐使用`*`作为分隔符。



### 代码

加入行内代码，只需要用\`分隔即可。

```markdown
Use the `printf()` function.
```

Use the `printf()` function.



### 删除线

文本前后用两个`~`符号：

`~~Mistaken text.~~` becomes ~~Mistaken text.~~



### 下划线

跟HTML语法一样。

`<u>Underline</u>` becomes <u>Underline</u>.



### 表情:happy:

插入笑脸用`:smile`，得到:smile:

还有其他一些标签，键入`:`会根据后面的英文字符查找相关表情。

比如`:cry:`，:cry:

遵循这样的规则使用即可。



### HTML

Typora只支持有限的HTML语法，如下，我就不翻译了。

Typora cannot render html fragments. But typora can parse and render very limited HTML fragments, as an extension of Markdown, including:

- Underline: `<u>underline</u>`
- Image: `<img src="http://www.w3.org/html/logo/img/mark-word-icon.png" width="200px" />` (And `width`, `height` attribute in HTML tag, and `width`, `height`, `zoom` style in `style` attribute will be applied.)
- Comments: `<!-- This is some comments -->`
- Hyperlink: `<a href="http://typora.io" target="_blank">link</a>`.

Most of their attributes, styles, or classes will be ignored. For other tags, typora will render them as raw HTML snippets. 

But those HTML will be exported on print or export.



### 行内数学公式

前面提到了，这里举个例子。

```markdown
$\lim_{x\to\infty}\exp(-x)=0$
```

$\lim_{x\to\infty}\exp(-x)=0$

同样的公式，我用`$$`分隔就会变成：


$$
\lim_{x\to\infty}\exp(-x)=0
$$


可以看到显示效果的不同哈。



### 下标

用一个`~`，如果要写水分子，我们可以使用`H~2~O`，它就会变成H~2~O.

这个功能默认没打开，To use this feature, first, please enable it in `Preference` Panel -> `Markdown` Tab. 

现在看看H~2~O。



### 上标

用一个`^`前后分隔。

X的平方可以表示为`X^2^`，即X^2^ 。



### 高亮

用两个`==`在前后分隔对文本高亮。

`==highlight==`会变成

==highlight==



# 最后的话

因为个人能力有限，翻译和讲解不可能尽善尽美，望谅解。

关于Typora的使用除了查看本文档外，还需要大家自己多摸索。用`Ctrl`加逗号键可以打开Typora的选项设定，有的功能没有激活，需要我们手动设定下，也可以看看有其他什么有趣有用的功能。欢迎留言评论。

最后说明，因为Markdown编辑器不同，简书或者博客上查阅可能有一些小问题。以Typora编辑器为准。为保证文章目录正常，去掉了一些实例显示。

