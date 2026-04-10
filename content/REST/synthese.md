+++
title = 'Synthèse'
date = 2024-03-04T08:04:13-05:00
draft = false
weight = 83
+++
## Projet : Radar de Recul Intelligent

**Objectif :** Créer une interface web qui surveille une distance en temps réel et permet de configurer dynamiquement les seuils d'alerte.

### 1. Le Matériel (Côté Raspberry Pi)
* **Senseur :** Un capteur de distance ultrasonique.
* **Actuateur :** Une **LED RGB** 

### 2. Le fonctionnement attendu
1.  Le Raspberry Pi mesure la distance en continu.
2.  La page Web affiche la distance mesurée (en cm).
3.  L'utilisateur utilise un **Slider** sur la page Web pour définir la `Distance Critique`.
4.  La LED change de couleur selon la logique suivante :
    * **Vert :** Distance > Distance Critique.
    * **Jaune :** Distance entre la moitié de la Distance Critique et la Distance Critique.
    * **Rouge :** Distance < moitié de la Distance Critique.

---

### 3. html

```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Radar de Proximité IoT</title>
    <style>
        body { font-family: sans-serif; text-align: center; background: #f4f4f9; padding: 50px; }
        .card { background: white; padding: 30px; border-radius: 15px; box-shadow: 0 4px 10px rgba(0,0,0,0.1); display: inline-block; }
        h1 { color: #333; }
        .distance-display { font-size: 3rem; font-weight: bold; color: #2c3e50; margin: 20px 0; }
        .slider-container { margin: 30px 0; }
        input[type=range] { width: 80%; cursor: pointer; }
        .status-dot { height: 25px; width: 25px; border-radius: 50%; display: inline-block; background: #bbb; }
        .label { font-weight: bold; display: block; margin-bottom: 10px; }
    </style>
</head>
<body>

    <div class="card">
        <h1>Contrôleur de Radar Pi</h1>
        
        <div class="label">Distance mesurée par le HC-SR04 :</div>
        <div class="distance-display"><span id="dist-val">--</span> cm</div>

        <hr>

        <div class="slider-container">
            <label class="label">Seuil d'alerte (Distance Critique) : <span id="seuil-val">50</span> cm</label>
            <input type="range" id="seuil-slider" min="10" max="150" value="50">
        </div>

        <div class="label">État de la LED physique :</div>
        <div id="led-indicator" class="status-dot"></div>
        <p id="led-text">En attente de données...</p>
    </div>

    <script>
        const distSpan = document.getElementById('dist-val');
        const seuilSlider = document.getElementById('seuil-slider');
        const seuilSpan = document.getElementById('seuil-val');
        const ledInd = document.getElementById('led-indicator');
        const ledText = document.getElementById('led-text');

        // 1. Mise à jour du seuil (Envoie au serveur quand le slider bouge)
        seuilSlider.oninput = function() {
            const val = this.value;
            seuilSpan.innerText = val;

            fetch('/set_threshold', {
                method: 'POST',
                headers: {'Content-Type': 'application/json'},
                body: JSON.stringify({ threshold: parseInt(val) })
            });
        };

        // 2. Récupération de la distance (Toutes les 500ms)
        setInterval(() => {
            fetch('/get_distance')
                .then(response => response.json())
                .then(data => {
                    const d = data.distance;
                    const s = parseInt(seuilSlider.value);
                    distSpan.innerText = d.toFixed(1);

                    // Mise à jour visuelle de l'interface pour correspondre à la LED
                    if (d > s) {
                        ledInd.style.background = "green";
                        ledText.innerText = "SÉCURITÉ (Vert)";
                    } else if (d <= s && d > s/2) {
                        ledInd.style.background = "yellow";
                        ledText.innerText = "ATTENTION (Jaune)";
                    } else {
                        ledInd.style.background = "red";
                        ledText.innerText = "DANGER (Rouge)";
                    }
                })
                .catch(err => console.error("Erreur de connexion Flask :", err));
        }, 500);
    </script>
</body>
</html>
```

# 4. Capteur de distance

```python
import pigpio
import time

pi = pigpio.pi()

TRIG = 23
ECHO = 24

def lecture_distance():
    # Déclenchement (Impulsion de 10µs)
    pi.write(TRIG, 1)
    time.sleep(0.00001)
    pi.write(TRIG, 0)

    start = time.time()
    stop = time.time()

    # Attente du début de l'écho
    while pi.read(ECHO) == 0:
        start = time.time()
    
    # Attente de la fin de l'écho
    while pi.read(ECHO) == 1:
        stop = time.time()

    duree = stop - start
    return round((duree * 34300) / 2, 1)
```
