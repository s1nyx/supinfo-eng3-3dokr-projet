# 3DOKR - Mini Projet

Auteurs : Anthony PRUYS, Killian VINCENT

Objectif : Conteneurisé une application web pour voter entre 2 options et afficher le résultat du vote.

## Utilisation

### Prérequis

Avant de démarrer, assurez-vous que les logiciels suivants sont installés sur votre système :

- Docker
- Docker Compose
- Docker Swarm
- Git

Ces outils sont nécessaires pour construire le projet.

### Informations sur l'Environnement

- `vote` : une application front-end qui permet aux utilisateurs de voter.
- `worker` : un service back-end qui traite les votes.
- `redis` : une base de données clé-valeur pour la mise en cache temporaire des votes.
- `postgres` : une base de données relationnelle pour le stockage persistant des votes.

### Cloner le projet

Premièrement, vous devez récupérer le projet en utilisant git grâce à la commande suivante :

```sh
git clone git@github.com:s1nyx/supinfo-eng3-3dokr-projet.git
```

### Démarrage des Services

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

### Vérification de l'état des services

Vous pouvez vérifier l'état des services en utilisant :

```sh
docker compose ps
```

### Accès aux services

Une fois les services démarrés, l'application vote sera accessible via http://localhost:8080 pour le module vote et http://localhost:8081 pour le module result.

Vous pouvez y accéder depuis une machine externe en remplaçant "localhost" par l'adresse IP de la machine ayant lancé les conteneurs.

### Arrêt des Services

```sh
docker compose down
```

## Déploiment avec un cluster Docker Swarm

### Créations des VM

Vous pouvez utilisez la méthode que vous souhaitez, tant que les VMs sont sur le même réseau et peuvent communiquer entre elles.

Pour notre configuration, nous avons utilisez VMWare. Une première VM à été créée avec Debian 11, 20 Go de stockage et 2 Go de RAM. Cette machine correspond au manager.

L'installation de docker et des autres ressources nécessaires pour l'utilisez ont été installés sur cette machine, puis elle a été clonée 2 fois pour créer les VM worker.

Chaque VM est sur le réseau NAT de VMWare et peuvent ainsi communiquer entre elles, et la machine hôte peut communiquer avec ces dernières (l'hôte a aussi une adresse IP sur ce réseau).

Les adresses IP sont attributés automatiquement en ordre croissant via DHCP. Il suffit juste de lancer la commande `ip a` pour la récupérer.


### Mise en place du cluster

On lance sur le manager la commande `docker swarm init --advertise-addr <adresse IP manager>` et on récupère le token.

Puis on rajoute les noeuds worker avec la commande `docker swarm join --token <token manager> <adresse IP manager>:2377`

Il ne reste plus qu'à lancer le stack.

## Modifications dans le code


### Module Result

Fichier server.js:

```js
const pool = new pg.Pool({
  connectionString: "postgres://postgres:postgres@localhost/postgres",
})
```

Modifié en :

```js
const pool = new pg.Pool({
  connectionString: "postgres://postgres:postgres@postgres/postgres",
})
```

### Module Worker

Fichier Program.cs :

ligne 19-20

```cs
var pgsql = OpenDbConnection("Server=localhost;Username=postgres;Password=postgres;");
var redisConn = OpenRedisConnection("localhost");

```

ligne 34 et 47

```cs
redisConn = OpenRedisConnection("localhost");
pgsql = OpenDbConnection("Server=localhost;Username=postgres;Password=postgres;");
```




Modifié en :

ligne 19-20

```cs
var pgsql = OpenDbConnection("Server=postgres;Username=postgres;Password=postgres;");
var redisConn = OpenRedisConnection("redis");
```

ligne 34 et 47

```cs
redisConn = OpenRedisConnection("redis");
pgsql = OpenDbConnection("Server=postgres;Username=postgres;Password=postgres;");
```


### Module Vote

Fichier app.py

```python
Flask.redis = Redis(host="localhost", db=0, socket_timeout=5)
```

Modifié en :

```python
Flask.redis = Redis(host="redis", db=0, socket_timeout=5)
```