# CFI-3-Redes
https://github.com/JPabloLobato/CFI-3-Redes.git

# Red Inteligente para Campus Universitario con IoT y Servicios Multimedia

**Autores:**  
María Díaz, Juan Pablo Lobato, Mauricio Murillo, Cintia Santillán, Jiachen Ye

---

## 1. Definición del Modelo de Comunicación

### 1.1 Revisión de Modelos:

#### Modelo OSI

**Capa Física**  
Encargado de la transmisión real de bits a través de medios físicos. Tanto el cálculo de la fórmula de Shannon, las técnicas de modulación, el wifi y los cables ethernet del proyecto pertenecen a esta capa.

**Capa de Enlace de datos**  
Colabora la transmisión libre de errores entre nodos, gestiona el direccionamiento a nivel de hardware. En el proyecto representa las direcciones MAC y el control de acceso al medio.

**Capa de Red**  
Distribuye el direccionamiento lógico y la toma de decisiones de enrutamiento, por ejemplo, el Subneteo y esquema de rutas. En el proyecto esta capa se encarga de la direccion de broadcast, rango de host, Enrutamiento con Dijkstra, routers y configuraciones IP.

**Capa de Transporte**  
Se encarga de ejecutar una entrega confiable o en tiempo real. Usaremos TCP + SFTP para la transferencia de archivos y UDP + RTP para el streaming.

**Capa de Sesión**  
Establece, administra y finaliza conexiones entre aplicaciones, para la gestión de diálogos entre diferentes sistemas. La multiplexación forma parte de esta capa.

**Capa de Presentación**  
Representa y codifica los datos, haciendo el cifrado, compresión y transformación de diferentes formatos.

**Capa de Aplicación**  
Se comunica directamente con los programas que utiliza el usuario, por ejemplo, protocolos FTP/SFTP para transferir archivos, HTTP/HTTPS para los de multimedia, etc.

#### Modelo TCP/IP

**Capa de Acceso a Red**  
Sería igual que la capa física y enlace de datos en el modelo OSI. Se encarga de la transmisión de datos a nivel de hardware y de la comunican por medio físicos como el ethernet. 

**Capa de Internet**  
Se asemeja a la capa de red del modelo OSI. En este se gestiona las direcciones IP y el enrutamiento, por ejemplo, el algoritmo de Dijkstra.

**Capa de Transporte**  
Parecida a la capa de transporte del modelo OSI. Gestiona los servicios de transporte ya sea confiable o sin conexión, TCP y UDP.

**Capa de Aplicación**  
Equivalen a las capas 5, 6 y 7 del modelo OSI. Protocolos de transferencia, FTP y SFTP, manejo de contenidos HTTP y HTTPS, etc.

---

### 1.2 Diagrama
````
                              [ Servidores ]
                                   |
                             +-- [ Switch ] --+
                             |                |
                      [ Firewall ]         [ internet ]
                             |
                         [ Switch ]
                            / | \
                           /  |  \
              +-----------+   |   +------------+
              |               |                |
        [ Switch ]       [ Switch ]        [ Switch ]
         /       \            |              /     \
 [PC]--[AP]    [PC]--[AP]  [Servidor]   [Switch]  [PC]
                                       /        \
                                    [PC]        [AP]

Dispositivos inalambricos-->[alarma]--[camara]---[termostato]---[sensor de movimiento]

````
Como se observa en el diagrama, se trata de una red donde hay un router con acceso a internet que lleva firewall con zona desmilitarizada donde se alojan los servidores www/mail/dns, después sale por un firewall interno que lleva a un switch núcleo que forma parte de un anillo de switches que recorre donde cada switch corresponde a su sala pertinente, como sala servidores, sala recepción, sala biblioteca, sala aula y sala laboratorio para finalmente volver al switch núcleo.

---

## 2. Capa Física – Capacidad, Modulación, Eficiencia

### 2.1 Cálculo de la Capacidad de Enlaces

Utilizamos la Fórmula de Shannon para calcular la capacidad necesaria en los enlaces cableados e inalámbricos del campus.

**Fórmula de Shannon:**  
\[ C = B*log_2(1 + SNR) \]

Donde:
- B = Ancho de banda  
- C = Capacidad  
- SNR = Relación señal/ruido

Una señal óptima SNR está entre el rango de 18db a 30db. Una buena señal empieza a partir de los 25db. Por lo que usaremos 25db como dato para la capacidad de Shannon.

**Cálculo:**  
- Cableado: 100 MHz → 831 Mbps  
- Inalámbrico: 80 MHz → 664.8 Mbps

**Conclusión:**  
El cableado con 100 MHz tiene capacidad de 831 Mbps, y el inalámbrico de 80 MHz, 664.8 Mbps.

### 2.2 Selección de Modulación y Evaluación de Encapsulamiento

Se hará uso del método OFDM, usando WiFi 5/6 y 5G, que permite transmisión eficiente con múltiples usuarios y baja interferencia.

**Encapsulamiento:**  
Se envían 1000 bytes incluyendo:  
- Cabecera TCP = 20 bytes  
- Cabecera IP = 20 bytes  
- Cabecera MAC (802.11) = 34 bytes  
- Tráiler (FCS y otros) = 4 bytes  

**Sobrecarga:** 78 bytes  
**Eficiencia:**  
\[ Eficiencia = 1000/(1000 + 78) = 92.8%]  
Sobrecarga: 7.2%

---

## 3. Capa de Red – Diseño de Subredes y Enrutamiento

### 3.1 Segmentación del Campus en Subredes

