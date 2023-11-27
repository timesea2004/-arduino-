# -arduino-
电机：L298N ；开发板：arduino uno R3； 四个减速电机；超声波测距模块；红外遥控模块；显示屏：ssd1306

```cpp
#include<Wire.h>//引入I2C的库
#include<IRremote.h>
#include<Adafruit_SSD1306.h>
#include <SPI.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#define IR_PIN A0
IRrecv myIRrecv(IR_PIN);
decode_results results;
#define OLED_RESET 4
Adafruit_SSD1306 display(OLED_RESET);
const int Int1 = 4;
const int Int2 = 5;
const int ENA = 3;
const int Int5 = 10;
const int Int6 = 11;
const int ENA3 = 9;
const int Trig = A3;
const int Echo = A2;
long control = results.value;
int x;
double distance,time;
void setup() {
  // put your setup code here, to run once:
  Serial.begin(9600);
  pinMode(Int1,OUTPUT);
  pinMode(Int2,OUTPUT);
  pinMode(ENA,OUTPUT);
  pinMode(Int5,OUTPUT);
  pinMode(Int6,OUTPUT);
  pinMode(ENA3,OUTPUT);
  pinMode(Trig,OUTPUT);
  pinMode(Echo,INPUT);
  display.begin(SSD1306_SWITCHCAPVCC,0x3C);//设置通讯地址
  display.clearDisplay();//清空显示
  display.setTextSize(2);//设置字体大小
  display.setTextColor(WHITE);
  display.setCursor(40,0);//设置光标位置
  display.println("shihai");
  display.display();
  analogWrite(ENA,100);//设置初始速度
  analogWrite(ENA3,100);//设置初始速度
  myIRrecv.enableIRIn();
}

void loop() {
  viewing();//显示屏函数
  IRremote();//红外控制函数
  detect();//超声波避障函数
}

void viewing(){
  if(distance<20){
    display.clearDisplay();
    display.println("DOWN");
    Serial.println("down");
    Serial.println(distance);
    display.display();
    display.setCursor(40,0);
  }
  else{
    display.clearDisplay();
    digitalWrite(ENA,200);
    display.println("GO");
    Serial.println("go");
    delay(200);
    display.display();
    display.setCursor(40,0);
  }
}

void detect(){
  digitalWrite(Trig,LOW);//delay以毫秒为单位的演示
  delayMicroseconds(2);//以微秒单位延时
  digitalWrite(Trig,HIGH);
  delayMicroseconds(10);
  digitalWrite(Trig,LOW);
  time = pulseIn(Echo,HIGH);
  distance=time/58;
  if(distance<20){//小于20减速
    digitalWrite(ENA,50);//减速至五十
    digitalWrite(ENA3,50);//减速至五十
    delay(200);
    digitalWrite(ENA,0);//减速至0
    digitalWrite(ENA3,0);//减速至0
    delay(200);
    x = 1;//设置判断值
  }
}

void IRremote(){
  if(myIRrecv.decode(&results)){
    switch(results.value){
      case 0xFF18E7:
        goForward();
        Serial.println("turn forward");
        break;
      case 0xFF11FF:
        goBackward();
        Serial.println("turn backward");
        break;
      case 0xCC16BB:
        turnLeft();
        Serial.println("turn left");
        break;
      case 0xBB15CC:
        turnRight();
        Serial.println("turn right");
        break;
    }
    
  }

}
void goForward(){
  digitalWrite(Int1,LOW);//定义引脚状态
  digitalWrite(Int2,HIGH);
  digitalWrite(Int5,LOW);//定义引脚状态
  digitalWrite(Int6,HIGH);
}
void goBackward(){
  digitalWrite(Int1,HIGH);//定义引脚状态
  digitalWrite(Int2,LOW);
  digitalWrite(Int5,HIGH);//定义引脚状态
  digitalWrite(Int6,LOW);
  if(x=1){
    digitalWrite(ENA,50);//缓慢后退
    digitalWrite(ENA3,50);//缓慢后退
    x=0;//回复判断值
  }
}
void turnLeft(){
  digitalWrite(Int1,HIGH);//定义引脚状态
  digitalWrite(Int2,LOW);
  digitalWrite(Int5,LOW);//定义引脚状态
  digitalWrite(Int6,HIGH);
}
void turnRight(){
  digitalWrite(Int1,LOW);//定义引脚状态
  digitalWrite(Int2,HIGH);
  digitalWrite(Int5,HIGH);//定义引脚状态
  digitalWrite(Int6,LOW);
}

```
