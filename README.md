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
| pfSense 2.9.0 | Pare-feu / routeur / NAT |
| Suricata | IDS (détection d'intrusion) |
| Wireshark | Capture et analyse du trafic |
| Debian 13 | Serveur en DMZ (Apache, SSH) |
| Kali Linux | Machine d'attaque / de test (LAN) |
| VMware Workstation | Virtualisation |

## 🗺️ Architecture

```
            Internet
                |
          NAT VMware (VMnet8)
                |
             pfSense
                |
        -----------------
        |               |
       LAN             DMZ
        |               |
      Kali         Debian Server
```

| Zone | Réseau | Machine | Rôle |
|------|--------|---------|------|
| WAN | DHCP (NAT VMware, VMnet8) | pfSense (em0) | Accès Internet |
| LAN | 192.168.1.0/24 (segment `lan`) | Kali Linux | Poste client / attaquant |
| DMZ | 192.168.2.0/24 (segment `dmz`) | Debian Server (192.168.2.10) | Serveur exposé |

La segmentation repose sur des **LAN Segments VMware** isolés (`lan` et `dmz`) :
chaque zone est sur un réseau virtuel distinct, et tout le trafic entre elles
passe obligatoirement par pfSense.

![Schéma réseau](screenshots/architecture.png)

## ⚙️ Configuration

### 1. Installation de pfSense
VM pfSense à 3 cartes réseau (WAN en NAT, LAN et DMZ en LAN Segments),
interfaces assignées depuis la console, puis Setup Wizard via l'interface web.

![Console pfSense](screenshots/01-install-pfsense.png)
![Dashboard](screenshots/01b-pfsense-dashboard.png)

### 2. Interfaces et segmentation
Interfaces WAN, LAN et DMZ (renommée depuis OPT1), serveur DHCP sur le LAN.

![Interfaces](screenshots/02-interfaces.png)
![DHCP LAN](screenshots/02c-dhcp.png)

### 3. Installation des VM
- **Kali Linux** (LAN) : IP obtenue en DHCP.
- **Debian Server** (DMZ) : IP fixe `192.168.2.10/24`, Apache et OpenSSH installés.

![Kali IP](screenshots/03-kali-ip.png)
![Debian IP](screenshots/03b-debian-ip.png)

### 4. Règles de pare-feu
Politique : trafic DMZ → LAN bloqué, LAN → DMZ filtré au cas par cas
(ping autorisé, SSH bloqué). Les règles de blocage sont journalisées.

![Règles LAN](screenshots/04-firewall-lan.png)
![Règles DMZ](screenshots/04b-firewall-dmz.png)

### 5. NAT
NAT sortant (mode automatique) pour l'accès Internet des deux zones.

![NAT sortant](screenshots/05-nat-outbound.png)

### 6. Tests de connectivité
Comparaison avant/après l'application des règles.

![Ping autorisé](screenshots/06-ping-ok.png)
![SSH bloqué](screenshots/06b-ssh-blocked.png)
![Logs pare-feu](screenshots/06c-firewall-logs.png)

### 7. IDS Suricata
Installation du package, activation des règles ET Open sur l'interface surveillée.

![Installation](screenshots/07-suricata-install.png)
![Interfaces](screenshots/07b-suricata-interfaces.png)
![Règles](screenshots/07c-suricata-rules.png)

### 8. Test de détection
Scan Nmap depuis Kali vers le serveur DMZ, alertes générées dans Suricata.

![Nmap](screenshots/08-nmap-kali.png)
![Alertes](screenshots/08b-suricata-alerts.png)

### 9. Capture Wireshark
Analyse du scan Nmap (rafale de paquets SYN) avec le filtre
`tcp.flags.syn==1 && tcp.flags.ack==0`.

![Capture](screenshots/09-wireshark-capture.png)
![Filtre](screenshots/09b-wireshark-filter.png)

## ✅ Tests et résultats

| Test | Attendu | Résultat |
|------|---------|----------|
| Ping LAN → DMZ | Autorisé | ⬜ |
| SSH LAN → DMZ | Bloqué | ⬜ |
| DMZ → LAN (toute connexion) | Bloqué | ⬜ |
| Scan Nmap depuis Kali | Détecté par Suricata | ⬜ |

## 📚 Ce que j'ai appris

- Principes de segmentation réseau et de défense en profondeur
- Écriture de règles firewall selon le moindre privilège
- Fonctionnement d'un IDS et analyse des alertes
- Lecture de paquets avec Wireshark

## 🚀 Pistes d'amélioration

- Segmentation par VLAN (802.1Q) au lieu de réseaux séparés
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
