+++
title = 'Formatif'
date = 2026-02-12T19:12:35-05:00
weight = 22
+++

---

# Exercice : Contrôle de Moteur Stepper via TCP

### Objectif

Développer une architecture client-serveur. Vous devrez coder la logique de mouvement sur le serveur (Raspberry Pi) et une interface de contrôle interactive sur le client (Container).

---

## 1. Architecture du réseau

* **Le Serveur (Raspberry Pi) :** Il reçoit les ordres et pilote physiquement le moteur.
* **Le Client (Container) :** C'est votre interface de commande. C'est ici que l'utilisateur saisit les angles et reçoit les notifications d'état.

---

## 2. Logique métier et Feedback

Le moteur réagit selon la valeur de l'angle (nombre entier) reçu :

* **Angle Impair :** Rotation dans le **sens horaire**.
* **Angle Pair :** Rotation dans le **sens anti-horaire**.

### Interaction Client-Serveur (Le protocole)

Pendant toute la durée du mouvement, le client ne doit pas pouvoir envoyer de nouvel ordre. L'échange doit ressembler à ceci :

1. **Client :** Envoie l'angle (ex: `90`).
2. **Serveur :** Reçoit l'angle, démarre le moteur et répond immédiatement : `"Le moteur tourne, veuillez patienter"`.
3. **Client :** Affiche ce message à l'utilisateur et "bloque" la saisie.
4. **Serveur :** Une fois le mouvement fini, envoie un signal de fin (ex: `"OK"` ou `"Terminé"`).
5. **Client :** Affiche `"Prêt pour une nouvelle commande"` et redonne la main à l'utilisateur.

---

## 3. Simulation du rendu dans le Terminal (Côté Client)

Voici ce que vos étudiants devraient obtenir dans leur terminal lorsqu'ils exécutent leur programme client :

```bash
etudiant@container:~$ python3 client_moteur.py
--- Contrôle du Stepper ---

Entrez l'angle souhaité : 45

[SERVEUR] : Le moteur tourne, veuillez patienter...
[SERVEUR] : Mouvement terminé avec succès.

Entrez l'angle souhaité : 

```

---

# Codes de base qui seraient disponible dans cet examen 

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
```python
import pigpio
import time

# GPIO
M1,M2,M3,M4 = 26,13,19,6

# Minimum recommandé: 2ms (0.002)
step_pause = 0.002


# Séquence "Full-step" = plus de couple 
seq_full = [
    [1,0,0,1],
    [1,1,0,0],
    [0,1,1,0],
    [0,0,1,1]
]


def stop_moteur():
    pi.write(M1, 0)
    pi.write(M2, 0)
    pi.write(M3, 0)
    pi.write(M4, 0)

pi = pigpio.pi()

pi.set_mode(M1,pigpio.OUTPUT)
pi.set_mode(M2,pigpio.OUTPUT)
pi.set_mode(M3,pigpio.OUTPUT)
pi.set_mode(M4,pigpio.OUTPUT)

try:
    while True:
        for step in seq_full: # Changez la séquence ici au besoin
            # Activer chacune des 4 bobines
            pi.write(M1, step[0])
            pi.write(M2, step[1])
            pi.write(M3, step[2])
            pi.write(M4, step[3])

            # La durée de la pause détermine la vitesse
            time.sleep(step_pause)

except KeyboardInterrupt:
    stop_moteur()
```
```bash
nc 1.2.3.4 8888
nc -lp 8888
``` 

