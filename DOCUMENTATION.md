# Mini Projet 3DOKR

## Prérequis

Avant de démarrer, assurez-vous que les logiciels suivants sont installés sur votre système :

- Docker
- Docker Compose

Ces outils sont nécessaires pour construire et exécuter les conteneurs définis dans le fichier `compose.yml`.

## Informations sur l'Environnement

- `vote` : une application front-end qui permet aux utilisateurs de voter.
- `worker` : un service back-end qui traite les votes.
- `redis` : une base de données clé-valeur pour la mise en cache temporaire des votes.
- `postgres` : une base de données relationnelle pour le stockage persistant des votes.

## Démarrage des Services

Pour démarrer l'ensemble des services, exécutez la commande suivante à la racine de votre projet :

```sh
docker compose up --build
```

Cette commande va :

1. Construire les images pour les services vote et worker si elles n'existent pas déjà.
2. Démarrer tous les services définis dans docker-compose.yml.
3. Attacher les logs des services à la console.

Si vous souhaitez exécuter les services en arrière-plan, ajoutez l'option -d :

```sh
docker compose up --build -d
```

## Vérification de l'état des services

Vous pouvez vérifier l'état des services en utilisant :

```sh
docker compose ps
```

## Accès aux services

Une fois les services démarrés, l'application vote sera accessible via http://localhost:8080

## Arrêt des Services

```sh
docker compose down
```