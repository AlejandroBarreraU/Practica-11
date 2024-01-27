# Practica 11
## Introducción
En esta práctica se mostrará paso a paso como instalar los materiales necesarios para crear una base de datos y hacer su conexión con WOKWI.
## Requisitos previos
1. Entrar a la pagina https://www.apachefriends.org/ .
2. Descargar XAMPP for Windows (8.2.4).
3. Se les decargara un archivo llamado xampp-windows-x64-8.2.4-0VS16-installer. 
4. Ejecutamos el archivo en modo administrador e instalamos.

## Materiales a utilizar
+ NODE-RED
+ XAMPP
+ WOKWI

## Instrucciones XAMPP
1. Debemos abrir el programa llamado XAMPP.
2. Dentro de la interfaz nos vamos a la fila llamada Mysql.
3. Le damos doble click al boton Admin.
4. Una vez en phpMyAdmin creamos una tabla con los sig. criterios

   ![](https://github.com/AlejandroBarreraU/Practica-11/blob/main/criterios%20de%20la%20tabla%20de%20mysql.png?raw=true
dashbord)

## Intrucciones en NODE-RED
1. Damos clic en las 3 lineas que estan en la parte superior del lado derecho.
2. Seleccionamos la opcion "Manage palette"
3. En el apartado de "Nodes" buscamos: node-red-node-mysql y damos clic en "install"
4. Agregamos el bloque de funcion MySQL
5. Agregamos un bloque de funcion y agregamos el siguiente código:

 ```
   var query = "INSERT INTO `tamulba7`(`ID`, `FECHA`, `DEVICE`, `TEMPERATURA`, `HUMEDAD`) VALUES (NULL, current_timestamp(), '";
query = query+msg.payload.DEVICE + "','";
query = query+msg.payload.TEMPERATURA + "','";
query = query+msg.payload.HUMEDAD + "');'";
msg.topic=query;
return msg;
  ``` 

   
   
NODE RED SE DEBE VISUALIZAR ASI

![](https://github.com/AlejandroBarreraU/Practica-11/blob/main/bloques%20de%20nodered.png?raw=true)

## Instrucciones en WOKWI

1. Escribimos el siguiente código:
 ```
#include <ArduinoJson.h>
#include <WiFi.h>
#include <PubSubClient.h>
#define BUILTIN_LED 2
#include "DHTesp.h"
const int DHT_PIN = 15;
DHTesp dhtSensor;
// Update these with values suitable for your network.

const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "18.193.219.109";
String username_mqtt="alejandro12";
String password_mqtt="12345678";

WiFiClient espClient;
PubSubClient client(espClient);
unsigned long lastMsg = 0;
#define MSG_BUFFER_SIZE  (50)
char msg[MSG_BUFFER_SIZE];
int value = 0;

void setup_wifi() {

 delay(10);
 // We start by connecting to a WiFi network
 Serial.println();
 Serial.print("Connecting to ");
 Serial.println(ssid);

 WiFi.mode(WIFI_STA);
 WiFi.begin(ssid, password);

 while (WiFi.status() != WL_CONNECTED) {
   delay(500);
   Serial.print(".");
 }

 randomSeed(micros());

 Serial.println("");
 Serial.println("WiFi connected");
 Serial.println("IP address: ");
 Serial.println(WiFi.localIP());
}

void callback(char* topic, byte* payload, unsigned int length) {
 Serial.print("Message arrived [");
 Serial.print(topic);
 Serial.print("] ");
 for (int i = 0; i < length; i++) {
   Serial.print((char)payload[i]);
 }
 Serial.println();

 // Switch on the LED if an 1 was received as first character
 if ((char)payload[0] == '1') {
   digitalWrite(BUILTIN_LED, LOW);   
   // Turn the LED on (Note that LOW is the voltage level
   // but actually the LED is on; this is because
   // it is active low on the ESP-01)
 } else {
   digitalWrite(BUILTIN_LED, HIGH);  
   // Turn the LED off by making the voltage HIGH
 }

}

void reconnect() {
 // Loop until we're reconnected
 while (!client.connected()) {
   Serial.print("Attempting MQTT connection...");
   // Create a random client ID
   String clientId = "ESP8266Client-";
   clientId += String(random(0xffff), HEX);
   // Attempt to connect
   if (client.connect(clientId.c_str(), username_mqtt.c_str() , password_mqtt.c_str())) {
     Serial.println("connected");
     // Once connected, publish an announcement...
     client.publish("outTopic", "hello world");
     // ... and resubscribe
     client.subscribe("inTopic");
   } else {
     Serial.print("failed, rc=");
     Serial.print(client.state());
     Serial.println(" try again in 5 seconds");
     // Wait 5 seconds before retrying
     delay(5000);
   }
 }
}

void setup() {
 pinMode(BUILTIN_LED, OUTPUT);     // Initialize the BUILTIN_LED pin as an output
 Serial.begin(115200);
 setup_wifi();
 client.setServer(mqtt_server, 1883);
 client.setCallback(callback);
 dhtSensor.setup(DHT_PIN, DHTesp::DHT22);
}

void loop() {


delay(1000);
TempAndHumidity  data = dhtSensor.getTempAndHumidity();
 if (!client.connected()) {
   reconnect();
 }
 client.loop();

 unsigned long now = millis();
 if (now - lastMsg > 2000) {
   lastMsg = now;
   //++value;
   //snprintf (msg, MSG_BUFFER_SIZE, "hello world #%ld", value);

   StaticJsonDocument<128> doc;

   doc["DEVICE"] = "ESP32";
   //doc["Anho"] = 2022;
   //doc["Empresa"] = "Educatronicos";
   doc["TEMPERATURA"] = String(data.temperature, 1);
   doc["HUMEDAD"] = String(data.humidity, 1);
  

   String output;
   
   serializeJson(doc, output);

   Serial.print("Publish message: ");
   Serial.println(output);
   Serial.println(output.c_str());
   client.publish("alejandro12", output.c_str());
 }
}

 ```
2. Descargar las sig librerias
+ArduinoJson
+DHT sensor library for ESPx
+PubSubClient
+MQTTPubSubClient

3. Realizar las sig. conexiones.


![](https://github.com/AlejandroBarreraU/Practica-11/blob/main/wowki.png?raw=true)




## Resultados


Resultados en NODE- RED



![](https://github.com/AlejandroBarreraU/Practica-11/blob/main/dasbord%20de%20nodered.png?raw=true)




Resultados en base de datos de MySQL
   



![](https://github.com/AlejandroBarreraU/Practica-11/blob/main/resultados%20bases%20de%20datos.png?raw=true) 
