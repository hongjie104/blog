---
layout:     post
title:      "TypeScript的export和static的方法的效率比较"
subtitle:   ""
date:       2015-11-06 12:00:00
author:     "鸿杰"
tags:
    - egret
    - TypeScript
---


工具类的方法呢在TypeScript中既可以使用export的方式，也可以在一些类中标记为静态，那么这两种方式我们该选择哪一种呢？

写了个二分法的方法

```js
module libra.utils.queryUtil {
    /**
     * 通过export导出
     */
    export function queryByType(ary: Array<any>, val: number, property: string = 'type'):any {
        if(!ary) return null;
        var leftIndex: number = 0, middleIndex: number = 0;
        var rightIndex: number = ary.length - 1;
        while(rightIndex >= leftIndex) {
            middleIndex = (rightIndex + leftIndex) >> 1;
            if(ary[middleIndex][property] > val) {
                rightIndex = middleIndex - 1;
            } else {
                leftIndex = middleIndex + 1;
            }
        }
        return ary[leftIndex - 1];
    };
    
    export class T {

        public constructor(){
            
        }
        
        /**
         * 通过静态的方式让别的实例使用
         */
        public static queryByType(ary: Array<any>, val: number, property: string = 'type'): any {
            if(!ary) return null;
            var leftIndex: number = 0, middleIndex: number = 0;
            var rightIndex: number = ary.length - 1;
            while(rightIndex >= leftIndex) {
                middleIndex = (rightIndex + leftIndex) >> 1;
                if(ary[middleIndex][property] > val) {
                    rightIndex = middleIndex - 1;
                } else {
                    leftIndex = middleIndex + 1;
                }
            }
            return ary[leftIndex - 1];
        }
    };
}
```

两个方法分别运行10W次，得到的耗时结果：
export方式：35-40毫秒
static方式：30-35毫秒

可见，性能消耗差别不是特别大，所以结论是，**爱用哪种方式就用哪种方式。**