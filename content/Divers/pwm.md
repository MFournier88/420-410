+++
title = 'PWM'
date = 2026-01-26T20:30:39-05:00

+++

---

### Analogie

Imaginez que vous êtes devant un interrupteur de lumière.

* Si vous l'allumez et l'éteignez **50 fois par seconde** :
* Si l'interrupteur reste **ON** 90% du temps, la LED paraîtra très brillante.
* Si l'interrupteur reste **ON** seulement 10% du temps, la LED paraîtra très faible.



C'est ce qu'on appelle le **Rapport Cyclique** (Duty Cycle).

---

### Exemple de code minimaliste (Aide-mémoire)

Voici le bloc de code "clé" qu'ils devront adapter. Cet exemple montre comment faire varier l'intensité d'une seule broche de 0 à 100% :

```python
import pigpio

pi = pigpio.pi()
BROCHE_LED = 17

# pigpio utilise une plage de 0 à 255
# 0   = 0%   (Éteint)
# 128 = 50%  (Lumière moyenne)
# 255 = 100% (Pleine puissance)

pi.set_PWM_dutycycle(BROCHE_LED, 128) # Allume à moitié

```

### Points clés à leur rappeler :

1. **Plage de valeurs :** Avec `pigpio`, la valeur envoyée à `set_PWM_dutycycle` doit être un entier entre **0 et 255**.
2. **Couleur aléatoire :** Pour la commande `allume`, ils devront générer trois nombres aléatoires (un pour chaque canal : Rouge, Vert, Bleu).
3. **Nettoyage :** Il est crucial d'utiliser `pi.stop()` à la fin pour libérer les ressources du démon.

---

