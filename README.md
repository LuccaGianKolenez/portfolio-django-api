# Portfolio — Django API (DRF + JWT + Swagger)

API de portfólio construída com **Django REST Framework**, autenticação **JWT (SimpleJWT)** e documentação **OpenAPI** via **drf-spectacular**. Projeto preparado para testes com **pytest** e qualidade de código com **ruff/black/isort/mypy**.

> Objetivo: servir como referência **sênior** de estrutura, organização e boas práticas para APIs em Django/DRF.

---

## Sumário
- [Arquitetura & Stack](#arquitetura--stack)
- [Estrutura de Pastas](#estrutura-de-pastas)
- [Variáveis de Ambiente](#variáveis-de-ambiente)
- [Como Rodar (Local)](#como-rodar-local)
- [Rotas Principais](#rotas-principais)
- [Autenticação (JWT)](#autenticação-jwt)
- [Domínio: Items (CRUD)](#domínio-items-crud)
- [Postman Collection](#postman-collection)
- [Testes & Qualidade](#testes--qualidade)
- [Erros Comuns](#erros-comuns)
- [Roadmap](#roadmap)
- [Licença](#licença)

---

## Arquitetura & Stack

- **Django 4+/5+** + **Django REST Framework**
- **JWT** com `djangorestframework-simplejwt`
- **OpenAPI/Swagger/ReDoc** com `drf-spectacular` (+ `drf-spectacular-sidecar`)
- **Filtros/Busca/Ordenação**: `django-filter`, `search_fields`, `ordering_fields`
- **Config 12-factor** via **.env** (`django-environ`)
- Qualidade: **pytest**, **ruff**, **black**, **isort**, **mypy**

---

## Estrutura de Pastas

```
portfolio-django-api/
├─ config/
│  ├─ settings/
│  │  ├─ base.py        # DRF, JWT, Spectacular, filtros, etc.
│  │  ├─ dev.py         # DEBUG=True
│  │  └─ prod.py        # DEBUG=False
│  ├─ urls.py           # roteamento principal (API/Swagger/ReDoc/JWT)
│  └─ wsgi.py
├─ core/
│  ├─ views.py          # /api/health/
│  └─ tests.py          # teste do health
├─ items/
│  ├─ models.py         # Item (UUID, name, price, created_at)
│  ├─ serializers.py
│  └─ views.py          # ViewSet com filtros, busca e ordenação
├─ postman/
│  └─ portfolio-django-api.postman_collection.json
├─ .env.example
├─ pyproject.toml       # pytest, ruff, black, isort, mypy
├─ manage.py
└─ .gitignore
```

---

## Variáveis de Ambiente

Arquivo: **`.env`** (baseado em `.env.example`)

```env
DJANGO_SECRET_KEY=change-me
DJANGO_DEBUG=true
DJANGO_ALLOWED_HOSTS=127.0.0.1,localhost
DATABASE_URL=sqlite:///db.sqlite3
```

> Para produção, ajuste `DJANGO_DEBUG=false`, configure `DJANGO_ALLOWED_HOSTS` e use um banco gerenciado (Postgres etc.) via `DATABASE_URL`.

---

## Como Rodar (Local)

```bash
python3 -m venv .venv
source .venv/bin/activate

python -m pip install -U pip setuptools wheel
pip install django djangorestframework django-filter \
            drf-spectacular drf-spectacular-sidecar \
            djangorestframework-simplejwt django-environ \
            pytest pytest-django ruff black isort mypy types-requests

cp .env.example .env
python manage.py makemigrations
python manage.py migrate

# (opcional) usuário admin para uso no Django Admin ou testes JWT
python manage.py createsuperuser

python manage.py runserver
```

- Swagger UI: **http://127.0.0.1:8000/api/docs/**
- OpenAPI (JSON): **http://127.0.0.1:8000/api/schema/**
- ReDoc: **http://127.0.0.1:8000/api/redoc/**
- Health: **http://127.0.0.1:8000/api/health/**

---

## Rotas Principais

| Recurso           | Método(s)                     | Caminho                         | Auth           |
|-------------------|-------------------------------|----------------------------------|----------------|
| **Health**        | `GET`                          | `/api/health/`                  | Público        |
| **OpenAPI JSON**  | `GET`                          | `/api/schema/`                  | Público        |
| **Swagger UI**    | `GET`                          | `/api/docs/`                    | Público        |
| **ReDoc**         | `GET`                          | `/api/redoc/`                   | Público        |
| **JWT Obtain**    | `POST`                         | `/api/token/`                   | Público        |
| **JWT Refresh**   | `POST`                         | `/api/token/refresh/`           | Público        |
| **Items**         | `GET \| POST`                  | `/api/items/`                   | **JWT**        |
| **Item por ID**   | `GET \| PUT \| PATCH \| DELETE`| `/api/items/{uuid}/`            | **JWT**        |

**Health (exemplo)**
```bash
curl -s http://127.0.0.1:8000/api/health/
# -> {"status":"ok"}
```

---

## Autenticação (JWT)

### Obter tokens (`/api/token/`)
```bash
curl -s -X POST http://127.0.0.1:8000/api/token/ \
  -H "Content-Type: application/json" \
  -d '{"username":"seuusuario","password":"suasenha"}'
```
**Resposta**
```json
{ "refresh": "...", "access": "..." }
```

### Renovar access (`/api/token/refresh/`)
```bash
curl -s -X POST http://127.0.0.1:8000/api/token/refresh/ \
  -H "Content-Type: application/json" \
  -d '{"refresh":"<token-de-refresh>"}'
```

> Configure o header `Authorization: Bearer <access>` em chamadas autenticadas.

---

## Domínio: Items (CRUD)

**Modelo**: `id (UUID, PK)`, `name (str)`, `price (decimal)`, `created_at (auto_now_add)`

**Listar** (paginação + busca + ordenação)
```bash
curl -s "http://127.0.0.1:8000/api/items/?search=note&ordering=-created_at" \
  -H "Authorization: Bearer <access>"
```
Parâmetros suportados:
- `search`: busca em `name`
- `ordering`: `name`, `price`, `created_at` (use `-` para desc)

**Criar**
```bash
curl -s -X POST http://127.0.0.1:8000/api/items/ \
  -H "Authorization: Bearer <access>" \
  -H "Content-Type: application/json" \
  -d '{"name":"Notebook","price":"1999.90"}'
```

**Detalhar / Atualizar / Patch / Remover**
```bash
# GET
curl -s http://127.0.0.1:8000/api/items/<uuid>/ -H "Authorization: Bearer <access>"

# PUT (replace)
curl -s -X PUT http://127.0.0.1:8000/api/items/<uuid>/ \
  -H "Authorization: Bearer <access>" -H "Content-Type: application/json" \
  -d '{"name":"Notebook Pro","price":"2999.90"}'

# PATCH (parcial)
curl -s -X PATCH http://127.0.0.1:8000/api/items/<uuid>/ \
  -H "Authorization: Bearer <access>" -H "Content-Type: application/json" \
  -d '{"price":"2799.90"}'

# DELETE
curl -s -X DELETE http://127.0.0.1:8000/api/items/<uuid>/ \
  -H "Authorization: Bearer <access>"
```

---

## Postman Collection

A coleção com todos os endpoints e **scripts de teste** para salvar `{{token}}`/`{{refresh}}` automaticamente está em:
```
postman/portfolio-django-api.postman_collection.json
```
Passos:
1. Importar no Postman
2. Executar **JWT • Obtain** (preencher variáveis `token` e `refresh`)
3. Usar as requisições de **Items** com `Authorization: Bearer {{token}}`

---

## Testes & Qualidade

**Testes**
```bash
pytest -q
```
**Linters e formatação**
```bash
ruff .
black --check .
isort --check-only .
# Para formatar:
black . && isort .
```

> Teste de exemplo: `core/tests.py` valida o **/api/health/**. Expanda com testes de domínio (items) e autenticação.

---

## Erros Comuns

- **401 Unauthorized**: faltou `Authorization: Bearer <access>` ou token expirado → gere/renove.
- **403 Forbidden**: usuário sem permissão para a ação.
- **400 Bad Request**: payload inválido; verifique tipos, requireds e formato (`Content-Type: application/json`).
- **404 Not Found**: ID inexistente ou rota incorreta.
- **Swagger 404**: confirme `drf_spectacular`/`sidecar` em `INSTALLED_APPS` e rotas `/api/schema`/`/api/docs`/`/api/redoc` no `urls.py`.
