# Déploiement ESN eXia

## Table des matières
- [Configuration Routeur](#configuration-routeur)
- [Configuration Switch](#configuration-switch)
- [Configuration Borne Wi-Fi](#configuration-borne-wi-fi)
- [Configuration Serveur DNS & FTP](#configuration-serveur-dns--ftp)
- [Configuration Postes Clients](#configuration-postes-clients)
- [Tests](#tests)
- [Finalisation Accès Web (NAT & Routage)](#finalisation-accès-web-nat--routage)

---

## Configuration Routeur

Le routeur est déjà en place, nous procédons donc uniquement à sa configuration pour sécuriser les accès administratifs via le chiffrement des mots de passe et configurer l'interface de sortie (FastEthernet0/0) avec la bonne passerelle pour le réseau de l'ESN eXia, en activant le NAT pour la sortie vers l'extérieur.
```cisco
Router>enable
Router#configure terminal
Router(config)#hostname Routeur_eXia
 
Routeur_eXia(config)#enable secret ciscosuperpass
Routeur_eXia(config)#service password-encryption
Routeur_eXia(config)#line console 0
Routeur_eXia(config-line)#password cisco
Routeur_eXia(config-line)#login
Routeur_eXia(config-line)#exit
 
Routeur_eXia(config)#interface Fa0/0
Routeur_eXia(config-if)#ip address 192.168.1.254 255.255.255.0
Routeur_eXia(config-if)#no shutdown
Routeur_eXia(config-if)#ip nat inside
Routeur_eXia(config-if)#exit
```

---

## Configuration Switch

Nous installons un switch 2960-24TT relié au routeur. Nous sécurisons d'abord l'accès console avec un mot de passe chiffré. Ensuite, nous configurons la sécurité des ports (Port Security) : les ports 1 à 5 (utilisés par les équipements) sont configurés pour apprendre l'adresse MAC de l'appareil branché (sticky) et se couper en cas de changement. Les ports 6 à 24 (inutilisés) sont configurés avec une adresse MAC fictive pour provoquer une coupure immédiate si quelqu'un tente de se brancher dessus.
```cisco
Switch>enable
Switch#configure terminal
Switch(config)#hostname Switch_eXia
 
Switch_eXia(config)#enable secret password_switch
Switch_eXia(config)#service password-encryption
Switch_eXia(config)#line console 0
Switch_eXia(config-line)#password password_switch
Switch_eXia(config-line)#login
Switch_eXia(config-line)#exit

Switch_eXia(config)#interface range fa0/1-5
Switch_eXia(config-if)#switchport mode access
Switch_eXia(config-if)#switchport port-security
Switch_eXia(config-if)#switchport port-security maximum 1
Switch_eXia(config-if)#switchport port-security violation shutdown
Switch_eXia(config-if)#switchport port-security mac-address sticky
Switch_eXia(config-if)#exit

Switch_eXia(config)#interface range fa0/6-24
Switch_eXia(config-if)#switchport mode access
Switch_eXia(config-if)#switchport port-security
Switch_eXia(config-if)#switchport port-security maximum 1
Switch_eXia(config-if)#switchport port-security violation shutdown
Switch_eXia(config-if)#switchport port-security mac-address 0000.0000.0000
```

---

## Configuration Borne Wi-Fi

Pour le réseau sans fil, nous appliquons la sécurité la plus haute disponible (WPA2-PSK avec chiffrement AES) afin de protéger l'accès au réseau local via la borne Wi-Fi.

### Paramètres Wi-Fi
- **SSID :** WiFi_eXia
- **Authentification :** WPA2-PSK
- **Encryption :** AES
- **Mot de passe :** password_eXia

---

## Configuration Serveur DNS & FTP

Nous attribuons une IP fixe au serveur (192.168.1.200) et nous activons deux services essentiels : le FTP pour le transfert de fichiers avec un utilisateur dédié (eXia_ftp), et le DNS pour la résolution de noms, en ajoutant une entrée spécifique pour www.google.com pointant vers l'IP 108.177.127.139.

### Configuration IP
- **IPv4 adresse :** 192.168.1.200
- **Masque :** 255.255.255.0
- **Passerelle par défaut :** 192.168.1.254
- **DNS :** 127.0.0.1

### Service FTP
- **Nom :** eXia_ftp
- **Mot de passe :** password_ftp

### Service DNS
- **Nom :** www.google.com 
- **Type :** A Record
- **Adresse :** 108.177.127.139

---

## Configuration Postes Clients

Les postes fixes et le portable sont configurés avec un adressage statique dans le réseau 192.168.1.0/24. Ils utilisent tous le routeur comme passerelle pour sortir sur le web et notre serveur local (192.168.1.200) comme DNS pour la résolution de noms.

### PC Fixe 1
- **IP :** 192.168.1.11
- **Masque :** 255.255.255.0
- **Passerelle :** 192.168.1.254
- **DNS :** 192.168.1.200

### PC Fixe 2
- **IP :** 192.168.1.12
- **Masque :** 255.255.255.0 
- **Passerelle :** 192.168.1.254
- **DNS :** 192.168.1.200 

### PC Portable 1
- **IP :** 192.168.1.21
- **Masque :** 255.255.255.0
- **Passerelle :** 192.168.1.254
- **DNS :** 192.168.1.200

---

## Tests

Nous validons l'installation par deux tests :

1. **Test FTP :** Une connexion FTP depuis un poste vers le serveur (192.168.1.200) pour vérifier l'authentification.
2. **Test DNS :** Un ping vers www.google.com pour confirmer que le serveur DNS renvoie bien l'adresse IP spécifique configurée (108.177.127.139).

---

## Finalisation Accès Web (NAT & Routage)

Pour permettre aux postes de l'entreprise d'accéder au Web, nous devons configurer l'interface connectée à Internet (WAN), activer la traduction d'adresses (NAT) pour que les IP privées (192.168.1.x) puissent sortir, et définir une route par défaut vers Internet.
```cisco
Routeur_eXia(config)#interface G0/0/0
Routeur_eXia(config-if)#ip address dhcp
Routeur_eXia(config-if)#no shutdown
Routeur_eXia(config-if)#ip nat outside
Routeur_eXia(config-if)#exit
Routeur_eXia(config)#access-list 1 permit 192.168.1.0 0.0.0.255
Routeur_eXia(config)#ip nat inside source list 1 interface G0/0/0 overload
Routeur_eXia(config)#ip route 0.0.0.0 0.0.0.0 G0/0/0
```

---

## Architecture Réseau

### Plan d'adressage
- **Réseau :** 192.168.1.0/24
- **Passerelle :** 192.168.1.254 (Routeur)
- **Serveur DNS/FTP :** 192.168.1.200
- **Postes fixes :** 192.168.1.11 - 192.168.1.12
- **Postes portables :** 192.168.1.21

### Équipements
- **Routeur :** Routeur_eXia
- **Switch :** Switch_eXia (2960-24TT)
- **Borne Wi-Fi :** WiFi_eXia (WPA2-PSK/AES)
- **Serveur :** DNS & FTP (192.168.1.200)
