---
layout: post
title: ERROR Watcher took too long to load
category: react-native
comments: true
---

在启动react server的时候，会有概率碰到ERROR Watcher took too long to load的错误，解决方法：
将node_modules/react-native/packager/react-packager/src/FileWatcher/index.js中的timeOut的值设置得大一些。