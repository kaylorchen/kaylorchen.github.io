---
title: How to control GPIO of ESP32
comments: true
date: 2019-05-01 16:05:00
tags:
- ESP32
- Arduino
- IoT
categories:
- IoT
---
# Hardware
![](/image/ESP32.jpg)
![](/image/GPIO.jpg)
# Software
```C++
#include "Arduino.h"
// constants won't change. Used here to set a pin number:
const int ledPin =  2;// the number of the LED pin
const int buttonPin = 0;
uint8_t value = 0;

void setup() {
  // set the digital pin as output/input:
  pinMode(ledPin, OUTPUT);
  pinMode(buttonPin, INPUT);
  Serial.begin(115200);
}

void loop() {
  value = digitalRead(buttonPin);
  Serial.printf("Button = %d\n\r",value);
  if (value == 0) {
    // set the LED with 1 when button is pressed:
    digitalWrite(ledPin, 1);
  }
  else{
    digitalWrite(ledPin, 0);
  } 
}
```
When the button is pressed, LED will be ON.
