# CFI-3-Redes
https://github.com/JPabloLobato/CFI-3-Redes.git

## 5. Capa de Aplicación y Multimedia – Servicios y Resolución de Nombres

### 5.1 Diseño de Servicios

Se usarán servicios basados en los protocolos HTTP/HTTPS y estarán destinados hacia dos propósitos dentro del campus universitario:

* **El Portal del Campus:** Un sitio web interactivo para los docentes, personal y alumnos. En dicho portal se proveerá información esencial como horarios de clases, eventos o recursos académicos entre otros. El HTTPS se destinará a mantener la seguridad y privacidad de información sensible con datos personales.

* **Un Distribuidor de Recursos Multimedia:** Se usará para alojar y distribuir recursos educativos y de otro tipo como:
    * Acceso a grabaciones de clases.
    * Material de apoyo (Apuntes, vídeos etc.).
    Para esto se utiliza HTTP y HTTPS, y según la importancia del contenido, se priorizará con HTTPS que requiere autenticación.

**Para la Resolución de Nombres (DNS):**

* Se usará DNS como componente fundamental para el funcionamiento del campus.
* Cuando se ingrese un nombre de dominio, el sistema lo traduce a una dirección IP correspondiente.
* Para mejorar la eficiencia se usará un sistema de caché DNS local dentro del campus.

**Para Autentificación de Usuarios:**

Entre los métodos para controlar el acceso de los usuarios a los servicios de red están:

* Autenticación por credenciales: Los usuarios tendrán un usuario y contraseña.

### 5.2 Streaming y Multimedia

Para el streaming y la distribución de contenido en el campus se hará uso de las siguiente UDP Streaming, HTTP Streaming y DASH:

* Se utilizará el **UDP** para transmisiones en vivo con baja latencia de tal forma que pueda tolerar pérdidas de paquetes y se usarán protocolos **RTP/RTCP** para gestionar la entrega y calidad del streaming.

* Para la distribución de contenido como grabaciones de clase, se usará el **HTTP** que dividirá el contenido en pequeños fragmentos que permiten una transmisión óptima con firewalls y proxies.

* Se implementará **DASH** para tecnicas de streaming adaptativo, ya que este permite al servidor proporcionar múltiples versiones del contenido a diferentes velocidades de bits, gracias a esto el cliente podrá cambiar entre estas versiones en función de su ancho de banda.

**En cuanto al Ancho de Banda:**

* La calidad del streaming se adaptará dinámicamente a la disponible para cada usuario, para esto el servidor codificará el contenido en múltiples velocidades de bits para que el cliente puede elegir la más optima.
* Se implementara buffering para suavizar las fluctuaciones del ancho de banda y así evitar interrupciones en la reproducción.
* Por último se hará uso de controles de gestión para evitar sobrecargas en la red.
