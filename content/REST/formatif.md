+++
title = 'Formatif'
date = 2024-03-04T08:04:13-05:00
draft = false
weight = 84
+++

---

Vous avez droit à w3school pour **javascript** uniquement!

# 📝 Examen : Le Luminaire Connecté "Smart-AP"

**Contexte :** Vous devez configurer un Raspberry Pi pour qu'il devienne un contrôleur de lumière d'ambiance. Le Pi crée son propre Wi-Fi (Mode AP). Les utilisateurs se connectent et pilotent une LED RGB via une interface web.

### 1. Prérequis Réseau (3 pts)
* **Mode AP :** Configurez le Pi pour diffuser un SSID nommé `Exam-IoT-VOTRENOM`.
* **Réseau :** IP statique `192.168.4.1`.
* **Serveur :** Flask doit être accessible sur le port `5000` depuis n'importe quel appareil connecté au Wi-Fi.

### 2. Montage Électronique (3 pts)
* **Bouton :** Branché sur le **GPIO 17** .
* **LED RGB :** * Rouge $\rightarrow$ **GPIO 12** | Vert $\rightarrow$ **GPIO 13** | Bleu $\rightarrow$ **GPIO 18**

---

### 3. Frontend : Code à compléter 
Insérez le code suivant dans la balise `<script>` de votre `index.html`. Vous devez compléter la fonction d'envoi.

```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Contrôle Smart-AP RGB</title>
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; text-align: center; background: #1a1a1a; color: white; padding: 20px; }
        .card { background: #333; padding: 30px; border-radius: 15px; box-shadow: 0 10px 30px rgba(0,0,0,0.5); display: inline-block; min-width: 300px; }
        
        h1 { color: #00d4ff; margin-bottom: 25px; }
        
        /* Indicateurs visuels */
        .status-container { display: flex; justify-content: space-around; margin-bottom: 20px; }
        .indicator { width: 80px; height: 80px; border-radius: 50%; margin: 10px auto; border: 3px solid #fff; transition: 0.3s; }
        .label { font-size: 0.9rem; color: #bbb; margin-top: 5px; }

        /* Sliders */
        .control-group { margin: 20px 0; text-align: left; }
        label { display: block; margin-bottom: 5px; font-weight: bold; }
        input[type=range] { width: 100%; cursor: pointer; height: 8px; border-radius: 5px; background: #555; outline: none; }
        
        #r { accent-color: #ff4b4b; }
        #g { accent-color: #4bff4b; }
        #b { accent-color: #4b4bff; }

        button { background: #00d4ff; border: none; padding: 12px 25px; border-radius: 5px; color: #1a1a1a; font-weight: bold; font-size: 1rem; cursor: pointer; transition: 0.2s; margin-top: 10px; width: 100%; }
        button:hover { background: #0099cc; transform: translateY(-2px); }
        button:active { transform: translateY(0); }
    </style>
</head>
<body>

    <div class="card">
        <h1>Smart-AP Light</h1>
        
        <div class="status-container">
            <div>
                <div id="preview" class="indicator" style="background-color: rgb(0,0,0);"></div>
                <div class="label">Aperçu</div>
            </div>
            <div>
                <div id="etat_led" class="indicator" style="background-color: gray;"></div>
                <div class="label">État LED</div>
            </div>
        </div>

        <hr style="border: 0.5px solid #444;">

        <div class="control-group">
            <label>Rouge : <span id="valR">0</span></label>
            <input type="range" id="r" min="0" max="255" value="0" oninput="updateUI()">
            
            <label>Vert : <span id="valG">0</span></label>
            <input type="range" id="g" min="0" max="255" value="0" oninput="updateUI()">
            
            <label>Bleu : <span id="valB">0</span></label>
            <input type="range" id="b" min="0" max="255" value="0" oninput="updateUI()">
        </div>

        <button onclick="envoyerCouleur()">APPLIQUER LA COULEUR</button>
    </div>

    <script>
        const preview = document.getElementById('preview');
        const etatLedDiv = document.getElementById('etat_led');

        /**
         * Met à jour les chiffres et le cercle 'preview' en temps réel
         * (Exigence 4 - Feedback Visuel 1)
         */
        function updateUI() {
            const r = document.getElementById('r').value;
            const g = document.getElementById('g').value;
            const b = document.getElementById('b').value;
            
            document.getElementById('valR').innerText = r;
            document.getElementById('valG').innerText = g;
            document.getElementById('valB').innerText = b;

            preview.style.backgroundColor = `rgb(${r}, ${g}, ${b})`;    
        }

        /**
         * Envoie les données au serveur Flask via une requête POST
         * (Exigence 4 - Requête Fetch & UX)
         */
        function envoyerCouleur() {
            // 1. Récupérer les valeurs des 3 sliders (r, g, b)
            
            // 2. Créer un objet JSON contenant ces valeurs
            
            // 3. Envoyer cet objet via une requête POST à /api/set_rgb
            // Utiliser fetch() avec la méthode POST et les bons headers
        }

        /**
         * Vérifie l'état réel de la LED sur le Raspberry Pi
         * (Exigence 4 - Feedback Visuel 2)
         */
        setInterval(() => {
            fetch('/api/led_state')
                .then(res => res.json())
                .then(data => {
                    // Logique : Si data.power est vrai, mettre 'etat_led' en jaune (ou couleur vive)
                    // Sinon, mettre 'etat_led' en gris.
                })
                .catch(err => console.error("Erreur de synchronisation"));
        }, 1000);
    </script>
</body>
</html>
```
---

