---
layout: post
title: “指尖上的魔法” -- 谈谈React-Native中的手势(转)
category: react-native
comments: true
---

![](https://cloud.githubusercontent.com/assets/2700425/9882570/d93d55ee-5c07-11e5-9dca-c3822e6b7eb9.png)
React-Native是一款由Facebook开发并开源的框架，主要卖点是使用JavaScript编写原生的移动应用。从2015年3月份开源到现在，已经差不多有半年。目前，React-Native正在以几乎每周一个版本的速度进行快速迭代，开源社区非常活跃。2015年9月15日，React-Native正式宣布支持安卓，并在项目主页中更新了相关文档，这意味着React-Native已经完全覆盖了目前主流的iOS和Android系统，做到了“learn once，write everywhere”。React-Native能否颠覆传统的APP开发方式，现在下结论还为时尚早，但从微博和Twitter对React-Native相关消息的转发数量和评论来看，React-Native在未来的一段时间内都将是移动开发的热点。

从我自己的实际使用经历出发，在使用React-Native写了几个Demo之后，我觉得React-Native是一个非常有前途的框架。虽然目前文档并不能做到面面俱到，实际使用过程中坑也略多，但是填坑的速度也非常快。在项目的issues中提一个issue，基本都能在几个小时之内获得解答或者解决方案。因此，我决定比较系统的学习一下React-Native。

在移动应用开发中，手势是不可忽视的一个重要组成部分，React-Native针对应用中的手势处理，提供了gesture responder system，从最基本的点击手势，到复杂的滑动，都有现成的解决方案。

和以往的Hybrid应用相比，使用React-Native开发的原生应用的一大优势就是可以流畅的响应用户的手势操作，这也是使用React-Native相比以往在原生应用中插入webview控件的一个优势，因此，相比web端的手势，React-Native应用中的手势要复杂得多。我在初次接触React-Native手势之初也是看的一头雾水，经过搜索也发现相关的资料比较少，因此萌发了写一篇相关文章的想法。这也是写作本文的初衷，一方面总结自己学习和摸索的经验，以作为后来使用中的备忘录，另一方面也作为交流分享之用。

##Touch*手势

移动应用中最简单的手势，就是touch手势，而这也是应用中最常使用的手势，类比web开发中的事件，就好比web开发中的click。在web开发中，浏览器内部实现了click事件，我们可以通过onclick或者addEventListener('click',callback)来绑定click事件。React-Native也针对Touch手势进行了类似的实现，在React-Native中，一共有四个和Touch相关的组件：

1. TouchableHighlight
2. TouchableNativeFeedback
3. TouchableOpacity
4. TouchableWithoutFeedback

使用这四个组件，我们就可以在应用的某个部分绑定上Touch事件，来个简单的例子：

```
/**
 * Sample React Native App
 * https://github.com/facebook/react-native
 */
'use strict';

var React = require('react-native');
var {
  AppRegistry,
  StyleSheet,
  Text,
  View,
  TouchableHighlight
} = React;

var gesture = React.createClass({
  _onPressIn(){
    this.start = Date.now()
    console.log("press in")
  },
  _onPressOut(){
    console.log("press out")
  },
  _onPress(){
    console.log("press")
  },
  _onLonePress(){
    console.log("long press "+(Date.now()-this.start))
  },
  render: function() {
    return (
      <View style={styles.container}>
        <TouchableHighlight
          style={styles.touchable}
          onPressIn={this._onPressIn}
          onPressOut={this._onPressOut}
          onPress={this._onPress}
          onLongPress={this._onLonePress}>
          <View style={styles.button}>
          </View>
        </TouchableHighlight>
      </View>
    );
  }
});

var styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#F5FCFF',
  },
  button:{
    width: 200,
    height: 200,
    borderRadius: 100,
    backgroundColor: 'red'
  },
  touchable: {
    borderRadius: 100
  }
});

AppRegistry.registerComponent('gesture', () => gesture);
```

在上面的代码中，主要部分就是一个作为容器的View，一个作为按钮的View，因为想要给这个按钮绑定Touch手势，因此使用了TouchableHighlight这个和Touch相关的组件，将它作为按钮的一个包裹，在这个包裹的props中规定相应的回调即可。

前面提到了和Touch相关的组件一共有四个，它们的基本用法都很类似，只是实现的功能不太相同。先说最常用的TouchableHighlight，这个组件的作用，除了给内部元素增加绑定事件之外，还负责给内部元素增加“点击态”。所谓的“点击态”，就是在用户在点击的时候，会产生一个短暂出现覆盖层，用来告诉用户这个区块被点击到了。TouchableNativeFeedback这个组件只能用在安卓上，它可以针对点击在点击区域中显示不同的效果，例如最新安卓系统中的Material Design的点击波纹效果。TouchableOpacity这个组件用来给为内部元素在点击时添加透明度。TouchableWithoutFeedback这个组件只响应touch手势，不增加点击态，不推荐使用。

四个组件的用法大致相同，具体的细节方法可以参看具体文档。回头看上面的代码，在iOS模拟器中运行的效果图如下：

![](https://cloud.githubusercontent.com/assets/2700425/9883811/e480a378-5c0e-11e5-9937-5ac90b19690a.png)

在上面的代码中，TouchableHighlight组件上绑定了4个方法：

1. onPress
2. onPressIn
3. onPressOut
4. onLonePress

这4个方法也是React-Native帮助用户实现的4个手势，通过在4个相应的回调函数中输出不同的内容，我们可以研究4个手势出现的条件和顺序。打开chrome debug模式，点击模拟器中的按钮，可以看到浏览器控制台里面的输出内容:

![](https://cloud.githubusercontent.com/assets/2700425/9884001/052eb96a-5c10-11e5-9800-6452f3b95ced.gif)