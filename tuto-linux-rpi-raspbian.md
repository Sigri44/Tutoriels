# Les bases d'installation RaspBian

Configuration basée sur la dernière en date, Raspbian Stretch (Debian 9)
On passe le côté dL, copie d'image img et compagnie, et on attaque la première config

## 1 - Configuration basique

En premier lieu, étendre le système sur toute la carte :
- `sudo raspi-config`

## 2 - Suppression fichiers inutiles

Pour gagner environ 1 Go  de place, et si surtout vous ne comptez PAS utiliser la partie graphique :

### a) Désinstallation des paquets superflus
`sudo aptitude remove -y --purge desktop-base x11-common midori omxplayer scratch dillo xpdf galculator netsurf-common netsurf-gtk lxde-common lxde-icon-theme hicolor-icon-theme libpoppler19 ed lxsession lxappearance lxpolkit lxrandr lxsession-edit lxshortcut lxtask lxterminal xauth debian-reference-common fontconfig fontconfig-config fonts-freefont-ttf wolfram-engine dbus-x11 desktop-file-utils libxmuu1 libraspberrypi-doc`

### b) Désinstallation des paquets et dépendances qui ne sont plus utilisées
`aptitude autoremove -y`

### c) Suppression du cache des paquets de apt
`aptitude clean`

### d) Suppression des fichiers inutiles
ATTENTION : Si vous souhaitez utiliser les composants vidéo du Pi et notamment les outils raspivid ou raspistill, vous ne devez pas supprimer le répertoire /opt/vc. De manière générale, ne supprimer ce répertoire que si vous n’utilisez votre Pi en tant que serveur  (LAMP, NAS, FTP,…) sans utiliser de composants propres au Pi. Dans le doute, ne supprimez pas ce répertoire. Si vous l’avez supprimé par erreur, vous pouvez réinstaller son contenu en lançant la commande `rpi-update`.

`rm -rf /opt/* /usr/share/icons/* /usr/games /usr/share/squeak /usr/share/sounds /usr/share/wallpapers /usr/share/themes /usr/share/kde4 /usr/share/images/* /home/pi/python_games`

Voilà avec cela, vous repassez sous la barre des 1/2 Go utilisés, et vous gardez un Raspbian parfaitement fonctionnelle.
