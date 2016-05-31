---
layout:     post
title:      "ERROR Watcher took too long to load"
subtitle:   ""
date:       2016-01-05 12:00:00
author:     "鸿杰"
tags:
    - react-native
---

在启动react server的时候，会有概率碰到**ERROR Watcher took too long to load**的错误，解决方法：
将**node_modules/react-native/packager/react-packager/src/FileWatcher/index.js**中的**timeOut**的值设置得大一些。