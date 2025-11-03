
<img width="1120" height="777" alt="image" src="https://github.com/user-attachments/assets/0ca72545-b40b-4796-b4bf-2b9b3c13ca82" />

## 🧩 **Configuración Inicial

1. **Configure el nombre de host apropiado en cada router/switch.**
    
2. **Configure la contraseña de acceso privilegiado (**`**enable secret**`**) con el valor** `**MiLaboratorioRed**` **en cada router/switch.** Utilice el algoritmo de hashing tipo 9 si está disponible; de lo contrario, utilice el tipo 5.
    
3. **Configure la cuenta de usuario** `**cisco**` **con la contraseña secreta** `**ccna**` **en cada router/switch.** Utilice el algoritmo de hashing tipo 9 si está disponible; de lo contrario, utilice el tipo 5.
    
4. **Configure la línea de consola** para requerir inicio de sesión con una cuenta de usuario local. Establezca un tiempo de inactividad de 30 minutos. Habilite el registro síncrono (logging synchronous).

### En **CSW1, CSW2, DSW-A1, DSW-A2, DSW-B1 y DSW-B2

```bash
hostname (title)
enable algorithm-type scrypt secret milaboratoriored
username cisco algorithm-type scrypt secret ccna
line console 0
login local
exec-timeout 30
logging synchronous
do write memory
```

### En **ALL the REST

```bash
hostname (title)
enable secret milaboratoriored
username cisco secret ccna
line console 0
login local
exec-timeout 30
logging synchronous
do write memory
```
---

## 🧩 **Configuración de VLANs, Layer-2 EtherChannel**

1. **En la Oficina A**, configure un EtherChannel de Capa 2 llamado `PortChannel1` entre `DSW-A1` y `DSW-A2` usando un protocolo propietario de Cisco. Ambos switches deben intentar activar el EtherChannel de forma activa.
    
2. **En la Oficina B**, configure un EtherChannel de Capa 2 llamado `PortChannel1` entre `DSW-B1` y `DSW-B2` utilizando un protocolo estándar abierto. Ambos switches deben intentar activar el EtherChannel de forma activa.
    
3. **Configure todos los enlaces** entre los switches de Acceso y Distribución (incluyendo los EtherChannels) como enlaces troncales:
    
    1. Deshabilite explícitamente DTP en todos los puertos.
        
    2. Establezca la VLAN nativa de cada troncal a la VLAN 1000 (sin uso).
        
    3. **En la Oficina A**, permita las VLANs 10, 20, 40 y 99 en todos los troncales.
        
    4. **En la Oficina B**, permita las VLANs 10, 20, 30 y 99 en todos los troncales.
        
4. Configure uno de los switches de Distribución de cada oficina como un servidor VTPv2 con el dominio `MiLaboratorioRed`:
    
    1. Verifique que los otros switches se unan al dominio.
        
    2. Configure todos los switches de Acceso como clientes VTP.
        
5. **En la Oficina A**, cree y nombre las siguientes VLANs en uno de los switches de Distribución. Asegúrese de que VTP propague los cambios:
    
    1. VLAN 10: PCs
        
    2. VLAN 20: Teléfonos
        
    3. VLAN 40: Wi-Fi
        
    4. VLAN 99: Gestión
        
6. **En la Oficina B**, cree y nombre las siguientes VLANs en uno de los switches de Distribución. Asegúrese de que VTP propague los cambios:
    
    1. VLAN 10: PCs
        
    2. VLAN 20: Teléfonos
        
    3. VLAN 30: Servidores
        
    4. VLAN 99: Gestión
        
7. Configure los puertos de acceso de cada switch de Acceso:
    
    1. Las LWAPs no usarán FlexConnect.
        
    2. PCs en VLAN 10, teléfonos en VLAN 20.
        
    3. `SRV1` en VLAN 30.
        
    4. Configure el modo `access` manualmente y deshabilite DTP explícitamente.
        
8. Configure la conexión de `ASW-A1` con `WLC1`:
    
    1. Debe soportar las VLANs de Wi-Fi y Gestión.
        
    2. La VLAN de Gestión debe estar sin etiquetar.
        
    3. Deshabilite DTP.
        
9. Deshabilite administrativamente todos los puertos no utilizados en los switches de Acceso y Distribución.
### ⚙️ Configuración en **DSW-A1** y **DSW-A2**

#### En **DSW-A1**

```bash
show cdp neighbors
interface range gigabitEthernet 1/0/4-5
channel-group 1 mode desirable

interface range gigabitEthernet 1/0/1-3
switchport mode trunk
switchport nonegotiate
switchport trunk native vlan 1000
switchport trunk allowed vlan 10,20,40,99

interface port-channel 1
switchport mode trunk
switchport nonegotiate
switchport trunk native vlan 1000
switchport trunk allowed vlan 10,20,40,99

vtp domain MiLaboratorioRed
vtp version 2

vlan 10
name PCs
vlan 20
name Phones
vlan 40
name Wi-Fi
vlan 99
name Management

interface range gigabitEthernet 1/0/6-24,g1/1/3-4
shutdown

```

#### En **DSW-A2**

```bash
interface range gigabitEthernet 1/0/4-5
channel-group 1 mode desirable

interface range gigabitEthernet 1/0/1-3
switchport mode trunk
switchport nonegotiate
switchport trunk native vlan 1000
switchport trunk allowed vlan 10,20,40,99
interface port-channel 1
switchport mode trunk
switchport nonegotiate
switchport trunk native vlan 1000
switchport trunk allowed vlan 10,20,40,99

interface range gigabitEthernet 1/0/6-24,g1/1/3-4
shutdown
```

#### En **ASW-A1,ASW-A2,ASW-A3**

```bash

interface range gigabitEthernet 0/1-2
switchport mode trunk
switchport nonegotiate
switchport trunk native vlan 1000
switchport trunk allowed vlan 10,20,40,99

vtp mode client

```


### ⚙️ Configuración en **DSW-B1** y **DSW-B2**

#### En **DSW-B1**

