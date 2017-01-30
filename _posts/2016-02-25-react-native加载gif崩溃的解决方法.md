---
layout:     post
title:      "react-native加载gif崩溃的解决方法"
subtitle:   ""
date:       2016-02-25 12:00:00
author:     "鸿杰"
tags:
    - react-native
---

由于react-native中image的原生实现为[fresco](http://www.fresco-cn.org/)，在加载gif图片时crash。具体原因：[Load gif lib error](https://github.com/facebook/fresco/issues/209)

解决方法，在android的proguard文件中加入：

```
-keep class com.facebook.imagepipeline.gif.** { *; }

-keep class com.facebook.imagepipeline.webp.** { *; }
```