
---

### `docs/LOCAL_DEV.md`
```md
# Desenvolvimento Local

## Requisitos
- **Python 3.11+**
- **Virtualenv** (recomendado)
- **SQLite** por padrão (ou configure `DATABASE_URL`)

## Variáveis de ambiente (`.env`)
```env
DJANGO_SECRET_KEY=change-me
DJANGO_DEBUG=true
DJANGO_ALLOWED_HOSTS=127.0.0.1,localhost
DATABASE_URL=sqlite:///db.sqlite3
