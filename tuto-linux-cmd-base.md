# Commandes de Base pour le système Unix

## Profil / Gestion du bashrc, autodémarrage, etc

### Démarrage
Pour lancer des commandes dès le login de l'utilisateur, il faut créer un .bash_profile dans $home :
- `sudo mkdir $HOME/.bash_profile`

Et y indiquer les commandes à lancer, comme `startx` par exemple, pour lancer l'interface graphique dès le démarrage.
