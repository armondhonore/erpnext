# Nexlayer — erpnext

<!-- nexlayer:meta version=1 analyzed=2026-06-28T05:13:46Z repo=https://github.com/armondhonore/erpnext branch=develop -->

> **For AI agents (Claude Code, Cursor, Gemini CLI, Copilot):**
> This file is the **project context** for this Nexlayer deployment — tech stack, env vars, secrets, live URL.
> For full platform detail (nexlayer.yaml schema, Dockerfile rules, CI/CD, task recipes) read **`nexlayer.skills`** in this repo.
>
> **Critical rules (full detail in `nexlayer.skills`):**
> - Inter-pod refs: `${podName:port}` only — never `localhost` or bare hostnames
> - Docker Hub images: prefix with `mirror.gcr.io/library/` — bare tags fail on the cluster
> - Secrets: set in the Nexlayer dashboard — never commit to `nexlayer.yaml` or Dockerfile
>
> **This file:** `agent-managed` sections update automatically. `user-editable` sections (Local Development Setup, Nexlayer Deployment Plan, Build Notes) are yours — preserved across re-analysis.

## Project Summary
<!-- nexlayer:section agent-managed=project_summary -->
ERPNext is a comprehensive open-source Enterprise Resource Planning system built on the Frappe Framework, providing integrated modules for accounting, inventory, manufacturing, and asset management.
<!-- nexlayer:end -->

## Technology Stack
<!-- nexlayer:section agent-managed=tech_stack -->
| Name | Kind | Version | Detected From |
|------|------|---------|---------------|
| Python | language | >=3.14 | pyproject.toml |
| Frappe Framework | framework | >=17.0.0-dev | pyproject.toml |
| MariaDB | database | latest | README.md |
| Redis | database | latest | README.md |
| Node.js | language | latest | package.json |
| Yarn | tool | latest | yarn.lock |
<!-- nexlayer:end -->

## Repository Structure
<!-- nexlayer:section agent-managed=structure_map -->
- erpnext/ — Core ERPNext application logic and modules
- banking/ — Banking specific frontend assets and JS logic
- pyproject.toml — Python project metadata and dependencies
- package.json — Node.js dependencies and build scripts
<!-- nexlayer:end -->

## External Services Required
<!-- nexlayer:section agent-managed=external_deps -->
Services that must be configured separately (not deployed by Nexlayer):

- Google Maps API
- Plaid API
<!-- nexlayer:end -->

## Local Development Setup
<!-- nexlayer:section user-editable=local_setup -->
### Prerequisites

- Python >= 3.14
- Node.js
- Yarn
- MariaDB
- Redis

### Environment variables

Copy `.env.example` to `.env.local` and fill in:

```
DB_HOST=localhost
REDIS_CACHE=localhost
REDIS_QUEUE=localhost
```

### Steps

1. `yarn install` — Install global node dependencies
2. `cd banking && yarn install` — Install banking module dependencies
3. `pip install .` — Install Python dependencies from pyproject.toml
4. `yarn dev` — Start development assets builder

<!-- nexlayer:end -->

## Nexlayer Setup
<!-- nexlayer:section agent-managed=nexlayer_setup -->
### Pod Environment Variables

| Pod | Variable | Value | Kind |
|-----|----------|-------|------|
| `app` | `DB_HOST` | `erpnext-mariadb-service` | plain |
| `app` | `DB_PORT` | `"3306"` | plain |
| `app` | `DB_PASSWORD` | _(set via Nexlayer dashboard)_ | secret |
| `app` | `ADMIN_PASSWORD` | _(set via Nexlayer dashboard)_ | secret |
| `app` | `REDIS_CACHE` | `erpnext-redis-service:6379` | plain |
| `app` | `REDIS_QUEUE` | `erpnext-redis-service:6379` | plain |
| `app` | `SOCKETIO_PORT` | `"9000"` | plain |
| `mariadb` | `MYSQL_ROOT_PASSWORD` | `"${MYSQL_ROOT_PASSWORD}"` | inter-pod |
| `mariadb` | `MYSQL_DATABASE` | `erpnext` | plain |
| `mariadb` | `MYSQL_USER` | `erpnext` | plain |
| `mariadb` | `MYSQL_PASSWORD` | `"${MYSQL_PASSWORD}"` | inter-pod |
| `erpnext-db` | `mountPath` | `/var/lib/mysql` | plain |
| `erpnext-db` | `size` | `10Gi` | plain |

### Secrets Required

Set these in the Nexlayer dashboard before deploying:

