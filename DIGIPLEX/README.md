# Déploiement DIGIPLEX

## Table des matières
- [Plan d'adressage (Calcul VLSM)](#plan-dadressage-calcul-vlsm)
- [Configuration du Routeur](#configuration-du-routeur)
- [Configuration du Switch L3](#configuration-du-switch-l3)
- [Configuration des Switchs du RDC](#configuration-des-switchs-du-rdc)
- [Configuration des Switchs de l'Étage 1](#configuration-des-switchs-de-létage-1)
- [Configuration des Switchs de l'Étage 2](#configuration-des-switchs-de-létage-2)
- [Configuration des Switchs de l'Étage 3](#configuration-des-switchs-de-létage-3)
- [Configuration du Serveur DHCP](#configuration-du-serveur-dhcp)
- [Configuration du Contrôleur WiFi](#configuration-du-contrôleur-wifi)

---

## Plan d'adressage (Calcul VLSM)

Avant de toucher au matériel, nous devons définir les plages IP. Nous partons du réseau de base 192.168.0.0 et nous le découpons selon les besoins pour optimiser l'espace.

| Réseau | Masque | Intervalle hôtes | Sous-réseau | Broadcast |
|--------|--------|------------------|-------------|-----------|
| Routeur vers SWL3 | /30 | 192.168.0.1 - 192.168.0.2 | 192.168.0.0 | 192.168.0.3 |
| VLAN 10 « Conception » | /24 | 192.168.10.1 - 192.168.10.254 | 192.168.10.0 | 192.168.10.255 |
| VLAN 20 « Commercial » | /24 | 192.168.20.1 - 192.168.20.254 | 192.168.20.0 | 192.168.20.255 |
| VLAN 30 « Ressources_Humaine » | /24 | 192.168.30.1 - 192.168.30.254 | 192.168.30.0 | 192.168.30.255 |
| VLAN 40 « Hotline » | /24 | 192.168.40.1 - 192.168.40.254 | 192.168.40.0 | 192.168.40.255 |
| VLAN 50 « Wifi_Entreprise » | /24 | 192.168.50.1 - 192.168.50.254 | 192.168.50.0 | 192.168.50.255 |
| VLAN 60 « Wifi_invités » | /24 | 192.168.60.1 - 192.168.60.254 | 192.168.60.0 | 192.168.60.255 |
| VLAN 70 « Server » | /24 | 192.168.70.1 - 192.168.70.254 | 192.168.70.0 | 192.168.70.255 |
| VLAN 80 « Management » | /24 | 192.168.80.1 - 192.168.80.254 | 192.168.80.0 | 192.168.80.255 |

---

## Configuration du Routeur

### Configuration du nom du routeur
```cisco
Router>enable
Router#conf t
Router(config)#hostname Routeur_DIGIPLEX
```

### Configuration des mots de passe
```cisco
Routeur_DIGIPLEX(config)#enable secret password_routeur
Routeur_DIGIPLEX(config)#service password-encryption
Routeur_DIGIPLEX(config)#line console 0
Routeur_DIGIPLEX(config-line)#password password_routeur
Routeur_DIGIPLEX(config-line)#login
Routeur_DIGIPLEX(config-line)#exit
```

### Configuration interface vers le switch L3
```cisco
Routeur_DIGIPLEX(config)#interface Fa0/0
Routeur_DIGIPLEX(config-if)#ip address 192.168.0.1 255.255.255.252
Routeur_DIGIPLEX(config-if)#no shutdown
Routeur_DIGIPLEX(config-if)#ip nat inside
Routeur_DIGIPLEX(config-if)#exit
```

### Configuration de l'interface WAN et NAT
```cisco
Routeur_DIGIPLEX(config)#interface G0/0/0
Routeur_DIGIPLEX(config-if)#ip address dhcp
Routeur_DIGIPLEX(config-if)#no shutdown
Routeur_DIGIPLEX(config-if)#ip nat outside
Routeur_DIGIPLEX(config-if)#exit
Routeur_DIGIPLEX(config)#ip route 192.168.0.0 255.255.0.0 192.168.0.2
Routeur_DIGIPLEX(config)#access-list 1 permit 192.168.0.0 0.0.255.255
Routeur_DIGIPLEX(config)#ip nat inside source list 1 interface g0/0/0 overload
```

---

## Configuration du Switch L3

### Configuration du nom
```cisco
Switch>enable
Switch#conf t
Switch(config)#hostname SwitchL3_DIGIPLEX
```

### Configuration des mots de passe
```cisco
SwitchL3_DIGIPLEX(config)#enable secret password_switch
SwitchL3_DIGIPLEX(config)#service password-encryption
SwitchL3_DIGIPLEX(config)#line console 0
SwitchL3_DIGIPLEX(config-line)#password password_switch
SwitchL3_DIGIPLEX(config-line)#login
SwitchL3_DIGIPLEX(config-line)#exit
```

### Configuration de la connexion en SSH
```cisco
SwitchL3_DIGIPLEX(config)#ip domain-name digiplex.funky
SwitchL3_DIGIPLEX(config)#username admin secret password_switch_ssh
SwitchL3_DIGIPLEX(config)#crypto key generate rsa
```

### Application du SSH sur les lignes virtuelles
```cisco
SwitchL3_DIGIPLEX(config)#line vty 0 15
SwitchL3_DIGIPLEX(config-line)#transport input ssh
SwitchL3_DIGIPLEX(config-line)#login local
SwitchL3_DIGIPLEX(config-line)#exit
```

### Mise en place du protocole VTP

Propager les VLANs sur les switch L2.
```cisco
SwitchL3_DIGIPLEX(config)#vtp mode server
SwitchL3_DIGIPLEX(config)#vtp domain DIGIPLEX
SwitchL3_DIGIPLEX(config)#vtp password password_vtp
```

### Configuration de l'Etherchannel entre les étages et les switch RDC

#### SWL2 02
```cisco
SwitchL3_DIGIPLEX(config)#interface range Fa0/3-4
SwitchL3_DIGIPLEX(config-if-range)#channel-group 2 mode active
SwitchL3_DIGIPLEX(config-if-range)#no shutdown

SwitchL3_DIGIPLEX(config)#interface port-channel 2
SwitchL3_DIGIPLEX(config-if)#switchport trunk encapsulation dot1q
SwitchL3_DIGIPLEX(config-if)#switchport mode trunk
SwitchL3_DIGIPLEX(config-if)#no shutdown
```

#### SWL2 03
```cisco
SwitchL3_DIGIPLEX(config)#interface range Fa0/5-6
SwitchL3_DIGIPLEX(config-if-range)#channel-group 3 mode active
SwitchL3_DIGIPLEX(config-if-range)#no shutdown

SwitchL3_DIGIPLEX(config)#interface port-channel 3
SwitchL3_DIGIPLEX(config-if)#switchport trunk encapsulation dot1q
SwitchL3_DIGIPLEX(config-if)#switchport mode trunk
SwitchL3_DIGIPLEX(config-if)#no shutdown
```

#### Étage 1
```cisco
SwitchL3_DIGIPLEX(config)#interface range Fa0/11-12
SwitchL3_DIGIPLEX(config-if-range)#channel-group 11 mode active
SwitchL3_DIGIPLEX(config-if-range)#no shutdown

SwitchL3_DIGIPLEX(config)#interface port-channel 11
SwitchL3_DIGIPLEX(config-if)#switchport trunk encapsulation dot1q
SwitchL3_DIGIPLEX(config-if)#switchport mode trunk
SwitchL3_DIGIPLEX(config-if)#no shutdown
```

#### Étage 2
```cisco
SwitchL3_DIGIPLEX(config)#interface range Fa0/1-2
SwitchL3_DIGIPLEX(config-if-range)#channel-group 21 mode active
SwitchL3_DIGIPLEX(config-if-range)#no shutdown

SwitchL3_DIGIPLEX(config)#interface port-channel 21
SwitchL3_DIGIPLEX(config-if)#switchport trunk encapsulation dot1q
SwitchL3_DIGIPLEX(config-if)#switchport mode trunk
SwitchL3_DIGIPLEX(config-if)#no shutdown
```

#### Étage 3
```cisco
SwitchL3_DIGIPLEX(config)#interface range Fa0/9-10
SwitchL3_DIGIPLEX(config-if-range)#channel-group 31 mode active
SwitchL3_DIGIPLEX(config-if-range)#no shutdown

SwitchL3_DIGIPLEX(config)#interface port-channel 31
SwitchL3_DIGIPLEX(config-if)#switchport trunk encapsulation dot1q
SwitchL3_DIGIPLEX(config-if)#switchport mode trunk
SwitchL3_DIGIPLEX(config-if)#no shutdown
```

### Configuration des VLANs
```cisco
SwitchL3_DIGIPLEX(config)#vlan 10
SwitchL3_DIGIPLEX(config)#name CONCEPTION
SwitchL3_DIGIPLEX(config)#vlan 20
SwitchL3_DIGIPLEX(config)#name COMMERCIEL
SwitchL3_DIGIPLEX(config)#vlan 30
SwitchL3_DIGIPLEX(config)#name RESSOURCES_HUMAINES
SwitchL3_DIGIPLEX(config)#vlan 40
SwitchL3_DIGIPLEX(config)#name HOTLINE
SwitchL3_DIGIPLEX(config)#vlan 50
SwitchL3_DIGIPLEX(config)#name WIFI_ENTREPRISE
SwitchL3_DIGIPLEX(config)#vlan 60
SwitchL3_DIGIPLEX(config)#name WIFI_INVITES
SwitchL3_DIGIPLEX(config)#vlan 70
SwitchL3_DIGIPLEX(config)#name SERVER
SwitchL3_DIGIPLEX(config)#vlan 80
SwitchL3_DIGIPLEX(config)#name MANAGEMENT
```

### Configuration inter-VLAN
```cisco
SwitchL3_DIGIPLEX(config)#ip routing
```

#### VLAN 10
```cisco
SwitchL3_DIGIPLEX(config)#interface vlan 10
SwitchL3_DIGIPLEX(config-if)#ip address 192.168.10.254 255.255.255.0
SwitchL3_DIGIPLEX(config-if)#ip helper-address 192.168.70.1
SwitchL3_DIGIPLEX(config-if)#no shutdown
```

#### VLAN 20
```cisco
SwitchL3_DIGIPLEX(config)#interface vlan 20
SwitchL3_DIGIPLEX(config-if)#ip address 192.168.20.254 255.255.255.0
SwitchL3_DIGIPLEX(config-if)#ip helper-address 192.168.70.1
SwitchL3_DIGIPLEX(config-if)#no shutdown
```

#### VLAN 30
```cisco
SwitchL3_DIGIPLEX(config)#interface vlan 30
SwitchL3_DIGIPLEX(config-if)#ip address 192.168.30.254 255.255.255.0
SwitchL3_DIGIPLEX(config-if)#ip helper-address 192.168.70.1
SwitchL3_DIGIPLEX(config-if)#no shutdown
```

#### VLAN 40
```cisco
SwitchL3_DIGIPLEX(config)#interface vlan 40
SwitchL3_DIGIPLEX(config-if)#ip address 192.168.40.254 255.255.255.0
SwitchL3_DIGIPLEX(config-if)#ip helper-address 192.168.70.1
SwitchL3_DIGIPLEX(config-if)#no shutdown
```

#### VLAN 50
```cisco
SwitchL3_DIGIPLEX(config)#interface vlan 50
SwitchL3_DIGIPLEX(config-if)#ip address 192.168.50.254 255.255.255.0
SwitchL3_DIGIPLEX(config-if)#ip helper-address 192.168.70.1
SwitchL3_DIGIPLEX(config-if)#no shutdown
```

#### VLAN 60
```cisco
SwitchL3_DIGIPLEX(config)#interface vlan 60
SwitchL3_DIGIPLEX(config-if)#ip address 192.168.60.254 255.255.255.0
SwitchL3_DIGIPLEX(config-if)#ip helper-address 192.168.70.1
SwitchL3_DIGIPLEX(config-if)#no shutdown
```

#### VLAN 70
```cisco
SwitchL3_DIGIPLEX(config)#interface vlan 70
SwitchL3_DIGIPLEX(config-if)#ip address 192.168.70.254 255.255.255.0
SwitchL3_DIGIPLEX(config-if)#no shutdown
```

#### VLAN 80
```cisco
SwitchL3_DIGIPLEX(config)#interface vlan 80
SwitchL3_DIGIPLEX(config-if)#ip address 192.168.80.254 255.255.255.0
SwitchL3_DIGIPLEX(config-if)#no shutdown
```

### Connexion vers le routeur Internet
```cisco
SwitchL3_DIGIPLEX(config)#interface Fa0/8
SwitchL3_DIGIPLEX(config-if)#no switchport 
SwitchL3_DIGIPLEX(config-if)#ip address 192.168.0.2 255.255.255.252
SwitchL3_DIGIPLEX(config-if)#no shutdown
SwitchL3_DIGIPLEX(config)#ip route 0.0.0.0 0.0.0.0 192.168.0.1
```

### Configuration des interfaces vers les serveurs
```cisco
SwitchL3_DIGIPLEX(config)#interface range Fa0/20-24
SwitchL3_DIGIPLEX(config-if)#switchport mode access
SwitchL3_DIGIPLEX(config-if)#switchport access vlan 70

SwitchL3_DIGIPLEX(config)#interface Fa0/7
SwitchL3_DIGIPLEX(config-if)#switchport trunk encapsulation dot1q
SwitchL3_DIGIPLEX(config-if)#switchport mode trunk
SwitchL3_DIGIPLEX(config-if)#switchport trunk native vlan 70
```

---

## Configuration des Switchs du RDC

### SW_RDC_02

#### Configuration du nom
```cisco
Switch>enable
Switch#conf t
Switch(config)#hostname SW_RDC_02
```

#### Mise en place de la sécurité
```cisco
SW_RDC_02(config)#enable secret password_switch
SW_RDC_02(config)#service password-encryption
SW_RDC_02(config)#line console 0
SW_RDC_02(config-line)#password password_switch
SW_RDC_02(config-line)#login
SW_RDC_02(config-line)#exit
```

#### Mise en place du VTP Client
```cisco
SW_RDC_02(config)#vtp mode client
SW_RDC_02(config)#vtp domain DIGIPLEX
SW_RDC_02(config)#vtp password password_vtp
```

#### Configuration de la connexion à distance en SSH
```cisco
SW_RDC_02(config)#ip domain-name digiplex.funky
SW_RDC_02(config)#crypto key generate rsa
SW_RDC_02(config)#username admin secret password_switch_ssh
SW_RDC_02(config)#line vty 0 15
SW_RDC_02(config-line)#transport input ssh
SW_RDC_02(config-line)#login local
SW_RDC_02(config-line)#exit
```

#### Configuration de l'IP de management
```cisco
SW_RDC_02(config)#interface vlan 80
SW_RDC_02(config-if)#ip address 192.168.80.2 255.255.255.0
SW_RDC_02(config-if)#no shutdown
SW_RDC_02(config)#ip default-gateway 192.168.80.254
```

#### Configuration de l'EtherChannel vers le switch L3
```cisco
SW_RDC_02(config)#interface range fa0/1-2
SW_RDC_02(config-if-range)#channel-group 1 mode active 
SW_RDC_02(config-if-range)#no shutdown

SW_RDC_02(config)#interface port-channel 1
SW_RDC_02(config-if)#switchport mode trunk
```

#### Configuration des ports utilisés par les clients

**VLAN 10 - CONCEPTION**
```cisco
SW_RDC_02(config)#interface range Fa0/3-6
SW_RDC_02(config-if)#switchport mode access
SW_RDC_02(config-if)#switchport access vlan 10
```

---

### SW_RDC_03

#### Configuration du nom
```cisco
Switch>enable
Switch#conf t
Switch(config)#hostname SW_RDC_03
```

#### Mise en place de la sécurité
```cisco
SW_RDC_03(config)#enable secret password_switch
SW_RDC_03(config)#service password-encryption
SW_RDC_03(config)#line console 0
SW_RDC_03(config-line)#password password_switch
SW_RDC_03(config-line)#login
SW_RDC_03(config-line)#exit
```

#### Mise en place du VTP Client
```cisco
SW_RDC_03(config)#vtp mode client
SW_RDC_03(config)#vtp domain DIGIPLEX
SW_RDC_03(config)#vtp password password_vtp
```

#### Configuration de la connexion à distance en SSH
```cisco
SW_RDC_03(config)#ip domain-name digiplex.funky
SW_RDC_03(config)#crypto key generate rsa
SW_RDC_03(config)#username admin secret password_switch_ssh
SW_RDC_03(config)#line vty 0 15
SW_RDC_03(config-line)#transport input ssh
SW_RDC_03(config-line)#login local
SW_RDC_03(config-line)#exit
```

#### Configuration de l'IP de management
```cisco
SW_RDC_03(config)#interface vlan 80
SW_RDC_03(config-if)#ip address 192.168.80.3 255.255.255.0
SW_RDC_03(config-if)#no shutdown
SW_RDC_03(config)#ip default-gateway 192.168.80.254
```

#### Configuration de l'EtherChannel vers le switch L3
```cisco
SW_RDC_03(config)#interface range fa0/1-2
SW_RDC_03(config-if-range)#channel-group 1 mode active 
SW_RDC_03(config-if-range)#no shutdown

SW_RDC_03(config)#interface port-channel 1
SW_RDC_03(config-if)#switchport mode trunk
```

#### Configuration des ports utilisés par les clients

**VLAN 20 - COMMERCIAL**
```cisco
SW_RDC_03(config)#interface range Fa0/3-6
SW_RDC_03(config-if)#switchport mode access
SW_RDC_03(config-if)#switchport access vlan 20
```

---

## Configuration des Switchs de l'Étage 1

### SW_ET1_01

#### Configuration du nom
```cisco
Switch>enable
Switch#conf t
Switch(config)#hostname SW_ET1_01
```

#### Mise en place de la sécurité
```cisco
SW_ET1_01(config)#enable secret password_switch
SW_ET1_01(config)#service password-encryption
SW_ET1_01(config)#line console 0
SW_ET1_01(config-line)#password password_switch
SW_ET1_01(config-line)#login
SW_ET1_01(config-line)#exit
```

#### Mise en place du VTP Client
```cisco
SW_ET1_01(config)#vtp mode client
SW_ET1_01(config)#vtp domain DIGIPLEX
SW_ET1_01(config)#vtp password password_vtp
```

#### Configuration de la connexion à distance en SSH
```cisco
SW_ET1_01(config)#ip domain-name digiplex.funky
SW_ET1_01(config)#crypto key generate rsa
SW_ET1_01(config)#username admin secret password_switch_ssh
SW_ET1_01(config)#line vty 0 15
SW_ET1_01(config-line)#transport input ssh
SW_ET1_01(config-line)#login local
SW_ET1_01(config-line)#exit
```

#### Configuration de l'IP de management
```cisco
SW_ET1_01(config)#interface vlan 80
SW_ET1_01(config-if)#ip address 192.168.80.11 255.255.255.0
SW_ET1_01(config-if)#no shutdown
SW_ET1_01(config)#ip default-gateway 192.168.80.254
```

#### Configuration de l'EtherChannel vers le switch L3 du RDC
```cisco
SW_ET1_01(config)#interface range fa0/11-12
SW_ET1_01(config-if-range)#channel-group 1 mode active
SW_ET1_01(config-if-range)#no shutdown
SW_ET1_01(config-if-range)#exit
SW_ET1_01(config)#interface port-channel 1
SW_ET1_01(config-if)#switchport mode trunk
SW_ET1_01(config-if)#exit
```

#### Configuration de l'EtherChannel vers les switchs de l'étage 1

**Vers SW_ET1_03**
```cisco
SW_ET1_01(config)#interface range fa0/1-2
SW_ET1_01(config-if-range)#channel-group 3 mode active
SW_ET1_01(config-if-range)#no shutdown
SW_ET1_01(config-if-range)#exit
SW_ET1_01(config)#interface port-channel 3
SW_ET1_01(config-if)#switchport mode trunk
SW_ET1_01(config-if)#exit
```

**Vers SW_ET1_02**
```cisco
SW_ET1_01(config)#interface range fa0/3-4
SW_ET1_01(config-if-range)#channel-group 2 mode active
SW_ET1_01(config-if-range)#no shutdown
SW_ET1_01(config-if-range)#exit
SW_ET1_01(config)#interface port-channel 2
SW_ET1_01(config-if)#switchport mode trunk
SW_ET1_01(config-if)#exit
```

---

### SW_ET1_02

#### Configuration du nom
```cisco
Switch>enable
Switch#conf t
Switch(config)#hostname SW_ET1_02
```

#### Mise en place de la sécurité
```cisco
SW_ET1_02(config)#enable secret password_switch
SW_ET1_02(config)#service password-encryption
SW_ET1_02(config)#line console 0
SW_ET1_02(config-line)#password password_switch
SW_ET1_02(config-line)#login
SW_ET1_02(config-line)#exit
```

#### Mise en place du VTP Client
```cisco
SW_ET1_02(config)#vtp mode client
SW_ET1_02(config)#vtp domain DIGIPLEX
SW_ET1_02(config)#vtp password password_vtp
```

#### Configuration de la connexion à distance en SSH
```cisco
SW_ET1_02(config)#ip domain-name digiplex.funky
SW_ET1_02(config)#crypto key generate rsa
SW_ET1_02(config)#username admin secret password_switch_ssh
SW_ET1_02(config)#line vty 0 15
SW_ET1_02(config-line)#transport input ssh
SW_ET1_02(config-line)#login local
SW_ET1_02(config-line)#exit
```

#### Configuration de l'IP de management
```cisco
SW_ET1_02(config)#interface vlan 80
SW_ET1_02(config-if)#ip address 192.168.80.12 255.255.255.0
SW_ET1_02(config-if)#no shutdown
SW_ET1_02(config)#ip default-gateway 192.168.80.254
```

#### Configuration de l'EtherChannel vers le switch L2 01
```cisco
SW_ET1_02(config)#interface range fa0/1-2
SW_ET1_02(config-if-range)#channel-group 1 mode active
SW_ET1_02(config-if-range)#no shutdown
SW_ET1_02(config-if-range)#exit
SW_ET1_02(config)#interface port-channel 1
SW_ET1_02(config-if)#switchport mode trunk
SW_ET1_02(config-if)#exit
```

#### Configuration des ports utilisés par les clients

**VLAN 30 - Ressources_Humaine**
```cisco
SW_ET1_02(config)#interface range fa0/3-13
SW_ET1_02(config-if-range)#switchport mode access
SW_ET1_02(config-if-range)#switchport access vlan 30
SW_ET1_02(config-if-range)#exit
```

**VLAN 10 - Conception**
```cisco
SW_ET1_02(config)#interface range fa0/14-24
SW_ET1_02(config-if-range)#switchport mode access
SW_ET1_02(config-if-range)#switchport access vlan 10
SW_ET1_02(config-if-range)#exit
```

---

### SW_ET1_03

#### Configuration du nom
```cisco
Switch>enable
Switch#conf t
Switch(config)#hostname SW_ET1_03
```

#### Mise en place de la sécurité
```cisco
SW_ET1_03(config)#enable secret password_switch
SW_ET1_03(config)#service password-encryption
SW_ET1_03(config)#line console 0
SW_ET1_03(config-line)#password password_switch
SW_ET1_03(config-line)#login
SW_ET1_03(config-line)#exit
```

#### Mise en place du VTP Client
```cisco
SW_ET1_03(config)#vtp mode client
SW_ET1_03(config)#vtp domain DIGIPLEX
SW_ET1_03(config)#vtp password password_vtp
```

#### Configuration de la connexion à distance en SSH
```cisco
SW_ET1_03(config)#ip domain-name digiplex.funky
SW_ET1_03(config)#crypto key generate rsa
SW_ET1_03(config)#username admin secret password_switch_ssh
SW_ET1_03(config)#line vty 0 15
SW_ET1_03(config-line)#transport input ssh
SW_ET1_03(config-line)#login local
SW_ET1_03(config-line)#exit
```

#### Configuration de l'IP de management
```cisco
SW_ET1_03(config)#interface vlan 80
SW_ET1_03(config-if)#ip address 192.168.80.13 255.255.255.0
SW_ET1_03(config-if)#no shutdown
SW_ET1_03(config)#ip default-gateway 192.168.80.254
```

#### Configuration de l'EtherChannel vers le switch L2 01
```cisco
SW_ET1_03(config)#interface range fa0/1-2
SW_ET1_03(config-if-range)#channel-group 1 mode active
SW_ET1_03(config-if-range)#no shutdown
SW_ET1_03(config-if-range)#exit
SW_ET1_03(config)#interface port-channel 1
SW_ET1_03(config-if)#switchport mode trunk
SW_ET1_03(config-if)#exit
```

#### Configuration des ports utilisés par les clients

**VLAN 20 - Commercial**
```cisco
SW_ET1_03(config)#interface range fa0/3-18
SW_ET1_03(config-if-range)#switchport mode access
SW_ET1_03(config-if-range)#switchport access vlan 20
SW_ET1_03(config-if-range)#exit
```

#### Configuration de la borne Wi-Fi
```cisco
SW_ET1_03(config)#interface fa0/19
SW_ET1_03(config-if)#switchport mode trunk
SW_ET1_03(config-if)#switchport trunk native vlan 70
SW_ET1_03(config-if)#switchport trunk allowed vlan 50,60,70
SW_ET1_03(config-if)#no shutdown
SW_ET1_03(config-if)#exit
```

---

## Configuration des Switchs de l'Étage 2

### SW_ET2_01

#### Configuration du nom
```cisco
Switch>enable
Switch#conf t
Switch(config)#hostname SW_ET2_01
```

#### Mise en place de la sécurité
```cisco
SW_ET2_01(config)#enable secret password_switch
SW_ET2_01(config)#service password-encryption
SW_ET2_01(config)#line console 0
SW_ET2_01(config-line)#password password_switch
SW_ET2_01(config-line)#login
SW_ET2_01(config-line)#exit
```

#### Mise en place du VTP Client
```cisco
SW_ET2_01(config)#vtp mode client
SW_ET2_01(config)#vtp domain DIGIPLEX
SW_ET2_01(config)#vtp password password_vtp
```

#### Configuration de la connexion à distance en SSH
```cisco
SW_ET2_01(config)#ip domain-name digiplex.funky
SW_ET2_01(config)#crypto key generate rsa
SW_ET2_01(config)#username admin secret password_switch_ssh
SW_ET2_01(config)#line vty 0 15
SW_ET2_01(config-line)#transport input ssh
SW_ET2_01(config-line)#login local
SW_ET2_01(config-line)#exit
```

#### Configuration de l'IP de management
```cisco
SW_ET2_01(config)#interface vlan 80
SW_ET2_01(config-if)#ip address 192.168.80.21 255.255.255.0
SW_ET2_01(config-if)#no shutdown
SW_ET2_01(config)#ip default-gateway 192.168.80.254
```

#### Configuration de l'EtherChannel vers le switch L3 du RDC
```cisco
SW_ET2_01(config)#interface range fa0/1-2
SW_ET2_01(config-if-range)#channel-group 1 mode active
SW_ET2_01(config-if-range)#no shutdown
SW_ET2_01(config-if-range)#exit
SW_ET2_01(config)#interface port-channel 1
SW_ET2_01(config-if)#switchport mode trunk
SW_ET2_01(config-if)#exit
```

#### Configuration de l'EtherChannel vers les switchs de l'étage 2
```cisco
SW_ET2_01(config)#interface range fa0/3-4
SW_ET2_01(config-if-range)#channel-group 3 mode active
SW_ET2_01(config-if-range)#no shutdown
SW_ET2_01(config-if-range)#exit
SW_ET2_01(config)#interface port-channel 3
SW_ET2_01(config-if)#switchport mode trunk
SW_ET2_01(config-if)#exit

SW_ET2_01(config)#interface range fa0/5-6
SW_ET2_01(config-if-range)#channel-group 2 mode active
SW_ET2_01(config-if-range)#no shutdown
SW_ET2_01(config-if-range)#exit
SW_ET2_01(config)#interface port-channel 2
SW_ET2_01(config-if)#switchport mode trunk
SW_ET2_01(config-if)#exit
```

---

### SW_ET2_02

#### Configuration du nom
```cisco
Switch>enable
Switch#conf t
Switch(config)#hostname SW_ET2_02
```

#### Mise en place de la sécurité
```cisco
SW_ET2_02(config)#enable secret password_switch
SW_ET2_02(config)#service password-encryption
SW_ET2_02(config)#line console 0
SW_ET2_02(config-line)#password password_switch
SW_ET2_02(config-line)#login
SW_ET2_02(config-line)#exit
```

#### Mise en place du VTP Client
```cisco
SW_ET2_02(config)#vtp mode client
SW_ET2_02(config)#vtp domain DIGIPLEX
SW_ET2_02(config)#vtp password password_vtp
```

#### Configuration de la connexion à distance en SSH
```cisco
SW_ET2_02(config)#ip domain-name digiplex.funky
SW_ET2_02(config)#crypto key generate rsa
SW_ET2_02(config)#username admin secret password_switch_ssh
SW_ET2_02(config)#line vty 0 15
SW_ET2_02(config-line)#transport input ssh
SW_ET2_02(config-line)#login local
SW_ET2_02(config-line)#exit
```

#### Configuration de l'IP de management
```cisco
SW_ET2_02(config)#interface vlan 80
SW_ET2_02(config-if)#ip address 192.168.80.22 255.255.255.0
SW_ET2_02(config-if)#no shutdown
SW_ET2_02(config)#ip default-gateway 192.168.80.254
```

#### Configuration de l'EtherChannel vers le switch L2 01
```cisco
SW_ET2_02(config)#interface range fa0/1-2
SW_ET2_02(config-if-range)#channel-group 1 mode active
SW_ET2_02(config-if-range)#no shutdown
SW_ET2_02(config-if-range)#exit
SW_ET2_02(config)#interface port-channel 1
SW_ET2_02(config-if)#switchport mode trunk
SW_ET2_02(config-if)#exit
```

#### Configuration des ports utilisés par les clients

**VLAN 20 - Commercial**
```cisco
SW_ET2_02(config)#interface range fa0/3-5
SW_ET2_02(config-if-range)#switchport mode access
SW_ET2_02(config-if-range)#switchport access vlan 20
SW_ET2_02(config-if-range)#exit
```

---

### SW_ET2_03

#### Configuration du nom
```cisco
Switch>enable
Switch#conf t
Switch(config)#hostname SW_ET2_03
```

#### Mise en place de la sécurité
```cisco
SW_ET2_03(config)#enable secret password_switch
SW_ET2_03(config)#service password-encryption
SW_ET2_03(config)#line console 0
SW_ET2_03(config-line)#password password_switch
SW_ET2_03(config-line)#login
SW_ET2_03(config-line)#exit
```

#### Mise en place du VTP Client
```cisco
SW_ET2_03(config)#vtp mode client
SW_ET2_03(config)#vtp domain DIGIPLEX
SW_ET2_03(config)#vtp password password_vtp
```

#### Configuration de la connexion à distance en SSH
```cisco
SW_ET2_03(config)#ip domain-name digiplex.funky
SW_ET2_03(config)#crypto key generate rsa
SW_ET2_03(config)#username admin secret password_switch_ssh
SW_ET2_03(config)#line vty 0 15
SW_ET2_03(config-line)#transport input ssh
SW_ET2_03(config-line)#login local
SW_ET2_03(config-line)#exit
```

#### Configuration de l'IP de management
```cisco
SW_ET2_03(config)#interface vlan 80
SW_ET2_03(config-if)#ip address 192.168.80.23 255.255.255.0
SW_ET2_03(config-if)#no shutdown
SW_ET2_03(config)#ip default-gateway 192.168.80.254
```

#### Configuration de l'EtherChannel vers le switch L2 01
```cisco
SW_ET2_03(config)#interface range fa0/1-2
SW_ET2_03(config-if-range)#channel-group 1 mode active
SW_ET2_03(config-if-range)#no shutdown
SW_ET2_03(config-if-range)#exit
SW_ET2_03(config)#interface port-channel 1
SW_ET2_03(config-if)#switchport mode trunk
SW_ET2_03(config-if)#exit
```

#### Configuration des ports utilisés par les clients

**VLAN 10 - Conception**
```cisco
SW_ET2_03(config)#interface range fa0/3-4
SW_ET2_03(config-if-range)#switchport mode access
SW_ET2_03(config-if-range)#switchport access vlan 10
SW_ET2_03(config-if-range)#exit
```

**VLAN 40 - Hotline**
```cisco
SW_ET2_03(config)#interface fa0/5
SW_ET2_03(config-if)#switchport mode access
SW_ET2_03(config-if)#switchport access vlan 40
SW_ET2_03(config-if)#exit
```

---

## Configuration des Switchs de l'Étage 3

### SW_ET3_01

#### Configuration du nom
```cisco
Switch>enable
Switch#conf t
Switch(config)#hostname SW_ET3_01
```

#### Mise en place de la sécurité
```cisco
SW_ET3_01(config)#enable secret password_switch
SW_ET3_01(config)#service password-encryption
SW_ET3_01(config)#line console 0
SW_ET3_01(config-line)#password password_switch
SW_ET3_01(config-line)#login
SW_ET3_01(config-line)#exit
```

#### Mise en place du VTP Client
```cisco
SW_ET3_01(config)#vtp mode client
SW_ET3_01(config)#vtp domain DIGIPLEX
SW_ET3_01(config)#vtp password password_vtp
```

#### Configuration de la connexion à distance en SSH
```cisco
SW_ET3_01(config)#ip domain-name digiplex.funky
SW_ET3_01(config)#crypto key generate rsa
SW_ET3_01(config)#username admin secret password_switch_ssh
SW_ET3_01(config)#line vty 0 15
SW_ET3_01(config-line)#transport input ssh
SW_ET3_01(config-line)#login local
SW_ET3_01(config-line)#exit
```

#### Configuration de l'IP de management
```cisco
SW_ET3_01(config)#interface vlan 80
SW_ET3_01(config-if)#ip address 192.168.80.31 255.255.255.0
SW_ET3_01(config-if)#no shutdown
SW_ET3_01(config)#ip default-gateway 192.168.80.254
```

#### Configuration de l'EtherChannel vers le switch L3 du RDC
```cisco
SW_ET3_01(config)#interface range fa0/1-2
SW_ET3_01(config-if-range)#channel-group 1 mode active
SW_ET3_01(config-if-range)#no shutdown
SW_ET3_01(config-if-range)#exit
SW_ET3_01(config)#interface port-channel 1
SW_ET3_01(config-if)#switchport mode trunk
SW_ET3_01(config-if)#exit
```

#### Configuration de l'EtherChannel vers les switchs de l'étage 3

**Vers SW_ET3_02**
```cisco
SW_ET3_01(config)#interface range fa0/21-22
SW_ET3_01(config-if-range)#channel-group 2 mode active
SW_ET3_01(config-if-range)#no shutdown
SW_ET3_01(config-if-range)#exit
SW_ET3_01(config)#interface port-channel 2
SW_ET3_01(config-if)#switchport mode trunk
SW_ET3_01(config-if)#exit
```

**Vers SW_ET3_03**
```cisco
SW_ET3_01(config)#interface range fa0/23-24
SW_ET3_01(config-if-range)#channel-group 3 mode active
SW_ET3_01(config-if-range)#no shutdown
SW_ET3_01(config-if-range)#exit
SW_ET3_01(config)#interface port-channel 3
SW_ET3_01(config-if)#switchport mode trunk
SW_ET3_01(config-if)#exit
```

**Vers SW_ET3_04**
```cisco
SW_ET3_01(config)#interface range fa0/19-20
SW_ET3_01(config-if-range)#channel-group 4 mode active
SW_ET3_01(config-if-range)#no shutdown
SW_ET3_01(config-if-range)#exit
SW_ET3_01(config)#interface port-channel 4
SW_ET3_01(config-if)#switchport mode trunk
SW_ET3_01(config-if)#exit
```

---

### SW_ET3_02

#### Configuration du nom
```cisco
Switch>enable
Switch#conf t
Switch(config)#hostname SW_ET3_02
```

#### Mise en place de la sécurité
```cisco
SW_ET3_02(config)#enable secret password_switch
SW_ET3_02(config)#service password-encryption
SW_ET3_02(config)#line console 0
SW_ET3_02(config-line)#password password_switch
SW_ET3_02(config-line)#login
SW_ET3_02(config-line)#exit
```

#### Mise en place du VTP Client
```cisco
SW_ET3_02(config)#vtp mode client
SW_ET3_02(config)#vtp domain DIGIPLEX
SW_ET3_02(config)#vtp password password_vtp
```

#### Configuration de la connexion à distance en SSH
```cisco
SW_ET3_02(config)#ip domain-name digiplex.funky
SW_ET3_02(config)#crypto key generate rsa
SW_ET3_02(config)#username admin secret password_switch_ssh
SW_ET3_02(config)#line vty 0 15
SW_ET3_02(config-line)#transport input ssh
SW_ET3_02(config-line)#login local
SW_ET3_02(config-line)#exit
```

#### Configuration de l'IP de management
```cisco
SW_ET3_02(config)#interface vlan 80
SW_ET3_02(config-if)#ip address 192.168.80.32 255.255.255.0
SW_ET3_02(config-if)#no shutdown
SW_ET3_02(config)#ip default-gateway 192.168.80.254
```

#### Configuration de l'EtherChannel vers le switch L2 01
```cisco
SW_ET3_02(config)#interface range fa0/23-24
SW_ET3_02(config-if-range)#channel-group 1 mode active
SW_ET3_02(config-if-range)#no shutdown
SW_ET3_02(config-if-range)#exit
SW_ET3_02(config)#interface port-channel 1
SW_ET3_02(config-if)#switchport mode trunk
SW_ET3_02(config-if)#exit
```

#### Configuration de la borne Wi-Fi
```cisco
SW_ET3_02(config)#interface fa0/1 
SW_ET3_02(config-if)#switchport mode trunk
SW_ET3_02(config-if)#switchport trunk native vlan 70
SW_ET3_02(config-if)#switchport trunk allowed vlan 50,60,70
SW_ET3_02(config-if)#no shutdown
SW_ET3_02(config-if)#exit

SW_ET3_02(config)#interface range fa0/20-21
SW_ET3_02(config-if-range)#switchport mode trunk
SW_ET3_02(config-if-range)#switchport trunk native vlan 70
SW_ET3_02(config-if-range)#switchport trunk allowed vlan 50,60,70
SW_ET3_02(config-if-range)#no shutdown
SW_ET3_02(config-if-range)#exit
```

#### Configuration des ports utilisés par les clients

**VLAN 40 - Hotline**
```cisco
SW_ET3_02(config)#interface range fa0/2-12
SW_ET3_02(config-if-range)#switchport mode access
SW_ET3_02(config-if-range)#switchport access vlan 40
SW_ET3_02(config-if-range)#exit
```

**VLAN 20 - Commercial**
```cisco
SW_ET3_02(config)#interface range fa0/13-19
SW_ET3_02(config-if-range)#switchport mode access
SW_ET3_02(config-if-range)#switchport access vlan 20
SW_ET3_02(config-if-range)#exit
```

---

### SW_ET3_03

#### Configuration du nom
```cisco
Switch>enable
Switch#conf t
Switch(config)#hostname SW_ET3_03
```

#### Mise en place de la sécurité
```cisco
SW_ET3_03(config)#enable secret password_switch
SW_ET3_03(config)#service password-encryption
SW_ET3_03(config)#line console 0
SW_ET3_03(config-line)#password password_switch
SW_ET3_03(config-line)#login
SW_ET3_03(config-line)#exit
```

#### Mise en place du VTP Client
```cisco
SW_ET3_03(config)#vtp mode client
SW_ET3_03(config)#vtp domain DIGIPLEX
SW_ET3_03(config)#vtp password password_vtp
```

#### Configuration de la connexion à distance en SSH
```cisco
SW_ET3_03(config)#ip domain-name digiplex.funky
SW_ET3_03(config)#crypto key generate rsa
SW_ET3_03(config)#username admin secret password_switch_ssh
SW_ET3_03(config)#line vty 0 15
SW_ET3_03(config-line)#transport input ssh
SW_ET3_03(config-line)#login local
SW_ET3_03(config-line)#exit
```

#### Configuration de l'IP de management
```cisco
SW_ET3_03(config)#interface vlan 80
SW_ET3_03(config-if)#ip address 192.168.80.33 255.255.255.0
SW_ET3_03(config-if)#no shutdown
SW_ET3_03(config)#ip default-gateway 192.168.80.254
```

#### Configuration de l'EtherChannel vers le switch L2 01
```cisco
SW_ET3_03(config)#interface range fa0/23-24
SW_ET3_03(config-if-range)#channel-group 1 mode active
SW_ET3_03(config-if-range)#no shutdown
SW_ET3_03(config-if-range)#exit
SW_ET3_03(config)#interface port-channel 1
SW_ET3_03(config-if)#switchport mode trunk
SW_ET3_03(config-if)#exit
```

#### Configuration des ports utilisés par les clients

**VLAN 30 - RH**
```cisco
SW_ET3_03(config)#interface range fa0/1-9
SW_ET3_03(config-if-range)#switchport mode access
SW_ET3_03(config-if-range)#switchport access vlan 30
SW_ET3_03(config-if-range)#exit
```

**VLAN 20 - Commercial**
```cisco
SW_ET3_03(config)#interface range fa0/10-17
SW_ET3_03(config-if-range)#switchport mode access
SW_ET3_03(config-if-range)#switchport access vlan 20
SW_ET3_03(config-if-range)#exit
```

---

### SW_ET3_04

#### Configuration du nom
```cisco
Switch>enable
Switch#conf t
Switch(config)#hostname SW_ET3_04
```

#### Mise en place de la sécurité
```cisco
SW_ET3_04(config)#enable secret password_switch
SW_ET3_04(config)#service password-encryption
SW_ET3_04(config)#line console 0
SW_ET3_04(config-line)#password password_switch
SW_ET3_04(config-line)#login
SW_ET3_04(config-line)#exit
```

#### Mise en place du VTP Client
```cisco
SW_ET3_04(config)#vtp mode client
SW_ET3_04(config)#vtp domain DIGIPLEX
SW_ET3_04(config)#vtp password password_vtp
```

#### Configuration de la connexion à distance en SSH
```cisco
SW_ET3_04(config)#ip domain-name digiplex.funky
SW_ET3_04(config)#crypto key generate rsa
SW_ET3_04(config)#username admin secret password_switch_ssh
SW_ET3_04(config)#line vty 0 15
SW_ET3_04(config-line)#transport input ssh
SW_ET3_04(config-line)#login local
SW_ET3_04(config-line)#exit
```

#### Configuration de l'IP de management
```cisco
SW_ET3_04(config)#interface vlan 80
SW_ET3_04(config-if)#ip address 192.168.80.34 255.255.255.0
SW_ET3_04(config-if)#no shutdown
SW_ET3_04(config)#ip default-gateway 192.168.80.254
```

#### Configuration de l'EtherChannel vers le switch L2 01
```cisco
SW_ET3_04(config)#interface range fa0/23-24
SW_ET3_04(config-if-range)#channel-group 3 mode active
SW_ET3_04(config-if-range)#no shutdown
SW_ET3_04(config-if-range)#exit
SW_ET3_04(config)#interface port-channel 3
SW_ET3_04(config-if)#switchport mode trunk
SW_ET3_04(config-if)#exit
```

#### Configuration des ports utilisés par les clients

**VLAN 10 - Conception**
```cisco
SW_ET3_04(config)#interface range fa0/2-13
SW_ET3_04(config-if-range)#switchport mode access
SW_ET3_04(config-if-range)#switchport access vlan 10
SW_ET3_04(config-if-range)#exit
```

**VLAN 30 - RH**
```cisco
SW_ET3_04(config)#interface range fa0/14-17
SW_ET3_04(config-if-range)#switchport mode access
SW_ET3_04(config-if-range)#switchport access vlan 30
SW_ET3_04(config-if-range)#exit
```

#### Configuration de la borne Wi-Fi
```cisco
SW_ET3_04(config)#interface fa0/18 
SW_ET3_04(config-if)#switchport mode trunk
SW_ET3_04(config-if)#switchport trunk native vlan 70
SW_ET3_04(config-if)#switchport trunk allowed vlan 50,60,70
SW_ET3_04(config-if)#no shutdown
SW_ET3_04(config-if)#exit
```

---

## Configuration du Serveur DHCP

C'est lui qui va donner les adresses à tout le monde.

### Configuration IP du Serveur (Desktop > IP Configuration)

- **Adresse IP :** 192.168.70.1
- **Masque :** 255.255.255.0
- **Passerelle :** 192.168.70.254
- **DNS :** 192.168.70.3

### Configuration des Pools DHCP

Ensuite, on va créer un « Pool » pour chaque VLAN :

#### VLAN_10_CONCEPTION

- **Passerelle :** 192.168.10.254
- **DNS :** 192.168.70.3
- **Start IP :** 192.168.10.1
- **Masque :** 255.255.255.0
- **Maximum utilisateurs :** 254

#### VLAN_20_COMMERCIAL

- **Passerelle :** 192.168.20.254
- **DNS :** 192.168.70.3
- **Start IP :** 192.168.20.1
- **Masque :** 255.255.255.0
- **Maximum utilisateurs :** 254

#### VLAN_30_RH

- **Passerelle :** 192.168.30.254
- **DNS :** 192.168.70.3
- **Start IP :** 192.168.30.1
- **Masque :** 255.255.255.0
- **Maximum utilisateurs :** 254

#### VLAN_40_HOTLINE

- **Passerelle :** 192.168.40.254
- **DNS :** 192.168.70.3
- **Start IP :** 192.168.40.1
- **Masque :** 255.255.255.0
- **Maximum utilisateurs :** 254

#### VLAN_50_WiFiENTREPRISE

- **Passerelle :** 192.168.50.254
- **DNS :** 192.168.70.3
- **Start IP :** 192.168.50.1
- **Masque :** 255.255.255.0
- **Maximum utilisateurs :** 254

#### VLAN_60_WiFiINVITES

- **Passerelle :** 192.168.60.254
- **DNS :** 192.168.70.3
- **Start IP :** 192.168.60.1
- **Masque :** 255.255.255.0
- **Maximum utilisateurs :** 254

---

## Configuration du Contrôleur WiFi

### 1. Configuration IP du WLC

WLC > Config > Management

- **Adresse IP :** 192.168.70.10
- **Masque :** 255.255.255.0
- **Passerelle :** 192.168.70.254

### 2. Créer les WLANs (WLANs > Create New)

#### WLAN 1

- **Profile Name :** WiFiEntreprise 
- **SSID :** WiFi_ENTREPRISE
- **VLAN :** 50
- **Sécurité :** WPA2 + PSK (Mot de passe)
- **Mot de passe :** password_wifi

#### WLAN 2

- **Profile Name :** WiFiInvites
- **SSID :** WiFi_INVITES
- **VLAN :** 60
- **Sécurité :** Disabled

---

## Architecture Réseau

### VLANs

| VLAN | Nom | Réseau | Passerelle | Utilisateurs |
|------|-----|--------|------------|--------------|
| 10 | CONCEPTION | 192.168.10.0/24 | 192.168.10.254 | 254 |
| 20 | COMMERCIAL | 192.168.20.0/24 | 192.168.20.254 | 254 |
| 30 | RESSOURCES_HUMAINES | 192.168.30.0/24 | 192.168.30.254 | 254 |
| 40 | HOTLINE | 192.168.40.0/24 | 192.168.40.254 | 254 |
| 50 | WIFI_ENTREPRISE | 192.168.50.0/24 | 192.168.50.254 | 254 |
| 60 | WIFI_INVITES | 192.168.60.0/24 | 192.168.60.254 | 254 |
| 70 | SERVER | 192.168.70.0/24 | 192.168.70.254 | 254 |
| 80 | MANAGEMENT | 192.168.80.0/24 | 192.168.80.254 | 254 |

### Équipements

- **Routeur :** Routeur_DIGIPLEX
- **Switch L3 :** SwitchL3_DIGIPLEX (Mode VTP Server)
- **Switchs L2 :** 
  - RDC : SW_RDC_02, SW_RDC_03
  - Étage 1 : SW_ET1_01, SW_ET1_02, SW_ET1_03
  - Étage 2 : SW_ET2_01, SW_ET2_02, SW_ET2_03
  - Étage 3 : SW_ET3_01, SW_ET3_02, SW_ET3_03, SW_ET3_04
- **Serveur DHCP :** 192.168.70.1
- **Contrôleur WiFi :** 192.168.70.10

### Technologies

- **VTP :** Domaine DIGIPLEX (Server sur SWL3, Client sur tous les autres)
- **EtherChannel :** LACP (mode active) pour tous les liens trunk
- **Routage Inter-VLAN :** Layer 3 Switch (SwitchL3_DIGIPLEX)
- **DHCP :** Serveur centralisé avec relay (ip helper-address)
- **SSH :** Activé sur tous les équipements d'administration
- **NAT :** Configuré pour l'accès Internet
- **WiFi :** Contrôleur WLC avec 2 SSIDs (Entreprise sécurisé / Invités ouvert)
