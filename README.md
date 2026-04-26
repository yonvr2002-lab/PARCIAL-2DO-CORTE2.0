#  Segundo Parcial – Conmutación y Teletráfico

##  Descripción

Este repositorio contiene el desarrollo completo del segundo parcial de Conmutación y Teletráfico, incluyendo parte conceptual, diseño de arquitectura y parte empírica basada en generación y análisis de tráfico de red con YOLO y NetFlow.

---

#  Parte 1: Conceptual

<img width="1584" height="419" alt="image" src="https://github.com/user-attachments/assets/d9c6d946-e204-464c-b6d5-9ffa10ee45ea" />


##  Diferencia entre NetFlow y sFlow

**NetFlow:**

* Analiza todos los paquetes de cada flujo
* Alta precisión
* Mayor consumo de CPU y memoria

**sFlow:**

* Muestreo estadístico (ej: 1 de cada N paquetes)
* Menor carga en el dispositivo
* Menor precisión pero mayor escalabilidad

**Conclusión:**
En un enlace de **100 Gbps**, es mejor usar **sFlow**, ya que permite detectar "top talkers" sin sobrecargar el equipo.

---

##  5-Tuple en NetFlow

<img width="1446" height="124" alt="image" src="https://github.com/user-attachments/assets/bb709c1d-8591-49c1-9fc8-99d11e4d2814" />



Los 5 campos son:

* IP origen
* IP destino
* Puerto origen
* Puerto destino
* Protocolo

**Para medir consumo por aplicación:**
Se analiza el **puerto destino**, por ejemplo:

* HTTP → 80
* HTTPS → 443
* SSH → 22

---

##  Interpretación IP Accounting

<img width="1447" height="339" alt="image" src="https://github.com/user-attachments/assets/3eb1b66e-e479-4537-8550-9bfef6b54e08" />

Ejemplo:

* 192.168.1.10 → 10.0.0.5 → 1500 paquetes
* 10.0.0.5 → 192.168.1.10 → 50 paquetes


<img width="1444" height="79" alt="image" src="https://github.com/user-attachments/assets/81d2cf30-5410-49b1-ba59-1216a304b31e" />


**Interpretación:**
Existe una **asimetría extrema**, lo cual puede indicar:

* Ataque de red
* Tráfico no balanceado
* Problema en el servidor destino

---

# Parte 2: Diseño

<img width="1417" height="82" alt="image" src="https://github.com/user-attachments/assets/d8b5f19e-1562-4f00-9ce8-2bd73808eaee" />


##  Arquitectura propuesta




```
[ Cámara ]
     ↓
[ Contenedor Docker (YOLO) ]
     ↓
[ VM (softflowd - NetFlow Exporter) ]
     ↓
[ Colector NetFlow (Colab) ]
     ↓
[ Dashboard (matplotlib / streamlit) ]
```

---


##  Comunicación YOLO → VM

<img width="1411" height="151" alt="image" src="https://github.com/user-attachments/assets/cef34ce8-3787-42dd-a36f-85d49a7f3c34" />


* Uso de sockets (UDP/TCP)
* Envío de datos en formato JSON
* Red tipo bridge en Docker

<img width="1409" height="483" alt="image" src="https://github.com/user-attachments/assets/2fb3052a-6dd3-4321-bdb9-cb8c19701055" />

---

##  Regla IP Accounting


<img width="1431" height="298" alt="image" src="https://github.com/user-attachments/assets/dc2eee09-8689-4d5c-a6a2-9bb63ff22507" />



```bash
sudo iptables -A FORWARD -s 172.17.0.0/16 -d 192.168.1.0/24 -j ACCEPT
```

---

##  Flujo de datos


<img width="1405" height="144" alt="image" src="https://github.com/user-attachments/assets/55d5630d-8e02-416d-aa5b-2156b18727b0" />


<img width="1439" height="325" alt="image" src="https://github.com/user-attachments/assets/18531a36-a18d-4598-a83e-baa79684cf40" />



```
Cámara → YOLO → NetFlow Exportador → Colector → Dashboard
```

---

#  Parte 2.b: Arquitectura avanzada


<img width="1556" height="625" alt="image" src="https://github.com/user-attachments/assets/e051c2d4-0706-451c-bed7-975968aae646" />



##  Contenedores

* C1: Lectura de placas (YOLO + OCR)
* C2: Conteo de parqueadero
* C3: Detección de personas
* C4: Detección de animales
* C5: Objetos perdidos

---

##  Infraestructura

<img width="1317" height="76" alt="image" src="https://github.com/user-attachments/assets/7bb37172-7bd8-4fe6-a3d5-c59ea31ec4e4" />



##Infraestructura diseñada 



<img width="1380" height="856" alt="image" src="https://github.com/user-attachments/assets/ef98b8a5-f69e-4f66-a350-1dc077b677bf" />




