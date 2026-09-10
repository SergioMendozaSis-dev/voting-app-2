# Vote


## Dockerfile

- usamos Imagen base: python:3.11-slim
- en el puerto interno: 80 (EXPOSE 80)
- CMD ["gunicorn", "--bind", "0.0.0.0:80", "app:app"]

## Puerto en Compose

8080:80 → http://localhost:8080

## Variables

| Variable | Valor |
|----------|-------|
| REDIS_HOST | redis |
| OPTION_A | Café |
| OPTION_B | Té |
| DATABASE_HOST | database|
| DATABASE_USER | postgres |
| DATABASE_PASSWORD | postgre |
| DATABASE_NAME | votes |
