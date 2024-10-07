# IOT
#include <Wire.h>  // library for I2C protocol  
 #include <LiquidCrystal_I2C.h> // library for I2C LCD  
 LiquidCrystal_I2C lcd(0x27,20,4); // set the LCD address to 0x27 for a 16 chars and 2 line display  

 #include <DFRobot_DHT11.h>
DFRobot_DHT11 DHT;
#define DHT11_PIN D8

#define BLYNK_TEMPLATE_ID "TMPLD1eBY7zE"
#define BLYNK_DEVICE_NAME "ioTload"
#define BLYNK_AUTH_TOKEN "B5FnQVxfAXLwtjkruKvUp65vWH-4ymoE"

#define sonMax 6
#define tmpMax 30
#define tmpMin 25

#define wtrMax 90
#define wtrMin 25


#define ledStr V1
#define lightStr V2
#define fanStr V3
#define acStr V4
#define pmpStr V5
#define tmpStr V6
#define hmdStr V7
#define wtrStr V8
#define autoStr V10
#define ledPin D3
#define lightPin D4
#define fanPin D5
#define acPin D6
#define pmpPin D0

#include <NewPing.h>

#define TRIGGER_PIN  D7  // Arduino pin tied to trigger pin on the ultrasonic sensor.
#define ECHO_PIN     1  // Arduino pin tied to echo pin on the ultrasonic sensor.
#define MAX_DISTANCE 20 // Maximum distance we want to ping for (in centimeters). Maximum sensor distance is rated at 400-500cm.

NewPing sonar(TRIGGER_PIN, ECHO_PIN, MAX_DISTANCE); // NewPing setup of pins and maximum distance.



#include <ESP8266WiFi.h>
#include <BlynkSimpleEsp8266.h>
char auth[] = BLYNK_AUTH_TOKEN;
//char ssid[] = "TP-Link_4954";
//char pass[] = "SRK01675512655";
char ssid[] = "Bodyshape";
char pass[] = "Gym@123332200#";
int counter=0, ledState=0;
int tmpDt=0, hmDt=0, waterLevel = 0, waterLevel2 =0;
bool autoFAN = 0;
uint8_t storage = 0;
BlynkTimer timer;


BLYNK_WRITE(ledStr){ //Serial.println("inside switch function");
  int value = param.asInt();
  digitalWrite(ledPin, !value);
}
BLYNK_WRITE(lightStr){ //Serial.println("inside switch function");
  int value = param.asInt();
  digitalWrite(lightPin, !value);
}
BLYNK_WRITE(fanStr){ //Serial.println("inside switch function");
  int value = param.asInt();
  digitalWrite(fanPin, !value);
}
BLYNK_WRITE(acStr){ //Serial.println("inside switch function");
  int value = param.asInt();
  digitalWrite(acPin, !value);
}
BLYNK_WRITE(pmpStr){ //Serial.println("inside switch function");
  int value = param.asInt();
  digitalWrite(pmpPin, !value);
}
BLYNK_WRITE(autoStr){ //Serial.println("inside switch function");
  autoFAN = param.asInt();
}
BLYNK_CONNECTED(){ 
  Blynk.syncAll(); 
  pinMode(3,OUTPUT);
  digitalWrite(3,LOW);
  }

void myTimerEvent()
{
  DHT.read(DHT11_PIN);
  if(DHT.temperature < 255) tmpDt =  DHT.temperature -5;
  if(DHT.humidity < 255)  hmDt = DHT.humidity;
  int sonData = sonar.ping_cm();
  if(sonData>=0 && sonData< 9) { waterLevel = 8 - sonData;
    if( waterLevel < 0) waterLevel = 0;
  else if(waterLevel>8) waterLevel = 8;
  waterLevel2 = map(waterLevel, 0,sonMax, 0, 100);

}
//  Serial.print("SonData:");
//  Serial.print(sonData);
//  Serial.print("  waterLevel:");
//  Serial.print(waterLevel);
//  Serial.print("  waterLevel2:");
//  Serial.println(waterLevel2);
  
  Blynk.virtualWrite(wtrStr, waterLevel2);
  Blynk.virtualWrite(tmpStr, tmpDt);
  Blynk.virtualWrite(hmdStr, hmDt);

  if(waterLevel2 < wtrMin ) { Blynk.virtualWrite(pmpStr, 1); digitalWrite(pmpPin,0); }
  else if(waterLevel2 > wtrMax ) { Blynk.virtualWrite(pmpStr, 0); digitalWrite(pmpPin,1); }

  if(autoFAN){
    if(tmpDt > tmpMax ) {
      Blynk.virtualWrite(autoStr, 1);
      Blynk.virtualWrite(fanStr, 0); digitalWrite(fanPin,1); 
      Blynk.virtualWrite(acStr, 1); digitalWrite(acPin,0); 
    }
    
    else if(tmpDt > tmpMin) { 
      Blynk.virtualWrite(autoStr, 1); 
      Blynk.virtualWrite(fanStr, 1); digitalWrite(fanPin,0); 
      Blynk.virtualWrite(acStr, 0); digitalWrite(acPin,1); 
      } 
    else Blynk.virtualWrite(autoStr, 0);
  }
  

  lcd.setCursor(0,0);
  lcd.print("IoT Home Automation");  
  lcd.setCursor(0,1);  
  lcd.print("Temperature: ");  lcd.print(tmpDt); lcd.print(" C");

  lcd.setCursor(0,2);  
  lcd.print("Humidity: "); lcd.print(hmDt); lcd.print(" %");

  lcd.setCursor(0,3);  
  lcd.print("Water Level: ");  lcd.print(waterLevel2); lcd.print(" %");
  
//  Serial.print("temp:");
//  Serial.print(tmpDt);
//  Serial.print("  humi:");
//  
//  Serial.println(hmDt);
  
}

void setup()
{
  delay(5000);
 //Serial.begin(115200);
  Blynk.begin(auth, ssid, pass);
//    for(int i = 1; i<8; i++){
//    pinMode(i,OUTPUT);
//  }
//  Blynk.virtualWrite(ledStr, 0);
//  Blynk.virtualWrite(lightStr, 0);
//  Blynk.virtualWrite(fanStr, 0);
//  Blynk.virtualWrite(acStr, 0);
//  Blynk.virtualWrite(pmpStr, 0);
//  Blynk.virtualWrite(autoStr, 0);
  pinMode(ledPin,OUTPUT); digitalWrite(ledPin, 1);
  pinMode(lightPin,OUTPUT); digitalWrite(lightPin, 1);
  pinMode(fanPin,OUTPUT); digitalWrite(fanPin, 1);
  pinMode(acPin,OUTPUT); digitalWrite(acPin, 1);
  pinMode(pmpPin,OUTPUT); digitalWrite(pmpPin, 1);


  
  lcd.init();    // initialize the lcd   
  lcd.backlight(); // backlight ON  
  timer.setInterval(500L, myTimerEvent);
}

void loop()
{
  Blynk.run();
  timer.run();
}
