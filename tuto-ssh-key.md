# Créer une clé SSH pour son serveur Linux

Si vous avez plusieurs serveurs Linux (je vous rassure, cela fonctionne aussi si vous n'en avez qu'un !), et que vous en avez marre de devoir taper votre mot de passe à chaque connexion SSH, alors ce tuto est fait pour vous !  
Cette méthode va permettre de créer un système de paire de clés SSH.

Nous aurons une clé Publique (à déposer sur nos/notre serveur), ainsi qu'une clé Privée (pour notre PC client, **à bien conserver !**)

Ainsi, quand nous essayons de nous connecter à notre serveur avec notre clé Privée, il compare avec sa clé Publique, et vous ouvre la porte.  
Cela permet un gain de temps considérable, et d'éviter de faire surchauffer votre disque dur mental avec tous ces mots de passe.

## Génération de la clé

### - Méthode Windows (Putty)

Voici comment procéder pour créer la clé :  
- Générer une paire de clé via puttygen.exe (longueur: 1024 bits minimum)  
- Charger la clé dans son profile PuTTY (Connection -> SSH -> Auth)

### - Méthode Macintosh (Terminal)

Voici comment procéder pour créer la clé :  
- `ssh-keygen -t rsa`

Fini ! Et oui, ce n'est pas plus dur que cela !

## Configuration du serveur (Linux)

- Rajouter la clé publique dans son serveur, dans le fichier `$HOME/.ssh/authorized_keys` sur une seule ligne (doit commencer par ssh-rsa)  
Sur Windows, le faire à la mano, alors que sur une base Unix, préférez : `cat ~/.ssh/id_rsa.pub | ssh user@IPduServer "cat - >> ~/.ssh/authorized_keys"` ou encore `ssh-copy-id -i $HOME/.ssh/id_dsa.pub user@IPduServer` (plus court !)  
- `sudo chmod 700 ~/.ssh`  
- `sudo chmod 600 ~/.ssh/authorized_keys`  
- `sudo chown $USER:$USER ~/.ssh -R`  
- Décommenter dans : `/etc/ssh/sshd_config` la ligne contenant : `AuthorizedKeysFile %h/.ssh/authorized_keys`  
- `sudo service ssh restart`  
En cas d'erreur, consulter les logs via : `tail -f /var/log/auth.log`

## Utilisation

Il faut désormais ce déconnecter du serveur, et tester la connexion via clé SSH.

Connectez-vous, soit en passant par PuTTY (pour les Windowsiens), soit directement depuis le teminal (MacOS et & Tux's users) via la commande : `ssh user@IPduServer`

La connexion sera automatique, pas besoin de mot de passe !

/!\ ATTENTION : En cas de connexion sur plusieurs Users différents, bien penser à mettre la clé publique dans le dossier .ssh de **CHAQUE UTILISATEUR**.
