title: arduino开发日志Day2
date: 2015-08-14 20:36:31
categories: 
  - arduino
tags:
  - arduino
---
##确定任务书
<!--more-->
确定十天开发内容，任务为**基于arduino的GPS追踪器原型制作**。
###器材
 - 使用模块有:*arduino UNO板*，*蜂鸣器*，*蓝牙模块*，*GPS模块*和*GSM模块*。

###功能
 - 手机端通过蓝牙与其近距离相连，通过命令开关蜂鸣器，达到近距离追踪找回物品的效果。
 - 通过短信可与追踪器进行远距离的定位追踪，通过短信显示位置。

###任务
 - 实现蓝牙和手机之间的通信。
 - 将GPS模块中的数据解析成坐标.显示在手机上。
 - 通过手机，控制追踪器上蜂鸣器的开关。
 - 通过GSM模块，实现手机与追踪器间短信的收发。
 - 学会将GPS信息通过GSM模块发送至手机

##GPS模块的使用
###GPS模块介绍
![模块样式][pic1]
GPS模块有四个引脚，分别为R,T,G,V,分别对应arduino上的T,R,G,V。
 >###**注意**
由于GPS信号收发可能与arduino上的RT有干扰，所以当有别的模块时，使用`SoftwareSerial ss(R,T)`将函数中的*pin R，T*口设置为串口进行信息传递。前一个数字即Rx，后一个为Tx。

###GPS模块信息接收
从网上找到了一个简单的收发程序
```cpp
#include <SoftwareSerial.h>

SoftwareSerial gps(3,4);      //Arduino Uno 的 pin 3是 Rx， pin 4是 Tx

void setup()
{
  Serial.begin(9600);
  gps.begin(9600);
}

void loop()
{
  while(gps.available()>0)
  {
    char c = gps.read();
    Serial.println(c);
  }
}

```

通过该程序，在监视口可以出现读数，但是读数如下图![][pic2]。
查看程序后，将`println`改为`print`后，有读数。![][pic3]。

###GPS模块信息转化为坐标
调用[`TinyGps`][link1]库，通过样例，测试GPS模块的坐标转换功能。
>**注意**
在使用时尽量把模块放到窗外或室外空旷地带，否则无法进行定位。

```cpp
#include <SoftwareSerial.h>

#include <TinyGPS.h>

/* This sample code demonstrates the normal use of a TinyGPS object.
   It requires the use of SoftwareSerial, and assumes that you have a
   4800-baud serial GPS device hooked up on pins 4(rx) and 3(tx).
*/

TinyGPS gps;
SoftwareSerial ss(4, 3);

void setup()
{
  Serial.begin(115200);
  ss.begin(4800);
  
  Serial.print("Simple TinyGPS library v. "); Serial.println(TinyGPS::library_version());
  Serial.println("by Mikal Hart");
  Serial.println();
}

void loop()
{
  bool newData = false;
  unsigned long chars;
  unsigned short sentences, failed;

  // For one second we parse GPS data and report some key values
  for (unsigned long start = millis(); millis() - start < 1000;)
  {
    while (ss.available())
    {
      char c = ss.read();
      // Serial.write(c); // uncomment this line if you want to see the GPS data flowing
      if (gps.encode(c)) // Did a new valid sentence come in?
        newData = true;
    }
  }

  if (newData)
  {
    float flat, flon;
    unsigned long age;
    gps.f_get_position(&flat, &flon, &age);
    Serial.print("LAT=");
    Serial.print(flat == TinyGPS::GPS_INVALID_F_ANGLE ? 0.0 : flat, 6);
    Serial.print(" LON=");
    Serial.print(flon == TinyGPS::GPS_INVALID_F_ANGLE ? 0.0 : flon, 6);
    Serial.print(" SAT=");
    Serial.print(gps.satellites() == TinyGPS::GPS_INVALID_SATELLITES ? 0 : gps.satellites());
    Serial.print(" PREC=");
    Serial.print(gps.hdop() == TinyGPS::GPS_INVALID_HDOP ? 0 : gps.hdop());
  }
  
  gps.stats(&chars, &sentences, &failed);
  Serial.print(" CHARS=");
  Serial.print(chars);
  Serial.print(" SENTENCES=");
  Serial.print(sentences);
  Serial.print(" CSUM ERR=");
  Serial.println(failed);
  if (chars == 0)
    Serial.println("** No characters received from GPS: check wiring **");
}
```
经过测试，发现在测试工具中可以显示其坐标信息,但是使用标程却无法获取信息，但是在使用另外一个库的标程后，发现可以获得准确坐标。
![][pic4]
经过对比，发现模块的波特率为9600，把`ss.begin(4800)`改为`ss.begin(9600)`后即可接收到准确信号。
###整合蓝牙和蜂鸣器，实现近距离追踪
蜂鸣器为无源蜂鸣器，其两个引脚，分别接*pin口与GRD口*，pin口需要自己设置，可以在开头用`#define tonepin 6`或`int tonepin=6`来设定对应pin口,此处设为6。详细代码如下。
>**注意**
由于无源蜂鸣器是通过电流的通断来生成不同赫兹的声音，所以需要一个循环来让其可以发出声音。

