# Deploy on Coolify (with Supabase)

Deploy **your** app on [Coolify](https://coolify.io) using **Supabase** as the database (no Postgres container).

---

## 1. Supabase

- Create a project at [supabase.com](https://supabase.com).
- In **Project Settings → Database**, copy the **Connection string** (Session pooler, port **6543**).
- Replace `[YOUR-PASSWORD]` with your database password.

---

## 2. Coolify setup

1. **New resource**
   - In Coolify: **Project** → **+ Add Resource** (or **Create New Resource**).

2. **Source**
   - Choose **Public Repository** (or **GitHub App** / **Deploy Key** if you use private repo).
   - **Repository URL**: `https://github.com/YOUR_USERNAME/YOUR_REPO`
   - **Branch**: `main` (or your default branch).

3. **Build pack**
   - Select **Docker Compose** (not Nixpacks).

4. **Compose file**
   - **Docker Compose Location**: `packages/twenty-docker/docker-compose.supabase.coolify.yml`
   - **Base Directory**: `/` (repo root).

5. **Environment variables** (in Coolify’s env UI)

   Set these (required):

   | Variable | Example | Notes |
   |----------|---------|--------|
   | `PG_DATABASE_URL` | `postgresql://postgres.xxx:password@aws-0-region.pooler.supabase.com:6543/postgres` | From Supabase → Database |
   | `SERVER_URL` | `https://your-app.coolify.domain.com` | The URL Coolify gives your app (or your custom domain) |
   | `APP_SECRET` | (random string) | e.g. run `openssl rand -base64 32` |

   Optional (defaults are fine to start):

   | Variable | Default |
   |----------|---------|
   | `REDIS_URL` | `redis://redis:6379` |
   | `STORAGE_TYPE` | `local` |
   | `DISABLE_DB_MIGRATIONS` | `false` (so first deploy runs migrations on Supabase) |

6. **Port / domain**
   - The compose file exposes port **3000**. In Coolify, assign a **domain** (or use the generated one) for the **server** service so you get HTTPS.
   - Set **SERVER_URL** to that exact URL (e.g. `https://crm.yourdomain.com`).

7. **Deploy**
   - Click **Deploy**. Coolify will clone the repo, build the image from the Dockerfile (server + worker), start Redis, and run the stack. First run will run DB migrations against Supabase.

---

## 3. What runs (no Postgres in Docker)

| Service | Where it runs |
|---------|----------------|
| **server** | Built from your repo in Coolify, uses Supabase |
| **worker** | Same image as server |
| **redis** | Docker (in Coolify) |
| **Database** | **Supabase only** (no Postgres container) |

---

## 4. If you use the non-Coolify Supabase compose

If you prefer to use a **pre-built image** (e.g. from a registry) instead of Coolify building from the repo:

- Use **Docker Compose Location**: `packages/twenty-docker/docker-compose.supabase.yml`
- Set **APP_IMAGE** in Coolify env to your image (e.g. `ghcr.io/yourname/your-app:latest`).
- Ensure that image is built with `REACT_APP_SERVER_BASE_URL` equal to **SERVER_URL**.

For “build from repo” on Coolify, use **docker-compose.supabase.coolify.yml** as above.