- `DB_PASSWORD` (`app` pod)
- `ADMIN_PASSWORD` (`app` pod)

### nexlayer.yaml

```yaml
application:
  name: erpnext
  pods:
  - name: app
    image: mirror.gcr.io/frappe/erpnext:latest
    path: /
    servicePorts:
    - 8000
    vars:
      DB_HOST: erpnext-mariadb-service
      DB_PORT: "3306"
      DB_PASSWORD: erpnext123
      ADMIN_PASSWORD: admin123
      REDIS_CACHE: erpnext-redis-service:6379
      REDIS_QUEUE: erpnext-redis-service:6379
      SOCKETIO_PORT: "9000"
  - name: mariadb
    image: mirror.gcr.io/library/mariadb:10.6
    servicePorts:
    - 3306
    vars:
      MYSQL_ROOT_PASSWORD: "${MYSQL_ROOT_PASSWORD}"
      MYSQL_DATABASE: erpnext
      MYSQL_USER: erpnext
      MYSQL_PASSWORD: "${MYSQL_PASSWORD}"
    volumes:
    - name: erpnext-db
      mountPath: /var/lib/mysql
      size: 10Gi
  - name: redis
    image: mirror.gcr.io/library/redis:7-alpine
    servicePorts:
    - 6379
```

<!-- nexlayer:end -->

## Nexlayer Deployment Plan
<!-- nexlayer:section user-editable=deployment_plan -->
### Pod Topology

| Pod | Image | Port | Role |
|-----|-------|------|------|
| erpnext-web | mirror.gcr.io/library/python:3.14-slim | 8000 | web |
| erpnext-worker | mirror.gcr.io/library/python:3.14-slim | 8000 | worker |
| erpnext-scheduler | mirror.gcr.io/library/python:3.14-slim | 8000 | scheduler |
| mariadb | mirror.gcr.io/library/mariadb:10.11 | 3306 | database |
| redis-cache | mirror.gcr.io/library/redis:7-alpine | 6379 | cache |
| redis-queue | mirror.gcr.io/library/redis:7-alpine | 6379 | queue |

### Deployment notes

- The web, worker, and scheduler pods are separated to adhere to the one-service-per-pod rule.
- Database communication uses mariadb.pod:3306.
- Cache and Queue are split into separate Redis pods (redis-cache.pod and redis-queue.pod) to avoid co-location.
- Frontend assets from the banking folder must be built and served as static files.

<!-- nexlayer:end -->

## Build Notes
<!-- nexlayer:section user-editable=build_notes -->
<!-- Add notes for future builds here — preserved across re-analysis -->
<!-- nexlayer:end -->

## Nexlayer Configuration
<!-- nexlayer:section agent-managed=nexlayer_config -->
**Last deployed:** 2026-06-28T05:19:08Z  
**Live URL:** https://relaxed-weasel-erpnext.cloud.nexlayer.ai  
**Runtime:**  · **Port:** auto-detected  
**Deploy branch:** develop  

```yaml
application:
  name: erpnext
  pods:
  - name: app
    image: mirror.gcr.io/frappe/erpnext:latest
    path: /
    servicePorts:
    - 8000
    vars:
      DB_HOST: erpnext-mariadb-service
      DB_PORT: "3306"
      DB_PASSWORD: erpnext123
      ADMIN_PASSWORD: admin123
      REDIS_CACHE: erpnext-redis-service:6379
      REDIS_QUEUE: erpnext-redis-service:6379
      SOCKETIO_PORT: "9000"
  - name: mariadb
    image: mirror.gcr.io/library/mariadb:10.6
    servicePorts:
    - 3306
    vars:
      MYSQL_ROOT_PASSWORD: "${MYSQL_ROOT_PASSWORD}"
      MYSQL_DATABASE: erpnext
      MYSQL_USER: erpnext
      MYSQL_PASSWORD: "${MYSQL_PASSWORD}"
    volumes:
    - name: erpnext-db
      mountPath: /var/lib/mysql
      size: 10Gi
  - name: redis
    image: mirror.gcr.io/library/redis:7-alpine
    servicePorts:
    - 6379
```
<!-- nexlayer:end -->

## Build History
<!-- nexlayer:section agent-managed=build_history -->
| Date | Status | Notes |
|------|--------|-------|
| 2026-06-28T05:13:46Z | analyzed | initial repo analysis |
| 2026-06-28T05:19:08Z | success | deployed https://relaxed-weasel-erpnext.cloud.nexlayer.ai |
<!-- nexlayer:end -->
