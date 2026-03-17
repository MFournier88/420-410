+++
title = 'Synthèse'
draft = false
weight = 54
+++
---

# TP : Contrôle d'Éclairage Intelligent et Supervision MQTT


Voici votre formatif pour l'examen 2. Pour vous mettre en condition réelle, utilisez uniquement les informations disponibles sur le container. Si vous constatez des lacunes dans la documentation, avertissez-moi afin que je puisse (potentiellement) les combler pour le jour de l'épreuve.

**Objectif :** Réaliser un système d'éclairage automatique (Lumière < Seuil) avec une fonction de forçage à distance via une passerelle Bluetooth/MQTT.

---

## 1. Architecture et Rôles

* **Pi A (Capteur de Luminosité) :** Lit un capteur de lumière (ou un potentiomètre simulant la luminosité) et envoie la valeur en continu au Pi B via **Bluetooth**.
* **Pi B (Contrôleur & Gateway) :** * Possède une **LED**.
* Reçoit la luminosité du Pi A.
* Gère 3 modes : `AUTO` (par défaut), `ON`, `OFF`.
* Connecté au **Broker MQTT** (Container) pour recevoir les ordres de mode.


* **Container (Le Superviseur) :** Envoie les commandes de mode et affiche l'état du système.

---

## 2. Le Noeud Capteur (Pi A)

Le Pi A doit simplement diffuser la donnée brute.

* **Script :** `sensor.py`
* **Mission :** Lire le capteur de luminosité et l'envoyer via `BluetoothServer` (ou Client) au Pi B toutes les secondes.
* **Format suggéré :** Envoyer la valeur directement au Pi B

---

## 3. Le Contrôleur Intelligent (Pi B)

C'est ici que réside toute l'intelligence. Le script doit gérer deux sources d'entrée : le Bluetooth (donnée capteur) et le MQTT (ordre du superviseur).

### Logique du mode Automatique

1. Si `mode == "AUTO"` :
* Si `luminosité < SEUIL` : Allumer la LED.
* Sinon : Éteindre la LED.


2. Si `mode == "ON"` : La LED reste allumée, peu importe le capteur.
3. Si `mode == "OFF"` : La LED reste éteinte.

### Communication MQTT

Le Pi B doit s'abonner au topic `maison/eclairage/commande` d'un broker que vous devez créez sur votre container. Les messages acceptés sont : `AUTO`, `ON`, `OFF`. Vous devez implementer la connexion avec authentification.

> **Astuce :** Utilisez une variable globale `current_mode` dans votre script Python qui est mise à jour dans la fonction `on_message` de Paho-MQTT.

---


![alt text](/420-410/images/formatif-grille.png)