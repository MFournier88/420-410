+++
title = 'Module ADS1115'
date = 2024-08-20T11:53:41-04:00
draft = false
weight = 1001
+++

Le _RaspberryPi_, à la base, peut seulement envoyer ou recevoir des signaux digitaux.

On a vue dans un chapitre précédent comment contourner cette limitation en utilisant PWM pour simuler l'**envoi** d'un signal analogique. Par contre PWM ne peut pas être utilisé pour **recevoir** des signaux analogiques.

Dans cette section, nous verrons comment utiliser un **convertisseur analogique-digital** pour recevoir des signaux d'un senseur analogique afin de les rendre utilisables par un _RaspberryPi_.

Nous utiliserons le convertisseur ADS1115 de *Adafruit*

Tous les senseurs analogiques qu'on souhaite utiliser doivent se connecter sur ce convertisseur: on peut en connecter jusqu'à 4 sur les ports `A0`, `A1`, `A2` et `A3`. Pour récupérer le signal qui provient des senseurs, le *Raspberry Pi* doit lui aussi se connecter sur le convertisseur (ports `VDD`, `GND`, `SCL` et `SDA`).

La librairie I2C doit être utilisée pour pouvoir interpréter ces signaux dans un programme python.


## Connexions
+ VDD sur 3.3V
+ GND sur Ground
+ SCL sur I2C1 SCL
+ SDA sur I2C1 SDA

## Installer les librairies
À partir d'une ligne de commande, installez les librairies python requises avec `pip3 install` dans votre environnement virtuel comme suit:
```bash
pip3 install adafruit-circuitpython-ads1x15 --break-system-packages
```

## Activer interface I2C
Pour activer l'interface *i2c*, il faut lancer la commande `raspi-config`:
```bash
sudo raspi-config
```
Ensuite allez dans `Interface options` pour activer I2C.

## Tester la communication
Vous pouvez tester si votre Pi peut communiquer avec les capteurs en exécutant ce script Python:

*python testADC.py*:
```python
import busio
import board
import time

from adafruit_ads1x15.ads1115 import ADS1115
from adafruit_ads1x15.analog_in import AnalogIn
 
# Initialisation de l'interface i2c
i2c = busio.I2C(board.SCL, board.SDA)
 
# Créer une instance de la classe ADS1115 
# et l'associer à l'interface i2c
ads = ADS1115(i2c)
 
# Créer une instance d'entrée analogique
# et l'associer à la broche 0 du module ADC
data = AnalogIn(ads, 0)
 
# Lire la valeur numérique et le voltage
try:
    while True:
        print(data.value, data.voltage)
        time.sleep(0.5)

except KeyboardInterrupt:
    print("Programme interrompu.")
```
Il n'est pas nécessaire de connecter un senseur: le module ADS1115 devrait retourner des valeurs à lui seul.

Vous devriez voir 2 valeurs s'afficher à chaque demi-seconde (valeur numérique et voltage), comme dans l'exemple suivant:
```
pi@prof:~ $ python3 testADC.py 
7358 0.915125
7345 0.913875
7352 0.914375
7339 0.9167500000000001
7328 0.9185
7314 0.9197500000000001
7317 0.917625
```

## Références
- Convertisseur analogue/digital ADS1115: https://cdn-learn.adafruit.com/downloads/pdf/adafruit-4-channel-adc-breakouts.pdf
- Librairie python pour le module: https://docs.circuitpython.org/projects/ads1x15/