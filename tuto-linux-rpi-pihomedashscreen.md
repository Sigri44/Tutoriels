# RPi - PiHomeDashScreen

Cette partie va traiter sur la création d'un système d'homescreen sur un RaspberryPi afin de le transformer en monitoring, pour suivre la conso de son RPi, ou encore l'activité de sa BP, ou même le nombre d'utilisateurs connectés à son VPN maison.
Pour cela rien de plus simple, il vous faut :
- un kit RPi complet (cart SD, alim, la base !)
- un écran (préférez du LED (angle de vue), et pas trop grand)
- support VESA c'est le top !
- du temps pour suivre ce tuto

Pour le boot automatique sous Wheezy : `1:2345:respawn:/sbin/login -f pi tty1/dev/tty1 2>&1` dans /etc/inittab


Je passe l'étape de l'installation de RaspBian, qui est déjà le sujet d'un autre tuto, on attaque directement.

## Déplacer les fichiers temporaires

Afin d'éviter d'user la carte SD en cycle d'écriture, nous allons déplacer les temp et réduire leur taille, vers la RAM via l'outil tmpfs :

`sudo nano /etc/fstab`

et y ajouter 
```
tmpfs /tmp tmpfs defaults,noatime,nosuid,size=10m 0 0
tmpfs /var/tmp tmpfs defaults,noatime,nosuid,size=10m 0 0
```
et pour les logs :

`wget http://www.tremende.com/ramlog/download/ramlog_2.0.0_all.deb && sudo aptitude install lsof && sudo dpkg -i ramlog_2.0.0_all.deb && sudo reboot`

puis au démarrage :

`sudo /etc/init.d/ramlog status`

### En cas d'erreur au boot

Si jamais au démarrage vous avez l'erreur `[FAIL] Starting ramlog-tmpfs 2.0 error /var/log is in use... failed !`, il suffit de faire :

`sudo apt-get install rsync lsof -y && wget http://www.tremende.com/ramlog/download/ramlog_2.0.0_all.deb && sudo dpkg -i ramlog_2.0.0_all.deb`

et ajouter dans le fichier `/etc/init.d/ramlog` :
```
# X-Start-Before: rsyslog
# X-Stop-After: rsyslog
```
dans le cadre qui contient `###BEGIN INIT INFO`.

puis la commande `sudo insserv && sudo reboot`

et enfin au démarrage, de nouveau pour vérifier si le service est bien activé :

`sudo /etc/init.d/ramlog status`

## Instalation des dépendances

En premier lieu, il va falloir installer toutes les mise à jour, et enfin l'attirail complet du homescreen :

`sudo aptitude update -y && sudo aptitude upgrade -y && sudo rpi-update`

`sudo aptitude install apache2 php5 php5-cli xinit openbox chromium-browser imagemagick php5-imagick php5-gd xplanet unclutter mingetty x11-xserver-utils lightdm`

- apache2 : pour le serveur Web
- php5 : pour exécuter le PHP
- chromium : pour lancer notre page Web
- unclutter : afin de masquer le curseur
- openbox : gestionnaire de fenêtre très léger pour gérer l’affichage du navigateur
- xinit : permet d’installer tout ce qu’il faut pour le serveur X
- imagemagick : pour manipuler des images
- xplanet : pour générer les images de la terre et de la lune
- x11-xserver-utils : pour gérer la mise en veille de l’écran

### Désactiver la mise en veille

Afin de ne pas être embêté par la mise en veille régulière de l'écran, nous allons réguler tout cela :

`crontab -e`

```
0 0 * * *       su - pi -c "xset -display :0 dpms force off" # met le moniteur en veille à minuit
0 8 * * *      su - pi -c "xset -display :0 dpms force on" #réveille le moniteur à 8h00 du matin
```

### Faciliter les droits d'accès

- `sudo chown -R www-data:www-data /var/www`
- `sudo usermod -aG www-data pi`
- `sudo chown -R g+w /var/www`

On vérifie que c’est bon en saisissant l’adresse “http://%IPduRPi/” dans un navigateur, une page “It Works !” doit s’afficher.

### Configurer notre site

Ce n'est pas dur, nous allons faire au plus simple, remplacer le contenu de :

`sudo nano /etc/apache2/sites-enabled/000-default (rajouter .conf à la fin selon les versions)`

par

```
<VirtualHost *:80>
        DocumentRoot /var/www/
</VirtualHost>
```

## Associer l’ouverture de cette page web au lancement du serveur X

Dans `sudo raspi-config`, choisir l'option `3 - Boot Options`, puis `B1 Desktop / CLI` puis `B4 Desktop Autologin`

Cela va permettre au RPi de booter directement sur l'interface graphique.

