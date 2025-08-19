# API — Endpoints e exemplos

Esta API usa **Django REST Framework** com **JWT (SimpleJWT)** e **OpenAPI** via **drf-spectacular**.

## Endpoints principais

- **Health**: `GET /api/health/` → `200 OK`
- **OpenAPI JSON**: `GET /api/schema/`
- **Swagger UI**: `GET /api/docs/`
- **ReDoc**: `GET /api/redoc/`
- **JWT**
  - `POST /api/token/` (obter `access` e `refresh`)
  - `POST /api/token/refresh/` (renovar `access`)
- **Items (CRUD)**
  - `GET | POST /api/items/`
  - `GET | PUT | PATCH | DELETE /api/items/{id}/`

> Permissões padrão: **IsAuthenticated** para `items/*` e **AllowAny** para `/api/health/`.

---

## Health

```bash
curl -s http://127.0.0.1:8000/api/health/ | jq
