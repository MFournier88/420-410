+++
type = "chapter"
pre = "2. "
title = "Sockets"
weight = 2
draft = false
+++

## Protocoles de communication : TCP vs UDP

Pour établir une communication entre un Raspberry Pi et un autre hôte, on utilise l'approche **client-serveur**. La distinction entre les deux repose sur l'initiative de la communication :

* Le **serveur** est celui qui se prépare à recevoir : il "écoute" sur un port spécifique.
* Le **client** est celui qui initie la communication : il s'adresse à l'adresse IP et au port du serveur.

### 1. TCP (Transmission Control Protocol) : Le mode connecté

TCP est comparable à un **appel téléphonique**. Une ligne est établie avant que l'échange ne commence.

* **Connexion :** Une connexion est établie (Handshake) entre le client et le serveur.
* **Fiabilité :** Le protocole vérifie que chaque message est bien arrivé. Si un paquet est perdu, il est renvoyé.
* **Ordre :** Les messages arrivent exactement dans l'ordre où ils ont été envoyés.
* **Clôture :** La connexion doit être explicitement fermée par l'un des deux hôtes.

### 2. UDP (User Datagram Protocol) : Le mode non-connecté

UDP est comparable à **l'envoi de lettres par la poste**. On envoie un paquet sans savoir si le destinataire est présent.

* **Pas de connexion :** Le client envoie des messages au serveur sans établir de liaison préalable.
* **Rapidité :** Il n'y a pas de vérification de réception, ce qui le rend beaucoup plus léger et rapide.
* **Incertitude :** On ne peut pas savoir si le serveur est en ligne ou s'il a bien reçu le message. Si un message est perdu, il est perdu pour de bon.

---

### Les Sockets : le point d'entrée

Pour utiliser ces protocoles, on utilise des **sockets**.

Un socket représente le point terminal d'une communication. C'est la "prise" sur laquelle ton programme se branche pour envoyer ou recevoir des données. Un socket est défini par deux éléments :

1. **Une adresse IP** (pour trouver l'hôte sur le réseau).
2. **Un port** (pour trouver le programme spécifique sur cet hôte).

| Type de Socket | Protocole associé | Constante Python |
| --- | --- | --- |
| **Stream Socket** | TCP | `socket.SOCK_STREAM` |
| **Datagram Socket** | UDP | `socket.SOCK_DGRAM` |

---

