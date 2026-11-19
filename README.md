# SentinelLLM Frontend

A responsive React + Vite security dashboard for the existing SentinelLLM FastAPI backend.

## Features
- Live backend health and model-loaded status (`GET /health`)
- Single prompt scan (`POST /scan`, JSON body uses **`text`**)
- CSV batch analysis (`POST /batch`, multipart field name `file`)
- Evaluation report (`GET /evaluation`)
- Batch result table and CSV export
- Session-only recent activity (no invented historical analytics)

## Run locally
```bash
npm install
cp .env.example .env
npm run dev
```
Open the local URL printed by Vite.

## Backend configuration
Default API: `https://sentinel-llm-rust.vercel.app`

To change it, set `VITE_API_BASE_URL` in a `.env` file or in Vercel Project Settings → Environment Variables, then redeploy.

## Deploy frontend to Vercel
1. Create a **new** GitHub repository for this frontend and upload the contents of this folder.
2. In Vercel, import that new repository.
3. Framework preset: **Vite**. Build command: `npm run build`. Output directory: `dist`.
4. Add `VITE_API_BASE_URL=https://sentinel-llm-rust.vercel.app` as an environment variable.
5. Deploy.

Keep the existing backend Vercel project unchanged.

## Important: CORS
The browser must be allowed by the FastAPI backend to call it from the frontend domain. The current frontend displays API/CORS errors rather than pretending requests succeeded. If browser devtools reports a CORS error, add `CORSMiddleware` to the backend and allow the exact deployed frontend origin (and localhost during development), then redeploy the backend. Do not use `*` with credentials.

Example backend middleware (add near app creation, before routes):
```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "http://localhost:5173",
        "https://YOUR-FRONTEND.vercel.app",
    ],
    allow_credentials=False,
    allow_methods=["GET", "POST", "OPTIONS"],
    allow_headers=["Content-Type", "Accept"],
)
```

## API contract used
- `GET /` — backend info (not required by UI)
- `GET /health` — expects `status` and `model_loaded`
- `POST /scan` — JSON `{ "text": "..." }`
- `POST /batch` — multipart form field `file` with a `.csv` file containing a `text` column; max 5,000 rows
- `GET /evaluation` — renders the returned JSON structure dynamically

The UI does not include authentication because the inspected API does not expose an authentication flow. Do not expose private API credentials in frontend code.
