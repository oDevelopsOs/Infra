# Search Router (opcional)

Compose y nginx para el servicio `search-router` (Go). La **imagen se construye desde el módulo backend** del monorepo principal (no está incluido en el repo Infra).

## Build

Por defecto, `podman-compose.yml` usa:

```yaml
context: ../../backend
```

Eso es válido si la estructura es:

```text
InvestPlatform/
  backend/
  infra/search-router/   # o en repo Infra aislado, coloca una copia/symlink de backend
```

### Repo Infra solo (este clon)

Opciones:

1. Clona también el repo de la aplicación y apunta el contexto al `backend/`:

   ```bash
   export SEARCH_ROUTER_BUILD_CONTEXT=/opt/InvestPlatform/backend
   ```

   y en un `docker-compose.override.yml` local (no commiteado) sobrescribe `build.context`.

2. Crea un symlink: `ln -s /ruta/a/InvestPlatform/backend /opt/Infra/backend` y usa `context: ../../backend` desde `search-router/` si colocas `Infra` junto a `InvestPlatform` de forma que `../../backend` resuelva.

3. No levantes este servicio si solo necesitas SearXNG (`../searxng/`).

## Arranque (cuando el context de build es correcto)

Por defecto el compose usa `search-router.env.example` como `env_file`. Para secretos propios, copia a `search-router.env` y en un `docker-compose.override.yml` local pon `env_file: ./search-router.env` (no subas secretos al repo).

```bash
cd search-router
podman compose -f podman-compose.yml up --build -d
```
