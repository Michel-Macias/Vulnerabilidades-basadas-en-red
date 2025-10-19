# Tema 6: Ataques de Capa 2 y Protocolos de Red

**Objetivo:** Entender los ataques que explotan la confianza y los protocolos de la capa de enlace de datos (Capa 2) en una red de área local (LAN), y conocer sus mitigaciones específicas en entornos Cisco.

---

## 1. Ataques a la Tabla MAC (CAM Table)

La tabla CAM de un switch almacena las direcciones MAC asociadas a cada puerto. Esto le permite enviar las tramas únicamente al puerto de destino. Los ataques a esta tabla buscan romper esta funcionalidad.

*   **Ataque: MAC Flooding**
    *   **Descripción:** El atacante inunda el switch con miles de tramas Ethernet, cada una con una dirección MAC de origen falsa. Esto llena por completo la tabla CAM del switch.
    *   **Efecto:** Cuando la tabla CAM está llena, el switch no sabe por dónde enviar las tramas y entra en un modo de "falla", actuando como un hub: reenvía todas las tramas por todos sus puertos. Esto permite al atacante capturar todo el tráfico de la red.
    *   **Mitigación (Cisco):** **Port Security**. Permite limitar el número de direcciones MAC que se pueden aprender en un puerto. Si se supera el límite, el puerto puede configurarse para apagarse (`shutdown`), descartar el tráfico (`restrict`) o enviar una alerta (`protect`).

*   **Ataque: MAC Spoofing**
    *   **Descripción:** El atacante cambia la dirección MAC de su tarjeta de red para suplantar la de un dispositivo legítimo (ej: un servidor o la puerta de enlace). Esto puede usarse para saltarse filtros MAC o para realizar ataques MitM.
    *   **Mitigación (Cisco):** **Port Security** con direcciones MAC estáticas o "pegajosas" (sticky MACs), que aprenden la primera MAC y la bloquean a ese puerto.

---

## 2. Ataques ARP (Address Resolution Protocol)

*   **Ataque: ARP Spoofing / Poisoning**
    *   **Descripción:** El atacante envía respuestas ARP falsificadas a la red para asociar su dirección MAC con la IP de otro host (normalmente, la puerta de enlace). Las víctimas actualizan su caché ARP con esta información falsa.
    *   **Efecto:** Todo el tráfico de las víctimas hacia la puerta de enlace (y viceversa) pasa a través del atacante, permitiendo un ataque Man-in-the-Middle.
    *   **Mitigación (Cisco):** **Dynamic ARP Inspection (DAI)**. Es una característica de seguridad que intercepta todas las respuestas ARP y las valida contra una base de datos de confianza construida con DHCP Snooping. Las respuestas ARP no válidas son descartadas.

---

## 3. Ataques a VLAN (Virtual LAN)

*   **Ataque: VLAN Hopping (Switch Spoofing)**
    *   **Descripción:** El atacante aprovecha que algunos puertos de switch están configurados para negociar automáticamente un enlace troncal (trunk) a través del protocolo DTP (Dynamic Trunking Protocol). El atacante simula ser un switch y negocia un trunk, obteniendo acceso a todas las VLANs que pasan por ese enlace.
    *   **Mitigación (Cisco):** Deshabilitar DTP en los puertos de acceso (`switchport nonegotiate`) y configurarlos manualmente como puertos de acceso (`switchport mode access`).

*   **Ataque: VLAN Hopping (Double Tagging)**
    *   **Descripción:** Un ataque más avanzado donde el atacante, desde una VLAN, envía una trama con dos etiquetas 802.1Q. La primera (externa) es la de su propia VLAN, que el primer switch ve y retira. Al reenviar la trama, el segundo switch ve la segunda etiqueta (interna), que corresponde a la VLAN de la víctima, y la envía a dicha VLAN.
    *   **Mitigación (Cisco):** No usar la VLAN nativa para el tráfico de usuarios. Mover la VLAN nativa a una VLAN dedicada y no utilizada.

---

## 4. Ataques a STP (Spanning Tree Protocol)

*   **Ataque: Manipulación de STP**
    *   **Descripción:** STP previene bucles en la red eligiendo un "puente raíz" (Root Bridge). El atacante envía unidades de datos de puente (BPDU) con una prioridad superior a la del puente raíz actual para forzar una reelección y convertirse él en el nuevo puente raíz.
    *   **Efecto:** Como nuevo puente raíz, una gran parte del tráfico de la red pasará a través del atacante, facilitando un ataque MitM.
    *   **Mitigación (Cisco):** **BPDU Guard** en los puertos de acceso para apagarlos si reciben alguna BPDU. **Root Guard** en los puertos troncales para prevenir que un nuevo switch en esa línea se convierta en el raíz.

---

## 5. Explotación de Protocolos de Descubrimiento

*   **Ataque: Reconocimiento con CDP y LLDP**
    *   **Descripción:** CDP (Cisco Discovery Protocol) y LLDP (Link Layer Discovery Protocol) son protocolos usados por los dispositivos de red para anunciarse a sus vecinos. Un atacante conectado a un puerto puede escuchar estos anuncios.
    *   **Efecto:** Permite al atacante obtener información crítica sin esfuerzo: modelo del switch/router, versión del sistema operativo (IOS), dirección IP, VLAN nativa, etc. Esta información es clave para buscar vulnerabilidades conocidas.
    *   **Mitigación (Cisco):** Deshabilitar CDP y LLDP en los puertos que no lo necesiten, especialmente en los puertos de acceso donde se conectan los usuarios finales (`no cdp enable`, `no lldp run`).

---

## 6. Evasión de Listas de Control de Acceso (ACLs)

Las ACLs son reglas que permiten o deniegan tráfico en un router o switch. Aunque son más de Capa 3/4, su evasión es un concepto clave en la seguridad de la red.

*   **Ataque: Fragmentación de Paquetes**
    *   **Descripción:** Algunas ACLs simples solo inspeccionan el primer fragmento de un paquete IP. Un atacante puede dividir un paquete malicioso en múltiples fragmentos, poniendo la información inofensiva en el primero (que pasa la ACL) y la carga maliciosa en los fragmentos posteriores (que no son inspeccionados).
    *   **Mitigación:** Usar ACLs más avanzadas o firewalls de estado (stateful firewalls) que reensamblan los paquetes antes de inspeccionarlos.