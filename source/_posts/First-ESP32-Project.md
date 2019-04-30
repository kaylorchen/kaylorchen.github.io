---
title: First ESP32 Project
comments: true
date: 2019-04-30 19:53:30
tags:
- ESP32
- Arduino
- IoT
categories:
- IoT
---
# Initialize the Project
Open Visual Studio Code, **New Project**:
![](/image/PlatformIONewProject.png)
![](/image/FirstProject.png)

Append a new line in ***platformio.ini***, the final file follow as:
```
[env:esp32dev]
platform = espressif32
board = esp32dev
framework = arduino
monitor_speed = 115200
```
# Program Application Code
Edit **src/main.cpp**.  
```
#include "Arduino.h"
uint64_t chipid;  
void setup(){
    Serial.begin(115200);
}
void loop(){
    chipid=ESP.getEfuseMac();//The chip ID is essentially its MAC address(length: 6 bytes).
    Serial.printf("ESP32 Chip ID = %04X",(uint16_t)(chipid>>32));//print High 2 bytes
    Serial.printf("%08X\n",(uint32_t)chipid);//print Low 4bytes.
    delay(1500);
}
```
![](/image/FirstResult.png)
