This github is about the project in ICS4U1-03, creating tempture sensor and send data to thingspeak

Code is
 
 
 /*
File-name:advance_program   for me
Class-code:ICS4U1
Used program:Arduino
Editor:Satoshi Muta
 */

/*
  Loading libraries 
  TSL2561-Light sensor
  Si7021-Humidity or temperature sensor
*/
#include "ThingSpeak.h"
#include <WiFi.h>
#include <Wire.h>
#include <SPI.h>
#include <Adafruit_Sensor.h>
#include <Adafruit_TSL2561_U.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include "Adafruit_Si7021.h"

float rhumidrecord; //Default setting for humidity record
float rtemprecord;  //Default setting for temperature record
float rlightrecord; //Default setting for light record
float rhumidchange = 5;   //notice in serial monitor if humidity changed dramatically as value is
float rtempchange = 5;    //notice in serial monitor if temperature changed dramatically as value is
float rlightchange = 400; //notice in serial monitor if light changed dramatically as value is
char ssid[] = "Miyauchi";   // your network SSID (name) 
char pass[] = "Smiya461";   // your network password
int keyIndex = 0;            // your network key Index number (needed only for WEP)
int icounted = 0;
int icountsenddata = 10;
unsigned long myChannelNumber = 787803; //your thingspeak number
const char * myWriteAPIKey = "FTDQMR0SR1K64XE6";//your thingspeak read api key
WiFiClient  client;

//Defining defalut settings
#define  idelay 1500 // 1000 for one second 
#define  idisplay 1 //  0:basic info from sensors   1:simple format   Other:horizontal bar graph





//Default setting for SSD1306 display
#define  SCREEN_WIDTH 128 
#define  SCREEN_HEIGHT 32 
#define  OLED_MOSI  32
#define  OLED_CLK   14
#define  OLED_DC    33
#define  OLED_CS    27
#define  OLED_RESET 15
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT,OLED_MOSI, OLED_CLK, OLED_DC, OLED_RESET, OLED_CS);
Adafruit_Si7021 tempsensor = Adafruit_Si7021();
Adafruit_TSL2561_Unified tsl = Adafruit_TSL2561_Unified(TSL2561_ADDR_FLOAT, 12345);

//Void for prepare of light sensor

void Configurelightsensor(void){
  tsl.enableAutoRange(true);
  tsl.setIntegrationTime(TSL2561_INTEGRATIONTIME_13MS); 
}




void setup(){ 
  sensors_event_t event;
  tsl.getEvent(&event); 
  #define rhumidity tempsensor.readHumidity()
  #define rtemperature tempsensor.readTemperature()
  #define rlight (int)event.light 


//Check sensor is connected
  if (!tempsensor.begin()) {
    Serial.println("Did not find Si7021 sensor!");
    while (true)
      ;
  }
  
  Serial.begin(115200);
  Serial.println("Setting up sensors...");
  Configurelightsensor();
  Serial.println("DONE!!");

  WiFi.mode(WIFI_STA);   
  ThingSpeak.begin(client);  // Initialize ThingSpeak
  

//Check SSD1306 work. 
  if(!display.begin(SSD1306_SWITCHCAPVCC)) {
    Serial.println(F("SSD1306 allocation failed"));
    for(;;); // Don't proceed, loop forever, check SSD1306 works
  }
  
  display.display();
  delay(2000); 
  display.clearDisplay();

//setting first record
  rhumidrecord = rhumidity;
  rtemprecord = rtemperature;
  rlightrecord = rlight;
}



