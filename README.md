# Exercice 6 - Utilisation de Docker Compose

La documentation de Docker Compose se trouve [ici](https://docs.docker.com/reference/compose-file/). N'hésitez-pas à vous familiariser avec afin de vous y référer lorsque vous développerez vos compose files.

### 1. Création d'un premier service

- Créez un service nommé **front** avec les caractéristiques suivantes :
    - **Image** - httpd
    - **Version de l'image** - 2.4

Pour rappel, la commande **[docker compose up](https://docs.docker.com/reference/cli/docker/compose/up/)** permet de créer les ressources listées dans un compose file. N'oubliez pas de préciser le nom de votre compose file (s'il ne s'appelle pas docker-compose.yml) avec l'argument **-f**, et d'ajouter systématiquement le paramètre **-d** pour pouvoir travailler sur un unique terminal.

- Exécutez la commande **docker ps** et notez le nom du conteneur : Par défaut, Docker Compose donne le nom **{répertoire-courant}-{service}-{index}** à chaque conteneur qu'il crée.

- Exécutez maintenant la commande **docker network ls** et constatez que Docker Compose a créé un réseau nommé **{répertoire-courant}_{default}** : Lorsqu'aucun réseau custom n'est spécifié dans un compose file, Docker Compose crée un réseau custom par défaut.

### 2. Mise à jour du service

- Dans votre compose file, ajoutez un nom au conteneur du service front : **web** par exemple.

- Exécutez la commande docker-compose up et notez que Docker Compose recrée le conteneur avec le nom que vous venez de configurer.

### 3. Port-binding

- Modifiez votre compose file pour mapper le port 8080 du Docker Host avec le port 80 du conteneur du service front

- Exécutez la commande docker-compose up et notez que Docker Compose recrée le conteneur

- Exécutez la commande **docker ps** et constatez que le port-binding configuré est bien en place sur le nouveau conteneur

- Accédez au site web publié par votre conteneur via l'URL **http://localhost:8080**

### 4. Ajout d'un service

- Ajoutez un service nommé **back** avec les caractéristiques suivantes :
    - **Image** - postgres
    - **Version de l'image** - 17.2
    - **Valeur de la variable d'environnement POSTGRES_USER** - {au-choix}
    - **Valeur de la variable d'environnement POSTGRES_PASSWORD** - {au-choix}
    - **Valeur de la variable d'environnement POSTGRES_DB** - {au-choix}

- Notez que Docker Compose a ajouté le conteneur du service que vous venez de déclaré, et qu'il n'a pas touché au service front.

En effet, Docker Compose est [idempotant](https://fr.wikipedia.org/wiki/Idempotence) : l'exécution de la commande **docker compose up** n'a aucune incidence sur les ressources pour lesquelles aucun modification n'a été apportée dans le compose file.

### 5. Connectivité inter-services

- Connectez-vous au conteneur **web** et installez-y l'utilitaire **ping** via la commande `apt update && apt install -y iputils-ping`

- Exécutez la commande `ping {nom-du-conteneur-du-service-back}` et notez que le ping fonctionne

Contrairement aux conteneurs connectés au réseau **bridge** créé par Docker, les conteneurs connectés au réseau par défaut de Docker Compose peuvent se joindre en utilisant leurs noms respectifs.

- Toujours depuis le conteneur **web**, exécutez la commande `ping back` et notez que le ping fonctionne également

Dans Docker Compose, il est également possible d'atteindre un conteneur en utilisant le nom du service qui lui est associé.

- Nommez le conteneur du service back **db**

### 6. La persistence de données

- Persistez les données du répertoire **/var/lib/postgresql/data** du conteneur **db** dans un répertoire de votre Docker Host

- Notez que Docker Compose effectue une recréation du conteneur **db**

- Exécutez la commande **docker inspect** pour confirmer que le bind mounting est bien en place sur le conteneur

- Modifiez votre compose file pour que le répertoire /var/lib/postgresql/data du conteneur db soit persisté dans un volume nommé **db-data** plutôt que dans un répertoire de votre Docker Host.

- Listez les volumes et notez que Docker Compose a appelé le volume **{répertoire-courant}_{nom-volume}**

- Modifiez votre compose file pour que le nom de ce volume soit **db-data** et exécutez la commande **docker compose up -d**

Notez que :

- Docker Compose ne remplace pas le volume mais en crée un nouveau
- L'ancien volume est toujours attaché au conteneur db

Docker Compose se comporte ainsi afin d'éviter une perte de données accidentelle. Dans son fonctionnement, chaque volume est identifié par son nom, qu'il soit mentionné explicitement (nom dans le compose file) ou non (nom auto-généré). Ainsi, lorsque vous avez ajouté le nom explicite du volume dans votre compose file, Docker Compose considère que vous avez ajouté un second volume.

- Exécutez la commande **docker compose up --force-recreate back -d** pour forcer la recréation du conteneur **db** avec le nouveau volume bindé sur son répertoire /var/lib/postgresql/data

- Vérifiez que tout est conforme (docker ps, docker inspect, docker volume ls, etc...)

### 7. Le fichier de variables d'environnement

- Créez un fichier nommé **db.env** et définissez-y les variables d'environnement **POSTGRES_USER, POSTGRES_PASSWORD et POSTGRES_DB** du conteneur db

- Modifiez votre compose file pour que le conteneur db utilise le fichier db.env pour configurer ses variables d'environnement

- Utilisez la commande **docker compose up -d** et notez que Docker Compose n'effectue aucune action. En effet, aucun paramétrage n'a réèllement changé

- Modifiez la valeur de **POSTGRES_PASSWORD** dans le fichier **db.env** et re-exécutez la commande **docker compose up -d**. Notez cette fois-ci que Docker Compose recrée le conteneur

- Utilisez la commande **docker inspect** pour confirmer que la valeur de **POSTGRES_PASSWORD** a changé.


### 8. Les réseaux custom

Mettez à jour votre compose file en ajoutant les réseaux front-tier et back-tier :

- Front-tier
    - **Nom** - front-tier
    - **Driver** - default
    - **Adresse de sous-réseau** - 172.77.0.0/16

- Back-tier
    - **Nom** - back-tier
    - **Driver** - default
    - **Adresse de sous-réseau** - 172.88.0.0/16

- Exécutez la commande **docker compose up -d** et notez que Docker Compose n'effectue aucune action

- Modifiez le conteneur **web** pour qu'il soit connecté aux réseaux **front-tier** et **back-tier**

- Modifiez le conteneur **db** pour qu'il soit connecté au réseau **back-tier**

- Exécutez la commande **docker compose up -d** et constatez les actions réalisées par Docker Compose

### 9. Les replicas

- A l'aide d'un nouveau compose file, créez les ressources suivantes :

    - Réseau
        - **Nom** - front-end
        - **Adresse de sous-réseau** - 172.16.0.0/16

    - Service
        - **Nom** - api
        - **Image** - httpd
        - **Version de l'image** - 2.4.62
        - **Réseau** - front-end

- Mettez à jour le service pour qu'il contienne 10 replicas de son conteneur.

- A l'aide de la commande `docker network inspect front-end`, confirmez que les 10 replicas sont bien connectés au réseau front-tier et disposent chacun d'une adresse IP.

- Mettez à jour le service pour que le port 80 de ses replicas soit bindé au port 8080 du Docker Host.

- Vous obtenez normalement l'erreur **Une seule utilisation de chaque adresse de socket (protocole/adresse réseau/port) est habituellement autorisée**.

- Exécutez la commande `docker ps` et notez que Docker Compose n'a pu recréer qu'un conteneur : Supprimez ce conteneur.

- Mettez à jour le service pour que le port 80 de chaque replica soit bindé avec un port du Docker Host allant de 8081 à 8090.

- Notez que la même erreur est obtenue : Docker Compose ne peut pas répartir les ports d'une range fournie sur les replicas d'un service.