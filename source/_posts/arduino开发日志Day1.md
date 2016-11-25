title: arduino开发日志Day1
date: 2015-08-13 09:29:07
categories: 
  - arduino
tags:
  - arduino
---

##修改HC-05蓝牙模块参数
<!--more-->
修改方法分为两种，分别为基于arduino修改和TTL接USB，直接在电脑上修改**在更改时，BT模块需要加入AT模式**，HC-05模块在通电前按住模块上的红点，若通电后，模块上的指示灯为常亮后熄灭，则代表进入AT模式，否则重新连接。
###基于arduino修改
基于arduino修改原理为通过AT指令来修改，先将指令烧录至arduino中，再连接arduino与BT模块**由于arduino缓冲区以及电压等问题，修改可能不成功，可以多试几次**，最好还是使用电脑离线修改。
####编写修改参数代码
``` cpp
void setup()
{
	Serial.begin(38400);
	delay(100);
	Serial.println("AT");
	delay(100);
	Serial.println("AT+NAME=31401110");//修改模块名
	delay(100);
	Serial.println("AT+ROLE=0");//设置主从模式，0从机，1主机
	delay(100);
	Serial.println("AT+PSWD=1128");//设置配对密码
	delay(100);
	Serial.println("AT+UART=9600,0,0");//设置波特率，停止位和校验位
	delay(100);
	Serial.println("AT+RMAAD");
}

void loop()
{

}
```
写完后烧录至arduino版

####连接BT和arduino
连接BT模块和arduino版，连接示意图如图所示,**记得进入AT模式**
![][pic1]

###电脑上修改
用TTL与USB转接口使蓝牙模块与电脑连接，下载完USB驱动后，打开[蓝牙修改器][link1],选择正确串口，然后**选择正确的模块和波特率**，通讯测试后，若返回*OK*，则代表连接成功，然后即可在程序上更改。
![修改成功示意图][pic2]


##蓝牙下位机尝试
尝试用蓝牙控制LED灯的开关。
###读取字符串
由于读取时，*Serial.read()*为单字读取，所以，可以定义一个空的*String*，通过循环，成为字符串。用*Serial.available()*判断是否为结尾。
###控制灯的开关
控制即和普通代码一样，通过*if - else*语句判断。烧入程序后，运行如下显示。
![连接蓝牙后，在手机上输入on，led打开][pic3] 
![输入off，led关闭][pic4]
![手机命令][pic5]

```cpp
String comdata = "";
int Flag = 0;
#define redled 2

void setup() {
  // put your setup code here, to run once:
  pinMode(redled,OUTPUT);  
  Serial.begin(9600);
  delay(100);

}

void loop() {
  // put your main code here, to run repeatedly:
   while(Serial.available() > 0)
   {
    comdata += char(Serial.read());   //接受字符，并组成字符串
    delay(2); //通过延迟来保证读取的稳定
   }
   
   if(comdata.length() > 0)   //当接收到字符后，判断命令
   {
    comdata += "\0";

    if(comdata == "on") //当命令为“on”时
    {
       if(Flag == 0) //原为关闭则打开
      {
         digitalWrite(redled,HIGH);
         Serial.println("open the light"); //向手机发出状态说明
         Flag = 1;
      }     
      else
      {
        Serial.println("The Light has already opened.");    
      }
      delay(2);
    }

    else if(comdata == "off") //命令为“off”时
    {
      if(Flag == 1) //原为打开则关闭
      {
        digitalWrite(redled,LOW);
         Serial.println("close the light");
         Flag = 0;
      }  
      else
      {
        Serial.println("The Light has already closed.");    
      }
       delay(2);
    }
    else
    {
      Serial.println("Unclear Command");
      delay(2);
    }
    comdata = "";
   }
}
```

##LCD屏与arduino的连接以及显示
LCD屏为**LCD Keypad Shield**,连接方式为与UNO版直接对接，插上电源后，即工作。该版有2行16个字符液晶，未使用的IO口都有扩展出来备用，充分利用IO口。
![连接如图所示][pic6]
###LCD屏按键检查
通过在arduino版中写入程序来进行检测，测试代码如下
！[程序效果如图][pic7]

```cpp
#include <LiquidCrystal.h>

LiquidCrystal lcd(8, 13, 9, 4, 5, 6, 7);

char msgs[5][16] = {"Right Key OK ",  //设置输出信息
                    "Up Key OK    ",               
                    "Down Key OK  ",
                    "Left Key OK  ",
                    "Select Key OK" };

int adc_key_val[5] ={50, 200, 400, 600, 800 };
int NUM_KEYS = 5;
int adc_key_in;
int key=-1;
int oldkey=-1;

void setup()
{
  lcd.clear(); 
  lcd.begin(16, 2);
  lcd.setCursor(0,0); 
  lcd.print("ADC key testing");  //在LCD屏上层显示文字
}

void loop()
{
  adc_key_in = analogRead(0);    // 读取来自传感器的值
  key = get_key(adc_key_in);  // 转换成按键
 
  if (key != oldkey)   // 如果按键被检测到
   {
    delay(50);  // 等待反跳时间
    adc_key_in = analogRead(0);    // 读取来自传感器的值
    key = get_key(adc_key_in);    // 转换成按键
    if (key != oldkey)    
    {   
      lcd.setCursor(0, 1);
      oldkey = key;
      if (key >=0){
           lcd.print(msgs[key]);              
      }
    }
  }
 delay(100);
}

// ADC值转换为数字键
int get_key(unsigned int input)
{
    int k;
   
    for (k = 0; k < NUM_KEYS; k++)
    {
      if (input < adc_key_val[k])
      {
            return k;
        }
   }
   
    if (k >= NUM_KEYS)k = -1;  // 没有有效的键按下
    return k;
}
```

###蓝牙与LCD显示
在LCD的扩展IO口上按正确引脚接上蓝牙，并通过更改前面的代码，实现在LCD屏上显示文字
连接示意图如下图
![][pic8]


[pic1]: /img/Day1/BT.png
[pic2]: /img/Day1/BT2.jpg
[link1]: http://pan.baidu.com/s/1ntMjXc1
[pic3]:/img/Day1/BTC2.jpg
[pic4]:/img/Day1/BTC1.jpg
[pic5]:/img/Day1/BTC3.jpg
[pic6]: /img/Day1/LCD1.jpg
[pic7]: /img/Day1/LCD2.jpg
[pic8]:/img/Day1/LCD3.jpg