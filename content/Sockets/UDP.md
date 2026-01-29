+++
title = 'UDP'
date = 2024-02-06T12:05:07-05:00
weight = 20
draft = false
+++

---



#### UDP - Client
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
```
sudo apt update
sudo apt install netcat-openbsd
```
Vous pouvez tester ce programme comme suit: à partir d'un hôte sur le même réseau que le Pi, ouvrez un port UDP en mode "listen" avec la commande `nc`. Cet hôte joue donc le rôle de serveur. Par exemple, pour ouvrir le port UDP 8888 la commande est la suivante :
```bash
nc -ulp 8888
``` 

> Assurez-vous que la variable `DEST_IP` du programme a bien comme valeur l'adresse de l'hôte où vous avez lancé la commande `nc`.

> Vous pouvez ajouter l'option `-k` si vous voulez gardez la connexion ouverte.


#### UDP - Serveur
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