{{% expand "Solution" %}}
client.py
```python
import socket
import os
import termios
import sys


def flush_input():
    """
    Fonction qui permet de libérer le buffer de input. Pas nécessaire selon mon énoncé, mais plus propre comme code.
    """
    try:
        if os.name == 'nt':  # Windows
            import msvcrt
            while msvcrt.kbhit():
                msvcrt.getch()
        else:  # Linux / macOS
            import termios
            termios.tcflush(sys.stdin, termios.TCIFLUSH)
    except ImportError:
        # Cas rare où les modules ne sont pas dispo
        pass
    
PORT = 8887
DEST_IP = "127.0.0.1"
BUFFER_SIZE = 1024
MESSAGE_TOURNE = "Le moteur tourne, veuillez patienter..."
MESSAGE_CONFIRMATION = "Mouvement terminé avec succès."
MESSAGE_ERREUR = "Une erreur est survenue, veuillez réessayer"
MESSAGE_CLOSE = "Déconnecté"

# Create the socket
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Initialize the destination address
dest_addr = (DEST_IP, PORT)



try:
    # Create the connection
    sock.connect(dest_addr)
    print("--- Contrôle du Stepper ---")
    connected = True
    while connected:
        flush_input()
        message = input("\nEntrez l'angle souhaité : ")
        print("")
        # Send the message and close the connection
        sock.send(message.encode())

        msgServeur  = sock.recv(BUFFER_SIZE).decode()
        print(f"Reçu : {msgServeur}")
        if not msgServeur or msgServeur == MESSAGE_CLOSE:
            print("Déconnecté")
            break
        if msgServeur == MESSAGE_ERREUR:
            print(f"[SERVEUR] : {MESSAGE_ERREUR}")
            continue

        if msgServeur == MESSAGE_TOURNE:
            print(f"[SERVEUR] : {MESSAGE_TOURNE}")

            while True:
                confirmation = sock.recv(BUFFER_SIZE).decode()
                if not confirmation or confirmation == MESSAGE_CLOSE:
                    print("Déconnecté")
                    connected = False
                    break
                if confirmation == MESSAGE_ERREUR:
                    print(f"[SERVEUR] : {MESSAGE_ERREUR}")
                    break
                if confirmation == MESSAGE_CONFIRMATION:
                    print(f"[SERVEUR] : {MESSAGE_CONFIRMATION}")
                    break
            if not connected:
                break
            
except KeyboardInterrupt:
    pass
finally:
    sock.close()
```

serveur.py
```python
import socket
import pigpio
import time

"""
1. Test le matériel
2. Test la connexion
3. Ajoute le moteur dans le fichier serveur juste pour tourner un tour
4. Everything!
5. Assurer que le code ne brise pas 
"""

# Constantes pour le serveur
PORT = 8887
BUFFER_SIZE = 1024
MESSAGE_TOURNE = "Le moteur tourne, veuillez patienter..."
MESSAGE_CONFIRMATION = "Mouvement terminé avec succès."
MESSAGE_ERREUR = "Une erreur est survenue, veuillez réessayer"
MESSAGE_CLOSE = "Déconnecté"
# Constantes pour le moteur
M1,M2,M3,M4 = 26,13,19,6
STEP_PAR_TOUR = 2048

#######    Set-up Moteur    #######
# Vitesse moteur : Minimum recommandé: 2ms (0.002)
step_pause = 0.002

# Séquence "Full-step" = plus de couple 
seq_anti_horaire= [
    [1,0,0,1],
    [1,1,0,0],
    [0,1,1,0],
    [0,0,1,1]
]

seq_horaire = seq_anti_horaire.copy()
seq_horaire.reverse()

def stop_moteur():
    pi.write(M1, 0)
    pi.write(M2, 0)
    pi.write(M3, 0)
    pi.write(M4, 0)

pi = pigpio.pi()

pi.set_mode(M1,pigpio.OUTPUT)
pi.set_mode(M2,pigpio.OUTPUT)
pi.set_mode(M3,pigpio.OUTPUT)
pi.set_mode(M4,pigpio.OUTPUT)

######    Set-up Socket    #######
socket_local = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
address = ('', PORT)  
socket_local.bind(address)
socket_local.listen(3)
print(f"Serveur TCP en écoute au port {PORT}...")


#####     Fct      #######

def controle_moteur(angle, seq):
    compteur_pas = 0
    pas_a_faire = angle * STEP_PAR_TOUR / 360
    while compteur_pas < pas_a_faire:
            for step in seq: # Changez la séquence ici au besoin
                # Activer chacune des 4 bobines
                pi.write(M1, step[0])
                pi.write(M2, step[1])
                pi.write(M3, step[2])
                pi.write(M4, step[3])

                # La durée de la pause détermine la vitesse
                time.sleep(step_pause)
                compteur_pas += 1


# Attente de connexion
socket_dist, client_address = socket_local.accept()
print(f"Socket distant: {client_address}")



try:
    # Réception des messages
    while True:
        # Données
        data = socket_dist.recv(BUFFER_SIZE)
        if not data:
            print("Déconnecté")
            break
        
        # Décoder + afficher les message
        message = data.decode().strip()
        # Filtre le message
        try:
            angle = int(message)
            if angle < 0: continue
        except:
            socket_dist.send(MESSAGE_ERREUR.encode())
            continue   

        socket_dist.send(MESSAGE_TOURNE.encode())
        
        if angle % 2 == 0: 
            seq = seq_anti_horaire
        else:
            seq = seq_horaire
        controle_moteur(angle, seq)
        socket_dist.send(MESSAGE_CONFIRMATION.encode())

except KeyboardInterrupt:
    socket_dist.send(MESSAGE_CLOSE.encode())
    time.sleep(1)
finally:
    socket_dist.close()
    socket_local.close()
```
{{% /expand %}}