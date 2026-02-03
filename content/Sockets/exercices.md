+++
title = 'Exercices'
date = 2025-01-26T15:07:06-05:00
draft = false
weight = 21
+++


### UDP

1. Créez un **serveur** UDP sur votre PI qui s'attend à recevoir l'une des trois commandes suivante: `allume`, `ferme`, `exit`.

- `allume` : Allume une led RGB avec une couleur aléatoire à chaque fois (PWM). Si la lumière est déjà allumé, ne fait rien.
- `ferme` : Éteint la led
- `exit` : S'assure de fermer la led et de bien fermer le serveur.

Créez ensuite le **client** UDP sur votre ordinateur (pas le Pi). Le client doit pouvoir envoyer ces trois messages. 


{{% expand "Solution" %}}
client.py
```python
import socket

PORT = 8888
DEST_IP = "127.0.0.1"

# Créer le socket
sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

# Créer un objet pour l'adresse
dest_addr = (DEST_IP, PORT)

try:
    while True:
        message = input(f"Envoyer un message à {DEST_IP} : ")
        # Envoyer le message
        sock.sendto(message.encode(), dest_addr)
        if message == "exit":
            break
    # Fermer le socket
except KeyboardInterrupt:
    pass
finally:
    sock.close()
```

serveur.py
```python
import pigpio
import time
import socket
import random

PORT = 8888
BUFFER_SIZE = 1024

pi = pigpio.pi()

pinB = 22
pinR = 26
pinG = 17

BTN = 4

pi.set_mode(pinB,pigpio.OUTPUT)
pi.set_mode(pinR,pigpio.OUTPUT)
pi.set_mode(pinG,pigpio.OUTPUT)

pi.set_mode(BTN, pigpio.INPUT)


pi.write(pinB, 1)
pi.write(pinR, 1)
pi.write(pinG, 1)

# Créer le socket
socket_local = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
adr_local = ('', PORT)

# Lier le socket à l'adresse
socket_local.bind(adr_local)

print(f"On attend les messages UDP entrants au port {PORT}...")

lastState = "ferme"

try:
    # Réception des messages
    while True:
        # Réception des données
        buffer, adr_dist = socket_local.recvfrom(BUFFER_SIZE)
        
        # Décodage binaire -> texte
        message = buffer.decode()

        print(f"Message reçu: {message}")

        print(f"LastState reçu: {lastState}")

        match message.strip():
            case "allume":
                if lastState == "ferme":
                    pi.set_PWM_dutycycle(pinR, random.randint(0, 255))
                    pi.set_PWM_dutycycle(pinB, random.randint(0, 255))
                    pi.set_PWM_dutycycle(pinG, random.randint(0, 255))
                    lastState = "allume"
            case "ferme":
                pi.write(pinB,1)
                pi.write(pinR,1)
                pi.write(pinG,1)
                lastState = "ferme"
            case "exit":
                break
except KeyboardInterrupt:
    print("Programme arrêté")
finally:
    pi.write(pinB,1)
    pi.write(pinR,1)
    pi.write(pinG,1)

    pi.stop()

    socket_local.close()

    print("Bye Bye")
```
{{% /expand %}}

2. En équipe de deux, reproduiser l'exercices : `Exercice 2 : Machine à états et gestion d’un bouton-poussoir` mais avec le bouton sur un pi et la lumière sur le 2e pi.

{{% expand "Solution" %}}
client.py
```python
import socket
import pigpio
import time

pi = pigpio.pi()



BTN = 4

etat = -1
prev = -1

PORT = 8888
DEST_IP = "127.0.0.1"


# Créer le socket
sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

# Créer un objet pour l'adresse
dest_addr = (DEST_IP, PORT)
NIVEAU_ROBUSTESSE = 5
try:
  
    while True:
        while True:
            etat = pi.read(BTN)
            # print(etat)
            if etat != prev:
                somme = 0
                isNew = True
                for i in range(NIVEAU_ROBUSTESSE):
                    if pi.read(BTN) != etat:
                        isNew = False
                        break
                    time.sleep(0.01)
                if isNew: 
                    prev = etat
                    if etat == 0:
                        print("click")
                        sock.sendto("click".encode(), dest_addr)       
            time.sleep(0.02)
    
except KeyboardInterrupt:
    print("Au revoir!")

finally:
    # Envoi avis de fermeture au serveur
    sock.sendto("exit".encode(), dest_addr)
    # Cleanup
    sock.close()
    pi.stop()
```

