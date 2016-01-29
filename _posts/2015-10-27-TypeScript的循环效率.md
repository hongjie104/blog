---
layout: post
title: TypeScript的循环效率
category: TypeScript
comments: true
---


在TypeScript中的循环语句无非for和while，写了个测试看看哪个比较快一些。

```
class Test{
    public constructor(main:Main){
        var t:number = egret.getTimer();
        for(var i: number = 0;i < 900000; i++){
            this.t();
        }
        console.log("for 耗时:", egret.getTimer() - t);
        
        t = egret.getTimer();
        var j = 900000;
        while(--j > -1){
            this.t();
        }
        console.log("while 耗时:", egret.getTimer() - t);
    }
    
    private t():void{
        var a: number = 0;
        a++;
    }
}
```

输出如下：

```
for 耗时: 10
Test.js:17 while 耗时: 2
```

由此，以后都用while来循环吧。