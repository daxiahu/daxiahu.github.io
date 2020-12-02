---
title: 记一次Javascript内存溢出事件
date: 2020-12-01 16:35:42
tags: 前端日常
cover: https://cdn.jsdelivr.net/gh/daxiahu/ImageBed@master/img/20201201163745.png
top_img: https://cdn.jsdelivr.net/gh/daxiahu/ImageBed@master/img/20201201145811.png
description: npm start运行项目时控制台总报错提示‘Javascript heap out of memory’怎么办😵
---
<p style="text-indent:2em">
今天要改一个上了年纪的preact项目，娴熟的进行了git clone => npm i => npm start三连击之后，等待building到94%的时候突然控制台报了一大堆错
</p>

![](https://cdn.jsdelivr.net/gh/daxiahu/ImageBed@master/img/20201201163745.png)

简单判断了一下应该是node报的错，尝试了切换node版本，重新npm i之后都没有解决，仔细查看了报错信息之后，发现一行比较有用的报错信息：

> **Javascript heap out of memory**

提示内存溢出，记得之前也遇到过类似的问题，最后通过修改node内存分配解决了。于是通过关键词上网百度了一番：

1. 在Node中通过JavaScript使用内存时只能使用部分内容,64位系统下约为1.4GB,32位系统下约为0.7GB.
2. 当我们在代码中声明变量并赋值时,所使用对象的内存就分配在堆中.如果已申请的堆空间内存不够分配新的对象,将继续申请堆空间,直到堆的大小超过V8的限制为止.
3. Node在启动时,可以通过设置参数来调整内存限制的大小.

```
node --max-old-space-size=1700 test.js //设置老生代内存空间最大值,单位为MB
node --max-new-space-size=1024 test.js //设置新生代内存空间最大值,单位为KB
```
V8 的垃圾回收策略主要基于分代式垃圾回收机制。所谓分代式，就是将内存空间分为新生代和老生代两种，然后采用不同的回收算法进行回收。

- 新生代空间中的对象为存活时间较短的对象，大多数的对象被分配在这里，这个区域很小但是垃圾回特别频繁 。新生代：64 位系统 和 32 位系统分别为 32M 和 16 M （from 和 to 空间各占一半）
- 老生代空间中的对象为存活时间长或常驻内存对象，大多数从新生代晋升的对象会被移动到这里。老生代：64 位系统 和 32 位系统分别为 1400M 和 700 M

这里附上原文链接[https://www.cnblogs.com/onepixel/p/7422820.html](https://www.cnblogs.com/onepixel/p/7422820.html)

于是我尝试了一下在script的node命令中使用了
–max_old_space_size=4096 这个参数，设置最大4gb的内存（**也可以通过打开cmd命令直接设置node 内存
setx NODE_OPTIONS --max_old_space_size=4096**） 
<div style="text-align:center">
<p style="color:red;font-size:24px">结果还是报错...</p>
<img width="300" src="https://cdn.jsdelivr.net/gh/daxiahu/ImageBed@master/img/20201202104719.png"/>
</div>
<br/>
<p style="text-indent:2em">
冷静下来后分析一波，如果不是内存的原因，在运行的时候报内存溢出要么就是代码里面有死循环导致内存溢出，要么就是node_modules中一些第三方库版本不正确，导致占用内存过大。代码应该没有问题，因为同事的项目可以跑起来，那么很可能是node_modules的问题。但是我试过重新安装node_modules也没有解决啊。此时想起了一个罪魁祸首，package-lock.json文件，估计是哪个憨憨提npm i更新了这个文件并提交了，导致某些三方库升级了，版本存在兼容问题。太坑了！！！于是我恢复了package-lock.json文件到上个版本，并重新npm i了一次，果然项目跑了起来。
</p>
<p style="text-indent:2em">
当你执行 npm i时，nodeJS会从你的package.json中读取所有的dependencies信息，package.json文件只记录你通过npm install方式安装的模块信息，而这些模块所依赖的其他子模块的信息不会记录。package-lock.json文件锁定所有模块的版本号(包括主模块和所有依赖子模块)。
</p>
<p style="text-indent:2em">
package.json文件只能锁定大版本，即版本号的第一位，不能锁定后面的小版本，你每次npm install时候拉取的该大版本下面最新的版本。所以为了稳定性考虑我们不能随意升级依赖包，而package-lock.json就是来解决包锁定不升级问题的。
</p>

所以如果不是出于项目需要，平时尽量不要将package-lock.json文件的改动提交上去，否则可能会导致其他小伙伴无法正常运行项目。
![](https://cdn.jsdelivr.net/gh/daxiahu/ImageBed@master/img/20201202110730.png)