serveur.py
```python
import pigpio
import time
import socket
import random

PORT = 8888
BUFFER_SIZE = 1024

#pigpio - setup
pi = pigpio.pi()

pinB = 22
pinR = 26
pinG = 17

BTN = 4

pi.set_mode(pinB,pigpio.OUTPUT)
pi.set_mode(pinR,pigpio.OUTPUT)
pi.set_mode(pinG,pigpio.OUTPUT)

pi.set_mode(BTN, pigpio.INPUT)


pi.write(pinB, 1)
pi.write(pinR, 1)
pi.write(pinG, 1)



class EtatLed:
    def __init__(self, next, pin, pi):
        self.pi = pi
        self.next = next
        self.pin = pin
    def allume(self):
        self.pi.write(self.pin, 0)
    def eteint(self):
        self.pi.write(self.pin, 1)

class EtatEteint(EtatLed):
    def allume(self):
        pass
    def eteint(self):
        pass

eteint = EtatEteint(None, 0, pi)
ledG   = EtatLed(eteint, pinG, pi)
ledB   = EtatLed(ledG, pinB, pi)
ledR   = EtatLed(ledB, pinR, pi)
eteint.next = ledR


# Créer le socket
socket_local = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
adr_local = ('', PORT)

# Lier le socket à l'adresse
socket_local.bind(adr_local)

print(f"On attend les messages UDP entrants au port {PORT}...")

led = eteint
try:
    # Réception des messages
    while True:
        # Réception des données
        buffer, adr_dist = socket_local.recvfrom(BUFFER_SIZE)
        
        # Décodage binaire -> texte
        message = buffer.decode().strip()

        print(f"Message reçu: {message}")

        if message == "click":
            led.eteint()
            led = led.next
            led.allume()
        elif message == "exit":
            break
        else:
            print("Erreur")
except KeyboardInterrupt:
    pass
finally:
    pi.write(pinR, 1)
    pi.write(pinB, 1)
    pi.write(pinG, 1)

    socket_local.close()
    pi.stop()
    print("\nBye bye")
```
{{% /expand %}}

### TCP

1. Faites deux programmes (`serveur1.py` et `client1.py`) ayant les mêmes fonctionnalités que celui de l'exercice 1 - UDP, mais en utilisant une connexion TCP. Au message _exit_, la connexion TCP est fermée des deux côtés.
{{% expand "Solution" %}}
client1.py
```python
import socket

PORT = 8888
DEST_IP = ""
MESSAGE = "abcdefg!\n"

# Create the socket
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Initialize the destination address
dest_addr = (DEST_IP, PORT)

# Create the connection
sock.connect(dest_addr)

try:
    while True:
        message = input(f"Envoyer un message à {DEST_IP} : ") + "\n"

        
        
        # Envoyer le message
        sock.send(message.encode())
        if message.strip() == "exit":
            break
except KeyboardInterrupt:
    pass
finally:
    print("Au revoir!")
    # Fermer le socket
    sock.close()
```

serveur1.py
```python
import pigpio
import time
import socket
import random

PORT = 8888
BUFFER_SIZE = 1024

pi = pigpio.pi()

pinB = 22
pinR = 26
pinG = 17

BTN = 4

pi.set_mode(pinB,pigpio.OUTPUT)
pi.set_mode(pinR,pigpio.OUTPUT)
pi.set_mode(pinG,pigpio.OUTPUT)

pi.set_mode(BTN, pigpio.INPUT)


pi.write(pinB, 1)
pi.write(pinR, 1)
pi.write(pinG, 1)

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

lastState = "ferme"

try:
    # Réception des messages
    while True:
        data = socket_dist.recv(BUFFER_SIZE)
        if not data:
            print("Déconnecté")
            break

        match data.decode().strip():
            case "allume":
                if lastState == "ferme":
                    pi.set_PWM_dutycycle(pinR, random.randint(0, 255))
                    pi.set_PWM_dutycycle(pinB, random.randint(0, 255))
                    pi.set_PWM_dutycycle(pinG, random.randint(0, 255))
                    lastState = "allume"
            case "ferme":
                pi.write(pinB,1)
                pi.write(pinR,1)
                pi.write(pinG,1)
                lastState = "ferme"
            case "exit":
                break
except KeyboardInterrupt:
    pass
finally:
    pi.write(pinB,1)
    pi.write(pinR,1)
    pi.write(pinG,1)

    pi.stop()

    socket_dist.close()
    socket_local.close()
    print("Bye Bye")


```


