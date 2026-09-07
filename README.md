# Primer Parcial — Computación en la Nube

**Materia:** Computación en la Nube  
**Instancia:** Primer parcial  
**Tema:** Contenerización e orquestación local de una aplicación distribuida

Voting App: cinco servicios orquestados con Docker Compose. Cada aplicación tiene su propio Dockerfile y un README con el puerto y el `CMD` que usa la imagen.

```bash
docker compose up --build
```

| Servicio | URL |
|----------|-----|
| Vote | http://localhost:8080 |
| Result | http://localhost:5001 |
| Worker (métricas) | http://localhost:5002 |

---

## Arquitectura

| Servicio | Rol |
|----------|-----|
| **Vote** | Web Flask. El usuario elige la opción A o B. El voto se encola en Redis. |
| **Worker** | Proceso Node.js. Lee la cola de Redis y persiste cada voto en PostgreSQL. |
| **Result** | Web Node.js. Lee PostgreSQL y publica los totales por WebSockets. |
| **Redis** | Cola de mensajes entre Vote y Worker. |
| **PostgreSQL** | Almacén de los votos. |

Flujo: **Vote → Redis → Worker → PostgreSQL → Result**.

Red: `voting`. Volúmenes: `pgdata` (PostgreSQL) y `redisdata` (Redis).

---

## Estructura del repositorio

```text
.
├── vote/
│   ├── Dockerfile
│   └── README.md            # puerto 80 → 8080, CMD gunicorn
├── worker/
│   ├── Dockerfile
│   └── README.md            # puerto 3000 → 5002, CMD node main.js
├── result/
│   ├── Dockerfile
│   └── README.md            # puerto 3000 → 5001, CMD node main.js
├── compose.yml
└── README.md
```

Build context en Compose: `./vote`, `./worker`, `./result`.

---

## Cómo levantar

```bash
docker compose up --build
```

1. Abrir http://localhost:8080 y votar.
2. Abrir http://localhost:5001 y ver el tablero actualizarse.
3. Reiniciar los contenedores: los votos deben persistir en el volumen de PostgreSQL.

```bash
docker compose down
docker compose up
```

Para borrar también los volúmenes: `docker compose down -v`.

Detalle de puertos, `CMD` y variables: [vote/README.md](vote/README.md), [worker/README.md](worker/README.md), [result/README.md](result/README.md).

---

## Servicios y variables de entorno

### Vote (Flask)

Ver [vote/README.md](vote/README.md).

| Variable | Valor |
|----------|-------|
| `REDIS_HOST` | `redis` |
| `OPTION_A` | `Café` |
| `OPTION_B` | `Té` |
| `DATABASE_HOST` | `database` |
| `DATABASE_USER` | `postgres` |
| `DATABASE_PASSWORD` | `postgres` |
| `DATABASE_NAME` | `votes` |

- Puerto interno: **80** (host **8080**; en macOS el 5000 suele estar ocupado por AirPlay).
- `CMD`: `gunicorn --bind 0.0.0.0:80 app:app`

### Worker (Node.js)

Ver [worker/README.md](worker/README.md).

| Variable | Valor |
|----------|-------|
| `REDIS_HOST` | `redis` |
| `DATABASE_HOST` | `database` |
| `DATABASE_USER` | `postgres` |
| `DATABASE_PASSWORD` | `postgres` |
| `DATABASE_NAME` | `votes` |

- Puerto interno: **3000** (host **5002**, solo métricas).
- `CMD`: `node main.js`

### Result (Node.js)

Ver [result/README.md](result/README.md).

| Variable | Valor |
|----------|-------|
| `DATABASE_HOST` | `database` |
| `DATABASE_USER` | `postgres` |
| `DATABASE_PASSWORD` | `postgres` |
| `DATABASE_NAME` | `votes` |
| `APP_PORT` | `3000` |

- Puerto interno: **3000** (host **5001**).
- `CMD`: `node main.js`

### Redis (no se vio en clase)

