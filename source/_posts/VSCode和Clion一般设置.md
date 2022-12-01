---
title: VSCode和Clion一般设置
comments: true
date: 2022-10-28 10:41:15
tags:
- VScode
categories:
- IDE
---

# 格式化代码
- VSCode
  - On Windows
      shift + alt + f //格式化整个文件  
      ctrl+k ctrl+f //格式化选中代码
  - On Linux
      ctrl + shift + i
  - On MacOs
      shift + option + i //格式化整个文件  
      cmd+k cmd+f //格式化选中代码

- Clion

    ctrl + alt + L

# 选择列编辑
- Vscode

alt + shift + 鼠标选择

- Clion

alt + 鼠标选择

# VSCode头文件路径配置
按下 ctrl + shift + P, 调用C/C++配置json文件，会在工程窗口生成 **_c_cpp_properties.json_** , 可以使用原生的UI配置。

# Doxygen函数注释
- VSCode

/** + 回车

- Clion

/*! + 回车 或者 /// + 回车

