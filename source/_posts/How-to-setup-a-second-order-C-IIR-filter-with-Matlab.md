---
title: How to setup a second order C++ IIR filter with Matlab
comments: true
date: 2019-08-08 17:56:22
tags: 
- Matlab
- Filter
- C++
categories:
- Robot
---

# Generating parameters with Matlab
Open your matlab and type ***fdatool***, and click "***Analysis->Filter Coefficients***".
![](/image/Filter.png)
Click "***Design Filter***" and get filter parameters.
Numerator: [ 1 2 1 ]
Denominator: [ 1 -1.8153410827045682   0.83100558934675761 ]
Gain: 0.0039161266605473692

# Test your IIR filter
## git IIR Code
```bash
git clone https://github.com/kaylorchen/IIR_Filter.git
```

## Initialize your filter:
```C++
IIR_Filter lowpass(1, 2, 1, -1.8153410827045682, 0.83100558934675761);  // Fs = 48000, Fc=1000
```
