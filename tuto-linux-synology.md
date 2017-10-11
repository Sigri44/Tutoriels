# Tutoriel sur l'utilisation Unix de Synology

## Installer gestionnaire paquets tiers (Entware)

A l'image d'aptitude, nous pouvons installer un gestionnaire compatible Synology 5 et 6 :

Premier lieu, connaitre son proc' :
`uname -a`

- Arm5 : `wget -O - http://pkg.entware.net/binaries/armv5/installer/entware_install.sh | /bin/sh`
- Arm7 : `wget -O - http://pkg.entware.net/binaries/armv7/installer/entware_install.sh | /bin/sh`

Créer un dossier sur le disque, en dehors du rootfs :
`mkdir -p /volume1/@entware-ng/opt`
`rm -rf /opt`
`ln -sf /volume1/@entware-ng/opt /opt`

On éditer le fichier `/etc/rc.local` et on ajoute à la fin du fichier:
```
/bin/ln -sf /volume1/@entware-ng/opt /opt
/opt/etc/init.d/rc.unslung start
```
La dernière ligne permet de lancer les services Entware lors du démarrage du NAS.

Depuis DSM 6, créer une tâche planifier depuis le panneau de configuration, et y indiquer le script suivant :
```
/bin/ln -sf /volume1/@entware-ng/opt /opt
/opt/etc/init.d/rc.unslung start
```

Et enfin ajouter cette ligne à la fin du fichier `/etc/profile`:

`. /opt/etc/profile`

`sudo reboot`

`sudo opkg update`

`sudo opkg install nano`
