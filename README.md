# P3

### Objetivo:
El objetivo de la practica es comprender el funcionamiento de WIFI Y BT.

### Materiales:
ESP32
Serial Bluetooth Terminal
### Programa:
  ```cpp
  /*
   ESP32 Web Server - STA Mode
   modified on 25 MAy 2019
   by Mohammadreza Akbari @ Electropeak
   https://electropeak.com/learn
  */


    #include <Arduino.h>
    #include <WiFi.h>
    #include <WebServer.h>

    void handle_root();

    // SSID & Password
    const char* ssid = "Nombre"; // Debe introducir el nombre de su SSID.
    const char* password = "Contraseña"; // Debe introducir la contraseña.

    WebServer server(80); // Objeto de la libreria WebServer, con puerto HTTP 80 por defecto

    void setup(){
        Serial.begin(115200);
        Serial.println("Conectando a la red WiFi...");
        Serial.println(ssid);
        WiFi.begin(ssid, password);
        while (WiFi.status() != WL_CONNECTED) {
            delay(1000);
            Serial.print(".");
        }
        Serial.println("");
        Serial.println("Conexión WiFi establecida");
        Serial.print("Dirección IP: ");
        Serial.println(WiFi.localIP());

        server.on("/", handle_root);
        server.begin();
        Serial.println("HTTP server started");
        delay(100);
    }

    void loop() {
        server.handleClient();
    }

    // HTML & CSS contents which display on web server
    // Crecion contenido de la pagina web de prueba, básica: 
    String HTML = "<!DOCTYPE html>\
    <html>\
    <body>\
    <h1>My Primera Pagina con ESP32 - Station Mode &#128522;</h1>\
    </body>\
    </html>";
    // Handle root url (/)

    void handle_root() {
        server.send(200, "text/html", HTML);
    }
```

### Salida:

```
    Conectando a la red WiFi...
    Conexión WiFi establecida
    Dirección IP: 192.168.1.43
    My Primera Pagina con ESP32
```

### Descripción:
Este código establece un servidor web simple en un ESP32 en modo de estación, que responde con una página HTML básica cuando se accede a la raíz del servidor.

### Código para la página web:
    
    #include <WiFi.h>
    #include <SPIFFS.h>
    #include <AsyncTCP.h>
    #include <WebServer.h>
    #include <Arduino.h>

    const char* ssid = "WebMBR";
    const char* password = "CL4V3D3F4";


    WebServer server;

    String webContent; 

    void handle_root() {
    server.send(200, "text/html", webContent);
    }

    String loadFile(String fileName) { 
    File file = SPIFFS.open(fileName);
    String content = "";

    if (!file) {
        Serial.println("Error al abrir el archivo");
        return content;
    }

    while (file.available()) {
        char c = file.read();
        content += c;
    }

    file.close();
    return content;
    }

    void setup() {
    Serial.begin(115200);

    if (!SPIFFS.begin(true)) {
        Serial.println("Error al montar SPIFFS");
        return;
    }

    webContent = loadFile("/PaginaWeb.html");
    delay(1000);

    Serial.println("Conectando a la red WiFi...");
    Serial.println(ssid);
    WiFi.begin(ssid, password);
    while (WiFi.status() != WL_CONNECTED) {
        delay(1000);
        Serial.print(".");
    }
    Serial.println("");
    Serial.println("Conexión WiFi establecida");
    Serial.print("Dirección IP: ");
    Serial.println(WiFi.localIP());

    server.begin();
    Serial.println("Servidor HTTP iniciado");
    
    server.on("/", handle_root);
    }

    void loop() {
    server.handleClient();
    }

### Página web sencilla
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Página Web</title>
</head>
<body>
    <h1>Bienvenido a mi Página Web</h1>
    <p>Aquí tienes un número aleatorio: <span id="randomNumber"></span></p>

    <script>
        // Generar un número aleatorio y mostrarlo en la página
        document.getElementById("randomNumber").textContent = Math.floor(Math.random() * 100);
    </script>
