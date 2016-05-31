---
layout:     post
title:      "github上fork后如何同步源的新更新"
subtitle:   ""
date:       2016-02-14 12:00:00
author:     "鸿杰"
catalog:    true
tags:
    - git
---

1. 只用github不用命令行的方法:

![](http://7u2qiz.com1.z0.glb.clouddn.com/cbd7f5298b0ceca142ec0487b4468add_b.jpg)
![](http://7u2qiz.com1.z0.glb.clouddn.com/3359d274bd9c4ade8d891b8717dab5a7_b.jpg)
![](http://7u2qiz.com1.z0.glb.clouddn.com/f19d82b82d307c86cb0f703b8dc4805e_b.jpg)
![](http://7u2qiz.com1.z0.glb.clouddn.com/1f354c1b8aa920142b965776a8fa1382_b.jpg)

这一页往下面拉:

![](http://7u2qiz.com1.z0.glb.clouddn.com/cf0f718887c6ff2e20d77884885dea13_b.jpg)

2. 更推荐命令行，流程如下：

首先要先确定一下是否建立了主repo的远程源：

```
git remote -v
```

如果里面只能看到你自己的两个源(fetch 和 push)，那就需要添加主repo的源：

```
git remote add upstream URL
git remote -v
```

然后你就能看到upstream了。

如果想与主repo合并：

```
git fetch upstream
git merge upstream/master
```