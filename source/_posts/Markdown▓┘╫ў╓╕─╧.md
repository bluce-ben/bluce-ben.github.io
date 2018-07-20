---
title: Markdown操作指南
date: 2017-12-06 16:12:20
categories: Markdown
tags: Markdown
---
### 标题
![](/uploads/2017/markdown/markdown_title.png)
注意要点：
* 标题间的空行是没有效果的，除非插入数据（这个是Markdown语法原因）
* 标题大小以标题首的“#”数量为准，标题末尾可添加可不添加“#”
<!--more-->

### 列表
Markdown 支持有序列表和无序列表。
* 无序列表使用星号、加号或是减号作为列表标记：
* 有序列表则使用数字接着一个英文句点：
![](/uploads/2017/markdown/markdown_list.png)
*（注：很重要的一点是，你在列表标记上使用的数字并不会影响输出的 HTML 结果）*

### 引用
![](/uploads/2017/markdown/markdown_quote.png)
*（注意：引用的区块内也可以使用其他的 Markdown 语法，包括标题、列表、代码区块等）*

### 粗体和斜体
![](/uploads/2017/markdown/markdown_strong.png)
如果要在文字前后直接插入普通的星号或底线，你可以用反斜线进行转义。
*（注：不管是粗体还是斜体，只有在一行才会有效果，否则会被当作普通的符号。）*


### 代码引用
![](/uploads/2017/markdown/markdown_code_quote.png)
*（注：代码引用里面，使用markdown其它语法不识别。且反引号的数量`代码后数量<=代码前数量`）*

### 表格
![](/uploads/2017/markdown/markdown_table.png)
*（注：输出表格前需要空行。且表格中表头不能为段落(即空格不能超过4个)，其它没要求。）*

### 链接
![](/uploads/2017/markdown/markdown_link.png)
　此处图片地址可使用绝对路径，也可使用相对路径。
*（注：参考式链接此处不做说明。）*
Markdown 支持以比较简短的自动链接形式来处理网址和电子邮件信箱，只要是用尖括号包起来， Markdown 就会自动把它转成链接。一般网址的链接文字就和链接地址一样，例如：
`<http://example.com/>`
Markdown 会转为：
`<a href="http://example.com/">http://example.com/</a>`

### 图片
行内式图片语法：
`![Alt text](img "title")`
*（注：参考式图片此处不做说明。）*

### 分割线
你可以在一行中用三个以上的星号、减号、底线来建立一个分隔线，行内不能有其他东西。你也可以在星号或是减号中间插入空格。下面每种写法都可以建立分隔线：
```
* * *

***

*****

- - -

---------------------------------------
```

### 缩进和段落
缩进：可使用全角下的空格即可实现缩进
段落：半角下使用4个缩进即可实现

### 特殊字符
在写博客时，一定注意不要写这些字符，如果要写，就进行转义 
有些人可能说可以使用反斜杠 \ 来处理，我这里不建议。 
下面是比较常见的几个:
```
 　&#32;　　　— 空格Space
!　&#33;　　　— 惊叹号Exclamation mark 
”　&#34; &quot;　— 双引号Quotation mark 
#　&#35;　　　— 数字标志Number sign 
$　&#36;　　　— 美元标志Dollar sign 
%　&#37;　　　— 百分号Percent sign 
&　&#38; &amp; — Ampersand 
‘　&#39;　　　— 单引号Apostrophe 
(　&#40;　　　— 小括号左边部分Left parenthesis 
)　&#41;　　　— 小括号右边部分Right parenthesis 
*　&#42;　　　— 星号Asterisk 
+　&#43;　　　— 加号Plus sign 
<　&#60; &lt; — 小于号Less than 
=　&#61;　　　— 等于符号Equals sign 
>　&#62; &gt; — 大于号Greater than 
?　&#63;　　　— 问号Question mark 
@　&#64;　　　— Commercial at 
[　&#91;　　　— 中括号左边部分Left square bracket 
\　&#92;　　　— 反斜杠Reverse solidus (backslash) 
]　&#93;　　　— 中括号右边部分Right square bracket 
{　&#123;　　　— 大括号左边部分Left curly brace 
|　&#124;　　　— 竖线Vertical bar 
}　&#125;　　　— 大括号右边部分Right curly brace 
```

### 嵌入HTML
##### 字体颜色
使用`<font>`标签：`<font color="red"></font>`
##### 图片大小控制
使用`<img>`标签： `<img src="url" width="" height="" />`
#### 表格宽度设定
>`<table>` 中表格的宽度由标题的 `<th>` 决定，我们只需要利用上 CSS 操作一番即可达到目的。
在 Markdown 中，在原表格前添加 CSS 代码，类似这样：
```
<style>
table th:first-of-type {
    width: 100px;
}
</style>

# 下方是表格的 Markdown 语法
名称|值|备注
---|---|---
```
这里涉及到CSS选择器的问题，首先 `<th>` 存在于 `<table>` 中；其次 `th:first-of-type` 的意思是每个 `<th>` 为其父级的第一个元素，这里指的就是围绕着【名称】的 `<th>`。同理第二、三个使用 `th:nth-of-type(2)`、`th:nth-of-type(3)` 就可以了，以此类推。 

#### 表格内插入 |
（1）若直接在表格内插入 | ，会导致表格混乱。需要将其转化为 `&#124;` 即可。
（2）若表格表头的首格为空，如果什么都不填写，会导致表格混乱。可填写 `&#32;`或`&nbsp;` 即可，表示空格（不会显示到界面，同时填充了首格为空的需求）。