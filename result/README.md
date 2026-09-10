# Result


## Dockerfile

- usamos la Imagen base: node:20-alpine
- en el puerto interno: 3000 (EXPOSE 3000)
- con CMD ["node", "main.js"]

## Puerto en Compose

5001:3000 → http://localhost:5001

## Variables

| Variable | Valor |
|----------|-------|
| DATABASE_HOST | database |
| DATABASE_USER | postgres |
| DATABASE_PASSWORD | postgres |
| DATABASE_NAME | votes |
| APP_PORT | 3000 |


