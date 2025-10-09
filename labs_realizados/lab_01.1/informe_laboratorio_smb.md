# Informe del Laboratorio de Enumeración SMB

## Resumen Ejecutivo

Este informe detalla el proceso de enumeración del servicio SMB en una máquina Windows 8.1 (`192.168.100.162`) desde una máquina Kali Linux. El objetivo era identificar usuarios, recursos compartidos y posibles debilidades. A pesar de que la máquina objetivo demostró estar bien securizada contra técnicas de enumeración anónima básicas (`enum4linux`, `smbclient`), se logró obtener información valiosa utilizando el Nmap Scripting Engine (NSE), descubriendo el sistema operativo exacto y un recurso compartido (`USERS`) con permisos de lectura anónimos, aunque el acceso final fue denegado por políticas de sesión.

---

## Fase 1: Descubrimiento de Activos

El primer paso fue localizar la máquina objetivo en el rango de red `192.168.100.0/24` utilizando `nmap`.

**Comando:**
```bash
nmap -sN 192.168.100.0/24
```

**Resultado (`nmap_scan.txt`):**
```text
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-09 23:56 CEST
Nmap scan report for UbuntuPcSalon (192.168.100.1)
Host is up (0.00018s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE         SERVICE
53/tcp open|filtered domain
MAC Address: 52:54:00:22:5E:44 (QEMU virtual NIC)

Nmap scan report for Windows8Labs.network_labs (192.168.100.162)
Host is up (0.00085s latency).
All 1000 scanned ports on Windows8Labs.network_labs (192.168.100.162) are in ignored states.
Not shown: 1000 open|filtered tcp ports (no-response)
MAC Address: 52:54:00:94:DF:99 (QEMU virtual NIC)

Nmap scan report for kali-VM-UbuntuPC.network_labs (192.168.100.134)
Host is up (0.0000040s latency).
Not shown: 995 closed tcp ports (reset)
PORT     STATE         SERVICE
3000/tcp open|filtered ppp
5001/tcp open|filtered commplex-link
8081/tcp open|filtered blackice-icecap
8082/tcp open|filtered blackice-alerts
9200/tcp open|filtered wap-wsp

Nmap done: 256 IP addresses (3 hosts up) scanned in 7.43 seconds
```
**Conclusión de la Fase:** Se identificó el objetivo como **`192.168.100.162`** basándose en el nombre de host `Windows8Labs.network_labs`.

---

## Fase 2: Enumeración con Enum4linux

Se intentó una enumeración completa utilizando `enum4linux`, una herramienta estándar para este propósito.

**Comando:**
```bash
enum4linux -a 192.168.100.162
```

**Resultado (`enum4linux_scan.txt`):**
```text
Starting enum4linux v0.9.1 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Fri Oct 10 00:23:39 2025

 =========================================( Target Information )=========================================

Target ........... 192.168.100.162
RID Range ........ 500-550,1000-1050
Username ......... 
Password ......... 
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none


 ==========================( Enumerating Workgroup/Domain on 192.168.100.162 )==========================

[+] Got domain/workgroup name: WORKGROUP


 ==============================( Nbtstat Information for 192.168.100.162 )==============================

Looking up status of 192.168.100.162
	WINDOWS8LABS    <00> -         B <ACTIVE>  Workstation Service
	WORKGROUP       <00> - <GROUP> B <ACTIVE>  Domain/Workgroup Name
	WINDOWS8LABS    <20> -         B <ACTIVE>  File Server Service
	WORKGROUP       <1e> - <GROUP> B <ACTIVE>  Browser Service Elections
	WORKGROUP       <1d> -         B <ACTIVE>  Master Browser
	..__MSBROWSE__. <01> - <GROUP> B <ACTIVE>  Master Browser

	MAC Address = 52-54-00-94-DF-99

 ==================================( Session Check on 192.168.100.162 )==================================

[E] Server doesn\u2019t allow session using username , password .  Aborting remainder of tests.

```
**Conclusión de la Fase:** La enumeración falló porque el servidor no permite "sesiones nulas" anónimas, una medida de seguridad efectiva que detuvo el escaneo.

---

## Fase 3: Diagnóstico y Enumeración Avanzada con Nmap

Tras el fallo de `enum4linux`, se intentó modificar las políticas de seguridad de Windows y desactivar el firewall para el diagnóstico, sin éxito. Esto sugirió una configuración de seguridad no estándar en la VM. Se procedió a utilizar herramientas alternativas más potentes.

**Herramienta:** Nmap Scripting Engine (NSE)

**Comando:**
```bash
nmap -p 445 --script smb-enum-shares,smb-enum-users,smb-os-discovery 192.168.100.162
```

**Resultado (`nmap_script_scan.txt`):**
```text
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-10 00:36 CEST
Nmap scan report for Windows8Labs.network_labs (192.168.100.162)
Host is up (0.00057s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds
MAC Address: 52:54:00:94:DF:99 (QEMU virtual NIC)

Host script results:
| smb-os-discovery: 
|   OS: Windows 8.1 Pro 9600 (Windows 8.1 Pro 6.3) 
|   OS CPE: cpe:/o:microsoft:windows_8.1:-
|   Computer name: Windows8Labs
|   NetBIOS computer name: WINDOWS8LABS
|   Workgroup: WORKGROUP
|_  System time: 2025-10-10T00:36:44+02:00
| smb-enum-shares: 
|   note: ERROR: Enumerating shares failed, guessing at common ones (NT_STATUS_ACCESS_DENIED)
|   account_used: 
|   \192.168.100.162\ADMIN$:
|     warning: Couldn\u2019t get details for share: NT_STATUS_ACCESS_DENIED
|     Anonymous access: <none>
|   \192.168.100.162\C$:
|     warning: Couldn\u2019t get details for share: NT_STATUS_ACCESS_DENIED
|     Anonymous access: <none>
|   \192.168.100.162\IPC$:
|     warning: Couldn\u2019t get details for share: NT_STATUS_ACCESS_DENIED
|     Anonymous access: READ
|   \192.168.100.162\USERS:
|_    Anonymous access: READ

Nmap done: 1 IP address (1 host up) scanned in 1.20 seconds
```
**Conclusión de la Fase:** El escaneo con NSE fue un éxito. Aunque la enumeración de usuarios y la lista de recursos compartidos fallaron (`NT_STATUS_ACCESS_DENIED`), se obtuvieron dos datos cruciales:
1.  **Información detallada del SO** (`smb-os-discovery`).
2.  La existencia de un recurso compartido **`USERS`** con aparentes permisos de lectura anónima (`smb-enum-shares`).

---

## Conclusión Final del Laboratorio

El laboratorio demostró que la máquina objetivo `192.168.100.162` está eficazmente protegida contra la enumeración anónima de SMB a nivel de sesión. Herramientas como `enum4linux` y `smbclient` no pudieron establecer una sesión sin credenciales.

Sin embargo, el uso de una herramienta más avanzada como **Nmap Scripting Engine** permitió descubrir información valiosa, incluyendo la versión exacta del SO y la existencia de un recurso compartido (`USERS`) con permisos de lectura anónimos a nivel de recurso. El intento de conexión posterior a este recurso fue denegado, lo que evidencia una **seguridad por capas**: aunque los permisos del recurso eran laxos, la política de seguridad de la sesión de red impidió el acceso.

El laboratorio se considera un éxito, ya que se ha evaluado la postura de seguridad del servicio SMB y se ha determinado que, para proceder, sería necesario un vector de ataque diferente que permita obtener credenciales válidas.
