+++
title = 'Wifi'
date = 2025-04-04T07:17:23-04:00
draft = false
weight = 61
+++

---

# 🛰️ Guide Complet : Réseau Privé P2P entre Raspberry Pi

Ce guide permet d'établir une connexion directe entre deux Raspberry Pi (AP Mode) pour échanger des données via Sockets Python, sans dépendre d'un routeur externe.

## 1. Architecture du réseau
Remplacer XXX par le numéro de votre (Pi + 100)
* **Pi A (Serveur/Point d'Accès) :** Crée le Wi-Fi. IP Statique : `192.168.XXX.1`.
* **Pi B (Client) :** Se connecte au Pi A. IP Dynamique (DHCP) : `192.168.XXX.10` à `100`.
* **Note sur l'IP :** Le sous-réseau `50.x` évite les conflits avec les box internet standards (souvent en `1.x` ou `0.x`).

---

## 2. Configuration du Serveur (Pi A)

### A. NetworkManager (Interface Radio)
On configure le Pi pour diffuser un signal Wi-Fi.
```bash

sudo nmcli radio wifi on

sudo rfkill unblock wifi

sudo ip link set wlan0 up

# 1. Sauvegarde et configuration de base
sudo cp /etc/NetworkManager/NetworkManager.conf /etc/NetworkManager/NetworkManager.conf.sauv
# Modifier /etc/NetworkManager/NetworkManager.conf : [main] plugins=keyfile, dns=none | [ifupdown] managed=true

# 2. Création du point d'accès
nmcli connection add type wifi ifname wlan0 mode ap con-name [NOM_PROFIL] ssid [SSID_DU_RESEAU]

# 3. Paramètres de sécurité et IP statique
sudo nmcli connection modify [NOM_PROFIL] 802-11-wireless.band bg
sudo nmcli connection modify [NOM_PROFIL] 802-11-wireless.channel 6
sudo nmcli connection modify [NOM_PROFIL] wifi-sec.key-mgmt wpa-psk
sudo nmcli connection modify [NOM_PROFIL] wifi-sec.psk "votre_mot_de_passe"
sudo nmcli connection modify [NOM_PROFIL] ipv4.method manual ipv4.addresses 192.168.XXX.1/24

# 4. Activation
sudo nmcli connection up [NOM_PROFIL]

sudo systemctl restart NetworkManager
```

### B. DHCP avec `dnsmasq` (Attribution des IP)
Permet aux clients de recevoir une adresse automatiquement.
1.  **Installation :** `sudo apt install dnsmasq`
2.  **Configuration :** Créer `/etc/dnsmasq.d/[NOM_PROFIL].conf` :
```conf
interface=wlan0
dhcp-range=192.168.XXX.10,192.168.XXX.100,12h
domain-needed
bogus-priv
port=0
```
3.  **Démarrage :** `sudo systemctl restart dnsmasq`

---

## 3. Configuration du Client (Pi B)

### A. Connexion au réseau
```bash
sudo nmcli radio wifi on

sudo rfkill unblock wifi

sudo ip link set wlan0 up

# 1. Configurer le pays (Crucial pour débloquer les fréquences)
sudo raspi-config # Localisation Options -> WLAN Country -> CA ou FR

# 2. Scanner et se connecter
nmcli device wifi list
sudo nmcli device wifi connect "[SSID_DU_RESEAU]" password "[MOT_DE_PASSE]"

# 3. Vérification
nmcli device status # wlan0 doit être "connected"


sudo systemctl restart NetworkManager
```

---

## 4. Partage de données (Python Sockets)

Va voir dans les notes de cours!

---

## 5. Diagnostic et "Quick Fix"

### Forcer le trafic vers le Wi-Fi (si l'Ethernet bloque)
Si le Pi tente d'envoyer les données par le câble au lieu du Wi-Fi :
```bash
sudo ip route add 192.168.XXX.0/24 dev wlan0 metric 10
```

### Commandes de vérification
* `rfkill list` : Vérifier que le Wi-Fi n'est pas bloqué logiciellement (`Soft blocked: no`).
* `nmcli radio wifi on` : Forcer l'activation de la puce Wi-Fi.
* `ping 192.168.XXX.1` : Test ultime de connectivité.
* `nc -ulp 8888` : Simuler un serveur UDP pour tester la réception sans code Python.
sudo nmcli connection delete "[SSID_DU_RESEAU]"
---

## 6. Pourquoi ce mode plutôt que le Wi-Fi domestique ?

| Avantage | Description |
| :--- | :--- |
| **Autonomie** | Le système fonctionne partout (robotique mobile, extérieur). |
| **Latence** | Communication directe "circuit court", plus rapide que via une box. |
| **Sécurité** | Réseau privé étanche, isolé des autres appareils de la maison. |
| **Performance** | Bande passante dédiée, non partagée avec le streaming de la famille. |

> **Attention :** En mode Point d'Accès, le Pi A perd son accès à Internet par Wi-Fi. Pour revenir en mode client, il faut restaurer le fichier `NetworkManager.conf` d'origine.


# Informatif : Pourquoi 192.168
Pour comprendre pourquoi on utilise précisément **192** et **168**, il faut descendre d'un étage et regarder comment les ordinateurs lisent ces chiffres : en **binaire** (des 0 et des 1).

Voici l'explication technique, mais simplifiée, de chaque bloc :

### 1. Pourquoi "192" ? (La Classe du réseau)
Le premier nombre définit la "taille" du réseau. En binaire, **192** s'écrit `11000000`.

* **La règle d'or :** Dans les années 80, on a décidé que tous les réseaux de **Classe C** (petits réseaux pour la maison ou les PME) devaient obligatoirement commencer par les bits `110`.
* Le plus petit nombre commençant par `110` est **192**.
* **Résultat :** Quand un Raspberry Pi voit "192", il sait immédiatement : *"Ok, je suis dans un petit réseau local, pas dans un réseau géant comme celui du gouvernement (Classe A)."*



---

### 2. Pourquoi "168" ? (La protection privée)
Le deuxième nombre, **168**, est là pour la **sécurité**. 

À l'époque, on avait peur de manquer d'adresses IP sur Internet. On a donc créé la norme **RFC 1918**, qui a "réservé" certains numéros pour qu'ils ne soient **jamais** utilisés sur le vrai Internet.
* Pour la Classe C, on a réservé tout le bloc qui va de `192.168.0.0` à `192.168.255.255`.
* **L'avantage :** Comme l'adresse `192.168.x.x` est "interdite" sur Internet, ton routeur ou ton Raspberry Pi sait que s'il reçoit un message pour cette adresse, il doit le garder pour lui et **ne jamais l'envoyer vers l'extérieur**.

---

### En résumé pour tes notes :

| Nombre | Signification Technique | Ce que ça veut dire pour ton Pi |
| :--- | :--- | :--- |
| **192** | **Identifiant de Classe C** | "C'est un petit réseau local." |
| **168** | **Extension Privée (RFC 1918)** | "C'est un réseau confidentiel, invisible d'Internet." |
| **XXX** | **Ton numéro de projet** | "C'est le nom de ma bulle Wi-Fi spécifique." |
| **1** | **Identifiant de la machine** | "C'est l'adresse précise du Pi A." |

