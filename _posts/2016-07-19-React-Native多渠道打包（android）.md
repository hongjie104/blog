---
layout:     post
title:      "React-Native多渠道打包（android）"
subtitle:   ""
date:       2016-07-19 12:00:00
author:     "鸿杰"
tags:
    - react-native
---

在AndroidManifest.xml中一般都会有**当前渠道**的配置信息，以talkingData为例：

```html
<meta-data android:name="TD_CHANNEL_ID" android:value="渠道名" />
```

所以在打包之前我们得有一个自动修改**当前渠道**的脚本。下面以python为例:

```python
# 将指定文件中符合指定正则的string替换为replaceStr
def modifyConfig(filePath, pattern, replaceStr):
	file = codecs.open(filePath, 'r', 'utf-8')
	fileContent = file.read()
	file.close()

	fileContent = re.sub(pattern, replaceStr, fileContent)

	file = codecs.open(filePath, 'wb', 'utf-8');
	file.write(fileContent)
	file.close()

# 修改AndroidManifest
modifyConfig(
	r".\app\src\main\AndroidManifest.xml",
	re.compile(r'<meta-data android:name="TD_CHANNEL_ID" android:value=".+" />'),
	u'<meta-data android:name="TD_CHANNEL_ID" android:value="新的渠道" />'
)	
```

然后加上打包指令：gradlew assembleRelease

```python
# 开始打包
ps = subprocess.Popen("gradlew assembleRelease", shell = True)
ps.wait()
```

将该python文件移动到RN根目录中的android目录下，然后执行，不出意外的话，将会生成一个渠道名为**新的渠道**的安装包。那么问题来了，生成的安装包名都是**app-release.apk**，如果批量生成多个安装包，那不岂不是会相互覆盖吗？

so，下一步便是修改生成apk的文件名。打开**android\app\build.gradle**文件，将applicationVariants.all的逻辑修改如下：（从*def outputFile = output.outputFile*开始是新加的逻辑）

```
applicationVariants.all { variant ->
    variant.outputs.each { output ->
        // For each separate APK per architecture, set a unique version code as described here:
        // http://tools.android.com/tech-docs/new-build-system/user-guide/apk-splits
        def versionCodes = ["armeabi-v7a":1, "x86":2]
        def abi = output.getFilter(OutputFile.ABI)
        if (abi != null) {  // null for the universal-debug, universal-release variants
            output.versionCodeOverride =
                    versionCodes.get(abi) * 1048576 + defaultConfig.versionCode
        }
        
        def outputFile = output.outputFile
        if (outputFile != null && outputFile.name.endsWith('.apk')) {
            //这里修改apk文件名
            def fileName = outputFile.name.replace("app", "测试");
            output.outputFile = new File(outputFile.parent, fileName)
        }
    }
}
```

新增加的逻辑便是修改apk的文件名了。

同时在打包python里加上修改build.gradle的逻辑，以便让AndroidManifest.xml中的渠道名就是build.gradle中的apk文件名。

```
# 修改build.gradle
modifyConfig(
	r".\app\build.gradle",
	re.compile(r'def fileName = outputFile.name.replace\("app", ".+"\);'),
	u'def fileName = outputFile.name.replace("app", "%s");' % channel
)
```

此时，再执行python，便会得到名为**新的渠道-release.apk**的安装包了。

最后便是加入多个渠道的打包逻辑，完整的python脚本如下：

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-
# 

'''
多渠道自动打包工具
'''

__author__ = "zhouhongjie@apowo.com"

import os, re, codecs, subprocess

# 渠道列表
channelList = ["snpole", u"应用宝", "360"]

def modifyConfig(filePath, pattern, replaceStr):
	file = codecs.open(filePath, 'r', 'utf-8')
	fileContent = file.read()
	file.close()

	fileContent = re.sub(pattern, replaceStr, fileContent)

	file = codecs.open(filePath, 'wb', 'utf-8');
	file.write(fileContent)
	file.close()

def createAPK(channel):
	# 修改AndroidManifest
	modifyConfig(
		r".\app\src\main\AndroidManifest.xml",
		re.compile(r'<meta-data android:name="TD_CHANNEL_ID" android:value=".+" />'),
		u'<meta-data android:name="TD_CHANNEL_ID" android:value="%s" />' % channel
	)
	# 修改build.gradle
	modifyConfig(
		r".\app\build.gradle",
		re.compile(r'def fileName = outputFile.name.replace\("app", ".+"\);'),
		u'def fileName = outputFile.name.replace("app", "%s");' % channel
	)
	
	# 开始打包
	ps = subprocess.Popen("gradlew assembleRelease", shell = True)
	ps.wait()

if __name__ == '__main__':
	for x in channelList:
		createAPK(x);
	print(u"搞定了")
```

我这个是比较简单粗暴的打包脚本了，看上去似乎没啥技术含量，但是其实很多问题只要解决了，不管技术方案如何，那就是个好的方案。

那有没有更高大上的解决方案呢？诺，拿去：
[packer-ng-plugin](https://github.com/mcxiaoke/packer-ng-plugin)