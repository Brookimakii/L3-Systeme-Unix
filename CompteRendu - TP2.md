# TP2
## Secure Shell: SSH
### Connection ssh root

```man sshd_config``` 
Option changer: PermitRootLogin dont la valeur par défaut est "prohibit-password".

##### Les différentes possibilités:
| Valeur | Utilité | Aventages | Inconvénients | Cas d'utilisation |
|:-------|:--------|:----------|:--------------|:------------------|
| yes    |Autorise tout login root. | Simplifie les logins. | N'est pas sécurisé car quiconque connait le mdp root peut s'y log. | A utiliser le temps de la configuration de d'autre options. |
| prohibit-password | Désactive mdp et interractions clavier pour un login root. | Obliger les connection avec une clef d'authentification. | Si la cléf est perdu ou le machine de utilisateur n'est pas la même toute connection devient impossible. | Lorsque le serveur est "public" permettant d'agir dessus tout en étant sécurisé. |
| forced-commands-only | Permet les connection root mais empêche les access d'interraction. | Empêche les trop grosse modification par root, empêchant ainsi les erreurs pouvant être fatale. | Même le root ne peut prèsque rien faire entant que root. |  |
| no     | Désactive tout login root. | Empêche quiconque d'agir en root empêchant divers tentatives de sabotage. | Le propiétaire de la machine ne pourra pas non plus se connecter à distance en tant que root. |  |
___

### Authentification par clef / Génération de clefs

Execution de la commande ```ssh-keygen``` sur la machine hôte créant la cléf d'authantification.
Execution de la commande ```type C:\Users\${Nom utilisateur}\.ssh\id_rsa.pub | ssh root@localhost "cat >> .ssh/authorized_keys"``` pour copier la cléf vers le server SSH.
J'ai ensuite ajouté la ligne "PasswordAuthentication no" au fichier sshd_config pour désactiver les connection par mpd

### Authentification par clef / Connection serveur

Grâce à l'étape précédente le fichier authorized_keys à déjà été créer.

Pour en modifier les droits pour que seul le propriétaire (root) puisse: le lire, l'écrire ou l'exécuter, on exécute la commande suivante: ```chmod 700 authorized_keys``` dans le dossier ".ssh".

### Authentification par clef: depuis la machine hote

La commande ```ssh -i C:\\Users\\###\\.ssh\\id_rsa.pub root@ipserveur``` ne fonctionne pas pour cause de clef "trop public". Voici le message d'eurreur:
```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions for 'C:\\Users\\###\\.ssh\\id_rsa.pub' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "C:\\Users\\Brookii-PC\\.ssh\\id_rsa.pub": bad permissions
root@localhost: Permission denied (publickey).
```
Cenpendant la commande ```ssh -i C:\\Users\\###\\.ssh\\id_rsa root@ipserveur``` fonctionne et log correctement l'utilisteur.

### Sécurisez
l'ajout de la ligne "PasswordAuthentication no" au fichier sshd_config à la deuxième étape permet de désactiver les logins par mdp

## Processus

### Etude des processus UNIX
#### A l’aide de la commande ps
Utilisation de la commande ```ps -eo user,pid,%cpu,%mem,stat,start,time,command```

- A quoi correspond l'information TIME ?
> Il s'agit du temps total que le CPU passe sur le processus

- Quel est le processus ayant le plus utilisé le processeur sur votre machine ?
> Le processus le ayant le plus utiliser le CPU est:
> ```
> USER  PID    PPID %CPU %MEM STAT  STARTED     TIME  COMMAND
> root  1485      2  0.0  0.0 I    15:37:11 00:00:02 [kworker/0:1-events]
> ```
> et ce grâce à l'option de la commande ps ```--sort=time```

- Quel a été le premier processus lancé après le démarrage du système ?
> ```
> USER         PID %CPU %MEM STAT             STARTED          TIME  COMMAND
> root           1  0.0  0.1 Ss   Fri Oct 13 07:40:51 2023 00:00:01 /sbin/init
> ```
> et ce grâce à l'option de la commande ps ```--sort=start```

