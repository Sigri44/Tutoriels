# RPi - HomeScreen

Cette partie va traiter sur la création d'un système d'homescreen sur un RaspberryPi afin de le transformer en monitoring, pour suivre la conso de son RPi, ou encore l'activité de sa BP, ou même le nombre d'utilisateurs connectés à son VPN maison.
Pour cela rien de plus simple, il vous faut :
- un kit RPi complet (cart SD, alim, la base !)
- un écran (préférez du LED (angle de vue), et pas trop grand)
- support VESA c'est le top !
- du temps pour suivre ce tuto

Je passe l'étape de l'installation de RaspBian, qui est déjà le sujet d'un autre tuto, on attaque directement.

## Déplacer les fichiers temporaires

Afin d'éviter d'user la carte SD en cycle d'écriture, nous allons déplacer les temp et réduire leur taille, vers la RAM via l'outil tmpfs :

- `sudo nano /etc/fstab`

et y ajouter 

`tmpfs /tmp tmpfs defaults,noatime,nosuid,size=10m 0 0`
`tmpfs /var/tmp tmpfs defaults,noatime,nosuid,size=10m 0 0`

et pour les logs :

`wget http://www.tremende.com/ramlog/download/ramlog_2.0.0_all.deb && sudo aptitude install lsof`
`sudo dpkg -i ramlog_2.0.0_all.deb`
`sudo reboot`
`sudo /etc/init.d/ramlog status`

## Instalation des dépendances

En premier lieu, il va falloir installer toutes les mise à jour, et enfin l'attirail complet du homescreen :

- `sudo aptitude update -y && sudo aptitude upgrade -y && sudo rpi-update`

- `sudo aptitude install apache2 php5 unclutter chromium`

- apache2 : pour le serveur Web
- php5 : pour exécuter le PHP
- chromium : pour lancer notre page Web
- unclutter : afin de masquer le curseur

### Faciliter les droits d'accès

- `sudo chown -R www-data:www-data /var/www`
- `sudo usermod -aG www-data pi`
- `sudo chown -R g+w /var/www`

On vérifie que c’est bon en saisissant l’adresse “http://%IPduRPi/” dans un navigateur, une page “It Works !” doit s’afficher.

## Associer l’ouverture de cette page web au lancement du serveur X

`sudo nano /home/pi/.xinitrx`

Et y saisir :

`#!/bin/sh
# Cache le curseur de la souris au bout de 1 seconde
unclutter -idle 1 &
# Demarre le gestionnaire de fenetres
openbox-session &
# Demarre Chromium
chromium –kiosk –incognito « http://localhost/ »`

On peut tester si ça marche en tapant `startx` à l’invite de commande : une fenêtre noire doit apparaître, avec le curseur de souris au milieu, puis le navigateur internet s’ouvre en plein écran et notre page noire de toute à l’heure s’affiche.

### Paramétrer le lancement automatique de l’afficheur eu démarrage du RPi
Rien de compliqué, il suffit d’ajouter une ligne au fichier `sudo nano /etc/rc.local`, avant la ligne finale `exit 0` qui doit obligatoirement clore ce dernier :
`su – pi -c ‘startx’ &`

NB1 : su – pi = permet de démarrer un nouveau shell de connexion en tant que pi (il y a bien un espace avant “pi”)
NB2 : -c ‘startx’ = permet de lancer la commande de démarrage de X
NB3 : & = permet de lancer tout ça sans s’arrêter et bloquer la suite des événements.

Et voilà, au démarrage du RPi, on doit tomber automatiquement sur notre page noir, ne reste plus qu’à y ajouter des modules, la partie la plus sympa du projet !

## Ajout de modules