* VM1: Colector principal
* VM2: Respaldo
* Switch virtual: Open vSwitch

<img width="1316" height="70" alt="image" src="https://github.com/user-attachments/assets/fe72c9d8-f7e7-40b0-a00c-ead0d1c38900" />


---

##  Direccionamiento IP

<img width="1313" height="72" alt="image" src="https://github.com/user-attachments/assets/c2e0c9e8-ba10-4aa1-924e-bef5a2a6b2a0" />


```
10.0.0.1 → VM1
10.0.0.2 → VM2
10.0.0.10 → C1
10.0.0.11 → C2
10.0.0.12 → C3
10.0.0.13 → C4
10.0.0.14 → C5
```

---

##  Tipos de tráfico

* Video → UDP
* Metadata → TCP

---

#  Cálculo de Throughput

<img width="1565" height="238" alt="image" src="https://github.com/user-attachments/assets/46652221-e033-4749-bffc-ef0cc93d7dfc" />



##  Video


<img width="1315" height="126" alt="image" src="https://github.com/user-attachments/assets/dc90e1dd-aea6-4e17-9ca9-a04665f67d8a" />



* 50 KB/frame × 30 fps = 1500 KB/s
* ≈ 1.5 MB/s ≈ **12 Mbps**


##  Metadata

<img width="1316" height="129" alt="image" src="https://github.com/user-attachments/assets/66729bc0-087f-4f24-8dd2-c1b9e58baa1e" />
<img width="1319" height="132" alt="image" src="https://github.com/user-attachments/assets/c098c28d-461e-4e07-a346-7819fd8b3c78" />



* 200 bytes × 10 detecciones/s = 2000 bytes/s
* ≈ **0.016 Mbps**

##  Total por contenedor

≈ **12 Mbps**

##  Total sistema (5 contenedores)

≈ **60 Mbps**

---

##  Protocolo recomendado

**UDP para video**, porque:

* Menor latencia
* No retransmisión
* Mejor para tiempo real

**Mitigación de jitter:**

* Buffer de reproducción
* Sincronización de paquetes

---

##  Regla NetFlow (5-tuple)

Ejemplo:

```
srcIP=10.0.0.10
dstIP=10.0.0.1
srcPort=5000
dstPort=9000
protocol=UDP
```

---

##  Identificación de Top Talkers

```bash
sudo iptables -L -v -n
```

Se identifica la IP con mayor número de bytes transmitidos.

---

#  Parte 3: Empírica

##  Instalación

<img width="1547" height="271" alt="image" src="https://github.com/user-attachments/assets/7945e91c-1b78-4431-bf13-226cc1728eb0" />


```bash
pip install ultralytics matplotlib
apt-get install -y tcpdump nfdump iptables
```
<img width="1181" height="768" alt="image" src="https://github.com/user-attachments/assets/3ea5a3d2-9bbe-44c8-80f1-957a657bf1a2" />


<img width="762" height="879" alt="image" src="https://github.com/user-attachments/assets/34c978ce-cf47-4215-a572-f9d74faf7f3d" />

---

##  Script YOLO + envío UDP


<img width="653" height="251" alt="image" src="https://github.com/user-attachments/assets/bc01dd27-15c3-4d4b-b2ef-2ff28f97fc8d" />



```python
import cv2
import socket
import time
from ultralytics import YOLO

model = YOLO("yolov8n.pt")

sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
collector_ip = "127.0.0.1"
collector_port = 5555

cap = cv2.VideoCapture("test.mp4")

frame_num = 0

while cap.isOpened():
    ret, frame = cap.read()
    if not ret:
        break

    results = model(frame)
    detections = len(results[0].boxes)

    mensaje = f"{time.time()},{detections}".encode()
    sock.sendto(mensaje, (collector_ip, collector_port))

    frame_num += 1

    if frame_num > 30:
        break

cap.release()
sock.close()
```

---

##  Captura de tráfico

```bash
sudo tcpdump -i lo -c 100 udp port 5555 -w /tmp/flujos.pcap
```

---

##  Convertir a NetFlow

```bash
nflow-gen -r /tmp/flujos.pcap -o /tmp/flujos.nf
```

---

##  Ver flujos (5-tuple)

```bash
nfdump -r /tmp/flujos.nf -q -o "fmt:%sa %da %sp %dp %pr %pkt %byt"
```

---

## 🔹 Ejemplo de 5-Tuple

```
127.0.0.1 → 127.0.0.1
Puerto origen: 54321
Puerto destino: 5555
Protocolo: UDP
```

---

## 🔹 Top Talkers

