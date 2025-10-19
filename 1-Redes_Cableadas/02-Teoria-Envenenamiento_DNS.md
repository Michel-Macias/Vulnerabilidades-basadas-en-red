# Tema 2: Ataque de Envenenamiento de Caché DNS

## Resumen del Concepto

El **Sistema de Nombres de Dominio (DNS)** es el protocolo que traduce nombres de dominio legibles para humanos (ej: `www.google.com`) a direcciones IP numéricas que las máquinas pueden entender. Para mejorar la eficiencia y reducir la latencia, tanto los servidores DNS como los sistemas operativos de los clientes mantienen una **caché** con las respuestas a las consultas recientes.

El **envenenamiento de caché DNS (DNS Cache Poisoning o DNS Spoofing)** es un ataque en el que se introducen datos falsos en la caché de un servidor DNS. El objetivo es que el servidor devuelva una dirección IP maliciosa cuando un usuario consulte por un dominio legítimo. Como resultado, la víctima es redirigida, sin saberlo, a un servidor controlado por el atacante en lugar del sitio web real que intentaba visitar.

### La Vulnerabilidad

La vulnerabilidad fundamental proviene del diseño original de DNS, que utilizaba el protocolo **UDP**, un protocolo "sin conexión" que no verifica la identidad del remitente. Para que un ataque de envenenamiento tenga éxito, un atacante debe "ganar la carrera" a la respuesta legítima del servidor de nombres autoritativo y, lo más importante, adivinar dos valores clave en la consulta DNS:

1.  **El ID de Transacción (TXID):** Un número de 16 bits que debe coincidir entre la consulta y la respuesta.
2.  **El Puerto de Origen UDP:** El puerto desde el cual el servidor DNS recursivo envió la consulta.

Si un atacante puede predecir o adivinar estos dos valores, puede forjar una respuesta DNS que el servidor aceptará como legítima, "envenenando" así su caché.

### Impacto del Ataque

*   **Phishing:** Redirigir a los usuarios a sitios web falsos (ej. un clon del sitio de un banco) para robar credenciales.
*   **Distribución de Malware:** Engañar a los usuarios para que descarguen software malicioso desde un servidor controlado por el atacante.
*   **Censura y Manipulación:** Bloquear el acceso a sitios legítimos o redirigir a los usuarios a propaganda.

---

## Mitigación

1.  **DNSSEC (DNS Security Extensions):** Es la solución criptográfica al problema. DNSSEC añade firmas digitales a los registros DNS, permitiendo a un servidor DNS verificar que la respuesta que ha recibido es auténtica y no ha sido manipulada. Es la contramedida más robusta.

2.  **Aleatorización del Puerto de Origen:** Los servidores DNS modernos ya no usan puertos predecibles para sus consultas salientes. Utilizan un puerto UDP aleatorio de entre los miles disponibles, lo que hace exponencialmente más difícil para un atacante adivinar la combinación correcta de TXID y puerto.

3.  **Uso de Servidores DNS Confiables:** Utilizar servidores DNS recursivos públicos y bien mantenidos (como 8.8.8.8 de Google, 1.1.1.1 de Cloudflare o 9.9.9.9 de Quad9) que implementan estas y otras medidas de seguridad.

4.  **Limitar la Recursión:** Configurar los servidores DNS para que solo realicen consultas recursivas para clientes autorizados (dentro de su propia red), y no para cualquier petición desde Internet.

---

## Posibles Preguntas de Examen

1.  **Pregunta:** ¿Cuál es el objetivo principal de un ataque de envenenamiento de caché DNS?
    *   A) Degradar el rendimiento de un servidor DNS mediante una sobrecarga de peticiones.
    *   B) Interceptar correos electrónicos entre dos usuarios.
    *   C) Asociar un dominio legítimo con una dirección IP maliciosa en la caché de un servidor.
    *   D) Obtener los hashes de contraseña de los administradores del servidor DNS.

    **Respuesta Correcta:** C. El propósito es corromper la caché para redirigir el tráfico destinado a un sitio legítimo hacia un sitio controlado por el atacante.

2.  **Pregunta:** Para que un ataque clásico de envenenamiento de caché DNS tenga éxito, ¿qué dos valores debe adivinar o predecir el atacante en la comunicación UDP?
    *   A) La dirección MAC del servidor y el número de secuencia TCP.
    *   B) El ID de Transacción (TXID) y el puerto de origen UDP.
    *   C) El nombre de host del servidor y la dirección IP de destino.
    *   D) La clave pública del servidor y el TTL del registro.

    **Respuesta Correcta:** B. Estos dos elementos son necesarios para que el servidor DNS víctima acepte la respuesta falsificada como la respuesta legítima a su consulta.

3.  **Pregunta:** ¿Cuál de las siguientes tecnologías está diseñada específicamente para proteger la integridad y autenticidad de los registros DNS mediante el uso de firmas digitales?
    *   A) TLS (Transport Layer Security)
    *   B) SSL (Secure Sockets Layer)
    *   C) DNSSEC (DNS Security Extensions)
    *   D) DHCP Snooping

    **Respuesta Correcta:** C. DNSSEC fue creado para añadir una capa de confianza criptográfica al sistema DNS, previniendo ataques de envenenamiento y manipulación.
