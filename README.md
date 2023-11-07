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

Vous pouvez utiliser la méthode que vous souhaitez, tant que les VMs sont sur le même réseau et peuvent communiquer entre elles.

Pour notre configuration, nous avons utilisé VMWare. Une première VM a été créée avec Debian 11, 20 Go de stockage et 2 Go de RAM. Cette machine correspond au manager.

L'installation de docker et des autres ressources nécessaires pour l'utiliser ont été installés sur cette machine, puis elle a été clonée 2 fois pour créer les VM worker.

Chaque VM est sur le réseau NAT de VMWare et peuvent ainsi communiquer entre elles, et la machine hôte peut communiquer avec ces dernières (l'hôte a aussi une adresse IP sur ce réseau).

Les adresses IP sont attribuées automatiquement en ordre croissant via DHCP. Il suffit juste de lancer la commande `ip a` pour la récupérer.

Pensez à changer les hostnames de chaque machine (dans le fichier /etc/hostname et /etc/hosts) pour pouvoir les différencier sur le manager.

### Mise en place du cluster

On lance sur le manager la commande 
```sh
docker swarm init --advertise-addr <adresse IP manager>
``` 

ex:
```sh
docker swarm init --advertise-addr 192.168.99.128
```

Dans le résultat, copiez la commande affichée. Elle nous permettra de rajouter les noeuds workers au cluster.

Sur les noeuds worker, lancez la commande copiée.

```sh
docker swarm join --token <token manager> <adresse IP manager>:2377
```

ex:
```sh
docker swarm join --token \
SWMTKN-1-06v5wj8agvqv0u2qc3i24wiwszgfq9ktr8xlzo87r6pc9b91am-8n7w48r076h7qz73e1m1rlf5k \
192.168.99.128:2377
```

Une fois que tous les noeuds sont rajoutez, utilisez la commande :
```sh
docker node ls
```
résultat (ID généré aléatoirement):
```
ID                            HOSTNAME   STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
5kmwqq4hmc0q78um6rithyouf *   master1    Ready     Active         Leader           24.0.6
xu6kydrbqvjkgkt10nkbu6y5j     worker     Ready     Active                          24.0.6
z9uh4xwjso6ang309f9b3eb7o     worker2    Ready     Active                          24.0.6
```

Il ne reste plus qu'à lancer le stack.

### Déploiment du stack

Cloner le projet git sur le manager :

```sh
git clone git@github.com:s1nyx/supinfo-eng3-3dokr-projet.git
```

Puis aller dans le dossier racine du projet et lancer la commande suivante :

```sh
docker stack deploy --compose-file compose-swarm.yaml dokrproject
```

Si le cluster est correctement déployé, vous pourrez voir le statut de chaque service avec la commande suivante :

```sh
docker service ls
```

```
ID             NAME                   MODE         REPLICAS               IMAGE                            PORTS
uhnue2c4t9ab   dokrproject_postgres   replicated   1/1                    postgres:16.0-alpine
5w08f1rrpeaw   dokrproject_redis      replicated   1/1                    redis:7.2.3-alpine
h8fowce1r2ks   dokrproject_result     replicated   2/2 (max 1 per node)   supinfoant/3dokrproject:result   *:8081->8888/tcp
nasdofl5wz4g   dokrproject_vote       replicated   2/2 (max 1 per node)   supinfoant/3dokrproject:vote     *:8080->8080/tcp
q4ozhbz1l4ef   dokrproject_worker     replicated   1/1                    supinfoant/3dokrproject:worker
```

Les services peuvent prendre un peu de temps avant d'avoir lancé tous les conteneurs, donc le nombre pour les "replicas" peut être différent pendant le lancement.

Une fois que tous les services sont prêts, vous pourrez accéder aux pages web en utilisant l'adresse IP d'un des noeuds (peu importe lequel) et le port associé :
`http://<ip>:8080` pour voter et `http://<ip>:8081` pour le résultat.

ex: `http://192.168.99.128:8080` ou `http://192.168.99.129:8080` pour votez et `http://192.168.99.128:8081` ou `http://192.168.99.129:8081` pour le résultat.

Si un des workers quitte le cluster, le site web restera toujours accessible sur les adresses IP des noeuds toujours présents dans le cluster.

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