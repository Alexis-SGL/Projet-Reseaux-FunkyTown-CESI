<img src="[https://github.com/user-attachments/assets/311a3983-f518-4410-8429-bc025f07f575](https://alexis-sgl.fr/wp-content/uploads/2026/01/logo_CESI_projet_etudiant_NB.png)" alt="Logo CESI" width="100" align="right" />
<br><br>


# 🌐 Infrastructure Réseau FunkyTown

> Conception et déploiement d'une infrastructure réseau d'entreprise moderne connectant 4 sites via un datacenter centralisé avec accès Internet et tunnel IPv6 vers le cloud.

---

## 🎯 Vue d'ensemble

Infrastructure réseau complète reliant **4 organisations** via un datacenter, avec connectivité Internet sécurisée et tunnel IPv6 vers le cloud.

### Sites déployés

🏢 **ESN eXia** - Services Numériques  
📚 **Bibliothèque** - Accès public Wi-Fi  
⚡ **ENGIE** - Segmentation par VLANs  
🏗️ **DIGIPLEX** - Multi-étages avec switch L3  

---

## 🛠️ Stack Technique

**Réseau Local**
- VLANs & Segmentation
- DHCP & DNS
- NAT & Port Security

**Interconnexion**
- Routage statique
- EtherChannel (LACP)
- Switch Layer 3
- VTP

**Sécurité**
- SSH
- WPA2-PSK
- Chiffrement des mots de passe

**Avancé**
- Tunnel IPv6 over IPv4
- SLAAC (Auto-config IPv6)

---

## 📊 Architecture
```
┌─────────────┐
│   Internet  │
└──────┬──────┘
       │
┌──────▼──────┐
│ Datacenter  │
│   (DSLAM)   │
└──┬──┬──┬──┬─┘
   │  │  │  │
   │  │  │  └─────► DIGIPLEX (8 VLANs, L3 Switch)
   │  │  └────────► ENGIE (3 VLANs, DHCP Server)
   │  └───────────► Bibliothèque (Wi-Fi Public, DHCP)
   └──────────────► ESN eXia (IPv6 Tunnel ↔ Cloud)
```

---

## 🎨 Fonctionnalités clés

| Site | Highlights |
|------|-----------|
| **eXia** | 🔒 Port Security • 🌐 IPv6 Tunnel • 📁 DNS/FTP |
| **Bibliothèque** | 📶 Wi-Fi Ouvert • ⚙️ DHCP Auto • 🔐 SSH |
| **ENGIE** | 🏷️ 3 VLANs • 📊 VLSM • 🖥️ DHCP Centralisé |
| **DIGIPLEX** | 🏗️ 8 VLANs • 🔄 Switch L3 • ⚡ EtherChannel |

---

## 📈 Résultats

✅ Infrastructure évolutive et sécurisée  
✅ Connectivité Internet pour 4 sites  
✅ Segmentation réseau par département  
✅ Administration à distance SSH  
✅ Tunnel IPv6 vers le cloud  

---

## 🔧 Technologies

![Cisco](https://img.shields.io/badge/Cisco-Packet_Tracer-1BA0D7?style=flat-square&logo=cisco)
![IPv4](https://img.shields.io/badge/Protocol-IPv4-orange?style=flat-square)
![IPv6](https://img.shields.io/badge/Protocol-IPv6-blue?style=flat-square)
![VLANs](https://img.shields.io/badge/Network-VLANs-green?style=flat-square)
![SSH](https://img.shields.io/badge/Security-SSH-red?style=flat-square)

**Équipements** : Routeurs Cisco • Switches  
**Protocoles** : IPv4/IPv6 • SSH • DHCP • DNS • NAT • VTP • LACP  

---

Projet réalisé dans le cadre du module Réseaux et Système de l'école d'ingénieurs CESI.
