# Tema 4: Vulnerabilidades y Explotaciones de FTP

## Resumen del Concepto

El **Protocolo de Transferencia de Archivos (FTP)** es uno de los protocolos más antiguos de Internet, diseñado para transferir archivos entre un cliente y un servidor. A pesar de su utilidad, el diseño original de FTP carece de características de seguridad esenciales, lo que lo convierte en un vector de ataque común si no se gestiona adecuadamente.

FTP opera utilizando dos canales de comunicación separados:

1.  **Canal de Control (Puerto 21/TCP):** Se utiliza para enviar comandos (como usuario, contraseña, listar archivos, etc.) y recibir las respuestas del servidor.
2.  **Canal de Datos:** Se utiliza para la transferencia real de archivos. La forma en que se establece este canal define los dos modos de FTP: Activo y Pasivo.

### Vulnerabilidades Principales

1.  **Credenciales en Texto Plano:** La vulnerabilidad más crítica de FTP estándar es que **el nombre de usuario y la contraseña se transmiten por el canal de control sin ningún tipo de cifrado**. Cualquiera que pueda interceptar el tráfico (un ataque Man-in-the-Middle) puede capturar las credenciales de inicio de sesión.

2.  **Acceso Anónimo Mal Configurado:** Muchos servidores FTP se configuran para permitir el "acceso anónimo" (con el usuario `anonymous` o `ftp`). Si este acceso anónimo tiene permisos de escritura, un atacante puede subir archivos maliciosos al servidor, como shells web, malware o scripts para realizar ataques posteriores.

3.  **Ataque de Rebote FTP (FTP Bounce Attack):** Una vulnerabilidad más antigua pero clásica. Un atacante puede abusar del comando `PORT` para instruir al servidor FTP a que establezca un canal de datos no con el cliente, sino con un **tercer equipo**. Esto permite al atacante usar el servidor FTP como un proxy para escanear puertos en otras máquinas de la red, ocultando su verdadera IP y, potencialmente, eludiendo reglas de firewall.

4.  **Software Desactualizado:** Como cualquier servicio de red, los servidores FTP (ej. vsftpd, ProFTPD, FileZilla Server) pueden tener vulnerabilidades conocidas (desbordamientos de búfer, fallos de formato de cadena, etc.) que pueden ser explotadas para obtener ejecución de código remoto (RCE) si no se mantienen actualizados.

---

## Mitigación

1.  **Utilizar Alternativas Seguras:** La medida más importante es reemplazar FTP por protocolos que cifren toda la comunicación.
    *   **SFTP (SSH File Transfer Protocol):** No está relacionado con FTP. Opera sobre el protocolo SSH (puerto 22) y cifra tanto la autenticación como la transferencia de datos.
    *   **FTPS (FTP Secure):** Es una extensión de FTP que añade una capa de cifrado mediante SSL/TLS. Puede operar de forma explícita (el cliente solicita cifrado después de conectar al puerto 21) o implícita (se conecta directamente a un puerto seguro como el 990).

2.  **Deshabilitar el Acceso Anónimo:** A menos que sea estrictamente necesario para un propósito público, el acceso anónimo debe ser deshabilitado. Si debe estar activo, asegurarse de que los permisos sean de solo lectura.

3.  **Mantener el Software Actualizado:** Aplicar parches y actualizaciones de seguridad al software del servidor FTP con regularidad.

4.  **Configuración Segura:** Configurar el servidor para que no sea vulnerable a ataques de rebote (la mayoría de los servidores modernos lo hacen por defecto) y usar listas de control de acceso (ACLs) para restringir las conexiones a direcciones IP de confianza.

---

## Posibles Preguntas de Examen

1.  **Pregunta:** Un analista de seguridad captura tráfico de red y puede leer un nombre de usuario y una contraseña en texto claro durante una transferencia de archivos. ¿Qué protocolo se estaba utilizando probablemente?
    *   A) SFTP
    *   B) FTPS
    *   C) FTP
    *   D) SCP

    **Respuesta Correcta:** C. El FTP estándar es conocido por enviar credenciales de autenticación sin cifrar, haciéndolas visibles a cualquiera que pueda escuchar el tráfico.

2.  **Pregunta:** Un atacante logra subir un archivo malicioso a un servidor público. ¿Cuál es la configuración errónea más probable que permitió esta acción?
    *   A) El servidor FTP estaba en modo pasivo.
    *   B) El servidor permitía el acceso anónimo con permisos de escritura.
    *   C) El servidor tenía el puerto 21 abierto.
    *   D) El servidor estaba usando FTPS en lugar de SFTP.

    **Respuesta Correcta:** B. Permitir que usuarios anónimos escriban archivos en el servidor es una vulnerabilidad crítica que facilita la distribución de malware o la desfiguración de sitios web.

3.  **Pregunta:** ¿Cuál de las siguientes opciones es la mejor alternativa a FTP para garantizar que tanto las credenciales como los datos transferidos estén cifrados?
    *   A) Telnet
    *   B) TFTP
    *   C) SFTP
    *   D) HTTP

    **Respuesta Correcta:** C. SFTP (SSH File Transfer Protocol) opera sobre una conexión SSH segura, proporcionando cifrado robusto para toda la sesión.
