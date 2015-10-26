---
layout: post
title: 使用WebStorm做为Egert的开发环境
category: 技术
comments: true
---

试用了一会Egret官方提供的Egret Wing 2.0做为Egret的开发IDE，发现其书写体验还有很大的提升空间。那么试一试WebStorm吧，毕竟别人是收费软件并且经过这么多年的发展，理论上应该是不错的。

1. **第一步，下载[WebStorm](http://www.jetbrains.com/webstorm/)**
2. **大家都懂的**

```
WebStorm注册码
User Name：
EMBRACE
License Key:
===== LICENSE BEGIN =====
24718-12042010  
00001h6wzKLpfo3gmjJ8xoTPw5mQvY  
YA8vwka9tH!vibaUKS4FIDIkUfy!!f  
3C"rQCIRbShpSlDcFT1xmJi5h0yQS6
===== LICENSE END =====
------来自网络，如果侵犯您的权利请留言或联系我，我会及时删除
```

3. **直接点open 找到你的 Egret 的一个工程的目录**
 *（或者通过cmd egret create testpro 创建一个项目，再找到这个项目，通过cmd命令创建的Egret项目就在，cmd create的当前目录，例如你：cd c:/EgretWorkSapce这个目录下创建）*

4. **添加一个custom的FileWatcher**
![](http://www.4yue.net/content/uploadfile/201505/916b1430585397.png)

5. **配置FileWatcher**
![](http://www.4yue.net/content/uploadfile/201505/825f1430586835.png)

6. **调试 egret 项目**
如果不习惯使用 Chrome 浏览器进行调试，而是更希望在 IDE 里直接调试，可以遵循如下步骤进行设置
1) 右键单击 launcher/index.html
2) 单击 Debug "index.html"
3) IDE 要求去chrome商店安装插件
4) DONE
