+++
title = 'Moteurs pas à pas'
date = 2025-01-27T16:59:54-05:00
draft = false
weight = 32
+++

![stepper28byj](/420-410/images/stepper28byj.png?width=300px)

Les moteurs "pas à pas" (en anglais _stepper motors_) fonctionnent en utilisant les mêmes principes que les moteurs DC. Cependant il diffèrent de ces derniers de deux manières:
- Ils contiennent une série d'engrenages qui servent à démultiplier la vitesse de rotation pour le convertir en force de rotation (le _couple_)
- Ils utilisent plus de deux bobines (ou "coils" en anglais) pour générer la rotation du moteur, ce qui permet de contrôler cette rotation avec une plus grande précision.
  
Cette combinaison de couple relativement élevé et de haute précision est typique des moteurs pas-à-pas. Ils sont donc très utilisés dans les systèmes où ces propriétés sont importantes, comme par exemple en robotique ou dans les imprimantes.

## Fonctionnement
Comme dans un moteur DC, l'arbre du rotor (la tige qui effectue le mouvement de rotation) est mise en mouvement par l'action d'un champ électrique sur les deux pôles d'un aimant:

![singlestep](/420-410/images/singlestep.png?width=600px)

Dans le schéma précédent, le courant électrique dans chacun des bobines tour à tour entraîne le mouvement d'un quart de tour du rotor. C'est en raison de cette séquence qu'on nomme ce type de moteur "stepper": chaque étape de la séquence est un "step". 

Il y a plus d'une manière de réaliser ces séquences, chacune ayant ses avantages par rapport aux autres. L'exemple précédent est simple et illustre bien le fonctionnement d'un moteur pas-à-pas mais la séquence qu'il représente est rarement utilisée. On préfère généralement la séquence suivante:

![fullstep](/420-410/images/fullstep.png?width=600px)

Dans cette séquence, le fait d'alimenter deux bobines au lieu d'une seule a pour effet d'augmenter le couple: le mouvement peut ainsi entraîner des charges plus lourdes. On nomme généralement cette séquence "Full step".

Une autre séquence (nommée "half-step") est constituée de 8 étapes:

![halfstep](/420-410/images/halfstep.png?width=600px)

L'avantage d'utiliser cette séquence est qu'elle a une plus grande précision carle mouvement angulaire de chaque étape est plus petit (45° au lieu de 90° dans cet exemple). Le couple est légèrement plus faible que dans une séquence "full step" cependant.

## Connexions
#### Alimentation
Le module L239D, comme on l'a vu dans la section précédente, permet de contrôler 2 moteurs DC, qui chacun comprennent deux bobines. Comme notre moteur contient 4 bobines, il est aussi possible de le contrôler avec ce module. 

Le moteur 28BYJ-48 que nous utiliserons est alimenté par un courant de 5V, comme le module L239D. On peut donc utiliser une même broche 5V du Raspberry Pi comme source de courant pour les deux. Les broches _Enable1_ et _Enable2_ peuvent être également connectées sur 5V car on n'utilisera pas la fonction d'interrupteur du module L239D (le moteur sera toujours activé). De la même manière, on connectera tous les ports GND du module sur la même broche GND du Pi. Vous pouvez donc relier les broches entre elles comme suit:

![stepconnect1](/420-410/images/stepconnect1.png?width=600px)

Ensuite, reliez le tout aux broches 5V et GND du RaspberryPi:

![stepconnect2](/420-410/images/stepconnect2.png?width=600px)

#### Moteur
Le moteur 28BYJ-48 dispose de 5 fils. Le fil rouge est utilisé pour un courant général de 5V, mais nous n'en avons pas besoin. Les 4 fils qui restent doivent chacun être reliés à une bobine selon les correspondances suivante:
- orange: coil1 
- jaune: coil2 
- rose: coil3 
- bleu: coil4

> Attention: la documentation technique du moteur inverse les fils rose et jaune. Il est donc possible que vous deviez essaiyer différentes combinaisons.

Dans les exemples de code qui suivent, on utilise les broches suivantes pour contrôler chacune des bobines:
- GPIO26: Coil1 
- GPIO13: Coil2
- GPIO19: Coil3
- GPIO6:  Coil4

Connectez donc votre Pi et le moteur au module L239D comme suit:

![L293Dgpio](/420-410/images/L293Dgpio.png?width=500px)

## Programme

