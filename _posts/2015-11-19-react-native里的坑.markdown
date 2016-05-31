---
layout:     post
title:      "Reac-Native里的坑"
subtitle:   ""
date:       2015-11-19 12:00:00
author:     "shane_xu"
catalog:    true
tags:
    - react-native
---

当TouchableHighlight中包含Image并且Image包含Text时，点击时会报错:

```html
<TouchableHighlight onPress={this.onTryLogin} underlayColor="#d9f2f3">
	<Image style={ {justifyContent:'center'} } source={require('./../common/imgs/btnBg.png')}>
		<Text style={styles.loginText}>登录</Text>
	</Image>
</TouchableHighlight>
```

报错如图:
![报错图](http://7u2qiz.com1.z0.glb.clouddn.com/Screenshot_2015-11-19-12-59-48.png)

具体原因不明，解决方法，在Image外面再包一个View:

```html
<View>
	<Image style={ {justifyContent:'center'} } source={require('./../common/imgs/btnBg.png')}>
		<Text style={styles.loginText}>登录</Text>
	</Image>
</View>
```

java方法的参数，Callback类型的参数后面要么没有参数，要么全是Callback类型的，否则会引起闪退。原因不明。

错误的写法如下:

```java
@ReactMethod
public void showDatepicker(Callback successCallback, int year, int month, int day, ) {
	DialogFragment dateDialog = new DatePicker(successCallback, year, month, day, );
	dateDialog.show(mActivity.getSupportFragmentManager(), "datePicker");
}
```

正确的写法如下:

```java
@ReactMethod
public void showDatepicker(int year, int month, int day, Callback successCallback) {
	DialogFragment dateDialog = new DatePicker(year, month, day, successCallback);
	dateDialog.show(mActivity.getSupportFragmentManager(), "datePicker");
}
```

precomputeStyle.js找不到。不知道从哪个版本开始，RN将precomputeStyle.js给删除了，那某些第三方组件用到了precomputeStyle就会报错，怎么破？将precomputeStyle( … )  改为   { style: { … } }就行了。

例如：

```js
// 旧版写法
this.scrollView.setNativeProps(precomputeStyle({
	transform: [{translateX: -1 * offsetX}],
}));
```
改成:

```js
// 新版写法
this.scrollView.setNativeProps({style:{transform: [{translateX: -1 * offsetX}]}});
```