void loop(){


//Connect or reconnect to WiFi
    if(WiFi.status() != WL_CONNECTED){
      Serial.print("Attempting to connect to SSID: ");
      Serial.println(ssid);
      while(WiFi.status() != WL_CONNECTED){
        WiFi.begin(ssid, pass); // Connect to WPA/WPA2 network. Change this line if using open or WEP network
        Serial.print(".");
        delay(5000);     
      } 
      Serial.println("\nConnected.");
    }

    sensors_event_t event;
    tsl.getEvent(&event);
    display.setTextSize(1);      
    display.setTextColor(WHITE); 
    Serial.print(rhumidity);Serial.print(" %    ");Serial.print(rtemperature);Serial.print(" C    ");Serial.print(rlight);Serial.print(" lux    ");Serial.println(icounted);

//Check sensors change dramatically or not

     if(rhumidity<=rhumidrecord-rhumidchange||rhumidity>=rhumidrecord+rhumidchange){
       Serial.println("----------------------------------------------------");
       Serial.print("Humidity sensed high/low!!  ");Serial.println(rhumidity-rhumidrecord);
       Serial.print("Humidity record ");Serial.print(rhumidrecord);Serial.print(" % is updated to ");Serial.print(rhumidity); Serial.println(" %");
       Serial.println("----------------------------------------------------");
       rhumidrecord = rhumidity;
     }
     
     if(rtemperature<=rtemprecord-rtempchange||rtemperature>=rtemprecord+rtempchange){
       Serial.println("----------------------------------------------------");
       Serial.print("Temperature sensed high/low!!  ");Serial.println(rtemperature-rtemprecord);
       Serial.print("Temperature record ");Serial.print(rtemprecord);Serial.print(" C is updated to ");Serial.print(rtemperature); Serial.println(" C");
       Serial.println("----------------------------------------------------");
       rtemprecord = rtemperature;
     }

     if(rlight<=rlightrecord-rlightchange||rlight>=rlightrecord+rlightchange){
       Serial.println("----------------------------------------------------");
       Serial.print("Light sensed high/low!!  ");Serial.println(rlight-rlightrecord);
       Serial.print("Light record ");Serial.print(rlightrecord);Serial.print(" % is updated to ");Serial.print(rlight); Serial.println(" %");
       Serial.println("----------------------------------------------------");
       rlightrecord = rlight;
     }




    
    if(idisplay==0){
      for (int i = 0; i <= 3; i++) {
        sensors_event_t event;
        tsl.getEvent(&event);
        display.clearDisplay();
        display.setCursor(0, 0); 
        display.print(rhumidity);    display.println(" %");
        display.print(rtemperature); display.println(" C");
        display.print(rlight);       display.println(" lux");
        display.fillRect( SCREEN_WIDTH-32, 0, SCREEN_WIDTH-2, SCREEN_HEIGHT-10*(3-i), WHITE);
        display.display(); 
      }
    }else if(idisplay==1){
        display.clearDisplay(); 
        display.setCursor(0, 0); 
        display.print("HUMIDITY    "); display.print(rhumidity);    display.println(" %");
        display.print("TEMPERATURE "); display.print(rtemperature); display.println(" C");
        display.print("LIGHT       "); display.print(rlight);       display.println(" lux");
        display.display();   
    }else{
        display.clearDisplay(); 
        display.fillRect( 0,                 0,    rhumidity, SCREEN_HEIGHT/4, WHITE);  display.setCursor(    rhumidity,                 0); display.print(rhumidity);     display.print("%");
        display.fillRect( 0,   SCREEN_HEIGHT/3, rtemperature, SCREEN_HEIGHT/4, WHITE);  display.setCursor( rtemperature,   SCREEN_HEIGHT/3); display.print(rtemperature);  display.print("C");
        display.fillRect( 0, 2*SCREEN_HEIGHT/3,    rlight/10, SCREEN_HEIGHT/4, WHITE);  display.setCursor(    rlight/10, 2*SCREEN_HEIGHT/3); display.print(rlight);        display.print("lux");
        display.display();   
    }
    
    delay(idelay);





    if(icounted==icountsenddata){
//set every field
      ThingSpeak.setField(1, (float)rhumidity);
      ThingSpeak.setField(2, (float)rtemperature);
      ThingSpeak.setField(3, (float)rlight);
//send every data to thingspeak
      int x = ThingSpeak.writeFields(myChannelNumber, myWriteAPIKey);
      if(x == 200){
        Serial.println("The data was successfully updata to thingspeak.");
      }
      else{
        Serial.println("Problem happen while updating. HTTP error code " + String(x));
      }
      icounted=0;
    }else{
      icounted++;
    }
}
