# 📊 Plan d'Adressage Global

## 🏢 Réseaux Privés des Entreprises

### ESN eXia

| Protocole | CIDR | Réseau | Plage utilisable | Hôtes disponibles |
|-----------|------|--------|------------------|-------------------|
| **IPv4** | `/24` | `192.168.1.0` | `192.168.1.1 - 192.168.1.254` | 254 |
| **IPv6** | `/64` | `2001:DB8:2000::` | `2001:DB8:2000::1 - 2001:DB8:2000:ffff:ffff:ffff:ffff` | 3,4 × 10³⁸ |

---

### Bibliothèque Municipale

| CIDR | Réseau | Plage utilisable | Hôtes disponibles |
|------|--------|------------------|-------------------|
| `/24` | `192.168.0.0` | `192.168.0.1 - 192.168.0.254` | 254 |

---

### ENGIE (Segmentation VLSM)

| VLAN | Nom | CIDR | Réseau | Plage utilisable | Hôtes |
|------|-----|------|--------|------------------|-------|
| **10** | Technique | `/25` | `192.168.0.0` | `192.168.0.1 - 192.168.0.126` | 126 |
| **11** | Commercial | `/26` | `192.168.0.128` | `192.168.0.129 - 192.168.0.190` | 62 |
| **12** | Invités | `/27` | `192.168.0.192` | `192.168.0.193 - 192.168.0.222` | 30 |
| **-** | Routeur & Serveur | `/30` | `192.168.0.224` | `192.168.0.225 - 192.168.0.226` | 2 |

---

### DIGIPLEX (Architecture Multi-VLANs)

| VLAN | Nom | CIDR | Réseau | Plage utilisable | Hôtes |
|------|-----|------|--------|------------------|-------|
| **-** | Routeur ↔ SWL3 | `/30` | `192.168.0.0` | `192.168.0.1 - 192.168.0.2` | 2 |
| **10** | Conception | `/24` | `192.168.10.0` | `192.168.10.1 - 192.168.10.254` | 254 |
| **20** | Commercial | `/24` | `192.168.20.0` | `192.168.20.1 - 192.168.20.254` | 254 |
| **30** | Ressources Humaines | `/24` | `192.168.30.0` | `192.168.30.1 - 192.168.30.254` | 254 |
| **40** | Hotline | `/24` | `192.168.40.0` | `192.168.40.1 - 192.168.40.254` | 254 |
| **50** | WiFi Entreprise | `/24` | `192.168.50.0` | `192.168.50.1 - 192.168.50.254` | 254 |
| **60** | WiFi Invités | `/24` | `192.168.60.0` | `192.168.60.1 - 192.168.60.254` | 254 |
| **70** | Serveurs | `/24` | `192.168.70.0` | `192.168.70.1 - 192.168.70.254` | 254 |
| **80** | Management | `/24` | `192.168.80.0` | `192.168.80.1 - 192.168.80.254` | 254 |

---

## 🔗 Interconnexions WAN

### Liaisons Sites ↔ DSLAM FAI

| Site | CIDR | Réseau | IP Site | IP DSLAM |
|------|------|--------|---------|----------|
| **eXia** | `/30` | `101.0.0.0` | `101.0.0.2` | `101.0.0.1` |
| **Bibliothèque** | `/30` | `102.0.0.0` | `102.0.0.2` | `102.0.0.1` |
| **ENGIE** | `/30` | `103.0.0.0` | `103.0.0.2` | `103.0.0.1` |
| **DIGIPLEX** | `/30` | `104.0.0.0` | `104.0.0.2` | `104.0.0.1` |

---

## 🌐 Tunnel IPv6

### eXia ↔ Cloud Meraki

| Type | CIDR | Réseau | Description |
|------|------|--------|-------------|
| **Bureau eXia** | `/64` | `2001:DB8:2000::` | Réseau local IPv6 |
| **Tunnel** | `/64` | `2001:DB8:3000::` | Encapsulation 6in4 |
| **Cloud Meraki** | `/64` | `2001:DB8:1000::` | Serveur datacenter |

**Adresses tunnel :**
- eXia : `2001:DB8:3000::1`
- Meraki : `2001:DB8:3000::2`

---

## 🌍 Serveurs Publics

### Infrastructure Internet

| Service | CIDR | Réseau | Plage utilisable |
|---------|------|--------|------------------|
| **DNS** | `/24` | `8.8.8.0` | `8.8.8.1 - 8.8.8.254` |
| **Google** | `/24` | `108.177.127.0` | `108.177.127.1 - 108.177.127.254` |

---

## 📈 Synthèse par type

### Réseaux privés (RFC 1918)
```
192.168.0.0/16  → Sites locaux (eXia, Bibliothèque, ENGIE, DIGIPLEX)
```

### Réseaux publics simulés
```
101.0.0.0/30    → eXia ↔ Internet
102.0.0.0/30    → Bibliothèque ↔ Internet
103.0.0.0/30    → ENGIE ↔ Internet
104.0.0.0/30    → DIGIPLEX ↔ Internet
8.8.8.0/24       → DNS Public
108.177.127.0/24 → Google
```

### IPv6 (RFC 3849 - Documentation)
```
2001:DB8:1000::/64 → Cloud Meraki
2001:DB8:2000::/64 → Bureau eXia
2001:DB8:3000::/64 → Tunnel 6in4
```

---

## 🎯 Résumé statistique

| Catégorie | Nombre de réseaux | Total d'hôtes |
|-----------|-------------------|---------------|
| **Entreprises** | 4 | ~3 000 |
| **VLANs** | 12 | ~2 500 |
| **Interconnexions** | 4 | 8 |
| **Tunnels IPv6** | 1 | ∞ |

---

<p align="center">
  <sub>Plan d'adressage optimisé avec VLSM pour une utilisation efficace de l'espace d'adressage</sub>
</p>
