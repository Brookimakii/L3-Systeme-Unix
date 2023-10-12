# Compte rendu du TP

Url du mini.iso: http://ftp.lip6.fr/pub/linux/distributions/debian/dists/bookworm/main/installer-amd64/current/images/netboot/
___
## Installation de la VM
L'installation s'est dérouler dans accros.

---

## Installation et Configuration SSH
J'ai eu des diffictultées à installer SSH mais après un redémarage de la VM celà fonctionne grâce à la commande ```apt install ssh```:
- ```dpkg -l ssh``` Verifier l'installation de ssh
- ```ssh root@127.0.0.1``` connect to itself
- ```systemctl status ssh``` see service ssh server actifity
- ```cd /etc/ssh```
- ```cat sshd_config | grep PermitRoot``` Changer Permition root to yes
- ```systemctl restart ssh``` restart ssh

Problème: ```ssh root@127.0.0.1``` fonctionne correctement mais les connections/pings de la machine hôte ne fonctionnent pas pour des raisons pour le moment inconnue...

Solution: ajouter un lien de pont entre la VM et l'hôte sur le port 22 puis connection avec ```ssh root@localhost```

---
#### Debuggage
```ip a``` → inet 10.0.2.15/24  
```ping 10.0.2.15``` → Paquets : envoyés = 4, reçus = 0, perdus = 4 (perte 100%)
```ssh root@10.0.2.15``` → ssh: connect to host 10.0.0 2.15 port 22: Connection timed out

---
### Nombre de Paquets
après la commande: ```dpkg -l | wc -l``` je possède 353 paquets

---
### Résultat de commande
| nom | commande | résultat | explication |
|:----|:---------|:---------|:------------|
|locales|```echo $LANG```|fr_FR.UTF-8| Retourne la langue dans laquele est le systeme |
|nom machine|```hostname```|server1| Retourne le nom de Host du DNS |
|domaine|```domainname```| (none) | Retourne le nom de domaine de la machine |
|verification emplacement depots| ```cat /etc/apt/sources.list \| grep -v -E ’^#\|^$’``` | http://deb.debian.org/debian/ bookworm main | Retourne l'origine et le type d'installer utiliser |
|passwd/shadow|```cat /etc/shadow \| grep -vE ’:\*:\|:!\*:’```| root:\$y\$j9T\$6ImByg2ixv8uFsGJw2hBP/\$JHtTxY0iS2ltRY<br>nOzJUOIK5TKBxif/Ca3EOwLQjYNA3:19632:0:99999:7:::<br><br>messagebus:!:19632::::::<br>avahi-autoipd:!:19632::::::<br>sshd:!:19636:::::: | Retourne la liste des utilisateurs avec: nom, mdp hasher, quand le mdp à été changer pour la dernière fois, le nombre de jour minimum et maximum de validité du mdp, le nombre de jour ou l'utilisateur sera prévenue de l'expiration du mdp |
|compte utilisateurs|```cat /etc/passwd \| grep -vE ’nologin\|sync’```| root:x:0:0:root:/root:/bin/bash | Retourne la liste des utilisateurs avec: nom, mdp (masquer), uid, gid |

---

- expliquer le retour des commandes:
  - ``` fdisk -l``` → affiche les détails du disk avec la liste des partitions
  - ```fdisk -x``` → la liste des partitions avec leurs identifiant
  - ```df -h``` → affiche l'espace occuper et disponible des disk

### Aller plus loin

- A quoi sert ```preseed``` ?


reconstruit l'iso d'installation de la machine.

---

- Comment changer le passwd root ?

On peut utiliser la commande ```passwd``` cela demande un nouveau mot de passe à confirmer.

---

- Comment redimentionner la partition racine ?

Étape 1: ```fdisk /``` supprimer la partition.  
Étape 2: reboot la machine.  
Étape 3: ```resize2fs /```  redimentionner la partition.
