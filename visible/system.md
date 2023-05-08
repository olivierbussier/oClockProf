## Quelques commandes systeme ubuntu

Avoir des informations sur les utilisauteurs et le systeme
```bash
$ finger [user] # Info sur le user courant ou celui passé en paramètre
$ whoami # affiche l'utilisateur de la session courante
```

Executer une commande avec les droits administrateur ou changer de compte
```bash
sudo <commande> #executer une seule commande avec les droits root
sudo -i # Se connecter en tant que root
sudo -i -u <user> # se connecter en tant que <user>
```

Le systeme vous demande une première fois votre mot de passe qu'il conservera 15mn

Avoir des informations sur les services

```bash
sudo systemctl status <service> # Etat d'un service
sudo systemctl [list-units] # Liste l'état de tous les services
sudo systemctl lis-unit-files # Liste les fichiers en visi de systemd

sudo systemctl start <service> # Démarrage d'un service
sudo systemctl stop <service> # Arrêt d'un service
sudo systemctl restart <service> # Arrêt et redémarrage d'un service
sudo systemctl reload <service> # Rechargement des config sans redémarrage
sudo systemctl reload-or-restart <service> # reload si cela est implémenté, restart sinon

sudo systemctl enable <service> # Active le service au démarrage
sudo systemctl disable <service> # Désactive le service au démarrage
```
Editer un fichier
```shell
$ nano <fichier>
```