```bash
show cdp neighbors
interface range gigabitEthernet 1/0/4-5
channel-group 1 mode active

interface range gigabitEthernet 1/0/1-3
switchport mode trunk
switchport nonegotiate
switchport trunk native vlan 1000
switchport trunk allowed vlan 10,20,30,99

interface port-channel 1
switchport mode trunk
switchport nonegotiate
switchport trunk native vlan 1000
switchport trunk allowed vlan 10,20,30,99

vtp domain MiLaboratorioRed
vtp version 2

vlan 10
name PCs
vlan 20
name Phones
vlan 30
name Servers
vlan 99
name Management

interface range gigabitEthernet 1/0/6-24,g1/1/3-4
shutdown
```

#### En **DSW-B2**

```bash
interface range gigabitEthernet 1/0/4-5
channel-group 1 mode active

interface range gigabitEthernet 1/0/1-3
switchport mode trunk
switchport nonegotiate
switchport trunk native vlan 1000
switchport trunk allowed vlan 10,20,30,99

interface port-channel 1
switchport mode trunk
switchport nonegotiate
switchport trunk native vlan 1000
switchport trunk allowed vlan 10,20,30,99

interface range gigabitEthernet 1/0/6-24,g1/1/3-4
shutdown
```

#### En **ASW-B1,ASW-B2,BSW-A3**

```bash

interface range gigabitEthernet 0/1-2
switchport mode trunk
switchport nonegotiate
switchport trunk native vlan 1000
switchport trunk allowed vlan 10,20,30,99

vtp mode client
```


### Configuración en **ASW-A1**

```bash
interface fastEthernet 0/1
switchport mode access
switchport nonegotiate
switchport access vlan 99

interface fastEthernet 0/2
switchport mode trunk
switchport trunk allowed vlan 40,99
switchport trunk native vlan 99
switchport nonegotiate

interface range fastEthernet 0/3-24
shutdown

```
### Configuración **ASW-B1**

```bash
interface fastEthernet 0/1
switchport mode access
switchport nonegotiate
switchport access vlan 99

interface range fastEthernet 0/2-24
shutdown
```

### Configuración en **ASW-A2, ASW-A3 y ASW-B2**

```bash
interface fastEthernet 0/1
switchport mode access
switchport nonegotiate
switchport access vlan 10
switchport voice vlan 20

interface range fastEthernet 0/2-24
shutdown
```

### Configuración en **ASW-B3***

```bash
interface fastEthernet 0/1
switchport mode access
switchport nonegotiate
switchport access vlan 30

interface range fastEthernet 0/2-24
shutdown
```


## 🧩 **Configuración de IP Addresses, Layer-3 EtherChannel, HSRP

1. Configure las siguientes direcciones IP en las interfaces de `R1` y habilítelas:
    
    1. `G0/0/0`: Cliente DHCP
        
    2. `G0/1/0`: Cliente DHCP
        
    3. `G0/0`: 10.0.0.33/30
        
    4. `G0/1`: 10.0.0.37/30
        
    5. `Loopback0`: 10.0.0.76/32
        
2. Habilite el enrutamiento IPv4 en todos los switches de Capa Core y Distribución.
    
3. Cree un EtherChannel de Capa 3 entre `CSW1` y `CSW2` usando un protocolo propietario de Cisco. Ambos switches deben intentar formar el EtherChannel de forma activa. Configure las siguientes direcciones IP:
    
    1. `CSW1` `PortChannel1`: 10.0.0.41/30
        
    2. `CSW2` `PortChannel1`: 10.0.0.42/30
        
4. Configure las siguientes direcciones IP en `CSW1` y deshabilite todas las interfaces no utilizadas:
    
    1. `G1/0/1`: 10.0.0.34/30
        
    2. `G1/1/1`: 10.0.0.45/30
        
    3. `G1/1/2`: 10.0.0.49/30
        
    4. `G1/1/3`: 10.0.0.53/30
        
    5. `G1/1/4`: 10.0.0.57/30
        
    6. `Loopback0`: 10.0.0.77/32
        
5. Configure las siguientes direcciones IP en `CSW2` y deshabilite todas las interfaces no utilizadas:
    
    1. `G1/0/1`: 10.0.0.38/30
        
    2. `G1/1/1`: 10.0.0.61/30
        
    3. `G1/1/2`: 10.0.0.65/30
        
    4. `G1/1/3`: 10.0.0.69/30
        
    5. `G1/1/4`: 10.0.0.73/30
        
    6. `Loopback0`: 10.0.0.78/32
        
6. Configure las siguientes direcciones IP en `DSW-A1`:
    
    1. `G1/1/1`: 10.0.0.46/30
        
    2. `G1/1/2`: 10.0.0.62/30
        
    3. `Loopback0`: 10.0.0.79/32
        
7. Configure las siguientes direcciones IP en `DSW-A2`:
    
    1. `G1/1/1`: 10.0.0.50/30
        
    2. `G1/1/2`: 10.0.0.66/30
        
    3. `Loopback0`: 10.0.0.80/32
        
8. Configure las siguientes direcciones IP en `DSW-B1`:
    
    1. `G1/1/1`: 10.0.0.54/30
        
    2. `G1/1/2`: 10.0.0.70/30
        
    3. `Loopback0`: 10.0.0.81/32
        
9. Configure las siguientes direcciones IP en `DSW-B2`:
    
    1. `G1/1/1`: 10.0.0.58/30
        
    2. `G1/1/2`: 10.0.0.74/30
        
    3. `Loopback0`: 10.0.0.82/32
        
10. Configure manualmente los parámetros IP de `SRV1`:
    
    1. Puerta de enlace predeterminada: 10.5.0.1
        
    2. Dirección IPv4: 10.5.0.4
        
    3. Máscara de subred: 255.255.255.0
        
11. Configure las IPs de administración en los switches de Acceso (`interface VLAN 99`), usando la primera dirección utilizable de cada subred como puerta de enlace predeterminada:
    
    1. `ASW-A1`: 10.0.0.4/28
        
    2. `ASW-A2`: 10.0.0.5/28
        
    3. `ASW-A3`: 10.0.0.6/28
        
    4. `ASW-B1`: 10.0.0.20/28
        
    5. `ASW-B2`: 10.0.0.21/28
        
    6. `ASW-B3`: 10.0.0.22/28
        
