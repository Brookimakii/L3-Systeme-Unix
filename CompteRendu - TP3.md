# TP3

## Exercice : paramètres
Écrivez un script analyse.sh qui affiche :

Bonjour, vous avez rentré ***nombre de paramètres*** paramètres.  
Le nom du script est ***nom du script***  
Le 3ème paramètre est ***3ème paramètre***  
Voici la liste des paramètres : ***liste des paramètres***
```sh
#!/bin/bash

echo Bonjour, vous avez rentré $# paramètres.
echo Le nom du script est $0
echo Le 3ème paramètre est $3
echo Voici la liste des paramètres: $@
```
> \# bash analyse.sh -arg1 -arg2 -arg3  
> Bonjour, vous avez rentré 3 paramètres.  
> Le nom du script est analyse.sh  
> Le 3ème paramètre est -arg3  
> Voici la liste des paramètres: -arg1 -arg2 -arg3  

## Exercice : vérification du nombre de paramètres
Écrivez un script **concat.sh** qui prend en paramètre 2 mots et fait ce qui suit.
- si l’utilisateur rentre autre chose que 2 paramètres, indique à l’utilisateur qu’il doit rentrer exactement 2 paramètres, et quitte en renvoyant une erreur.
- sinon le script calcule dans une variable **CONCAT** la concaténation des 2 mots rentrés puis affiche le résulta

Pour tous les exercices suivants vous vérifirez systématiquement le nombre de paramètres.
```sh
#!/bin/bash

if [ $# -eq 2 ]
then
  CONCAT="$1 $2"
  echo $CONCAT
  exit 1
else
  echo Please enter EXACTLY 2 arguments
fi
```
> \# sh concat.sh  
> Please enter EXACTLY 2 arguments
---
> \# sh concat.sh -arg1 -arg2  
> -arg1 -arg2

## Exercice : argument type et droits
Créer un script **test-fichier**, qui précisera le type du fichier passé en paramètre, ses permissions d’accès pour l’utilisateur, ou s’il n’existe pas.

Exemple de résultats :
> Le fichier /etc est un répertoire  
> "/etc" est accessible par root en lecture écriture exécution

> Le fichier /etc/smb.conf est un fichier ordinaire qui n’est pas vide  
> "/etc/smb.conf" est accessible par jean en lecture.
```sh
#!/bin/bash

fichier=$1

if [ -e $fichier ]; then
  result="Le fichier $fichier est"
  if [ -d $fichier ]; then
    result="$result un répertoire."
  elif [ -f $fichier ]; then
    result="$result un fichier ordinaire"
    if [ -s $fichier ]; then
      result="$result non vide."
    else
      result="$result vide."
    fi
  elif [ -L $fichier ]; then
    result="$result un lien symbolique."
  else
    result="$result d'un type inconnu."
  fi
  result="$result\n'$fichier' est accessible par $(whoami) en"
  if [ -r $fichier ]; then
    result="$result lecture"
  fi
  if [ -w $fichier ]; then
    result="$result écriture"
  fi
  if [ -x $fichier ]; then
    result="$result exécution"
  fi
  result="$result."
  printf "$result"
else
  echo "Le fichier $fichier n'existe pas."
fi
```
> \#  sh test-fichier.sh concat.sh
> Le fichier concat.sh est un fichier ordinaire non vide.
> 'concat.sh' est accessible par root en lecture écriture
> ```
> # ls -l | grep concat
> -rw-r--r-- 1 root root 119 20 oct.  14:58 concat.sh
> ```

## Exercice : Afficher le contenu d’un répertoire
Écrire un script bash listedir.sh permettant d’afficher le contenu d’un répertoire en séparant les fichiers et les (sous)répertoires.
Exemple d’utilisation :
```$ ./listdir.sh /boot```
affichera :

> ####### fichier dans /boot/  
> /boot/config-3.16.0-4-amd64  
> /boot/initrd.img-3.16.0-4-amd64  
> /boot/System.map-3.16.0-4-amd64  
> /boot/vmlinuz-3.16.0-4-amd64  
> ####### repertoires dans /boot/  
> /boot/grub  
```sh
#!bin/bash

if [ $# -ne 1 ]; then
        echo "Veuillez n'entrer qu'un seul argument."
        exit 1
fi

repertoire="$1"
if [ -d $repertoire ]; then
  echo "####### fichier dans $repertoire/"
  for fichier in "$repertoire"/*; do
    if [ -f "$fichier" ]; then
      echo "$fichier"
    fi
  done

  echo "####### répertoire dnas $repertoire/"
  for dossier in "$repertoire"/*; do
    if [ -d "$dossier" ]; then
      echo "$dossier"
    fi
  done
else
  echo "Le répertoire $repertoire n'existe pas"
fi
```
> \# sh listdir.sh /boot  
> ####### fichier dans /boot/  
> /boot/System.map-6.1.0-10-amd64  
> /boot/config-6.1.0-10-amd64  
> /boot/initrd.img-6.1.0-10-amd64  
> /boot/vmlinuz-6.1.0-10-amd64  
> ####### répertoire dnas /boot/  
> /boot/grub  

