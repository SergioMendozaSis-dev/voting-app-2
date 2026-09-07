# Result

Servicio Node.js que lee los votos de PostgreSQL y los muestra en tiempo real con Socket.IO. La carpeta `views/` debe ir en la imagen.

## Dockerfile

| | |
|---|---|
| Imagen base | `node:20-alpine` |
| Puerto interno | **3000** (`APP_PORT`) |
| `EXPOSE` | `3000` |
| `CMD` | `node main.js` |

```dockerfile
CMD ["node", "main.js"]
```

Equivalente: `npm start`.

## Puerto en Compose

| Interno | Host | URL |
|---------|------|-----|
| 3000 | 5001 | http://localhost:5001 |

## Variables

| Variable | Valor en Compose |
|----------|------------------|
| `DATABASE_HOST` | `database` |
| `DATABASE_USER` | `postgres` |
| `DATABASE_PASSWORD` | `postgres` |
| `DATABASE_NAME` | `votes` |
| `APP_PORT` | `3000` |

## Rutas

| Ruta | Uso |
|------|-----|
| `/` | Tablero de resultados |
| `/metrics` | Prometheus |
| `/healthz` | Health check |

## Construir solo este servicio

```bash
docker build -t result ./result
```
