# Docker_commandes
commandes docker 


<hr>
Renommer un conteneur<br>
`docker container rename CONTENEUR NOUVEAU_NOM`

Mettre en pause et reprendre les processus<br>
`docker container pause`<br>
`docker container unpause`<br>

Exécuter des commandes dans des conteneurs en cours d'exécution<br>
`docker container exec`<br>exemple :<br>
```
docker run -d --name redis redis
docker exec -it redis redis-cli
set cle 42 #commande redis qui fonctionne
get cle
```
