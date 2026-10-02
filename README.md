# Piwpiw Snap — Website

A React + Vite + Tailwind frontend for Piwpiw Snap.

## Prerequisites

- Node.js 18+ and npm

## Setup

1. Install dependencies

```powershell
npm install
```

2. Configure environment (optional for local dev)

- Copy `.env.example` to `.env` and set values:
  - `VITE_API_URL` (defaults to `http://localhost:3001/api` if not set)
  - `VITE_DISCORD_CLIENT_ID` only if needed for a backend-owned OAuth flow.
  - Keep `DISCORD_CLIENT_SECRET` exclusively in the backend private environment; never expose it through a `VITE_` variable.

3. Run the dev server

```powershell
npm run dev
```

- Frontend will be available at: http://localhost:3000/
- Backend (if used) is expected at: http://localhost:3001/api

## Build

```powershell
npm run build
```

Outputs to `dist/`.

## Notes

- Env vars are read via `import.meta.env.*` in Vite. See `src/services/api.ts` for API base URL.
- `.env` is ignored by git; do not commit secrets.
