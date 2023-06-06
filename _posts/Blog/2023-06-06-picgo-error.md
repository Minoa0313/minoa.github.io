---
layout: post
title:  "Typora通过PicGo上传图片失败原因"
date:   2023-06-06
category: Blog
---

> 很久以前就通过typora和PicGo，向github自动上传图片了。
>
> 近来又一次发生了上传失败的错误，解决后决定还是写下来以便之后查阅。

### 出现的问题

直接拖拽图片到PicGo，可以成功上传到GitHub。但是在typora中右键图片上传却失败了。

错误信息：Failed to fetch



### 解决方案

意外的简单，打开PicGo的设置，修改Server的监听端口为36677。

之前端口被错误地设为了366771，可能是由于启动了两次PicGo，它自动更改了端口。

