---
layout: post
title: React Native ListView sticky效果实现(转)
category: 技术
comments: true
---

[点击查看原文](http://www.ghugo.com/react-native-listview-stickyheaderindices/)

React Native中，ScrollView组件可以使用*stickyHeaderIndices*轻松实现*sticky*效果。

而使用ListView组件时，使用*stickyHeaderIndices*则不生效。

在IOS中的ListView的内部结构，实际是由多个Section组成，最典型的案例就是iphone手机的通讯录，滚动时每个section header会吸顶。

而在web端，使用*position : -weblit-sticky*实现的吸顶效果，也是类似的原理。具体可以看下之前的文章：
[《position:sticky 使用条件分析》](http://www.ghugo.com/position-sticky-how-to-use/)

好了，废话不多。在ListView中实现sticky，需要使用cloneWithRowsAndSections 方法，将dataBlob(object),sectionIDs (array),rowIDs (array) 三个值传进去即可。

##dataBlob

dataBlob包含ListView所需的所有数据（section header 和 rows），在ListView渲染数据时，使用 getSectionData 和 getRowData 来渲染每一行数据。dataBlob的key值包含sectionID + rowId

[](http://7qnca0.com1.z0.glb.clouddn.com/wp-content/uploads/2015/08/51.png)

##sectionIDs

sectionIDs 用于标识每组section。

[](http://7qnca0.com1.z0.glb.clouddn.com/wp-content/uploads/2015/08/52.png)

##rowIDs

[](http://7qnca0.com1.z0.glb.clouddn.com/wp-content/uploads/2015/08/53.png)

rowIDs 用于描述每个section里的每行数据的位置及是否需要渲染。在ListView渲染时，会先遍历rowIDs获取到对应的dataBlob数据。

根据上面3个数据的定义，模拟出对应的数据结构如下：

```
var dataBlob = {
     'sectionID1' : { ...section1 data },
     'sectionID1:rowID1' : { ...row1 data },
     'sectionID1:rowID2' : { ..row2 data },
     'sectionID2' : { ...section2 data },
     'sectionID2:rowID1' : { ...row1 data },
     'sectionID2:rowID2' : { ..row2 data },
     ...
}
 
var sectionIDs = [ 'sectionID1', 'sectionID2', ... ]
 
var rowIDs = [ [ 'rowID1', 'rowID2' ], [ 'rowID1', 'rowID2' ], ... ]
```

在DataSource中，告诉ListView获取row和section的方法。

```
var getSectionData = (dataBlob, sectionID) =&gt; {
      return dataBlob[sectionID];
 }
var getRowData = (dataBlob, sectionID, rowID) =&gt; {
      return dataBlob[sectionID + ':' + rowID];
}
this.ds = new ListView.DataSource({
      getSectionData: getSectionData,
      getRowData: getRowData,
      rowHasChanged: (r1, r2) =&gt; r1 !== r2,
      sectionHeaderHasChanged: (s1, s2) =&gt; s1 !== s2
})
```

最后将数据传进ListView

```
this.dataSource.cloneWithRowsAndSections(dataBlob, sectionIDs, rowIDs)
```

最终效果如图：

[](http://7qnca0.com1.z0.glb.clouddn.com/wp-content/uploads/2015/08/1.gif)

[点击查看完整代码](https://github.com/hugohua/rn-listview-example)

>参考文章:[http://moduscreate.com/react-native-listview-with-section-headers/](http://moduscreate.com/react-native-listview-with-section-headers/)