+++
title = 'Bluetooth-Synthèse'
date = 2026-02-26T18:29:45-05:00
weight = 63
+++

---

## Challenge IoT : "L'Alerte de l'Ombre" 🕵️‍♂️

**Durée :** 3h | **Équipes :** 2 personnes | **Objectif :** Transmettre l'état d'un capteur via un pont Bluetooth vers un serveur UDP centralisé.

### 📋 Architecture du système

Le projet se divise en trois maillons :

1. **Le Nœud Capteur (Pi A) :** Lit le capteur de lumière Keyestudio et envoie l'état au Pi B via **Bluetooth**. <a href="https://cegepmv.github.io/420-314/7-analogin/ads1115/index.html">Doc</a>
2. **La Passerelle (Pi B) :** Reçoit la donnée de luminosité par Bluetooth et la retransmet au professeur via un **paquet UDP**.
3. **Le Dashboard (Prof) :** Affiche en temps réel qui est "dans le noir" et qui a réussi la liaison.

---

### ⏱️ Déroulement du TP (3 Étapes)

#### Étape 1 : Hardware et Lecture (45 min)

* **Branchement :** Connecter le capteur de lumière Keyestudio sur le Pi A (GND, VCC, Signal sur une pin GPIO).
* **Script de lecture :** Créer un programme Python qui détecte un seuil de luminosité.
* *Condition :* Si `valeur < seuil` ➔ `status = "CACHÉ"`, sinon `status = "OK"`.


* **Test :** Afficher l'état dans la console du Pi A.

#### Étape 2 : Le Pont Bluetooth (1h00)

* **Appairage :** Rendre les deux Pi détectables et les lier (`bluetoothctl`).
* **Communication :** * Le **Pi A** (Client) doit envoyer le statut toutes les 2 secondes au Pi B.
* Le **Pi B** (Serveur) doit écouter par bluettoth et afficher ce qu'il reçoit.


* *Optionnel :* Si le Bluetooth déconnecte, le script doit tenter de se reconnecter.

#### Étape 3 : Le Sprint UDP vers le Prof (1h15)

* **Formatage :** Le Pi B doit construire un message contenant :
* Le nom de l'équipe.
* L'état du capteur ("CACHÉ" ou "OK").

| **Payload UDP** | Exemple : `"Alpha:CACHÉ"` |

* **Envoi UDP :** Le Pi B envoie ce message à l'adresse IP du professeur sur le port défini (ex: 8888). Je vais vous donner l'IP quand j'aurai installé mon Pi.
* **Validation :** Dès que le message s'affiche correctement à l'écran partagé du prof, l'équipe gagne ! Je ferrai un ranking de ceux qui réussissent le plus rapidement, mais le code doit fonctionner et être acceptable. Venez m'avertir dès que ça fonctionne!

---

