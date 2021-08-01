#include <Wire.h>
#include "MAX30100_PulseOximeter.h"
#include <SoftwareSerial.h>
#include <OneWire.h>
#include <DallasTemperature.h>

#define RX 3
#define TX 4

#define I2C_SDA 4
#define I2C_SCL 5

#define REPORTING_PERIOD_MS     1500
 
PulseOximeter pox;

uint32_t tsLastReport = 0;

#define ONE_WIRE_BUS 8

OneWire oneWire(ONE_WIRE_BUS);

DallasTemperature sensors(&oneWire);

float Celsius = 0;
float Fahrenheit = 0;
float HR = 1;
float spo2 = 2;

String AP = "Only If Required";       // AP NAME
String PASS = "simmi2424"; // AP PASSWORD
String API = "IGSOBBVJ00M98LIU";   // Write API KEY
String HOST = "api.thingspeak.com";
String PORT = "80";
int countTrueCommand;
int countTimeCommand; 
boolean found = false; 
int valSensor = 1;
  
SoftwareSerial esp8266(RX,TX); 

 
void onBeatDetected()
{
    Serial.println("Beat!");
}

void setup() {
  Serial.begin(9600);
  esp8266.begin(115200);
  sendCommand("AT",5,"OK");
  sendCommand("AT+CWMODE=1",5,"OK");
  sendCommand("AT+CWJAP=\""+ AP +"\",\""+ PASS +"\"",20,"OK");
  
  Serial.print("Initializing pulse oximeter..");
    if (!pox.begin()) {
        Serial.println("FAILED");
        for(;;);
    } else {
        Serial.println("SUCCESS");
    }
   // pox.setIRLedCurrent(MAX30100_LED_CURR_7_6MA);
 
    // Register a callback for the beat detection
    pox.setOnBeatDetectedCallback(onBeatDetected);
}

void loop() {

 // Make sure to call update as fast as possible
 pox.update();
 HR = pox.getHeartRate();
 if (millis() - tsLastReport > REPORTING_PERIOD_MS) {
        Serial.print("Heart rate:");
        Serial.print(pox.getHeartRate());
        Serial.print("bpm / SpO2:");
        Serial.print(pox.getSpO2());
        Serial.println("%");
        
        tsLastReport = millis();
  }
   
 String getData = "GET /update?api_key="+ API +"&field1="+getTemperatureValue()+"&field2="+getHR()+"&field3="+getspo2();
 sendCommand("AT+CIPMUX=1",5,"OK");
 sendCommand("AT+CIPSTART=0,\"TCP\",\""+ HOST +"\","+ PORT,15,"OK");
 sendCommand("AT+CIPSEND=0," +String(getData.length()+4),4,">");
 esp8266.println(getData);delay(1500);countTrueCommand++;
 sendCommand("AT+CIPCLOSE=0",5,"OK");
}


String getTemperatureValue(){

  sensors.requestTemperatures();
  Celsius = sensors.getTempCByIndex(0);
  Fahrenheit = sensors.toFahrenheit(Celsius);
  Serial.println(Fahrenheit); 
  Serial.println("F");
  return String(Fahrenheit);

}
String getHR(){
  HR = pox.getHeartRate();
  Serial.print("Heart rate:");
  Serial.print(pox.getHeartRate());
  Serial.print("bpm");
  tsLastReport = millis();
  return String(HR);

}
String getspo2(){
  spo2 = pox.getSpO2();
  Serial.print("Spo2:");
  Serial.print(pox.getSpO2());
  Serial.println("%");
  
  tsLastReport = millis();
  return String(spo2);

}
void sendCommand(String command, int maxTime, char readReplay[]) {
  Serial.print(countTrueCommand);
  Serial.print(". at command => ");
  Serial.print(command);
  Serial.print(" ");
  while(countTimeCommand < (maxTime*1))
  {
    esp8266.println(command);//at+cipsend
    if(esp8266.find(readReplay))//ok
    {
      found = true;
      break;
    }
  
    countTimeCommand++;
  }
  
  if(found == true)
  {
    Serial.println("OYI");
    countTrueCommand++;
    countTimeCommand = 0;
  }
  
  if(found == false)
  {
    Serial.println("Fail");
    countTrueCommand = 0;
    countTimeCommand = 0;
  }
  
  found = false;
 }
