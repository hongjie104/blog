---
layout:     post
title:      "win上搭建react-native android环境"
subtitle:   ""
date:       2015-11-10 12:00:00
author:     "鸿杰"
tags:
    - react-native
---

1. **安装JDK**

	既然是android开发，那么java肯定逃不掉了。从[Java官网](http://www.oracle.com/technetwork/java/javase/downloads/index-jsp-138363.html)下载安装包吧。我的电脑是64位的，所以我选择下载64位的安装包。

2. **安装Android SDK**

	先下载Android Studio，如果翻墙不方便的话，可以从[AndroidDevTools](http://androiddevtools.cn/)上下载，我是翻墙从官网下载的。安装过程中会提示选择SDK的目录，默认是在系统盘，我选择的是F/android/sdk。安装完成后，先要配置一下sdk Manager的网络设置，否则在不翻墙的情况下安装sdk会很慢很慢。腾讯Bugly的镜像就挺好的，[查看说明](http://android-mirror.bugly.qq.com:8080/include/usage.html)。

	安装以下项目：

	> * Tools/Android SDK Tools (24.4.1)
	> * Tools/Android SDK Platform-tools (23.0.1)
	> * Tools/Android SDK Build-tools (23.0.2)
	> * Android 6.0 (API 23)/SDK Platform (1)
	> * Extras/Android Support Library(23.1)

	最后在系统环境变量中加入ANDROID_HOME，路径为sdk目录。

3. **安装Git**

	下载[Git](https://git-for-windows.github.io/)，记得把git.exe的路径写入系统环境变量，因为在执行react-native init命名时会调用git去下载react-native的源码。

4. **安装Node.js**

	到[官网](https://nodejs.org/)下载最新版的安装包安装即可。npm（node package manager）是随着node.js就安装好的，为了加速安装其他的package，在cmd里输入以下命令：

	```
	npm config set registry https://registry.npm.taobao.org
	npm config set disturl https://npm.taobao.org/dist
	```

5. **安装react-native命令行工具**

	```
	npm install -g react-native-cli
	```

6. **创建react-native android项目**

	```
	react-native init MyProject
	```
	这一步要等上很长时间，原因不明。不想等待的话可以在[这里](http://react-native.cn/bbs/post/35)下载项目文件，然后将android目录下的gradle.properties文件中的sdk.dir设为本机上sdk的目录。

	```
	sdk.dir=F:\\android\\sdk
	```

7. **启动react-native服务**

	进入项目文件夹，输入启动命令

	```
	react-native start
	```
	在浏览器中访问地址：<http://localhost:8081/index.android.bundle?platform=android>
	第一次访问需要骚等一会，这是在生成android的bundle文件。**cmd窗口别关，一直保持开启状态**。

8. **安装并启动android模拟器**

	推荐[逍遥模拟器](http://www.xyaz.cn/)，安装好并启动。试用了很多模拟器，有些模拟器adb无法连接，有些模拟器adb连上了，但是在却没有“摇一摇”或者菜单键，非常蛋疼。终于让我发现了逍遥模拟器，嗯，很不错。启动模拟器，在命令行中输入

	```
	adb devices
	```
	看看adb是否和设备连接上了，如果没有连接上，那么需要手动连接，连接设备需要一个端口号，那么这个端口号是多少呢？聪明的我通过查找模拟器log找到了答案。打开模拟器安装目录，然后找到MEmu\MemuHyperv VMs\MEmu\Logs下的log文件，打开此log，通过查找关键词“ port ”（注意port两边都带上空格），可以发现：

	```
	00:00:00.277099 NAT: set redirect TCP host port 21505 => guest port 21505 @ 10.0.2.15
	00:00:00.278027 NAT: set redirect TCP host port 21504 => guest port 21504 @ 10.0.2.15
	00:00:00.278123 NAT: set redirect TCP host port 21501 => guest port 21501 @ 10.0.2.15
	00:00:00.278199 NAT: set redirect TCP host port 21500 => guest port 21500 @ 10.0.2.15
	00:00:00.278274 NAT: set redirect TCP host port 21502 => guest port 21502 @ 10.0.2.15
	00:00:00.278342 NAT: set redirect TCP host port 21503 => guest port 5555 @ 10.0.2.15
	```
	诶，找到了，端口号是21503，然后回到命令行，输入：

	```
	adb connect 127.0.0.1:21503
	```
	然后再一次输入

	```
	adb devices
	```
	adb是不是连上模拟器了？那么接下去就是在模拟器中运行我们的小demo了。

9. **在模拟器中运行**

	进入项目目录，输入命令：

	```
	react-native run-android
	```
	第一次运行时会需要下载一些东西，等待就行。
	build成功后便会在模拟器上自动运行了
	![react-native的hello world](http://7u2qiz.com1.z0.glb.clouddn.com/QQ截图20151111131648.png)
	如果是连真机的话，很大可能看到的结果是一片白啊一片白。经查，发现是安全中心中的“悬浮窗”权限并没有对我们的这个新app开放，那么将权限开放，重启app，啊呀，一片红啊一片红。。。长按物理菜单键或者死命摇一摇手机，会弹出一个小窗口，选择“Dev Settings”，然后选择“Debug server host for device”，会弹出一个输入框，输入电脑ip地址和默认的8081端口，再次重新app，啊呀，粗线了粗线了。。。