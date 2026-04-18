# Infra

Despliegue con **Podman** (Compose): motor **SearXNG**, terminación TLS (**Caddy** / Hetzner) y opcionalmente **search-router**.

Este contenido se publica también como repositorio independiente: [github.com/oDevelopsOs/Infra](https://github.com/oDevelopsOs/Infra).

## Contenido

| Directorio | Descripción |
|------------|-------------|
| [searxng/](searxng/) | SearXNG + Redis (limiter) + Nginx edge. Red `investplatform_search`. |
| [hetzner/](hetzner/) | Caddy (HTTPS) para API y dominio de búsqueda; une redes con el backend y con `searxng`. |
| [search-router/](search-router/) | Router de búsqueda avanzado; el **build** requiere el módulo Go del backend (ver [search-router/README.md](search-router/README.md)). |

## Variables de entorno (tres ficheros)

| Archivo | Qué va |
|---------|--------|
| `searxng/.env.searxng` | Solo motor de búsqueda (plantilla `env.searxng.example`). |
| `hetzner/.env.caddy` | Solo dominios TLS y ACME (plantilla `env.caddy.example`). |
| Backend (fuera de este repo) | `backend/.env` en la app: `SEARXNG_URL`, Postgres, Redis app, API keys. Fragmento: `hetzner/env.backend.server.example`. |

No mezcles secretos del motor en el `.env` de la aplicación.

## Orden recomendado en servidor

1. Crear red si hace falta: `podman network create investplatform_search` (o al levantar `searxng/`).
2. `searxng/` → `podman compose -f podman-compose.yml --env-file .env.searxng up -d`
3. Clonar/compilar **backend** del monorepo principal y levantar su compose (red `investplatform_internal`).
4. `hetzner/` → Caddy con `.env.caddy`.

Detalle: [hetzner/README.md](hetzner/README.md).

## Clonar solo este repo en el VPS

```bash
git clone https://github.com/oDevelopsOs/Infra.git
cd Infra/searxng
cp env.searxng.example .env.searxng
# editar .env.searxng
podman compose -f podman-compose.yml --env-file .env.searxng up -d
```

El backend de la aplicación vive en el repositorio principal del producto; aquí solo hay infraestructura de búsqueda y proxy.