12. Configure HSRPv2 para la subred de Gestión (VLAN 99) en la Oficina A (`Grupo 1`). Haga a `DSW-A1` el router Activo elevando su prioridad a 5 unidades por encima del valor predeterminado y habilite la preempción:
    
    1. Subred: 10.0.0.0/28
        
    2. IP virtual (VIP): 10.0.0.1
        
    3. `DSW-A1`: 10.0.0.2
        
    4. `DSW-A2`: 10.0.0.3
        
13. Configure HSRPv2 para la subred de PCs (VLAN 10) en la Oficina A (`Grupo 2`). Haga a `DSW-A1` el router Activo como antes:
    
    1. Subred: 10.1.0.0/24
        
    2. VIP: 10.1.0.1
        
    3. `DSW-A1`: 10.1.0.2
        
    4. `DSW-A2`: 10.1.0.3
        
14. Configure HSRPv2 para la subred de Teléfonos (VLAN 20) en la Oficina A (`Grupo 3`). Haga a `DSW-A2` el router Activo subiendo su prioridad:
    
    1. Subred: 10.2.0.0/24
        
    2. VIP: 10.2.0.1
        
    3. `DSW-A1`: 10.2.0.2
        
    4. `DSW-A2`: 10.2.0.3
        
15. Configure HSRPv2 para la subred de Wi-Fi (VLAN 40) en la Oficina A (`Grupo 4`). Haga a `DSW-A2` el router Activo como antes:
    
    1. Subred: 10.6.0.0/24
        
    2. VIP: 10.6.0.1
        
    3. `DSW-A1`: 10.6.0.2
        
    4. `DSW-A2`: 10.6.0.3
        
16. Configure HSRPv2 para la subred de Gestión (VLAN 99) en la Oficina B (`Grupo 1`). Haga a `DSW-B1` el router Activo elevando su prioridad:
    
    1. Subred: 10.0.0.16/28
        
    2. VIP: 10.0.0.17
        
    3. `DSW-B1`: 10.0.0.18
        
    4. `DSW-B2`: 10.0.0.19
        
17. Configure HSRPv2 para la subred de PCs (VLAN 10) en la Oficina B (`Grupo 2`). Haga a `DSW-B1` el router Activo como antes:
    
    1. Subred: 10.3.0.0/24
        
    2. VIP: 10.3.0.1
        
    3. `DSW-B1`: 10.3.0.2
        
    4. `DSW-B2`: 10.3.0.3
        
18. Configure HSRPv2 para la subred de Teléfonos (VLAN 20) en la Oficina B (`Grupo 3`). Haga a `DSW-B2` el router Activo:
    
    1. Subred: 10.4.0.0/24
        
    2. VIP: 10.4.0.1
        
    3. `DSW-B1`: 10.4.0.2
        
    4. `DSW-B2`: 10.4.0.3
        
19. Configure HSRPv2 para la subred de Servidores (VLAN 30) en la Oficina B (`Grupo 4`). Haga a `DSW-B2` el router Activo:
    
    1. Subred: 10.5.0.0/24
        
    2. VIP: 10.5.0.1
        
    3. `DSW-B1`: 10.5.0.2
        
    4. `DSW-B2`: 10.5.0.3
### Configuración en **R1**

```bash
interface range gigabitEthernet 0/0/0,g0/1/0
ip address dhcp
no shutdown

interface gigabitEthernet 0/0
ip address 10.0.0.33 255.255.255.252
no shutdown

interface gigabitEthernet 0/1
ip address 10.0.0.37 255.255.255.252
no shutdown

interface loopback 0
ip address 10.0.0.76 255.255.255.255

```

### En **CSW1, CSW2, DSW-A1, DSW-A2, DSW-B1 y DSW-B2**

```bash
ip routing
```

### En ***CSW1

```bash
interface range gigabitEthernet 1/0/2-3
no switchport
channel-group 1 mode desirable

interface port-channel 1
ip address 10.0.0.41 255.255.255.252

Interface g1/0/1
No switchport
IP address 10.0.0.34 255.255.255.252
Interface g1/1/1
No switchport
IP address 10.0.0.45 255.255.255.252
Interface g1/1/2
No switchport
IP address 10.0.0.49 255.255.255.252
Interface g1/1/3
No switchport
IP address 10.0.0.53 255.255.255.252
Interface g1/1/4
No switchport
IP address 10.0.0.57 255.255.255.252
Interface loopback0
IP address 10.0.0.77 255.255.255.255
Interface range g1/0/4-24
Shutdown

```

### En ***CSW2

```bash
interface range gigabitEthernet 1/0/2-3
no switchport
channel-group 1 mode desirable

interface port-channel 1
ip address 10.0.0.42 255.255.255.252

Interface g1/0/1
No switchport
IP address 10.0.0.38 255.255.255.252
Interface g1/1/1
No switchport
IP address 10.0.0.61 255.255.255.252
Interface g1/1/2
No switchport
IP address 10.0.0.65 255.255.255.252
Interface g1/1/3
No switchport
IP address 10.0.0.69 255.255.255.252
Interface g1/1/4
No switchport
IP address 10.0.0.73 255.255.255.252
Interface loopback0
IP address 10.0.0.78 255.255.255.255
Interface range g1/0/4-24
Shutdown

```

### En ***DSW-A1

```BASH
Interface g1/1/1
No switchport
IP address 10.0.0.46 255.255.255.252
Interface g1/1/2
No switchport
IP address 10.0.0.62 255.255.255.252
Interface loopback 0
IP address 10.0.0.79 255.255.255.255
```

### En ***DSW-A2

```BASH
Interface g1/1/1
No switchport
IP address 10.0.0.50 255.255.255.252
Interface g1/1/2
No switchport
IP address 10.0.0.66 255.255.255.252
Interface loopback0
IP address 10.0.0.80 255.255.255.255
```

### En ***DSW-B1

```BASH
Interface g1/1/1
No switchport
IP address 10.0.0.54 255.255.255.252
Interface g1/1/2
No switchport
IP address 10.0.0.70 255.255.255.252
Interface loopback0
IP address 10.0.0.81 255.255.255.255
```

