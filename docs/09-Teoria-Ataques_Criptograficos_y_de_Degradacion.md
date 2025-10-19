# Tema 9: Ataques Criptográficos y de Degradación

**Objetivo:** Entender los ataques que se dirigen a las implementaciones de protocolos criptográficos como SSL/TLS, con el objetivo de debilitar la seguridad de la comunicación, ya sea forzando el uso de cifrados obsoletos (ataques de degradación) o explotando fallos en su diseño.

---

## 1. Ataques de Degradación (Downgrade Attacks)

Un ataque de degradación es un tipo de ataque Man-in-the-Middle donde el atacante fuerza a un cliente y a un servidor a negociar y utilizar una versión de un protocolo o un conjunto de algoritmos de cifrado (cipher suite) más antiguos y débiles de lo que ambos soportan, para luego explotar las vulnerabilidades conocidas de esa versión degradada.

### Ataque: POODLE (Padding Oracle On Downgraded Legacy Encryption)

*   **Descripción:** Descubierto en 2014, POODLE es un ataque que explota una vulnerabilidad en el diseño del protocolo **SSL 3.0**. Aunque para esa fecha la mayoría de los servidores y clientes ya usaban TLS (un protocolo mucho más seguro), mantenían la compatibilidad con SSL 3.0 por si se encontraban con sistemas muy antiguos.
*   **Flujo del Ataque:**
    1.  El atacante se posiciona como Man-in-the-Middle.
    2.  Cuando el cliente intenta iniciar una conexión segura (con TLS), el atacante interfiere y hace que la conexión falle repetidamente.
    3.  Muchos navegadores, al ver que la conexión con TLS falla, están programados para reintentar la conexión usando protocolos más antiguos. El atacante sigue forzando fallos hasta que el navegador intenta conectarse usando SSL 3.0.
    4.  Una vez que la conexión se establece con SSL 3.0, el atacante explota una vulnerabilidad en el modo de cifrado por bloques (CBC) que se usaba en ese protocolo. Mediante una técnica de "oráculo de relleno" (padding oracle), el atacante puede enviar múltiples peticiones manipuladas para descifrar, byte a byte, partes de la comunicación, como las cookies de sesión o los tokens de autenticación.
*   **Mitigación:** La única mitigación real es **deshabilitar por completo el soporte para SSL 2.0 y SSL 3.0** tanto en los servidores como en los clientes. Los protocolos modernos (TLS 1.2 y 1.3) no son vulnerables a POODLE.

### Ataque: SSL Stripping

*   **Descripción:** Como vimos en el Tema 6, este es un ataque de degradación que no ataca el protocolo SSL/TLS en sí, sino que lo elimina por completo de la vista del usuario.
*   **Flujo del Ataque:** El atacante (MitM) intercepta la petición inicial de un usuario a un sitio web. Cuando el servidor redirige al usuario de HTTP a HTTPS, el atacante intercepta esta redirección. Él establece la conexión HTTPS con el servidor, pero le presenta a la víctima una versión HTTP del sitio. El usuario navega por una versión insegura del sitio sin darse cuenta, enviando toda su información en texto plano.
*   **Mitigación:** **HSTS (HTTP Strict Transport Security)**. Es una cabecera que el servidor envía al navegador y que le ordena conectarse siempre a través de HTTPS durante un período de tiempo determinado, impidiendo que el navegador acepte la versión HTTP que le ofrece el atacante.

---

## 2. Otros Ataques Criptográficos Notables

Aunque hay muchos, es importante conocer algunos de los más famosos por su impacto.

### Ataque: Heartbleed (2014)

*   **Descripción:** Heartbleed no era un fallo en el diseño del protocolo SSL/TLS, sino una **vulnerabilidad de implementación** muy grave en la popular biblioteca de criptografía **OpenSSL**. Afectaba a una extensión de TLS llamada "Heartbeat", que se usaba para mantener viva una conexión.
*   **Efecto:** Un atacante podía enviar una petición Heartbeat malformada a un servidor vulnerable. Debido a un fallo en la validación de la longitud de los datos, el servidor respondía no solo con los datos que el atacante le había enviado, sino con hasta 64 KB de datos adicionales extraídos directamente de la memoria RAM del servidor. Este proceso podía repetirse una y otra vez.
*   **Impacto:** En esos 64 KB de memoria podían encontrarse datos extremadamente sensibles: claves privadas del servidor (lo que permitía al atacante suplantar al servidor), contraseñas de usuarios, cookies de sesión, etc. Fue una de las vulnerabilidades más críticas de la historia de Internet.
*   **Mitigación:** Actualizar la versión de la biblioteca OpenSSL a una que no fuera vulnerable.

### Ataque: BEAST (Browser Exploit Against SSL/TLS)

*   **Descripción:** Un ataque de 2011 que explotaba una debilidad en el modo de cifrado por bloques (CBC) tal como se implementaba en TLS 1.0. Permitía a un atacante MitM descifrar cookies de sesión cifradas.
*   **Mitigación:** La industria reaccionó priorizando el uso de cifrados de flujo como RC4 (aunque luego se descubrió que RC4 también tenía debilidades) y, finalmente, la adopción de versiones más nuevas de TLS (1.1, 1.2, 1.3) que corrigen este problema.

---

## Posibles Preguntas de Examen

1.  **Pregunta:** Un atacante intercepta el tráfico de un usuario y fuerza repetidamente a que fallen los intentos de conexión TLS, provocando que el navegador del usuario finalmente negocie una conexión usando el obsoleto protocolo SSL 3.0. ¿Qué ataque se está llevando a cabo?
    *   A) Heartbleed
    *   B) POODLE
    *   C) BGP Hijacking
    *   D) Kerberoasting

    **Respuesta Correcta:** B. POODLE es el ataque de degradación específico que fuerza el uso de SSL 3.0 para explotar sus vulnerabilidades.

2.  **Pregunta:** ¿Cuál es la defensa más efectiva que puede implementar un administrador de un servidor web para prevenir ataques de SSL Stripping?
    *   A) Comprar un certificado SSL de una autoridad de confianza.
    *   B) Usar una versión actualizada de OpenSSL.
    *   C) Implementar la cabecera HSTS (HTTP Strict Transport Security).
    *   D) Deshabilitar el soporte para SSL 3.0.

    **Respuesta Correcta:** C. HSTS es la medida diseñada específicamente para instruir a los navegadores a que nunca acepten una conexión HTTP a un dominio, frustrando así el SSL Stripping.

3.  **Pregunta:** Una vulnerabilidad crítica en una biblioteca de software criptográfico permitió a los atacantes leer hasta 64 KB de la memoria de un servidor, exponiendo claves privadas y contraseñas. ¿A qué vulnerabilidad se refiere esta descripción?
    *   A) POODLE
    *   B) BEAST
    *   C) Heartbleed
    *   D) MAC Flooding

    **Respuesta Correcta:** C. Heartbleed fue la famosa vulnerabilidad en la implementación de OpenSSL que permitía la fuga de información de la memoria del servidor.