# Segundo Parcial - Conmutación y Teletráfico

## Información general

Nombre: Lesly Juliana Ascencio Peréz 
Asignatura: Conmutación y Teletráfico  
Tema: NetFlow, sFlow, IP Accounting, YOLO y análisis de tráfico de red  

## Objetivo

Aplicar conceptos de flujos de red, NetFlow, IP Accounting y detección de top talkers mediante un sistema basado en YOLO que genera tráfico UDP y permite analizar el comportamiento de la red.

## Contenido del repositorio

- Parte conceptual
- Parte de diseño 2.a
- Parte de diseño 2.b
- Parte empírica en Google Colab
- Evidencias
- Conclusiones

## 1. Parte conceptual

### Diferencia entre NetFlow y sFlow

NetFlow analiza los flujos completos de tráfico de red. Un flujo se identifica usando campos como IP origen, IP destino, puerto origen, puerto destino y protocolo. Es útil cuando se necesita información detallada del tráfico.

sFlow utiliza muestreo estadístico de paquetes. No analiza todos los paquetes, sino una muestra representativa. Esto reduce la carga del equipo de red.

En un enlace de 100 Gbps elegiría sFlow para detectar top talkers porque permite monitorear grandes volúmenes de tráfico sin generar tanta sobrecarga en el router o switch.

### 5-tuple en NetFlow

La 5-tuple está compuesta por:

1. IP origen  
2. IP destino  
3. Puerto origen  
4. Puerto destino  
5. Protocolo  

Para medir consumo por aplicación, el colector debe inspeccionar principalmente el puerto destino o puerto origen.  
Por ejemplo:

- HTTP: puerto 80
- HTTPS: puerto 443
- SSH: puerto 22
- YOLO UDP del laboratorio: puerto 5555

### Interpretación de IP Accounting

En la tabla se observa que la IP 192.168.1.10 envía 1500 paquetes hacia 10.0.0.5, mientras que 10.0.0.5 solo responde con 50 paquetes hacia 192.168.1.10.

Esto indica una asimetría fuerte en el tráfico. Puede significar descarga de datos, transmisión unidireccional, congestión, pérdida de paquetes o incluso un posible comportamiento anómalo.

## 2.a Arquitectura YOLO + NetFlow

La arquitectura propuesta contiene:

- Un contenedor Docker con Python y YOLOv8.
- Una cámara USB simulada en Colab.
- Una máquina virtual ligera con softflowd como exportador NetFlow.
- Un colector NetFlow dentro de Colab.
- Un dashboard para visualizar top talkers y consumo de tráfico.

### Flujo de datos

Cámara → Contenedor YOLO → Envío de detecciones → VM exportadora NetFlow → Colector → Dashboard

### Comunicación entre contenedor y VM

El contenedor YOLO envía las detecciones por UDP o TCP hacia la VM.  
La VM analiza ese tráfico usando softflowd y exporta los flujos al colector NetFlow.

### Regla IP Accounting con iptables

```bash
sudo iptables -A INPUT -p udp --dport 5555 -j ACCEPT
sudo iptables -L -v -n
```
Esta regla permite contar los paquetes y bytes recibidos hacia el puerto UDP 5555.


---

## 2.b Arquitectura estación de trenes

La estación cuenta con cinco cámaras inteligentes:

- C1: Lectura de placas con YOLO + OCR
- C2: Conteo de ocupación de parqueadero
- C3: Detección de personas
- C4: Detección de animales
- C5: Detección de objetos perdidos

Cada cámara se ejecuta en un contenedor Docker independiente.

### Direccionamiento IP

| Equipo | Función | IP |
|---|---|---|
| C1 | Placas | 10.0.0.11 |
| C2 | Parqueadero | 10.0.0.12 |
| C3 | Personas | 10.0.0.13 |
| C4 | Animales | 10.0.0.14 |
| C5 | Objetos perdidos | 10.0.0.15 |
| VM1 | Colector principal | 10.0.0.20 |
| VM2 | Colector respaldo | 10.0.0.21 |

### Cálculo de throughput

Video:

30 fps × 50 KB = 1500 KB/s

1500 KB/s × 8 = 12000 Kbps

12000 Kbps = 12 Mbps por contenedor

Metadata:

200 bytes × 10 detecciones/s = 2000 bytes/s

2000 × 8 = 16000 bps

16000 bps = 0.016 Mbps por contenedor

Total por contenedor:

12 Mbps + 0.016 Mbps = 12.016 Mbps

Total para 5 contenedores:

12.016 × 5 = 60.08 Mbps

