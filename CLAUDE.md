# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Stack

Stack Docker para flujos RAG con n8n. Un único `docker-compose.yml` define cuatro servicios en la red `postgres_network`:

| Servicio | Puerto host | Rol |
|---|---|---|
| `n8n_app` | 5678 | Motor de automatización |
| `postgres_db` | 5433 | BD principal de n8n |
| `pgvector` | 5434 | PostgreSQL + extensión vectorial |
| `pgadmin_ui` | 5050 | UI de administración |

## Comandos principales

```bash
# Levantar todos los servicios en segundo plano
docker compose up -d

# Ver estado
docker compose ps

# Ver logs de un servicio
docker compose logs -f n8n

# Detener (conserva volúmenes)
docker compose down

# Detener y eliminar volúmenes (reset completo)
docker compose down -v

# Resetear solo pgAdmin (para recargar servers.json)
docker compose stop pgadmin && docker compose rm -f pgadmin && docker compose up -d pgadmin
```

## Arquitectura RAG

El flujo RAG usa:
- **Ingestión**: n8n lee archivos desde `/home/node/.n8n-files/` (montado desde `./files/` en el host)
- **Embeddings**: `gemini-embedding-001` — 3072 dimensiones
- **Almacenamiento vectorial**: tabla `documents` en la BD `rag_db` del servicio `pgvector`

### Limitación crítica de pgvector

pgvector 0.8.x soporta índices `ivfflat`/`hnsw` solo hasta 2000 dimensiones. Con `gemini-embedding-001` (3072 dims) la tabla **funciona sin índice**. Para escalar con índice, migrar a `halfvec(3072)`.

### Esquema de la tabla `documents`

```sql
CREATE EXTENSION IF NOT EXISTS vector;  -- debe activarse por BD

CREATE TABLE documents (
  id        SERIAL PRIMARY KEY,
  content   TEXT,
  metadata  JSONB,
  embedding VECTOR(3072)
);
```

## Reglas de red Docker

Desde dentro de cualquier contenedor, usar siempre el **nombre del servicio** como host, nunca `localhost`:
- BD principal: `postgres:5432`
- pgvector: `pgvector:5432`

Desde el host Mac:
- BD principal: `localhost:5433`
- pgvector: `localhost:5434`

## Archivos accesibles desde n8n

Los nodos de n8n deben usar la ruta interna del contenedor:

```
/home/node/.n8n-files/<nombre-archivo>
```

Equivale a `./files/` en el repositorio (ignorado en git).

## pgAdmin

`pgadmin/servers.json` pre-registra ambos servidores PostgreSQL. Este archivo **solo se carga en el primer arranque** (cuando no existe volumen previo de pgAdmin). Si se modifica `servers.json`, hay que resetear el contenedor pgAdmin con el comando indicado arriba.
