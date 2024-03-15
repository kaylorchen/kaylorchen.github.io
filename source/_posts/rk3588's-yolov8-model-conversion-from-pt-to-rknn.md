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
docker pull kaylor/rk3588_pt2onnx
git clone https://github.com/airockchip/ultralytics_yolov8.git
cd ultralytics_yolov8
git checkout 5b7ddd8f821c8f6edb389aa30cfbc88bd903867b
```
Download the newest model files from [Yolov8 github repository](https://github.com/ultralytics/ultralytics).  
For example, I download the model named **yolov8n.pt**
```bash
wget https://github.com/ultralytics/assets/releases/download/v8.1.0/yolov8n.pt
```
Edit ./ultralytics/cfg/default.yaml, replace "yolov8m-seg.pt" with "yolov8n.pt"
```yaml
model: yolov8n.pt # (str, optional) path to model file, i.e. yolov8n.pt, yolov8n.yaml
```
Convert pt to onnx...
```bash
host # docker run -it -v ${PWD}:/root/ws kaylor/rk3588_pt2onnx bash
----------------------------------
docker # cd /root/ws
docker # export PYTHONPATH=./ 
docker # python ./ultralytics/engine/exporter.py
docker # exit
```

# onnx to rknn
 
 Clone rk3588-convert-to-rknn repository and pull docker
 ```bash
 cd ../
 docker pull kaylor/rk3588_onnx2rknn
 git clone https://github.com/kaylorchen/rk3588-convert-to-rknn.git
 cp ultralytics_yolov8/yolov8n.onnx rk3588-convert-to-rknn
 cd rk3588-convert-to-rknn
 ```
Convert onnx to rknn
```bash
host # docker run -it -v ${PWD}:root/ws kaylor/rk3588_onnx2rknn bash
----------------------------------
docker # cd /root/ws
docker # python convert.py yolov8n.onnx rk3588 i8 yolov8n.rknn
docker # exit
```

Enjoy ~~







