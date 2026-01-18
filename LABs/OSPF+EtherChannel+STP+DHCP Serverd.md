Objetivo del lab:
Simular una red real con alta disponibilidad, enrutamiento dinámico, redundancia de enlaces y servicios centralizados, validando conectividad entre VLANs y rutas OSPF.

<img width="1330" height="725" alt="lab cisco 1" src="https://github.com/user-attachments/assets/2e9809a8-9cad-4f74-9fd1-3e818c982f0e" />

-------------------------------------------------------------------
-------------------------------------------------------------------
PARTE 1   CREACION DE VLANs
-------------------------------------------------------------------
-------------------------------------------------------------------

SW SERVER
```
enable
config terminal
vtp domain lab-ccna
vtp mode server
vtp password cisco123

vlan 10
 name equipo10
vlan 15
 name equipo15
vlan 20
 name equipo20
```
-----------------------------------
SW 1
```
enable
config terminal
vtp domain lab-ccna
vtp mode client
vtp password cisco123

interface f0/10
sw mode access
sw access vlan 10
exit
interface f0/15
sw mode access
sw access vlan 15
exit
interface f0/20
sw mode access
sw access vlan 20
exit
```
-----------------------------------
SW 2
```
enable
config terminal
vtp domain lab-ccna
vtp mode client
vtp password cisco123

interface f0/10
sw mode access
sw access vlan 10
exit
interface f0/15
sw mode access
sw access vlan 15
exit
interface f0/20
sw mode access
sw access vlan 20
exit
```

-------------------------------------------------------------------
-------------------------------------------------------------------
PARTE 2   CREACION DE ETHERCHANNEL
-------------------------------------------------------------------
-------------------------------------------------------------------

SW 1
```
interface range f0/1-3
channel-group 1 mode desirable
exit
interface port-channel 1
sw mode trunk

interface range f0/4-6
channel-group 3 mode active
exit
interface port-channel 3
sw mode trunk
```
-----------------------------------
SW SERVER
```
interface range f0/4-6
channel-group 1 mode auto
exit
interface port-channel 1
sw mode trunk

interface range f0/1-3
channel-group 2 mode active
exit
interface port-channel 2
sw mode trunk
```
-----------------------------------
SW 2
```
interface range f0/4-6
channel-group 2 mode passive
exit
interface port-channel 2
sw mode trunk

interface range f0/1-3
channel-group 3 mode passive
exit
interface port-channel 3
sw mode trunk
```

-------------------------------------------------------------------
-------------------------------------------------------------------
PARTE 3   CREACION STP
-------------------------------------------------------------------
-------------------------------------------------------------------


SW SERVER
```
spanning-tree vlan 1,10,15,20 priority 4096
spanning-tree vlan 1 priority 4096
```
-----------------------------------
SW 1
```
spanning-tree portfast default
inter f0/10
spanning-tree BPDUGUARD enable
exit
inter f0/15
spanning-tree BPDUGUARD enable
exit
inter f0/20
spanning-tree BPDUGUARD enable
exit
```
-----------------------------------

SW 2
```
spanning-tree portfast default
inter f0/10
spanning-tree BPDUGUARD enable
exit
inter f0/15
spanning-tree BPDUGUARD enable
exit
inter f0/20
spanning-tree BPDUGUARD enable
exit
```

-------------------------------------------------------------------
-------------------------------------------------------------------
PARTE 4   CONFIGURACION TRUNK DE SW SERVER A ROUTER BORDE
-------------------------------------------------------------------
-------------------------------------------------------------------

SW SERVER
```
enable
config terminal
inter G0/1
sw mode trunk
```
-----------------------------------

ROUTER BORDE
```
enable
config terminal
interface G0/0
no shut
ip add 10.10.10.1 255.255.255.252
exit

interface g0/0.10
encapsulation dot1q 10
ip add 192.168.10.1 255.255.255.0
ip helper-address 192.168.60.100
exit
interface g0/0.15
encapsulation dot1q 15
ip add 192.168.15.1 255.255.255.0
ip helper-address 192.168.60.100
exit
interface g0/0.20
encapsulation dot1q 20
ip add 192.168.20.1 255.255.255.0
ip helper-address 192.168.60.100
exit
```
-------------------------------------------------------------------
-------------------------------------------------------------------
PARTE 5   OSPF
-------------------------------------------------------------------
-------------------------------------------------------------------

ROUTER BORDE:

