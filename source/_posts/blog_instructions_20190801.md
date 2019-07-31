---
title: 博客使用说明
tags: 
  - acm队伍相关
  - 教程
categories:
  - acm队伍相关
  - 基本说明
date: 2019-8-1
top: True
---



本博客基于hexo melody主题搭建，队员初次构建个人博客可以参考此文章



# 博客

博客由markdown格式书写，文件名统一规定为`博客英文名_20xx-xx-xx.md`，文件名请使用英文，不熟悉markdown的请先学习[markdown基础用法](https://dcmtruman.github.io/2019/07/31/markdown_base_20190731/)

完成之后使用个人在github文件到[该网站目录下](https://github.com/DcmTruman/DcmTruman.github.io/tree/blog_source/source/_posts),记得上传的时候填写commit

![上传博客](https://b2.bmp.ovh/imgs/2019/08/f3c5efed2fbefa5e.png)

稍等片刻，travis CI将会自动构建新的博客，延迟大概在五分钟

**注意** ：博客在首页预览时会自动节选前150字，但会影响排版，如果有美观需求，请手动添加`<!-- more -->`决定预览位置
<!-- more -->

# 文件头：文章名、目录、标签和日期

任何一个文章都有它的出生地，我们需要在每篇博客前加上头文件形如，将`top`属性置位`True`可以实现**置顶效果**

```yml
---
title: 博客标题
tags: 
  - 标签1
  - 标签2
categories:
  - 一级目录
  - 二级子目录
  - 三级子目录
date: 2019-3-17
top: False
---
```

关于目录结构暂时如下

```
站点:.       
├─acm队伍相关 
│  ├─基本说明 
│  ├─训练计划 
│  └─队伍总结 
├─cm      
│  ├─acm题解
│  ├─日报
│  └─......  
├─key      
│  ├─acm题解
│  ├─日报
│  └─......   
├─xyw      
│  ├─acm_题
│  ├─日报
│  └─......
└─......  
```

为了方便检索管理，统一部分标签的使用

- cm/xyw/key：文章作者标签

- acm题解：与acm题题解相关的博客
- 学习笔记：学习笔记
- 每日总结：每日总结
- 队伍总结：与队伍相关的总结
- 总结 or 心得 ：总结或者心得
- 教程：发布个人教程
- 杂谈：与学习知识没有直接相关的，比如思考的某些事，作品观后感，旅游心得或者自己最近的想法等，都行，灵活使用

其他标签大家随意发挥

# 图片

如果已经学会使用markdown语法了，肯定知道，在插入图片的时候是需要图片地址的，特此规定一下，**禁止将博客图片直接放到词博客项目的github地址上**，可以push到个人仓库，也可以用一些图片资源托管网站注入[ImgURL](https://imgurl.org/)

![](https://b2.bmp.ovh/imgs/2019/08/b1719bf3683d8511.png)

# 幻灯片

如果是纯幻灯片，基础用法如下,注意文件开头的`layout: slides`部分,[预览地址](https://dcmtruman.github.io/2019/07/30/test_slide/)

```markdow	
---
title: 一个幻灯片测试用例
tags: 
  - 功能测试
categories:
  - 功能测试
date: 2019-7-31
layout: slides
slide:
  theme: night
---


# 按理说这是第一页

- 1.1
- 1.2

--

- 2.3
- 2.3

---

# 按理说这是第二部分

- 3.2
- 3.3

---

hahaha
```
