---
title: 关于 canvas 的范围机制的一些探索
date: 2019-04-09 20:37:32
tags: front-end
---
> 本篇博客是来自于我在项目中实际碰到的关于 canvas 绘图所遇到的一些问题

# canvas 的大小
* canvas 的大小分为两种，一种是我们平时所了解的，作为 css 属性的宽高，而另一种则是 canvas 本身的大小.
* canvas 本身的大小我们可以在 html 标签属性上的 width 和 height 进行设置，单位为像素，而默认大小是 300 *150
* canvas 元素可以使用CSS来定义大小，但在绘制时图像会伸缩以适应它的框架尺寸：如果CSS的尺寸与初始画布的比例不一致，它会出现扭曲。

# canvas 坐标机制
* canvas 的坐标起点为左上角，向右为 X 轴正方向，向下为 Y 轴正方向。
* canvas 的很多方法比如 fillRect(x, y, width, height)、strokeRect(x, y, width, height)、clearRect(x, y, width, height) 等方法所指的坐标和宽高值得都是 canvas 本身的宽高

# canvas 绘制图片
canvas 绘制的图片类型可以有以下几种:
* HTMLImageElement(这些图片是由Image()函数构造出来的，或者任何的 img 元素)
* HTMLVideoElement(用一个HTML的 video 元素作为你的图片源，可以从视频中抓取当前帧作为一个图像)
* HTMLCanvasElement(可以使用另一个 canvas 元素作为你的图片源)
* ImageBitmap(这是一个高性能的位图，可以低延迟地绘制，它可以从上述的所有源以及其它几种源中生成)

在 drawImage 方法参数有三种 
* void ctx.drawImage(image, dx, dy);
* void ctx.drawImage(image, dx, dy, dWidth, dHeight);
* void ctx.drawImage(image, sx, sy, sWidth, sHeight, dx, dy, dWidth, dHeight);
其中，各参数含义如下:
* dx: canvas 坐标，图片被绘制在 canvas 的什么地方
* sx: 图片的坐标，从什么地方开始绘制图片

# 其它注意点
* canvas 必须要有闭合标签