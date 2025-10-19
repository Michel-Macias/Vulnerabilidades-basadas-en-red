# Tema 7: Ataques Basados en Kerberos y LDAP (Kerberoasting)

## Resumen de Conceptos

En los dominios de Windows (Active Directory), **Kerberos** es el protocolo de autenticación por defecto. Es más seguro que su predecesor, NTLM. Por otro lado, **LDAP (Lightweight Directory Access Protocol)** es el protocolo utilizado para consultar y modificar la información del directorio de Active Directory (usuarios, grupos, equipos, etc.).

### Fundamentos de Kerberos

Kerberos funciona como un servicio de terceros de confianza, el **Key Distribution Center (KDC)**, que se ejecuta en el Controlador de Dominio. Su función es emitir "tickets" que los usuarios presentan a los servicios para autenticarse.

El proceso simplificado es:
1.  **AS-REQ/AS-REP:** El usuario solicita al KDC un **Ticket Granting Ticket (TGT)**. Para ello, envía su nombre y una marca de tiempo cifrada con su propio hash de contraseña. El KDC verifica al usuario y le devuelve el TGT, cifrado con el hash de un usuario especial (`krbtgt`).
2.  **TGS-REQ/TGS-REP:** Cuando el usuario quiere acceder a un servicio (ej: un servidor de archivos), presenta su TGT al KDC y solicita un **Ticket de Servicio (TGS)** para ese servicio en particular. El KDC verifica el TGT y emite un TGS, cifrado con el hash de la cuenta de servicio que ejecuta dicho servicio.
3.  **AP-REQ/AP-REP:** El usuario presenta el TGS al servicio, que puede descifrarlo (porque conoce su propio hash) y así valida la identidad del usuario.

## El Ataque: Kerberoasting

**Kerberoasting** es un ataque de post-explotación que abusa del paso 2 del proceso de Kerberos. El objetivo es obtener los hashes de las contraseñas de las **cuentas de servicio**. Muchas veces, estas cuentas de servicio se configuran con contraseñas débiles y que no expiran, y a menudo tienen privilegios elevados en la red.

### Flujo del Ataque

1.  **Compromiso Inicial:** El atacante necesita acceso a la red con **cualquier cuenta de usuario del dominio**, no necesita privilegios elevados.

2.  **Descubrimiento (LDAP):** El atacante realiza una consulta LDAP al Controlador de Dominio para encontrar cuentas de usuario que estén configuradas como cuentas de servicio. Esto se hace buscando usuarios que tengan un **Service Principal Name (SPN)**. Un SPN asocia una cuenta de servicio con el servicio que ejecuta (ej: `MSSQLSvc/servidor.dominio.com`).

3.  **Solicitud de Ticket de Servicio (TGS-REQ):** Una vez identificadas las cuentas con SPN, el atacante, como un usuario de dominio válido, solicita al KDC un Ticket de Servicio (TGS) para cada uno de esos servicios. Esta es una acción **totalmente legítima** en el protocolo Kerberos, por lo que no levanta sospechas.

4.  **Extracción del Hash:** El KDC responde con un TGS (TGS-REP). La parte crucial es que una porción de este ticket está **cifrada con el hash NTLM de la cuenta de servicio**. El atacante extrae esta porción cifrada del ticket de la memoria de su propia sesión.

5.  **Crackeo Offline:** El atacante toma estos hashes y los crackea offline utilizando herramientas como `Hashcat` o `John the Ripper`. Como los hashes son de tipo RC4-HMAC (un tipo de hash relativamente rápido de crackear) y las contraseñas de las cuentas de servicio suelen ser débiles, es muy probable que el atacante obtenga la contraseña en texto plano.

Una vez que el atacante tiene la contraseña de una cuenta de servicio, puede usarla para autenticarse como esa cuenta y, si tiene privilegios, moverse lateralmente o escalar privilegios.

### Herramientas

*   **GetUserSPNs.py (Impacket):** Automatiza los pasos 2 y 3 para descubrir y solicitar los tickets.
*   **Rubeus:** Una herramienta muy popular en C# para interactuar con Kerberos y realizar Kerberoasting.
*   **Hashcat:** Para el crackeo offline de los hashes obtenidos.

### Mitigación

*   **Contraseñas Fuertes y Rotación:** Utilizar contraseñas largas y complejas (más de 25 caracteres) para las cuentas de servicio. Esto hace que el crackeo offline sea inviable. Implementar una política de rotación periódica de estas contraseñas.
*   **Cuentas de Servicio Administradas (gMSA):** Usar Group Managed Service Accounts. Windows gestiona automáticamente las contraseñas de estas cuentas, haciéndolas extremadamente largas y complejas y rotándolas periódicamente.
*   **Monitorización:** Vigilar un número elevado de solicitudes de TGS desde una misma cuenta en un corto período de tiempo, aunque es difícil de detectar ya que son peticiones válidas.

---

## Posibles Preguntas de Examen

1.  **Pregunta:** Un atacante con una cuenta de usuario de dominio sin privilegios quiere escalar privilegios. ¿Qué técnica le permitiría obtener los hashes de las cuentas de servicio para crackearlos offline?
    *   A) Pass-the-Hash
    *   B) Kerberoasting
    *   C) Golden Ticket
    *   D) Envenenamiento ARP

    **Respuesta Correcta:** B. Kerberoasting se centra específicamente en abusar de los SPN y los tickets de servicio (TGS) para obtener los hashes de las cuentas de servicio.

2.  **Pregunta:** Durante un ataque de Kerberoasting, ¿qué tipo de información busca un atacante en el directorio de Active Directory para identificar sus objetivos?
    *   A) Usuarios que son miembros del grupo "Domain Admins".
    *   B) Cuentas de equipo que no han iniciado sesión recientemente.
    *   C) Cuentas de usuario con un Service Principal Name (SPN) configurado.
    *   D) Políticas de grupo (GPO) con configuraciones de seguridad débiles.

    **Respuesta Correcta:** C. El SPN es el indicador de que una cuenta de usuario está siendo utilizada para ejecutar un servicio, lo que la convierte en un objetivo para Kerberoasting.

3.  **Pregunta:** ¿Cuál es la medida de mitigación más efectiva contra el Kerberoasting?
    *   A) Deshabilitar Kerberos y usar solo NTLM.
    *   B) Usar contraseñas muy largas y complejas para las cuentas de servicio o usar gMSA.
    *   C) Bloquear el tráfico LDAP en la red.
    *   D) Implementar la autenticación de dos factores (2FA) para todos los usuarios.

    **Respuesta Correcta:** B. El ataque de Kerberoasting depende de la capacidad de crackear el hash offline. Si la contraseña es lo suficientemente fuerte, el ataque fracasa.