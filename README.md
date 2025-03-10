# **Practica3**
## Introducción
La práctica se centra en el uso de wifi y bluetooth, utilizando el microprocesador ESP32. Se generará un servidor web desde la placa y se establecerá una comunicación serie con un dispositivo móvil a través de bluetooth.

### PRACTICA PARTE 1- PÁGINA WEB
Se incluye un fragmento de código en lenguaje C++ que configura la placa ESP32 para actuar como servidor web y generar una página HTML.
En la función setup(), se inicializa la comunicación serial, se conecta a la red Wi-Fi y se inicia el servidor web. Mientras que en loop(), se manejan las solicitudes de los clientes al servidor web.
El código utiliza las librerías WiFi.h y WebServer.h para lograr esto.
```c++
#include <WiFi.h>
#include <WebServer.h>
#include <Arduino.h>

const char* ssid = "Nautilus"; 
const char* password = "20000Leguas"; 
WebServer server(80);

void handle_root();

// Object of WebServer(HTTP port, 80 is defult)
void setup() {
Serial.begin(115200);
Serial.println("Try Connecting to ");
Serial.println(ssid);
// Connect to your wi-fi modem
WiFi.begin(ssid, password);
// Check wi-fi is connected to wi-fi network
while (WiFi.status() != WL_CONNECTED) {
delay(1000);
Serial.print(".");
}
Serial.println("");
Serial.println("WiFi connected successfully");
Serial.print("Got IP: ");
Serial.println(WiFi.localIP()); //Show ESP32 IP on serial
server.on("/", handle_root);
server.begin();
Serial.println("HTTP server started");
delay(100);
}
void loop() {
server.handleClient();
}
```

#### HTML:
Además, se proporciona el código HTML para la página web generada.
```
String HTML = "<!DOCTYPE html>\
<html>\
<body>\
<h1> Pagina Web creada , Practica 3 - Wifi &#128522;</h1>\
</body>\
</html>";

// Handle root url (/)
void handle_root() {
 server.send(200, "text/html", HTML);
}
```


#### Funcionamiento y salida por terminal:

Este código se encarga de convertir la placa ESP32 en un servidor web que genera una página HTML. Cuando accedes a la dirección IP de la placa desde un navegador, puedes ver esta página web.
- En la función setup(), se prepara la placa para actuar como servidor. Se establece la comunicación serial, se conecta a una red Wi-Fi y se inicia el servidor web.
- En la función loop(), se gestiona la interacción con los clientes que acceden al servidor web. Esto significa que se atienden las solicitudes que los clientes hacen al servidor.
Para lograr esto, se utilizan las librerías WiFi.h y WebServer.h, las cuales proporcionan las herramientas necesarias para configurar la comunicación Wi-Fi y establecer el servidor web, respectivamente.

Ejemplo de salida por terminal que indica la conexión exitosa a la red Wi-Fi y la dirección IP asignada al ESP32:
```
Try Connecting to 
Nautilus
...........................
WiFi connected successfully
Got IP: 192.168.1.100
HTTP server started
````

### PRACTICA PARTE 2- COMUNICACIÓN BLUETOOTH CON MOVIL
El código en C++ muestra que establece una comunicación bluetooth entre la placa ESP32 y un dispositivo móvil. El ESP32 se nombra como "ESP32test" para su identificación por bluetooth.

```c++
#include <Arduino.h>
#include <BLEDevice.h>
#include <BLEServer.h>
#include <BLEUtils.h>
#include <BLE2902.h>

// Define UUIDs for the BLE service and characteristic
#define SERVICE_UUID        "4fafc201-1fb5-459e-8fcc-c5c9c331914b"
#define CHARACTERISTIC_UUID "beb5483e-36e1-4688-b7f5-ea07361b26a8"

// Create pointers for BLE objects
BLEServer* pServer = NULL;
BLECharacteristic* pCharacteristic = NULL;
bool deviceConnected = false;
bool oldDeviceConnected = false;

