---
title: Install and Test MQTT Client C Library in Raspberry Pi
comments: true
date: 2019-05-04 20:57:58
tags:
- IoT
- Raspberry Pi
- MQTT
categories:
- Raspberry Pi
- IoT
---
# Install MQTT Client C Library
```
apt update
apt install cmake libssl-dev
git clone https://github.com/eclipse/paho.mqtt.c.git
cd paho.mqtt.c
git checkout v1.2.1
cmake -Bbuild -H. -DPAHO_WITH_SSL=ON
sudo cmake --build build/ --target install
sudo ldconfig
```
# Test Demo 
Edit **src/sample/CMakeLists.txt**:
```C
diff --git a/src/samples/CMakeLists.txt b/src/samples/CMakeLists.txt
index 79ea886..230599a 100644
--- a/src/samples/CMakeLists.txt
+++ b/src/samples/CMakeLists.txt
@@ -18,6 +18,7 @@
 # Note: on OS X you should install XCode and the associated command-line tools

 ## compilation/linkage settings
+cmake_minimum_required(VERSION 3.10)
 INCLUDE_DIRECTORIES(
     .
     ${CMAKE_SOURCE_DIR}/src
@@ -51,15 +52,15 @@ TARGET_LINK_LIBRARIES(MQTTClient_subscribe paho-mqtt3c)
 TARGET_LINK_LIBRARIES(MQTTClient_publish paho-mqtt3c)
 TARGET_LINK_LIBRARIES(MQTTClient_publish_async paho-mqtt3c)

-INSTALL(TARGETS paho_c_sub
-                paho_c_pub
-                paho_cs_sub
-                paho_cs_pub
-                MQTTAsync_subscribe
-                MQTTAsync_publish
-                MQTTClient_subscribe
-                MQTTClient_publish
-                MQTTClient_publish_async
-
-    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
-    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR})
+#INSTALL(TARGETS paho_c_sub
+#                paho_c_pub
+#                paho_cs_sub
+#                paho_cs_pub
+#                MQTTAsync_subscribe
+#                MQTTAsync_publish
+#                MQTTClient_subscribe
+#                MQTTClient_publish
+#                MQTTClient_publish_async
+#
+#    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
+#    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR})
```
Edit **MQTTClient_publish_async.c** and **MQTTClient_subscribe.c**:
```C
diff --git a/src/samples/MQTTClient_publish_async.c b/src/samples/MQTTClient_publish_async.c
index 7784349..57ef1ab 100644
--- a/src/samples/MQTTClient_publish_async.c
+++ b/src/samples/MQTTClient_publish_async.c
@@ -19,7 +19,7 @@
 #include <string.h>
 #include "MQTTClient.h"

-#define ADDRESS     "tcp://localhost:1883"
+#define ADDRESS     "tcp://example.com:11111" 
 #define CLIENTID    "ExampleClientPub"
 #define TOPIC       "MQTT Examples"
 #define PAYLOAD     "Hello World!"
@@ -72,6 +72,8 @@ int main(int argc, char* argv[])
         MQTTCLIENT_PERSISTENCE_NONE, NULL);
     conn_opts.keepAliveInterval = 20;
     conn_opts.cleansession = 1;
+    conn_opts.username = "username";
+    conn_opts.password = "passwd";

     MQTTClient_setCallbacks(client, NULL, connlost, msgarrvd, delivered);

```
```C
diff --git a/src/samples/MQTTClient_subscribe.c b/src/samples/MQTTClient_subscribe.c
index c675ecc..e99fd84 100644
--- a/src/samples/MQTTClient_subscribe.c
+++ b/src/samples/MQTTClient_subscribe.c
@@ -19,7 +19,7 @@
 #include <string.h>
 #include "MQTTClient.h"

-#define ADDRESS     "tcp://localhost:1883"
+#define ADDRESS     "tcp://example.com:11111"
 #define CLIENTID    "ExampleClientSub"
 #define TOPIC       "MQTT Examples"
 #define PAYLOAD     "Hello World!"
@@ -71,6 +71,8 @@ int main(int argc, char* argv[])
         MQTTCLIENT_PERSISTENCE_NONE, NULL);
     conn_opts.keepAliveInterval = 20;
     conn_opts.cleansession = 1;
+    conn_opts.username = "username";
+    conn_opts.password = "passwd";

     MQTTClient_setCallbacks(client, NULL, connlost, msgarrvd, delivered);
```
Type the below command:
```bash
mkdir build
cd build
cmake ..
make
.MQTTClient_subscribe
```
Open another terminal:``./MQTTClient_publish_async`` 

- Result:
![](/image/RPi_MQTT.jpg)

