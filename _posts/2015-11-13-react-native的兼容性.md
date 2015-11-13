---
layout: post
title: react-native的兼容性（Android、Ios）
category: react-native
comments: true
---

*本文转自[segmentfault社区](http://segmentfault.com/a/1190000003883126)，作者：shane_xu*

刚开始学习RN的时候，写的代码只支持ios版本，写起来感觉还是比较顺手的，也没有太多的疑难杂症，以及模拟器不支持一些标签的情况，今天写了支持android版本的代码后，我整个人都不好了。。。

### 在定义导航的时候就出现了问题

如果是ios我们就可以用NavigatorIOS组件，创建方式如下1：

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
      <NavigatorIOS
        style={styles.container}
        initialRoute={{
          title: '页面标题',
          component: Home,
        }}
      />
    );
  }
});

var styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: 'white',
  },
});

AppRegistry.registerComponent('AwesomeProject', () => AwesomeProject);
```
几行代码轻松搞定。

而到了android那就不是那么简单的了，需要用Navigator组件来搞定，而且你必须要设置renderScene属性，来设置各个页面的跳转路由，创建方式如下：

```
//index.android.js

'use strict';

var React = require('react-native');

var Home = require('文件的路径');
var Others = require('文件的路径');

var {
    AppRegistry,
    Navigator,
    StyleSheet,
    View,
    BackAndroid,
    ToolbarAndroid,
    } = React;

var _navigator;
BackAndroid.addEventListener('hardwareBackPress', () => {
  if (_navigator && _navigator.getCurrentRoutes().length > 1) {
    _navigator.pop();
    return true;
  }
  return false;
});

var RouteMapper = function(route, navigationOperations) {
  _navigator = navigationOperations;
  if (route.name === 'Home') {
    return (
        <View style={{flex:1}}>
          <ToolbarAndroid
              actions={[]}
              navIcon={require('image!android_back_white')}
              onIconClicked={navigationOperations.pop}
              style={styles.toolbar}
              titleColor="white"
              title="页面标题" />
          <Home navigator={navigationOperations} />
        </View>
    );
  }
  else if (route.name === 'Others') {
    return (
        <View style={{flex: 1}}>
          <ToolbarAndroid
              actions={[]}
              navIcon={require('image!android_back_white')}
              onIconClicked={navigationOperations.pop}
              style={styles.toolbar}
              titleColor="white"
              title={route.Others.title} />
          <Others
              style={{flex: 1}}
              navigator={navigationOperations}
              Others={route.Others}
              />
        </View>
    );
  }
};

var AwesomeProject = React.createClass({
  render: function() {
    var initialRoute = {name: 'Home'};
    return (
        <Navigator
            style={styles.container}
            initialRoute={initialRoute}
            configureScene={() => Navigator.SceneConfigs.FadeAndroid}
            renderScene={RouteMapper}
        />
    );
  }
});

var styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff'
  },
  toolbar: {
    backgroundColor: '#a9a9a9',
    height: 56,
  }
});

AppRegistry.registerComponent('AwesomeProject', () => AwesomeProject);

//Home文件里会有这么一段代码：
this.props.navigator.push({
    title:responseText.data.title,
    name:'Others',
    Others:responseText.data
});
```
上面代码中，route.name对应Home文件里的name，ToolbarAndroid是android的导航条，只有页面里添加这个标签才能有导航条否则默认是没有导航条的，Home文件里的Others是相关Others页面的数据。

### 如何判断设备是ios还是android

下面这段代码轻松搞定这件事：

```
var {
    Platform
} = React;

if(Platform.OS === 'ios'){
    //ios相关操作
}else{
    //android相关操作
}
```

### 弹出框

```
//ios
alert();

//android
var {
    ToastAndroid
} = React;

ToastAndroid.show('提示的信息', ToastAndroid.SHORT);
```

### TextIput

ios默认无下划线的，并且文字垂直居中，而在android里，看下面代码：

```
<TextInput
    underlineColorAndroid = "transparent"  //android需要设置下划线为透明才能去掉下划线
    textAlignVertical = "top"  //设置垂直位置
>
</TextInput>
```

### android不支持WebView标签
