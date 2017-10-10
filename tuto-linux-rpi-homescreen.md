# RPi - HomeScreen

Cette partie va traiter sur la création d'un système d'homescreen sur un RaspberryPi afin de le transformer en monitoring, pour suivre la conso de son RPi, ou encore l'activité de sa BP, ou même le nombre d'utilisateurs connectés à son VPN maison.
Pour cela rien de plus simple, il vous faut :
- un kit RPi complet (cart SD, alim, la base !)
- un écran (préférez du LED (angle de vue), et pas trop grand)
- support VESA c'est le top !
- du temps pour suivre ce tuto

Je passe l'étape de l'installation de RaspBian, qui est déjà le sujet d'un autre tuto, on attaque directement.

## Instalation des dépendances

En premier lieu, il va falloir installer tout l'attirail du homescreen :

- `sudo aptitude update -y && sudo aptitude upgrade -y && sudo aptitude install xxxxxxxx`
