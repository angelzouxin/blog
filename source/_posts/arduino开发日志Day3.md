title: arduino开发日志Day3
date: 2015-08-15 15:08:08
categories: arduino
tags: arduino
---

##近距离GPS追踪
<!--more-->
使用蓝牙与GPS模块，实现近距离的坐标收发。
![为了便于排线，将所有模块先放在面包板上][pic1]
由于蓝牙和GPS同时具有收发功能，所以使用软串口接GPS，来解决信息的冲突问题。*在这里，我们设**3,4**软串口作为GPS的RT口*。

>##注意
在使用GPS模块时，一定要设置正确波特率，该模块为**9600**，若设置错误，将无法获取正确的GPS信息。
整合前面的代码时，需要注意，输出串口要设置正确。

```cpp
String comdata = "";

//gps set
#include <SoftwareSerial.h>
#include "TinyGPS.h"
TinyGPS gps;
SoftwareSerial Gsmserial(10,11);
String TEMPdata="";

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

char Time_sms[20]={"\0"};
char Date_sms[20]={"\0"};
char Lat_sms[20]={"\0"};
char Lon_sms[20]={"\0"};


// initialize the library with the numbers of the interface pins


// The TinyGPS++ object
TinyGPSPlus gps;

// The serial connection to the GPS device
SoftwareSerial ss(3, 4);


void setup() {
  ss.begin(9600);
  Serial.begin(9600);
  delay(100);
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
      if(comdata == "Find")
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
    itoa(_month,gps_mon,10); 
    if(strlen(gps_mon)==1)  strcat(Date_sms,"0");     
    strcat(Date_sms,gps_mon);                 
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

[pic1]:/img/Day3/GPSBT.jpg

#整合近距离追踪代码
近距离部分分为
- 用上位机连接蓝牙
- 通过接收上位机发送的对应指令，使下位机执行对应的命令
- 用上位机开关蜂鸣器
- 用上位机获取GPS信息

代码如下
```cpp
String comdata = "";
bool Flag = false;

//gps set
#include <SoftwareSerial.h>
#include "TinyGPS.h"
TinyGPS gps;
SoftwareSerial Gsmserial(10,11);
String TEMPdata="";

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

char Time_sms[20]={"\0"};
char Date_sms[20]={"\0"};
char Lat_sms[20]={"\0"};
char Lon_sms[20]={"\0"};

#define tonepin 6

// The TinyGPS++ object
TinyGPSPlus gps;

// The serial connection to the GPS device
SoftwareSerial ss(3, 2);


void setup() {
  ss.begin(9600);
  Serial.begin(9600);
  delay(100);
  pinMode(tonepin,OUTPUT);
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
      flag = true;
    }     
      delay(2);
  }
  else if(comdata == "off") //命令为“off”时
    flag = false; 
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
 if(flag)
 {
   for(int i = 0; i < 100; i++)
   {
      digital.write(tonepin,HIGH);
      delay(2);
      digital.write(tonepin,LOW);
   }
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