Bloque base: `10.1.0.0/16`  
División en áreas lógicas con máscara `/22` (1022 hosts útiles cada una).

| Área         | Bloque         | Máscara         | Hosts útiles | Uso principal                        |
|--------------|----------------|------------------|---------------|--------------------------------------|
| IoT          | 10.1.0.0/22    | 255.255.252.0    | 1022          | Sensores ambientales, cámaras IP     |
| Usuarios     | 10.1.4.0/22    | 255.255.252.0    | 1022          | Estudiantes                          |
| Profesores   | 10.1.7.0/22    | 255.255.252.0    | 1022          | Profesores                           |
| Multimedia   | 10.1.8.0/22    | 255.255.252.0    | 1022          | Streaming, señalización digital      |
| Administración | 10.1.12.0/22 | 255.255.252.0    | 1022          | Servidores, VPN, controladores       |

#### Subredes de Ejemplo

##### Área IoT (10.1.0.0/22)

| VLAN        | Subred             | Máscara             | Hosts | Puerta de enlace | Broadcast        |
|-------------|--------------------|----------------------|--------|-------------------|------------------|
| IoT_P1      | 10.1.0.0/26        | 255.255.255.192      | 62     | 10.1.0.1          | 10.1.0.63        |
| IoT_P2      | 10.1.0.64/26       | 255.255.255.192      | 62     | 10.1.0.65         | 10.1.0.127       |
| IoT_Signal  | 10.1.0.128/27      | 255.255.255.224      | 30     | 10.1.0.129        | 10.1.0.159       |
| IoT_Serv    | 10.1.0.160/28      | 255.255.255.240      | 14     | 10.1.0.161        | 10.1.0.175       |

(El resto de las subredes de Usuarios, Multimedia y Administración tienen estructura similar.)

---

## 4. Capa de Transporte – Selección de Protocolos y Cálculo de Ventana

### 1. Definición de Protocolos

**TCP**  
- Usos: sistemas administrativos, transferencias, portales.  
- Ventajas: fiabilidad, control de flujo y congestión.

**UDP**  
- Usos: cámaras IP, sensores IoT, streaming.  
- Ventajas: baja latencia, menos sobrecarga.

### 2. Cálculo del Tamaño de Ventana

**Fórmula:**  
- BDP (bytes) = RTT (s) × Ancho de banda (bytes/s)  
- Ventana (segmentos) = BDP / MSS

**Parámetros:**
- Backbone (10 Gb/s, RTT 2 ms, MSS 1460 B):  
  BDP = 2.5×10⁶ B → Ventana ≈ 1712 segmentos

- Distribución–acceso (1 Gb/s, RTT 5 ms, MSS 1460 B):  
  BDP = 625,000 B → Ventana ≈ 428 segmentos

### 3. Contribución al Control de Congestión

- **Slow Start**: crecimiento exponencial  
- **Congestion Avoidance**: crecimiento lineal  
- **Fast Retransmit / Fast Recovery**: recuperación ante pérdida

### 4. Gestión de Flujo en UDP

- Uso de buffers, FEC, jitter buffering  
- Adaptación dinámica por parte de la aplicación

---

## 5. Capa de Aplicación y Multimedia – Servicios y Resolución de Nombres

### 5.1 Diseño de Servicios

**Portal del Campus:**  
- Interactivo, información académica, acceso seguro via HTTPS

**Distribuidor Multimedia:**  
- Grabaciones, apuntes, accesible con HTTP/HTTPS

**Resolución de Nombres:**  
- Sistema DNS local con caché  
- Seguridad mediante DNSSEC

**Autenticación:**  
- Usuario/contraseña

### 5.2 Streaming y Multimedia

Para el streaming y la distribución de contenido en el campus se hará uso de las siguiente UDP Streaming, HTTP Streaming y DASH: 

Se utilizará el UDP para transmisiones en vivo con baja latencia de tal forma que pueda tolerar pérdidas de paquetes y se usarán protocolos RTP/RTCP para gestionar la entrega y calidad del streaming. 

Para la distribución de contenido como grabaciones de clase, se usará el HTTP que dividirá el contenido en pequeños fragmentos que permiten una transmisión óptima con firewalls y proxies. 

Se implementará DASH para tecinas de streaming adaptativo, ya que este permite al servidor proporcionar múltiples versiones del contenido a diferentes velocidades de bits, gracias a esto el cliente podrá cambiar entre estas versiones en función de su ancho de banda. 

En cuanto al Ancho de Banda, la calidad del streaming se adaptará dinámicamente a la disponible para cada usuario, para esto el servidor codificará el contenido en múltiples velocidades de bits para que el cliente puede elegir la más optima, se implementara buffering para suavizar las fluctuaciones del ancho de banda y así evitar interrupciones en la reproducción y por último se hará uso de controles de gestión para evitar sobrecargaas en la red.

---

## 6. Seguridad en Redes – Plan y Configuración

### 6.1 Plan Integral de Seguridad

| Control                  | Tecnología / Protocolo        | Objetivo                                                   |
|--------------------------|-------------------------------|-------------------------------------------------------------|
| Perímetro                | Cisco FPR 2140 + FortiGate    | Filtrado de tráfico, DDoS, IPS/IDS                         |
| Acceso remoto seguro     | VPN IPsec (AES-256/SHA-2)     | Cifrado para administración y teletrabajo                  |
| Cifrado extremo a extremo| TLS 1.3 y SRTP                | Protección para HTTP/HTTPS, SIP y multimedia               |
| Protección DNS           | DNSSEC                        | Firmas digitales para evitar spoofing                      |
