# AplicacionesIoT_Unidad1
Instrumento de evaluación de la Unidad I de la materia Aplicaciones de IoT

- **Nombre:**
- Miguel Angel Alvarez Ibarra
- **Número de Control:**
- 1223100379
- **Carrera:**
- Desarrollo de Software Multiplataforma
- **Institución:**
- Universidad Tecnológica del Norte de Guanajuato
- **Asignatura:**
- Aplicaciones de IoT
-  **Unidad:**
- I. Adquisición y Procesamiento de Datos

## Link a carpeta con vídeos de demostración.
-- [Carpeta Completa](https://drive.google.com/drive/folders/1xw73QppfqjuF_hzutWYUCsdSybzAyCbZ?usp=sharing)

## Actividades en pareja.
- [Actividad 1 y 2 parejas](https://drive.google.com/file/d/1AY2FTR8uPCJS0bn5l2eF9jXhlDHp-9AZ/view?usp=drive_link)
### Diagrama de Conexión:

```json
  [
    {
        "id": "567153203a11f536",
        "type": "tab",
        "label": "Conexión_Sensores",
        "disabled": false,
        "info": "",
        "env": []
    },
    {
        "id": "ac87398af7c4a476",
        "type": "mqtt in",
        "z": "567153203a11f536",
        "name": "",
        "topic": "utng/sensors",
        "qos": "2",
        "datatype": "auto-detect",
        "broker": "0d16543b2ffdbac5",
        "nl": false,
        "rap": true,
        "rh": 0,
        "inputs": 0,
        "x": 290,
        "y": 200,
        "wires": [
            [
                "1a7eea378e93a6c8"
            ]
        ]
    },
    {
        "id": "1a7eea378e93a6c8",
        "type": "postgresql",
        "z": "567153203a11f536",
        "name": "",
        "query": "INSERT INTO sensor_details (sensor_id, user_id, value) VALUES (1,1,'{{{msg.payload}}}');",
        "postgreSQLConfig": "ef157743cd3ef5d6",
        "split": false,
        "rowsPerMsg": 1,
        "outputs": 1,
        "x": 550,
        "y": 200,
        "wires": [
            [
                "0474a167f419d54e"
            ]
        ]
    },
    {
        "id": "0474a167f419d54e",
        "type": "debug",
        "z": "567153203a11f536",
        "name": "debug 6",
        "active": true,
        "tosidebar": true,
        "console": false,
        "tostatus": false,
        "complete": "false",
        "statusVal": "",
        "statusType": "auto",
        "x": 760,
        "y": 200,
        "wires": []
    },
    {
        "id": "0d16543b2ffdbac5",
        "type": "mqtt-broker",
        "name": "",
        "broker": "broker.emqx.io",
        "port": 1883,
        "clientid": "",
        "autoConnect": true,
        "usetls": false,
        "protocolVersion": 4,
        "keepalive": 60,
        "cleansession": true,
        "autoUnsubscribe": true,
        "birthTopic": "",
        "birthQos": "0",
        "birthRetain": "false",
        "birthPayload": "",
        "birthMsg": {},
        "closeTopic": "",
        "closeQos": "0",
        "closeRetain": "false",
        "closePayload": "",
        "closeMsg": {},
        "willTopic": "",
        "willQos": "0",
        "willRetain": "false",
        "willPayload": "",
        "willMsg": {},
        "userProps": "",
        "sessionExpiry": ""
    },
    {
        "id": "ef157743cd3ef5d6",
        "type": "postgreSQLConfig",
        "name": "aiot",
        "host": "127.0.0.1",
        "hostFieldType": "str",
        "port": 5432,
        "portFieldType": "num",
        "database": "aiot",
        "databaseFieldType": "str",
        "ssl": "false",
        "sslFieldType": "bool",
        "applicationName": "",
        "applicationNameType": "str",
        "max": 10,
        "maxFieldType": "num",
        "idle": 1000,
        "idleFieldType": "num",
        "connectionTimeout": 10000,
        "connectionTimeoutFieldType": "num",
        "user": "utng",
        "userFieldType": "str",
        "password": "1234",
        "passwordFieldType": "str"
    }

]
```

### Código Documentado
```python
# Importamos las bibliotecas necesarias para la conexión a la red y MQTT
import network
from umqtt.simple import MQTTClient  # Protocolo MQTT para comunicación con el broker

# Importamos módulos para controlar hardware
from machine import Pin  # Manejo de pines GPIO
from time import sleep  # Pausas en la ejecución
from hcsr04 import HCSR04  # Control del sensor de ultrasonido

# Configuración del servidor MQTT
MQTT_BROKER = "broker.emqx.io"  # Dirección del broker
MQTT_USER = ""  # Usuario (vacío si no se requiere autenticación)
MQTT_PASSWORD = ""  # Contraseña (vacío si no se requiere autenticación)
MQTT_CLIENT_ID = ""  # Identificador único del cliente
MQTT_TOPIC = "utng/sensor"  # Tema donde se enviarán los datos
MQTT_PORT = 1883  # Puerto estándar de MQTT

# Inicializamos el sensor de distancia con los pines correspondientes
sensor = HCSR04(trigger_pin=16, echo_pin=4, echo_timeout_us=24000)

# Configuración de pines para los LEDs (simulando un semáforo)
led_rojo = Pin(2, Pin.OUT)  # LED rojo
led_amarillo = Pin(5, Pin.OUT)  # LED amarillo
led_verde = Pin(18, Pin.OUT)  # LED verde

# Aseguramos que todos los LEDs inicien apagados
led_rojo.value(0)
led_amarillo.value(0)
led_verde.value(0)

# Función para conectar a una red WiFi
def conectar_wifi():
    print("Conectando a WiFi...", end="")
    sta_if = network.WLAN(network.STA_IF)  # Creamos un objeto para manejar la conexión WiFi
    sta_if.active(True)  # Activamos la interfaz WiFi
    sta_if.connect('Red-Peter', '12345678')  # Nos conectamos con la red WiFi
    while not sta_if.isconnected():  # Esperamos a que la conexión sea exitosa
        print(".", end="")
        sleep(0.3)
    print("¡WiFi conectado!")

# Función para suscribirse a un broker MQTT y recibir mensajes

def subscribir():
    client = MQTTClient(MQTT_CLIENT_ID,
                        MQTT_BROKER, port=MQTT_PORT,
                        user=MQTT_USER,
                        password=MQTT_PASSWORD,
                        keepalive=0)
    client.set_callback(llegada_mensaje)  # Definimos qué hacer cuando llega un mensaje
    client.connect()  # Nos conectamos al broker
    client.subscribe(MQTT_TOPIC)  # Nos suscribimos al tema MQTT
    print("Conectado a %s, en el tópico %s" % (MQTT_BROKER, MQTT_TOPIC))
    return client

# Función que se ejecuta cuando llega un mensaje al tópico MQTT

def llegada_mensaje(topic, msg):
    print("Mensaje recibido:", msg)

# Función que enciende los LEDs dependiendo de la distancia detectada

def controlar_leds(distancia):
    # Apagamos todos los LEDs antes de encender el necesario
    led_rojo.value(0)
    led_amarillo.value(0)
    led_verde.value(0)

    # Encendemos el LED correspondiente según la distancia
    if distancia < 10:
        led_rojo.value(1)  # Distancia peligrosa (Rojo encendido)
    elif 10 <= distancia < 20:
        led_amarillo.value(1)  # Distancia moderada (Amarillo encendido)
    else:
        led_verde.value(1)  # Distancia segura (Verde encendido)

# Conectamos a WiFi antes de iniciar la comunicación MQTT
conectar_wifi()

# Nos suscribimos al broker MQTT para enviar y recibir datos
client = subscribir()

# Variable para evitar enviar datos repetidos si la distancia no cambia
distancia_anterior = 0

# Bucle principal del programa
while True:
    client.check_msg()  # Revisamos si hay mensajes nuevos en el tema MQTT
    distancia = int(sensor.distance_cm())  # Leemos la distancia en centímetros
    if distancia != distancia_anterior:  # Solo enviamos datos si la distancia cambia
        print(f"La distancia es {distancia} cm.")
        client.publish(MQTT_TOPIC, str(distancia))  # Enviamos la distancia al broker MQTT
        controlar_leds(distancia)  # Actualizamos los LEDs según la distancia detectada
    distancia_anterior = distancia  # Guardamos la última distancia medida
    sleep(2)  # Esperamos 2 segundos antes de la siguiente lectura
```

## Actividades individuales.
- [CRUD en PostgreSQL](https://drive.google.com/file/d/1PkWyeWwjPNzwnqLOt63gcUZfbxzAmDb1/view?usp=drive_link)
- [Instalación y Configuraciones Básicas](https://drive.google.com/file/d/16CI9v2FEhKU4_qPN3p96XlsLyfAJSfV5/view?usp=drive_link)
- [LED y Botón con Raspberry Pi](https://drive.google.com/file/d/1LDSNreH223prNx8Q7Yzjva9aW6vR8lIO/view?usp=drive_link)
- [LED con Raspberry Pi](https://drive.google.com/file/d/1k8_ygdkSOHWrOXoa1LoVQC-LuSm0Ei4v/view?usp=drive_link)
- [Conexión MQTT en Node-RED](https://drive.google.com/file/d/1P6hpbPVd6mowNhcBRhz3o0vhynZU9-ww/view?usp=drive_link)

## Placa Fenólica
| Imagen 1 | Imagen 2 |
|----------|----------|
|![Imagen de WhatsApp 2025-02-07 a las 10 42 37_f6588a44](https://github.com/user-attachments/assets/9e51f6fc-894e-4f66-8ece-06b9039261fc)|![Imagen de WhatsApp 2025-02-07 a las 10 42 37_b0a23419](https://github.com/user-attachments/assets/90149323-95f1-437d-a54f-e07fe6a1066e)|



## Calificaciones Curso Fundamentos de Python 2

| Examen | Calificación |
|--------|-------------|
| Examen 1 | ![imagen](https://github.com/user-attachments/assets/acd0e6b6-83b8-42f4-a3e8-190c173da0bd)|
| Examen 2 | ![imagen](https://github.com/user-attachments/assets/76c7787b-18a2-4f65-a037-c730df396d25)|
| Examen 3 | ![imagen](https://github.com/user-attachments/assets/f706eded-d478-4ba4-aa66-f3d6d6502a58)|
| Examen 4 | ![imagen](https://github.com/user-attachments/assets/9631f320-bd85-49bf-950d-4308ab96cdb8)|
| Examen Final | ![imagen](https://github.com/user-attachments/assets/5a1da52e-f585-4da6-be19-33b24c4e4818)|
