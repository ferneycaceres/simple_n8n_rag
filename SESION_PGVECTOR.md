# Sesión: Configuración de pgvector con Docker

**Fecha:** 2026-06-05

---

## Resumen

Configuración completa de pgvector (extensión de vectores para PostgreSQL) usando Docker, con conexión desde pgAdmin también en Docker.

---

## Pasos realizados

### 1. Descarga de la imagen Docker
```bash
docker pull pgvector/pgvector:pg17
```

### 2. Levantar el contenedor
El puerto 5432 y 5433 estaban ocupados, se usó el **5434**.

```bash
docker run -d \
  --name pgvector \
  -e POSTGRES_PASSWORD=postgres \
  -e POSTGRES_DB=postgres \
  -p 5434:5432 \
  pgvector/pgvector:pg17
```

### 3. Conexión desde pgAdmin
pgAdmin también corre en Docker (`dpage/pgadmin4`), por lo que **no se puede usar `localhost`**.

**Configuración correcta en pgAdmin:**

| Campo    | Valor                    |
|----------|--------------------------|
| Host     | `host.docker.internal`   |
| Port     | `5434`                   |
| Username | `postgres`               |
| Password | `postgres`               |
| Database | `postgres`               |

### 4. Activar la extensión vector
La extensión debe activarse **dentro de cada base de datos** donde se vaya a usar.

```sql
CREATE EXTENSION IF NOT EXISTS vector;
```

### 5. Crear tabla con tipo VECTOR
```sql
CREATE TABLE IF NOT EXISTS documents (
  id       SERIAL PRIMARY KEY,
  content  TEXT,
  metadata JSONB,
  embedding VECTOR(1536)
);
```

### 6. Listar tablas
```sql
SELECT table_name FROM information_schema.tables
WHERE table_schema = 'public';
```

---

## Problemas encontrados y soluciones

| Problema | Causa | Solución |
|----------|-------|----------|
| Puerto 5432/5433 ocupado | Otro proceso usando el puerto | Usar puerto 5434 |
| Conexión rechazada en pgAdmin | pgAdmin corre en Docker; `localhost` apunta al propio contenedor | Usar `host.docker.internal` |
| Error tipo VECTOR al crear tabla | Extensión `vector` no activada | Ejecutar `CREATE EXTENSION IF NOT EXISTS vector;` primero |

---

## Datos de conexión

```
Host:     localhost (o host.docker.internal desde Docker)
Port:     5434
User:     postgres
Password: postgres
Database: postgres
```
