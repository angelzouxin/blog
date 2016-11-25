title: arduino开发日志Day4
date: 2015-08-16 10:53:27
categories: arduino
tags: arduino
---
#优化蜂鸣器
<!--more-->
##用蜂鸣器演奏音乐
在整合近距离追踪装置时，因为蜂鸣器声音过于刺耳，所以想将其改为演奏音乐，由于无源蜂鸣器是通过电流的频率来发出不同声音，所以可以利用这点，用蜂鸣器演奏出音乐，该处将其改为卡农。用蜂鸣器演奏卡农的程序如下。
```cpp
#define NTD0 -1
#define NTD1 294
#define NTD2 330
#define NTD3 350
#define NTD4 393
#define NTD5 441
#define NTD6 495
#define NTD7 556

#define NTDL1 147
#define NTDL2 165
#define NTDL3 175
#define NTDL4 196
#define NTDL5 221
#define NTDL6 248
#define NTDL7 278

#define NTDH1 589
#define NTDH2 661
#define NTDH3 700
#define NTDH4 786
#define NTDH5 882
#define NTDH6 990
#define NTDH7 112
//列出全部D调的频率
#define WHOLE 1
#define HALF 0.5
#define QUARTER 0.25
#define EIGHTH 0.25
#define SIXTEENTH 0.625
//列出所有节拍
int tune[]=                 //根据简谱列出各频率
{
  NTD3,NTD3,NTD4,NTD5,
  NTD5,NTD4,NTD3,NTD2,
  NTD1,NTD1,NTD2,NTD3,
  NTD3,NTD2,NTD2,
  NTD3,NTD3,NTD4,NTD5,
  NTD5,NTD4,NTD3,NTD2,
  NTD1,NTD1,NTD2,NTD3,
  NTD2,NTD1,NTD1,
  NTD2,NTD2,NTD3,NTD1,
  NTD2,NTD3,NTD4,NTD3,NTD1,
  NTD2,NTD3,NTD4,NTD3,NTD2,
  NTD1,NTD2,NTDL5,NTD0,
  NTD3,NTD3,NTD4,NTD5,
  NTD5,NTD4,NTD3,NTD4,NTD2,
  NTD1,NTD1,NTD2,NTD3,
  NTD2,NTD1,NTD1
};
float durt[]=                   //根据简谱列出各节拍
{
  1,1,1,1,
  1,1,1,1,
  1,1,1,1,
  1+0.5,0.5,1+1,
  1,1,1,1,
  1,1,1,1,
  1,1,1,1,
  1+0.5,0.5,1+1,
  1,1,1,1,
  1,0.5,0.5,1,1,
  1,0.5,0.5,1,1,
  1,1,1,1,
  1,1,1,1,
  1,1,1,0.5,0.5,
  1,1,1,1,
  1+0.5,0.5,1+1,
};
int length;
int tonepin=6;   //得用6号接口
void setup()
{
  pinMode(tonepin,OUTPUT);
  length=sizeof(tune)/sizeof(tune[0]);   //计算长度
}
void loop()
{
  for(int x=0;x<length;x++)
  {
    tone(tonepin,tune[x]);
    delay(500*durt[x]);   //这里用来根据节拍调节延时，500这个指数可以自己调整，在该音乐中，我发现用500比较合适。
    noTone(tonepin);
  }
  delay(2000);
}
```

##整合代码
将音乐演奏整合至模块时，碰到了有关问题，**如何使蜂鸣器在用蓝牙控制时，播出完整的歌。** 
解决方法：通过一个bool类型变量记录其是否为开启状态，再用i记录当前音符，每循环一次，音符位置向后移动一位（i++）。

