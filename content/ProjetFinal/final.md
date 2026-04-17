+++
title = 'Projet-Final'
date = 2024-03-04T08:04:13-05:00
draft = false
weight = 84
+++

---

## Rappel du plan de cours

La personne étudiante doit concevoir en équipe de deux un objet connecté répondant aux besoins énoncés dans un cahier de charge fournit par la personne enseignante. L'épreuve se réalise en partie hors des heures de cours.

Étant donné qu'une partie de l'évaluation se déroule hors classe, une présentation orale individuelle est requise pour valider l'acquisition des compétences. La note obtenue lors de cet entretien servira à pondérer celle du travail d'équipe, confirmant ainsi la maîtrise personnelle du projet soumis.

### Les critères de correction sont les suivants:
- Respect rigoureux des consignes
- Le produit final doit réunir les caractéristiques suivantes :
    - L’objet connecté est en communication avec un serveur roulant sur une autre machine
    - Une action sur l’application de contrôle du serveur doit pouvoir influencer le comportement de l’objet connecté
    - L’objet connecté doit communiquer des informations venant de ses senseurs au serveur à intervalles réguliers
- Choix approprié des instructions, des algorithmes, des types de données élémentaires et des structures de données
- Cohésion de chaque composant du produit
- Couplage approprié des composants du produit
- Organisation logique des instructions et lisibilité du code
- Repérage complet des erreurs, fonctionnement correct du programme et des 
composantes électroniques

---

### **Spécifications Techniques Additionnelles**

* **Interface Utilisateur (Front-end) :** Le serveur doit impérativement disposer d'une **interface web accessible par navigateur** permettant la visualisation des données et le contrôle de l'objet.
* **Infrastructure Réseau :** L'ensemble du système doit opérer sur un **réseau Wi-Fi local privé**, dont le point d'accès est directement hébergé et géré par le serveur.
* **Autonomie au démarrage :** Le microcontrôleur (Pi) doit être configuré pour une **exécution automatique (headless)**. Dès la mise sous tension, il doit établir sa connexion réseau et lancer le script principal sans intervention humaine.
* **Indicateur d'état (Feedback) :** Le système doit intégrer un **signal visuel ou sonore** (ex: DEL de statut) informant l'utilisateur que l'initialisation est terminée et que l'appareil est opérationnel.

* **Robustesse et Tolérance aux pannes** : Le système doit être conçu pour prévenir les plantages critiques. Cela inclut une gestion rigoureuse des exceptions (try/except), la reconnexion automatique en cas de perte de signal Wi-Fi, et la stabilité du code face à des données imprévues ou erronées.
---

### **Pièces disponibles à l'école**

[Site d'inventaire](https://airtable.com/app1DhJbke7rmcxYz/shrb71ct8A6K8DiNG/tblfFzkF6xNzl8Wbn)

## Livrables

1. Vendredi 24 avril 23h59 : Description complète du projet (5%)
2. Vendredi 1 mai 12h  : Démo 1 et amendement au descriptif du projet(5%)  
3. Vendredi 8 mai 12h  : Démo 2 et amendement finaux au projet (10%)

*Si dans l'un des 3 premiers livrables, vous perdez des points, il sera possible d'en récupérer la moitié si les correctifs nécessaires sont appliqués avant le prochain livrable.*

---
4. Vendredi 15 mai 12  : Démo Finale de groupe(20%)
5. Entre le 20 mai et le 27 mai : Évaluation oral individuelle.(Pondérée)

---


# Description Complète du Projet : [Insérer Nom du Projet]

**Équipe :** [Nom Membre 1] & [Nom Membre 2]  
**Date :** 24 avril 2026  
**Cours :** Objet Connecté – Épreuve Finale  

---

## 1. Résumé de la solution (Le "Quoi")
*Fournissez une description claire du besoin et de la solution.*

---

## 2. Architecture du système (Le "Comment")

### 2.1 Infrastructure Réseau
* **Wi-Fi Local :** Le serveur (Raspberry Pi) agira comme point d'accès Wi-Fi privé (SSID : `[Nom du Projet]_Net`). L'objet connecté s'y connectera automatiquement au démarrage.
* **Protocole :** La communication s'effectuera via [HTTP REST / MQTT / WebSockets] pour garantir un échange de données structuré.

### 2.2 Composantes logicielles et matérielles
* **L’Objet Connecté (Client) :** * Microcontrôleur : [ex: ESP32 ou Raspberry Pi Pico W].
    * **Signal de prêt :** Une [ex: LED Bleue] s'allumera de façon fixe dès que le code est lancé et que la connexion au Wi-Fi local est établie.
    * **Autonomie :** Le script est configuré pour se lancer via [ex: service systemd / crontab] dès la mise sous tension.
* **Le Serveur :**
    * Machine : [ex: Raspberry Pi 4].
    * Back-end : [ex: Node.js / Python Flask].
    * Front-end : Page web accessible localement (HTML/CSS/JS).

---

## 3. Spécifications fonctionnelles

* **Flux de données (Senseur → Serveur) :** L'objet lira les valeurs de [Nom du capteur, ex: Capteur d'humidité du sol] toutes les [X secondes/minutes]. Ces données seront envoyées au serveur sous format JSON.
* **Contrôle (Serveur → Objet) :** Depuis l'interface web, l'utilisateur pourra activer/désactiver [Nom de l'actionneur, ex: une pompe à eau / un ventilateur].
* **Interaction en temps réel :** Une modification sur la page web entraînera une réaction de l'objet en moins de [X] secondes.

---

## 4. Conception technique

### 4.1 Structures de données
Les messages échangés entre l'objet et le serveur suivront ce format JSON type :
```json
{
  "device_id": "sensor_01",
  "timestamp": "2026-04-24T12:00:00Z",
  "status": "ready",
  "data": {
    "valeur_1": 22.5,
    "unite": "Celsius"
  }
}
```

### 4.2 Logique et Gestion des erreurs
* **Algorithme de démarrage :** Connexion Wi-Fi -> Validation serveur -> Allumage de la LED de prêt -> Boucle de lecture capteur.

* **Robustesse :** En cas de perte de signal Wi-Fi, l'objet [ex: tentera une reconnexion toutes les 10 secondes et fera clignoter la LED de statut pour avertir l'utilisateur].

