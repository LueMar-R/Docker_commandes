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
docker run -d --name redis redis
docker exec -it redis redis-cli
set cle 42 #commande redis qui fonctionne
get cle
```
```shell
docker container exec conteneur1 mkdir /app 
docker container exec conteneur1 /app/fichier.txt #ajout d'un dossier/fichier dnas le container
```
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
Pour donner un nom à la construction, ajouter l'option tag<br>
`docker build -t lenom:latest .`<br>



__RUN__<br>
il faut utiliser le package manager de l'image utilisée pour installer les différentes library. Sur Ubuntu par exemple, on utilise `apt` (ex : `apt-get update && apt-get -y dist-upgrade`), sur alpine `apk` (ex : `apk add --update nodejs`), etc.

__COPY__<br>
`COPY ./fromhost ./tocontainer 