### En ***DSW-B2

```BASH
Interface g1/1/1
No switchport
IP address 10.0.0.58 255.255.255.252
Interface g1/1/2
No switchport
IP address 10.0.0.74 255.255.255.252
Interface loopback0
IP address 10.0.0.82 255.255.255.255
```

### Configure manualmente la IP de **SRV1:
a. Puerta de enlace predeterminada: 10.5.0.1
b. Dirección IPv4: 10.5.0.4
c. Máscara de subred: 255.255.255.0

### Configuración en **ASW-A1, ASW-A2 y ASW-A3**

```bash
ASW-A1
ip default-gateway 10.0.0.1
interface vlan 99
ip address 10.0.0.4 255.255.255.240
-----
ASW-A2
ip default-gateway 10.0.0.1
interface vlan 99
ip address 10.0.0.5 255.255.255.240

ASW-A3
ip default-gateway 10.0.0.1
interface vlan 99
ip address 10.0.0.6 255.255.255.240
```
### Configuración en **ASW-B1, ASW-B2 y ASW-B3**

```bash
ASW-B1
ip default-gateway 10.0.0.17
interface vlan 99
ip address 10.0.0.20 255.255.255.240
-----
ASW-B2
ip default-gateway 10.0.0.17
interface vlan 99
ip address 10.0.0.21 255.255.255.240

ASW-B3
ip default-gateway 10.0.0.17
interface vlan 99
ip address 10.0.0.22 255.255.255.240
```

### En ***DSW-A1

```BASH
interface vlan 99
ip address 10.0.0.2 255.255.255.240
standby version 2
standby 1 ip 10.0.0.1
standby 1 priority 105
standby 1 preempt

interface vlan 10
ip address 10.1.0.2 255.255.255.0
standby version 2
standby 2 ip 10.1.0.1
standby 2 priority 105
standby 2 preempt

interface vlan 20
ip address 10.2.0.2 255.255.255.0
standby version 2
standby 3 ip 10.2.0.1

interface vlan 40
ip address 10.6.0.2 255.255.255.0
standby version 2
standby 4 ip 10.6.0.1
```

### En ***DSW-A2

```BASH
interface vlan 99
ip address 10.0.0.3 255.255.255.240
standby version 2
standby 1 ip 10.0.0.1

interface vlan 10
ip address 10.1.0.3 255.255.255.0
standby version 2
standby 2 ip 10.1.0.1

interface vlan 20
ip address 10.2.0.3 255.255.255.0
standby version 2
standby 3 ip 10.2.0.1
standby 3 priority 105
standby 3 preempt

interface vlan 40
ip address 10.6.0.3 255.255.255.0
standby version 2
standby 4 ip 10.6.0.1
standby 4 priority 105
standby 4 preempt
```


### En **DSW-B1

```bash
Interface VLAN 99
IP address 10.0.0.18 255.255.255.240
Standby version 2
Standby 1 IP 10.0.0.17
Standby 1 priority 105
Standby 1 preempt

Interface VLAN 10
IP address 10.3.0.2 255.255.255.0
Standby version 2
Standby 2 IP 10.3.0.1
Standby 2 priority 105
Standby 2 preempt

Interface VLAN 20
IP address 10.4.0.2 255.255.255.0
Standby version 2
Standby 3 IP 10.4.0.1

Interface VLAN 30
IP address 10.5.0.2 255.255.255.0
Standby version 2
Standby 4 IP 10.5.0.1

```

### En ***DSW-B2

```bash
Interface VLAN 99
IP address 10.0.0.19 255.255.255.240
Standby version 2
Standby 1 IP 10.0.0.17

Interface VLAN 10
IP address 10.3.0.3 255.255.255.0
Standby version 2
Standby 2 IP 10.3.0.1

Interface VLAN 20
IP address 10.4.0.3 255.255.255.0
Standby version 2
Standby 3 IP 10.4.0.1
Standby 3 priority 105
Standby 3 Preempt

Interface VLAN 30
IP Address 10.5.0.3 255.255.255.0
Standby Version 2
Standby 4 IP 10.5.0.1
Standby 4 Priority 105
Standby 4 Preempt

```

## 🧩 **Configuración de Rapid Spanning Tree Protocol

1. Configure **Rapid PVST+** en todos los switches de Acceso y Distribución. 
	1. a. Asegúrese de que el **Root Bridge** para cada VLAN se alinee con el router **Activo de HSRP** configurando la prioridad STP más baja posible (**0**). 
	2. b. Configure el router **En Espera de HSRP** para cada VLAN con una prioridad STP un incremento por encima de la prioridad más baja (**4096**).
2. Habilite **PortFast** y **BPDU Guard** en todos los puertos conectados a hosts finales (incluyendo WLC1). Realice las configuraciones en modo de configuración de interfaz.
### En **DSW-A1,DSW-A2, DSW-B1, DSW-B2, ASW-A1, ASW-A2, ASW-A3, ASW-B1, ASW-B2 y ASW-B3

```BASH
show spanning-tree
spanning-tree mode rapid-pvst
```

### En ***DSW-A1

```BASH
spanning-tree vlan 10,99 priority 0
spanning-tree vlan 20,40 priority 4096
```

### En ***DSW-A2

```BASH
spanning-tree vlan 20,40 priority 0
spanning-tree vlan 10,99 priority 4096
```

### En ***DSW-B1

```BASH
spanning-tree vlan 10,99 priority 0
spanning-tree vlan 20,30 priority 4096
```

### En ***DSW-B2

```BASH
spanning-tree vlan 20,30 priority 0
spanning-tree vlan 10,99 priority 4096
```

### En ***ASW-A1

```BASH
interface fastEthernet 0/1
spanning-tree portfast
spanning-tree bpduguard enable

interface fastEthernet 0/2
spanning-tree portfast trunk
spanning-tree bpduguard enable
```
### En ***ASW-A2, ASW-A3, ASW-B1, ASW-B2 y ASW-B3

```BASH
interface fastEthernet 0/1
spanning-tree portfast
spanning-tree bpduguard enable
```



## 🧩 **Configuración de Static and Dynamic Routing

