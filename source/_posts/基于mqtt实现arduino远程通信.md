---
title: 基于MQTT实现Arduino远程通信
tags:
  - Arduino
  - MQTT
  - 物联网
id: '84'
categories:
  - - 物联网
abbrlink: 2ad6b2a
date: 2020-07-24 15:30:40
---

> 原载于CSDN 2019-04-19 
> 
> [https://blog.csdn.net/qq_44350275/article/details/89406527](https://blog.csdn.net/qq_44350275/article/details/89406527)
> 
> 现搬运到自己的Blog  
> 这里放上我的CSDN链接  
> [https://me.csdn.net/qq_44350275?ref=miniprofile](https://me.csdn.net/qq_44350275?ref=miniprofile)

我在做竞赛项目的时候，在板子和板子远程交互上做的东西  
百度上找到的东西也都是七零八碎的，  
同时也希望我的博客能给大家一点微薄的帮助

## 准备工作

1.  Arduino ide的准备安装。
2.  ESP8266模块+UNO 或者 NodeMCU和WIFIduino（初学者推荐后两个选择）
3.  搭建自己的MQTT服务器
4.  准备两个库 PubSubclient 和 esp8266wifi

## MQTT服务器的介绍

MQTT（Message Queuing Telemetry Transport，消息队列遥测传输协议），是一种基于发布/订阅（publish/subscribe）模式的"轻量级"通讯协议，该协议构建于TCP/IP协议上，由IBM在1999年发布。MQTT最大优点在于，可以以极少的代码和有限的带宽，为连接远程设备提供实时可靠的消息服务。作为一种低开销、低带宽占用的即时通讯协议，使其在物联网、小型设备、移动应用等方面有较广泛的应用。

MQTT是一个基于客户端-服务器的消息发布/订阅传输协议。MQTT协议是轻量、简单、开放和易于实现的，这些特点使它适用范围非常广泛。在很多情况下，包括受限的环境中，如：机器与机器（M2M）通信和物联网（IoT）。其在，通过卫星链路通信传感器、偶尔拨号的医疗设备、智能家居、及一些小型化设备中已广泛使用。

而对于Arduino来说——只需要简单的发布消息和接收消息——是再好不过的选择，而且有现成的库文件，也不需要我们再去做很多其他工作。

## MQTT服务器的搭建

我在Windows上搭建MQTT服务器的时候 一直遇到一个.dll文件缺失 所以我就转到Ubuntu上搭建服务器了。  
个人推荐用EMQ这个软件来搭（管理台界面好看而且有中文）  
自己电脑上搭服务器的缺点是，必须在同局域网下——即你的板子你的手机和你的电脑链接的是同一个WIFI——才能链接这个服务器，主要原因是你的电脑不具有公网IP没有办法在公网下访问，如果只是自己学东西建议是在自己的电脑上搭。  
如果是要出产品 建议去租阿里的服务器（我是白嫖了朋友一个）

## 两个库文件

PubSubclient 这个库文件应该是对MQTT支持比较完美的库文件  
请到该github链接上下载或者官网  
github下载链接: [link](https://github.com/knolleary/pubsubclient).  
官网下载链接: [link](https://www.arduinolibraries.info/libraries/pub-sub-client).  
这个库文件的详细说明请见该库文件的API文档  
API文档: [link](https://pubsubclient.knolleary.net/api.html).

![如图选中的示例](https://img-blog.csdnimg.cn/20190419200225754.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0MzUwMjc1,size_16,color_FFFFFF,t_70)

而我们真正用到的 只是其中的一个示例：  
ESP8266WIFI  
该库可在我的github上找到：  
下载链接: [link](https://github.com/Varsion/ESP8266WiFi).

接下来我会详细的解释这个例子，这个例子实现了最简单的发布信息和接受信息：

```
/*
基本的ESP8266 MQTT示例

 该草图演示了pubsub库的组合功能
 与ESP8266板/库。
 它连接到MQTT服务器，然后：
   - 每两秒向“outTopic”主题发布“hello world”
   - 订阅“inTopic”主题，打印出任何消息
    它接收。 NB  - 它假设收到的有效载荷是非二进制的字符串
   - 如果主题“inTopic”的第一个字符为1，则打开ESP Led，
    否则关掉它
 如果使用阻止丢失连接，它将重新连接到服务器
 重新连接功能。有关如何使用，请参阅'mqtt_reconnect_nonblocking'示例
 在不阻塞主循环的情况下实现相同的结果。

 要安装ESP8266板，（使用Arduino 1.6.4+）：
   - 在“文件 - >首选项 - >其他板卡管理器URL”下添加以下第三方板卡管理器：
      http://arduino.esp8266.com/stable/package_esp8266com_index.json
   - 打开“工具 - >板 - >板管理器”，然后单击ESP8266的安装“
   - 在“工具 - >板”中选择您的ESP8266
这个是我用翻译把顶部注释翻译了
*/

#include <ESP8266WiFi.h>
#include <PubSubClient.h>

const char* ssid = "........";//你要让板子链接的WiFi的名字
const char* password = "........";//该WiFi的密码
const char* mqtt_server = "broker.mqtt-dashboard.com";//你的服务器地址

WiFiClient espClient;
PubSubClient client(espClient);
long lastMsg = 0;
char msg[50];
int value = 0;

void setup_wifi() {

  delay(10);
   Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  randomSeed(micros());

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}//链接WiFi

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }//串口打印出收到的信息
  Serial.println();

  // 如果收到1作为第一个字符，则打开LED
  if ((char)payload[0] == '1') {
    digitalWrite(BUILTIN_LED, LOW);   // 这里他说接受的‘1’就打开灯 但是我在用的时候 接收到0才会打开  这一行的‘LOW’和下面的‘HIGH’应该换下位置，下面也说了 ESP-01是这样的
    // but actually the LED is on; this is because
    // it is active low on the ESP-01)
  } else {
    digitalWrite(BUILTIN_LED, HIGH);  // Turn the LED off by making the voltage HIGH
  }

}

void reconnect() {
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Create a random client ID
    String clientId = "ESP8266Client-";//该板子的链接名称
    clientId += String(random(0xffff), HEX);//产生一个随机数字 以免多块板子重名
    //尝试连接
    if (client.connect(clientId.c_str())) {
      Serial.println("connected");
      // 连接后，发布公告......
      client.publish("outTopic", "hello world");//链接成功后 会发布这个主题和语句
      // ......并订阅
      client.subscribe("inTopic");//这个是你让板子订阅的主题（接受该主题的消息）
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      // 如果链接失败 等待五分钟重新链接
      delay(5000);
    }
  }
}

void setup() {
  pinMode(BUILTIN_LED, OUTPUT);     // 将BUILTIN_LED引脚初始化为输出
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, 1883);//MQTT默认端口是1883
  client.setCallback(callback);
}

void loop() {

  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  long now = millis();
  if (now - lastMsg > 2000) {
    lastMsg = now;
    ++value;
    snprintf (msg, 50, "hello world #%ld", value);
    Serial.print("Publish message: ");
    Serial.println(msg);//串口打印，串口调试器可以看到的
    client.publish("outTopic", msg);//接收该主题消息
  }
}

```

## 手机调试软件——MyMQTT

![链接设置](https://img-blog.csdnimg.cn/20190419204641993.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0MzUwMjc1,size_16,color_FFFFFF,t_70)

![功能用法](https://img-blog.csdnimg.cn/20190419204651168.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0MzUwMjc1,size_16,color_FFFFFF,t_70)

该软件已经上架Google，没有条件翻墙的朋友 我把该软件上传了百度网盘 可以在该链接下下载  
链接: [link](https://pan.baidu.com/s/1KPXMB7nSl9yBXLF7gqVL1A).  
提取码：jdd8

### 下面贴上的是我自己将发布和订阅分开的代码（待完善）

###### 发布消息

```
/*
基本的ESP8266 MQTT示例

 该草图演示了pubsub库的组合功能
 与ESP8266板/库。
仅仅发布信息，以光敏电阻为例子
*/

#include <ESP8266WiFi.h>
#include <PubSubClient.h>

const char* ssid = "***";//wifi名称
const char* password = "***";//wifi密码
const char* mqtt_server = "***";//服务器地址
const char* out_topic_name="***";//所发布主题的名称
const char* bolid_name="***";//该板的编号-名称
/**/

WiFiClient espClient;
PubSubClient client(espClient);
long lastMsg = 0;
char msg[50];
int value = 0;

void setup_wifi() 
{
  delay(10);
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".....");
  }//链接wifi

  randomSeed(micros());
  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

void reconnect()
{
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    String clientId = bolid_name;
    if (client.connect(clientId.c_str())) {
      Serial.println("connected");
      // 连接后，发布公告 Success
      client.publish("out_topic_name", "Success!");
      } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      delay(5000);
    }
  }
}//一直循环直到连接上服务器之后

void setup() {
  Serial.begin(115200);//设置波特率
  setup_wifi();
  client.setServer(mqtt_server, 1883);
}

void loop() {
 int sensorValue = analogRead(A0);//光敏读数
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

    snprintf (msg, 50, "%ld",sensorValue );
    Serial.print("Publish message: ");
    Serial.println(msg);//同时串口输出
    client.publish("out_topic_name", msg);
    delay(10000);
  
}

```

###### 订阅消息

```
//接受信息 来控制灯泡的亮灭
#include <ESP8266WiFi.h>
#include <PubSubClient.h>

const char* ssid = "***";
const char* password = "***";
const char* mqtt_server = "***";
const char* in_Topic_name="***";//所接收主题的名称
const char* bolid_name="***";//该板的名称-编号

WiFiClient espClient;
PubSubClient client(espClient);
char msg[50];
int light = 255;

void setup_wifi() 
{ delay(10);
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print("....");
  }

  randomSeed(micros());

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}//链接wifi

void callback(char* topic, byte* payload, unsigned int length) 
{ int l=0;
  int p=1;
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("]： ");
  for (int i = length-1; i >=0; i--) {
    l+=(int)((char)payload[i]-'0')*p;
    Serial.print(l);//增加串口显示
     p*=10;
  }
  Serial.println();
  light=l;

}//传入信息改变light的值 1-亮 0-不亮

void reconnect() {
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    String clientId = "bolid_name";//该板子名称
    // Attempt to connect
    if (client.connect(clientId.c_str())) {
      Serial.println("connected");
     
      client.subscribe("in_Topic_name");//订阅主题
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      // 等待五秒钟直到链接上
      delay(5000);
    }
  }
}

void setup() 
{ pinMode(D2, OUTPUT); //控制d2的灯泡 
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
}

void loop() 
{  if (!client.connected()) {
    reconnect();
  }//如果链接没有建立 那么重新连接
  client.loop();
  delay(500);
  analogWrite(D2,light);
}

```

更多PubSubclien库的用法 详见上文的官方API文档，

* * *