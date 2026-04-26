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

### 5. Parte de diseño 2.b
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

  
<img width="792" height="453" alt="image" src="https://github.com/user-attachments/assets/a55c4dc6-f65e-464e-b078-3dfbc7ccdeea" />

<img width="779" height="681" alt="image" src="https://github.com/user-attachments/assets/a281d15f-e83d-489a-ba59-dea68a943145" />

<img width="741" height="458" alt="image" src="https://github.com/user-attachments/assets/cfa3615e-2114-4180-b6f4-fed278e91ef1" />

<img width="728" height="639" alt="image" src="https://github.com/user-attachments/assets/33ce803c-e9eb-49e5-bf79-b253f8ef241e" />
