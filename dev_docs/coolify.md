# Deploy on Coolify (with Supabase)

Step-by-step guide to deploy this app on [Coolify](https://coolify.io) using **Supabase** as the database. No PostgreSQL container runs in Docker; the app connects to your Supabase project.

---

## Overview

| What              | Where it runs                          |
|-------------------|----------------------------------------|
| **App (server)**  | Built from your repo in Coolify        |
| **Worker**        | Same image as server                   |
| **Redis**         | Docker container in Coolify            |
| **Database**      | **Supabase only** (no Postgres in Docker) |

---

## Prerequisites

- A [Coolify](https://coolify.io) instance (self-hosted or Coolify Cloud).
- A [Supabase](https://supabase.com) account and project.
- This repository pushed to GitHub (or another Git host Coolify supports).

---

## Step 1: Supabase setup

1. Go to [supabase.com](https://supabase.com) and sign in.
2. Create a new project (or use an existing one).
3. Open **Project Settings** (gear icon) → **Database**.
4. Under **Connection string**, select **URI**.
5. Copy the **Session pooler** connection string (port **6543**).
   - It looks like:  
     `postgresql://postgres.[PROJECT_REF]:[YOUR-PASSWORD]@aws-0-[REGION].pooler.supabase.com:6543/postgres`
6. Replace `[YOUR-PASSWORD]` with your actual database password (the one you set when creating the project).
7. Save this full URL; you will use it as `PG_DATABASE_URL` in Coolify.

**Optional:** In Supabase → **Database** → **Connection pooling**, note if your project uses a different pooler host/port and adjust the URL if needed.

---

## Step 2: Coolify – create new resource

1. In Coolify, open the **Project** where you want the app.
2. Click **+ Add Resource** (or **Create New Resource** / **New** depending on your Coolify version).
3. Choose **Public Repository** (or **GitHub App** / **Deploy Key** for a private repo).

---

## Step 3: Connect your Git repo

1. **Repository URL**  
   - Example: `https://github.com/YOUR_USERNAME/YOUR_REPO.git`  
   - Use your actual GitHub (or GitLab, etc.) repo URL.
2. **Branch**  
   - Usually `main` or `master` (your default branch).
3. **Build pack**  
   - Select **Docker Compose** (do **not** use Nixpacks).

---

## Step 4: Docker Compose configuration

1. **Docker Compose Location**  
   - Set to:  
     `packages/twenty-docker/docker-compose.supabase.coolify.yml`  
   - This file builds the app from the repo and uses Supabase (no Postgres container).
2. **Base directory**  
   - Set to: **`/`** (repo root).  
   - Coolify will run the compose from the repo root so the build context is correct.

---

## Step 5: Environment variables

In Coolify’s **Environment Variables** section for this resource, add the following.

### Required

| Variable           | Description | Example |
|--------------------|-------------|--------|
| `PG_DATABASE_URL`  | Full Supabase connection string (Session pooler, port 6543). | `postgresql://postgres.abcdefgh:YourPassword@aws-0-us-east-1.pooler.supabase.com:6543/postgres` |
| `SERVER_URL`       | The exact URL where users will open the app (HTTPS). Must match the domain you assign in Coolify. | `https://your-app.xxx.coolify.io` or `https://crm.yourdomain.com` |
| `APP_SECRET`       | Random secret for sessions/tokens. Generate with: `openssl rand -base64 32` | Long random string |

### Optional (defaults are usually fine)

| Variable                  | Default | Notes |
|---------------------------|---------|--------|
| `REDIS_URL`               | `redis://redis:6379` | Leave default when using the Redis service from the compose file. |
| `STORAGE_TYPE`            | `local` | Use `local` for file storage on the container; change if you use S3 later. |
| `DISABLE_DB_MIGRATIONS`   | `false` | Keep `false` on first deploy so migrations run against Supabase. |
| `DISABLE_CRON_JOBS_REGISTRATION` | (empty) | Can leave unset. |

**Important:** `SERVER_URL` must be the **final** URL (with `https://`) that you assign to the app in Coolify. The Dockerfile uses it as `REACT_APP_SERVER_BASE_URL` so the frontend talks to the correct API.

---

## Step 6: Domain and port

1. The compose file exposes the **server** service on port **3000**.
2. In Coolify, assign a **domain** to this service:
   - Use the auto-generated Coolify domain (e.g. `https://your-app.xxx.coolify.io`), or  
   - Add your own domain and point DNS to your Coolify server.
3. Set **SERVER_URL** in env vars to that **exact** URL (including `https://`).
4. Coolify will handle HTTPS (e.g. Let’s Encrypt) when you use its proxy.

---

## Step 7: Deploy

1. Save the resource configuration.
2. Click **Deploy** (or **Start** / **Build and deploy**).
3. Coolify will:
   - Clone your repo.
   - Build the Docker image from `packages/twenty-docker/twenty/Dockerfile` (using `SERVER_URL` for the frontend).
   - Start **server**, **worker**, and **redis**.
   - **Not** start any Postgres container; the app uses **Supabase** via `PG_DATABASE_URL`.
4. On **first deploy**, the server entrypoint runs database migrations against your Supabase database.
5. Check the **server** service logs in Coolify for errors. When you see the app listening on port 3000, open **SERVER_URL** in the browser.

---

## Troubleshooting

### Build fails

- Confirm **Base directory** is **`/`** (repo root).
- Confirm **Docker Compose Location** is exactly:  
  `packages/twenty-docker/docker-compose.supabase.coolify.yml`
- Check build logs in Coolify for missing files or Docker build errors.

### App starts but frontend can’t reach API

- Ensure **SERVER_URL** matches the domain you assigned in Coolify (same host, `https`).
- Rebuild and redeploy after changing **SERVER_URL** (it is passed as a build arg).

### Database connection errors

- Verify **PG_DATABASE_URL** is the **Session pooler** string (port **6543**), not the direct connection (5432) unless you intend to use that.
- Ensure `[YOUR-PASSWORD]` is replaced and the password has no characters that break the URL (e.g. encode special characters if needed).
- In Supabase, check that **Database** → **Network** allows connections from your Coolify server’s IP if you use IP restrictions.

### Migrations not running

- Keep **DISABLE_DB_MIGRATIONS** unset or set to `false` on first deploy.
- Check server logs for migration or Postgres connection errors.

---

## Files in this repo

| File | Purpose |
|------|--------|
| `packages/twenty-docker/docker-compose.supabase.coolify.yml` | Compose file for Coolify: build from repo + Supabase, no Postgres container. |
| `packages/twenty-docker/README-COOLIFY.md` | Short Coolify + Supabase reference in the docker package. |
| `dev_docs/coolify.md` | This detailed guide for future reference. |

---

## Summary checklist

- [ ] Supabase project created; Session pooler connection string (port 6543) copied; password replaced in URL.
- [ ] Coolify resource created; Git repo and branch set; build pack = **Docker Compose**.
- [ ] Docker Compose path = `packages/twenty-docker/docker-compose.supabase.coolify.yml`; Base directory = `/`.
- [ ] Env vars set: `PG_DATABASE_URL`, `SERVER_URL`, `APP_SECRET`.
- [ ] Domain assigned to the server service; `SERVER_URL` matches that domain (with `https://`).
- [ ] Deploy run; server logs checked; app opened at `SERVER_URL`.
