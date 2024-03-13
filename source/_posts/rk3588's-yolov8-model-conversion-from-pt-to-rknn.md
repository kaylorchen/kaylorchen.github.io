---
title: rk3588's yolov8 model conversion from pt to rknn 
date: 2024-02-09 11:45:41
tags:
- rk3588
- yolo
categories:
- rk3588
comments: true
---

# Background

Yolov8's original model includes post-processing. Some shapes exceed the matrix calculation limit of 3588, so some cropping of the output layer is required.

# pt to onnx

Clone ultralytics_yolov8 repository and pull docker
```bash
git clone https://github.com/airockchip/ultralytics_yolov8.git
cd ultralytics_yolov8
git checkout 5b7ddd8f821c8f6edb389aa30cfbc88bd903867b
docker pull kaylor/rk3588_pt2onnx
```





