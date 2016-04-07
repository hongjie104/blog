---
layout: post
title: React-Native 实现增量热更新的思路(1)
category: react-native
comments: true
---

所谓热更新就是在**不重新安装**的前提下进行代码和资源的更新，相信在整个宇宙中还不存在觉得热更新不重要的程序猿。

增量热更新就更牛逼了，只需要把修改过和新增的代码和资源推送给用户下载即可，增量部分的代码和资源都比较小，所以整个热更新流程可以在用户**无感**的情况下完成，我已经想不到更好的更新方式可以让我装更大的逼了。

##### 一.实现脚本的热更新

###### 1.为什么可以热更新

简单地说，因为RN是使用脚本语言来编写的，所谓脚本语言就是不需要编译就可以运行的语言，也就是“即读即运行”。我们在“读”之前将之替换成新版本的脚本，运行时执行的便是新的逻辑了，稍微抽象一下，图片资源是不是也是“即读即运行”？所以脚本本质上和图片资源一样，都是可以进行热更新的。

###### 2.RN加载脚本的机制

要实现RN的脚本热更新，我们要搞明白RN是如何去加载脚本的。
在编写业务逻辑的时候，我们会有许多个js文件，打包的时候RN会将这些个js文件打包成一个叫index.android.bundle(ios的是index.ios.bundle)的文件，所有的js代码(包括rn源代码、第三方库、业务逻辑的代码)都在这一个文件里，启动App时会第一时间加载bundle文件，所以脚本热更新要做的事情就是替换掉这个bundle文件。

###### 3.生成bundle文件

我们在RN项目根目执行以下命令来得到bundle文件和图片资源:

```
react-native bundle --entry-file index.android.js --bundle-output ./bundle/index.android.bundle --platform android --assets-dest ./bundle --dev false
```

其中*--entry*是入口js文件，android系统就是index.android.js，ios系统就是index.ios.js，*--bundle-output*就是生成的bundle文件路径，*--platform*是平台，*--assets-dest*是图片资源的输出目录，这个在后面的图片增量更新中会用到，*--dev*表示是否是开发版本，打正式版的安装包时我们将其赋值为false。
生成的bundle文件体积还是不小的，空项目的话恐怕至少也有900K，所以我们将其打成zip包并放到web服务器上以供客户端去下载。

###### 4.下载bundle文件

