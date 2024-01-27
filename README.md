# PROYECTO-FINAL-LLENADO-DE-BOTELLA

## Este repositorio muestra el llenado de botellas con sensor ultrasonico donde mostraremos paso a paso el modelo 

# Introducción 
## Descripción

La Esp32 la utilizamos en un entorno de adquision de datos, lo cual en esta practica ocuparemos un sensor (Ultrasonico) para medir la distancia de una jarra 
donde podra parar el llenado y conectaremos por medio a wifi el programa y poder visualizar la medicion.

# Material Necesario
## Para realizar esta practica necesitas lo siguiente

WOKWI

Tarjeta ESP 32

HC-SR04 Ultrasonic Distance Sensor

Protoboar

Fuente de 12 VDC

Bomba sumergible

Node red

Electro valvula

Boton 

# Instrucciones
## programamos el codigo en la plataforma de ardruino donde vinculamos por medio de la IP local para poder visualizar la distancia.
```
#include <ArduinoJson.h>
#include <WiFi.h>
#include <PubSubClient.h>
#define BUILTIN_LED 2

#define Pecho 12
#define Ptrig 13
#define Pbotton 14

const char* ssid = "ARRIS-6512";
const char* password = "7B27DE7C57047399";
const char* mqtt_server = "18.193.219.109";

String username_mqtt="proyectof";
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





float duracion, distancia;

void setup() {
  // put your setup code here, to run once:

  pinMode(BUILTIN_LED, OUTPUT);     // Initialize the BUILTIN_LED pin as an output
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);


  pinMode(Pecho, INPUT);
  pinMode(Ptrig, OUTPUT);
  pinMode(Pbotton, INPUT);
  pinMode(33, OUTPUT);
  pinMode(32, OUTPUT);
  Serial.begin(9600);
}

void loop() {



  digitalWrite(Ptrig, LOW);
  delayMicroseconds(2);
  digitalWrite(Ptrig,  HIGH);
  delayMicroseconds(10);
  digitalWrite (Ptrig, LOW);

  duracion= pulseIn (Pecho, HIGH);
  distancia = (duracion/2) / 29;

  Serial.print(distancia);
  Serial.print ("cm");
  Serial.println();
  delay(500);
  

if (digitalRead(Pbotton) == HIGH && distancia >= 19.00){
Serial.print ("Llenando vaso  ");
digitalWrite(33, LOW);
digitalWrite(32, LOW);
}
else {
Serial.print ("Vaso lleno  ");
digitalWrite(33, HIGH);
digitalWrite(32, HIGH);
}

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
    doc["DISTANCIA"] = String(distancia);
  
   

    String output;
    
    serializeJson(doc, output);

    Serial.print("Publish message: ");
    Serial.println(output);
    Serial.println(output.c_str());
    client.publish("proyectof", output.c_str());
  }




}


```
### AGREGAMOS LAS SIGIENTES LIBRERIAS
 ![image](https://github.com/ErickRomeroRamos/PROYECTO-FINAL-LLENADO-DE-BOTELLA/assets/153964793/07105146-cb58-4dce-bf95-fe09c391c57f)
## CONFIGURAMOS LOS BLOQUES DE IN. Y USAMOS JSON Y LA GRAFICA DE DISTANCIA
![image](https://github.com/ErickRomeroRamos/PROYECTO-FINAL-LLENADO-DE-BOTELLA/assets/153964793/c46d18cb-1271-4123-adb4-30025bcb5323)
### instruccion json
![image](https://github.com/ErickRomeroRamos/PROYECTO-FINAL-LLENADO-DE-BOTELLA/assets/153964793/9100688d-02c2-4e87-bbdf-fdc193b1b86e)
### intruccion servidor mqqt
![image](https://github.com/ErickRomeroRamos/PROYECTO-FINAL-LLENADO-DE-BOTELLA/assets/153964793/b2341c00-a47d-4a91-a1ac-a8d0e31ca941)
## EVIDENCIA NODE RED
![image](https://github.com/ErickRomeroRamos/PROYECTO-FINAL-LLENADO-DE-BOTELLA/assets/153964793/69907875-dc5a-4650-9d5d-ef6cf2e7568d)
# INSTALACION MECANICA Y ELECTRICA DEL LLENADO DE BOTELLLAS AUTOMATICO
## Instrucciones:

### Se realiza la conexión ESP32 al puerto USB del ordenador

### En arduino, en el apartado de herramienta elegimos la siguiente 

## Conexiones físicas
![image](https://github.com/ErickRomeroRamos/PROYECTO-FINAL-LLENADO-DE-BOTELLA/assets/153964793/2559436e-6b34-4cfa-9cf5-1be922cf9eb1)
![image](https://github.com/ErickRomeroRamos/PROYECTO-FINAL-LLENADO-DE-BOTELLA/assets/153964793/3952fd47-9e0f-4ccc-ba1a-ff49a2512d4b)
![image](https://github.com/ErickRomeroRamos/PROYECTO-FINAL-LLENADO-DE-BOTELLA/assets/153964793/72c7598e-e35a-4786-83af-07961a8292d6)
![image](https://github.com/ErickRomeroRamos/PROYECTO-FINAL-LLENADO-DE-BOTELLA/assets/153964793/b9c0b690-b165-482b-8b2d-5eb45ac19ab3)










