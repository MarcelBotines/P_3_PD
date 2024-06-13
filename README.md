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

const char* ssid = "Xiaomi_11T_Pro"; 
const char* password = "12345678"; 
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
Xiaomi_11T_Pro
...........................
WiFi connected successfully
Got IP: 192.168.1.100
HTTP server started
````

### PRACTICA PARTE 2- COMUNICACIÓN BLUETOOTH CON MOVIL
El código en C++ muestra que establece una comunicación bluetooth entre la placa ESP32 y un dispositivo móvil. El ESP32 se nombra como "ESP32test" para su identificación por bluetooth.

```c++
#include "BluetoothSerial.h"
#if !defined(CONFIG_BT_ENABLED) || !defined(CONFIG_BLUEDROID_ENABLED)
#error Bluetooth is not enabled! Please run `make menuconfig` to and enable it
#endif
BluetoothSerial SerialBT;
void setup() {
Serial.begin(115200);
SerialBT.begin("ESP32test"); //Bluetooth device name
Serial.println("The device started, now you can pair it with bluetooth!");
}
void loop() {
if (Serial.available()) {
SerialBT.write(Serial.read());
}
if (SerialBT.available()) {
Serial.write(SerialBT.read());
}
delay(20);
}
```

#### Funcionamiento y salidas que se obtienen:

Este código permite establecer una comunicación bidireccional entre la placa ESP32 y un dispositivo móvil a través de bluetooth. Esto significa que ambos dispositivos pueden enviar y recibir datos entre sí.
Las salidas que se obtienen se visualizan a través del puerto serie, lo que permite monitorear los datos que se envían y reciben durante la comunicación entre los dos dispositivos. 
La comunicación puede ocurrir en dos direcciones diferentes: desde el ESP32 hacia el dispositivo móvil y desde el dispositivo móvil hacia el ESP32.

