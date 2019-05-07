---
title: Connect to your Wifi with ESP32
comments: true
date: 2019-05-02 10:30:59
tags:
- ESP32
- Arduino
- IoT
categories:
- IoT
---
# Requirements
- Connect to a Wifi as a station

# Software
```C++
#include <WiFi.h>

const char* ssid     = "Xiaomi_kaylordut";
const char* password = "kaylordut.com";

void setup()
{
    Serial.begin(115200);
    delay(10);

    // We start by connecting to a WiFi network

    Serial.println();
    Serial.println();
    Serial.print("Connecting to ");
    Serial.println(ssid);

    WiFi.begin(ssid, password);

    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }

    Serial.println("");
    Serial.println("WiFi connected");
    Serial.println("IP address: ");
    Serial.println(WiFi.localIP());
}

void loop()
{
}
```
- Result:
![](/image/Station.jpg)
