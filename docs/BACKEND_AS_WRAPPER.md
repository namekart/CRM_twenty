# Using Twenty Backend as a CRM Wrapper

Use Twenty’s **backend only** and build your own frontend (or reuse parts of Twenty’s UI) as a wrapper around the API.

## Why backend-only?

- **Faster dev loop:** Run only server + worker (no full frontend build/watch).
- **Your UI:** Your app talks to Twenty via GraphQL; you control the UX.
- **Reuse what you need:** Use Twenty’s frontend components (e.g. from `twenty-ui`) in your app if you want.

---

## 1. Run only the backend

From repo root:

```bash
# Start Postgres + Redis (if using Docker)
docker compose -f docker-compose.dev.yml up -d

# Run only server + worker (no Twenty frontend)
yarn start:backend
```

- **First run:** Nx will still build `twenty-shared`, `twenty-ui`, `twenty-emails` (server dependencies). Do this once.
- **Later runs:** With Nx cache, startup is faster. You only run server + worker, no frontend.

**Endpoints:**

| What        | URL                        |
|------------|----------------------------|
| API        | http://localhost:3000     |
| GraphQL    | http://localhost:3000/graphql |
| Health     | http://localhost:3000/healthz |

---

## 2. Use the API from your wrapper

### Option A: GraphQL directly

- **Endpoint:** `POST http://localhost:3000/graphql`
- Send GraphQL queries/mutations. Auth via **Bearer token** (or your auth layer in front of Twenty).
- You can explore the schema at `http://localhost:3000/graphql` (if GraphQL Playground is enabled in dev).

### Option B: Twenty SDK (recommended)

- **Package:** `twenty-sdk` (in this repo: `packages/twenty-sdk`).
- Typed clients: `CoreApiClient` (workspace data), `MetadataApiClient` (config, file uploads).
- In your wrapper app: `yarn add twenty-sdk` and point it at `http://localhost:3000` (or your deployed backend).

```ts
import { CoreApiClient } from 'twenty-sdk';

const client = new CoreApiClient({
  apiUrl: 'http://localhost:3000',
  apiKey: 'your-api-key', // or use auth token
});
// Use client for queries/mutations
```

See `packages/twenty-sdk/README.md` and [Twenty apps docs](https://docs.twenty.com/developers/extend/capabilities/apps).

---

## 3. Reusing Twenty’s frontend pieces

- **`packages/twenty-ui`:** Shared UI components (buttons, inputs, etc.). You can depend on it in your app and import what you need.
- **`packages/twenty-front`:** Full CRM UI (React). You can copy specific modules or run it only when you need the full UI; your main app stays your wrapper that calls the backend.

---

## 4. Suggested workflow

1. **Backend:** `yarn start:backend` → server on port 3000, worker running.
2. **Your wrapper app:** Separate repo or folder (e.g. Next.js, Vite, etc.) that calls `http://localhost:3000/graphql` or uses `twenty-sdk`.
3. **Optional:** Use `twenty-ui` in your wrapper for a consistent look with Twenty’s UI.

---

## 5. One-time full build (optional)

To avoid rebuilding server deps on every run:

```bash
# Build server and all its dependencies once
npx cross-env NX_DAEMON=false npx nx run twenty-server:build

# Then run server (uses existing build)
cd packages/twenty-server && npx cross-env NODE_ENV=development node dist/main.js
# In another terminal, run the worker:
npx nx run twenty-server:worker
```

For day-to-day dev, `yarn start:backend` is usually enough.
