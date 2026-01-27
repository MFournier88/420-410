+++
title = 'TCP'
date = 2026-01-26T20:19:07-05:00
weight = 21
+++


---

#### Client
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

#### Serveur
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
