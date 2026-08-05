# Release Notes — v1.0.1

**Released:** 2026-08-05
**Tag:** `v1.0.1`
**Branch:** `master`

---

## 📦 Release Summary

This release establishes the **v1.0 release line** of the **Distributed Issue Tracker — FastAPI Microservices** platform: 9 independently deployable services, containerized images on GitHub Container Registry (GHCR), and all 9 services published to PyPI as independent, self-contained packages (non-commercial license).

---

## 🚀 New in This Release

### 1. CI/CD Automation (GitHub Actions)

| Workflow | Purpose |
|----------|---------|
| `.github/workflows/build-images.yml` | Builds & pushes all 9 service images to GHCR on `v*` tags and manual dispatch |
| `.github/workflows/publish-pypi.yml` | Builds & publishes all 9 service wheels to PyPI using **Trusted Publishing (OIDC)** — no API tokens required |

### 2. PyPI Distribution (9 independent packages)

Each service is published to PyPI as an independent, self-contained package (suffix `-byom`), each embedding the `shared/` library and declaring its full runtime dependencies:

| PyPI Package | Service | Description |
|--------------|---------|-------------|
| `api-gateway-byom` | API Gateway | Central routing, JWT validation, rate limiting, TLS |
| `auth-service-byom` | Auth Service | Authentication, JWT issuance, OAuth2, RBAC |
| `user-service-byom` | User Service | User profiles, teams, roles, preferences |
| `project-service-byom` | Project Service | Project lifecycle, membership, permissions |
| `issue-service-byom` | Issue Service | Issue CRUD, workflows, SLA, assignments |
| `comment-service-byom` | Comment Service | Comments, mentions, edit history, real-time |
| `notification-service-byom` | Notification Service | Email / in-app / webhook notifications via Celery |
| `search-service-byom` | Search Service | Full-text + faceted search via Elasticsearch |
| `audit-service-byom` | Audit Service | Audit logs, compliance events, data access tracking |

**Packaging metadata includes:** author (om kumar sahu), license (CC BY-NC 4.0), keywords, classifiers, homepage/repository/documentation URLs, and long description from each service's `README.md`.

### 3. Container Images (GHCR)

All 9 services build as Docker images and are published to:

```
ghcr.io/omkumar01/distributed-issue-tracker-fastapi-microservices-/<service>
```

Tagged with:
- `v1.0.1` (release tag)
- `sha-<commit-sha>` (per-commit)

### 4. Documentation

- Added comprehensive `README.md` for **7 services** (api-gateway, user-service, project-service, issue-service, comment-service, notification-service, search-service), covering features, API endpoints, configuration, and development.
- `auth-service` and `audit-service` retain their existing detailed READMEs.

---

## 📦 Packages

### PyPI Wheels
```
api-gateway-byom==1.0.1
auth-service-byom==1.0.1
user-service-byom==1.0.1
project-service-byom==1.0.1
issue-service-byom==1.0.1
comment-service-byom==1.0.1
notification-service-byom==1.0.1
search-service-byom==1.0.1
audit-service-byom==1.0.1
```

### Install
```bash
pip install api-gateway-byom
# or
pip install user-service-byom issue-service-byom
```

Each wheel bundles its service code, the `shared/` package, and all runtime dependencies.

---

## 🏗️ Architecture

- **8 backend services + 1 API gateway** (microservices pattern)
- **Database-per-service**: PostgreSQL (async via SQLAlchemy + asyncpg)
- **Event-driven**: RabbitMQ + Pika / Celery for async workflows
- **Caching**: Redis
- **Search**: Elasticsearch
- **Observability**: OpenTelemetry + Jaeger distributed tracing
- **API Gateway**: Centralized routing with JWT validation (Traefik-based)

---

## 🛠️ Installation & Usage

### Docker
```bash
docker-compose up -d --build
```

### Kubernetes
```bash
kubectl apply -f k8s/
```

### Python Packages
```bash
pip install <service>-byom
```

---

## ⚖️ License

**CC BY-NC 4.0** (Creative Commons Attribution-NonCommercial 4.0 International) — all 9 packages.

This is a **non-commercial** license: sharing and adaptation are permitted with attribution, but **commercial use is prohibited** without additional permission.

---

## 🤝 Contributors

- **om kumar sahu** — <omkumarsahu747@gmail.com>

---

## 🗺️ Roadmap

- [ ] OAuth2 provider integration (Google, GitHub)
- [ ] Email verification & password reset flows
- [ ] Two-factor authentication (2FA)
- [ ] WebSocket real-time support in API Gateway
- [ ] Kubernetes production deployment manifests
- [ ] Prometheus/Grafana metrics dashboards

---

## 🔗 Links

- **Repository:** https://github.com/omkumar01/Distributed-Issue-Tracker-FastAPI-Microservices-
- **Issues / Bug Tracker:** https://github.com/omkumar01/Distributed-Issue-Tracker-FastAPI-Microservices-/issues
- **PyPI:** https://pypi.org/ (search `*-byom`)
- **GHCR:** https://github.com/omkumar01/Distributed-Issue-Tracker-FastAPI-Microservices-/packages
