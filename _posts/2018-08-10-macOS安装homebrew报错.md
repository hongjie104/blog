---
layout:     post
title:      "macOS安装homebrew报错"
subtitle:   ""
date:       2018-08-10 12:00:00
catalog:    true
tags:
    - mac
    - homebrew
---

##### 安装

```
curl "https://nodejs.org/dist/latest/node-${VERSION:-$(wget -qO- https://nodejs.org/dist/latest/ | sed -nE 's|.*>node-(.*)\.pkg</a>.*|\1|p')}.pkg" > "$HOME/Downloads/node-latest.pkg" && sudo installer -store -pkg "$HOME/Downloads/node-latest.pkg" -target "/"
```

##### 报错

```
LibreSSL SSL_read: SSL_ERROR_SYSCALL, errno 54
```

##### 解决

1.执行下面这句命令，更换为中科院的镜像：

```
git clone git://mirrors.ustc.edu.cn/homebrew-core.git/ /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core --depth=1
```

2.把homebrew-core的镜像地址也设为中科院的国内镜像

```
cd "$(brew --repo)"
git remote set-url origin https://mirrors.ustc.edu.cn/brew.git
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git
```

##### 更新

```
brew update
```