1. Configure **OSPF** en **R1** (interfaces con la LAN) y en todos los switches Core y de Distribución (todas las interfaces de Capa 3). 
	1. a. Use el ID de proceso **1** y el Área **0**. 
	2. b. Configure manualmente el **RID** de cada dispositivo para que coincida con la IP de la interfaz **loopback**. 
	3. c. En los switches, use el comando `network` para que coincida con la dirección IP **exacta** de cada interfaz. 
	4. d. En **R1**, habilite OSPF en modo de configuración de interfaz. 
	5. e. Asegúrese de que OSPF esté habilitado en todas las interfaces **loopback**. Las interfaces loopback deben ser **pasivas**. 
	6. f. Los **SVIs** de cada switch de Distribución (excepto el SVI de VLAN de Gestión) también deben ser **pasivos**. 
	7. g. Configure todas las conexiones físicas entre vecinos OSPF para usar un tipo de red que **no elija un DR/BDR** (`point-to-point`). _NOTA_: Esto no funciona en las interfaces Layer-3 PortChannel entre CSW1/CSW2. Déjelas con el tipo de red predeterminado.
    
2. Configure una **ruta estática predeterminada** para cada una de las conexiones a Internet de **R1**. Deben ser rutas **recursivas**.
	1. a. Haga que la ruta a través de `G0/1/0` sea una **ruta estática flotante** configurando un valor de Distancia Administrativa (**AD**) **1 mayor** que el predeterminado (1+1=**2**). 
	2. b. **R1** debe funcionar como un **ASBR de OSPF**, anunciando su ruta predeterminada a otros routers en el dominio OSPF (`default-information originate`).
### En ***R1

```BASH
router ospf 1
router-id 10.0.0.76
passive-interface loopback 0

interface loopback 0
ip ospf 1 area 0

interface range gigabitEthernet 0/0-1
ip ospf 1 area 0
ip ospf network point-to-point

--------------
ip route 0.0.0.0 0.0.0.0 203.0.113.1
ip route 0.0.0.0 0.0.0.0 203.0.113.5 2

router ospf 1
default-information originate
```

### En ***CSW1

```BASH
router ospf 1
router-id 10.0.0.77
passive-interface loopback 0
network 10.0.0.41 0.0.0.0 area 0
network 10.0.0.34 0.0.0.0 area 0
network 10.0.0.45 0.0.0.0 area 0
network 10.0.0.49 0.0.0.0 area 0
network 10.0.0.53 0.0.0.0 area 0
network 10.0.0.57 0.0.0.0 area 0
network 10.0.0.77 0.0.0.0 area 0

interface range gigabitEthernet 1/0/1,g1/1/1-4
ip ospf network point-to-point

```


### En ***CSW2

```BASH
router ospf 1
router-id 10.0.0.78
passive-interface loopback0
network 10.0.0.42 0.0.0.0 area 0
network 10.0.0.38 0.0.0.0 area 0
network 10.0.0.61 0.0.0.0 area 0
network 10.0.0.65 0.0.0.0 area 0
network 10.0.0.69 0.0.0.0 area 0
network 10.0.0.73 0.0.0.0 area 0
network 10.0.0.78 0.0.0.0 area 0

interface range gigabitEthernet 1/0/1,g1/1/1-4
ip ospf network point-to-point
```

### En ***DSW-A1

```BASH
router ospf 1
router-id 10.0.0.79
passive-interface loopback0
passive-interface vlan 10
passive-interface vlan 20
passive-interface vlan 40
network 10.0.0.46 0.0.0.0 area 0
network 10.0.0.62 0.0.0.0 area 0
network 10.0.0.79 0.0.0.0 area 0
network 10.1.0.2 0.0.0.0 area 0
network 10.2.0.2 0.0.0.0 area 0
network 10.0.0.2 0.0.0.0 area 0
network 10.6.0.2 0.0.0.0 area 0

interface range gigabitEthernet 1/1/1-2
ip ospf network point-to-point
```

### En ***DSW-A2

```BASH
Router OSPF 1
Router-ID 10.0.0.80
Passive-Interface Loopback 0
Passive-Interface VLAN 10
Passive-Interface VLAN 20
Passive-Interface VLAN 40
Network 10.0.0.50 0.0.0.0 Area 0
Network 10.0.0.66 0.0.0.0 Area 0
Network 10.0.0.80 0.0.0.0 Area 0
Network 10.1.0.3 0.0.0.0 Area 0
Network 10.2.0.3 0.0.0.0 Area 0
Network 10.0.0.3 0.0.0.0 Area 0
Network 10.6.0.3 0.0.0.0 Area 0

interface range gigabitEthernet 1/1/1-2
ip ospf network point-to-point
```

### En ***DSW-B1

```BASH
router ospf 1
router-id 10.0.0.81
passive-interface loopback0
passive-interface vlan 10
passive-interface vlan 20
passive-interface vlan 30
network 10.0.0.54 0.0.0.0 area 0
network 10.0.0.70 0.0.0.0 area 0
network 10.0.0.81 0.0.0.0 area 0
network 10.3.0.2 0.0.0.0 area 0
network 10.4.0.2 0.0.0.0 area 0
network 10.5.0.2 0.0.0.0 area 0
network 10.0.0.18 0.0.0.0 area 0

interface range gigabitEthernet 1/1/1-2
ip ospf network point-to-point
```

### En ***DSW-B2

```BASH
router OSPF 1
router-id 10.0.0.82
passive-interface loopback0
passive-interface vlan 10
passive-interface vlan 20
passive-interface vlan 30
network 10.0.0.58 0.0.0.0 area 0
network 10.0.0.74 0.0.0.0 area 0
network 10.0.0.82 0.0.0.0 area 0
network 10.3.0.3 0.0.0.0 area 0
network 10.4.0.3 0.0.0.0 area 0
network 10.5.0.3 0.0.0.0 area 0
network 10.0.0.19 0.0.0.0 area 0

interface range gigabitEthernet 1/1/1-2
ip ospf network point-to-point
```

## 🧩 **Configuración de Network Services: DHCP, DNS, NTP, SNMP, Syslog, FTP, SSH, NAT

