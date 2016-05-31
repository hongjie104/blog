---
layout:     post
title:      "centOS上搭建NodeJS环境"
subtitle:   ""
date:       2016-02-15 12:00:00
author:     "鸿杰"
catalog:    true
tags:
    - nodejs
---

第一次使用linux系统，相关的知识积累可谓是一点都没有。但是木有关系，我们还有搜索引擎呢。

安装NodeJS有很多方式，大家比较推荐的方式是下载源码然后编译安装。

从[nodeJS下载列表](http://nodejs.org/dist/)中找到想要下载的版本地址，一般来说，第一次安装的话都是选择最新版。到[官网](http://nodejs.org)上可以查到有两个版本：V4.3.0LTS和V5.6.0，V4版本是长期维护版本，V5版本是稳定版本。我这里选择的是V4版本，那也可以从[这里](https://nodejs.org/dist/latest-v4.x/)找到V4版中的最新版本。

在linux的终端中使用wget命令下载源码压缩包

```
wget http://nodejs.org/dist/latest-v4.x/node-v4.3.0.tar.gz
```

然后解压缩

```
tar -xf node-v4.3.0.tar.gz
```

进入解压缩得到的目录

```
cd node-v4.3.0
```

进行编译

```
make && make install
```

编译中遇到一个警告:

> WARNING: C++ compiler too old, need g++ 4.8 or clang++ 3.4 (CXX=g++)

这是由于 CentOS 6 中编译 node4+ 需要高版本号的 C++ 编译器，而升级C++ 编译器貌似是一件挺麻烦的事情，至少我在yum中是无法升级C++ 编辑器。

于是走一条捷径，将已经编译好的二进制版本下载到linux中。

还是在[这里](https://nodejs.org/dist/latest-v4.x/)找到编译好的二进制版本:node-v4.3.0-linux-x64.tar.gz。

依然通过wget命令将其下载下来

```
wget http://nodejs.org/dist/latest-v4.x/node-v4.3.0-linux-x64.tar.gz
```

然后解压缩

```
tar -zvxf node-v4.3.0-linux-x64.tar.gz
```

重命名

```
mv node-v4.3.0-linux-x64 node
```

如果node目录不在opt目录下，那么我们将其移动到opt目录下

```
mv node /opt
```

接下去就是配置环境了。

打开etc下的profile文件

```
vi /etc/profile
```

在最后加入以下内容：

```
export NODE_HOME=/opt/nodejs
export PATH=$PATH:$NODE_HOME/bin
export NODE_PATH=$NODE_HOME/lib/node_modules
```

退出vi，再使用source命令让配置生效

```
source /etc/profile
```

此时就可以查看node版本号

```
node -v
```

如果版本号显示了，说明nodeJS已经安装完成了。

如果关闭终端再进去发现 node 环境变量失效了，那么修改~/.bash_profile

```
vi ~/.bash_profile
```

在最后加入以下内容：

```
export NODE_HOME=/opt/nodejs
export PATH=$PATH:$NODE_HOME/bin
export NODE_PATH=$NODE_HOME/lib/node_modules
```