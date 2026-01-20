# Déploiement ENGIE

## Table des matières
- [Plan d'adressage (Calcul VLSM)](#plan-dadressage-calcul-vlsm)
- [Configuration du Routeur](#configuration-du-routeur)
- [Configuration du Switch 1](#configuration-du-switch-1)
- [Configuration du Switch 2](#configuration-du-switch-2)
- [Configuration du Serveur (DHCP & DNS)](#configuration-du-serveur-dhcp--dns)
- [Configuration de la Borne Wi-Fi](#configuration-de-la-borne-wi-fi)
- [Configuration des Postes Clients](#configuration-des-postes-clients)

---

## Plan d'adressage (Calcul VLSM)

Avant de toucher au matériel, nous devons définir les plages IP. Nous partons du réseau de base 192.168.0.0 et nous le découpons selon les besoins pour optimiser l'espace.

| Engie | Masque | Intervalle hôtes | Sous-réseau | Broadcast |
|-------|--------|------------------|-------------|-----------|
| VLAN 10 | /25 | 192.168.0.1 - 192.168.0.126 | 192.168.0.0 | 192.168.0.127 |
| VLAN 11 | /26 | 192.168.0.129 - 192.168.0.190 | 192.168.0.128 | 192.168.0.191 |
| VLAN 12 | /27 | 192.168.0.193 - 192.168.0.222 | 192.168.0.192 | 192.168.0.223 |
| Routeur & Server | /30 | 192.168.0.225 - 192.168.0.226 | 192.168.0.224 | 192.168.0.227 |

---

## Configuration du Routeur

Le routeur va gérer le Routage Inter-VLAN (Router-on-a-stick). Il possède une sous-interface pour chaque VLAN. Comme le serveur DHCP est dans un autre réseau, nous utilisons la commande `ip helper-address` sur les autres interfaces pour relayer les demandes DHCP vers lui.

### Configuration du nom du routeur
```cisco
Router>enable
Router#conf t
Router(config)#hostname Routeur_ENGIE
```

### Configuration des mots de passe d'accès
```cisco
Routeur_ENGIE(config)#enable secret password_routeur
Routeur_ENGIE(config)#service password-encryption
Routeur_ENGIE(config)#line console 0
Routeur_ENGIE(config-line)#password password_routeur
Routeur_ENGIE(config-line)#login
Routeur_ENGIE(config-line)#exit
```

### Configuration de l'accès à distance SSH
```cisco
Routeur_ENGIE(config)#ip domain-name engie.funky
Routeur_ENGIE(config)#username admin secret password_routeur_ssh
Routeur_ENGIE(config)#crypto key generate rsa
```

### Forcer la vérification de l'utilisateur pour le SSH
```cisco
Routeur_ENGIE(config)#line vty 0 4
Routeur_ENGIE(config-line)#login local
Routeur_ENGIE(config-line)#transport input ssh
Routeur_ENGIE(config-line)#exit
```

### Création des VLANs
```cisco
Routeur_ENGIE#vlan database
Routeur_ENGIE(vlan)#vlan 10 name TECHNIQUE
Routeur_ENGIE(vlan)#vlan 11 name COMMERCIAL
Routeur_ENGIE(vlan)#vlan 12 name INVITES
```

### Configuration VLAN 10
```cisco
Routeur_ENGIE(config)#interface vlan 10
Routeur_ENGIE(config-if)#ip address 192.168.0.126 255.255.255.128
Routeur_ENGIE(config-if)#ip helper-address 192.168.0.225
Routeur_ENGIE(config-if)#no shutdown
Routeur_ENGIE(config-if)#ip nat inside
Routeur_ENGIE(config-if)#exit
```

### Configuration VLAN 11
```cisco
Routeur_ENGIE(config)#interface vlan 11
Routeur_ENGIE(config-if)#ip address 192.168.0.190 255.255.255.192
Routeur_ENGIE(config-if)#ip helper-address 192.168.0.225
Routeur_ENGIE(config-if)#no shutdown
Routeur_ENGIE(config-if)#ip nat inside
Routeur_ENGIE(config-if)#exit
```

### Configuration VLAN 12
```cisco
Routeur_ENGIE(config)#interface vlan 12
Routeur_ENGIE(config-if)#ip address 192.168.0.222 255.255.255.224
Routeur_ENGIE(config-if)#ip helper-address 192.168.0.225
Routeur_ENGIE(config-if)#no shutdown
Routeur_ENGIE(config-if)#ip nat inside
Routeur_ENGIE(config-if)#exit
```

### Configuration des ports physiques vers les switchs
```cisco
Routeur_ENGIE(config)#interface range FastEthernet 1/0 - 1
Routeur_ENGIE(config-if)#switchport mode trunk
Routeur_ENGIE(config-if)#no shutdown
Routeur_ENGIE(config-if)#exit
```

### Configuration interface Serveur
```cisco
Routeur_ENGIE(config)#interface Fa0/0
Routeur_ENGIE(config-if)#ip address 192.168.0.226 255.255.255.252
Routeur_ENGIE(config-if)#no shutdown
Routeur_ENGIE(config-if)#ip nat inside
Routeur_ENGIE(config-if)#exit
```

### Configuration de l'interface WAN et NAT
```cisco
Routeur_ENGIE(config)#interface G0/0/0
Routeur_ENGIE(config-if)#ip address dhcp
Routeur_ENGIE(config-if)#no shutdown
Routeur_ENGIE(config-if)#ip nat outside
Routeur_ENGIE(config-if)#exit
Routeur_ENGIE(config)#access-list 1 permit 192.168.0.0 0.0.0.255
Routeur_ENGIE(config)#ip nat inside source list 1 interface G0/0/0 overload 
Routeur_ENGIE(config)#ip route 0.0.0.0 0.0.0.0 G0/0/0
```

---

## Configuration du Switch 1

Les switchs sont configurés en mode VTP transparent. Les VLANs sont créés manuellement sur chaque switch.

### Configuration du nom du switch
```cisco
Switch>enable
Switch#conf t
Switch(config)#hostname Switch1_ENGIE
```

### Mise en place de la sécurité
```cisco
Switch1_ENGIE(config)#enable secret password_switch
Switch1_ENGIE(config)#service password-encryption
Switch1_ENGIE(config)#line console 0
Switch1_ENGIE(config-line)#password password_switch
Switch1_ENGIE(config-line)#login
Switch1_ENGIE(config-line)#exit
```

### Configuration VTP mode transparent
```cisco
Switch1_ENGIE(config)#vtp mode transparent
```

### Création des VLANs
```cisco
Switch1_ENGIE(config)#vlan 10
Switch1_ENGIE(config-vlan)#name TECHNIQUE
Switch1_ENGIE(config)#vlan 11
Switch1_ENGIE(config-vlan)#name COMMERCIAL
Switch1_ENGIE(config)#vlan 12
Switch1_ENGIE(config-vlan)#name INVITES
```

### Configuration du lien TRUNK (Vers Routeur)
```cisco
Switch1_ENGIE(config)#interface fa0/24
Switch1_ENGIE(config-if-range)#switchport mode trunk
Switch1_ENGIE(config-if-range)#switchport trunk allowed vlan 10,11,12
```

### Affectation des ports

#### VLAN 10 – TECHNIQUE
```cisco
Switch1_ENGIE(config)#interface range fa0/5-17
Switch1_ENGIE(config-if-range)#switchport mode access
Switch1_ENGIE(config-if-range)#switchport access vlan 10
```

#### VLAN 11 – COMMERCIAL
```cisco
Switch1_ENGIE(config)#interface range fa0/1-4
Switch1_ENGIE(config-if-range)#switchport mode access
Switch1_ENGIE(config-if-range)#switchport access vlan 11
```

#### VLAN 12 – INVITÉS
```cisco
Switch1_ENGIE(config)#interface fa0/18
Switch1_ENGIE(config-if-range)#switchport mode access
Switch1_ENGIE(config-if-range)#switchport access vlan 12
```

---

## Configuration du Switch 2

Les switchs sont configurés en mode VTP transparent. Les VLANs sont créés manuellement sur chaque switch.

### Configuration du nom du switch
```cisco
Switch>enable
Switch#conf t
Switch(config)#hostname Switch2_ENGIE
```

### Mise en place de la sécurité
```cisco
Switch2_ENGIE(config)#enable secret password_switch
Switch2_ENGIE(config)#service password-encryption
Switch2_ENGIE(config)#line console 0
Switch2_ENGIE(config-line)#password password_switch
Switch2_ENGIE(config-line)#login
Switch2_ENGIE(config-line)#exit
```

### Configuration VTP mode transparent
```cisco
Switch2_ENGIE(config)#vtp mode transparent
```

### Création des VLANs
```cisco
Switch2_ENGIE(config)#vlan 10
Switch2_ENGIE(config-vlan)#name TECHNIQUE
Switch2_ENGIE(config)#vlan 11
Switch2_ENGIE(config-vlan)#name COMMERCIAL
Switch2_ENGIE(config)#vlan 12
Switch2_ENGIE(config-vlan)#name INVITES
```

### Configuration du lien TRUNK (Vers Routeur)
```cisco
Switch2_ENGIE(config)#interface fa0/24
Switch2_ENGIE(config-if-range)#switchport mode trunk
Switch2_ENGIE(config-if-range)#switchport trunk allowed vlan 10,11,12
```

### Affectation des ports

#### VLAN 10 – TECHNIQUE
```cisco
Switch2_ENGIE(config)#interface fa0/1
Switch2_ENGIE(config-if-range)#switchport access vlan 10
Switch2_ENGIE(config-if-range)#switchport mode access
```

#### VLAN 11 – COMMERCIAL
```cisco
Switch2_ENGIE(config)#interface range fa0/2-11
Switch2_ENGIE(config-if-range)#switchport access vlan 11
Switch2_ENGIE(config-if-range)#switchport mode access
```

#### VLAN 12 – INVITÉS
```cisco
Switch2_ENGIE(config)#interface fa0/12
Switch2_ENGIE(config-if-range)#switchport access vlan 12
Switch2_ENGIE(config-if-range)#switchport mode access
```

---

## Configuration du Serveur (DHCP & DNS)

Le serveur est branché sur un port configuré dans le sous-réseau 192.168.0.224/30.

### Configuration IP Statique (Desktop > IP Config)

- **Adresse IP :** 192.168.0.225
- **Masque :** 255.255.255.252 (/30)
- **Passerelle :** 192.168.0.226
- **DNS Serveur :** 127.0.0.1

### Configuration Service DHCP

Nous créons 3 pools pour alimenter les VLANs. Le routeur relaiera les demandes ici.

#### Pool VLAN_10

- **Passerelle par défaut :** 192.168.0.126
- **DNS Serveur :** 192.168.0.225
- **Début IP :** 192.168.0.1
- **Masque :** 255.255.255.128
- **Utilisateurs max :** 100

#### Pool VLAN_11

- **Passerelle par défaut :** 192.168.0.190
- **DNS Serveur :** 192.168.0.225
- **Start IP :** 192.168.0.129
- **Masque :** 255.255.255.192
- **Utilisateurs max :** 60

#### Pool VLAN_12

- **Passerelle par défaut :** 192.168.0.222
- **DNS Serveur :** 8.8.8.8 (DNS Public pour les invités)
- **Start IP :** 192.168.0.193
- **Masque :** 255.255.255.224
- **Utilisateurs max :** 20

---

## Configuration de la Borne Wi-Fi

La borne Wi-Fi est configurée pour offrir un accès ouvert au public, sans mot de passe, facilitant la connexion des usagers.

### Paramètres Wi-Fi

- **SSID :** WiFi_ENGIE_Guest
- **Authentification :** Disabled
- **Encryption :** None
- **Mot de passe :** Aucun

---

## Configuration des Postes Clients

### PC Fixes (Jaune/Orange)

- **Configuration :** DHCP
- **IP attendues :** 
  - VLAN 10 : 192.168.0.1 - 192.168.0.126
  - VLAN 11 : 192.168.0.129 - 192.168.0.190

### PC Portables & Téléphones (Bleu)

- **Connexion :** WiFi WiFi_ENGIE_Guest
- **Configuration IP :** DHCP
- **IP attendues :** 192.168.0.193 - 192.168.0.222

---

## Architecture Réseau

### VLANs

| VLAN | Nom | Réseau | Masque | Passerelle | Utilisateurs |
|------|-----|--------|--------|------------|--------------|
| 10 | TECHNIQUE | 192.168.0.0/25 | 255.255.255.128 | 192.168.0.126 | 100 |
| 11 | COMMERCIAL | 192.168.0.128/26 | 255.255.255.192 | 192.168.0.190 | 60 |
| 12 | INVITES | 192.168.0.192/27 | 255.255.255.224 | 192.168.0.222 | 20 |

### Équipements

- **Routeur :** Routeur_ENGIE (Router-on-a-stick)
- **Switch 1 :** Switch1_ENGIE (Mode VTP Transparent)
- **Switch 2 :** Switch2_ENGIE (Mode VTP Transparent)
- **Serveur :** 192.168.0.225 (DHCP & DNS)
- **Borne Wi-Fi :** WiFi_ENGIE_Guest (Open Access)

### Services

- **DHCP :** 3 pools (VLAN 10, 11, 12)
- **DNS :** Serveur local pour VLAN 10/11, Google DNS (8.8.8.8) pour VLAN 12
- **Routage Inter-VLAN :** Via Router-on-a-stick
- **NAT :** Configuré pour l'accès Internet
- **SSH :** Activé sur le routeur