1. Configure los siguientes **pools DHCP** en `R1` para que sirva como servidor DHCP para hosts en las Oficinas A y B. **Excluya las primeras diez direcciones host utilizables** de cada pool. 
	1. a. Pool: **A-Mgmt** (10.0.0.0/28, DG: 10.0.0.1, Domain: mialboratoriored.com, DNS: 10.5.0.4, WLC: **10.0.0.7**) 
	2. b. Pool: **A-PC** (10.1.0.0/24, DG: 10.1.0.1, Domain: mialboratoriored.com, DNS: 10.5.0.4) 
	3. c. Pool: **A-Phone** (10.2.0.0/24, DG: 10.2.0.1, Domain: mialboratoriored.com, DNS: 10.5.0.4) 
	4. d. Pool: **B-Mgmt** (10.0.0.16/28, DG: 10.0.0.17, Domain: mialboratoriored.com, DNS: 10.5.0.4, WLC: **10.0.0.7**) 
	5. e. Pool: **B-PC** (10.3.0.0/24, DG: 10.3.0.1, Domain: mialboratoriored.com, DNS: 10.5.0.4) 
	6. f. Pool: **B-Phone** (10.4.0.0/24, DG: 10.4.0.1, Domain: mialboratoriored.com, DNS: 10.5.0.4) 
	7. g. Pool: **Wi-Fi** (10.6.0.0/24, DG: 10.6.0.1, Domain: mialboratoriored.com, DNS: 10.5.0.4)
    
2. Configure los switches de Distribución para retransmitir los mensajes de difusión de los clientes DHCP cableados a la dirección IP de **Loopback0 de `R1`** (**ip helper-address 10.0.0.76**).
    
3. Configure las siguientes entradas DNS en **`SRV1`**: 
	1. a. google.com = 172.253.62.100 
	2. b. youtube.com = 152.250.31.93
	3. c.mialboratoriored.com=66.235.200.200
    
4. Configure todos los routers y switches para usar el **nombre de dominio mialboratoriored.com** y usar **`SRV1`** como su **servidor DNS** (**10.5.0.4**).
    
5. Configure **NTP** en **`R1`**: a. Convierta a `R1` en un **servidor NTP estrato 5** (**ntp master 5**). b. `R1` debe obtener la hora del servidor NTP **216.239.35.0**.
    
6. Todos los switches Core, Distribución y Acceso deben usar la interfaz **Loopback de `R1`** como su **servidor NTP** (**10.0.0.76**). a. Los clientes deben autenticar a `R1` usando la **clave número 1** y la contraseña **`ccna`**.
    
7. Configure la **cadena de comunidad SNMP `SNMPSTRING`** en todos los routers y switches. La cadena debe permitir mensajes **GET** (**ro**), pero no mensajes SET.
    
8. Configure **Syslog** en todos los routers y switches: 
	1. a. Envíe mensajes Syslog a **`SRV1`** (**10.5.0.4**). Se deben registrar mensajes de **todos los niveles de gravedad** (**logging trap debugging**). 
	2. b. Habilite el registro en el búfer. Reserve **8192 bytes** de memoria para el búfer.
    
9. Use **FTP** en `R1` para descargar una nueva versión de IOS desde `SRV1`: 
	1. a. Configure las credenciales FTP predeterminadas de `R1`: **usuario `cisco`, contraseña `cisco`**. 
	2. b. Use FTP para copiar el archivo **`c2900-universalk9-mz.SPA.155-3.M4a.bin`** de `SRV1` al disco flash de `R1`. 
	3. c. **Reinicie** `R1` usando el nuevo archivo IOS y luego **elimine el antiguo** del flash.
    
13. Configure **SSH** para acceso remoto seguro en todos los routers y switches. 
	1. a. Use el **tamaño de módulo más grande** para las claves RSA (**4096**). 
	2. b. Permita solo conexiones **SSHv2**. 
	3. c. Cree la **ACL estándar 1** para permitir solo paquetes originados en la **subred de PCs de la Oficina A** (**10.1.0.0/24**). Aplique la ACL a todas las líneas VTY para restringir el acceso SSH. 
	4. d. Permita solo conexiones **SSH** a las líneas VTY. 
	5. e. Requiera que los usuarios inicien sesión con una **cuenta de usuario local** al conectarse por SSH. f. Configure el **registro síncrono** en las líneas VTY.
    
14. Configure **NAT estático** en `R1` para permitir que los hosts en Internet accedan a `SRV1` a través de la dirección IP **203.0.113.113**.
    
15. Configure **PAT dinámico basado en pool** en `R1` para permitir que los hosts en las subredes PCs Oficina A, Teléfonos Oficina A, PCs Oficina B, Teléfonos Oficina B y Wi-Fi accedan a Internet. 
	1. a. Use la **ACL estándar 2** para definir los rangos de direcciones locales internas. b. Defina un rango de direcciones globales internas llamado **`POOL1`**, especificando el rango **203.0.113.200 a 203.0.113.207** con una máscara de red **/29** (**255.255.255.248**). c. Mapee la ACL 2 al POOL1 y habilite PAT (**overload**). d
	2. . Verifique que la conmutación por error del enlace a Internet funcione deshabilitando la interfaz `G0/0/0` de `R1` y haciendo ping nuevamente. (Esto requiere comandos `no default-information originate` y `default-information originate` para la conmutación por error en Packet Tracer).
    
16. **Deshabilite CDP** en todos los dispositivos y habilite **LLDP** en su lugar. a. **Deshabilite LLDP Tx** en el puerto de acceso de cada switch de Acceso (`F0/1`).
### En **R1

