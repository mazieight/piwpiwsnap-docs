# Deploy Piwpiw Snap to a Windows VPS (RDP) with HTTPS

This guide sets up your built Vite app behind Caddy on `piwpiwsnap.site` with automatic HTTPS and an `/api` reverse proxy to your backend on `127.0.0.1:3001`.

## Prereqs
- Windows Server/Windows VPS (RDP access as Administrator)
- Node.js LTS installed (https://nodejs.org)
- Your domain `piwpiwsnap.site` A record pointing to the VPS public IP

## One‑time setup
1. Copy the project to the VPS (Git clone or upload zip and extract)
2. Open Windows PowerShell as Administrator in the project folder
3. Run the deploy script:

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force
./deploy/deploy.ps1
```

What the script does:
- Installs deps and runs `npm run build`
- Copies `dist/` to `C:\web\piwpiwsnap\dist`
- Places `Caddyfile` at `C:\web\piwpiwsnap\Caddyfile`
- Downloads Caddy (if missing) and installs it as a Windows service
- Opens firewall ports 80/443 and starts Caddy

Caddy auto‑provisions HTTPS via Let’s Encrypt once DNS resolves to this VPS.

## Caddyfile
`deploy/Caddyfile` is set to:
- Serve static files from `C:\web\piwpiwsnap\dist`
- Reverse proxy `/api/*` to `127.0.0.1:3001`
- SPA fallback to `index.html`

If your API runs on a different port/host, update this line:
```
reverse_proxy 127.0.0.1:3001
```

## Updating the site later
After you pull new changes:
```powershell
npm run build
# recopy dist to the web root if needed
Copy-Item -Recurse -Force dist C:\web\piwpiwsnap\dist
# reload caddy config (optional if Caddyfile changed)
& C:\web\caddy\caddy.exe reload --config C:\web\piwpiwsnap\Caddyfile
```

## Logs
- Caddy access logs: `C:\web\piwpiwsnap\caddy.log`

## Troubleshooting
- DNS: ensure `piwpiwsnap.site` A record points to your VPS IP
- Firewall: ports 80 and 443 must be open on the VPS and any provider network
- Certificates: Caddy needs an open 80/443 and a resolvable public DNS to issue TLS
- API: If you don’t proxy at `/api`, set `VITE_API_URL` to your API URL and rebuild
