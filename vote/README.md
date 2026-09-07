# Vote

Servicio Flask para emitir votos (opción A o B). Encola cada voto en Redis y expone métricas desde PostgreSQL.

## Dockerfile

| | |
|---|---|
| Imagen base | `python:3.11-slim` |
| Puerto interno | **80** |
| `EXPOSE` | `80` |
| `CMD` | `gunicorn --bind 0.0.0.0:80 app:app` |

```dockerfile
CMD ["gunicorn", "--bind", "0.0.0.0:80", "app:app"]
```

## Puerto en Compose

| Interno | Host | URL |
|---------|------|-----|
| 80 | 8080 | http://localhost:8080 |

## Variables

| Variable | Valor en Compose |
|----------|------------------|
| `REDIS_HOST` | `redis` |
| `OPTION_A` | `Café` |
| `OPTION_B` | `Té` |
| `DATABASE_HOST` | `database` |
| `DATABASE_USER` | `postgres` |
| `DATABASE_PASSWORD` | `postgres` |
| `DATABASE_NAME` | `votes` |

## Rutas

| Ruta | Uso |
|------|-----|
| `/` | Formulario de votación |
| `/stats` | Totales en JSON |
| `/metrics` | Prometheus |
| `/healthz` | Health check |

## Construir solo este servicio

```bash
docker build -t vote ./vote
```
