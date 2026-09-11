# CipherWatch / CryptoTrace — Deploy Package

## ✅ What I tested
- **Backend**: installs cleanly with `pip install -r requirements.txt`, imports with no errors, and all routes load (`/`, `/api/config`, `/api/trace`, `/api/trace/stream`, `/api/cases`, `/api/reports/{id}/pdf`, `/api/freeze-requests/...`).
- **Frontend**: `npm install` + `npm run build` succeed with no errors (TanStack Start, Cloudflare Workers build preset).

## ⚠️ Important — read before you demo this
**The frontend dashboard is currently 100% mock data.** `src/routes/index.tsx` imports everything (`cases`, `transactions`, `riskFactors`, etc.) from `src/data/mock.ts`. There is **no `fetch`/`axios`/`EventSource` call anywhere in the frontend** that hits the backend's `/api/...` endpoints. So right now:
- Backend and frontend both run fine **independently**.
- They are **not wired together**. Deploying both won't make the dashboard show real traced data — it'll show the same mock numbers whether the backend is up or down.

This is fine for a prototype demo (it'll look complete and won't crash), but if a judge/reviewer asks "is this live data?", be ready to say it's using seed/mock data for the UI while the backend trace engine works independently (you can show that via `/docs`).

If you want it truly wired up next, the smallest fix is in `src/routes/index.tsx`: replace the mock imports with a `fetch(import.meta.env.VITE_API_URL + "/api/...")`/`EventSource` call to the deployed backend URL. I can do this for you if you want — just say so.

## Secrets
I removed the real `.env` (with your teammate's live Etherscan key + Supabase DB URL) from this package for safety. Use `backend/.env.example` as the template and set the real values only in your hosting provider's environment variable settings — never commit them.

## How to deploy

### Backend (FastAPI) — e.g. Render / Railway / Fly.io
1. Push `backend/` as its own repo (or subfolder root).
2. Set env vars from `.env.example`: `EtherAPI`, `SUPABASE_DB_URL`.
3. Build: `pip install -r requirements.txt`
4. Start: `uvicorn main:app --host 0.0.0.0 --port $PORT`
5. Update CORS in `main.py` → add your deployed frontend URL to `allow_origins`.

### Frontend (TanStack Start) — e.g. Cloudflare Pages/Workers, Vercel, Netlify
1. Push `frontend/` as its own repo (or subfolder root).
2. Build: `npm install && npm run build`
3. The build already targets a Cloudflare Workers preset (`.output/`, `wrangler.json` generated) — easiest path is Cloudflare Pages/Workers via `npx wrangler deploy`. For Vercel/Netlify you'd want to switch the Nitro preset in `vite.config.ts`/TanStack Start config to `vercel`/`netlify`.
4. No env vars are required today since it doesn't call the backend yet.

## Folder structure
```
backend/    FastAPI service (CipherWatch API) — tracing, cases, reports, freeze requests
frontend/   TanStack Start + React dashboard (cryptotrace-prj) — currently mock-data UI
```
