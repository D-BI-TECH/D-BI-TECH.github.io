---
layout: post
title:  M函数：Text.ReplaceAll
date:   2019-05-18 07:03:50 +0000
image:  08.jpg
tags:   [Power BI,Power Query]
author-name: Daniil Maslyuk
author-image: Daniil.jpg
---

![Replacement dictionary for Text.ReplaceAll](https://img-blog.csdnimg.cn/20191208153340999.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

2019年6月4日更新: 不幸的是，该函数不会被发布。该函数是经过认证的连接器的一部分，现在已隐藏。

在Power BI Desktop 2019年5月的更新文档，其中包括一个新的M函数：Text.ReplaceAll（截至2019年5月18日，未在微软文档中记录）。此函数简化了以前必须使用自定义函数完成的多个单词替换。

#### 背景

使用Power Query一次替换多个文本字符串是常见的情况。几位知名博主已经编写了自己的函数来替代现有功能——比如 Chris Webb, Ivan Bondarenko 和 Imke Feldmann（Imke Feldmann的公式稍有不同——稍后将详细介绍）。

在本文，我将使用和他们一样的数据:

#### 文本查询

<div class="table-container">
  <table>
    <tr><th>文本</th>
    <tr><td>the cat sat on the mat</td>
    <tr><td>the cat sat next to the dog</td>
    <tr><td>the dog chased the cat</td>
    <tr><td>the dog sat on the mat</td>
  </table>
</div>

>```Python
let
    Source = #table(
        type table [Text = text],
        {
            {"the cat sat on the mat"},
            {"the cat sat next to the dog"},
            {"the dog chased the cat"},
            {"the dog sat on the mat"}
        }
    )
in
    Source
>```

#### 替换查询

<div class="table-container">
  <table>
    <tr><th>被替换词</th><th>替换为</th>
    <tr><td>cat</td><td>bear</td>
    <tr><td>mat</td><td>chair</td>
    <tr><td>dog</td><td>dragon</td>
    <tr><td>the</td><td>THE</td>
  </table>
</div>

>```Python
let
    Source = #table(
        type table [
            Word to Replace = text,
            Replace With = text
        ],
        {
            {"cat", "bear"},
            {"mat", "chair"},
            {"dog", "dragon"},
            {"the", "THE"}
        }
    )
in
    Source
>```

#### 函数语法

Text.ReplaceAll 接受两个参数:

1. 文本
2. 替换

其中第二个参数是一组对值：{"被替换词","替换为"}

例如以下表达式返回 “yyz”:

>```Python
Text.ReplaceAll("xyz", {{"x", "y"}})
>```

#### 使用 Text.ReplaceAll

要在Chris的数据上使用这个函数，我们需要首先将字典转换为列表中的列表。为此，我们可以使用List.Zip。假设我们的字典表位于名为Source的步骤中，那么下面的表达式将准确地给出我们所需要的：

>```Python
= List.Zip({Source[Word to Replace], Source[Replace With]})
>```

因此完整的替换查询语句应为:

>```Python
let
        Source = #table(
        type table [
            Word to Replace = text,
            Replace With = text
        ],
        {
            {"cat", "bear"},
            {"mat", "chair"},
            {"dog", "dragon"},
            {"the", "THE"}
        }
    ),
    ToList = List.Zip(
        {
            Source[Word to Replace],
            Source[Replace With]
        }
    )
in
    ToList
>```

应用这个函数就很简单了。如果添加自定义列，则可以使用以下公式：

>```Python
= Text.ReplaceAll([Text], Replacements)
>```

结果如下:

![Text.ReplaceAll replaces all strings with replacement strings](https://img-blog.csdnimg.cn/20191208153435886.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

#### 注意事项

需要注意的是，该函数当前只替换子字符串，而不是整个单词。也就是说，如果在Replacements表中添加一行，如下所示：

<div class="table-container">
  <table>
    <tr><th>被替换词</th><th>替换为</th>
    <tr><td>cat</td><td>bear</td>
    <tr><td>mat</td><td>chair</td>
    <tr><td>dog</td><td>dragon</td>
    <tr><td>the</td><td>THE</td>
    <tr><td>air</td><td>water</td>
  </table>
</div>

那么结果很可能不是你想要的:

![Text.ReplaceAll does not have an option to replace whole words only](https://img-blog.csdnimg.cn/20191208153449672.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

如果您只需要替换整个单词，我将使用[Imke的方法](https://www.thebiccountant.com/2016/05/22/multiple-replacements-in-power-bi-and-power-query/)来解决这个问题。

请注意，该函数按顺序执行替换：如果将air-water放在列表顶部，则会得到不同的结果。

你可以下载我的.pbix文件：[TextReplaceAll.pbix](https://1drv.ms/u/s!Arstm99Oom00gdx_HrAm85o3brqjmw)

----------------------
由Davis ZHANG翻译，[点此](https://xxlbi.com/blog/text-replaceall/)查看英文原文

----------------------