```BASH
ip domain-name mialboratoriored.com
ip name-server 10.5.0.4

ip dhcp excluded-address 10.0.0.1 10.0.0.10
ip dhcp excluded-address 10.1.0.1 10.1.0.10
ip dhcp excluded-address 10.2.0.1 10.2.0.10
ip dhcp excluded-address 10.0.0.17 10.0.0.26
ip dhcp excluded-address 10.3.0.1 10.3.0.10
ip dhcp excluded-address 10.4.0.1 10.4.0.10
ip dhcp excluded-address 10.6.0.1 10.6.0.10

ip dhcp pool A-Mgmt
network 10.0.0.0 255.255.255.240
default-router 10.0.0.1
domain-name mialboratoriored.com
dns-server 10.5.0.4
option 43 ip 10.0.0.7

ip dhcp pool A-PC
network 10.1.0.0 255.255.255.0
default-router 10.1.0.1
dns-server 10.5.0.4
domain-name mialboratoriored.com

ip dhcp pool A-Phone
network 10.2.0.0 255.255.255.0
default-router 10.2.0.1
dns-server 10.5.0.4
domain-name mialboratoriored.com

ip dhcp pool B-Mgmt
Network 10.0.0.16 255.255.255.240
default-router 10.0.0.17
dns-server 10.5.0.4
domain-name mialboratoriored.com
option 43 ip 10.0.0.7

ip dhcp pool B-PC
network 10.3.0.0 255.255.255.0
default-router 10.3.0.1
domain-name mialboratoriored.com
dns-server 10.5.0.4

ip dhcp pool B-Phone
network 10.4.0.0 255.255.255.0
default-router 10.4.0.1
domain-name mialboratoriored.com
dns-server 10.5.0.4

ip dhcp pool Wi-Fi
network 10.6.0.0 255.255.255.0
default-router 10.6.0.1
dns-server 10.5.0.4
domain-name mialboratoriored.com

-----------------


ntp master 5
ntp server 216.239.35.0
ntp authentication-key 1 md5 ccna
ntp trusted-key 1

snmp-server community SNMPSTRING ro

logging 10.5.0.4
logging trap debugging
logging buffered 8192

ip ftp username cisco
ip ftp password cisco

-------------------

do copy ftp flash
Address or name of remote host []? 10.5.0.4
Source filename []? c2900-universalk9-mz.SPA.155-3.M4a.bin

boot system flash:c2900-universalk9-mz.SPA.155-3.M4a.bin
do wr
do reload

delete flash:c2900-universalk9-mz.SPA.151-4.M4.bin

-----------------------

crypto key generate rsa
4096
ip ssh version 2
access-list 1 permit 10.1.0.0 0.0.0.255
line vty 0 15
access-class 1 in
transport input ssh
login local
logging synchronous

---------------------
ip nat inside source static 10.5.0.4 203.0.113.113
interface range gigabitEthernet 0/0/0,g0/1/0
ip nat outside
interface range gigabitEthernet 0/0-1
ip nat inside

access-list 2 permit 10.1.0.0 0.0.0.255
access-list 2 permit 10.2.0.0 0.0.0.255
access-list 2 permit 10.3.0.0 0.0.0.255
access-list 2 permit 10.4.0.0 0.0.0.255
access-list 2 permit 10.6.0.0 0.0.0.255
ip nat pool POOL1 203.0.113.200 203.0.113.207 netmask 255.255.255.248
ip nat inside source list 2 pool POOL1 overload

--------------
interface gigabitEthernet 0/0/0
shutdown

router ospf 1
no default-information originate
default-information originate

interface gigabitEthernet 0/0/0
no shutdown

router ospf 1
no default-information originate
default-information originate

no cdp run
lldp run

```

### En **DSW-A1 y DSW-A2

```BASH

interface vlan 10
ip helper-address 10.0.0.76
interface vlan 20
ip helper-address 10.0.0.76
interface vlan 40
ip helper-address 10.0.0.76
interface vlan 99
ip helper-address 10.0.0.76

```

### En **DSW-B1 y DSW-B2

```BASH

interface vlan 10
ip helper-address 10.0.0.76
interface vlan 20
ip helper-address 10.0.0.76
interface vlan 30
ip helper-address 10.0.0.76
interface vlan 99
ip helper-address 10.0.0.76

```

### **All DEVICES

```bash
ip domain-name mialboratoriored.com
ip name-server 10.5.0.4

ntp authentication-key 1 md5 ccna
ntp trusted-key 1
ntp server 10.0.0.76 key 1

snmp-server community SNMPSTRING ro

logging 10.5.0.4
logging trap debugging
logging buffered 8192


crypto key generate rsa
4096
ip ssh version 2
access-list 1 permit 10.1.0.0 0.0.0.255
line vty 0 15
access-class 1 in
transport input ssh
login local
logging synchronous

---------------router--------------
no cdp run
lldp run
--------------switches------------
no cdp run
lldp run
interface fastEthernet 0/1
no lldp transmit
```


## 🧩 **Configuración de Security: ACLs and Layer-2 Security Features

1. Configure la **ACL extendida `OfficeA_to_OfficeB`** donde sea apropiado: 
	1. a. Permita mensajes **ICMP** de la subred PCs Oficina A a la subred PCs Oficina B. 
	2. b. **Bloquee todo el demás tráfico** de la subred PCs Oficina A a la subred PCs Oficina B. 
	3. c. **Permita todo el demás tráfico** (**permit ip any any**). 
	4. d. Aplique la ACL de acuerdo con la mejor práctica general para ACL extendidas (**en `DSW-A1` en `VLAN 10` como `in`**).
    
2. Configure **Port Security** en el puerto `F0/1` de cada switch de Acceso: 
	1. a. Permita el número mínimo necesario de direcciones MAC en cada puerto (**maximum 1** para `ASW-A1, B1, B3`; **maximum 2** para `ASW-A2, A3, B2`). 
	2. b. Configure un modo de violación que bloquee el tráfico no válido sin afectar el tráfico válido (**violation restrict**). 
	3. c. Los switches deben guardar automáticamente las direcciones MAC seguras que aprenden en la running-config (**mac-address sticky**).
    
3. Configure **DHCP Snooping** en todos los switches de Acceso. 
	1. a. Habilítelo para todas las VLANs activas en cada LAN. 
	2. b. Confíe en los puertos apropiados (**trust** en enlaces troncales). 
	3. c. Deshabilite la inserción de la **Opción 82** de DHCP (**no ip dhcp snooping information option**). 
	4. d. Establezca un límite de tasa de DHCP de **15 pps** en los puertos no confiables activos (`F0/1`). 
	5. e. Establezca un límite más alto (**100 pps**) en la conexión de `ASW-A1` a `WLC1` (`F0/2`).
    
