---
layout: post
title:  "Sending MQTT Messages between Arduino Wifi Shield and Ubuntu"
date:   2017-05-18 10:17:57 +0800
excerpt_separator: <!-- excerpt -->
---

In this tutorial, we will send MQTT messages between Arduino Wifi Shield and Ubuntu PC. Then we will verify the receiving messages from both Arduino and Ubuntu PC.
<!-- excerpt -->

### Equipment
1. 1 x Arduino Uno
2. 1 x Wifi shield from Arduino
3. 1 x Router
4. 1 x PC with Ubuntu installed


### MQTT Library for Arduino
You can download MQTT library from [here](https://github.com/knolleary/pubsubclient){:target="_blank"}. Actually, we only need 2 files from [here](https://github.com/knolleary/pubsubclient/tree/master/src){:target="_blank"}. You can either save it the Arduino libraries or you can just save it in your Arduino folder that you are going to start writing.

### Install Mosquitto MQTT Messaging Broker

Now, we will install MQTT at Ubuntu PC.

```sh
$ sudo apt-get install mosquitto mosquitto-clients
```

Once the installation is done, you can start testing MQTT alone

First, we start the subscriber

```sh
$ mosquitto_sub -h localhost -t test
```
At another terminal, type in the following line to publish a message to the subscriber.

```sh
$ mosquitto_pub -h localhost -t test -m "hello world"
```

You should see the "hello world" message at the subscriber terminal.

### Setup Arduino Wifi Shield 

First, make sure Wifi Shield jumper connection is correct. You can check the connection from [here](https://www.arduino.cc/en/Guide/ArduinoWiFiShield){:target="_blank"}. Once you had verified the connection, then you should test your wifi shield with the simple program from [here](https://www.arduino.cc/en/Guide/ArduinoWiFiShield#toc5){:target="_blank"}. Make sure your Arduino can scan your router network. Even the simple program is not working, you should consider updating your wifi shield from [here](https://www.arduino.cc/en/Hacking/WiFiShieldFirmwareUpgrading){:target="_blank"}.

### Program Arduino

```c
#include <SPI.h>
#include <WiFi.h>
#include "PubSubClient.h"

char ssid[] = "Elune";     //  your network SSID (name) 
char pass[] = "MoonWell";    // your network password
int status = WL_IDLE_STATUS;     // the Wifi radio's status

IPAddress server(192, 168, 0, 101); //MQTT server IP Address 
WiFiClient wifiClient;
PubSubClient client(wifiClient);

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i=0;i<length;i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();
  client.publish("fromArduino","Greeting from Arduino");
}

void reconnect() {
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Attempt to connect
    if (client.connect("arduinoClient")) {
      Serial.println("connected");
      // Once connected, publish an announcement...
      client.publish("fromArduino","Greeting from Arduino");
      // ... and resubscribe
      client.subscribe("fromUbuntu");
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(5000);
    }
  }
}

void setup()
{

Serial.println("Attempting to connect to WPA network...");
  status = WiFi.begin(ssid, pass);

  // if you're not connected, stop here:
  if ( status != WL_CONNECTED) { 
    Serial.println("Couldn't get a wifi connection");
    while(true);
  } 
  // if you are connected, print out info about the connection:
  else {
    Serial.println("Connected to network");
  }

  Serial.begin(57600);

  client.setServer(server, 1883);
  client.setCallback(callback);

  // Allow the hardware to sort itself out
  delay(1500);
}

void loop()
{
  if (!client.connected()) {
    reconnect();
  }
  client.loop();
}

```
The above program is the complete program to get it working. First, The Arduino board will connect to a router. Once the connection is OK, It will try to make MQTT connection. After the MQTT connection is established, the Arduino board will publish the topic (the publisher topic name is called `fromArduino`) and it will listen from the topic (the subscriber topic is called `fromUbuntu`). Once the board receives a message from `fromUbuntu` topic, it will publish a message to the topic `fromArduino`.

These are the parameters you should change in the Arduino based on your network
```
char ssid[] = "Elune";     //  your network SSID (name) 
char pass[] = "MoonWell";    // your network password
IPAddress server(192, 168, 0, 101); //MQTT server IP Address 
```
> Note: 192.168.0.101 is Ubuntu PC IP Address for my case.

### Verification MQTT messages 
At your Ubuntu system, type in the following line to subscribe the topic from the Arduino board.  

```
$ mosquitto_sub -h localhost -t fromArduino
```

At your Ubuntu system again but at the different terminal, type in the following line to publish topic to the Arduino board.
```
$ mosquitto_pub -h localhost -t fromUbuntu -m "sending from Ubuntu"
```
You should receive the message back from Arduino like this on your Ubuntu system at the subscriber terminal.
```
Greeting from Arduino
```