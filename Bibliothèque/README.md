# Déploiement Bibliothèque

## Table des matières
- [Configuration Routeur & Services](#configuration-routeur--services)
- [Configuration du Switch](#configuration-du-switch)
- [Configuration de la Borne Wi-Fi](#configuration-de-la-borne-wi-fi)
- [Tests](#tests)

---

## Configuration Routeur & Services

Le routeur est le cœur du réseau. Nous sécurisons son accès physique et configurons un accès à distance chiffré via SSH. Cela nécessite la création d'un utilisateur local et la génération de clés cryptographiques. En parallèle, nous configurons l'interface LAN avec une IP statique (192.168.0.254). Pour répondre au besoin de flexibilité, nous configurons un serveur DHCP qui distribuera automatiquement les adresses IP aux PC et portables. Enfin, nous activons le NAT sur l'interface WAN pour permettre l'accès au Web.

### Configuration du nom
```cisco
Router>enable
Router#conf t
Router(config)#hostname Routeur_Bibliotheque
```

### Configuration des mots de passe
```cisco
Routeur_Bibliotheque(config)#enable secret password_routeur
Routeur_Bibliotheque(config)#service password-encryption
Routeur_Bibliotheque(config)#line console 0
Routeur_Bibliotheque(config-line)#password password_routeur
Routeur_Bibliotheque(config-line)#login
Routeur_Bibliotheque(config-line)#exit
```

### Configuration de l'accès à distance SSH
```cisco
Routeur_Bibliotheque(config)#ip domain-name bibliotheque.funky
Routeur_Bibliotheque(config)#username admin secret password_routeur_ssh
Routeur_Bibliotheque(config)#crypto key generate rsa
```

### Forcer la vérification de l'utilisateur pour le SSH
```cisco
Routeur_Bibliotheque(config)#line vty 0 4
Routeur_Bibliotheque(config-line)#login local
Routeur_Bibliotheque(config-line)#transport input ssh
Routeur_Bibliotheque(config-line)#exit
```

### Configuration du port LAN
```cisco
Routeur_Bibliotheque(config)#interface Fa0/0
Routeur_Bibliotheque(config-if)#ip address 192.168.0.254 255.255.255.0
Routeur_Bibliotheque(config-if)#no shutdown
Routeur_Bibliotheque(config-if)#ip nat inside
Routeur_Bibliotheque(config-if)#exit
```

### Configuration du DHCP sur le routeur
```cisco
Routeur_Bibliotheque(config)#ip dhcp pool LAN_BIBLIOTHEQUE
Routeur_Bibliotheque(dhcp-config)#network 192.168.0.0 255.255.255.0
Routeur_Bibliotheque(dhcp-config)#default-router 192.168.0.254
Routeur_Bibliotheque(dhcp-config)#dns-server 8.8.8.8
Routeur_Bibliotheque(dhcp-config)#exit
```

### Configuration de l'interface WAN et NAT
```cisco
Routeur_Bibliotheque(config)#interface G0/0/0
Routeur_Bibliotheque(config-if)#ip address dhcp
Routeur_Bibliotheque(config-if)#no shutdown
Routeur_Bibliotheque(config-if)#ip nat outside
Routeur_Bibliotheque(config-if)#exit
Routeur_Bibliotheque(config)#access-list 1 permit 192.168.0.0 0.0.0.255
Routeur_Bibliotheque(config)#ip nat inside source list 1 interface G0/0/0 overload 
Routeur_Bibliotheque(config)#ip route 0.0.0.0 0.0.0.0 G0/0/0
```

---

## Configuration du Switch

Le switch assure la connexion des équipements. Nous sécurisons son accès physique et configurons l'administration à distance via SSH. Pour cela, le switch a besoin d'une adresse IP de gestion (VLAN 1) et d'une passerelle par défaut pour répondre aux requêtes venant d'autres réseaux.

### Configuration du nom
```cisco
Switch>enable
Switch#conf t
Switch(config)#hostname Switch_Bibliotheque
```

### Configuration des mots de passe
```cisco
Switch_Bibliotheque(config)#enable secret password_switch
Switch_Bibliotheque(config)#service password-encryption
Switch_Bibliotheque(config)#line console 0
Switch_Bibliotheque(config-line)#password password_switch
Switch_Bibliotheque(config-line)#login
Switch_Bibliotheque(config-line)#exit
```

### Configuration de la connexion en SSH
```cisco
Switch_Bibliotheque(config)#ip domain-name bibliotheque.funky
Switch_Bibliotheque(config)#username admin secret password_switch_ssh
Switch_Bibliotheque(config)#crypto key generate rsa
```

### Application du SSH sur les lignes virtuelles

Obliger le switch à n'accepter que les connexions à distance chiffrées (SSH) pour empêcher qu'un pirate n'intercepte vos mots de passe.
```cisco
Switch_Bibliotheque(config)#line vty 0 15
Switch_Bibliotheque(config-line)#transport input ssh
Switch_Bibliotheque(config-line)#login local
Switch_Bibliotheque(config-line)#exit
```

### Configurer une IP de gestion
```cisco
Switch_Bibliotheque(config)#interface vlan 1
Switch_Bibliotheque(config-if)#ip address 192.168.0.253 255.255.255.0
Switch_Bibliotheque(config-if)#no shutdown
Switch_Bibliotheque(config-if)#exit
```

### Configurer la passerelle
```cisco
Switch_Bibliotheque(config)#ip default-gateway 192.168.0.254
```

---

## Configuration de la Borne Wi-Fi

Conformément à la politique d'accès libre de la bibliothèque municipale, la borne Wi-Fi est configurée pour offrir un accès ouvert au public, sans mot de passe, facilitant la connexion des usagers.

### Paramètres Wi-Fi
- **SSID :** Biblio_Gratuit
- **Authentification :** Disabled (Open)
- **Encryption :** None
- **Mot de passe :** Aucun

---

## Tests

Nous validons le déploiement par les tests suivants :

### 1. Vérification DHCP et DNS

Sur un PC client, vérifier qu'il a bien obtenu une adresse IP automatiquement via la commande `ipconfig /all`.

**Résultat attendu :**
- **Adresse IP :** 192.168.0.x
- **Passerelle :** 192.168.0.254
- **Serveur DNS :** 8.8.8.8 (Cela confirme que la consigne du DNS public est respectée)

### 2. Test d'accès distant sécurisé pour le routeur (SSH)

Depuis un PC, nous testons l'administration du routeur via la commande :
```bash
ssh -l admin 192.168.0.254
```

L'option `-l` permet de spécifier le login "admin". Le mot de passe `password_routeur_ssh` sera demandé, validant ainsi que la connexion est chiffrée et authentifiée.

### 3. Test Switch SSH

Depuis un PC, taper la commande :
```bash
ssh -l admin 192.168.0.253
```

Si le switch demande le mot de passe, l'administration sécurisée est fonctionnelle.

---

## Architecture Réseau

### Plan d'adressage
- **Réseau :** 192.168.0.0/24
- **Passerelle :** 192.168.0.254 (Routeur)
- **IP de gestion Switch :** 192.168.0.253
- **Pool DHCP :** 192.168.0.x (attribution automatique)
- **DNS :** 8.8.8.8 (Google DNS)

### Équipements
- **Routeur :** Routeur_Bibliotheque
- **Switch :** Switch_Bibliotheque
- **Borne Wi-Fi :** Biblio_Gratuit (Open Access)

### Services
- **DHCP :** Activé sur le routeur
- **SSH :** Activé sur routeur et switch
- **NAT :** Configuré pour l'accès Internet
- **DNS :** Serveur public Google (8.8.8.8)