## Exercice : Lister les utilisateurs
Écrire un script bash affichant la liste des noms de login des utilisateurs définis dans **/etc/passwd**
ayant un UID supérieur à 100.
Indication : 
```bash 
for user in $(cat /etc/passwd); do echo $user; done
``` 
permet preque de parcourir les lignes du dit fichier. Cependant, quel est le problème ? Résoudre ce problème en utilisant ```cut``` (avec les bons arguments) au lieu de ```cat```. Faites la même chose avec la commande ```awk```.
> Le problème est que la ne considère pas les lignes du fichier correctement, en effet dans ce fichier, les différent champs sont séparer par des  deux-points ('**:**') 
---
> Avec La commande ```cut```
> ```sh
> for user in $(cut -d: -f1 /etc/passwd); do
>   uid=$(id -u "$user")
>   if [ "$uid" -gt 100 ]; then
>     echo "$user"
>   fi
> done
> ```
> \#  sh login-name.sh
> Liste des noms de login des utilisateurs avec un UID supérieur à 100.  
> nobody  
> systemd-network  
> systemd-timesync  
> avahi-autoipd  
> sshd  
---
> Avec La commande ```awk```
> ```sh
> awk -F: '$3 > 100 {print $1}' /etc/passwd
> ```
> \#  sh login-name.sh
> Liste des noms de login des utilisateurs avec un UID supérieur à 100.  
> nobody  
> systemd-network  
> systemd-timesync  
> avahi-autoipd  
> sshd  

## Exercice : Mon utilisateur existe t’il
Écrire un script qui vérifie si un utilisateur existe deja.
- en fonction d’un login passé en paramètres
- en fonction d’un UID passé en paramètres
Si l’utilisateur existe renvoyer son UID à l’affichage.
Sinon ne rien renvoyer.
```sh

```
> \# 

## Exercice : Creation utilisateur
Écrire un script pour créer un compte utilisateur voir : **man useradd**

Utilisez votre script de verification d’existance d’utilisateur avant de creer.

Il faudra vérifier que l’utilisateur en cours d’execution est bien root voir ***echo $USER***

Il faudra créer son home dans /home aprés avoir vérifier qu’il n’y a pas déjà un repertoire portant le même nom.

Il faudra répondre à une suite de question : voir **man read**
- login
- Nom
- Prenom
- UID
- GID
- Commentaires
```sh

```
> \# 

## Exercice : lecture au clavier
La commande bash **read** permet de lire une chaîne au clavier et de l’affecter à une variable.

Essayer les commandes suivante :
```bash
echo -n "Entrer votre nom: "
read nom
echo "Votre nom est $nom"
```
La commande file affiche des informations sur le contenu d’un fichier (elle applique des règles basées sur l’examen rapide du contenu du fichier).

Les fichiers de texte peuvent être affiché page par page avec la commande **more** (ou **less**, qui est légèrement plus sophistiquée, car *less* is *more*...).
- Question Tester les trois commandes : **read, file, more.**
  - comment quitter more ?
  - comment avancer d’une ligne ?
  - comment avancer d’une page ?
  - comment remonter d’une page ?
  - comment chercher une chaîne de caractères ? Passer à l’occurence suivante ?
Écrire un script qui propose à l’utilisateur de visualiser page par page chaque fichier texte du répertoire spécifié en argument. Le script affichera pour chaque fichier texte (et seulement ceux là, utiliser la commande file) la question “voulez vous visualiser le fichier machintruc ?”. En cas de réponse positive, il lancera more, avant de passer à l’examen du fichier suivant.
```sh

```
> \# 

## Exercice : appréciation
Créer un script qui demande à l’utilisateur de saisir une note et qui affiche un message en fonction de cette note :
- “très bien” si la note est entre 16 et 20 ;
- “bien” lorsqu’elle est entre 14 et 16 ;
- “assez bien” si la note est entre 12 et 14 ;
- “moyen” si la note est entre 10 et 12 ;
- “insuffisant” si la note est inférieur à 10.
Pour quitter le programme l’utiliateur devra appuyer sur q
```sh

```
> \# 