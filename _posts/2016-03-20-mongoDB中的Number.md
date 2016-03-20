---
layout: post
title: mongoDB中的Number
category: mongoDB
comments: true
---

今天在命令窗口中使用了update方法来更新一个Number类型字段的值：

```
db.users.update({"uid": 1000}, {"$set": {"age": 36}})
```

更新完成后发现该字段的值被转成了double类型，奇了个怪了。
那么如何保证在数据库中存入整形的数字呢？
翻阅了一下mongoDB的[文档](https://docs.mongodb.org/v3.0/core/shell-types/)，原来有一个叫做NumberInt的方法，可以将输入的数字转成int类型。

```
db.users.update({"uid": 1000}, {"$set": {"age": NumberInt(36)}})
```

这样就行了。