{{% /expand %}}

2. Ajoutez le fonctionnalité suivante: le serveur répond "OK" au client si la commande est _allume_, _ferme_ ou _exit_ ou "ERR" autrement (`serveur2.py` et `client2.py`).
{{% expand "Solution" %}}
client2.py
```python
# Coming soon
```


serveur2.py
```python
# Coming soon
```


{{% /expand %}}
3. Modifiez votre programme: lorsque le client envoit _exit_, la connexion TCP est terminée, le client se termine, mais le serveur continue à attendre d'autres connexions TCP (`serveur3.py` et `client3.py`).
{{% expand "Solution" %}}
client3.py
```python
# Coming soon
```

<!-- 
import socket

PORT = 9090
DEST_IP = "192.168.50.190"
BUFFER_SIZE = 1024

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
dest_addr = (DEST_IP, PORT)
sock.connect(dest_addr)

while True:
    message = input("> ")
    sock.send(message.encode() + b'\n')
    
    # Réception de la réponse
    data = sock.recv(BUFFER_SIZE)
    if data:
        reponse = data.decode().strip()
        print("<",reponse)
        
    if message == "exit":
        break

sock.close() -->
serveur3.py

```python
# Coming soon
```
<!-- 
```python
import socket
import pigpio

LED_PIN = 17
PORT = 9090
BUFFER_SIZE = 1024

pi = pigpio.pi()

socket_local = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
address = ('', PORT)  
socket_local.bind(address)

# Ajout d'une boucle pour écouter les demandes de connexions entrantes
while True:
    socket_local.listen(3)
    print(f"Écoute au port: {PORT}...")

    socket_desc, client_address = socket_local.accept()
    print(f"Socket distant: {client_address}")

    while True:
        data = socket_desc.recv(BUFFER_SIZE)
        if data:
            message = data.decode().strip()
            print(f"< {message}")
            
            reponse = "OK"
            if message == "allume":
                pi.write(LED_PIN, 1)
            elif message == "ferme":
                pi.write(LED_PIN, 0)
            elif message == "exit":
                pi.write(LED_PIN, 0)
                # Sortir de le boucle WHILE interne 
                break
            else:
                reponse = "ERR"
            
            # Envoyer la réponse
            socket_desc.send(reponse.encode() + b'\n')

    # Fermer la connexion
    socket_desc.close()
``` -->
{{% /expand %}}
4. Ajoutez une 2e LED sur votre Pi. Modifiez le programme serveur pour que les commandes envoyées permettent de spécifier laquelle des 2 LED allumer (les messages "led1" et "led2" doivent être envoyés au serveur pour qu'il sache quelle LED allumer ou éteindre). Le programme client n'a pas besoin d'être modifié. Les messages possibles du client sont donc: `led1, led2, allume, ferme, exit`. Les messages du serveur sont `OK, ERR, BYE`. La led1 s'allume vert et la led2 s'allume bleu.
{{% expand "Solution" %}}
serveur4.py

```python
# Coming soon
```
<!-- 
```python
import socket
import pigpio

LED1 = 17
LED2 = 27
PORT = 9090
BUFFER_SIZE = 1024

pi = pigpio.pi()

socket_local = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
address = ('', PORT)  
socket_local.bind(address)

# Ajout d'une boucle pour écouter les demandes de connexions entrantes
while True:
    socket_local.listen(3)
    print(f"Écoute au port: {PORT}...")

    socket_desc, client_address = socket_local.accept()
    print(f"Socket distant: {client_address}")

    # Donner une valeur par défaut à la LED active pour éviter les erreurs
    led_active = LED1

    while True:
        data = socket_desc.recv(BUFFER_SIZE)
        if data:
            message = data.decode().strip()
            print(f"< {message}")
            
            reponse = "OK"
            if message == "led1":
                led_active = LED1
            elif message == "led2":
                led_active = LED2
            elif message == "allume":
                pi.write(led_active, 1)
            elif message == "ferme":
                pi.write(led_active, 0)
            elif message == "exit":
                # Tout éteindre et sortir de le boucle WHILE interne
                pi.write(LED1, 0)
                pi.write(LED2, 0) 
                reponse = "BYE"
                socket_desc.send(reponse.encode() + b'\n')
                break
            else:
                reponse = "ERR"
            
            # Envoyer la réponse
            socket_desc.send(reponse.encode() + b'\n')

    # Fermer la connexion
    socket_desc.close()
``` -->
{{% /expand %}}

