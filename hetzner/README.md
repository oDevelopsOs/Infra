# Despliegue en Hetzner (Podman recomendado)

Este directorio forma parte del repo **[Infra](https://github.com/oDevelopsOs/Infra)** (o de la carpeta `infra/` del monorepo principal).

## Qué incluye

- Motor SearXNG: `../searxng/podman-compose.yml` (red `investplatform_search`, Nginx `searxng-edge`).
- Stack de aplicación (otro repositorio): `backend/docker-compose.yml` (red `investplatform_internal`).
- **Caddy** (`docker-compose.caddy.yml`): HTTPS en `API_DOMAIN` (backend) y `SEARCH_DOMAIN` (SearXNG vía `searxng-edge` en la red de búsqueda).

### Archivos `.env` separados (no mezclar)

| Ruta | Contenido |
|------|-----------|
| `searxng/.env.searxng` | Solo motor de búsqueda: secret, limiter, `SEARXNG_BASE_URL`, puertos. Plantilla: `searxng/env.searxng.example`. |
| `backend/.env` (app) | App, Postgres, Redis app, Oracle, **`SEARXNG_URL`**. Fragmento: `env.backend.server.example`. |
| `hetzner/.env.caddy` | Solo dominios TLS y email ACME. Plantilla: `env.caddy.example`. |

## 1. Servidor

- Ubuntu 22.04/24.04 en Hetzner Cloud: anota la **IP pública**.
- DNS: **A** `api.tudominio.com` → IP, **A** `search.tudominio.com` → IP.
- Firewall: **22, 80, 443**. Opcional: cerrar 8080/8088/5433 al público si solo usas Caddy.

## 2. Instalar Podman

```bash
sudo apt update && sudo apt install -y ca-certificates curl git podman
```

## 3a. Solo repositorio Infra + backend aparte

```bash
sudo mkdir -p /opt && cd /opt
git clone https://github.com/oDevelopsOs/Infra.git
cd Infra/searxng
cp env.searxng.example .env.searxng
nano .env.searxng
podman compose -f podman-compose.yml --env-file .env.searxng up -d
```

Clona el backend de la aplicación en otra ruta (p. ej. `/opt/InvestPlatform`), configura `backend/.env` y levanta su compose.

## 3b. Monorepo completo (InvestPlatform)

```bash
cd /opt/investplatform
git clone <tu-repo-investplatform> .
cd backend && cp .env.example .env && nano .env
# Secretos del motor: ../infra/searxng/.env.searxng (no en backend/.env)
```

## 4. Orden: SearXNG → backend → Caddy

```bash
# Desde raíz del repo Infra (o infra/ dentro del monorepo)
cd searxng
cp env.searxng.example .env.searxng
nano .env.searxng
podman compose -f podman-compose.yml --env-file .env.searxng up -d

# Backend (ruta según tu clon)
cd /opt/InvestPlatform/backend   # ejemplo
podman compose up -d --build
# SEARXNG_URL p.ej. http://host.containers.internal:8088

cd /opt/Infra/hetzner
cp env.caddy.example .env.caddy
nano .env.caddy
podman compose -f docker-compose.caddy.yml --env-file .env.caddy up -d
```

- SearXNG: `curl -sS "http://127.0.0.1:8088/search?q=test&format=json" | head -c 200`
- API: `http://127.0.0.1:8080` (o health de tu app)

## 5. Endurecer

- Postgres/Redis no expuestos a `0.0.0.0` en prod si no hace falta.
- Rota claves; **`SEARXNG_SECRET` solo en `searxng/.env.searxng`**.
- Alto volumen de búsqueda: capa de caché / search-router según necesidad.

## Comandos útiles

```bash
cd /opt/Infra/searxng
podman compose -f podman-compose.yml --env-file .env.searxng logs -f searxng
cd ../hetzner
podman compose -f docker-compose.caddy.yml --env-file .env.caddy logs -f caddy
```