Redis no se trabajó en la materia. No hay que programarlo ni escribir un Dockerfile: se usa la **imagen oficial** y Compose la levanta.

Es una base en memoria. En esta app **no guarda el resultado final**: funciona como **cola** entre Vote y Worker.

1. Vote hace `RPUSH` en la lista `votes` (mete el voto al final).
2. Worker hace `LPOP` de esa misma lista (saca el voto del principio).
3. Recién ahí el Worker lo escribe en PostgreSQL.

Si Redis está caído, Vote no puede encolar y Worker no tiene de dónde leer. Por eso Vote y Worker dependen de él (`depends_on` + healthcheck).

| | |
|---|---|
| Imagen | `redis:6` (versión **6**, no `latest`) |
| Puerto interno | **6379** (el default de Redis; no se cambia) |
| Puerto en el host | **6379** |
| Hostname en la red | `redis` — ese es el valor de `REDIS_HOST` |
| Volumen | `redisdata` → `/data` |
| Dockerfile | no hace falta; `image: redis:6` alcanza |

En `compose.yml` el servicio se declara así (no copies un Dockerfile de Python/Node para Redis):

```yaml
redis:
  image: redis:6
  ports:
    - "6379:6379"
  volumes:
    - redisdata:/data
  healthcheck:
    test: ["CMD", "redis-cli", "ping"]
    interval: 5s
    timeout: 3s
    retries: 5
  networks:
    - voting
```

`redis-cli ping` debe responder `PONG`. Si el healthcheck falla, Vote y Worker no arrancan.

Documentación: [Redis Docker Hub](https://hub.docker.com/_/redis) · [Listas RPUSH / LPOP](https://redis.io/docs/latest/commands/rpush/)

### PostgreSQL 15 (sí se vio en clase)

Es la misma idea que en las prácticas: imagen oficial, usuario, contraseña, nombre de BD y volumen. **La versión de este parcial es PostgreSQL 15** (`postgres:15`). No usar `postgres:16` ni `latest`.

| | |
|---|---|
| Imagen | `postgres:15` |
| Puerto interno | **5432** |
| Puerto en el host | **5432** |
| Hostname en la red | `database` — valor de `DATABASE_HOST` |
| Volumen | `pgdata` → `/var/lib/postgresql/data` |

| Variable de la imagen | Valor | Equivale en las apps a |
|----------------------|-------|-------------------------|
| `POSTGRES_USER` | `postgres` | `DATABASE_USER` |
| `POSTGRES_PASSWORD` | `postgres` | `DATABASE_PASSWORD` |
| `POSTGRES_DB` | `votes` | `DATABASE_NAME` |

El Worker crea la tabla `votes` si no existe. Result y Vote solo se conectan; no hace falta un script SQL inicial.

El volumen `pgdata` es el que conserva los votos al hacer `docker compose down` y volver a levantar. `docker compose down -v` borra ese dato.

---

## Orden de arranque y red

- PostgreSQL y Redis arrancan primero (`depends_on` + healthcheck).
- Todos los servicios usan la red `voting` y se resuelven por nombre (`redis`, `database`).

---

## Entregables

| Entregable | Ubicación |
|------------|-----------|
| Dockerfile Vote | `vote/Dockerfile` |
| README Vote | `vote/README.md` |
| Dockerfile Worker | `worker/Dockerfile` |
| README Worker | `worker/README.md` |
| Dockerfile Result | `result/Dockerfile` |
| README Result | `result/README.md` |
| Compose | `compose.yml` |

---

## Versiones

| Componente | Versión |
|------------|---------|
| Python (Vote) | 3.11 |
| Node (Worker / Result) | 20 LTS |
| Redis | 6 |
| PostgreSQL | 15 |

---

## Referencias

- [Docker](https://docs.docker.com/)
- [Docker Compose](https://docs.docker.com/compose/)
- [Dockerfile reference](https://docs.docker.com/reference/dockerfile/)

Aplicación basada en el [Docker Example Voting App](https://github.com/dockersamples/example-voting-app), adaptada para el primer parcial de Computación en la Nube.
