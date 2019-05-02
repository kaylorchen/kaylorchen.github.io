---
title: Configure your Wifi Information via WebServer
comments: true
date: 2019-05-02 11:09:47
tags:
- ESP32
- Arduino
- IoT
categories:
- IoT
---

# Requirments
- Start a WebServer with ESP32
- Control LED by browser
- Input your Wifi information by browser and enable **connect**

# SPIFFS files
You can download some web files via https://github.com/kaylorchen/ESP32/tree/master/WebConfig/data, and save them to your ***data***
directory.
Upload these files to your image: ***Terminal -> Run task -> PlatformIO: Upload file system image***

# Software
```
#include <WiFi.h>
#include <WiFiClient.h>
#include <WebServer.h>
#include <ESPmDNS.h>
#include "SPIFFS.h"
#include "FS.h"
bool loadFromSpiffs(String path);

char ssid[50] = "";
char password[50] = "";

WebServer server(80);
const int led = 2;

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

void handleRoot() {
  server.sendHeader("Location", "/index.html",true);   //Redirect to our html web page
  server.send(302, "text/plane","");
}

void handleLED(){
    if(server.hasArg("value")){
      String action = server.arg("value");
      if( action == "ON"){
        digitalWrite(led, 1);
        server.send(200, "text/html", "ON");
        return; 
        }else if( action == "OFF"){   
        digitalWrite(led, 0);
        server.send(200, "text/html", "OFF");
        return;  
        }
      }
}

void handleWifi(){
  if(server.hasArg("config")){
      String config = server.arg("config");
      String wifiname;
      String wifipwd;
      if(config == "on"){
        if(server.hasArg("name")){
          wifiname = server.arg("name");
          }
        if(server.hasArg("pwd")){
          wifipwd = server.arg("pwd");
          }
          Serial.println("ssid: "+ wifiname +"\r\npassword: " + wifipwd);
          wifiname.toCharArray(ssid, 50);
          wifipwd.toCharArray(password, 50);
          WiFi.mode(WIFI_STA);
          if(WiFi.status() == WL_CONNECTED){
            Serial.println("disconnect the original Wifi");
            WiFi.disconnect();
            }
          Serial.println("handle Wifi end");
        }
    }  
}

void handleWebRequests(){
  if(loadFromSpiffs(server.uri())) return;
  String message = "File Not Detected\n\n";
  message += "URI: ";
  message += server.uri();
  message += "\nMethod: ";
  message += (server.method() == HTTP_GET)?"GET":"POST";
  message += "\nArguments: ";
  message += server.args();
  message += "\n";
  for (uint8_t i=0; i<server.args(); i++){
    message += " NAME:"+server.argName(i) + "\n VALUE:" + server.arg(i) + "\n";
  }
  server.send(404, "text/plain", message);
  Serial.println(message);
}

void softAPConfig(){
  Serial.println("Configuring access point...");

  // You can remove the password parameter if you want the AP to be open.
  uint32_t chipID = ESP.getEfuseMac();
  String apName = "ESP32_" + String(chipID, HEX);
  Serial.println("AP name: " + apName );
  WiFi.mode(WIFI_AP);
  WiFi.softAP(apName.c_str());
  IPAddress myIP = WiFi.softAPIP();
  Serial.print("AP IP address: ");
  Serial.println(myIP);  
}

void wifiConfig(){
    Serial.print("Connecting to ");
    Serial.println(ssid);
    Serial.print("Password: ");
    Serial.println(password);
    WiFi.mode(WIFI_STA);
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



void setup() {
  // put your setup code here, to run once:
  pinMode(led, OUTPUT);
  digitalWrite(led, 0);
  Serial.begin(115200);
  delay(3000);

  if(!SPIFFS.begin(true)){
    Serial.println("An Error has occurred while mounting SPIFFS");
    return;
  }
  listDir(SPIFFS, "/", 0);
  softAPConfig();

  

  server.on("/", handleRoot);
  server.on("/led", HTTP_GET, handleLED);
  server.on("/wifi", HTTP_GET, handleWifi);
  server.onNotFound(handleWebRequests);
  server.begin();
  Serial.println("HTTP server started");

}

void loop() {
  // put your main code here, to run repeatedly:
  server.handleClient();
  if(WiFi.status() != WL_CONNECTED && WiFi.getMode() == WIFI_STA){
      wifiConfig();
    }
}

bool loadFromSpiffs(String path){
  String dataType = "text/plain";
  if(path.endsWith("/")) path += "index.htm";

  if(path.endsWith(".src")) path = path.substring(0, path.lastIndexOf("."));
  else if(path.endsWith(".html")) dataType = "text/html";
  else if(path.endsWith(".htm")) dataType = "text/html";
  else if(path.endsWith(".css")) dataType = "text/css";
  else if(path.endsWith(".js")) dataType = "application/javascript";
  else if(path.endsWith(".png")) dataType = "image/png";
  else if(path.endsWith(".gif")) dataType = "image/gif";
  else if(path.endsWith(".jpg")) dataType = "image/jpeg";
  else if(path.endsWith(".ico")) dataType = "image/x-icon";
  else if(path.endsWith(".xml")) dataType = "text/xml";
  else if(path.endsWith(".pdf")) dataType = "application/pdf";
  else if(path.endsWith(".zip")) dataType = "application/zip";
  File dataFile = SPIFFS.open(path.c_str(), "r");
  if (server.hasArg("download")) dataType = "application/octet-stream";
  if (server.streamFile(dataFile, dataType) != dataFile.size()) {
  }
  Serial.println(path);
  dataFile.close();
  return true;
}
```
- Result:

![](/image/Web.jpg)

<iframe width="560" height="315" src="https://www.youtube.com/embed/FXrBUMKSWI4" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

If you can't open Youtube, and please open this [link](https://v.youku.com/v_show/id_XNDE2MzAwNzU2OA==.html?spm=a2hzp.8244740.0.0).
