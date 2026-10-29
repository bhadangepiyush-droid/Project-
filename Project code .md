#include "WiFiS3.h" 
#include "arduino_secrets.h" 
#include "Adafruit_MQTT.h" 
#include "Adafruit_MQTT_Client.h" 
#include "MegunoLink.h" 
#include "ArduinoTimer.h" 
#include "SoftwareSerial.h" 
#include <TinyGPS++.h> 
const byte RxPin = 0; 
const byte TxPin = 1; 
 
SoftwareSerial GPSSerial(RxPin, TxPin); 
TinyGPSPlus GPSDecoder; 
ArduinoTimer ReportTimer; 
#define AIO_SERVER "io.adafruit.com" /*Initializing adafruit server*/ 
#define AIO_SERVERPORT 1883 /*Declaring adafruit portpumpSt1 number*/ 
#define AIO_USERNAME "Put Your Adafruit Username" /*Setting up with adafruit Account 
with Username*/ 
#define AIO_KEY "Put your Adafruit account Key" /*Connect Adafruit account*/ 
 
WiFiClient clients; /*CreateWi-FiClient Obeject named clients*/ 
Adafruit_MQTT_Client 
mqtt(&clients,AIO_SERVER,AIO_SERVERPORT,AIO_USERNAME,AIO_KEY); /*MQTT protocol*/ 
const char gps_FEED[] PROGMEM = AIO_USERNAME "/feeds/gpslatlng/csv"; 
//CSV=commasseperated values 
Adafruit_MQTT_Publish gpslatlng = Adafruit_MQTT_Publish(&mqtt,gps_FEED); 
void MQTT_connect(); /*Calling function MQTT_Connect()*/ 
#include <TFT_eSPI.h> // Hardware-specific library 
#include <TFT_eWidget.h> //widget library 
 
char ssid[]=SECRET_SSID; // your network SSDI(name) 
char pass[]=SECRET_PASS; // your network password (use for WPA, or use as key for WEP) 
 
//define wifi status i 
int status = WL_IDLE_STATUS; 
void setup() 
{ 
// init serial port 
Serial.begin(115200); 
GPSSerial.begin(115200); 
Serial.println(F("GPS Mapping")); 
Serial.println(F(" ----------- ")); 
// attempt to connect to WiFi network: 
while (status != WL_CONNECTED) 
{ 
Serial.print("Attempting to connect to WPA SSID: "); 
Serial.println(ssid); 
// connect to WPA/WPA2 network: 
 
status = WiFi.begin(ssid, pass); 
 
// wait 10 seconds for connection; 
delay(10000); 
} 
MQTT_connect(); 
delay(500); 
} 
void loop() 
{ 
char sendbuffer[30]; 
char *p = sendbuffer; 
 
// Feed the GPS decoder with data from the GPS device 
if (GPSSerial.available()) 
{ 
GPSDecoder.encode(GPSSerial.read()); 
} 
 
// Send location periodically. 
if (ReportTimer.TimePassed_Seconds(2)) 
{ 
if (GPSDecoder.location.isValid()) 
{ 
// add speed value 
dtostrf(GPSDecoder.speed.mph(), 2, 6, p); 
p +=strlen(p); 
p[0] = ','; p++; 
 
// concat latitude 
dtostrf(GPSDecoder.location.lat(), 2, 6, p); 
p +=strlen(p); 
p[0] = ','; p++; 
 
// concat longitude 
dtostrf(GPSDecoder.location.lng(), 3, 6, p); 
p +=strlen(p); 
p[0] = ','; p++; 
 
// concat altitude 
dtostrf(GPSDecoder.altitude.feet(), 2, 6, p); 
p +=strlen(p); 
// null terminate
p[0] = 0; 
 
gpslatlng.publish(sendbuffer); 
 
Serial.print("Sending: "); 
Serial.print(" \t"); 
Serial.print(GPSDecoder.location.lat(),6); 
Serial.print(" \t"); 
Serial.println(GPSDecoder.location.lng(),6); 
Serial.println(sendbuffer); 
Serial.println(" "); 
} 
else 
{ 
Serial.println(F("Location not available")); 
} 
} 
} 
 
void MQTT_connect() 
{ 
int8_t ret; 
/* Stop if already connected.*/ 
if (mqtt.connected()) 
{ 
return; 
} 
Serial.print("Connecting to MQTT... "); 
uint8_t retries = 3; 
while ((ret = mqtt.connect()) != 0) 
{ /* connect will return 0 for connected*/ 
Serial.println(mqtt.connectErrorString(ret)); 
Serial.println("Retrying MQTT connection in 5 seconds..."); 
mqtt.disconnect(); 
delay(5000); /* wait 5 seconds*/ 
retries--; 
if (retries == 0) 
{ 
while (1); 
} 
} 
Serial.println("MQTT Connected!"); 
}

