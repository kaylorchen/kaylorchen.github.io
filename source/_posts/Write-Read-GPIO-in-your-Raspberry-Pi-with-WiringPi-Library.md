---
title: Write/Read GPIO in your Raspberry Pi with WiringPi Library
date: 2019-04-30 12:14:55
tags:
- Raspberry Pi
categories:
- Raspberry Pi
---
# Edit GPIO Permission
- Create a rule file: ***/lib/udev/rules.d/60-gpio.rules***
```
KERNEL=="spidev*", GROUP="spi", MODE="0660"
SUBSYSTEM=="gpio*", PROGRAM="/bin/sh -c 'chown -R root:gpio /sys/class/gpio && chmod -R 770 /sys/class/gpio; chown -R root:gpio /sys/devices/virtual/gpio && chmod -R 770 /sys/devices/virtual/gpio; chown -R root:gpio /sys/devices/platform/soc/*.gpio/gpio && chmod -R 770 /sys/devices/platform/soc/*.gpio/gpio'"
```
- Add root user to GPIO group
```
groupadd gpio
usermod -a -G gpio root
reboot
```
# Edit C++ code:
```
#include <iostream>
#include "wiringPi.h"

void loop(uint16_t cnt, int ctlPin, int sigPin)
{
    pinMode(ctlPin, OUTPUT);
    pullUpDnControl(ctlPin, PUD_DOWN);
    pinMode(sigPin, INPUT);

    while (cnt --)
    {
        digitalWrite(ctlPin, 1);
//        delay(5000);
        uint8_t sig = digitalRead(sigPin);
        while(sig)
        {
            printf("sig state: %d\n",sig);
            sig = digitalRead(sigPin);
        }
        printf("___sig state: %d\n",sig);
        delay(10);
        while(!sig)
        {
            printf("sig state: %d\n",sig);
            sig = digitalRead(sigPin);
        }
        printf("###sig state: %d\n",sig);
        digitalWrite(ctlPin, 0);
    }
}

int main(){
    if (wiringPiSetup() < 0) return  1;
    loop(1, 24, 25);
    return 0;
}
```

# Edit CMakelists.txt
```
cmake_minimum_required(VERSION 3.10)
project(test)

set(CMAKE_CXX_STANDARD 11)
find_library(wiringPi_LIB wiringPi)
find_package(Threads)

add_executable(test main.cpp)
target_link_libraries(test ${wiringPi_LIB} ${CMAKE_THREAD_LIBS_INIT} rt crypt)
```

# Test your program
```
mkdir build
cd build
make
sudo ./test
```