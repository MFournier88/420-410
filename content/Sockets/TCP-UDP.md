+++
title = 'TCP et UDP'
date = 2024-02-06T12:05:07-05:00
weight = 20
draft = false
+++

---

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

## Client
#### UDP
Le programme suivant envoit un message par UDP sur un réseau. L'adresse de destination est `127.0.0.1` et le port est `8888`:
```python
import socket

PORT = 8888
DEST_IP = "127.0.0.1"
MESSAGE = "abcdefg!\n"

# Créer le socket
sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

# Créer un objet pour l'adresse
dest_addr = (DEST_IP, PORT)

# Envoyer le message
sock.sendto(MESSAGE.encode(), dest_addr)

# Fermer le socket
sock.close()
```

Vous pouvez tester ce programme comme suit: à partir d'un hôte sur le même réseau que le Pi, ouvrez un port UDP en mode "listen" avec la commande `nc`. Cet hôte joue donc le rôle de serveur. Par exemple, pour ouvrir le port UDP 8888 la commande est la suivante :
```bash
nc -ulp 8888
``` 
> Assurez-vous que la variable `DEST_IP` du programme a bien comme valeur l'adresse de l'hôte où vous avez lancé la commande `nc`.

#### TCP
Le programme suivant envoit un message par TCP sur un réseau. L'adresse de destination est `127.0.0.1` et le port est `8888`:
```python
import socket

PORT = 8888
DEST_IP = "127.0.0.1"
MESSAGE = "abcdefg!\n"

# Create the socket
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Initialize the destination address
dest_addr = (DEST_IP, PORT)

# Create the connection
sock.connect(dest_addr)

# Send the message and close the connection
sock.send(MESSAGE.encode())
sock.close()
```
Vous pouvez tester ce programme avec la commande `nc`. À partir d'un hôte sur le même réseau que le Pi (qui aura le rôle de serveur), ouvrez un port TCP en mode "listen" au port TCP 8888:
```bash
nc -lp 8888
``` 
> Assurez-vous que la variable `DEST_IP` du programme a bien comme valeur l'adresse de l'hôte où vous avez lancé la commande `nc`; ensuite exécutez-le.


## Serveur
#### UDP
Le programme suivant ouvre un socket au port UDP 8888 et affiche les messages entrants:
```python
import socket

PORT = 8888
BUFFER_SIZE = 1024

# Créer le socket
socket_local = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
adr_local = ('', PORT)

# Lier le socket à l'adresse
socket_local.bind(adr_local)

print(f"On attend les messages UDP entrants au port {PORT}...")

# Réception des messages
while True:
    # Réception des données
    buffer, adr_dist = socket_local.recvfrom(BUFFER_SIZE)
    
    # Décodage binaire -> texte
    message = buffer.decode()

    print(f"Message reçu: {message}", end='')
```
Pour tester ce programme à partir d'un autre hôte (qui agit comme client), lancez la commande `nc` (remplacez 1.2.3.4 par l'adresse IP du serveur) et tapez le message que vous voulez envoyer:
```bash
nc -u 1.2.3.4 8888
abcdefg!
```

#### TCP
Le programme suivant ouvre un socket au port TCP 8888 et attend les connexions entrantes. Lorsque la connexion est établie il affiche les messages et termine la connexion lorsque le message reçu est vide (taille 0):

```python
import socket

PORT = 8888
BUFFER_SIZE = 1024

# Créer le socket
socket_local = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
address = ('', PORT)  

# Lier le socket à l'adresse
socket_local.bind(address)

# Attendre une connexion
socket_local.listen(3)
print(f"Serveur TCP en écoute au port {PORT}...")

socket_dist, client_address = socket_local.accept()
print(f"Socket distant: {client_address}")

# Réception des messages
while True:
    # Données
    data = socket_dist.recv(BUFFER_SIZE)
    if not data:
        print("Déconnecté")
        break
    else:
        # Décoder + afficher les message
        message = data.decode()
        print(f"Reçu: {message}", end='')

socket_dist.close()
socket_local.close()
```
Pour tester ce programme à partir d'un autre hôte (qui agit comme client), lancez la commande `nc` (remplacez 1.2.3.4 par l'adresse IP du serveur) et tapez le message que vous voulez envoyer:
```bash
nc 1.2.3.4 8888
abcdefg!
```
