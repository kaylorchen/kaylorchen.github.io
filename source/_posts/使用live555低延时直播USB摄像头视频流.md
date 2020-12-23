---
title: 使用live555低延时直播USB摄像头视频流
comments: true
date: 2020-05-19 12:14:13
tags:
- Network 
- RTSP
- live555
categories:
- Network
- RTSP
---
# 背景
分析了live555的源码，其中的样例多数时从文件读取H264文件进行直播，没有直接从摄像头读取视频流直接编码实现的样例。根据自己需求，按照live555的样例实现了几个类。
# 具体实现
源码从我的github直接获取：[github](https://github.com/kaylorchen/live555_rtsp)
客户端接收可以使用VLC，也可以使用我的另外一个[代码](https://github.com/kaylorchen/live555_decode_rtsp)（有bug，显示没有问题）.
VLC需要设置缓存，导致了较大的延时，我的代码只是直接获取每一帧直接解码，这样的实时性根据我的测试，在局域网的640*480的分辨率下，可以到达140ms的延时，平均带宽2Mbit/s.