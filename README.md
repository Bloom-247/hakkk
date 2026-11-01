# Northwind decoy portal

The separate sign-in portal behind the **Aegis Sentinel** live capture demo. It looks like a small employee portal,
records every sign-in attempt, and exposes the capture over HTTP so the Aegis console can ingest it automatically.

- Live portal (Render): <https://northwind-decoy-portal.onrender.com>
- Aegis console (Vercel): <https://aegis-sentinel-blush.vercel.app/app>
- Build guide: [DEPLOY section](#deploy-on-render) below

**Demo only.** The accounts, passwords and geolocation values are fictional. Passwords are never stored or sent to
Aegis; only sign-in metadata (email, timestamp, origin, driver, outcome) is written to `data/`.

## What happens when someone tries to sign in

1. The browser posts `email` + `password` to `/api/login`.
2. The server matches the fictional account list, records the attempt, and returns the flagged result.
3. Every failed request is counted and flagged individually in the response and on the portal page.
4. The Aegis console polls `/api/events?limit=1000` every two seconds, turns the capture into the **Northwind portal
   logins · live** Logs dataset, and rebuilds the risk score and anomaly register.

Open <https://aegis-sentinel-blush.vercel.app/app> after a few failed attempts to see the dashboard strip, anomaly
board and Data Input Hub update.

## Run locally

Requires Node 20+.

```bash
npm install
npm start
```

Open <http://localhost:4300>. Environment variables:

| variable | default | purpose |
| --- | --- | --- |
| `PORT` | `4300` | port to listen on (Render injects this) |
| `PORTAL_PORT` | `4300` | local fallback when `PORT` is absent |
| `PORTAL_HOST` | `0.0.0.0` | bind address |
| `AEGIS_CONSOLE_URL` | `http://localhost:5173` | Aegis origin used by the portal's console links |

## Deploy on Render

This repository is a Render blueprint.

1. In Render, choose **New → Blueprint** and connect this repository.
2. Render reads `render.yaml` and creates the free Node web service `northwind-decoy-portal`.
3. Confirm `AEGIS_CONSOLE_URL` is `https://aegis-sentinel-blush.vercel.app` (change it if your console lives
   elsewhere), then deploy.
4. Verify `https://northwind-decoy-portal.onrender.com/api/health` returns `ok: true`.
5. In the Aegis Vercel project, make sure `VITE_PORTAL_API_BASE` is
   `https://northwind-decoy-portal.onrender.com` (already declared in the Aegis `render.yaml`), and redeploy if the
   service name differs.

Health checks hit `/api/health`. The service binds `0.0.0.0` and uses Render's `PORT`.

### Ephemeral capture

Events are stored in `data/login-events.json` and `data/login-events.csv` on the service's local disk. Render's free
plan resets that disk on restart or redeploy, which is intentional for a demo — the portal starts with a clean ledger.
Use **Reset capture** on the portal to clear it sooner.

## Endpoints

| method | path | purpose |
| --- | --- | --- |
| `GET` | `/api/health` | service health, capture totals and the Aegis console origin |
| `GET` | `/api/events?limit=1000` | raw capture plus `total`, `failed` and `granted` |
| `GET` | `/api/events.csv` | downloadable event ledger |
| `POST` | `/api/login` | record one sign-in attempt (`email`, `password`, optional `origin`) |
| `POST` | `/api/simulate` | write a scripted account-takeover or spray burst |
| `POST` | `/api/reset` | clear the capture |

CORS is open on the read-only demo endpoints so the Aegis console can poll them from another origin.

## Files

| file | purpose |
| --- | --- |
| `index.html` | the decoy sign-in page |
| `portal.css` | page styling |
| `portal.js` | form handling, event rendering and live refresh |
| `server.mjs` | zero-dependency Node HTTP server, capture store and API |
| `render.yaml` | Render blueprint (service name, start command, health check, env vars) |
