# network-security-lab
# 🛡️ Home Network Security Lab

Mini lab de sécurité réseau simulant une infrastructure d'entreprise
segmentée, protégée par un pare-feu pfSense et surveillée par un IDS Suricata.

## 🎯 Objectif

Concevoir et documenter une architecture réseau sécurisée avec
segmentation (LAN / DMZ), filtrage du trafic, NAT et détection d'intrusion,
puis valider le tout par des tests offensifs et défensifs.

## 🧰 Technologies

| Outil | Rôle |
|-------|------|
| pfSense | Pare-feu / routeur / NAT |
| Suricata | IDS (détection d'intrusion) |
| Wireshark | Capture et analyse du trafic |
| Ubuntu Server | Serveur en DMZ |
| Kali Linux | Machine d'attaque / de test (LAN) |
| VirtualBox / VMware | Virtualisation |

## 🗺️ Architecture

```
            Internet
                |
             pfSense
                |
        -----------------
        |               |
       LAN             DMZ
        |               |
      Kali         Ubuntu Server
```

| Zone | Réseau | Machine | Rôle |
|------|--------|---------|------|
| WAN | DHCP (NAT VirtualBox) | pfSense | Accès Internet |
| LAN | 192.168.10.0/24 | Kali Linux | Poste client / attaquant |
| DMZ | 192.168.20.0/24 | Ubuntu Server | Serveur exposé |

*(Adapte les plages IP à ton lab)*

![Schéma réseau](screenshots/architecture.png)

## ⚙️ Configuration

### 1. Installation de pfSense
Création de la VM, configuration des interfaces WAN/LAN/DMZ.
![Installation](screenshots/01-install-pfsense.png)

### 2. Segmentation réseau (VLAN)
Création et assignation des VLAN pour isoler les zones.
![VLAN](screenshots/02-vlan.png)

### 3. Règles de pare-feu
Politique appliquée : *deny by default*, autorisations au cas par cas.
![Firewall](screenshots/03-firewall-rules.png)

### 4. NAT
Configuration du NAT sortant et du port forwarding vers la DMZ.
![NAT](screenshots/04-nat.png)

### 5. IDS Suricata
Installation du package, activation des règles (ET Open) sur les interfaces.
![Suricata](screenshots/05-suricata.png)

### 6. Capture Wireshark
Analyse du trafic entre les zones.
![Wireshark](screenshots/06-wireshark.png)

## ✅ Tests et résultats

| Test | Attendu | Résultat |
|------|---------|----------|
| Ping LAN → DMZ | Autorisé | ✅ |
| SSH LAN → DMZ | Bloqué | ✅ |
| DMZ → LAN (toute connexion) | Bloqué | ✅ |
| Scan Nmap depuis Kali | Détecté par Suricata | ✅ |

## 📚 Ce que j'ai appris

- Principes de segmentation réseau et de défense en profondeur
- Écriture de règles firewall selon le moindre privilège
- Fonctionnement d'un IDS et analyse des alertes
- Lecture de paquets avec Wireshark

## 🚀 Pistes d'amélioration

- Ajout d'un SIEM (Wazuh / ELK) pour centraliser les logs
- Mise en place d'un VPN (OpenVPN / WireGuard)
- Écriture de règles Suricata personnalisées
- Ajout d'un serveur web vulnérable en DMZ (DVWA)

## 📁 Structure du dépôt

```
.
├── README.md
├── screenshots/
├── configs/        # exports de config (sans données sensibles)
└── docs/           # notes détaillées
```

## ⚠️ Avertissement

Ce lab est réalisé à des fins pédagogiques, dans un environnement isolé.
