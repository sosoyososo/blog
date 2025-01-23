---
title: "对比两个使用ARKit追踪眼睛注视点的方案"
date: 2024-11-28T15:10:12+08:00
draft: true
---

## 引用来源

[https://shiru99.medium.com/eye-tracking-with-arkit-ios-part-ii-2723f9bfe04e](https://shiru99.medium.com/eye-tracking-with-arkit-ios-part-ii-2723f9bfe04e)

[https://github.com/Shiru99/AR-Eye-Tracker](https://github.com/Shiru99/AR-Eye-Tracker)

[https://github.com/virakri/eye-tracking-ios-prototype](https://github.com/virakri/eye-tracking-ios-prototype)

## ARKit运行简单介绍

使用ARKit追踪眼睛注视点，主要是使用  ARFaceTrackingConfiguration 、ARSession、 ARSCNView，主要的逻辑是ARSession提供了运行ARKit的环境，ARFaceTrackingConfiguration 告诉ARSession想要识别人脸，我们可以监听ARSession回调函数来获取人脸识别的结果，比如脸部位置，眼睛位置，眼睛注视点。而ARSCNView本身携带了ARSession，他继承了[SceneKit](https://developer.apple.com/documentation/scenekit?language=objc)的SCNView 用来展示相机拍摄到的内容和ARKit识别到的结果。

ARSession的回调会告诉我们识别到脸部的位置，眼睛的位置，注视点的位置，那么我们需要做的就是在手机屏幕上显示这个注视点，逻辑上很容易理解。那么难点在哪里？

<!--more--> 

## 理解难点

理解的难点在于坐标变换和单位，首先你需要理解世界坐标空间、相机空间坐标和屏幕空间坐标。在回调函数中，你可以得到所有关键点的信息是通过4x4矩阵来表达的，这个矩阵包含了位置信息、旋转信息和缩放信息。但你不知道他是在哪个空间里表达的，同样你不知道他的单位是什么。

## 解决问题的方案对比

博客文章和[AR-Eye-Tracker](https://github.com/Shiru99/AR-Eye-Tracker)的方案是对注视点转换成世界坐标空间之后再转换为屏幕空间坐标，之后绘制到屏幕上，算是直球来解决问题。但我尝试了一下，看起来并不是很稳定。有人在issue中问代码里的魔术数字代表什么，怎么算的，他推荐了e[ye-tracking-ios-prototype](https://github.com/virakri/eye-tracking-ios-prototype)。

e[ye-tracking-ios-prototype](https://github.com/virakri/eye-tracking-ios-prototype) 的方案就很取巧，这也是我真心想推荐的聪明方案，他没有直接直接手动进行空间转换计算，而是巧妙的使用SceneKit，让系统SDK帮忙计算。他的方案是核心逻辑是在手机平面上增加一个node模拟用来显示注视点的平面，使用两只眼睛的位置和两个注视点的位置连成两个线段，使用hitTestWithSegment来获取两个线段跟平面的两个交叉点，取中间点作为真正注视点。

## 额外的细节点

两个代码里都有魔术数字0.0623908297，0.135096943231532。两个github项目都有相关讨论，[https://github.com/virakri/eye-tracking-ios-prototype/issues/3 ](https://github.com/virakri/eye-tracking-ios-prototype/issues/3)这里讨论相对更加细节一些。

简单说，这两个数字是以米为单位的屏幕尺寸。但不是从apple官网获取到的，而是通过像素数配合像素数量进行计算，先获得的单位是英寸，再以25.4mm/inch 来换算米.