// Variables for sensor data (example)
float temperature = 0;
unsigned long previousMillis = 0;
const long interval = 2000;  // Update interval in milliseconds

// Callback class for handling server events
class MyServerCallbacks: public BLEServerCallbacks {
    void onConnect(BLEServer* pServer) {
      deviceConnected = true;
      Serial.println("Device connected");
    }

    void onDisconnect(BLEServer* pServer) {
      deviceConnected = false;
      Serial.println("Device disconnected");
    }
};

// Callback for handling characteristic events
class CharacteristicCallbacks: public BLECharacteristicCallbacks {
    void onWrite(BLECharacteristic *pCharacteristic) {
      std::string value = pCharacteristic->getValue();
      
      if (value.length() > 0) {
        Serial.println("Received data: ");
        for (int i = 0; i < value.length(); i++) {
          Serial.print(value[i]);
        }
        Serial.println();
      }
    }
};

void setup() {
  Serial.begin(115200);
  Serial.println("Starting BLE application");

  // Initialize BLE device
  BLEDevice::init("ESP32-S3 BLE Device");
  
  // Create the BLE Server
  pServer = BLEDevice::createServer();
  pServer->setCallbacks(new MyServerCallbacks());

  // Create the BLE Service
  BLEService *pService = pServer->createService(SERVICE_UUID);

  // Create a BLE Characteristic
  pCharacteristic = pService->createCharacteristic(
                      CHARACTERISTIC_UUID,
                      BLECharacteristic::PROPERTY_READ   |
                      BLECharacteristic::PROPERTY_WRITE  |
                      BLECharacteristic::PROPERTY_NOTIFY |
                      BLECharacteristic::PROPERTY_INDICATE
                    );

  // Add the descriptor for notifications
  pCharacteristic->addDescriptor(new BLE2902());
  
  // Set callback to handle write events
  pCharacteristic->setCallbacks(new CharacteristicCallbacks());

  // Start the service
  pService->start();

  // Start advertising
  BLEAdvertising *pAdvertising = BLEDevice::getAdvertising();
  pAdvertising->addServiceUUID(SERVICE_UUID);
  pAdvertising->setScanResponse(true);
  pAdvertising->setMinPreferred(0x06);  // helps with iPhone connections issue
  pAdvertising->setMinPreferred(0x12);
  BLEDevice::startAdvertising();
  
  Serial.println("BLE service started. Waiting for a client connection...");
}

void loop() {
  unsigned long currentMillis = millis();
  
  // Simulate sensor reading and update value periodically
  if (currentMillis - previousMillis >= interval) {
    previousMillis = currentMillis;
    
    // Simulate temperature change
    temperature = random(2000, 3000) / 100.0;
    
    // If a device is connected, update the characteristic value
    if (deviceConnected) {
      String tempStr = String(temperature, 2);
      pCharacteristic->setValue(tempStr.c_str());
      pCharacteristic->notify();
      Serial.print("Temperature: ");
      Serial.println(tempStr);
    }
  }

  // Handle connection events
  if (!deviceConnected && oldDeviceConnected) {
    delay(500); // Give the Bluetooth stack time to get ready
    pServer->startAdvertising(); // Restart advertising
    Serial.println("Started advertising again");
    oldDeviceConnected = deviceConnected;
  }
  
  // If we've connected to a device
  if (deviceConnected && !oldDeviceConnected) {
    oldDeviceConnected = deviceConnected;
  }
  
  delay(10); // Small delay for stability
}

```

#### Funcionamiento y salidas que se obtienen:

Este código permite establecer una comunicación bidireccional entre la placa ESP32 y un dispositivo móvil a través de bluetooth. Esto significa que ambos dispositivos pueden enviar y recibir datos entre sí.
Las salidas que se obtienen se visualizan a través del puerto serie, lo que permite monitorear los datos que se envían y reciben durante la comunicación entre los dos dispositivos. 
La comunicación puede ocurrir en dos direcciones diferentes: desde el ESP32 hacia el dispositivo móvil y desde el dispositivo móvil hacia el ESP32.

