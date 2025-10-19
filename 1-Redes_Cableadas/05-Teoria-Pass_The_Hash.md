# Tema 5: Ataques de Transferencia de Hash (Pass-the-Hash)

## Resumen del Concepto

**Pass-the-Hash (PtH)** es una técnica de ataque post-explotación que permite a un atacante autenticarse en sistemas remotos utilizando el hash de la contraseña de un usuario, en lugar de la contraseña en texto plano. Este ataque es especialmente efectivo en entornos de red de Windows que utilizan el protocolo de autenticación NTLM.

La idea fundamental es que, en muchos escenarios, el protocolo de autenticación no necesita la contraseña real, sino que le basta con una prueba criptográfica de que se conoce la contraseña. El hash NTLM es esa prueba.

### ¿Por qué funciona?

Cuando un usuario se autentica en un servidor usando NTLM, el proceso (simplificado) es el siguiente:

1.  El cliente envía el nombre de usuario al servidor.
2.  El servidor responde con un "desafío" (un número aleatorio).
3.  El cliente cifra ese desafío utilizando el hash NTLM de la contraseña del usuario y envía la "respuesta" al servidor.
4.  El servidor, que ya conoce (o puede verificar) el hash del usuario, realiza la misma operación y comprueba si su resultado coincide con la respuesta del cliente. Si coincide, la autenticación es exitosa.

Como se puede ver, la contraseña en texto plano nunca viaja por la red. Un atacante que ha conseguido volcar los hashes de la memoria de un sistema (usando herramientas como **Mimikatz**) puede saltarse por completo la necesidad de crackear el hash. Simplemente lo "inyecta" en una nueva sesión de autenticación para hacerse pasar por el usuario.

### Flujo del Ataque

1.  **Compromiso Inicial:** El atacante obtiene acceso a una máquina en la red (por ejemplo, mediante phishing o explotando una vulnerabilidad).
2.  **Escalada de Privilegios:** El atacante escala privilegios en esa máquina para poder ejecutar herramientas que requieren permisos de administrador (como `mimikatz`).
3.  **Volcado de Credenciales:** El atacante utiliza `mimikatz` (con el comando `sekurlsa::logonpasswords`) para extraer de la memoria del proceso LSASS los hashes de las contraseñas de todos los usuarios que han iniciado sesión en esa máquina, incluyendo administradores de dominio.
4.  **Movimiento Lateral:** El atacante utiliza una herramienta (como `psexec`, `smbexec` o módulos de Metasploit) junto con el hash NTLM robado para autenticarse en otras máquinas de la red (servidores, controladores de dominio, etc.) como si fuera el usuario legítimo.

### Mitigación

*   **Credential Guard (en Windows 10/Server 2016 y superior):** Utiliza seguridad basada en virtualización (VBS) para aislar el proceso LSASS, impidiendo que incluso un administrador pueda acceder directamente a su memoria y volcar los hashes.
*   **Principio de Mínimo Privilegio:** Asegurarse de que los usuarios (y especialmente los administradores) solo inicien sesión en las máquinas que sean estrictamente necesarias. Evitar que un administrador de dominio inicie sesión en una estación de trabajo estándar.
*   **Segmentación de Red:** Limitar la capacidad de un atacante para moverse lateralmente. Las estaciones de trabajo no deberían poder iniciar conexiones SMB directamente a los controladores de dominio, por ejemplo.
*   **Monitoreo de Actividad Anómala:** Vigilar eventos de inicio de sesión inusuales, como una cuenta que accede a múltiples sistemas en un corto período de tiempo.
*   **Uso de Kerberos:** Aunque Kerberos también puede ser atacado (Pass-the-Ticket), es generalmente más robusto que NTLM. Sin embargo, NTLM a menudo se mantiene por compatibilidad.

---

## Posibles Preguntas de Examen

1.  **Pregunta:** Un atacante ha utilizado Mimikatz para extraer el hash NTLM de un administrador de dominio desde una estación de trabajo comprometida. ¿Qué técnica le permitiría al atacante usar ese hash para acceder a un controlador de dominio sin conocer la contraseña real?
    *   A) Fuerza bruta
    *   B) Pass-the-Hash
    *   C) Envenenamiento de caché DNS
    *   D) Inyección SQL

    **Respuesta Correcta:** B) Pass-the-Hash. Esta técnica se define precisamente por el uso directo del hash para la autenticación.

2.  **Pregunta:** ¿Cuál de las siguientes herramientas es más conocida por su capacidad de volcar hashes de contraseñas de la memoria del proceso LSASS en Windows?
    *   A) Nmap
    *   B) Wireshark
    *   C) John the Ripper
    *   D) Mimikatz

    **Respuesta Correcta:** D) Mimikatz. Es la herramienta por excelencia para la post-explotación de credenciales en entornos Windows.

3.  **Pregunta:** ¿Qué característica de seguridad de Windows, introducida en versiones modernas, es la más efectiva para mitigar los ataques de Pass-the-Hash al aislar el proceso LSASS?
    *   A) BitLocker
    *   B) Windows Defender
    *   C) Credential Guard
    *   D) User Account Control (UAC)

    **Respuesta Correcta:** C) Credential Guard. Utiliza la virtualización para proteger los secretos almacenados en LSASS, impidiendo su acceso directo.