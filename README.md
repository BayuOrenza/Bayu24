# Proyek Akhir Sistem Keamanan Pintu 

#include <Keypad.h>
#include <SoftwareSerial.h>// impor library softwareserial
#include <avr/wdt.h>
#include <Wire.h> 
#include <LiquidCrystal_I2C.h>

LiquidCrystal_I2C lcd(0x27,16,2);
SoftwareSerial BlueSer(14, 15); // RX, TX

const byte ROWS = 4; 
const byte COLS = 4; 

char hexaKeys[ROWS][COLS] = {
  {'1', '2', '3', 'A'},
  {'4', '5', '6', 'B'},
  {'7', '8', '9', 'C'},
  {'*', '0', '#', 'D'}
};

byte rowPins[ROWS] = {9, 8, 7, 6}; 
byte colPins[COLS] = {5, 4, 3, 2}; 

Keypad customKeypad = Keypad(makeKeymap(hexaKeys), rowPins, colPins, ROWS, COLS); 

int sensorPIR = 0, sensorState = 0, count = 0, salah = 0;
int BluetoothData, ncount = 0, noaction = 0;
unsigned long diam;

String password = "";

void setup(){
  Serial.begin(9600);
  Serial.println("Proyek Akhir");
  pinMode(10, INPUT);  // PIR
  pinMode(11, OUTPUT); // Buzzer
  digitalWrite(11, LOW); // Buzzer OFF
  pinMode(12, OUTPUT); // door lock
  digitalWrite(12, HIGH);
  BlueSer.begin(9600);
  lcd.init(); 
  lcd.backlight();
  lcd.print("Proyek Akhir");
}
  
void loop(){

  
  if ( sensorState == 1 ) {

    if ( noaction == 0 ) {
      if ( millis() - diam >= 1000 ) {
        ncount++;
        wdt_reset(); 
        diam = millis();
      }
      if ( ncount >= 10 ) {
        ncount = 0;
        digitalWrite(11,HIGH);
        Serial.println("System Reset");
        while(1);
      }
    }
   
    if (BlueSer.available()) {
      noaction = 1;
      wdt_disable();
      BluetoothData = BlueSer.read();
      Serial.print("Bluetooth: ");
      Serial.println(BluetoothData);
      if (BluetoothData == '1') {  
        digitalWrite(12,LOW);       
      }
      else if (BluetoothData == '0') {
        digitalWrite(12,HIGH);
      }  
    }
  
    char customKey = customKeypad.getKey();
    
    if (customKey){
      noaction = 1;
      wdt_disable();
      Serial.println(customKey);
      count++;
      if ( count > 3 ) {          
        password += String(customKey);
        Serial.print("Password: ");
        Serial.println(password);
        lcd.clear();
        lcd.setCursor(0,0);
        lcd.print("Pintu Terbuka");
        if (!password.equals("1234")) {
          salah++;
          Serial.println("Password salah.");
          lcd.clear();
          lcd.setCursor(0,0);
          lcd.print("Password Salah");
          if ( salah > 2 ) {
            digitalWrite(11, HIGH);
            delay(3000);
            digitalWrite(11, LOW);
            salah = 0;
          }
        }
        else {
          digitalWrite(12, LOW);
          delay(5000);
          digitalWrite(12, HIGH);
        }
        count = 0;
        password = "";
      }
      else {
        password += String(customKey);
      }
    }  
  }
  else {
    sensorPIR = digitalRead(10);  // Baca sensor PIR
    if ( sensorPIR == HIGH ) {
      if ( sensorState == 0 ) {
        sensorState = 1;        
        Serial.println("Motion detected.");
        lcd.setCursor(0,1);
        lcd.print("Motion detected.");
        wdt_enable(WDTO_2S);
        noaction = 0;
        diam = millis();
      }
    }
  }
} 
