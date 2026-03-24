+++
title = 'Broker'
date = 2025-03-23T17:45:31-04:00
draft = false
weight = 53
+++

## Installation
Pour installer un _broker_ MQTT dans linux, il suffit d'installer le paquet _mosquitto_. Dans les distributions basées sur _Debian_ la commande est celle-ci:
```
apt install mosquitto
```

Il est recommandé de créer un fichier de configuration avec vos propres paramètres. 
```
nano /etc/mosquitto/mosquitto.conf
chmod -R 755 /etc/mosquitto/mosquitto.conf
chown -R mosquitto:mosquitto /var/log/mosquitto
chmod -R 755 /var/log/mosquitto
```

Mettre ceci dans le fichier de config `/etc/mosquitto/mosquitto.conf`
```
persistence true
persistence_location /var/lib/mosquitto/
log_dest file /var/log/mosquitto/mosquitto.log
allow_anonymous true
listener 1883
```

La première ligne spécifique que le port 1883 (port TCP standard pour MQTT) doit être utilisé par le programme. La deuxième ligne permet aux clients d'utiliser le _broker_ sans avoir à s'authentifier.

En cas d'erreur, utiliser ce code pour avoir un message d'erreur détaillé:

```bash
mosquitto -c /etc/mosquitto/mosquitto.conf
```
```bash
killall mosquitto
```
Dès que vous apporté des modification à la configuration, faites un :

```bash
systemctl restart mosquitto
```
## Configurer l'authentification
C'est la configuration de l'agent ("broker") qui détermine si les clients doivent s'authentifier ou non. Dans le fichier de configuration du _broker_, la variable `allow_anonymous` doit être à `false`, et `password_file` doit indiquer le fichier qui contient les identifiants et mots de passe. Par exemple:
```
allow_anonymous false
password_file /etc/mosquitto/users
```
On peut créer autant d'utilisateurs qu'il y a de clients qui se connectent, mais il est aussi possible que plusieurs clients se partagent les mêmes identifiants.

La commande `mosquitto_passwd` permet de gérer les identifiants de connexion. On doit lui passer le chemin du fichier qui contient les mots de passe. 
- `mosquitto_passwd FICHIER UTILISATEUR` : Ajoute un utilisateur ou change le mot de passe d'un utilisateur existant.
- `mosquitto_passwd -D FICHIER UTILISATEUR` : Supprime l'utilisateur.
- `mosquitto_passwd -c FICHIER UTILISATEUR` : Crée le fichier des utilisateurs et y ajoute un utilisateur. Attention, si le fichier existe déjà il sera écrasé.


```bash
chmod 755 /etc/mosquitto/users
systemctl restart mosquitto
```

