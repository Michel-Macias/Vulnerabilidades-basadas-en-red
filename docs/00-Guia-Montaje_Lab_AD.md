# Guía de Montaje: Laboratorio de Active Directory

**Objetivo:** Crear un entorno de laboratorio funcional de Active Directory (AD) para poder realizar prácticas de ataques avanzados como Kerberoasting, Pass-the-Ticket, etc. Este entorno simulará una red corporativa básica.

**Componentes del Laboratorio:**
1.  **Controlador de Dominio (DC):** Windows Server 2022.
2.  **Estación de Trabajo:** Windows 8.1 (o Windows 10).
3.  **Máquina Atacante:** Kali Linux.

---

## Parte 1: Creación del Controlador de Dominio (DC)

### Paso 1: Descargar la ISO de Windows Server 2022

Microsoft proporciona una versión de evaluación de 180 días, ideal para este laboratorio.

*   **Enlace de Descarga:** [Microsoft Evaluation Center - Windows Server 2022](https://www.microsoft.com/es-es/evalcenter/download-windows-server-2022)

En la página, regístrate y selecciona la opción para descargar el archivo **ISO de 64 bits**.

### Paso 2: Instalación de la VM en Virt-Manager

1.  Abre Virt-Manager y crea una nueva máquina virtual.
2.  Selecciona "Medio de instalación local (imagen ISO o CDROM)" y busca la ISO que acabas de descargar.
3.  Asigna recursos a la VM. Recomendado:
    *   **CPUs:** 2 o 4
    *   **Memoria (RAM):** Mínimo 2048 MB, recomendado 4096 MB.
    *   **Disco:** Mínimo 32 GB, recomendado 50 GB.
4.  En el último paso, marca "Personalizar la configuración antes de instalar".
5.  **Configuración de Red:** Asegúrate de que la interfaz de red esté conectada a tu red NAT de laboratorio (ej: `NAT_lab`), la misma que usan tus VMs de Kali y Windows 8.1.
6.  Inicia la instalación.

> **Durante la instalación:** Cuando te pregunte qué versión instalar, elige **Windows Server 2022 Standard (Experiencia de escritorio)**. Esto te dará la interfaz gráfica.

### Paso 3: Configuración Post-Instalación

1.  **Establece una IP Estática:** Un DC nunca debe tener una IP dinámica. Ve a la configuración de red y asígnale una IP estática dentro del rango de tu red de laboratorio (ej: `192.168.122.10`).
    *   **Máscara de subred:** `255.255.255.0`
    *   **Puerta de enlace:** La IP de tu gateway (ej: `192.168.122.1`).
    *   **Servidor DNS:** **Usa la propia IP de la máquina** (ej: `192.168.122.10`). Esto es crucial, ya que el DC será su propio servidor DNS.
2.  **Cambia el nombre del equipo:** Ponle un nombre descriptivo, como `DC-01`.

### Paso 4: Promover a Controlador de Dominio

1.  Abre el **Administrador del servidor (Server Manager)**.
2.  Ve a `Administrar` > `Agregar roles y características`.
3.  Instala el rol **"Servicios de dominio de Active Directory"**. Acepta todas las características adicionales que te sugiera.
4.  Una vez instalado, verás una notificación con una bandera amarilla en el Administrador del servidor. Haz clic en ella y selecciona **"Promover este servidor a controlador de dominio"**.
5.  En el asistente, selecciona **"Agregar un nuevo bosque"**.
6.  Dale un nombre a tu dominio raíz. **Importante:** Usa un dominio que no exista en internet, como `laboratorio.local`.
7.  Establece una contraseña para el Modo de Restauración de Servicios de Directorio (DSRM). ¡No la olvides!
8.  Sigue el asistente con las opciones por defecto. El servidor se reiniciará.

¡Felicidades, ya tienes un Controlador de Dominio!

---

## Parte 2: Unir la Estación de Trabajo al Dominio

1.  Inicia tu VM de **Windows 8.1**.
2.  **Configura su DNS:** Ve a la configuración de red de la VM y cambia el servidor DNS. Debe apuntar **únicamente a la IP de tu Controlador de Dominio** (`192.168.122.10`). Esto es para que pueda encontrar el dominio `laboratorio.local`.
3.  **Únete al Dominio:**
    *   Ve a `Sistema` (puedes buscarlo en el menú de inicio).
    *   Haz clic en `Cambiar configuración` > `Cambiar...`.
    *   En "Miembro de", selecciona `Dominio` y escribe el nombre de tu dominio: `laboratorio.local`.
    *   Te pedirá credenciales. Usa las del administrador del dominio que creaste en el DC (ej: `laboratorio\Administrator` y su contraseña).
    *   Reinicia la máquina.

Ahora tu máquina Windows 8.1 es parte del dominio.

---

## Parte 3: Crear la Cuenta de Servicio para Kerberoasting

Finalmente, crearemos el objetivo para nuestro laboratorio de Kerberoasting.

1.  **En el Controlador de Dominio (DC)**, abre `Usuarios y equipos de Active Directory`.
2.  Crea un nuevo usuario. Por ejemplo:
    *   **Nombre de usuario:** `sql_service`
    *   **Contraseña:** Elige una contraseña relativamente simple que puedas crackear después, como `ServicioSQL1234`.
    *   **Importante:** Desmarca "El usuario debe cambiar la contraseña en el siguiente inicio de sesión" y marca "La contraseña nunca expira". Esto simula una cuenta de servicio mal configurada.
3.  **Asignar un Service Principal Name (SPN):**
    *   Abre un **PowerShell como Administrador** en el DC.
    *   Ejecuta el siguiente comando para asignarle un SPN falso al usuario. Esto es lo que lo hace "kerberoastable".
    ```powershell
    setspn -A MSSQLSvc/dc-01.laboratorio.local:1433 laboratorio\sql_service
    ```

¡Listo! Tu laboratorio de Active Directory está completo y tienes una cuenta de servicio vulnerable lista para ser atacada en la práctica de Kerberoasting.