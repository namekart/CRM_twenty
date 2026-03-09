# Deploy as your own app with Supabase

This setup runs **your** app (your repo / your image) in Docker **without** a PostgreSQL container. It uses your **Supabase** project as the database.

## What runs in Docker

- **server** – Your app (API + frontend)
- **worker** – Background jobs
- **redis** – Cache (in Docker)

**No** `db` / Postgres container; the app connects to Supabase using `PG_DATABASE_URL`.

---

## 1. Supabase

1. Create a project at [supabase.com](https://supabase.com).
2. In **Project Settings → Database** copy the **Connection string**.
   - **Session pooler** (port **6543**): use this for the app.
   - Replace `[YOUR-PASSWORD]` with your database password.
   - Example:  
     `postgresql://postgres.xxxxx:YOUR_PASSWORD@aws-0-us-east-1.pooler.supabase.com:6543/postgres`

---

## 2. Server setup

On your server (VM):

```bash
# Clone YOUR repo (not Twenty upstream)
git clone https://github.com/YOUR_USERNAME/YOUR_REPO.git app
cd app/packages/twenty-docker
```

---

## 3. Environment

```bash
cp .env.supabase.example .env
nano .env
```

Set at least:

| Variable | Where to get it |
|----------|------------------|
| `PG_DATABASE_URL` | Supabase → Project Settings → Database → Connection string (Session pooler, port 6543). Replace `[YOUR-PASSWORD]`. |
| `SERVER_URL` | URL to open the app, e.g. `https://crm.yourdomain.com` or `http://YOUR_SERVER_IP:3000` |
| `APP_SECRET` | Run `openssl rand -base64 32` and paste the result |

Example `.env`:

```env
TAG=latest
PG_DATABASE_URL=postgresql://postgres.abcdefgh:MySecurePass123@aws-0-us-east-1.pooler.supabase.com:6543/postgres
SERVER_URL=https://crm.yourdomain.com
APP_SECRET=your_openssl_output_here
REDIS_URL=redis://redis:6379
STORAGE_TYPE=local
```

---

## 4. Build and run **your** image (your app, not Twenty’s)

From the **repo root** (one level above `packages/twenty-docker`):

```bash
cd /path/to/app   # repo root
docker build -f packages/twenty-docker/twenty/Dockerfile \
  --build-arg REACT_APP_SERVER_BASE_URL=https://crm.yourdomain.com \
  -t myapp:latest .
```

Use the same URL as `SERVER_URL` for `REACT_APP_SERVER_BASE_URL`.

Then run with Supabase (no Postgres container):

```bash
cd packages/twenty-docker
# Tell compose to use your image
echo "APP_IMAGE=myapp:latest" >> .env
docker compose -f docker-compose.supabase.yml up -d
```

Or add `APP_IMAGE=myapp:latest` to your `.env` file so the server and worker use your image instead of `twentycrm/twenty:latest`.

---

## 5. First run (migrations)

On first start, the server **runs DB migrations** against Supabase (via `entrypoint.sh`). Ensure `PG_DATABASE_URL` is correct and that Supabase allows connections from your server’s IP (Supabase → Settings → Database → Connection pooling / Network if applicable).

Check logs:

```bash
docker compose -f docker-compose.supabase.yml logs -f server
```

When you see the app listening on port 3000, open `SERVER_URL` in the browser.

---

## 6. Summary

| Item | Use |
|------|-----|
| Database | **Supabase** (no Postgres container) |
| App image | **Your** build from your repo |
| Compose file | `docker-compose.supabase.yml` |
| Env file | `.env` from `.env.supabase.example` |

No PostgreSQL image is pulled or run in Docker; only your app, worker, and Redis run in Docker, and the app talks to Supabase via `PG_DATABASE_URL`.
