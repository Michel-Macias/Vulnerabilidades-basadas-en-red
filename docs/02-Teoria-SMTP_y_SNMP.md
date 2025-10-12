# Tema 2: Explotación de SMTP y SNMP

Este documento combina la información del curso sobre ataques y explotación de vulnerabilidades en los protocolos SMTP y SNMP.

---

## 1. Fundamentos de SMTP y SNMP

La administración y comunicación en redes depende de varios protocolos clave:

1. **SMTP (Simple Mail Transfer Protocol):** Protocolo estándar para el envío de correos electrónicos entre servidores.
2. **SNMP (Simple Network Management Protocol):** Protocolo para la monitorización y gestión remota de dispositivos de red.

Ambos protocolos son esenciales, pero presentan riesgos si no se configuran correctamente.

### Servicios y Puertos Clave

| Puerto      | Protocolo | Servicio/Descripción                                      |
| :---------- | :-------- | :------------------------------------------------------- |
| TCP 25      | TCP       | SMTP (envío de correo no cifrado)                        |
| TCP 587     | TCP       | SMTP seguro (STARTTLS, autenticación)                    |
| TCP 465     | TCP       | SMTPS (SMTP sobre SSL, obsoleto)                         |
| UDP 161     | UDP       | SNMP (comunicaciones de gestión de red)                  |
| UDP 162     | UDP       | SNMP Trap (notificaciones de eventos)                    |

---

## 2. Explotación de SNMP

### Flujo del Ataque

1. **Enumeración:** El atacante escanea el puerto UDP 161 en busca de dispositivos con SNMP habilitado.
2. **Descubrimiento de Comunidades:** Se prueban cadenas de comunidad predeterminadas o débiles (`public`, `private`).
3. **SNMP Walk:** Si se accede, se puede extraer información sensible (usuarios, configuraciones, rutas, etc.).
4. **Escalada:** Con acceso de escritura, el atacante puede modificar la configuración del dispositivo.

### Herramientas de Ataque

* **Nmap:** Scripts NSE como `snmp-info`, `snmp-brute`, `snmp-interfaces`.
* **snmp-check:** Realiza SNMP walk y muestra información detallada.
* **onesixtyone:** Fuerza bruta de cadenas de comunidad.

```bash
# Ejemplo de escaneo SNMP con Nmap
nmap -sU -p 161 --script snmp-info,snmp-brute [IP_OBJETIVO]
```

---

## 3. Explotación de SMTP

### Flujo del Ataque

1. **Enumeración:** El atacante identifica servidores SMTP abiertos en la red (puerto TCP 25, 587).
2. **Relé Abierto:** Prueba si el servidor permite enviar correos sin autenticación.
3. **Enumeración de Usuarios:** Usa comandos como `VRFY` y `EXPN` para descubrir usuarios válidos.
4. **Suplantación y Spam:** Si el relé está abierto, puede enviar correos falsos o spam.

### Herramientas de Ataque

* **Nmap:** Script NSE `smtp-open-relay.nse` para detectar relé abierto.
* **smtp-user-enum:** Automatiza la enumeración de usuarios.
* **searchsploit:** Busca vulnerabilidades conocidas en servidores SMTP.

```bash
# Detectar relé abierto
nmap --script smtp-open-relay.nse -p 25 [IP_OBJETIVO]

# Enumerar usuarios
smtp-user-enum -M VRFY -U users.txt -t [IP_OBJETIVO]
```

### Comandos SMTP Clave

| Comando   | Descripción                                                                                   |
| :-------- | :-------------------------------------------------------------------------------------------- |
| HELO      | Inicia una conversación SMTP con el servidor. Ejemplo: `HELO 10.1.2.14`                       |
| EHLO      | Inicia una conversación con un servidor SMTP extendido (ESMTP).                               |
| STARTTLS  | Inicia una conexión segura TLS con el servidor de correo.                                     |
| RCPT      | Indica la dirección de correo electrónico del destinatario.                                   |
| DATA      | Inicia la transferencia del contenido de un mensaje de correo electrónico.                    |
| RSET      | Restablece (cancela) una transacción de correo electrónico.                                   |
| MAIL      | Indica la dirección de correo electrónico del remitente.                                      |
| QUIT      | Cierra la conexión con el servidor.                                                           |
| HELP      | Muestra un menú de ayuda (si está disponible).                                                |
| AUTH      | Autentica un cliente en el servidor.                                                          |
| VRFY      | Verifica si existe la casilla de correo de un usuario.                                        |
| EXPN      | Solicita o expande una lista de correo en el servidor remoto.                                 |

---

## 4. Enumeración y Reconocimiento

Antes de atacar, es fundamental identificar servicios vulnerables y recopilar información:

* **Nmap:** Scripts para SNMP y SMTP.
* **snmpwalk:** Extrae información de dispositivos SNMP.
* **smtp-user-enum:** Descubre usuarios válidos en SMTP.

---

## 5. Mitigación y Detección

### Mitigación

1. **SNMP:** Cambiar cadenas de comunidad predeterminadas, limitar acceso por IP, usar SNMPv3 con autenticación y cifrado.
2. **SMTP:** Deshabilitar relé abierto, exigir autenticación, usar STARTTLS/SMTPS, deshabilitar comandos VRFY/EXPN.
3. **Segmentación de Red:** Limitar el acceso a servicios críticos solo a hosts autorizados.

### Detección

* **Monitoreo de logs:** Revisar registros de SNMP y SMTP para detectar accesos sospechosos.
* **Alertas de tráfico:** Configurar IDS/IPS para identificar patrones de ataque.
* **Auditorías periódicas:** Verificar configuraciones y buscar vulnerabilidades conocidas.

---

Este documento proporciona una base sólida para comprender y protegerse contra las vulnerabilidades asociadas con los protocolos SMTP y SNMP.