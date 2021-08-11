# Docker_commandes
commandes docker 


<hr>

Supprimer un conteneur en cours d'exécution<br>
`docker container rm -f redis`<br>

Supprimer tous les conteneurs arrêtés / les images non utilisées (dangling)<br>
`docker container prune `<br>
`docker image prune`<br>

Tout supprimer (containers qui ne sont pas en cours d'exécution, images dangling, cache, réseaux non utilisés)<br>
`docker system prune`<br>

Renommer un conteneur<br>
`docker container rename <container> nouveau_nom`<br>

Mettre en pause et reprendre les processus<br>
`docker container pause`<br>
`docker container unpause`<br>

Exécuter des commandes dans des conteneurs en cours d'exécution<br>
`docker container exec`<br>exemples :<br>
```
docker run -d --name redis redis
docker exec -it redis redis-cli
set cle 42 #commande redis qui fonctionne
get cle
```
```docker
docker container exec <container> mkdir /app 
docker container exec <container> /app/fichier.txt #ajout d'un dossier/fichier dnas le container
```