下载文件可以使用原生语言来写，也可以使用js实现，我个人推荐使用[React Native FileTransfer](https://github.com/remobile/react-native-file-transfer)来实现下载功能。
实现方法很简单：

```
import FileTransfer from 'react-native-file-transfer';

let fileTransfer = new FileTransfer();
fileTransfer.onprogress = (progress) => {
  console.log(parseInt(progress.loaded * 100 / progress.total))
};
// url：新版本bundle的zip的url地址
// bundlePath：存在新版本bundle的路径
// unzipJSZipFile：下载完成后执行的回调方法，这里是解压缩zip
fileTransfer.download(url, bundlePath, unzipJSZipFile, (err) => {
    console.log(err);
  }, true
);
```

解压缩的工作我们可以使用[react-native-zip](https://github.com/remobile/react-native-zip)来完成。

```
import Zip from 'react-native-zip';

function unzipJSZipFile() {
  // zipPath：zip的路径
 // documentPath：解压到的目录
  Zip.unzip(zipPath, documentPath, (err)=>{
    if (err) {
      // 解压失败
    } else {
      // 解压成功，将zip删除
      fs.unlink(zipPath).then(() => {
        // 通过解压得到的补丁文件生成最新版的jsBundle
      });
    }
  });
}
```

解压成功后，我们使用[react-native-fs](https://github.com/johanneslumpe/react-native-fs)来将zip删除。

###### 5.替换bundle文件
安装包中的bundle文件是在asset目录下的，而asset目录我们是没有写权限的，所以我们不能修改安装包中的bundle文件。好在RN中提供了修改读取bundle路径的方法。以android为例(ios的类似)，在ReactActivity类中有这么一个方法：

```
/**
 * Returns a custom path of the bundle file. This is used in cases the bundle should be loaded
 * from a custom path. By default it is loaded from Android assets, from a path specified
 * by {@link getBundleAssetName}.
 * e.g. "file://sdcard/myapp_cache/index.android.bundle"
 */
protected @Nullable String getJSBundleFile() {
  return null;
}
```

该方法返回了一个自定义的bundle文件路径，如果返回默认值null，RN会读取asset里的bundle。我们在MainActivity类中重写这个方法，返回可写目录一下的bundle文件路径：

```
@Override
protected @Nullable String getJSBundleFile() {
    String jsBundleFile = getFilesDir().getAbsolutePath() + "/index.android.bundle";
    File file = new File(jsBundleFile);
    return file != null && file.exists() ? jsBundleFile : null;
}
```

如果可写目录下没有bundle文件，还是返回null，RN依然读取的是asset中的bundle，如果可写目录下存在bundle，RN就会读取可写目录下的bundle文件。

我们将下载好的zip解压到**getFilesDir().getAbsolutePath()**目录下，再次启动App时便会读取该目录下的bundle文件了，以后再有新版本的bundle文件，依然是下载、解压并覆盖掉这个bundler文件，至此，我们便完成了代码的热更新工作。

###### 6.图片不见了

当我们使用可写目录下的bundle文件时会出现一个很严重的问题：所有的本地图片资源都无法显示了。

我们的图片资源都是通过require来获取的：

```
<Image source={require('./imgs/test.png')} />
```

为了找到图片消失的原因，我们打开image.android.js或者image.ios.js，找到渲染图片的方法：

```
render: function() {
  var source = resolveAssetSource(this.props.source);
  var loadingIndicatorSource = resolveAssetSource(this.props.loadingIndicatorSource);
  // ...
}
```

原来是通过resolveAssetSource方法来获取资源，那么找到resolveAssetSource方法：

```
function resolveAssetSource(source: any): ?ResolvedAssetSource {
  if (typeof source === 'object') {
    return source;
  }

  var asset = AssetRegistry.getAssetByID(source);
  if (asset) {
    return assetToImageSource(asset);
  }

  return null;
}

function assetToImageSource(asset): ResolvedAssetSource {
  var devServerURL = getDevServerURL();
  return {
    __packager_asset: true,
    width: asset.width,
    height: asset.height,
    uri: devServerURL ? getPathOnDevserver(devServerURL, asset) : getPathInArchive(asset),
    scale: pickScale(asset.scales, PixelRatio.get()),
  };
}
```

又发现是通过getPathInArchive方法来获取资源的，那么继续找到getPathInArchive方法：

```
/**
 * Returns the path at which the asset can be found in the archive
 */
function getPathInArchive(asset) {
  var offlinePath = getOfflinePath();
  if (Platform.OS === 'android') {
    if (offlinePath) {
      // E.g. 'file:///sdcard/AwesomeModule/drawable-mdpi/icon.png'
      return 'file://' + offlinePath + getAssetPathInDrawableFolder(asset);
    }
    // E.g. 'assets_awesomemodule_icon'
    // The Android resource system picks the correct scale.
    return assetPathUtils.getAndroidResourceIdentifier(asset);
  } else {
    // E.g. '/assets/AwesomeModule/icon@2x.png'
    return offlinePath + getScaledAssetPath(asset);
  }
}
```

该方法的逻辑是如果有离线脚本，那么就从该脚本所在目录里寻找图片资源，否则就从asset中读取图片资源，所谓离线脚本就是我们刚刚下载并解压的bundle文件，而我们并没有将图片资源放在这个目录下，所以所有的图片都不见了。
找到原因就好办了，我们在使用bundle命令生成bundle文件的时候也将图片资源输出出来了，那打包bundle文件的时候我们将所有图片也一并打包进zip，客户端下载zip并解压缩后，客户端可写目录下也就有了**所有**的图片资源，这样就即实现了脚本的热更新又实现了图片的热更新。

##### 二.减小更新包体积

将一个完整bundle文件和所有图片都打成zip，zip的体积让人不敢直视。

###### 1.增量更新图片

每一次的版本更新我们都将所有图片装进zip包未免有点太任性了，其实我们只需要将修改过和新增的图片资源放进zip就行了。
我们修改一下获取图片资源的方法里的逻辑：

```
/**
 * Returns the path at which the asset can be found in the archive
 */
function getPathInArchive(asset) {
  var offlinePath = getOfflinePath();
  if (Platform.OS === 'android') {
    if (offlinePath) {
      // 热更新修改  开始
      if(global.patchList){
        let picName = `${asset.name}.${asset.type}`;
        for (let i = 0; i < global.patchList.length; i++) {
          if(global.patchList[i].endsWith(picName)){
            return 'file://' + offlinePath + getAssetPathInDrawableFolder(asset);
          }
        }
      }
      // 热更新修改  结束
      // E.g. 'file:///sdcard/AwesomeModule/drawable-mdpi/icon.png'
      // return 'file://' + offlinePath + getAssetPathInDrawableFolder(asset);
    }
    // E.g. 'assets_awesomemodule_icon'
    // The Android resource system picks the correct scale.
    return assetPathUtils.getAndroidResourceIdentifier(asset);
  } else {
    // E.g. '/assets/AwesomeModule/icon@2x.png'
    return offlinePath + getScaledAssetPath(asset);
  }
}
```

其中global.patchList是一个数组，里面放的是自安装包版本以来所有修改过和新增的图片名，如果访问的图片名在这个数组中就从离线脚本所在目录里寻找图片资源，否则还是从asset中寻找图片资源。
我们在打包zip的时候，就只装修改过和新增的图片，并将这些图片名记录在更新配置文件里，客户端去读取更新配置文件时将配置中的图片名读取到并生成global.patchList，这样我们的更新包就小了许多了。
这么做的缺点就是每次更新RN版本的时候，都需要修改下RN的源码，不过我觉得这点小麻烦还是可以接受的，毕竟已上线的产品，我们还是以稳定为主，能不升级RN就不升级RN。

###### 2.增量更新脚本

bundle文件的体积，我们也得想想办法去减少它。
有两种思路：

1.  分离bundle。bundle里存放了RN源码、第三方库代码和业务逻辑代码，其中频繁更新的就只有业务逻辑代码，所以我们将RN源码和第三方库代码打包成一个bundle，业务逻辑打包成一个bundle，热更新的时候就只更新业务逻辑的bundle即可。

2.  打包补丁文件。我们可以使用[bsdiff](http://www.daemonology.net/bsdiff/)对比两个版本的bundle文件得到差异文件，也就是“**补丁**”，客户端下载好补丁文件，将其与本地的bundle进行**融合**从而得到最新版本的bundle文件。

这里重点讲解第二个思路的做法。

1) 生成补丁。

我们从[bsdiff官网](http://www.daemonology.net/bsdiff/)上下载到最新的源码，然后进行编译就得到可执行的二进制文件了。

如果是win系统，可以直接到我的[百度网盘](http://pan.baidu.com/s/1bo4HFdX)下载，下载密码：zq1x。解压下载好的zip，使用命令行进入到bsdiff的目录，输入命令：

```
bsdiff a.txt b.txt c.pat
```

上面的命令就是生成a.txt、b.txt两个文件的补丁c.pat。

如果是linux系统，可以依次执行以下命令:

```
yum install bzip2-devel
wget http://www.daemonology.net/bsdiff/bsdiff-4.3.tar.gz
tar zxvf bsdiff-4.3.tar.gz
cd bsdiff-4.3
```

编译完成后，会在目录下生成2个二进制文件：bsdiff、bspatch，这2个二进制文件可以直接使用，不过推荐拷贝到/usr/local/sbin/下：

```
cp bsdiff /usr/local/sbin/
cp bspatch /usr/local/sbin/
```

这样就可以在命令行中直接使用了:

```
bsdiff a.txt b.txt c.pat
```

2) 使用补丁。
得到了补丁文件，下一步就会使用补丁了，拿上面的a.txt、b.txt、c.pat做测试：

```
bspatch a.txt d.txt c.pat
```

得到文件d.txt，将其开打看看是否和b.txt一样，如果一样，说明测试成功。

3) 在RN中使用bsdiff。
待续。。。

##### 三.制作一键热更新工具

待续。。。