```cpp
#define tonepin 6;//设置控制蜂鸣器的数字6脚

void setup()
{
 pinMode(tonepin,OUTPUT);//设置数字IO脚模式，OUTPUT为输出
}

void loop()
{
   unsigned char i,j;
   while(1)
  {
    for(i=0;i<80;i++)//输出一个频率的声音
    {
      digitalWrite(tonepin,HIGH);//使该pin口通电，发声音
      delay(1);//延时1ms
      digitalWrite(tonepin,LOW);//断开电流，不发声音
      delay(1);//延时ms
    }
    for(i=0;i<100;i++)//输出另一个频率的声音,这里的100与前面的80一样，用来控制频率，可以自己调节
    {
      digitalWrite(tonepin,HIGH);
      delay(2);
      digitalWrite(tonepin,LOW);
      delay(2);
    }
  }
}

```
###整合蓝牙与蜂鸣器
在第一天中，我们学会了蓝牙接收字符串的方法，因此，我们可以用该方法，接收指定命令，来使蜂鸣器连通或断开，完成近距离的追踪，可以设*on*为打开，*off*为关闭，**注意，由于蜂鸣器需要**通过电流的通断来实现发声，所以，我们可以设定一个`bool`类的值如*flag*去判断是否打开蜂鸣器，判断完后，在里面加入上文中蜂鸣器的发声代码，来完成该模块。

```cpp
#define tonepin 6;//设置控制蜂鸣器的数字6脚

String comdata = ""; //用来接收命令
bool Tonpen = false; //判断蜂鸣器是否开启

void setup()
{
 pinMode(tonepin,OUTPUT);//设置数字IO脚模式，OUTPUT为输出
 Serial.begin(9600);
}

void loop()
{
  unsigned char i,j;

  while(Serial.available>0)
  {
    comdata += char(Serial.read());
    delay(2;)
  } 
  
  if(comdata.length() > 0)
  {
    if(comdata == "on") flag = true;
    else if(comdata == "off") flag = false;
    comdata = "";
    delay(2);
  }

  if(flag) //当蜂鸣器开启时，发声
  {
    for(i=0;i<100;i++) //输出另一个频率的声音,这里的i，用来控制频率，可以自己调节
    {
      digitalWrite(tonepin,HIGH);
      delay(2);
      digitalWrite(tonepin,LOW);
      delay(2);
    }
  }
  
}

```

[link1]:http://pan.baidu.com/s/1bnoEGr1
[pic1]:/img/Day2/GPS.jpg
[pic2]:/img/Day2/GPS2.jpg
[pic3]:/img/Day2/GPS1.jpg
[pic4]:/img/Day2/GPS3.jpg