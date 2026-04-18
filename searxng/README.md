# Stack SearXNG (Podman)

Motor de búsqueda agregada **separado** del backend. Usa **Podman Compose**.

## Archivo de entorno (solo este stack)

| Archivo | Uso |
|---------|-----|
| `env.searxng.example` | Plantilla en el repo (sin secretos reales). |
| **`.env.searxng`** | Copia local en el servidor: `cp env.searxng.example .env.searxng` y edita. **No** mezclar con `backend/.env`. |

Variables que van **solo** aquí: `SEARXNG_PUBLISH`, `SEARXNG_EDGE_PUBLISH`, `SEARXNG_BASE_URL`, `SEARXNG_SECRET`, `SEARXNG_LIMITER`, `SEARXNG_VALKEY_URL`.

El **backend** solo necesita en su `.env` la URL para llamar al motor: `SEARXNG_URL` (y opcionalmente nada más del motor).

## Arranque

La red **`investplatform_search`** debe existir antes (compartida con Caddy). Créala una vez:

```bash
podman network inspect investplatform_search >/dev/null 2>&1 || podman network create investplatform_search
```

Luego:

```bash
cd searxng
cp env.searxng.example .env.searxng
# Edita .env.searxng (SEARXNG_SECRET, SEARXNG_BASE_URL con tu dominio HTTPS en prod)
podman compose -p searxng -f podman-compose.yml --env-file .env.searxng up -d
```

Comprueba: `curl -sS "http://127.0.0.1:8088/search?q=test&format=json" | head -c 200`

## Red `investplatform_search`

El compose **no** define la red (evita la clave `name:` incompatible con `docker-compose` v1). Créala con el comando de la sección Arranque. [hetzner](../hetzner/) (Caddy) usa la misma red para `reverse_proxy searxng-edge:80`.

Orden en servidor (Hetzner): **1)** `searxng/` → **2)** backend (otro repo) → **3)** `hetzner/` (Caddy). Detalle: [../hetzner/README.md](../hetzner/README.md).

## Variables del backend (referencia)

- Host: `SEARXNG_URL=http://127.0.0.1:8088` o `http://127.0.0.1:8099` (edge).
- Contenedor backend sin red compartida: `SEARXNG_URL=http://host.containers.internal:8088` (Podman).
- `SEARXNG_BASE_URL` público se configura en **`.env.searxng`**, no en el backend.

## Producción y alto volumen

- Limiter + `searxng-redis`: valores por defecto en `env.searxng.example`.
- Motores upstream limitan abuso; escala en [search-router](../search-router/) (opcional; build requiere backend Go).

## SELinux (Fedora/RHEL)

Si hace falta, añade `:Z` a los bind mounts en `podman-compose.yml`.
