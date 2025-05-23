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
![](https://github.com/JPabloLobato/CFI-3-Redes/blob/7628140b8c698cc87a6e637a556018fe1d97c362/ImagenesDOC/drawio.png)
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


##### Área IoT (10.1.0.0/22)

| VLAN        | Subred             | Máscara             | Hosts | Puerta de enlace | Broadcast        |
|-------------|--------------------|----------------------|--------|-------------------|------------------|
| IoT_P1      | 10.1.0.0/26        | 255.255.255.192      | 62     | 10.1.0.1          | 10.1.0.63        |
| IoT_P2      | 10.1.0.64/26       | 255.255.255.192      | 62     | 10.1.0.65         | 10.1.0.127       |
| IoT_Signal  | 10.1.0.128/27      | 255.255.255.224      | 30     | 10.1.0.129        | 10.1.0.159       |
| IoT_Serv    | 10.1.0.160/28      | 255.255.255.240      | 14     | 10.1.0.161        | 10.1.0.175       |

#### Área Usuarios

| VLAN         | Subred         | Máscara           | Hosts útiles | Puerta de enlace | Broadcast     |
|--------------|----------------|-------------------|---------------|------------------|---------------|
| Users_P1     | 10.1.4.0/26    | 255.255.255.192   | 62            | 10.1.4.1         | 10.1.4.63     |
| Users_P2     | 10.1.4.64/26   | 255.255.255.192   | 62            | 10.1.4.65        | 10.1.4.127    |
| Users_Signal | 10.1.4.128/27  | 255.255.255.224   | 30            | 10.1.4.129       | 10.1.4.159    |
| Users_Serv   | 10.1.4.160/28  | 255.255.255.240   | 14            | 10.1.4.161       | 10.1.4.175    |

---

#### Área Multimedia (10.1.8.0/22)

| VLAN         | Subred         | Máscara           | Hosts útiles | Puerta de enlace | Broadcast     |
|--------------|----------------|-------------------|---------------|------------------|---------------|
| MM_Aulas_P1  | 10.1.8.0/26    | 255.255.255.192   | 62            | 10.1.8.1         | 10.1.8.63     |
| MM_Aulas_P2  | 10.1.8.64/26   | 255.255.255.192   | 62            | 10.1.8.65        | 10.1.8.127    |
| MM_Signal    | 10.1.8.128/27  | 255.255.255.224   | 30            | 10.1.8.129       | 10.1.8.159    |
| MM_Serv      | 10.1.8.160/28  | 255.255.255.240   | 14            | 10.1.8.161       | 10.1.8.175    |

---

#### Área Administración (10.1.12.0/22)

| VLAN           | Subred           | Máscara           | Hosts útiles | Puerta de enlace | Broadcast     |
|----------------|------------------|-------------------|---------------|------------------|---------------|
| Admin_P1       | 10.1.12.0/26     | 255.255.255.192   | 62            | 10.1.12.1        | 10.1.12.63    |
| Admin_P2       | 10.1.12.64/26    | 255.255.255.192   | 62            | 10.1.12.65       | 10.1.12.127   |
| Admin_Signal   | 10.1.12.128/27   | 255.255.255.224   | 30            | 10.1.12.129      | 10.1.12.159   |
| Admin_Serv     | 10.1.12.160/28   | 255.255.255.240   | 14            | 10.1.12.161      | 10.1.12.175   |


---

## 4. Capa de Transporte – Selección de Protocolos y Cálculo de Ventana

### 1. Definición de Protocolos

### TCP
- **Servicios críticos:** acceso a sistemas administrativos, transferencias de archivos, portal del campus, bases de datos.  
- **Ventajas:** control de flujo y congestión, fiabilidad (retransmisiones), garantía de orden.

### UDP
- **Transmisiones en tiempo real:** cámaras IP, sensores IoT que envían datos de forma continua, streaming en vivo con RTP/RTCP.  
- **Ventajas:** baja sobrecarga, latencia reducida.  
- **Gestión de calidad:** implementar *buffering adaptativo*, *FEC* y *control de tasa* para compensar la ausencia de retransmisión.

---

### 2. Cálculo del Tamaño de Ventana

Se utiliza el **Bandwidth–Delay Product (BDP)** para dimensionar la ventana:  
`BDP (bytes) = RTT (s) × Ancho de banda (bytes/s)`  
`Ventana (segmentos) = BDP / MSS`

### Parámetros:
- **Backbone** (10 Gb/s, RTT 2 ms, MSS 1460 B):  
  `BDP = 0.002 s × 10×10⁹ bit/s = 2.5 ×10⁶ B → Ventana ≈ 1712 segmentos`

- **Distribución–acceso** (1 Gb/s, RTT 5 ms, MSS 1460 B):  
  `BDP = 0.005 s × 1×10⁹ bit/s = 625,000 B → Ventana ≈ 428 segmentos`

---

### 3. Contribución al Control de Congestión

- **Slow Start:** crecimiento exponencial hasta *ssthresh* para evitar arranques bruscos.  
- **Congestion Avoidance:** crecimiento lineal tras alcanzar *ssthresh*.  
- **Fast Retransmit / Fast Recovery:** retransmisión rápida y ajuste de la ventana a la mitad en caso de pérdidas.

> Un tamaño de ventana igual al BDP previene:
> 1. Subutilización (ventana muy pequeña) → desperdicio de ancho de banda.
> 2. Congestión excesiva (ventana muy grande) → alta latencia y pérdidas.

---

### 4. Gestión de Flujo en UDP

- Sin control de congestión nativo: la aplicación debe:
  - Limitar la tasa de envío (RTP/RTCP)
  - Usar *jitter buffers*
  - Aplicar *Forward Error Correction (FEC)*
- Implementar *buffering adaptativo* en el receptor para suavizar variaciones de retardo.

---

## 5. Capa de Aplicación y Multimedia – Servicios y Resolución de Nombres

### 5.1 Diseño de Servicios

Se usarán servicios basados en los protocolos HTTP/HTTPS y estarán destinados hacía dos propositos dentro del campus universitario: 

El Portal del Campus, que se tratara de un sitio web interactivo para los docentes, personal y alumnos. En dicho portal se proveerá información esencial como horarios de clases, eventos o recursos académicos entre otros. El HTTPS se destinará a mantener la seguridad y privacidad de información sensible con datos personales. 

Un Distribuidor de Recursos Multimedia que se usará para alojar y distribuir recursos educativos y de otro tipo como: 

-Acceso a grabaciones de clases. 

-Material de apoyo (Apuntes, vídeos etc.). 

Para esto se utiliza HTTP y HTTPS, y según la importancia del contenido, se priorizará con HTTPS que requiere autenticación. 

Para la Resolución de Nombres (DNS): 

-Se usará DNS como componente fundamental para el funcionamiento del campus. 

-Cuando se ingrese un nombre de dominio, el sistema lo traduce a una dirección IP correspondiente. 

-Para mejorar la eficiencia se usará un sistema de caché DNS local dentro del campus.  

Para Autentificación de Usuarios: 

Entre los métodos para controlar el acceso de los usuarios a los servicios de red están: 

-Autenticación por credenciales: Los usuarios tendrán un usuario y contraseña. 

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

### 6.2 Configuración y justificación

Para asegurar el campus, se implementa un plan en capas. Cada nivel de la red cuenta con controles específicos:

- **Perímetro**: Se configuran firewalls Cisco FPR 5505 en alta disponibilidad. Se aplican políticas de filtrado, inspección profunda de paquetes (IPS) y protección DDoS.  
  **Justificación**: mitigar amenazas externas antes de que ingresen a la red interna.

- **Red Interna**: Segmentación por VLAN y switches L3 para aislar tráfico según función (IoT, usuarios, multimedia, administración).  
  **Justificación**: evitar movimientos laterales y contener posibles infecciones.

- **Acceso Remoto**: Se activa VPN IPsec con cifrado AES-256 y autenticación SHA-2. Solo usuarios autorizados acceden a servicios administrativos desde fuera.  
  **Justificación**: proteger la integridad de la red durante el teletrabajo o tareas remotas.

- **DNS Seguro**: Se despliega DNSSEC en los servidores DNS internos.  
  **Justificación**: prevenir ataques de suplantación (DNS spoofing) y garantizar integridad en la resolución de nombres.

- **Control de Usuarios y Dispositivos**: Implementación de NAC para identificar dispositivos y aplicar políticas según perfil. Integración con servidores con protocolo AAA para autenticación centralizada.  
  **Justificación**: asegurar que solo usuarios válidos accedan a la red.

---

## 7. Implementación Cisco Packet Tracer

### 7.1 Construcción de la Topología del Campus
![](https://github.com/JPabloLobato/CFI-3-Redes/blob/7628140b8c698cc87a6e637a556018fe1d97c362/ImagenesDOC/cisco.png)

La red dividida en segmentos:

![](https://github.com/JPabloLobato/CFI-3-Redes/blob/7628140b8c698cc87a6e637a556018fe1d97c362/ImagenesDOC/ciscodmz.png)
![](https://github.com/JPabloLobato/CFI-3-Redes/blob/7628140b8c698cc87a6e637a556018fe1d97c362/ImagenesDOC/ciscoiot.png)
![](https://github.com/JPabloLobato/CFI-3-Redes/blob/7628140b8c698cc87a6e637a556018fe1d97c362/ImagenesDOC/ciscoadministracion.png)
![](https://github.com/JPabloLobato/CFI-3-Redes/blob/7628140b8c698cc87a6e637a556018fe1d97c362/ImagenesDOC/ciscoaservidores.png)
![](https://github.com/JPabloLobato/CFI-3-Redes/blob/7628140b8c698cc87a6e637a556018fe1d97c362/ImagenesDOC/ciscousuario.png)

---

### 7.2 Configuración de Rutas y Políticas de Seguridad

**Topología de los switches:**

![](https://github.com/JPabloLobato/CFI-3-Redes/blob/7628140b8c698cc87a6e637a556018fe1d97c362/ImagenesDOC/ciscoswtich.png)

Hemos creado una red con una topología denominada **topología de estrella**, todos los switches están como *client* menos el primero de usuario que está como *transparent*, ya que pasa al de profesor y alumno.

En este caso no puedo calcular el enrutamiento de **Dijkstra** por el simple hecho de que cada switch maneja VLANs diferentes, haciendo que no se pueda hacer un ping entre ellas.

No hemos configurado el **VPN**, pero el **firewall 2** que se encuentra entre la **DMZ** y la red interna, tiene un **security-level 50** para lo que entra desde la red interna a la DMZ y un **security-level 100** para lo que entra desde la DMZ a la red interna, impidiendo su conexión.  
Al principio quería poner 100 solo a los segmentos de servidores e IoT pero no supe hacerlo y se lo coloqué a todos.

El **firewall 1**, situado entre el **router y la DMZ**, tiene un **security-level 0** para lo que sale hacia el router y un **50** para lo que entra desde el router a la DMZ.

Tampoco hemos puesto **ACLs** por el momento, aunque el firewall mirará cada bit de información que pase por la red.

---

### 7.3 Pruebas de funcionamiento

**Ping entre el PC que administra los IoT y el detector de monitoreo:**

![](https://github.com/JPabloLobato/CFI-3-Redes/blob/7628140b8c698cc87a6e637a556018fe1d97c362/ImagenesDOC/ciscoping.png)

Dejo aquí la muestra de cuál es el ping del sensor de monitoreo:

![](https://github.com/JPabloLobato/CFI-3-Redes/blob/7628140b8c698cc87a6e637a556018fe1d97c362/ImagenesDOC/ciscoiot2.png)

En esta imagen también verá que los **Access Point** están todos configurados de la misma manera.  
Todos tienen como contraseña el `<nombre del segmento>password` y como SSID `<nombre del segmento>AC`, menos el de **IoT** que se llama `IoT_P1-`.

**Servidor Web:**

![](https://github.com/JPabloLobato/CFI-3-Redes/blob/7628140b8c698cc87a6e637a556018fe1d97c362/ImagenesDOC/ciscoweb.png)

En este servidor solo tenemos activado el **HTTP y HTTPS** con sus archivos creados emulando una web completa de una universidad.

También hacemos una “falsa simulación” con el **proxy** y el **DNS**.  
Supuestamente lo que haría el DNS sería enviar al servidor proxy y luego al servidor web para protegerlo.