(OSPF se puede hacer de 2 maneras diferentes de configuracion)

METODO 1:
```
router ospf 1
 router-id 1.1.1.1
 passive-interface default
 no passive-interface g0/1
 no passive-interface g0/2
 network 10.10.11.0 0.0.0.3 area 0
 network 10.10.12.0 0.0.0.3 area 0
```

METODO 2: MAS RECOMENDABLE:
```
enable
conf t
router ospf 1
router-id 1.1.1.1
passive-interface g0/0
passive-interface g0/0.10
passive-interface g0/0.15
passive-interface g0/0.20
end

interface g0/1
 ip address 10.10.11.1 255.255.255.252
 ip ospf network point-to-point
 ip ospf 1 area 0
 ip ospf cost 100
 no shutdown

interface g0/2
 ip address 10.10.12.1 255.255.255.252
 ip ospf network point-to-point
 ip ospf 1 area 0
 ip ospf cost 150
 no shutdown

interface g0/0
 ip ospf network point-to-point
 ip ospf 1 area 0
 no shutdown

interface g0/0.10
 ip ospf 1 area 0
 exit

interface g0/0.15
 ip ospf 1 area 0
 exit

interface g0/0.20
 ip ospf 1 area 0
 exit
 ```


ROUTER CENTRAL:
```
enable
conf t
router ospf 1
router-id 2.2.2.2
end

interface g0/0
 ip address 10.10.11.2 255.255.255.252
 ip ospf network point-to-point
 ip ospf 1 area 0
 ip ospf cost 100
 no shutdown

interface g0/1
 ip address 10.10.13.2 255.255.255.252
 ip ospf network point-to-point
 ip ospf 1 area 0
 ip ospf cost 100
 no shutdown
```
-----------------------------------

ROUTER BACKUP:
```
enable
conf t
router ospf 1
router-id 3.3.3.3
end

interface g0/0
 ip address 10.10.12.2 255.255.255.252
 ip ospf network point-to-point
 ip ospf 1 area 0
 ip ospf cost 150
 no shutdown

interface g0/1
 ip address 10.10.14.2 255.255.255.252
 ip ospf network point-to-point
 ip ospf 1 area 0
 ip ospf cost 150
 no shutdown
```
-----------------------------------

ROUTER BORDE 2
```
enable
conf t
router ospf 1
router-id 4.4.4.4
passive-interface g0/2
passive-interface g0/2.40
passive-interface g0/2.50
passive-interface g0/2.60
end

interface g0/1
 ip address 10.10.13.1 255.255.255.252
 ip ospf network point-to-point
 ip ospf 1 area 0
 ip ospf cost 100
 no shutdown

interface g0/0
 ip address 10.10.14.1 255.255.255.252
 ip ospf network point-to-point
 ip ospf 1 area 0
 ip ospf cost 150
 no shutdown

interface g0/2
 ip ospf network point-to-point
 ip ospf 1 area 0
 no shutdown

interface g0/0.40
 ip ospf 1 area 0
 exit
interface g0/0.50
 ip ospf 1 area 0
 exit
interface g0/0.60
 ip ospf 1 area 0
 exit
```

-------------------------------------------------------------------
-------------------------------------------------------------------
PARTE 6   CONFIGURACION RED DE SERVIDOR DHCP
-------------------------------------------------------------------
-------------------------------------------------------------------

ROUTER BORDE 2
```
enable
config ter
inter g0/2
ip add 172.17.20.1 255.255.255.252
ip helper-address 192.168.60.100
no shut
exit

interface g0/2.40
encapsulation dot1q 40
ip add 192.168.40.1 255.255.255.0
ip helper-address 192.168.60.100
exit
interface g0/2.50
encapsulation dot1q 50
ip add 192.168.50.1 255.255.255.0
ip helper-address 192.168.60.100
exit
interface g0/2.60
encapsulation dot1q 60
ip add 192.168.60.1 255.255.255.0
ip helper-address 192.168.60.100
exit
```
NOTA: Hacer lo mismo en cada interfaz logica del Router BORDE.

-----------------------------------

SW CENTRAL
```
Inter g0/1
sw mode trunk
exit

Vlan 40
name vlan40
Vlan 50
name vlan50
Vlan 60
name dhcp-server

interface f0/5
sw mode access
sw access vlan 50
exit

interface f0/4
sw mode access
sw access vlan 40
exit

interface f0/1
sw mode access
sw access vlan 60
exit
```


