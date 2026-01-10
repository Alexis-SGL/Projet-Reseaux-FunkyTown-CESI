# Connexion Internet et Tunnel IPv6

## Table des matières
- [Partie 1 : Connexion vers Internet](#partie-1--connexion-vers-internet)
  - [Configuration du Routeur DSLAM](#configuration-du-routeur-dslam)
  - [Configuration des Routeurs de chaque Bâtiment](#configuration-des-routeurs-de-chaque-bâtiment)
  - [Configuration des Tables de Routage dans le Datacenter](#configuration-des-tables-de-routage-dans-le-datacenter)
- [Partie 2 : Tunnel IPv6 ESN eXia vers Cloud](#partie-2--tunnel-ipv6-esn-exia-vers-cloud)
  - [Prérequis](#prérequis)
  - [Configuration IPv4 pour atteindre le Cloud](#configuration-ipv4-pour-atteindre-le-cloud)
  - [Configuration du Tunnel IPv6](#configuration-du-tunnel-ipv6)
  - [Configuration des Équipements](#configuration-des-équipements)
  - [Tests de Connectivité](#tests-de-connectivité)

---

# Partie 1 : Connexion vers Internet

## Configuration du Routeur DSLAM

Le routeur DSLAM centralise les connexions de tous les bâtiments et leur fournit un accès vers Internet via le datacenter.

### Configuration du nom
```cisco
Router>enable
Router#conf t
Router(config)#hostname Routeur_DSLAM
```

### VERS DIGIPLEX (Port G0/0/0) - Réseau : 104.0.0.0 /30
```cisco
Routeur_DSLAM(config)#interface G0/0/0
Routeur_DSLAM(config-if)#description Vers_DIGIPLEX
Routeur_DSLAM(config-if)#ip address 104.0.0.1 255.255.255.252
Routeur_DSLAM(config-if)#no shutdown
Routeur_DSLAM(config-if)#exit
```

### VERS EXIA (Port G0/1/0) - Réseau : 101.0.0.0 /30
```cisco
Routeur_DSLAM(config)#interface G0/1/0
Routeur_DSLAM(config-if)#description Vers_EXIA
Routeur_DSLAM(config-if)#ip address 101.0.0.1 255.255.255.252
Routeur_DSLAM(config-if)#no shutdown
Routeur_DSLAM(config-if)#exit
```

### VERS BIBLIOTHEQUE (Port G0/2/0) - Réseau : 102.0.0.0 /30
```cisco
Routeur_DSLAM(config)#interface G0/2/0
Routeur_DSLAM(config-if)#description Vers_BIBLIOTHEQUE
Routeur_DSLAM(config-if)#ip address 102.0.0.1 255.255.255.252
Routeur_DSLAM(config-if)#no shutdown
Routeur_DSLAM(config-if)#exit
```

### VERS ENGIE (Port G0/3/0) - Réseau : 103.0.0.0 /30
```cisco
Routeur_DSLAM(config)#interface G0/3/0
Routeur_DSLAM(config-if)#description Vers_ENGIE
Routeur_DSLAM(config-if)#ip address 103.0.0.1 255.255.255.252
Routeur_DSLAM(config-if)#no shutdown
Routeur_DSLAM(config-if)#exit
```

---

## Configuration des Routeurs de chaque Bâtiment

Chaque bâtiment doit configurer son interface WAN pour se connecter au routeur DSLAM et établir une route par défaut.

### Routeur ESN eXia
```cisco
Routeur_eXia(config)#interface G0/0/0
Routeur_eXia(config-if)#no ip address dhcp
Routeur_eXia(config-if)#ip address 101.0.0.2 255.255.255.252
Routeur_eXia(config-if)#ip nat outside
Routeur_eXia(config-if)#no shutdown
Routeur_eXia(config-if)#exit

Routeur_eXia(config)#ip route 0.0.0.0 0.0.0.0 101.0.0.1
```

---

### Routeur Bibliothèque
```cisco
Routeur_Bibliotheque(config)#interface G0/0/0
Routeur_Bibliotheque(config-if)#no ip address dhcp
Routeur_Bibliotheque(config-if)#ip address 102.0.0.2 255.255.255.252
Routeur_Bibliotheque(config-if)#ip nat outside
Routeur_Bibliotheque(config-if)#no shutdown
Routeur_Bibliotheque(config-if)#exit

Routeur_Bibliotheque(config)#ip route 0.0.0.0 0.0.0.0 102.0.0.1
```

---

### Routeur Engie
```cisco
Routeur_ENGIE(config)#interface G0/0/0
Routeur_ENGIE(config-if)#no ip address dhcp
Routeur_ENGIE(config-if)#ip address 103.0.0.2 255.255.255.252
Routeur_ENGIE(config-if)#ip nat outside
Routeur_ENGIE(config-if)#no shutdown
Routeur_ENGIE(config-if)#exit

Routeur_ENGIE(config)#ip route 0.0.0.0 0.0.0.0 103.0.0.1
```

---

### Routeur Digiplex
```cisco
Routeur_DIGIPLEX(config)#interface G0/0/0
Routeur_DIGIPLEX(config-if)#no ip address dhcp
Routeur_DIGIPLEX(config-if)#ip address 104.0.0.2 255.255.255.252
Routeur_DIGIPLEX(config-if)#ip nat outside
Routeur_DIGIPLEX(config-if)#no shutdown
Routeur_DIGIPLEX(config-if)#exit

Routeur_DIGIPLEX(config)#ip route 0.0.0.0 0.0.0.0 104.0.0.1
```

---

## Configuration des Tables de Routage dans le Datacenter

Le datacenter est composé de plusieurs routeurs FAI qui acheminent le trafic entre le DSLAM et Internet (WAN).

### DSLAM

**Route par défaut vers FAI 1**
```cisco
Routeur_DSLAM(config)#ip route 0.0.0.0 0.0.0.0 80.0.0.6
```

---

### FAI 1

**Vers FAI 2 (route par défaut)**
```cisco
Routeur_FAI1(config)#ip route 0.0.0.0 0.0.0.0 80.0.0.13
```

**Vers DSLAM (retour des réseaux clients)**
```cisco
Routeur_FAI1(config)#ip route 100.0.0.0 255.0.0.0 80.0.0.5
```

---

### FAI 2

**Vers FAI 1 (réseaux clients)**
```cisco
Routeur_FAI2(config)#ip route 100.0.0.0 255.0.0.0 80.0.0.14
```

**Vers WAN (route par défaut)**
```cisco
Routeur_FAI2(config)#ip route 0.0.0.0 0.0.0.0 80.0.0.10
```

---

### WAN

**Vers FAI 2 (retour)**
```cisco
Routeur_WAN(config)#ip route 0.0.0.0 255.255.255.252 80.0.0.9
```

---

### Configuration alternative : DSLAM direct vers le WAN

Si on souhaite une connexion directe sans passer par les FAI intermédiaires :

**Sur le DSLAM**
```cisco
Routeur_DSLAM(config)#ip route 0.0.0.0 0.0.0.0 80.0.0.2
```

**Sur le WAN**
```cisco
Routeur_WAN(config)#ip route 0.0.0.0 0.0.0.0 80.0.0.1
```

---

## Architecture Réseau - Partie 1

### Plan d'adressage WAN

| Liaison | Réseau | Interface DSLAM | Interface Client |
|---------|--------|-----------------|------------------|
| DSLAM ↔ DIGIPLEX | 104.0.0.0/30 | 104.0.0.1 | 104.0.0.2 |
| DSLAM ↔ EXIA | 101.0.0.0/30 | 101.0.0.1 | 101.0.0.2 |
| DSLAM ↔ BIBLIOTHEQUE | 102.0.0.0/30 | 102.0.0.1 | 102.0.0.2 |
| DSLAM ↔ ENGIE | 103.0.0.0/30 | 103.0.0.1 | 103.0.0.2 |

### Topologie Datacenter
```
Bâtiments → DSLAM → FAI1 → FAI2 → WAN → Internet
```

### Équipements

- **Routeur DSLAM :** Point de concentration des connexions
- **Routeurs clients :** Routeur_eXia, Routeur_Bibliotheque, Routeur_ENGIE, Routeur_DIGIPLEX
- **Routeurs FAI :** FAI1, FAI2
- **Routeur WAN :** Sortie vers Internet

### Technologies

- **NAT :** Configuré sur tous les routeurs clients
- **Routage statique :** Tables de routage manuelles dans le datacenter
- **Agrégation de routes :** Réseau 100.0.0.0/8 pour tous les clients

---
---

# Partie 2 : Tunnel IPv6 ESN eXia vers Cloud

## Prérequis

Avant de configurer le tunnel IPv6, il faut s'assurer que la connectivité IPv4 est établie entre ESN eXia et le serveur Meraki dans le cloud.

> **Important :** Le tunnel IPv6 encapsule le trafic IPv6 dans des paquets IPv4 pour traverser Internet.

---

## Informations du Tunnel

### Adresses IPv4 (pour l'encapsulation)

- **Adresse Publique ESN eXia :** 101.0.0.2 (Interface G0/0/0)
- **Adresse Publique Meraki :** 90.154.127.203 (Interface S0/0/0)

### Réseaux IPv6

- **Réseau IPv6 Bureau (eXia) :** 2001:DB8:2000::/64
- **Réseau IPv6 Cloud (Meraki) :** 2001:DB8:1000::/64
- **Réseau IPv6 Tunnel :** 2001:DB8:3000::/64

---

## Configuration IPv4 pour atteindre le Cloud

Avant de créer le tunnel IPv6, il faut configurer les routes IPv4 pour que le réseau eXia puisse atteindre le serveur Meraki.

### FAI 1

**Vers FAI 4**
```cisco
Routeur_FAI1(config)#ip route 90.154.127.0 255.255.255.0 80.0.0.18
```

**Vers DSLAM**
```cisco
Routeur_FAI1(config)#ip route 101.0.0.0 255.255.255.252 80.0.0.5
```

---

### FAI 4

**Vers FAI 5**
```cisco
Routeur_FAI4(config)#ip route 90.154.127.0 255.255.255.0 80.0.0.25
```

**Vers FAI 1**
```cisco
Routeur_FAI4(config)#ip route 0.0.0.0 0.0.0.0 80.0.0.17
```

---

### FAI 5

**Vers FAI 4**
```cisco
Routeur_FAI5(config)#ip route 0.0.0.0 0.0.0.0 80.0.0.26
```

---

### Routeur Meraki

**Vers FAI 5**
```cisco
Routeur_Meraki(config)#ip route 0.0.0.0 0.0.0.0 90.154.127.254
```

> **Note :** Une fois que l'IPv4 permet d'atteindre le réseau du serveur Meraki (ping réussi), on peut mettre en place le tunnel IPv6.

---

## Configuration du Tunnel IPv6

### 1. Configuration du Routeur ESN eXia (Côté Bureau)

#### Activation de l'IPv6
```cisco
Routeur_eXia>enable
Routeur_eXia#conf t
Routeur_eXia(config)#ipv6 unicast-routing
```

#### Configuration du réseau local IPv6

On ajoute l'IPv6 sur le réseau interne en plus de l'IPv4 existante.
```cisco
Routeur_eXia(config)#interface FastEthernet0/0
Routeur_eXia(config-if)#ipv6 address 2001:DB8:2000::1/64
Routeur_eXia(config-if)#exit
```

#### Création du Tunnel

On crée une interface virtuelle qui encapsule l'IPv6 dans l'IPv4 pour traverser Internet.
```cisco
Routeur_eXia(config)#interface Tunnel0
Routeur_eXia(config-if)#ipv6 address 2001:DB8:3000::1/64
Routeur_eXia(config-if)#tunnel source GigabitEthernet0/0/0
Routeur_eXia(config-if)#tunnel destination 90.154.127.203
Routeur_eXia(config-if)#tunnel mode ipv6ip
Routeur_eXia(config-if)#exit
```

#### Configuration de la route IPv6 vers le Cloud
```cisco
Routeur_eXia(config)#ipv6 route 2001:DB8:1000::/64 2001:DB8:3000::2
```

---

### 2. Configuration du Routeur Meraki (Côté Cloud)

#### Activation de l'IPv6
```cisco
Routeur_Meraki>enable
Routeur_Meraki#conf t
Routeur_Meraki(config)#ipv6 unicast-routing
```

#### Configuration du réseau du serveur IPv6
```cisco
Routeur_Meraki(config)#interface GigabitEthernet0/2/0
Routeur_Meraki(config-if)#ipv6 address 2001:DB8:1000::254/64
Routeur_Meraki(config-if)#no shutdown
Routeur_Meraki(config-if)#exit
```

#### Création du Tunnel
```cisco
Routeur_Meraki(config)#interface Tunnel0
Routeur_Meraki(config-if)#ipv6 address 2001:DB8:3000::2/64
Routeur_Meraki(config-if)#tunnel source Serial0/0/0
Routeur_Meraki(config-if)#tunnel destination 101.0.0.2
Routeur_Meraki(config-if)#tunnel mode ipv6ip
Routeur_Meraki(config-if)#exit
```

#### Configuration de la route IPv6 vers le Bureau
```cisco
Routeur_Meraki(config)#ipv6 route 2001:DB8:2000::/64 2001:DB8:3000::1
```

---

## Configuration des Équipements

### Serveur Meraki (Desktop > IP Config)

- **IPv6 Address :** 2001:DB8:1000::1
- **Prefix :** /64
- **Passerelle IPv6 :** 2001:DB8:1000::254

### PC Bureau eXia (Desktop > IP Config)

- **IPv6 :** Cocher **Auto Config**

L'auto-configuration IPv6 (SLAAC) permettra au PC d'obtenir automatiquement une adresse IPv6 dans le réseau 2001:DB8:2000::/64.

---

## Tests de Connectivité

### Test 1 : Vérifier la connectivité IPv4 (avant tunnel)

Depuis un PC dans le réseau eXia :
```bash
ping 90.154.127.203
```

**Résultat attendu :** Réponse réussie → la connectivité IPv4 est établie.

---

### Test 2 : Vérifier le tunnel IPv6

Depuis le routeur eXia :
```cisco
Routeur_eXia#show ipv6 interface brief
Routeur_eXia#show interface tunnel 0
```

**Résultat attendu :** Interface Tunnel0 up/up

---

### Test 3 : Ping IPv6 vers le serveur Meraki

Depuis un PC Bureau eXia (avec IPv6 auto-configuré) :
```bash
ping 2001:DB8:1000::1
```

**Résultat attendu :** Réponse réussie → le tunnel IPv6 fonctionne !

---

## Architecture du Tunnel - Partie 2

### Schéma de connectivité
```
PC Bureau eXia (IPv6: 2001:DB8:2000::x)
    ↓
Routeur_eXia (Interface LAN IPv6: 2001:DB8:2000::1)
    ↓
Tunnel0 (IPv6: 2001:DB8:3000::1) ← Encapsulation IPv6 dans IPv4
    ↓
Internet (IPv4: 101.0.0.2 → 90.154.127.203)
    ↓
Tunnel0 (IPv6: 2001:DB8:3000::2) ← Décapsulation
    ↓
Routeur_Meraki (Interface Cloud IPv6: 2001:DB8:1000::254)
    ↓
Serveur Meraki (IPv6: 2001:DB8:1000::1)
```

### Plan d'adressage IPv6

| Équipement | Interface | Adresse IPv6 |
|------------|-----------|--------------|
| PC Bureau eXia | Ethernet | 2001:DB8:2000::x (auto) |
| Routeur eXia LAN | Fa0/0 | 2001:DB8:2000::1/64 |
| Routeur eXia Tunnel | Tunnel0 | 2001:DB8:3000::1/64 |
| Routeur Meraki Tunnel | Tunnel0 | 2001:DB8:3000::2/64 |
| Routeur Meraki Cloud | G0/2/0 | 2001:DB8:1000::254/64 |
| Serveur Meraki | Ethernet | 2001:DB8:1000::1/64 |

---

## Technologies Utilisées - Partie 2

- **Tunnel IPv6 over IPv4 (6in4) :** Encapsulation IPv6 dans IPv4
- **SLAAC :** Auto-configuration sans état pour les clients IPv6
- **Routage statique IPv6 :** Routes manuelles sur les routeurs
- **Double stack :** IPv4 et IPv6 coexistent sur le réseau eXia
