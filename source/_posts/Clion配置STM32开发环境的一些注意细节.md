---
title: Clion配置STM32开发环境的一些注意细节
comments: true
date: 2021-07-28 10:41:15
tags:
- STM32
- CLion
categories:
- Embedded
---
# 使用newlib库
cmake里面加
```
add_compile_options(--specs=nano.specs)
add_link_options(--specs=nosys.specs --specs=nano.specs -Wl,--start-group -lc -lm -Wl,--end-group)
```

# 启用硬浮点编译
```
#Uncomment for hardware floating point
add_compile_definitions(ARM_MATH_CM4;ARM_MATH_MATRIX_CHECK;ARM_MATH_ROUNDING)
add_compile_options(-mfloat-abi=hard -mfpu=fpv4-sp-d16)
add_link_options(-mfloat-abi=hard -mfpu=fpv4-sp-d16)

#Uncomment for software floating point
#add_compile_options(-mfloat-abi=soft)
```


