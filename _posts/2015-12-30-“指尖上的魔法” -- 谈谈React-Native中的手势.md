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

原生应用之所以为原生，和web应用相比，有两个比较主要的区别：

1. 原生应用会对触摸事件作出响应，也就是“点击态”；
2. 原生应用可以选择中途撤销触摸事件；

前面一点比较清楚，第二点选择中途撤销是什么意思呢？举个最简单的例子，用微信聊天的时候，点击了一个好友，可以进入聊天界面，但是如果我点中了一个好友，突然又不想和他聊天了，我会多按一会，然后将手指划开，这样就可以撤销刚才的触摸事件，就好像根本就没有点击过一样。平时使用得太习惯，可能没有意识到原来这个操作是撤消了触摸事件，现在回过头一想，还真是这么一回事。

通过前面的实验，我们可以对press,pressIn,pressOut,longPress事件的触发条件和触发顺序有一个比较清晰的了解：

1. 快速点击，只会触发press事件
2. 只要在点击时有一个“按”的操作，就是比快速点击要按的久一点，就会触发pressin事件
3. 如果同时绑定了pressIn,pressOut和press事件，那么当pressIn事件触发之后，如果用户的手指在绑定的组件中释放，那么接着会连续触发pressOut和press事件，此时的顺序是pressIn -> pressOut -> press。而如果用户的手指滑到了绑定组件之外才释放，那么此时将会不触发press事件，只会触发pressOut事件，此时的顺序是pressIn -> pressOut。后一种情况就是前面所说的中途取消，如果我们将回调函数绑定给press事件，那么后一种情况中回调函数并不会被触发，相当于"被取消"。
4. 如果绑定了longPress事件，那么在pressIn事件被触发之后，press事件不会被触发，通过打点计时，可以发现longPress事件的触发时间大概是在pressIn事件发生383ms之后，当longPress事件触发之后，无论用户的手指在哪里释放，都会接着触发pressOut事件，此时的触发顺序是 pressIn -> longPress -> pressOut

以上内容就是对React-Native对Touch事件的实现和用法分析，对于大部分应用来说，使用这四个Touch*组件再配合4个press事件就能对用户的手势进行响应。但是对于比较复杂的交互，还是得使用React-Native中的gesture responder system。

##gesture responder system

在React Native中，响应手势的基本单位是responder，具体来说，就是最常见的View组件。任何的View组件，都是潜在的responder，如果某个View组件没有响应手势操作，那是因为它还没有被“开发”。

将一个普通的View组件开发成为一个能响应手势操作的responder，非常简单，只需要按照React Native的gesture responder system的规范，在props上设置几个方法即可。具体如下：

1. View.props.onStartShouldSetResponder
2. View.props.onMoveShouldSetResponder
3. View.props.onResponderGrant
4. View.props.onResponderReject
5. View.props.onResponderMove
6. View.props.onResponderRelease
7. View.props.onResponderTerminationRequest
8. View.props.onResponderTerminate

乍看之下，这几个方法名字又长有奇怪，但是当了解了React Native对手势响应的流程之后，记忆这几个方法也非常容易。

要理解React Native的手势操作过程，首先要记住一点：
**一个React Native应用中只能存在一个responder**
正因为如此，gesture responder system中才存在reject和terminate方法。React Native事件响应的基本步骤如下：

1. 用户通过触摸或者滑动来“激活”某个responder，这个步骤由View.props.onStartShouldSetResponder以及View.props.onMoveShouldSetResponder这两个方法负负责处理，如果返回值为true，则表示这个View能够响应触摸或者滑动手势被激活
2. 如果组件被激活，View.props.onResponderGrant方法被调用，一般来说，这个时候需要去改变组建的底色或者透明度，来表示组件已经被激活
3. 接下来，用户开始滑动手指，此时View.props.onResponderMove方法被调用
4. 当用户的手指离开屏幕之后，View.props.onResponderRelease方法被调用，此时组件恢复被触摸之前的样式，例如底色和透明度恢复之前的样式，完成一次手势操作

综上所述，一次正常的手势操作的流程如下所示：

响应touch或者move手势 -> grant（被激活） -> move -> release(结束事件)

来段简单的示例代码：

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
} = React;

var pan = React.createClass({
  getInitialState(){
      return {
        bg: 'white'
      }
  },
  componentWillMount(){
    this._gestureHandlers = {
      onStartShouldSetResponder: () => true,
      onMoveShouldSetResponder: ()=> true,
      onResponderGrant: ()=>{this.setState({bg: 'red'})},
      onResponderMove: ()=>{console.log(123)},
      onResponderRelease: ()=>{this.setState({bg: 'white'})}
    }
  },
  render: function() {
    return (
      <View style={styles.container}>
        <View
          {...this._gestureHandlers}
          style={[styles.rect,{
            "backgroundColor": this.state.bg
          }]}></View>
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
  rect: {
    width: 200,
    height: 200,
    borderWidth: 1,
    borderColor: 'black'
  }
});

AppRegistry.registerComponent('pan', () => pan);
```

运行这段代码，当中间的正方形被激活时，底色变为红色，release之后，底色又变为白色。
![](https://cloud.githubusercontent.com/assets/2700425/9908259/7d33ddea-5cc5-11e5-8191-88de055a14d1.gif)

上面是正常事件响应流程，但是当应用中存在不止一个手势responder的时候，事情可能就复杂起来了。比如应用中存在两个responder，当使用一个手指激活一个responder之后，又去激活另一个responder会怎么样？因为React Native应用中只存在一个Responder，此时就会出现responder互斥的情况。具体来说过程如下：

1. 一个responder已经被激活
2. 第一个responder还没有被release，用户去尝试去激活第另一个responder
3. 后面将要被激活的responder去和前面还没有被释放的responder“协商”：兄弟，你都被激活这么久了，让我也活动一下呗？结果两种情况：
1)前面的responder比较“强硬”，非要占据唯一的responder的位置
2)前面的responder比较“好说话”，主动release

4. 前面一种情况，后面的responder的onResponderReject方法被调用，后面的responder没有被激活
5. 后面一种情况，后面的responder被激活，onResponderGrant方法被调用 ，前面的responder的onResponderTerminate方法被调用，前面的responder的状态被释放