- A quelle heure votre machine a-t-elle démarrée ? 
> D'après la question précédente la machine à démarrer Vendredi 13 Oct 2023 à 3 07:40:51

- Trouvez une autre commande permetant de trouver le temps depuis lequel votre serveur tourne.
> ```
> ps -eo user,pid,%cpu,%mem,stat,lstart,time,command --sort=lstart | grep ssh
> root        1492  0.0  0.1 Ss   Fri Oct 13 15:40:03 2023 00:00:00 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups
> root        1520  0.0  0.1 Ss   Fri Oct 13 15:47:42 2023 00:00:00 sshd: root@pts/1
> root        1637  0.0  0.1 Ss   Fri Oct 13 18:52:17 2023 00:00:00 sshd: root@pts/0
> root        1664  0.0  0.0 S+   Fri Oct 13 19:04:15 2023 00:00:00 grep ssh
> ```
> Donc le serveur ssh est démarré depuis le Vendredi 13 Octobre 2023 depuis 15h 40min 03s.

- Pouvez-vous établir le nombre approximatif de processus créés depuis le démarrage (“boot”) de votre machine ?
> ```
> ps -eo pid --sort=-pid | head
> PID
> 1671
> ```
Donc environ 1.671 ont été créer depuis le démarrage de la machine

#### Sous UNIX, chaque processus
- Trouver une option de la commande ps permettant d’afficher le PPID d’un processus.
> Il suffis d'ajouter "ppid" à la commande ```ps -eo```
> ```ps -eo user,pid,ppid,%cpu,%mem,stat,start,time,command```

- Donner la liste ordonnée de tous les processus ancêtres de la commande ps en cours d’exécution.
> ```
> ps -aef --forest
> root           1       0  0 07:40 ?        00:00:01 /sbin/init
> root        1492       1  0 15:40 ?        00:00:00 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 start
> root        1520    1492  0 15:47 ?        00:00:00  \_ sshd: root@pts/1
> root        1526    1520  0 15:47 pts/1    00:00:00  |   \_ -bash
> root        1637    1492  0 18:52 ?        00:00:00  \_ sshd: root@pts/0
> root        1643    1637  0 18:52 pts/0    00:00:00      \_ -bash
> root        1677    1643  0 19:16 pts/0    00:00:00          \_ ps -aef --forest
> ```

#### Reprendre la question précédente avec la commande ```pstree```.
TODO: redémarrer la machine physique pour qu'elle puisse se connecter à internet et installé ```pstree```
> ```
> 
> ```

#### Essayez la commande top
La touche ? permet d’afficher un résumé de l’aide de top. Afficher dans top la liste de processus triée par occupation mémoire (“resident memory”) décroissante.

- Quel est le processus le plus gourmand sur votre machine ? A quoi correspond-il ? (rappel : vous pouvez utiliser man truc pour découvrir ce que fait truc...).
> Le processus utilisant le plus de mémoire est "systemd"
> ```
> PID UTIL.  PR  NI    VIRT    RES    SHR S  %CPU  %MEM    TEMPS+ COM.
> 1   root   20   0  101,0m  13,2m   9,0m S   0,0   0,2   0:01.20 systemd
> ```

- Trouvez les commandes interactives permettant de : passer l’affichage en couleur, mettre en avant le colonne de trie, changer la colonne de trie.
> On change la couleur avec ```shift+z```, on met la colone de trie en évidence avec ```x``` et on change la column de trie avec ```<``` ou ```shift+<``` (```>```)

- Essayez la commande htop. Expliquez les avantages et/ou inconvénients à son utilisation par rapport à top.
>
TODO: redémarrer la machine physique pour qu'elle puisse se connecter à internet et installé ```htop```

## Arrêt d'un processus

- Lancer le 1er scripts. Le mettre en arrière plan (CTRL-Z).
- Lancer le 2eme scripts. Le mettre en arrière plan (CTRL-Z).