Le programme suivant fait tourner le moteur indéfiniment à sa vitesse maximum recommandée:
```python
import pigpio
import time

# GPIO
M1,M2,M3,M4 = 26,13,19,6

# Minimum recommandé: 2ms (0.002)
step_pause = 0.002

# Séquence "Single coil" 
seq_simple = [
    [1,0,0,0],
    [0,1,0,0],
    [0,0,1,0],
    [0,0,0,1]
]

# Séquence "Full-step" = plus de couple 
seq_full = [
    [1,0,0,1],
    [1,1,0,0],
    [0,1,1,0],
    [0,0,1,1]
]

# Séquence "Half-step" = plus de précision
seq_half = [
    [1,0,0,0],
    [1,1,0,0],
    [0,1,0,0],
    [0,1,1,0],
    [0,0,1,0],
    [0,0,1,1],
    [0,0,0,1],
    [1,0,0,1]
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
        for step in seq_simple: # Changez la séquence ici au besoin
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

## Exercices
1. Le moteur 28BYJ-48 doit effectuer 2048 _steps_ pour faire un tour complet. Combien de _steps_ correspondent environ à un degré de rotation?
2. Faites un programme qui fait un tour, puis un deuxième en sens contraire et s'arrête.
{{% expand "Solution" %}}
```python
import pigpio
import time

M1,M2,M3,M4 = 26,13,19,6
steps_par_tour = 2048

# Minimum recommandé: 2ms (0.002)
step_pause = 0.002

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

n = 0

try:
    while n < 2 * steps_par_tour:
        for step in seq_full:
            # Activer chacune des 4 bobines
            pi.write(M1, step[0])
            pi.write(M2, step[1])
            pi.write(M3, step[2])
            pi.write(M4, step[3])

            n += 1
            # La durée de la pause détermine la vitesse
            time.sleep(step_pause)
        if n == steps_par_tour: seq_full.reverse()

except KeyboardInterrupt:
    stop_moteur()
```
{{% /expand %}}

3. Faites un programme qui fait un tour, puis un deuxième en sens contraire à la moitié de la vitesse puis s'arrête. 
{{% expand "Solution" %}}
```python
import pigpio
import time

M1,M2,M3,M4 = 26,13,19,6
steps_par_tour = 2048
step_pause = 0.002

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

n = 0

try:
    while n < 2 * steps_par_tour:
        for step in seq_full:
            # Activer chacune des 4 bobines
            pi.write(M1, step[0])
            pi.write(M2, step[1])
            pi.write(M3, step[2])
            pi.write(M4, step[3])

            n += 1
            # La durée de la pause détermine la vitesse
            time.sleep(step_pause)
        if n == steps_par_tour: 
            seq_full.reverse()
            step_pause *= 2

except KeyboardInterrupt:
    stop_moteur()
```
{{% /expand %}}

4. Faites un programme qui fait une rotation de 45 degrés, change de sens, fait 20 degrés, change encore de direction et fait 90 degrés au quart de la vitesse.
{{% expand "Solution" %}}
```python
import pigpio
import time

STEPS_PAR_TOUR = 2048

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

def degree_to_steps(angle):
    return angle / 360 * 2048


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

angles = [45, 20, 90]

try:
    for angle in angles:
        compteur = 0
        steps_to_do = degree_to_steps(angle)
        while compteur < steps_to_do:
            for step in seq_full: # Changez la séquence ici au besoin
                # Activer chacune des 4 bobines
                pi.write(M1, step[0])
                pi.write(M2, step[1])
                pi.write(M3, step[2])
                pi.write(M4, step[3])
                compteur += 1
                # La durée de la pause détermine la vitesse
                time.sleep(step_pause)
        
        seq_full.reverse()
    
    

except KeyboardInterrupt:
    pass
finally:
    stop_moteur()

```
{{% /expand %}}

5. Faites un programme qui fait 1 tour en ralentissant graduellement, change de sens, et fait un tour en revenant graduellement à la vitesse initiale. Pour la vitesse maximale, utilisez un délai de 2ms et pour la vitesse minimale, 10ms.
{{% expand "Solution" %}}
```python
import pigpio
import time

STEPS_PAR_TOUR = 2048

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

def degree_to_steps(angle):
    return angle / 360 * 2048


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


steps_to_do = 2048 * 2
compteur = 0
sign = 1
try:
    while compteur < steps_to_do:
        for step in seq_full: # Changez la séquence ici au besoin
            # Activer chacune des 4 bobines
            pi.write(M1, step[0])
            pi.write(M2, step[1])
            pi.write(M3, step[2])
            pi.write(M4, step[3])
            compteur += 1
            # La durée de la pause détermine la vitesse
            time.sleep(step_pause)
            step_pause = step_pause + sign * 0.008 / 2048
            print(step_pause)
            # 0.002 -> 0.01 : Je dois augmenter de 0.008 en 2048 steps
        if compteur == STEPS_PAR_TOUR: 
            seq_full.reverse()
            sign = -1
    
    

except KeyboardInterrupt:
    pass
finally:
    stop_moteur()


```
{{% /expand %}}

