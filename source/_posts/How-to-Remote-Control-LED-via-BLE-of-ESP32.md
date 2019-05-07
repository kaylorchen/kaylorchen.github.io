---
title: How to Remote Control LED via BLE of ESP32
comments: true
date: 2019-05-07 16:15:39
tags:
- ESP32
- Arduino
- IoT
categories:
- IoT
---
# Requirements
- Turn on/off LED via Bluetooth

# Software
```C++
#include <BLEDevice.h>
#include <BLEServer.h>
#include <BLEUtils.h>
#include <BLE2902.h>
 
BLECharacteristic *pCharacteristic;
bool deviceConnected = false;
float txValue = 0;
const int LED = 2; // Could be different depending on the dev board. I used the DOIT ESP32 dev board.
 
#define SERVICE_UUID           "6E400001-B5A3-F393-E0A9-E50E24DCCA9E" // UART service UUID
#define CHARACTERISTIC_UUID_RX "6E400002-B5A3-F393-E0A9-E50E24DCCA9E"
#define CHARACTERISTIC_UUID_TX "6E400003-B5A3-F393-E0A9-E50E24DCCA9E"
 
class MyServerCallbacks: public BLEServerCallbacks {
    void onConnect(BLEServer* pServer) {
      deviceConnected = true;
    };
 
    void onDisconnect(BLEServer* pServer) {
      deviceConnected = false;
    }
};
 
class MyCallbacks: public BLECharacteristicCallbacks {
    void onWrite(BLECharacteristic *pCharacteristic) {
      std::string rxValue = pCharacteristic->getValue();
 
      if (rxValue.length() > 0) {
        Serial.println("*********");
        Serial.print("Received Value: ");
 
        for (int i = 0; i < rxValue.length(); i++) {
          Serial.print(rxValue[i]);
        }
        Serial.println();
        // Do stuff based on the command received from the app
        if (rxValue == "ON" || rxValue == "On" || rxValue == "on") { 
          Serial.print("Turning ON!");
          digitalWrite(LED, HIGH);
        }
        else if (rxValue == "OFF" || rxValue == "Off" || rxValue == "off") {
          Serial.print("Turning OFF!");
          digitalWrite(LED, LOW);
        }
        Serial.println();
        Serial.println("*********");
      }
    }
};
void setup() {
  Serial.begin(115200);
  pinMode(LED, OUTPUT);
  // Create the BLE Device
  BLEDevice::init("ESP32_BLE_LED"); // Give it a name
  // Create the BLE Server
  BLEServer *pServer = BLEDevice::createServer();
  pServer->setCallbacks(new MyServerCallbacks());
 
  // Create the BLE Service
  BLEService *pService = pServer->createService(SERVICE_UUID);
 
  // Create a BLE Characteristic
  pCharacteristic = pService->createCharacteristic(
                      CHARACTERISTIC_UUID_TX,
                      BLECharacteristic::PROPERTY_NOTIFY
                    );
                      
  pCharacteristic->addDescriptor(new BLE2902());
 
  BLECharacteristic *pCharacteristic = pService->createCharacteristic(
                                         CHARACTERISTIC_UUID_RX,
                                         BLECharacteristic::PROPERTY_WRITE
                                       );
 
  pCharacteristic->setCallbacks(new MyCallbacks());
 
  // Start the service
  pService->start();
 
  // Start advertising
  pServer->getAdvertising()->start();
  Serial.println("Waiting a client connection to notify...");
}
 
void loop() {
  if (deviceConnected) {

    pCharacteristic->setValue("Hello, this is a led test.");
    
    pCharacteristic->notify(); // Send the value to the app!
  }
  delay(1000);
}
```
# Test
- Download **nRF Connect** from Goggle Paly ,Apple Store or [github](https://github.com/NordicSemiconductor/Android-nRF-Connect/releases).
- Open the App and scan Bluetooth devices.
- Connect to "ESP32_BLE_LED" and send "on" or "off" to ESP.
<iframe width="560" height="315" src="https://www.youtube.com/embed/s9KjcactlzI" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
If you can't open the above video, please click this [link](https://v.youku.com/v_show/id_XNDE3MjAzMDAzNg==.html?spm=a2hzp.8244740.0.0).