- A l’aide des commandes jobs fg et CTRL-C, arrêter les 2 horloges.
> Après les avoir les lancer les sciptes on peur effectuer la commande ```job -l``` afin de visualiser les jobs en cours la commande ```fg %n``` redémarre le job avec l'identifiant 'n' et la commande ```kill %n``` "tue" le jon avec l'identifiant 'n'. 

- Même question en utilisant les commandes ps et kill (avec un PID).
> Avec la commande ```ps``` on peux chercher les processus ayant lancer les scrips:
> ```
>
> root        1927  0.0  0.0 T    Fri Oct 13 20:06:45 2023 00:00:00 sh date-toto.sh
> root        1931  0.0  0.0 T    Fri Oct 13 20:06:55 2023 00:00:00 sh date.sh
> ```
> On peut ensuite les tuer avec la commande ```kill n``` n étant le pid du processus. En suite la commande ```fg``` affiche les processus comme compéter

- Expliquer les scripts à l’aide du man
>D'après les man le premier script, pour toujours, toutes les secondes va afficher heure actuelle au format H:M:s précéder de "date ". Le second script est identique à la différence qu'il affiche heure d'il y a 5 heure précéder de "toto " 

## Les tubes
Quelle est la différence entre tee et cat ?
> ```cat``` lis les fichier qui lui sont donner, mais ```tee``` les écrit. 

|Commande|Fonction|
|:-------|:-------|
|$ ls \| cat | Renvois le résultat de ```ls``` dans la sortie principale de la console, affichant les fichiers du répertoire courrant dans la console à la verticale |
|$ ls -l \| cat > liste | Écrit la liste des fichiers dans un fichier nommer "liste" |
|$ ls -l \| tee liste | Écrit la liste des fichiers dans un fichier nommer "liste" tout en affichant le résultat de la commande dans la console |
|$ ls -l \| tee liste \| wc -l | Écrit la liste des fichiers dans un fichier nommer "liste" et affiche le nombre de ligne du fichier dans la console |

## Journal système rsyslog

- Le service rsyslog est-il lancé sur votre système ? Quel est le PID du démon ?
> Non

- Le principal fichier de configuration de rsyslog est ```/etc/rsyslog.conf```. Dans quel fichier
rsyslog écrit-il les messages issus des services standards ? Et la plupart des autres messages ?
Vérifier le contenu de ces fichiers.
> Ø

- A quoi sert le service cron ?
>Cron ou Crontab est un service permettant de lancer des application de manière régulière

- Que fait la commande ```tail -f``` ? A l’aide de cette commande, placer en bas de votre écran un fenêtre qui permette de visualiser en “temps réel” le contenu du fichier ```/var/log/messages```. 
Que voyez-vous si vous redémarrez le service cron depuis un autre shell ?
> ```tail -f``` permet d'afficher les nouvelles données ajouter dans un fichier. La suite de la question n'est pas possible car le fichier n'existe pas

- Expliquer à quoi sert le fichier ```/etc/logrotate.conf```.
> c'est un fichier permettant de parametrer la création des fichier logs.

- Examiner la sortie de la commande dmesg. Quel modèle de processeur linux détecte-il sur votre machine ? Quels modèles de cartes réseaux détecte-il ?
> Processeur : "11th Gen Intel(R) Core(TM) i7-11370H @ 3.30GHz"
> ```
>  dmesg | grep Hz
> [    0.000007] tsc: Detected 2995.206 MHz processor
> [    0.301177] smpboot: CPU0: 11th Gen Intel(R) Core(TM) i7-11370H @ 3.30GHz (family: 0x6, model: 0x8c, stepping: 0x1)
> [    1.562736] e1000 0000:00:03.0 eth0: (PCI:33MHz:32-bit) 08:00:27:c1:35:f5
> ```
> Carte reseau : "Intel(R) PRO/1000 Network Connection"
> ```
>   dmesg | grep eth
> [    1.562736] e1000 0000:00:03.0 eth0: (PCI:33MHz:32-bit) 08:00:27:c1:35:f5
> [    1.562742] e1000 0000:00:03.0 eth0: Intel(R) PRO/1000 Network Connection
> [    1.609982] e1000 0000:00:03.0 enp0s3: renamed from eth0
> ```