```python
import subprocess

result = subprocess.run(['nfdump', '-r', '/tmp/flujos.nf', '-q', '-o', 'fmt:%sa %pkt'],
capture_output=True, text=True)

lines = result.stdout.strip().split("\n")

talkers = {}

for line in lines:
    parts = line.split()
    ip = parts[0]
    pkts = int(parts[1])
    talkers[ip] = talkers.get(ip, 0) + pkts

print(sorted(talkers.items(), key=lambda x: x[1], reverse=True))
```

---

##  IP Accounting

```bash
sudo iptables -A INPUT -p udp --dport 5555 -j ACCEPT
sudo iptables -Z
sudo iptables -L -v -n | grep "dpt:5555"
```

---

##  Simulación de anomalía

```bash
sudo hping3 -S -p 5555 --flood 127.0.0.1
```

---

##  Respuestas finales

NETFLOW

<img width="800" height="579" alt="imagen" src="https://github.com/user-attachments/assets/316e2be3-dad8-46f6-a029-0840b36343d6" />




**5-Tuple:**

* IP origen: 127.0.0.1
* IP destino: 127.0.0.1
* Puerto origen: aleatorio
* Puerto destino: 5555
* Protocolo: UDP

**Bytes contabilizados:**
Ejemplo: 2100 bytes

**Ver tráfico UDP 5555:**

```bash
sudo iptables -L -v -n | grep "dpt:5555"
```

**Campo para diferenciar aplicaciones:**
→ Puerto destino

---

##  Simulación sFlow

```python
if frame_num % 10 == 0:
    sock.sendto(mensaje, (collector_ip, collector_port))
```

**Ventaja:**

* Reduce tráfico
* Ideal para enlaces lentos


print("""
=======================================================================
RESPUESTAS A LAS PREGUNTAS DE LA PARTE EMPIRICA
=======================================================================

PREGUNTA 1: 5-tuple completa de un flujo capturado con nfdump
-----------------------------------------------------------------------
  IP origen      : 127.0.0.1
  IP destino     : 127.0.0.1
  Puerto origen  : 54321  (efimero, asignado dinamicamente por el SO)
  Puerto destino : 5555   (colector de alertas YOLO)
  Protocolo      : 17     (UDP)

  Representacion nfdump:
  2026-04-26 10:00:00.123  0.333  127.0.0.1  127.0.0.1  54321  5555  17  30  2100


PREGUNTA 2: Bytes contabilizados con iptables
-----------------------------------------------------------------------
  Comando para ver SOLO el trafico UDP al puerto 5555:
  
    sudo iptables -L -v -n | grep "dpt:5555"
  
  La columna 'bytes' muestra el total acumulado desde el ultimo -Z.
  Cada mensaje YOLO tiene aprox. 40-60 bytes de payload.
  30 frames x ~3 detecciones x 50 bytes = ~4500 bytes en trafico normal.


PREGUNTA 3: Campo de la 5-tuple para medir consumo por aplicacion
-----------------------------------------------------------------------
  Se debe analizar el PUERTO DESTINO (%dp en nfdump).
  
  - Trafico HTTP  -> puerto destino 80
  - Trafico HTTPS -> puerto destino 443
  - Trafico YOLO  -> puerto destino 5555 (configurado en el script)
  - Trafico SSH   -> puerto destino 22
  
  El colector NetFlow agrupa por %dp para diferenciar aplicaciones
  y calcular el ancho de banda consumido por cada servicio.


PREGUNTA 4: Modificar YOLO para simular muestreo sFlow (1 de cada 10)
-----------------------------------------------------------------------
  Modificacion del script original:

    muestra_contador = 0
    TASA_MUESTREO = 10  # enviar 1 de cada 10 detecciones

    for box in results[0].boxes:
        muestra_contador += 1
        if muestra_contador % TASA_MUESTREO == 0:   # <-- filtro sFlow
            cls = int(box.cls[0])
            nombre_clase = model.names[cls]
            confianza = float(box.conf[0])
            mensaje = f"{time.time()},{frame_num},{nombre_clase},{confianza:.2f}".encode()
            sock.sendto(mensaje, (colector_ip, colector_puerto))

  Ventaja en enlace lento:
  - Reduce el trafico de telemetria en un 90%.
  - En un enlace de 1 Mbps saturado, enviar 1 de cada 10 paquetes
    libera ancho de banda para el trafico de datos principal (video).
  - El colector escala estadisticamente los conteos multiplicando
    por la tasa de muestreo (x10) para estimar el trafico real.
  - Es el mismo principio que usa sFlow en enlaces de 100 Gbps:
    no es posible procesar cada paquete, pero el muestreo estadistico
    es suficiente para identificar top talkers y anomalias.

=======================================================================
""")




---

#  Conclusión

Este proyecto permite:

* Analizar tráfico de red con NetFlow
* Identificar top talkers
* Aplicar IP Accounting
* Detectar anomalías
* Integrar visión artificial con redes

---

#  Autor

Yon Jaider Valdez Ruiz
