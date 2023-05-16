---
title: 功率MOS知识点总结
comments: true
mathjax: true
date: 2022-08-07 10:42:15
tags:
- 电路
categories:
- 电路
---

# 功耗和电流

本文参考Nexperia的AN11158应用手册计算，链接在[这里](https://assets.nexperia.com/documents/application-note/AN11158.pdf)

## 估算不同温度的$I_{d}$  

$$P=I^{2} \times R_{DS_{on}}$$
器件在结温的时候的耗散功率，这里假定${R_{DS_{on}}}$ 恒定且是最高结温的时候的导通电阻。有

$$I_{d}^{2}(T_{mb})\propto \frac{T_{j}-T_{mb}}{T_{j}-25^\circ C}$$

$$I_{d}(T_{mb})= I_{d}(25^\circ C)\times \sqrt{\frac{T_{j}-T_{mb}}{T_{j}-25^\circ C}}$$  

## 耗散功率

## 某个温度的耗散功率

比如，根据数据手册$25^\circ C$的时候的最大耗散功率是105W，那么$75^\circ C$的耗散功率等于：

$$P_{tot}(75)=P_{tot}(25)\times \frac{T_{j}-75}{T_{j}-25}=105 \times \frac{175-75}{175-25}=70 W  $$

## 从温升计算最大功耗和电流

从数据手册中可以得到结到壳的热阻是 $1.3^\circ C/W$ , 壳温是25度，结温是175的时候，耗散功率是150/1.3=115.38W，这时候可以看到数据手册写的是115W。  
从数据手册中可以查到结温是175的时候，导通电阻是6.7 $m\Omega$。  

$$I_{max}=\sqrt{\frac{115.38W}{0.0067 \Omega } } = 131.23A$$

这里需要注意，虽然计算是131A，但是根据数据手册描述，其他限制了这个MOS的能力，所以降额到了120A。
