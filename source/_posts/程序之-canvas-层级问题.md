---
title: 【已解决】小程序之 canvas 层级问题
date: 2019-07-08 11:44:26
tags:	
	- 小程序
categories:
	- 随记
---

> 循例交代背景：在使用 `canvas` 编辑图片，使用添加弹框来自定义添加文字，在微信开发工具上是正常弹窗不会被挡到，到手机上测试，发现弹框一直会在 `canvas` 组件后面

在官方文档中可以看到 [原生组件说明](https://developers.weixin.qq.com/miniprogram/dev/component/native-component.html)

```
1. 原生组件的层级是最高的，所以页面中的其他组件无论设置 z-index 为多少，都无法盖在原生组件上
2. 原生组件还无法在 picker-view 中使用。
3. 部分CSS样式无法应用于原生组件
4. 原生组件的事件监听不能使用 bind:eventname 的写法，只支持 bindeventname。原生组件也不支持 catch 和 capture 的事件绑定方式。
5. 原生组件会遮挡 vConsole 弹出的调试面板。 在工具上，原生组件是用web组件模拟的，因此很多情况并不能很好的还原真机的表现，建议开发者在使用到原生组件时尽量在真机上进行调试。
```

这就知道了为什么弹框会一直被挡住了，在小程序里面，原生组件的层级最高。

虽然提供了 `cover-view` 和 `cover-image` 组件，可以覆盖在原生组件上。但是不符合我的业务逻辑，我就没有用，在 `google` 之后，很多方式都是首先 `canvas`  组件转换为临时图片，然后用 `<image></image>` 将图片显示出来，但是这样无法继续编辑 `canvas` 。

所以我想了用将 `canvas` 和 `imgae` 一起用，代码如下：

```
## .wxml

<view>
    <canvas-drag class="{{radarImg ? 'gun' : ''}}" id="canvas-drag" graph="{{graph}}" width="750" height="1030">
    </canvas-drag>
    <image class="{{!radarImg ? 'gun' : ''}} radarImg" src='{{radarImg}}'></image>

    <view class="btn" bindtap="showModal">添加文字</view>
</view>
```

> 一开始不需要 `image` 组件，所以我们让他 `.gun`  到屏幕外面去了。同样，需要用到 `image`  的时候，我们会让 `canvas` 也 `.gun`  出屏幕外去

```
## .wxss

.radarImg {
    width: 750rpx;
    height: 1030rpx;
}

.gun {
    position: absolute;
    left: -999px;
    top: -0px;
}
```

> 在点击 `添加文字`  的时候，然后保存成临时文件，放到 image 组件里去，canvas 组件隐藏，弹出弹框

> 点击确定，canvas 组件重新回来

这样可以重新编辑，也可以弹框



*主要是提供一个思路，有什么不对也欢迎大家向我反馈，请多指教*