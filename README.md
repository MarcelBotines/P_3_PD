# **Práctica 3**  

## Introducción  
En esta práctica, se explora el uso de Wi-Fi y Bluetooth utilizando el microprocesador ESP32. Se configurará la placa para actuar como un servidor web y se establecerá una comunicación serie con un dispositivo móvil mediante Bluetooth.  

## **Parte 1: Creación de una página web**  
Se implementa un fragmento de código en C++ para configurar el ESP32 como servidor web, permitiéndole generar y alojar una página HTML.  

En la función `setup()`, se inicializa la comunicación serial, se conecta la placa a una red Wi-Fi y se activa el servidor web. Mientras tanto, en `loop()`, se gestionan las solicitudes de los clientes que acceden a la página.  

Para lograr esto, se utilizan las librerías `WiFi.h` y `WebServer.h`.  

### **Código C++**  
```c++
#include <WiFi.h>
#include <WebServer.h>
#include <Arduino.h>

const char* ssid = "Nautilus";  
const char* password = "20000Leguas";  
WebServer server(80);

void handle_root();

void setup() {
    Serial.begin(115200);
    Serial.println("Conectando a la red Wi-Fi...");
    WiFi.begin(ssid, password);

    while (WiFi.status() != WL_CONNECTED) {
        delay(1000);
        Serial.print(".");
    }

    Serial.println("\nConexión establecida");
    Serial.print("Dirección IP: ");
    Serial.println(WiFi.localIP());

    server.on("/", handle_root);
    server.begin();
    Serial.println("Servidor HTTP iniciado");
}

void loop() {
    server.handleClient();
}
```

### **Código HTML**  
```html
String HTML = "<!DOCTYPE html>\
<html>\
<body>\
<h1>Página web creada - Práctica 3 (Wi-Fi) &#128522;</h1>\
</body>\
</html>";

void handle_root() {
    server.send(200, "text/html", HTML);
}
```

### **Funcionamiento y salida esperada**  
Este código convierte la placa ESP32 en un servidor web capaz de alojar una página HTML. Al acceder a la dirección IP del ESP32 desde un navegador, se mostrará la página creada.  

En la consola serie, se visualizarán mensajes indicando la conexión exitosa y la dirección IP asignada. Ejemplo:  

```
Conectando a la red Wi-Fi...
...........................
Conexión establecida
Dirección IP: 192.168.1.100
Servidor HTTP iniciado
```

---

## **Parte 2: Comunicación Bluetooth con un móvil**  
En esta sección, se configura el ESP32 para comunicarse con un dispositivo móvil mediante Bluetooth. La placa se identificará como "ESP32test" y permitirá el intercambio bidireccional de datos.  

### **Código C++**  
```c++
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

### **Funcionamiento y salida esperada**  
Este código permite la comunicación bidireccional entre la placa ESP32 y un dispositivo móvil mediante Bluetooth. Desde el monitor serie, se puede observar el estado de conexión y los datos transmitidos.  
