# Commandes Docker



<hr>

## Bases

Supprimer un conteneur en cours d'exécution<br>
`docker container rm -f CONTENEUR`<br>

Supprimer tous les conteneurs arrêtés / les images non utilisées (dangling)<br>
`docker container prune `<br>
`docker image prune`<br>

Tout supprimer (containers qui ne sont pas en cours d'exécution, images dangling, cache, réseaux non utilisés)<br>
`docker system prune`<br>

Renommer un conteneur<br>
`docker container rename CONTENEUR nouveau_nom`<br>

Mettre en pause et reprendre les processus<br>
`docker container pause`<br>
`docker container unpause`<br>

Exécuter des commandes dans des conteneurs en cours d'exécution<br>
`docker container exec`<br>exemples :<br>
```shell
docker run -d --name appnode -p 80:80 mynode
docker exec -it appnode sh
/ ls #on est à l'intérieur du container
```
```shell
docker run -d --name redis redis
docker exec -it redis redis-cli
/ set cle 42 #commande redis qui fonctionne
/ get cle
```
`docker container exec conteneur1 mkdir /app `
`docker container exec conteneur1 /app/fichier.txt #ajout d'un dossier/fichier dnas le container`

Attacher un terminal (STDIN, STDOUT et STDERR)<br>
`docker start -ai CONTENEUR` (-a ou --attach pour STDOUT et STDERR, -i pour --interactive pour attacher également l'entére standard STDIN)<br>
cela revient à faire :<br>
`docker start CONTENEUR`<br>
`docker attach CONTENEUR`<br>

Copier dans le sens hôte -> conteneur<br>
`docker cp CHEMIN_SOURCE CONTENEUR:CHEMIN_DESTINATION`<br>
`docker cp ./fromhost conteneur1:/app`<br>
Copier dans le sens conteneur -> hôte<br>
`docker cp CONTENEUR:CHEMIN_SOURCE CHEMIN_DESTINATION `<br>
`docker cp conteneur1:./app/log.txt .`<br>

Inspecter les changements dans le système de fichiers d'un conteneur<br>
`docker container diff CONTENEUR`<br>

Inspecter les processus dans un conteneur en cours d'exécution (du point de vue de la machine hôte)<br>
`docker container top CONTENEUR`<br>
Pour se placer du point de vue du conteneur (exemple:)<br>
```shell
docker exec -it redis bash      # Pour obtenir un shell bash
apt update && apt install -y procps    # Pour pouvoir utiliser ps pour lister les processus
ps -ef
```

Obtenir toutes les options de configuration d'un conteneur<br>
`docker container inspect CONTENEUR`<br>

Visualiser l'utilisation des ressources par les conteneurs
`docker container stats`

Visualiser l'espace disque utilisé
`docker system df`
`docker system df -v`

## Image, Dockerfile

Construire une image<br>
`docker image build CHEMIN`<br>
ex : `docker image build .` va chercher le Dockerfile dnas le dossier courrant pour construire l'image.<br>
Pour donner un nom au moment de la construction, ajouter l'option tag :<br>
`docker build -t myimage:latest .`<br>
Pour ne pas utiliser de cache :<br>
`docker build --no-cache -t monimage:latest . `<br>

Seules les instructions `RUN`, `COPY` et `ADD` créent des nouvelles couches et augmentent la taille d'une image. Toutes les autres instructions ne font que créer des images intermédiaires temporaires et n'augmentent donc pas la taille de l'image finale.<br>
Il est obligatoire de mettre `apt-get update` et `apt-get install` dans la même instruction `RUN`.

Inspecter une image<br>
`docker image inspect monimage`<br>
Cela permet d'obtenir tous les hashs des couches de l'image, et d'accéder aux variables d'environnement qui ont été définies (entre autres).



__RUN__<br>
Il existe deux syntaxes pour l'instruction RUN, la forme `shell` et la forme `exec`.<br>
Forme exec : `RUN ["executable", "param1", "param2"]` par exemple `RUN ["/bin/bash", "-c", "echo Bonjour !"]` (guillemets doubles obligatoires)<br>
Forme shell : `RUN echo "Bonjour !"` (revient en fait à faire `/bin/sh -c`)<br>

Pour les installations, il faut utiliser le package manager de l'image utilisée pour installer les différentes library. <br>Sur Ubuntu par exemple, on utilise `apt` (ex : `apt-get update && apt-get -y dist-upgrade`), sur alpine `apk` (ex : `apk add --update nodejs`), etc.

__COPY__ et __ADD__<br>
`COPY ./source.txt ./destination/` <br>
`ADD source.txt ./destination/` <br>
Pour ADD (contrairement à COPY) la source peut être une URL ou un dossier compressé (qui sera automatiquement décompressé).<br>
Pour que Docker considère que la destination est un dossier il faut obligatoirement finir par `/`.

Patterns :<br>
`ADD package* /app/` copiera tous les fichiers commençant par package dans le dossier /app/ du conteneur.<br>

__WORKDIR__<br>
pour se positioner dans le container, modifier le répertoire de travail pour toutes les instructions qui suivront
`WORKDIR /dossier`

Possibilité d'utiliser des variables d'environnement :
```shell
ENV DIR=/app
WORKDIR $DIR/back
RUN pwd
```

__CMD__ et __ENTRYPOINT__ <br>
`CMD ["executable", "param1", "param2"]` <br>
`CMD ["python", "/app/app.py"]/` <br>

`ENTRYPOINT ["python", "/app/app.py"]` (forme exec, la plus courante)<br>
`ENTRYPOINT python "/app/app.js` (forme shell)<br>
ENTRYPOINT fait la même chopse que CMD, mais verrouille les instruction d'exécution.<br>
On peut éventuellemnet combiner les deux : <br>
```
ENTRYPOINT ["echo"]
CMD ["Hello"]
```
cela affichera Hello si aucune commande alternative n'est passée, ou bien le texte alternatif si on fait un `docker run test "textealternatif"`

__ARG__<br>
Permet de définir des variables qui seront utilisables par l'utilisateur lançant les conteneurs. S'utilise avec `--build-arg` au moment du build. Exemple :<br>
`ARG env`(dans le dockerfile)<br>
`docker build --build-arg env=prod`<br>

__ENV__<br>
Permet de définir des variables d'environnement.<br>
`ENV CLE1="Une valeur1" CLE2="Une valeur2"`<br>
La différence avec ARG est que les variables d'environnement sont persistées dans l'image après le build.

__LABEL__<br>
Permet d'ajouter des métadonnées à une image.
`LABEL auteur="moi@monmail.com" version="5.0.1"`

__EXPOSE__<br> 
Permet d'informer Docker qu'un conteneur écoute sur les ports spécifiés lors de son exécution. L'instruction ne publie pas le port spécifié. Il faut toujours passer l'option `-p`.
<br>
`EXPOSE 80`<br> écoute en tcp (par défaut). pour préciser udp il faut faire
`EXPOSE 80/udp`<br>

## Partager une image 

### Sur Docker Hub

Se connecter à / déconnecter de Docker Hub<br>
`docker login`<br>
`docker logout`<br>
Publier sur Docker Hub<br>
`docker image push monlogin/monapp:1.0`<br>
Télécharger localement l'image<br>
`docker image pull monlogin/monapp:1.0`<br>

(voir comment [crypter ses identifiants linux](https://doc.ubuntu-fr.org/gnupg))

### Hors Docker Hub

Exporter une image<br>
`docker image save [-o monimage.tar] monimage`<br>
On aussi compresser l'archive<br>
`docker save mon_image | gzip > mon_image.tar.gz`<br>

Charger une image au format .tar<br>
`docker image load < monimage.tar` ou<br>
`docker image load -i mon_image.tar`

## Exemples 

### Serveur node
app.js
```js
const express = require('express');
const app = express();
app.get('*', (req, res) => {
    res.status(200).json('Hello le World!');
})
app.listen(80);
```
package.json
```json
{
    "dependencies": {
        "nodemon" : "^2.0.6",
        "express" : "^4.17.1"
    }
}
```
Dockerfile
```Dockerfile
FROM node:alpine
WORKDIR /app
COPY ./package.json . 
RUN npm install
COPY . .
ENV PATH=$PATH:/app/node_modules/.bin 
CMD [ "nodemon", "/app/app.js" ]
```
- on ajoute node_modules (où se trouve npm) au path pour que la commande 'nodemon' puisse s'exécuter
- on copie package.json avant le npm install, mais on copie le reste des fichiers *après* pour éviter de refaire le npm install à chaque modificaton des fichiers (optimisation)

Pour lancer l'application :<br>
`docker build -t myapp .`<br>
`docker run -d --name appnode -p 80:80 myapp`<br>
(affiche "hello world sur localhost:80)


## Volumes, Bind-Mount, TMPFS (Persistance)
Par défaut, les fichiers d'un conteneur sont écrits dans une couche avec les droits d'écritures, mais cette couche est supprimée si le conteneur n'existe plus.

Les différentes solutions : [Manage data in Docker (official)](https://docs.docker.com/storage/)<br>
En résumé:<br>

Bind mount : il s'agit de fichiers et de dossiers n'importe où sur le système de fichiers de l'hôte. N'importe quel processus peut les modifier, y compris en dehors de Docker. Il faut préciser le chemin absolu du fichier ou du dossier à monter. Il peut être n'importe où sur l'hôte. Si le fichier ou le dossier n'existe pas sur l'hôte, il sera créé par Docker à l'emplacement indiqué.<br>On préfère généralement l'utilisation d'un volume à un Bind-Mount, sauf dans le cas d'un environnement de développement ou il est utile pour le live reload notamment.

Volume : Docker utilise le système de fichiers de la machine hôte mais gère lui même cet espace et l'utilisateur doit passer par le Docker CLI. Lorsque l'on crée un volume il est stocké dans un dossier sur l'hôte (/var/lib/docker/volumes/ pour GNU/Linux... ne jamais y toucher directement!). les volumes sont entièrement gérés par Docker et sont isolés de la machine hôte. Un volume peut être utilisé par plusieurs conteneurs.<br> 

TMPFS : il s'agit d'un stockage temporaire en mémoire vive (RAM). Il permet de stocker temporairement des données d'état ou des informations sensibles.

<br>

__Bind-mount__<br>
`docker run --mount type=bind,source=<fromhost>,target=<tocontainer> monimage`exemple :<br> 
`docker run --mount type=bind,source="$(pwd)"/data,target=/data alpine` ou en mode interactif :<br>
`docker run -it --mount type=bind,source="$(pwd)"/data,target=/data alpine sh`<br>
Tous les fichiers créés ou supprimés ensuite dans le dossier data sur l'hôte le seront également dans le dossier data du container, et réciproquement. La suppression du container n'entraîne pas la suppression du dossier data et de son contenu sur l'hôte.


__Volume__<br>
Créer et inspecter un volume<br>
`docker volume create mydata`<br>
`docker volume inspect mydata`<br>
Lancer un container avec ce volume (exemple avec une image alpine)<br>
`docker run --mount type=volume,source=mydata,target=/data alpine`ou en mode interactif :<br>
`docker run --mount type=volume,source=mydata,target=/data -it alpine sh`
Supprimer un volume<br>
`docker volume rm mydata`<br>
`docker volume prune` pour les supprimer tous<br>

Partager des volumes entre plusieurs conteneurs<br>
`docker container run -it --rm --volumes-from conteneur1 alpine sh`<br>

Effectuer la sauvegarde du contenu d'un volume (Backup)<br>
`docker run --mount type=volume,source=mydata,target=/data --mount type=bind,source="$(pwd)",target=/backup alpine tar -czf backup/mydata.tar.gz data` <br>
*on écrase la commande de base en la remplaçant par la commande tar. le dernier "data" fait réference au dossier "data" du container où on été partagée les données du volume mydata lors du premier --mount*<br>
`docker container run --rm --volumes-from conteneur1 --mount type=bind,src="$(pwd)",target=/backup alpine tar -cf /backup/backup.tar /data` (l'option --rm supprime le conteneur une fois que celui-ci est stoppé)<br>
Restaurer cette sauvegarde à l'intérieur d'un nouveau volume<br>
`docker volume create restore` on crée le volume "restore"<br>
`docker run --mount type=volume,source=restore,target=/data --mount type=bind,source="$(pwd)",target=/backup -it alpine tar -xf /backup/mydata.tar.gz -C /data` (-C permet de positionner le fichier) <br>

Persister une base de données avec un volume<br>
exemple avec mongodb :<br>
`docker container run -d --name mongodb --mount source=mydb,target=/data/db mongo`<br>
Comme le volume mydb n'existe pas, il sera automatiquement créé par Docker. Il faut absolument le monter dans /data/db et pas autre part car c'est ce chemin qui est utilisé par le démon mongod.<br>
A noter qu'il est très simpled'utiliser mongo avec Compass et Docker.<br>
`docker container run -d --name mongodb --mount source=mydb,target=/data/db -p 27018:27017 mongo`
ensuite on peut entrer dnas compass : `mongodb://localhost:27018`

## Réseaux

Drivers :<br>
Le __bridge__ est le driver par défaut pour les réseaux Docker. Pour communiquer avec un container sur le réseau bridge par défaut, il faut nécessairement son adresse IP.<br>
Le __host__ est le driver permettant de supprimer l'isolation réseau d'un ou plusieurs conteneurs avec l'hôte (comme si les conteneurs tournaient directement sur l'hôte - uniquement pour le réseau, Par pour l'isolation des processus, fichiers...).<br>
L'__overlay__ est le driver permettant de connecter plusieurs démons Docker (utilisé avec swarm). Pour faire communiquer des conteneurs qui sont situés sur des hôtes différents il faut utiliser un réseau overlay.

Inspecter les réseaux<br>
`docker network inspect bridge`<br>


### Bridge
On peut faire communiquer des containers entre eux par leur nom (et non leur IP, plus efficace dnas la mesure où l'on ne connaît pas à l'avance les IP qui sont attribuées), en les nommant à la création et en les dirigeant sur un réseau particulier avec l'option --network. La résolution DNS des noms des conteneurs en IP est automatique sur un réseau bridge personnalisé, contairement au réseau bridge par défaut.
```shell
docker network create mynet
docker run --network mynet --name server1 -d alpine ping google.fr # ce container est en train de pinger google
docker run --network mynet --name server2 -it alpine sh
/ # ping server1 # on peut pinger le container server1 depuis le container serveur2 : ils communiquent
```
`docker exec -it server1 ping server2` fonctionne également

Connecter / déconnecter un container à un réseau existant<br>
`docker network connect network1 conteneur1`<br>
`docker network disconnect bridge conteneur1`<br> (ici déco du réseau par défaut)

Avec le réseau "host" le réseau du conteneur ne sera pas isolé de l'hôte. Le conteneur et l'hôte auront les mêmes interfaces réseaux. Le conteneur n'aura donc par exemple pas sa propre adresse IP.<br>
Par ailleurs il n'y a pas besoin de publier les ports (ce n'est même pas possible) car ce sont les ports de l'hôte.<br>
`docker container run --rm -d --network host --name conteneur1 monimage`<br>

## Docker-compose

exemple<br>
fichier .yml
```yaml
version: "3.8"
services:
myalpine:
  image: alpine
```
On peut faire :<br>
`docker-compose up`(lancer tous les services du docker-compose)<br>
`docker-compose run myalpine`(exécuter un service)<br>
ou pour faire exécuter une commande par un service :<br>
`docker-compose run myalpine ls` (remplace la commande par défaut bin/sh par ls)<br>

Stopper les conteneurs lancés avec docker-compose<br>
`docker-compose down `<br>
pour suuprimer également les volumes créés avec docker-compose<br>
`docker-compose down -v`<br>

La commande 'docker-compose build' permet de construire toutes les images spécifiées dans le fichier de configuration (dans les configurations build) et de les taguer pour une future utilisation.<br>
`docker-compose build`<br>

Exemple de ce que peut contenir le docker-compose.yml :<br>
```yaml
version: '3.8' # toujours spécifier la version
services:
    a:          # container à partir d'une image, RAS
      image: alpine
      command: ['ls']
    b:          # container construit à partir d'une image personnalisée
      build:
        context: ./backend      # le dockerfile pour build l'image se trouve dans le dossier 'backend'
        dockerfile: Dockerfile
        args:
          - FOLDER=test         # passage d'arguments attendus par le dockerfile
        labels:
          - email=jean@gmail.com    # def de label (inspect b)
      ports:
        - "8081:80"             # publication des ports
      volumes:
        - type: bind            # un bind ne nécessite pas d'être déclaré dans la partie volumes
          source: ./data
          target: /app/data
        - type: volume          # pas de source : création automatique d'un volume anomyme
          target: /app/data2
        - type: volume          # volume nommé : déclaration nécessaire 
          source: data3
          target: /app/data3

volumes:
    data3:                      # si data3 n'existe pas, il sera créé par docker compose...
      external: true            # ...à moins qu'on ait forcé avec external:true              
```
En développement, plutôt que de faire des `docker-compose up` il est souvent pratique d'utiliser le `docker-compose build` suivi des `docker-compose run` pour les différentes images buildées.

<br>
``<br>
