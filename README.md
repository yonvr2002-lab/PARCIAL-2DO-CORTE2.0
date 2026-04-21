# 📡 Segundo Parcial – Conmutación y Teletráfico

## 📌 Descripción

Este repositorio contiene el desarrollo completo del segundo parcial de Conmutación y Teletráfico, incluyendo parte conceptual, diseño de arquitectura y parte empírica basada en generación y análisis de tráfico de red con YOLO y NetFlow.

---

# 🧠 Parte 1: Conceptual

<img width="1584" height="419" alt="image" src="https://github.com/user-attachments/assets/d9c6d946-e204-464c-b6d5-9ffa10ee45ea" />


## 🔹 Diferencia entre NetFlow y sFlow

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

## 🔹 5-Tuple en NetFlow

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

## 🔹 Interpretación IP Accounting

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

# 🏗️ Parte 2: Diseño

<img width="1417" height="82" alt="image" src="https://github.com/user-attachments/assets/d8b5f19e-1562-4f00-9ce8-2bd73808eaee" />


## 🔹 Arquitectura propuesta


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


## 🔹 Comunicación YOLO → VM

<img width="1411" height="151" alt="image" src="https://github.com/user-attachments/assets/cef34ce8-3787-42dd-a36f-85d49a7f3c34" />


* Uso de sockets (UDP/TCP)
* Envío de datos en formato JSON
* Red tipo bridge en Docker

<img width="1409" height="483" alt="image" src="https://github.com/user-attachments/assets/2fb3052a-6dd3-4321-bdb9-cb8c19701055" />

---

## 🔹 Regla IP Accounting


<img width="1431" height="298" alt="image" src="https://github.com/user-attachments/assets/dc2eee09-8689-4d5c-a6a2-9bb63ff22507" />



```bash
sudo iptables -A FORWARD -s 172.17.0.0/16 -d 192.168.1.0/24 -j ACCEPT
```

---

## 🔹 Flujo de datos


<img width="1405" height="144" alt="image" src="https://github.com/user-attachments/assets/55d5630d-8e02-416d-aa5b-2156b18727b0" />


```
Cámara → YOLO → NetFlow Exportador → Colector → Dashboard
```

---

# 🚉 Parte 2.b: Arquitectura avanzada


<img width="1556" height="625" alt="image" src="https://github.com/user-attachments/assets/e051c2d4-0706-451c-bed7-975968aae646" />



## 🔹 Contenedores

* C1: Lectura de placas (YOLO + OCR)
* C2: Conteo de parqueadero
* C3: Detección de personas
* C4: Detección de animales
* C5: Objetos perdidos

---

## 🔹 Infraestructura

<img width="1317" height="76" alt="image" src="https://github.com/user-attachments/assets/7bb37172-7bd8-4fe6-a3d5-c59ea31ec4e4" />


* VM1: Colector principal
* VM2: Respaldo
* Switch virtual: Open vSwitch

<img width="1316" height="70" alt="image" src="https://github.com/user-attachments/assets/fe72c9d8-f7e7-40b0-a00c-ead0d1c38900" />


---

## 🔹 Direccionamiento IP

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

## 🔹 Tipos de tráfico

* Video → UDP
* Metadata → TCP

---

# 📊 Cálculo de Throughput

## 🔹 Video

* 50 KB/frame × 30 fps = 1500 KB/s
* ≈ 1.5 MB/s ≈ **12 Mbps**

## 🔹 Metadata

* 200 bytes × 10 detecciones/s = 2000 bytes/s
* ≈ **0.016 Mbps**

## 🔹 Total por contenedor

≈ **12 Mbps**

## 🔹 Total sistema (5 contenedores)

≈ **60 Mbps**

---

## 🔹 Protocolo recomendado

**UDP para video**, porque:

* Menor latencia
* No retransmisión
* Mejor para tiempo real

**Mitigación de jitter:**

* Buffer de reproducción
* Sincronización de paquetes

---

## 🔹 Regla NetFlow (5-tuple)

Ejemplo:

```
srcIP=10.0.0.10
dstIP=10.0.0.1
srcPort=5000
dstPort=9000
protocol=UDP
```

---

## 🔹 Identificación de Top Talkers

```bash
sudo iptables -L -v -n
```

Se identifica la IP con mayor número de bytes transmitidos.

---

# 🧪 Parte 3: Empírica

## 🔹 Instalación

```bash
pip install ultralytics matplotlib
apt-get install -y tcpdump nfdump iptables
```

---

## 🔹 Script YOLO + envío UDP

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

## 🔹 Captura de tráfico

```bash
sudo tcpdump -i lo -c 100 udp port 5555 -w /tmp/flujos.pcap
```

---

## 🔹 Convertir a NetFlow

```bash
nflow-gen -r /tmp/flujos.pcap -o /tmp/flujos.nf
```

---

## 🔹 Ver flujos (5-tuple)

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

## 🔹 IP Accounting

```bash
sudo iptables -A INPUT -p udp --dport 5555 -j ACCEPT
sudo iptables -Z
sudo iptables -L -v -n | grep "dpt:5555"
```

---

## 🔹 Simulación de anomalía

```bash
sudo hping3 -S -p 5555 --flood 127.0.0.1
```

---

## 🔹 Respuestas finales

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

## 🔹 Simulación sFlow

```python
if frame_num % 10 == 0:
    sock.sendto(mensaje, (collector_ip, collector_port))
```

**Ventaja:**

* Reduce tráfico
* Ideal para enlaces lentos

---

# ✅ Conclusión

Este proyecto permite:

* Analizar tráfico de red con NetFlow
* Identificar top talkers
* Aplicar IP Accounting
* Detectar anomalías
* Integrar visión artificial con redes

---

# 👨‍💻 Autor

Yon Jaider Valdez Ruiz
