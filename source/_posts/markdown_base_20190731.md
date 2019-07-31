---
title: markdown基础语法
tags: 
  - acm队伍相关
  - 教程
categories:
  - acm队伍相关
date: 2019-8-1
---

# markdown快速上手



## Markdown简介

> Markdown 是一种轻量级标记语言，它允许人们使用易读易写的纯文本格式编写文档，然后转换成格式丰富的HTML页面。    —— [维基百科](https://zh.wikipedia.org/wiki/Markdown)

首先是一些基础功能

### 标题

通过`#`设置标题大小

# 一级标题 #
## 二级标题 ##
### 三级标题 ###

代码如下

```markdown
# 一级标题 #
## 二级标题 ##
### 三级标题 ###
```

### 有序/无序列表

通过`-. `或者`x. `（x为数字)的形式
1. test1
	1. test1.1
	2. test1.2
2. test2
3. test3

- test4
	- test4.1
	- test4.2
- test5
- test6

代码如下

```markdown
1. test1
	1. test1.1
	2. test1.2
2. test2
3. test3

- test4
	- test4.1
	- test4.2
- test5
- test6
```
### 文字

*斜体* ：`*斜体*`

**加粗**：`**加粗**`

### 分割线

`---`

### 引用

`>`

---


正如您在阅读的这份文档，它使用简单的符号标识不同的标题，将某些文字标记为**粗体**或者*斜体*，创建一个[链接](http://www.example.com)。下面列举了几个高级功能，具体方法可以参加github中该文章的[源文件](https://raw.githubusercontent.com/DcmTruman/DcmTruman.github.io/blog_source/source/_posts/markdown_base_20190731.md)

### 代码块
``` python
@requires_authorization
def somefunc(param1='', param2=0):
    '''A docstring'''
    if param1 > param2: # interesting
        print 'Greater'
    return (param2 - param1 + 1) or None
class SomeClass:
    pass
>>> message = '''interpreter
... prompt'''
```
### LaTeX 公式

可以创建行内公式，例如 $\Gamma(n) = (n-1)!\quad\forall n\in\mathbb N$。或者块级公式：

$$	x = \dfrac{-b \pm \sqrt{b^2 - 4ac}}{2a} $$

### 表格
| Item      |    Value | Qty  |
| :-------- | --------:| :--: |
| Computer  | 1600 USD |  5   |
| Phone     |   12 USD |  12  |
| Pipe      |    1 USD | 234  |



### 复选框

使用 `- [ ]` 和 `- [x]` 语法可以创建复选框，实现 todo-list 等功能。例如：

- [x] 已完成事项
- [ ] 待办事项1
- [ ] 待办事项2







[1]: http://maxiang.info/client_zh
[2]: https://chrome.google.com/webstore/detail/kidnkfckhbdkfgbicccmdggmpgogehop
[3]: http://adrai.github.io/flowchart.js/
[4]: http://bramp.github.io/js-sequence-diagrams/
[5]: https://dev.yinxiang.com/doc/articles/enml.php