---

## 5. Liste des composants (Bill of Materials)

| Composant | Quantité | Rôle |
| :--- | :---: | :--- |
| [ex: ESP32] | 1 | Cerveau de l'objet connecté |
| [ex: Raspberry Pi 4] | 1 | Hébergement du serveur et du point d'accès |
| [ex: Capteur DHT22] | 1 | Mesure de température et humidité |
| [ex: Relais 5V] | 1 | Commande de l'actionneur haute puissance |
| [ex: LED Rouge/Verte] | 2 | Indicateurs visuels d'état |

---

### Schéma d'architecture
*(N'oublie pas ton Draw.io/paint montrant les flux entre l'objet, le réseau Wi-Fi local et le serveur Web !)*

---

## 6. Calendrier des livrables

Afin de respecter les échéances et de maximiser la récupération de points potentiels, l'équipe devra choisir son propre calendrier de développement :

Exemple de calendrier:

### 6.1 Livrable 1 : Fondations et Architecture (24 avril)
* **Objectif :** Validation du concept et de la faisabilité technique.
* **Contenu :** * Remise de la description complète (ce document).
    * Sélection finale et réservation de l'équipement à l'école.

### 6.2 Livrable 2 : Démo 1 - Connectivité et Flux Montant (1er mai)
* **Objectif :** Établir le lien de communication de base.
* **Éléments présentés :**
    * **Infrastructure :** Le serveur diffuse son propre point d'accès Wi-Fi et le pi s'y connecte automatiquement.
    * **Flux "Senseur" :** Lecture d'une valeur réelle (ou simulée sur plaque d'essai) envoyée avec succès au serveur.
    * **Interface :** Affichage brut de la donnée reçue sur une page web simple.
    * **Gestion d'erreurs :** Reconnexion automatique en cas de coupure de l'alimentation ou du Wi-Fi.
    * **Flux "Contrôle" :** Un bouton sur l'interface web active physiquement un actionneur sur l'objet connecté (ex: LED ou Relais).

### 6.3 Livrable 3 : Démo 2 - Contrôle et Robustesse (8 mai)
* **Objectif :** Boucler l'interaction et stabiliser le système.
* **Éléments présentés :**
    * **Indicateur d'état :** Validation du signal visuel de "Prêt" (Feedback).
    * **Logique :** Le problème principal est résolu

### 6.4 Livrable 4 : Démo Finale - Produit Intégré (15 mai)
* **Objectif :** Présentation du produit fini, esthétique et fonctionnel.
* **Éléments présentés :**
    * **Exécution Headless :** Le système démarre sans écran/clavier externe dès le branchement.
    * **Code propre :** Remise du code source final (organisé, commenté et sans bogues apparents).
    * **Expérience Utilisateur (UX) :** Interface web soignée et réactive.
    * **Intégration physique :** Montage final propre (boîtier ou support si applicable).

