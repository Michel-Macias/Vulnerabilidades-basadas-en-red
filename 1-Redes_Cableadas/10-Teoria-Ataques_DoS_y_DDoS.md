# Tema 10: Ataques de Denegación de Servicio (DoS y DDoS)

**Objetivo:** Entender los principios de los ataques de denegación de servicio, diferenciar entre un ataque DoS y DDoS, y conocer las categorías y técnicas más comunes de estos ataques.

---

## 1. Conceptos Fundamentales

Un **Ataque de Denegación de Servicio (Denial of Service, DoS)** es un intento de hacer que un recurso informático (como un sitio web, un servidor de aplicaciones o un enlace de red) no esté disponible para sus usuarios legítimos. El ataque logra esto abrumando al objetivo con una cantidad masiva de tráfico o peticiones, o explotando una vulnerabilidad que causa que el servicio se bloquee o se reinicie.

### Diferencia entre DoS y DDoS

*   **DoS (Denial of Service):** El ataque se origina desde **una única fuente**. Es más simple de ejecutar, pero también más fácil de mitigar, ya que el administrador de la red puede bloquear la dirección IP del atacante.

*   **DDoS (Distributed Denial of Service):** El ataque se origina desde **múltiples fuentes distribuidas geográficamente**. Los atacantes suelen coordinar una red de dispositivos comprometidos, conocida como **botnet**, para lanzar el ataque simultáneamente. Esto hace que sea extremadamente difícil de mitigar, ya que no hay una única fuente que bloquear y es complicado distinguir el tráfico malicioso del legítimo.

---

## 2. Categorías de Ataques DDoS

Los ataques DDoS se suelen clasificar en tres grandes categorías:

### a) Ataques Volumétricos

*   **Objetivo:** Saturar el ancho de banda del enlace de red del objetivo.
*   **Descripción:** Se basan en la fuerza bruta, enviando un volumen de tráfico tan grande que la "tubería" de internet del objetivo se llena por completo, impidiendo que el tráfico legítimo pueda pasar. Se miden en Gigabits por segundo (Gbps) o Terabits por segundo (Tbps).
*   **Técnicas Comunes:**
    *   **Inundación UDP (UDP Flood):** El atacante envía una gran cantidad de paquetes UDP a puertos aleatorios del objetivo. El servidor comprueba si hay alguna aplicación escuchando en esos puertos y, al no encontrarla, responde con un paquete ICMP "Destination Unreachable". Este proceso consume recursos del servidor y ancho de banda.
    *   **Inundación ICMP (ICMP Flood / Ping Flood):** Similar al anterior, pero usando paquetes ICMP Echo Request (ping). El objetivo se ve forzado a consumir recursos para procesar las peticiones y enviar las respuestas (ICMP Echo Reply).
    *   **Ataques de Amplificación:** Son una forma más sofisticada. El atacante envía peticiones a servidores de terceros mal configurados (como servidores DNS abiertos o NTP) usando la dirección IP de la víctima como dirección de origen (IP spoofing). Estos servidores envían una respuesta mucho más grande que la petición original a la víctima. El atacante logra así "amplificar" el volumen de su ataque usando los recursos de otros. Un ejemplo clásico es la **amplificación de DNS**.

### b) Ataques de Protocolo (Agotamiento de Estado)

*   **Objetivo:** Consumir todos los recursos de los equipos de red intermedios (firewalls, balanceadores de carga) o del propio servidor.
*   **Descripción:** Estos ataques explotan la forma en que funcionan ciertos protocolos de red. No necesitan un volumen masivo de tráfico, sino un alto número de paquetes por segundo (pps) para agotar la tabla de estados de los dispositivos.
*   **Técnicas Comunes:**
    *   **Inundación SYN (SYN Flood):** Es el ataque más conocido de esta categoría. Explota el handshake de tres vías de TCP (`SYN`, `SYN-ACK`, `ACK`). El atacante envía una gran cantidad de paquetes `SYN` al servidor, pero nunca completa el handshake enviando el `ACK` final. El servidor deja la conexión en un estado "semi-abierto", esperando el `ACK`. Esto consume memoria en la tabla de conexiones del servidor hasta que se agota y no puede aceptar nuevas conexiones legítimas.

