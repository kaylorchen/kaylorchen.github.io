---
title: How to Test SPIFFS
comments: true
date: 2019-05-02 10:41:15
tags:
- ESP32
- Arduino
- IoT
categories:
- IoT
---
# Requirements
- Upload files to SPIFFS independently.
- Read files and display its contents in the monitor.

# Upload Files
Create a new **data** directory in the project root directory, and generate a txt file named "**kaylor.txt**". This file's contents are written as follows;``I am interested in you``.
![](/image/kaylor.txt.jpg)

***Terminal -> Run task -> PlatformIO: Upload file system image***, you will see logs as follows:
```
Processing esp32dev (platform: espressif32; board: esp32dev; framework: arduino)
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Verbose mode can be enabled via `-v, --verbose` option
CONFIGURATION: https://docs.platformio.org/page/boards/espressif32/esp32dev.html
PLATFORM: Espressif 32 > Espressif ESP32 Dev Module
HARDWARE: ESP32 240MHz 320KB RAM (4MB Flash)
DEBUG: CURRENT(esp-prog) EXTERNAL(esp-prog, iot-bus-jtag, jlink, minimodule, olimex-arm-usb-ocd, olimex-arm-usb-ocd-h, olimex-arm-usb-tiny-h, olimex-jtag-tiny, tumpa)
Converting FS.ino
Library Dependency Finder -> http://bit.ly/configure-pio-ldf
LDF MODES: FINDER(chain) COMPATIBILITY(soft)
Collected 26 compatible libraries
Scanning dependencies...
Dependency Graph
|-- <SPIFFS> 1.0
|   |-- <FS> 1.0
|-- <FS> 1.0
Building SPIFFS image from 'data' directory to .pioenvs\esp32dev\spiffs.bin
/kaylor.txt
Looking for upload port...
Auto-detected: COM13
Uploading .pioenvs\esp32dev\spiffs.bin
esptool.py v2.6
Serial port COM13
Connecting........__
Chip is ESP32D0WDQ6 (revision 1)
Features: WiFi, BT, Dual Core, 240MHz, VRef calibration in efuse, Coding Scheme None
MAC: cc:50:e3:a9:51:a4
Uploading stub...
Running stub...
Stub running...
Changing baud rate to 921600
Changed.
Configuring flash size...
Auto-detected Flash size: 4MB
Compressed 1503232 bytes to 2863...
Wrote 1503232 bytes (2863 compressed) at 0x00291000 in 0.1 seconds (effective 210979.3 kbit/s)...
Hash of data verified.
```
# Software 
```C++
#include "SPIFFS.h"
#include "FS.h"
void listDir(fs::FS &fs, const char * dirname, uint8_t levels){
    Serial.printf("Listing directory: %s\r\n", dirname);

    File root = fs.open(dirname);
    if(!root){
        Serial.println("- failed to open directory");
        return;
    }
    if(!root.isDirectory()){
        Serial.println(" - not a directory");
        return;
    }

    File file = root.openNextFile();
    while(file){
        if(file.isDirectory()){
            Serial.print("  DIR : ");
            Serial.println(file.name());
            if(levels){
                listDir(fs, file.name(), levels -1);
            }
        } else {
            Serial.print("  FILE: ");
            Serial.print(file.name());
            Serial.print("\tSIZE: ");
            Serial.println(file.size());
        }
        file = root.openNextFile();
    }
}

void setup() {
  // put your setup code here, to run once:
    Serial.begin(115200); 
  if(!SPIFFS.begin(true)){
    Serial.println("An Error has occurred while mounting SPIFFS");
    return;
  }
  listDir(SPIFFS, "/", 0); 
  File file = SPIFFS.open("/kaylor.txt");
  if(!file){
    Serial.println("Failed to open file for reading");
    return;
  } 
  Serial.println("File Content:");
  while(file.available()){
    Serial.write(file.read());
  }
  file.close();
}

void loop() {
  // put your main code here, to run repeatedly:

}
```
- Result:
![](/image/SPIFFS.jpg)

