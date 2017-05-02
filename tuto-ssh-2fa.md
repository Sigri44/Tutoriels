# Activer l’authentification en 2 étapes pour votre serveur SSH

Tout est dans le titre. Pour ceux qui ne connaitraient pas encore l'authentification à 2 étapes (2FA), il s'agit en plus du mot de passe habituel, d'utiliser un second code, généré via token (tel que Google Authentificator)

`sudo apt-get install libpam-google-authenticator`
puis :
`google-authenticator`

Première question, voulez-vous des codes basés sur le temps ? Répondez-oui, cela permet de générer des codes qui sont limités dans le temps. (c'est un peu le but !)
`y`

Ensuite, Google Authenticator vous générera un gros QR Code qu'il faudra flasher avec l'application Authenticator (Pour [iOS ici](https://itunes.apple.com/fr/app/google-authenticator/id388497605?mt=8) et pour [Android ici](https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2&hl=fr)).
Vous pouvez aussi y accéder via l'URL fournie dans le terminal.

**Attention TRÈS IMPORTANT** : Vous aurez aussi une série de codes d'urgences générez que vous devez absolument conserver dans un coin. Cela vous permettra de vous connecter quand même à votre serveur même si votre téléphone vous a lâché ou que vous l'avez formaté.

Appuyez ensuite sur la touche "y" pour mettre à jour la config Google Authenticator.

L'outil vous posera ensuite 3 questions relatives à la configuration :
- 1re : Désactiver l'utilisation multiple du même code. Niveau sécurité, je vous conseille de faire `y` pour désactiver cette possibilité. Cela vous protégera en cas d'attaques MITM.
- 2è  : Durée de péremption des codes. Par défaut, un code est valide **1min30** après qu'il soit passé. Cela permet d'absorber les latences qu'elles soient réseau ou humaines (si vous tapez lentement ). Là on vous demande si vous voulez garder cette durée à 1 min 30 ou la passer à 4 min si vous éprouvez des difficultés. Je vous conseille de répondre `n` à cette question.
- 3è  : Activer une limite maximum de codes erronés. Permet d'éviter les attaques de type bruteforce. Répondez `y` à cette question.

Ensuite, il faut configurer ssh pour qu'il prenne en compte ce module.
`sudo nano /etc/pam.d/sshd` et y ajouter :
`auth required pam_google_authenticator.so`

Puis éditez le fichier :
`sudo nano /etc/ssh/sshd_config`
Et mettez à `yes`, la ligne :
`ChallengeResponseAuthentication yes`

Puis relancez ssh :
`sudo service ssh restart`
