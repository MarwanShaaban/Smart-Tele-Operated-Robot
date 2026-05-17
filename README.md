# Smart-Tele-Operated-Robot
#include "BluetoothSerial.h"
#include <Wire.h> 
#include <LiquidCrystal_I2C.h>

BluetoothSerial SerialBT;
LiquidCrystal_I2C lcd(0x27, 16, 2); 

// --- تعريف البنات ---
int IN1_F = 12; int IN2_F = 13;
int IN3_F = 14; int IN4_F = 27;
int ENA_F = 18; int ENB_F = 19; 

int IN1_B = 26; int IN2_B = 25;
int IN3_B = 33; int IN4_B = 32;
int ENA_B = 5;  
int ENB_B = 15; 

int motorSpeed = 180;   // السرعة القصوى المختارة
int speedPercent = 70;  // النسبة المئوية المختارة

// دالة العرض: تأخذ قيمة السرعة الحالية (إما القيمة المختارة أو 0)
void displayCurrentStatus(int currentSpd, int currentPct) {
  lcd.setCursor(0, 1);
  lcd.print("                "); // مسح السطر التاني
  lcd.setCursor(0, 1);
  lcd.print("Spd:");
  lcd.print(currentPct);
  lcd.print("% (");
  lcd.print(currentSpd);
  lcd.print(")");
}

void updateLCD(String msg) {
  lcd.setCursor(0, 0);
  lcd.print("                "); 
  lcd.setCursor(0, 0);
  lcd.print(msg);
}

void setup() {
  Serial.begin(115200);
  SerialBT.begin("ESP32_Robot_Car");
  
  Wire.begin(21, 22); 
  lcd.init();
  lcd.backlight();
  
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Robot Ready!");
  displayCurrentStatus(0, 0); // يبدأ بـ 0 لأن الروبوت واقف

  pinMode(IN1_F, OUTPUT); pinMode(IN2_F, OUTPUT);
  pinMode(IN3_F, OUTPUT); pinMode(IN4_F, OUTPUT);
  pinMode(ENA_F, OUTPUT); pinMode(ENB_F, OUTPUT);
  pinMode(IN1_B, OUTPUT); pinMode(IN2_B, OUTPUT);
  pinMode(IN3_B, OUTPUT); pinMode(IN4_B, OUTPUT);
  pinMode(ENA_B, OUTPUT); pinMode(ENB_B, OUTPUT);
}

void loop() {
  if (SerialBT.available()) {
    char command = SerialBT.read();
    
    // تحديث "الإعدادات" فقط عند ضغط رقم، بدون تغيير الشاشة فوراً إلا لو الروبوت بيتحرك
    if (command >= '0' && command <= '9') {
      int level = command - '0';
      motorSpeed = map(level, 0, 9, 0, 255);
      speedPercent = map(level, 0, 9, 0, 100);
    }

    switch (command) {
      case 'F': 
        moveForward(); 
        updateLCD("Forward"); 
        displayCurrentStatus(motorSpeed, speedPercent); 
        break;
      case 'B': 
        moveBackward(); 
        updateLCD("Backward"); 
        displayCurrentStatus(motorSpeed, speedPercent); 
        break;
      case 'L': 
        turnLeft(); 
        updateLCD("Turn Left"); 
        displayCurrentStatus(motorSpeed, speedPercent); 
        break;   
      case 'R': 
        turnRight(); 
        updateLCD("Turn Right"); 
        displayCurrentStatus(motorSpeed, speedPercent); 
        break;  
      case 'S': 
        stopRobot(); 
        updateLCD("Stopped"); 
        displayCurrentStatus(0, 0); // يصفر السرعة على الشاشة عند الوقوف
        break;
    }
  }
}

void setSpeed(int l, int r) {
  analogWrite(ENA_F, l); analogWrite(ENA_B, l);
  analogWrite(ENB_F, r); analogWrite(ENB_B, r);
}

// --- دالات الحركة ---
void moveForward() {
  setSpeed(motorSpeed, motorSpeed);
  digitalWrite(IN1_F, LOW);  digitalWrite(IN2_F, HIGH);
  digitalWrite(IN3_F, LOW);  digitalWrite(IN4_F, HIGH);
  digitalWrite(IN1_B, LOW);  digitalWrite(IN2_B, HIGH);
  digitalWrite(IN3_B, LOW);  digitalWrite(IN4_B, HIGH);
}

void moveBackward() {
  setSpeed(motorSpeed, motorSpeed);
  digitalWrite(IN1_F, HIGH); digitalWrite(IN2_F, LOW);
  digitalWrite(IN3_F, HIGH); digitalWrite(IN4_F, LOW);
  digitalWrite(IN1_B, HIGH); digitalWrite(IN2_B, LOW);
  digitalWrite(IN3_B, HIGH); digitalWrite(IN4_B, LOW);
}

void turnRight() {
  setSpeed(motorSpeed, motorSpeed);
  digitalWrite(IN1_F, HIGH); digitalWrite(IN2_F, LOW); 
  digitalWrite(IN1_B, HIGH); digitalWrite(IN2_B, LOW);
  digitalWrite(IN3_F, LOW);  digitalWrite(IN4_F, HIGH); 
  digitalWrite(IN3_B, LOW);  digitalWrite(IN4_B, HIGH);
}

void turnLeft() {
  setSpeed(motorSpeed, motorSpeed);
  digitalWrite(IN1_F, LOW);  digitalWrite(IN2_F, HIGH); 
  digitalWrite(IN1_B, LOW);  digitalWrite(IN2_B, HIGH);
  digitalWrite(IN3_F, HIGH); digitalWrite(IN4_F, LOW);  
  digitalWrite(IN3_B, HIGH); digitalWrite(IN4_B, LOW);
}

void stopRobot() {
  digitalWrite(IN1_F, LOW); digitalWrite(IN2_F, LOW);
  digitalWrite(IN3_F, LOW); digitalWrite(IN4_F, LOW);
  digitalWrite(IN1_B, LOW); digitalWrite(IN2_B, LOW);
  digitalWrite(IN3_B, LOW); digitalWrite(IN4_B, LOW);
}