```cpp
String comdata = "";
int Flag = 0;

//gps set
#include <SoftwareSerial.h>
#include "TinyGPS.h"
TinyGPS gps;
SoftwareSerial Gsmserial(10,11);
String TEMPdata="";
#define apikey "oQ********************************fjgKEA"
unsigned long deviceid=**411;
String sensorid="TEMP";
String sensor1id="PM25";
////////////////////////////////////////////////////////////////////////////////////////
#define Apikey "tX***************************q2iNA4A"
unsigned long device1id=***05;
String sensor11id="GPSdata";
        bool newData = false; 
        char gps_year[8];  
        char gps_mon[3];  
        char gps_day[3];  
        char gps_hour[3];  
        char gps_min[3];  
        char gps_sec[3];        
        char gps_lon[20]={"\0"};  
        char gps_lat[20]={"\0"}; 
        char gps_heigh[20]={"\0"};
        char gps_sms[100];
//////////////////////////////////////////////////////////////////////////////
char Time_sms[20]={"\0"};
char Date_sms[20]={"\0"};
char Lat_sms[20]={"\0"};
char Lon_sms[20]={"\0"};

//buzzer set
#define NTD0 -1
#define NTD1 294
#define NTD2 330
#define NTD3 350
#define NTD4 393
#define NTD5 441
#define NTD6 495
#define NTD7 556

#define NTDL1 147
#define NTDL2 165
#define NTDL3 175
#define NTDL4 196
#define NTDL5 221
#define NTDL6 248
#define NTDL7 278

#define NTDH1 589
#define NTDH2 661
#define NTDH3 700
#define NTDH4 786
#define NTDH5 882
#define NTDH6 990
#define NTDH7 112
//列出全部D调的频率
#define WHOLE 1
#define HALF 0.5
#define QUARTER 0.25
#define EIGHTH 0.25
#define SIXTEENTH 0.625
//列出所有节拍
int tune[]=                 //根据简谱列出各频率
{
  NTD3,NTD3,NTD4,NTD5,
  NTD5,NTD4,NTD3,NTD2,
  NTD1,NTD1,NTD2,NTD3,
  NTD3,NTD2,NTD2,
  NTD3,NTD3,NTD4,NTD5,
  NTD5,NTD4,NTD3,NTD2,
  NTD1,NTD1,NTD2,NTD3,
  NTD2,NTD1,NTD1,
  NTD2,NTD2,NTD3,NTD1,
  NTD2,NTD3,NTD4,NTD3,NTD1,
  NTD2,NTD3,NTD4,NTD3,NTD2,
  NTD1,NTD2,NTDL5,NTD0,
  NTD3,NTD3,NTD4,NTD5,
  NTD5,NTD4,NTD3,NTD4,NTD2,
  NTD1,NTD1,NTD2,NTD3,
  NTD2,NTD1,NTD1
};
float durt[]=                   //根据简谱列出各节拍
{
  1,1,1,1,
  1,1,1,1,
  1,1,1,1,
  1+0.5,0.5,1+1,
  1,1,1,1,
  1,1,1,1,
  1,1,1,1,
  1+0.5,0.5,1+1,
  1,1,1,1,
  1,0.5,0.5,1,1,
  1,0.5,0.5,1,1,
  1,1,1,1,
  1,1,1,1,
  1,1,1,0.5,0.5,
  1,1,1,1,
  1+0.5,0.5,1+1,
};

int length;
int tonepin=6;   //调用6号接口



// initialize the library with the numbers of the interface pins


// The TinyGPS++ object
TinyGPSPlus gps;

// The serial connection to the GPS device
SoftwareSerial ss(3, 2);

// Keep track of which screen we are on
int currentScreen = 0;

// Signal Icon
byte signalIcon[8] = {
  0b00000,
  0b11111,
  0b00000,
  0b01110,
  0b00000,
  0b00100,
  0b00100,
  0b00000
};

// Direction Icon
byte courseIcon[8] = {
  0b00000,
  0b00000,
  0b00100,
  0b01110,
  0b01110,
  0b11011,
  0b10001,
  0b00000
};

// Setup Check
bool initiating = true;

void setup() {
  ss.begin(9600);
  Serial.begin(9600);
  delay(100);
  pinMode(tonepin,OUTPUT);
  length=sizeof(tune)/sizeof(tune[0]);  //计算长度
}

void loop() {
  // Feed the GPS data from Serial into the TinyGPS library
  while(Serial.available() > 0)
  {
    comdata += char(Serial.read());
    delay(2);
  }
  if(comdata.length() > 0)   //当接收到字符后，判断命令
   {
    comdata += "\0";
    if(comdata == "on") //当命令为“on”时
    {
       if(Flag == 0) //原为关闭则打开
      {
         digitalWrite(tonepin,HIGH);
         Serial.println("Follow The Music and find the goods.");
         for(int x=0;x<length;x++)
          {
            tone(tonepin,tune[x]);
            delay(500*durt[x]);   //这里用来根据节拍调节延时，500这个指数可以自己调整，在该音乐中，我发现用500比较合适。
            noTone(tonepin);
          }
         delay(2000);
       
         Flag = 1;
      }     
      delay(2);
    }
    else if(comdata == "off") //命令为“off”时
    {
      if(Flag == 1) //原为打开则关闭
      {
        digitalWrite(redled,LOW);
        Serial.println("The Buzzer has closed.");
         Flag = 0;
      }  
      else if(comdata == "Find")
      {
        Check_gps();
        Serial.println(gps_heigh);
        Serial.println(gps_lon);
        Serial.println(gps_lat);
        Serial.println(Data_sms);
        Serial.println(Time_gps);
        TEMPdata="";
      }
      delay(2);
    }
    comdata = "";
   }
}

void Check_gps()  
{   
  newData=false;    
  unsigned long chars;    
  unsigned short sentences, failed;       
   
  // For one second we parse GPS data and report some key values    
  for (unsigned long start = millis(); millis() - start < 1000;)    
  {      
    while (Serial.available())      
    {        
    char c = Serial.read();       
    // Serial.write(c);
            // uncomment this line if you want to see the GPS data flowing       
    if (gps.encode(c)) // Did a new valid sentence come in?          
    newData = true;      
    }    
  }       
  if (newData)    
  {  
  float flat, flon,alti;      
    unsigned long age;      
    int _year;     
    byte _month, _day,_hour,_minute,_second,_hundredths;  
    gps.f_altitude();    
    gps.f_get_position(&flat, &flon, &age);          
    gps.crack_datetime(&_year,&_month,&_day,&_hour,&_minute,&_second,&_hundredths,&age);     
    flat == TinyGPS::GPS_INVALID_F_ANGLE ? 0.0 : flat, 6;          
    flon == TinyGPS::GPS_INVALID_F_ANGLE ? 0.0 : flon, 6; 
    alti=gps.f_altitude(); 
    Serial.print(alti);
    dtostrf(alti, 0, 3, gps_heigh);   
    dtostrf(flat, 0, 6, gps_lat);       
    dtostrf(flon, 0, 6, gps_lon);                      
    strcpy(Lat_sms,"lat:");
    strcat(Lat_sms,gps_lat);        
    strcpy(Lon_sms,"lon: ");
    strcat(Lon_sms,gps_lon);         
    strcpy(Date_sms,"Date:");
    itoa(_year,gps_year,10);
    strcat(Date_sms,gps_year); 
    strcat(Date_sms,"/");     
    itoa(_month,gps_mon,10); 
    if(strlen(gps_mon)==1)  strcat(Date_sms,"0");     
    strcat(Date_sms,gps_mon);
    strcat(Date_sms,"/");                   
    itoa(_day,gps_day,10);   
    if(strlen(gps_day)==1)  strcat(Date_sms,"0");     
    strcat(Date_sms,gps_day); 
    _hour=_hour+8;
    strcpy(Time_sms,"Time:");
    itoa(_hour,gps_hour,10); 
    if(strlen(gps_hour)==1) strcat(Time_sms,"0");
    strcat(Time_sms,gps_hour);
    strcat(Time_sms,":");                 
    itoa(_minute,gps_min,10);
    if(strlen(gps_min)==1)  strcat(Time_sms,"0"); 
    strcat(Time_sms,gps_min);
    strcat(Time_sms,":");                 
    itoa(_second,gps_sec,10);
    if(strlen(gps_sec)==1)  strcat(gps_sms,"0"); 
    strcat(Time_sms,gps_sec);                         
  }             
}
```