### c) Ataques a la Capa de Aplicación (Capa 7)

*   **Objetivo:** Agotar los recursos de la aplicación o servicio web (CPU, memoria, hilos de ejecución).
*   **Descripción:** Son los ataques más sofisticados y difíciles de detectar, ya que imitan el comportamiento de usuarios legítimos. Se centran en peticiones que son costosas de procesar para el servidor.
*   **Técnicas Comunes:**
    *   **Inundación HTTP (HTTP Flood):** El atacante envía un gran número de peticiones HTTP `GET` o `POST` a una URL específica. Por ejemplo, peticiones a una página que realiza una búsqueda compleja en una base de datos o que genera un PDF. Esto consume los recursos del servidor web y de la base de datos.
    *   **Ataques de "Slow-rate" (ej: Slowloris):** En lugar de enviar muchas peticiones, el atacante abre muchas conexiones al servidor web y las mantiene abiertas el mayor tiempo posible, enviando los datos de la petición HTTP muy lentamente, byte a byte. Esto agota el número máximo de conexiones simultáneas que el servidor puede manejar, impidiendo que otros usuarios se conecten.

---

## 3. Mitigación de DoS y DDoS

La mitigación es compleja y suele requerir un enfoque por capas.

*   **Detección y Filtrado:** Usar servicios anti-DDoS (ya sea en la nube, como Cloudflare o Akamai, o a través del ISP) que puedan detectar patrones de ataque y filtrar el tráfico malicioso antes de que llegue a la red del objetivo.
*   **Configuración de Red:** Configurar firewalls y routers para descartar paquetes malformados o de fuentes conocidas de ataques.
*   **Escalabilidad y Redundancia:** Diseñar la infraestructura para que pueda absorber picos de tráfico (auto-escalado en la nube, balanceadores de carga).
*   **Limitar la Tasa de Peticiones (Rate Limiting):** Configurar servidores web y APIs para limitar el número de peticiones que un cliente puede hacer en un período de tiempo.

---

## Posibles Preguntas de Examen

1.  **Pregunta:** Un atacante utiliza una botnet para enviar una cantidad masiva de tráfico UDP desde miles de máquinas diferentes para saturar el ancho de banda de un servidor. ¿Qué tipo de ataque es este?
    *   A) DoS de capa de aplicación.
    *   B) DDoS volumétrico.
    *   C) Ataque de protocolo DoS.
    *   D) Ataque de degradación.

    **Respuesta Correcta:** B. Es un ataque distribuido (DDoS) y su objetivo es el volumen de tráfico para saturar el ancho de banda.

2.  **Pregunta:** ¿Qué técnica de ataque DoS explota el handshake de tres vías de TCP, dejando conexiones en estado semi-abierto para agotar la tabla de conexiones del servidor?
    *   A) Inundación ICMP
    *   B) Amplificación DNS
    *   C) Inundación SYN
    *   D) Inundación HTTP

    **Respuesta Correcta:** C. La inundación SYN se basa en enviar paquetes SYN sin completar la conexión con el ACK final.

3.  **Pregunta:** Un atacante envía peticiones HTTP GET a una función de búsqueda de un sitio web que es muy costosa en términos de CPU y acceso a base de datos. El atacante no necesita un gran ancho de banda, pero logra que el servidor se vuelva muy lento o deje de responder. ¿A qué categoría pertenece este ataque?
    *   A) Ataque de protocolo.
    *   B) Ataque volumétrico.
    *   C) Ataque a la capa de aplicación (Capa 7).
    *   D) Ataque de amplificación.

    **Respuesta Correcta:** C. Este ataque se dirige a una función específica de la aplicación para agotar sus recursos (CPU/memoria), no el ancho de banda de la red.