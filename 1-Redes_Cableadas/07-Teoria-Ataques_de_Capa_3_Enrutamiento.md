# Tema 7: Ataques de Capa 3 (Enrutamiento)

**Objetivo:** Comprender los ataques que manipulan los protocolos de enrutamiento para redirigir, interceptar o descartar tráfico a nivel de la capa de red (Capa 3), con un enfoque especial en el secuestro de BGP.

---

## 1. Manipulación de Rutas en Protocolos de Gateway Interno (IGP)

Los IGPs (Interior Gateway Protocols) son los protocolos que los routers usan para intercambiar información de enrutamiento dentro de una única red o sistema autónomo (AS). Ejemplos comunes son OSPF, EIGRP y RIP.

*   **Ataque: Inyección de Rutas Falsas**
    *   **Descripción:** Si un atacante logra obtener acceso a un router de la red (o conectar un router malicioso), puede empezar a anunciar rutas falsas. Por ejemplo, puede anunciar una ruta más específica (un prefijo de red más largo) hacia un destino popular (como un servidor interno o la salida a Internet). 
    *   **Efecto:** Los routers vecinos, siguiendo la lógica de "la ruta más específica gana", enviarán el tráfico destinado a ese recurso a través del router del atacante. Esto permite al atacante realizar un ataque Man-in-the-Middle a nivel de red o crear un agujero negro (descartando el tráfico).
    *   **Mitigación:** **Autenticación de Protocolos de Enrutamiento**. Protocolos como OSPF y EIGRP soportan la autenticación (normalmente con un hash MD5 o SHA). Esto asegura que un router solo acepte actualizaciones de enrutamiento de otros routers que compartan la misma clave secreta, impidiendo que un router no autorizado inyecte rutas.

---

## 2. Secuestro de BGP (BGP Hijacking)

BGP (Border Gateway Protocol) es el protocolo que hace funcionar Internet. Es un protocolo de gateway externo (EGP) que intercambia información de enrutamiento entre los diferentes Sistemas Autónomos (AS) que componen la red de redes (ej: un AS puede ser un gran ISP como Telefónica, Orange, etc.).

*   **Ataque: BGP Hijacking**
    *   **Descripción:** Un atacante que controla un router BGP (o compromete el de un AS) anuncia prefijos de red (bloques de direcciones IP) que no le pertenecen. Hay dos variantes principales:
        1.  **Anuncio de Origen Falso:** El atacante anuncia que su AS es el origen legítimo de un bloque de IPs que en realidad pertenece a otra organización (ej: un banco, un servicio en la nube).
        2.  **Anuncio de Ruta Más Específica:** Similar al ataque de IGP, el atacante anuncia una sub-red más específica del bloque de IPs de la víctima. Por ejemplo, si la víctima anuncia `123.45.0.0/16`, el atacante anuncia `123.45.1.0/24`. Los routers de Internet preferirán esta ruta más específica.
    *   **Efecto:** El tráfico global de Internet destinado a las IPs de la víctima es redirigido hacia el AS del atacante. Esto puede usarse para espionaje a gran escala, robo de datos (especialmente si se combina con DNS spoofing para suplantar sitios web) o para crear un agujero negro masivo, haciendo inaccesibles servicios enteros.
    *   **Ejemplos Reales:** Ha habido casos famosos, como en 2018 cuando un AS ruso redirigió el tráfico de Google, o en 2008 cuando Pakistán intentó bloquear YouTube para su país y accidentalmente lo bloqueó para todo el mundo.

*   **Mitigación: RPKI (Resource Public Key Infrastructure)**
    *   **Descripción:** RPKI es el principal mecanismo de defensa. Es un sistema de certificación que permite a los propietarios legítimos de bloques de IPs (a través de los Registros Regionales de Internet como RIPE o ARIN) crear un registro firmado digitalmente llamado **Route Origin Authorization (ROA)**. Un ROA declara qué AS está autorizado a originar un determinado prefijo de IP.
    *   **Funcionamiento:** Los routers BGP pueden entonces validar los anuncios que reciben contra esta base de datos pública. Si un AS anuncia un prefijo que no está autorizado a originar según el ROA, el router puede marcar esa ruta como inválida y descartarla.
    *   **Adopción:** La efectividad de RPKI depende de que tanto los propietarios de IPs creen los ROAs como de que los operadores de red implementen la validación. Su adopción está creciendo, pero aún no es universal.

---

## Posibles Preguntas de Examen

1.  **Pregunta:** Un atacante compromete un router en una red corporativa que usa OSPF. Anuncia una ruta hacia la red de servidores (`10.10.0.0/24`) con una métrica más favorable que la ruta legítima. ¿Cuál es el resultado más probable de este ataque?
    *   A) El switch principal bloqueará el puerto del router atacante.
    *   B) El tráfico destinado a la red de servidores será redirigido a través del router del atacante.
    *   C) El protocolo Kerberos revocará el acceso del atacante.
    *   D) El router del atacante será incapaz de procesar el tráfico y se reiniciará.

    **Respuesta Correcta:** B. Al anunciar una ruta con una métrica mejor (o un prefijo más específico), el atacante engaña a los otros routers para que le envíen el tráfico.

2.  **Pregunta:** Un ISP en un país anuncia por BGP que el origen de los prefijos de IP de un famoso servicio de redes sociales es su propio Sistema Autónomo (AS), a pesar de que pertenecen a otro AS. ¿Cómo se denomina este ataque?
    *   A) MAC Flooding
    *   B) Kerberoasting
    *   C) BGP Hijacking
    *   D) SSL Stripping

    **Respuesta Correcta:** C. Esto es la definición de un secuestro de BGP, donde un AS anuncia prefijos que no le pertenecen para desviar el tráfico de Internet.

3.  **Pregunta:** ¿Cuál es el principal mecanismo de seguridad diseñado para prevenir el secuestro de BGP, que permite a los operadores de red verificar criptográficamente si un Sistema Autónomo (AS) está autorizado para originar un prefijo de IP?
    *   A) Listas de Control de Acceso (ACLs)
    *   B) Autenticación MD5 en las sesiones BGP.
    *   C) RPKI (Resource Public Key Infrastructure).
    *   D) Dynamic ARP Inspection (DAI).

    **Respuesta Correcta:** C. RPKI y los registros ROA son el estándar diseñado específicamente para validar el origen de los anuncios BGP.