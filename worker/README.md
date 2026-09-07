# Worker

Proceso Node.js que lee la cola `votes` de Redis y hace UPSERT en PostgreSQL. También sirve métricas HTTP.

## Dockerfile

| | |
|---|---|
| Imagen base | `node:20-alpine` |
| Puerto interno | **3000** (métricas) |
| `EXPOSE` | `3000` |
| `CMD` | `node main.js` |

```dockerfile
CMD ["node", "main.js"]
```

Equivalente: `npm start`.

## Puerto en Compose

| Interno | Host | Uso |
|---------|------|-----|
| 3000 | 5002 | Métricas (`/metrics`, `/healthz`) |

El worker no tiene UI. El puerto se publica solo para inspección.

## Variables

| Variable | Valor en Compose |
|----------|------------------|
| `REDIS_HOST` | `redis` |
| `DATABASE_HOST` | `database` |
| `DATABASE_USER` | `postgres` |
| `DATABASE_PASSWORD` | `postgres` |
| `DATABASE_NAME` | `votes` |

## Rutas

| Ruta | Uso |
|------|-----|
| `/` | Estado del worker |
| `/metrics` | Prometheus |
| `/healthz` | Health check |

Al arrancar crea la tabla `votes` si no existe.

## Construir solo este servicio

```bash
docker build -t worker ./worker
```
