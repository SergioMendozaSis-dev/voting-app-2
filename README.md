# Primer Parcial — Computación en la Nube

**Materia:** Computación en la Nube  
**Instancia:** Primer parcial  
**Tema:** Contenerización e orquestación local de una aplicación distribuida

## Para qué sirve esta aplicación

Voting App es una votación en tiempo real (Café vs Té). Un usuario entra a una página, elige una opción y, en otra pantalla, el curso ve cómo se actualizan los totales.

Se usa para practicar **Docker** y **Compose**: no hay que escribir la lógica de negocio; hay que empaquetar cada servicio, conectarlos en una red y persistir datos con volúmenes.

## Servicios

Hay cinco servicios. Tres los construyes tú (Dockerfile) y dos salen de imágenes oficiales.

| Servicio | Qué es | Para qué se usa aquí |
|----------|--------|----------------------|
| **Vote** | Web Flask (Python) | Pantalla para votar. Recibe el click y manda el voto a Redis. |
| **Worker** | Proceso Node.js | Lee los votos de Redis y los guarda en PostgreSQL. |
| **Result** | Web Node.js | Tablero en vivo. Lee PostgreSQL y muestra los porcentajes. |
| **Redis 6** | Cola en memoria (imagen oficial) | Buffer entre Vote y Worker. Los votos no se pierden si Worker va un poco más lento. |
| **PostgreSQL 15** | Base de datos (imagen oficial) | Almacén definitivo de los votos. |

Flujo: **Vote → Redis → Worker → PostgreSQL → Result**.

Cada app (`vote`, `worker`, `result`) tiene un README con el puerto y el `CMD` del Dockerfile.

```bash
docker compose up --build
```

| Servicio | URL |
|----------|-----|
| Vote | http://localhost:8080 |
| Result | http://localhost:5001 |
| Worker (métricas) | http://localhost:5002 |

Red: `voting`. Volúmenes: `pgdata` (PostgreSQL) y `redisdata` (Redis).

---

## Estructura del repositorio

```text
.
├── vote/
│   ├── Dockerfile           ← CREAR
│   └── README.md            # puerto 80 → 8080, CMD gunicorn
├── worker/
│   ├── Dockerfile           ← CREAR
│   └── README.md            # puerto 3000 → 5002, CMD node main.js
├── result/
│   ├── Dockerfile           ← CREAR
│   └── README.md            # puerto 3000 → 5001, CMD node main.js
├── compose.yml              ← CREAR
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

### Redis 6

Misma idea que en el ejemplo de clase: imagen oficial, hostname, puerto y volumen. **Usar Redis 6** (`redis:6`). No usar `latest`. Puerto **6379**.

No hace falta Dockerfile; en Compose basta `image: redis:6`.

| | |
|---|---|
| Imagen | `redis:6` |
| Puerto interno | **6379** |
| Puerto en el host | **6379** |
| Hostname en la red | `redis` — valor de `REDIS_HOST` |
| Volumen | `redisdata` → `/data` |

Vote y Worker se conectan con `REDIS_HOST=redis`. No hay usuario ni contraseña en este parcial.

### PostgreSQL 15

Imagen oficial, usuario, contraseña, nombre de BD y volumen. **Usar PostgreSQL 15** (`postgres:15`). No usar `postgres:16` ni `latest`. Puerto **5432**.

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

El Worker crea la tabla `votes` si no existe. El volumen `pgdata` conserva los votos al hacer `docker compose down` y volver a levantar. `docker compose down -v` borra ese dato.

---

## Orden de arranque y red

- PostgreSQL y Redis arrancan primero; los demás servicios los esperan con `depends_on`.
- Todos los servicios usan la red `voting` y se resuelven por nombre (`redis`, `database`).

---

## Entregables

| Entregable | Ubicación / formato |
|------------|---------------------|
| Dockerfile Vote | `vote/Dockerfile` |
| README Vote | `vote/README.md` |
| Dockerfile Worker | `worker/Dockerfile` |
| README Worker | `worker/README.md` |
| Dockerfile Result | `result/Dockerfile` |
| README Result | `result/README.md` |
| Compose | `compose.yml` |
| Documento de evidencia | PDF o similar (ver abajo) |

### Documento (obligatorio)

Además del código, entregar **un documento** (PDF) que incluya:

1. **Capturas de la terminal de Docker** — por ejemplo `docker compose up --build`, `docker compose ps` o los logs con los cinco contenedores en ejecución.
2. **Capturas de las aplicaciones levantadas** — Vote (`http://localhost:8080`) y Result (`http://localhost:5001`), con un voto visible en el tablero.
3. **Link del repositorio** — URL del **fork** propio (no el repo original). El trabajo se entrega sobre ese fork.

El link del repo va **en el mismo documento**, junto a las capturas.

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