Ensuite, on va simplement dire à openbox de lancer directement un chromium au démarrage de startx :

Dans le fichier `/home/pi/.config/openbox/autostart`, y ajouter :

`chromium –kiosk –incognito http://localhost/`

On peut tester si ça marche en tapant `startx` à l’invite de commande : une fenêtre noire doit apparaître, avec le curseur de souris au milieu, puis le navigateur internet s’ouvre en plein écran et notre page noire de toute à l’heure s’affiche.

## Ajout de modules

### Créer notre premier module

Voici comment créer et donc, comment fonctionne, un module. Choisir un nom simple et significatif (exemple : horloge) :

Dans le répertoire /var/www/modules/, on ajoute un dosier protant le même nom que le module
Y placer les deux fichiers de base de chaque module :
– `index.php` : qui va contenir le programme du module,
– `nom-du-module.css` : qui va contenir sa mise en page => `horloge.css` pour notre 1er module.
Activer le module dans le fichier `config.json` :
En ajoutant au ficher `config.json` les lignes suivantes :
```
{
“name”: “nom-du-module”,
“width”: “530px”,
“top”: “10px”,
“left”: “10px”,
“update”: 1,
“args”:
{
“nom-argument-1”: “valeur”,
“nom-argument-2”: “valeur”
}
}
```
– “name” reprend le nom exact du module
– “width”, “height”, “top” et “left” permettent de dimensionner et de placer le module par rapport au coin haut gauche de l’écran
– “update” permet de donner une fréquence de rafraichissement du module, en secondes, ou “0” pour ne jamais le rafraichir.
– la sous-partie “args” est facultative et permet de passer au module des info supplémentaires, comme la taille de la police de caractères à utiliser pour l’heure et la date de notre module “horloge”.

Si on ouvre le fichier “index.php” du module “horloge” par exemple :
– on trouve d’abord quelques lignes de php permettant de récupérer les “args” du fichier `config.json`
– puis un script javascript qui récupère l’heure et la date à un instant t et les met en forme.

### Module xPlanet

Unzippez le fichier dans `/var/www/`

Pour lancer xPlanet au démarrage, il faut :

copier le script “xplanet.sh” dans le répertoire "/etc/init.d/”
`sudo cp /var/www/tdb314/ressources/xplanet/xplanet.sh /etc/init.d/xplanet.sh`
le rendre exécutable
`sudo chmod 755 /etc/init.d/xplanet.sh`
demander son lancement à chaque démarrage avec la commande
`update-rc.d xplanet.sh defaults`

Et ensuite, l'insérer dans notre site :

modifier le fichier `sudo nano /var/www/config.json` en ajoutant les lignes :
``
{
“name”: “xplanet_terre”,
“width”: “450px”,
“height”: “450px”,
“top”: “13px”,
“left”: “25px”,
“update”: 120
},
{
“name”: “xplanet_lune”,
“width”: “224px”,
“height”: “224px”,
“top”: “13px”,
“left”: “480px”,
“update”: 600
},
``
Libre à vous de modifier ces options !
NB : attention aux virgules dans votre fichier “.json” : il y a une virgule entre chaque accolade “fermante / ouvrante” qui séparent deux modules, mais il n’y a pas de virgule derrière l’accolade de fermeture du dernier module !

`sudo reboot`

### Module Ping

Dans le fichier `inc.php`, inscrire dans la fonction `$hosts_ip = array`, les adresses IP ainsi que les ports que vous souhaitez tester.

Normalement les lumières devraient s'allumer au vert sur votre DashScreen ! (ou rouge clignotant si hors-ligne)

### Module Bande Passante

Côté client (Routeur, Synology, SRV etc...), copier le fichier `nc_ifstat_clt.sh` dans le dossier `/opt/etc/init.d/` (ou simplement `/etc/init.d` suivant les systèmes). N'oubliez pas de modifier les IP pour qu'elles correspondent à votre réseau.

Puis créer une tâche planifiée de démarrage (via planificateur pour ma part), et y exécuter cette commande :

`/opt/etc/init.d/nc_ifstat_clt.sh start`

Côté Serveur (RPi), pour tester si la fonction de base fonctionne, faites un `ifstat -n -i eth0`, sinon installez le paquet.

Puis installer le fichier `nc_ifstat_srv.sh` dans `/etc/init.d/` et lancer la commande :

`update-rc.d nc_ifstat_srv.sh defaults`
`sudo reboot`


### Module Météo

Pas grand chose à faire, ne serait-ce qu'aller sur le site [!Widget Météo]http://www.meteocity.com/widget/ afin de créer son propre widget, et ensuite copier-coller le code fourni dans le fichier `inc.php`