</body>
</html>
```
### Salida:
```
    Conectando a la red WiFi...
    Conexión WiFi establecida
    Dirección IP: 192.168.1.43
    Bienvenido a mi Página Web
    Aquí tienes un número aleatorio 6
```

# Bluetooth

### Código:
```cpp
#include <Arduino.h>
#include <BLEDevice.h>
#include <BLEServer.h>
#include <BLEUtils.h>
#include <BLE2902.h>

#define SERVICE_UUID        "4fafc201-1fb5-459e-8fcc-c5c9c331914b"
#define CHARACTERISTIC_UUID "beb5483e-36e1-4688-b7f5-ea07361b26a8"

BLEServer* pServer = NULL;
BLECharacteristic* pCharacteristic = NULL;
bool deviceConnected = false;
bool oldDeviceConnected = false;

float temperature = 0;
unsigned long previousMillis = 0;
const long interval = 2000;

class MyServerCallbacks: public BLEServerCallbacks {
    void onConnect(BLEServer* pServer) {
        deviceConnected = true;
        Serial.println("Dispositivo conectado");
    }

    void onDisconnect(BLEServer* pServer) {
        deviceConnected = false;
        Serial.println("Dispositivo desconectado");
    }
};

class CharacteristicCallbacks: public BLECharacteristicCallbacks {
    void onWrite(BLECharacteristic *pCharacteristic) {
        std::string value = pCharacteristic->getValue();
        
        if (value.length() > 0) {
            Serial.println("Datos recibidos: ");
            for (int i = 0; i < value.length(); i++) {
                Serial.print(value[i]);
            }
            Serial.println();
        }
    }
};

void setup() {
    Serial.begin(115200);
    Serial.println("Iniciando Bluetooth Low Energy...");

    BLEDevice::init("ESP32test");
    pServer = BLEDevice::createServer();
    pServer->setCallbacks(new MyServerCallbacks());

    BLEService *pService = pServer->createService(SERVICE_UUID);
    pCharacteristic = pService->createCharacteristic(
                        CHARACTERISTIC_UUID,
                        BLECharacteristic::PROPERTY_READ   |
                        BLECharacteristic::PROPERTY_WRITE  |
                        BLECharacteristic::PROPERTY_NOTIFY |
                        BLECharacteristic::PROPERTY_INDICATE
                    );

    pCharacteristic->addDescriptor(new BLE2902());
    pCharacteristic->setCallbacks(new CharacteristicCallbacks());
    pService->start();

    BLEAdvertising *pAdvertising = BLEDevice::getAdvertising();
    pAdvertising->addServiceUUID(SERVICE_UUID);
    pAdvertising->setScanResponse(true);
    pAdvertising->setMinPreferred(0x06);
    pAdvertising->setMinPreferred(0x12);
    BLEDevice::startAdvertising();

    Serial.println("Servicio BLE iniciado. Esperando conexión...");
}

void loop() {
    unsigned long currentMillis = millis();
    
    if (currentMillis - previousMillis >= interval) {
        previousMillis = currentMillis;
        temperature = random(2000, 3000) / 100.0;

        if (deviceConnected) {
            String tempStr = String(temperature, 2);
            pCharacteristic->setValue(tempStr.c_str());
            pCharacteristic->notify();
            Serial.print("Temperatura: ");
            Serial.println(tempStr);
        }
    }

    if (!deviceConnected && oldDeviceConnected) {
        delay(500);
        pServer->startAdvertising();
        Serial.println("Reiniciando publicidad Bluetooth...");
        oldDeviceConnected = deviceConnected;
    }
    
    if (deviceConnected && !oldDeviceConnected) {
        oldDeviceConnected = deviceConnected;
    }
    
    delay(10);
}
```


### Descripción:
Este programa crea una conexion bluetooth en la placa la cual uno se puede connectar. Una vez connectado sale la salida puesta anteriormente, si sale un error saldrá en la salida la frase "Bluetooth is not enabled! Please run make menuconfig to and enable it".
