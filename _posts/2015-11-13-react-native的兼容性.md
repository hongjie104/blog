---
layout: post
title: react-native的兼容性（Android、Ios）
category: react-native
comments: true
---

*本文转自[segmentfault社区](http://segmentfault.com/a/1190000003883126)，作者：shane_xu*

刚开始学习RN的时候，写的代码只支持ios版本，写起来感觉还是比较顺手的，也没有太多的疑难杂症，以及模拟器不支持一些标签的情况，今天写了支持android版本的代码后，我整个人都不好了。。。

### 在定义导航的时候就出现了问题

如果是ios我们就可以用NavigatorIOS组件，创建方式如下：

```
'use strict';

var React = require('react-native');
var {
  AppRegistry,
  NavigatorIOS,
  StyleSheet,
} = React;

var Home = require('Home文件的路径');

var AwesomeProject = React.createClass({
  render: function() {
    return (
      
    );
  }
});
```


