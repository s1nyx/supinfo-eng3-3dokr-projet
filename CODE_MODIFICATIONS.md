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