### 4. Travail Backend 

#### A. Serveur Flask & PWM 
Développez `app.py`. Le serveur doit :
1. Initialiser `pigpio` et configurer les fréquences PWM à **800Hz**.
2. **Route `POST /api/set_rgb` :** * Doit extraire les valeurs `r`, `g`, `b` du JSON reçu.
    * Appliquer ces valeurs aux GPIO via `set_PWM_dutycycle`.
    * Gérer une erreur (400) si les valeurs ne sont pas entre 0 et 255.
3. **Route `GET /api/led_state` :** Renvoie un JSON indiquant si la LED émet de la lumière ou non (ex: `{"power": true}`).

#### B. Logique du Bouton "Haute Performance"
Le bouton physique doit fonctionner comme un interrupteur (Toggle) :
* **Action :** Un clic (appui + relâchement) inverse l'état de la LED (On/Off).
* **Mémoire :** Si on éteint la LED avec le bouton, elle doit "retenir" sa couleur précédente lorsqu'on la rallume.
* **Rapidité :** Utilisez une **interruption** (*callback*) plutôt qu'une boucle `while`. Le système doit être capable de traiter des clics rapides (jusqu'à 10 par seconde) sans rebonds (*debounce* logiciel de 50ms recommandé).


---

### Grille de Correction Détaillée (Total : 20 pts)

#### 1. Infrastructure Réseau (5 pts)
* **[2 pts] Visibilité :** Le SSID `Exam-IoT-VOTRENOM` apparaît sur les appareils à proximité.
* **[2 pts] Accessibilité :** Le serveur répond sur `192.168.4.1:5000`. (0.5 pt si c'est une autre IP mais que ça marche).
* **[1 pt] Persistance :** Le point d'accès est stable et ne crash pas lors de la connexion d'un client.

#### 2. Logique du Bouton Physique (4 pts)
* **[2 pts] Mode Toggle :** Un clic franc (appui/relâche) change l'état (On vers Off / Off vers On).
* **[1 pt] Rapidité (10Hz) :** Le bouton répond sans "manquer" d'appuis lors d'une rafale rapide (absence de rebonds/debounce).
* **[1 pt] Mémoire :** En rallumant avec le bouton, la LED reprend la dernière couleur définie par le Web (ne revient pas à blanc ou rouge par défaut).

#### 3. Backend & API Flask (6 pts)
* **[2 pts] Route POST :** La route `/api/set_rgb` réceptionne correctement le JSON et extrait les 3 valeurs.
* **[2 pts] Qualité du Signal :** Les 3 couleurs sont pilotées en PWM (mélange fluide, pas de "tout ou rien").
* **[1 pt] Route GET :** La route `/api/led_state` renvoie un JSON valide reflétant l'état réel.
* **[1 pt] Robustesse :** Le serveur ne plante pas si on envoie une valeur hors limite ou un JSON mal formé.

#### 4. Intégration HTML/JS (5 pts)
* **[2 pts] Requête Fetch :** Le JavaScript récupère correctement les valeurs des sliders et les envoie au bon format.
* **[1 pts] Feedback Visuel :** Le cercle `preview` sur la page web change de couleur en même temps que les slider.
* **[1 pts] Feedback Visuel 2 :** Le cercle `etat_led` sur la page web reflète l'état de la led rafraichi 1 fois par seconde.
* **[1 pt] UX :** L'interface est fluide, pas de latence excessive entre l'appui sur "Appliquer" et la réaction de la LED.

---

### Tableau de Correction "Live" 

| Critère | Sous-élément | Max | Note | Commentaires |
| :--- | :--- | :---: | :---: | :--- |
| **Réseau** | SSID conforme et IP statique (192.168.4.1) | 3 | | |
| | Flask écoute sur 0.0.0.0 (Visible Wi-Fi) | 2 | | |
| **Bouton** | Changement d'état On/Off (Toggle) | 2 | | |
| | Réponse rapide (Interruption/Debounce) | 2 | | |
| **Backend** | Parsing JSON et contrôle PWM (RGB) | 4 | | |
| | Route GET Status fonctionnelle | 2 | | |
| **Frontend** | Envoi Fetch POST (JSON stringify) | 3 | | |
| | Prévisualisation CSS synchronisée | 2 | | |
| **TOTAL** | | **20** | **/20** | |

---