4. Configure **DAI** (**Dynamic ARP Inspection**) en todos los switches de Acceso.
	1. a. Habilítelo para todas las VLANs activas en cada LAN. 
	2. b. Confíe en los puertos apropiados (**trust** en enlaces troncales). 
	3. c. Habilite todas las comprobaciones de validación opcionales (**validate dst-mac src-mac ip**).
### En **DSW-A1 y DSW-A2

```bash

ip access-list extended OfficeA_to_OfficeB
permit icmp 10.1.0.0 0.0.0.255 10.3.0.0 0.0.0.255
deny ip 10.1.0.0 0.0.0.255 10.3.0.0 0.0.0.255
permit ip any any

interface vlan 10
ip access-group OfficeA_to_OfficeB in

```

### En **ASW-A1, ASW-B1 y ASW-B3

```bash

interface fastEthernet 0/1
switchport port-security
switchport port-security violation restrict 
switchport port-security mac-address sticky

```

### En **ASW-A2, ASW-A3 y ASW-B2

```bash

interface fastEthernet 0/1
switchport port-security
switchport port-security maximum 2
switchport port-security violation restrict 
switchport port-security mac-address sticky

```

### En **ASW-A1, ASW-A2 y ASW-A3

```bash

ip dhcp snooping
ip dhcp snooping vlan 10,20,40,99
no ip dhcp snooping information option
interface range gigabitEthernet 0/1-2
ip dhcp snooping trust
interface fastEthernet 0/1
ip dhcp snooping limit rate 15

----------ASW-A1-------------------
interface fastEthernet 0/2
ip dhcp snooping limit rate 100

```

### En **ASW-B1, ASW-B2 y ASW-B3

```bash

ip dhcp snooping
ip dhcp snooping vlan 10,20,30,99
no ip dhcp snooping information option
interface range gigabitEthernet 0/1-2
ip dhcp snooping trust
interface fastEthernet 0/1
ip dhcp snooping limit rate 15

```

### En **ASW-A1, ASW-A2 y ASW-A3

```bash
ip arp inspection vlan 10,20,40,99
ip arp inspection validate dst-mac src-mac ip
interface range gigabitEthernet 0/1-2
ip arp inspection trust
```

### En **ASW-B1, ASW-B2 y ASW-B3

```bash
ip arp inspection vlan 10,20,30,99
ip arp inspection validate dst-mac src-mac ip
interface range gigabitEthernet 0/1-2
ip arp inspection trust
```

## 🧩 **Configuración de IPv6

1. Para prepararse para una migración a IPv6, habilite el **enrutamiento IPv6** y configure las direcciones IPv6 en `R1`, `CSW1` y `CSW2`: 
	1. a. `R1` `G0/0/0`: **2001:db8:a::2/64** 
	2. b. `R1` `G0/1/0`: **2001:db8:b::2/64** 
	3. c. `R1` `G0/0` y `CSW1` `G1/0/1`: Use el prefijo **2001:db8:a1::/64** y **EUI-64**. 
	4. d. `R1` `G0/1` y `CSW2` `G1/0/1`: Use el prefijo **2001:db8:a2::/64** y **EUI-64**.
	5. e. `CSW1` `Po1` y `CSW2` `Po1`: **Habilite IPv6** sin usar el comando `ipv6 address` (**ipv6 enable**).
    
2. Configure **dos rutas estáticas predeterminadas** en `R1`: 
	1. a. Una ruta **recursiva** a través del siguiente salto **2001:db8:a::1**. 
	2. b. Una ruta **completamente especificada** a través del siguiente salto **2001:db8:b::1**. Conviértala en una **ruta flotante** configurando la AD 1 más alta que la predeterminada (**AD 2**).
### En **R1

```BASH
ipv6 unicast-routing
interface gigabitEthernet 0/0/0
ipv6 address 2001:db8:a::2/64

interface gigabitEthernet 0/1/0
ipv6 address 2001:db8:b::2/64

interface gigabitEthernet 0/1
ipv6 address 2001:db8:a2::/64 eui-64

interface gigabitEthernet 0/0
ipv6 address 2001:db8:a1::/64 eui-64

ipv6 route ::/0 2001:db8:a::1
ipv6 route ::/0 g0/1/0 2001:db8:b::1 2

```

### En **CSW1 y CSW2

```BASH
----------CSW1------------------
ipv6 unicast-routing
interface gigabitEthernet 1/0/1
ipv6 address 2001:db8:a1::/64 eui-64
----------CSW2------------------
ipv6 unicast-routing
interface gigabitEthernet 1/0/1
ipv6 address 2001:db8:a2::/64 eui-64
-----------------------


interface port-channel 1
ipv6 enable

```

## 🧩 **Configuración de Wireless

1. Acceda a la **GUI de `WLC1`** ([https://10.0.0.7](https://10.0.0.7)) desde una de las PCs. a. Usuario: **admin** b. Contraseña: **adminPW12**
    
2. Configure una **interfaz dinámica** para la WLAN Wi-Fi (10.6.0.0/24) 
	1. a. Nombre: **Wi-Fi** 
	2. b. VLAN: **40** 
	3. c. Número de puerto: **1** 
	4. d. Dirección IP: **10.6.0.4** (Nota: el video usa 10.6.0.2, que es un duplicado, así que se usa la siguiente dirección utilizable). 
	5. e. Puerta de enlace: **10.6.0.1** 
	6. f. Servidor DHCP: **10.0.0.76**
    
3. Configure y habilite la siguiente **WLAN**: 
	1. a. Nombre de perfil: **Wi-Fi** 
	2. b. SSID: **Wi-Fi** 
	3. c. ID: **1** 
	4. d. Estado: **Habilitado** 
	5. e. Seguridad: **WPA2 Policy** con cifrado **AES**, **PSK** de **`cisco123`**.
    
4. Verifique que ambos LWAPs se hayan asociado con `WLC1`.
### En **WLC1
#### Interface
```
Wi-Fi
vlan:40

port n: 1

Interface Address
40
10.6.0.2
255.255.255.0
10.6.0.1

DHCP Information
10.0.0.76

```
#### **WLANs

```
Wi-Fi
Wi-Fi


```
