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