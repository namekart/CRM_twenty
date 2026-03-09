# Environment variables setup

## Quick start (local with Docker)

### 1. Server — `packages/twenty-server/.env`

Create from example and **edit only what you need**:

```bash
cp packages/twenty-server/.env.example packages/twenty-server/.env
```

| Variable | Required | What to set |
|----------|----------|-------------|
| `NODE_ENV` | ✅ | `development` |
| `PG_DATABASE_URL` | ✅ | `postgres://postgres:postgres@localhost:5432/default` (or your Supabase/DB URL) |
| `REDIS_URL` | ✅ | `redis://localhost:6379` (or your Redis Cloud URL) |
| `APP_SECRET` | ✅ | **Change this.** Any long random string (e.g. `openssl rand -hex 32`) |
| `FRONTEND_URL` | ✅ | `http://localhost:3001` |
| `SIGN_IN_PREFILLED` | ✅ | `true` for local dev (prefills login) |
| `CODE_INTERPRETER_TYPE` | ✅ | `local` |
| `IS_WORKSPACE_CREATION_LIMITED_TO_SERVER_ADMINS` | ✅ | `false` |

Everything else in `.env.example` is **optional** (auth, email, storage, etc.). Defaults are fine for local.

### 2. Frontend — `packages/twenty-front/.env`

```bash
cp packages/twenty-front/.env.example packages/twenty-front/.env
```

| Variable | Required | What to set |
|----------|----------|-------------|
| `REACT_APP_SERVER_BASE_URL` | ✅ | `http://localhost:3000` (your server URL) |
| `VITE_BUILD_SOURCEMAP` | Optional | `false` |

No other frontend env vars are needed for local.

---

## When you change database or Redis

- **Supabase:** set `PG_DATABASE_URL` to the Supabase connection string (URI from Project Settings → Database).
- **Redis Cloud:** set `REDIS_URL` to the Redis Cloud URL (e.g. `redis://default:PASSWORD@host:port` or `rediss://...` for TLS).

---

## Optional (later)

- **Google / Microsoft login:** uncomment and set `AUTH_GOOGLE_*` or `AUTH_MICROSOFT_*` in server `.env`.
- **Email:** uncomment `EMAIL_*` and set your SMTP or use `EMAIL_DRIVER=LOGGER` for local.
- **Production:** set `APP_SECRET` to a strong random value and adjust `FRONTEND_URL` / `SERVER_URL` to your real domain.
