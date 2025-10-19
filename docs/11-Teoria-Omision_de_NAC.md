# Tema 11: Omisión del Control de Acceso a la Red (NAC)

**Objetivo:** Entender qué es el Control de Acceso a la Red (NAC), cómo funciona, y cuáles son las técnicas comunes utilizadas por los atacantes para evadir sus mecanismos de protección y obtener acceso no autorizado a la red.

---

## 1. ¿Qué es el Control de Acceso a la Red (NAC)?

El **Control de Acceso a la Red (NAC)** es una solución de seguridad que implementa políticas para controlar el acceso a la red por parte de los dispositivos que intentan conectarse. El objetivo principal del NAC es evitar que dispositivos no autorizados o que no cumplen con los requisitos de seguridad de la organización (dispositivos "no saludables") se conecten y puedan propagar malware o realizar ataques.

El NAC puede operar en dos fases:

1.  **Autenticación Previa (Pre-admission):** Antes de que un dispositivo obtenga acceso a la red, el NAC verifica su identidad y su estado. Puede comprobar:
    *   **Quién/Qué es:** Autenticación del usuario o del dispositivo (usando 802.1X, certificados, dirección MAC).
    *   **Su "salud":** Si tiene el antivirus actualizado, el sistema operativo parcheado, el firewall activado, etc. (esto se conoce como "postura" o "posture assessment").

2.  **Control Posterior (Post-admission):** Una vez que el dispositivo está en la red, el NAC puede continuar monitoreando su comportamiento y limitar su acceso solo a los recursos que necesita, o ponerlo en cuarentena si su estado cambia o realiza acciones sospechosas.

Si un dispositivo no cumple con las políticas, el NAC puede tomar varias acciones: bloquear el acceso, permitir el acceso solo a una VLAN de cuarentena con recursos limitados (para que pueda remediar su estado, por ejemplo, actualizando el antivirus), o simplemente registrar la violación.

---

## 2. Técnicas de Omisión de NAC

Los atacantes han desarrollado diversas técnicas para intentar eludir las protecciones del NAC.

### a) Suplantación de Dirección MAC (MAC Spoofing)

*   **Descripción:** Esta es la técnica más simple y común. Si el NAC utiliza la dirección MAC como el único identificador para permitir el acceso a un dispositivo (una lista blanca de MACs), un atacante puede eludirlo fácilmente.
*   **Flujo del Ataque:**
    1.  El atacante primero necesita encontrar la dirección MAC de un dispositivo que sí esté autorizado. Puede hacerlo escuchando pasivamente el tráfico de la red.
    2.  Una vez que tiene una MAC autorizada, el atacante cambia la dirección MAC de su propia tarjeta de red para que coincida con la del dispositivo legítimo.
    3.  El atacante se conecta a la red. El sistema NAC ve la MAC autorizada y le concede el acceso.
*   **Contramedida:** No depender únicamente de la dirección MAC para la autenticación. Combinarlo con otros métodos como 802.1X.

### b) Explotación de Fallos en la Evaluación de Postura

*   **Descripción:** La evaluación de la "salud" de un dispositivo suele ser realizada por un agente de software instalado en el cliente. Los atacantes pueden intentar manipular o engañar a este agente.
*   **Técnicas:**
    *   **Micro-VMs o Emuladores:** Un atacante podría ejecutar el agente del NAC dentro de un entorno virtualizado o un emulador en su propia máquina. Configura este entorno para que cumpla con todos los requisitos de salud, mientras que su sistema operativo anfitrión (el que usará para atacar) no los cumple. El agente reporta un estado saludable, y una vez que obtiene acceso, el atacante opera desde su sistema anfitrión.
    *   **Time-of-Check-to-Time-of-Use (TOCTOU):** El atacante presenta un sistema saludable durante la evaluación inicial. Inmediatamente después de ser admitido en la red, deshabilita las herramientas de seguridad (como el firewall o el antivirus) para poder lanzar sus ataques.
*   **Contramedida:** NAC con capacidades de monitoreo continuo (post-admission) que puedan detectar si el estado de un dispositivo cambia después de la conexión inicial.

### c) Ataques a 802.1X

802.1X es un estándar de autenticación de puertos mucho más robusto que el simple filtrado MAC. Sin embargo, también puede ser atacado.

*   **Descripción:** Si un atacante obtiene acceso físico a un puerto de red donde ya hay un dispositivo legítimo conectado (como un teléfono IP o una impresora), puede intentar colocarse en medio.
*   **Flujo del Ataque:** El atacante conecta un pequeño hub o switch entre el dispositivo legítimo y el puerto de la pared. El dispositivo legítimo se autentica correctamente con 802.1X. Como el switch ya ha autenticado ese puerto, el atacante puede conectar su propio dispositivo al hub y obtener acceso a la red sin pasar por el proceso de autenticación.
*   **Contramedida:** Configurar el switch para permitir una sola dirección MAC por puerto (usando `switchport port-security maximum 1`), lo que impediría que el atacante conecte un segundo dispositivo.

### d) Búsqueda de Puertos no Protegidos

*   **Descripción:** A menudo, en las redes grandes, la implementación del NAC no es perfecta. Puede haber puertos de red en salas de conferencias, impresoras o áreas comunes que, por error u omisión, no han sido configurados bajo la protección del NAC.
*   **Flujo del Ataque:** El atacante simplemente busca físicamente estos puertos y se conecta a ellos, obteniendo acceso directo a la red.
*   **Contramedida:** Auditorías de red regulares y una implementación exhaustiva y disciplinada del NAC en todos los puertos de la red.

---

## Posibles Preguntas de Examen

1.  **Pregunta:** Un sistema NAC está configurado para autorizar dispositivos basándose en una lista blanca de direcciones MAC. Un atacante escucha el tráfico, identifica la MAC de una impresora autorizada y configura su portátil para usar esa misma MAC. ¿Qué técnica está utilizando?
    *   A) VLAN Hopping
    *   B) MAC Spoofing
    *   C) ARP Poisoning
    *   D) DHCP Starvation

    **Respuesta Correcta:** B. El atacante está suplantando una dirección MAC para eludir un control de seguridad.

2.  **Pregunta:** Un empleado lleva su portátil personal a la oficina. Al conectarlo a la red, se le redirige a un portal web que le indica que su antivirus está desactualizado y le proporciona un enlace para descargar la última versión. ¿Qué tecnología de seguridad está probablemente en uso?
    *   A) Un firewall de capa de aplicación.
    *   B) Un sistema de detección de intrusiones (IDS).
    *   C) Un Control de Acceso a la Red (NAC).
    *   D) Un proxy web.

    **Respuesta Correcta:** C. El NAC es responsable de evaluar la "postura" o "salud" de un dispositivo y redirigirlo a un portal de remediación si no cumple con las políticas.

3.  **Pregunta:** ¿Cuál de las siguientes es la defensa más robusta contra un atacante que intenta conectar un segundo dispositivo a un puerto de red que ya está siendo utilizado por un dispositivo legítimo autenticado?
    *   A) Deshabilitar CDP en el puerto.
    *   B) Configurar `switchport port-security maximum 1`.
    *   C) Implementar DHCP Snooping.
    *   D) Usar VLANs separadas.

    **Respuesta Correcta:** B. La seguridad de puerto (Port Security) configurada para permitir una única dirección MAC es la contramedida directa para este ataque físico.