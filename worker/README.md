# Worker


## Dockerfile

- usamos la imagen base: node:20-alpine
- en el puerto interno: 3000` (EXPOSE 3000, solo métricas)
- CMD ["node", "main.js"]

## Puerto en Compose

5002:3000 solo para ver `/metrics`

## Variables

| Variable | Valor |
|----------|-------|
| REDIS_HOST | redis |
| DATABASE_HOST | database |
| DATABASE_USER | postgres |
| DATABASE_PASSWORD | postgres |
| DATABASE_NAME | votes` |