### Protocolo recomendado para video

Para video es más adecuado UDP porque tiene menor latencia y no espera confirmación de entrega como TCP.

Para mitigar el jitter se puede usar:

- Buffer en el receptor
- Control de QoS
- Priorización del tráfico de video
- Sincronización por timestamps

## Metodología

Se siguieron los siguientes pasos:

1. Generación de tráfico UDP usando Python y YOLO
2. Captura del tráfico con tcpdump
3. Identificación de la 5-tuple
4. Análisis del tráfico generado
5. Simulación de IP Accounting
6. Generación de tráfico anómalo

### Descarga del video

Se descargó un video de prueba que será utilizado como entrada para el modelo YOLO.
<img width="792" height="453" alt="image" src="https://github.com/user-attachments/assets/a55c4dc6-f65e-464e-b078-3dfbc7ccdeea" /> 
*Figura 1. Descarga del video*

### Carga del modelo YOLO
Se cargó correctamente el modelo YOLOv8, verificando su funcionamiento en el entorno de ejecución.
<img width="750" height="341" alt="image" src="https://github.com/user-attachments/assets/07a52c02-f71b-4a02-8b12-58e56a25c36a" />
*Figura 2. Carga del modelo YOLO*

### Captura de tráfico con tcpdump
Se inició la captura de paquetes UDP en el puerto 5555 utilizando tcpdump.
<img width="783" height="673" alt="image" src="https://github.com/user-attachments/assets/7a498cde-7e4b-4020-948a-0b9d8fc2d5af" />
*Figura 3. Captura de tráfico don tdpdump*

### Identificación de la 5-tuple
La 5-tuple es un conjunto de cinco parámetros que identifican un flujo de red.
Se analizaron los paquetes capturados, identificando los campos de la 5-tuple:

- IP origen: 127.0.0.1  
- IP destino: 127.0.0.1  
- Puerto origen: 44042  
- Puerto destino: 5555  
- Protocolo: UDP  
<img width="1133" height="673" alt="image" src="https://github.com/user-attachments/assets/b7b10e5e-8221-430f-9393-6ac40b6cf4b6" />
*Figura 4. Análisis de paquetes (5-tuple)*

### Tráfico normal

Se generaron 30 paquetes UDP, contabilizando:
- Paquetes: 30  
- Bytes: 610  
<img width="741" height="458" alt="image" src="https://github.com/user-attachments/assets/cfa3615e-2114-4180-b6f4-fed278e91ef1" />
*Figura 5. Tráfico normal*

### Simulación de anomalía
Se incrementó el tráfico a 500 paquetes UDP, observando:

- Paquetes: 500  
- Bytes: 10765  

Esto evidencia un aumento significativo en el tráfico, indicando posible congestión o comportamiento anómalo.
<img width="728" height="639" alt="image" src="https://github.com/user-attachments/assets/33ce803c-e9eb-49e5-bf79-b253f8ef241e" />
*Figura 6. Tráfico con anomalia*

## Respuestas del parcial

### ¿Qué campo de la 5-tuple permite diferenciar aplicaciones?

Para diferenciar aplicaciones se analiza el puerto de destino dentro de la 5-tuple, ya que cada aplicación utiliza puertos específicos. 
Por ejemplo, HTTP usa el puerto 80, mientras que en este laboratorio el tráfico generado utiliza el puerto 5555.

---

### ¿Cómo simular sFlow?

Se puede simular sFlow enviando solo una muestra del tráfico, por ejemplo:

```python
if i % 10 == 0:
    sock.sendto(mensaje, ("127.0.0.1", 5555))
```
Esto reduce la cantidad de paquetes enviados y es útil en enlaces de alta velocidad.
---
### ¿Cuántos bytes se contabilizaron?

En condiciones normales se enviaron 30 paquetes, con un total de 610 bytes.
---
### ¿Qué comando permite ver el tráfico UDP hacia el puerto 5555?
```bash
tcpdump -nn -r /tmp/flujos.pcap
```
También se puede usar:
```bash
iptables -L -v -n | grep dpt:5555
```
Debido a limitaciones en Colab, este conteo se simuló mediante Python.
---

## Conclusión

Se aplicaron conceptos clave de teletráfico como la identificación de flujos mediante la 5-tuple, el análisis de tráfico con tcpdump y la simulación de IP Accounting.
Además, se evidenció cómo un incremento en el número de paquetes genera una anomalía en la red, lo cual permite detectar posibles situaciones de congestión o ataques.
Estos resultados demuestran la importancia del monitoreo de red en sistemas modernos.
