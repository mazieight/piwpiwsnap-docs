# Deploy Multiple Sites to Windows VPS with HTTPS

This guide helps you deploy **3 websites** (`piwpiwsnap.site`, `piwpiw.site`, and `saadla.site`) on the same Windows VPS using Caddy with automatic HTTPS.

## Architecture Overview

- **Caddy** serves all 3 sites on ports 80/443 with automatic SSL
- Each site has its own:
  - Directory: `C:\web\{sitename}\dist`
  - API backend port (3001, 3002, 3003)
  - Log file: `C:\web\{sitename}\caddy.log`

## Prerequisites

- Windows Server/VPS with RDP access (Administrator)
- Node.js LTS installed (https://nodejs.org)
- DNS A records for all 3 domains pointing to your VPS public IP:
  - `piwpiwsnap.site` → VPS IP
  - `piwpiw.site` → VPS IP
  - `saadla.site` → VPS IP

## Quick Start Deployment

### Deploy All 3 Sites at Once

```powershell
# Open PowerShell as Administrator
Set-ExecutionPolicy Bypass -Scope Process -Force
cd "C:\Users\PROJECT_USER\The world snap\website"
./deploy/deploy-multi-site.ps1
```

### Deploy a Single Site

```powershell
# Deploy only piwpiwsnap.site
./deploy/deploy-multi-site.ps1 -Site piwpiwsnap

# Deploy only piwpiw.site
./deploy/deploy-multi-site.ps1 -Site piwpiw

# Deploy only saadla.site
./deploy/deploy-multi-site.ps1 -Site saadla
```

## What the Script Does

1. ✅ Downloads and installs Caddy (if not present)
2. ✅ Builds your React/Vite project (`npm run build`)
3. ✅ Copies `dist/` to each site's directory
4. ✅ Configures Caddy to serve all 3 sites
5. ✅ Opens firewall ports 80 and 443
6. ✅ Installs Caddy as a Windows service
7. ✅ Auto-provisions HTTPS certificates for all domains

## Site Configuration

| Site              | Directory              | Backend Port | API Endpoint            |
|-------------------|------------------------|--------------|-------------------------|
| piwpiwsnap.site   | C:\web\piwpiwsnap\dist | 3001         | 127.0.0.1:3001          |
| piwpiw.site       | C:\web\piwpiw\dist     | 3002         | 127.0.0.1:3002          |
| saadla.site       | C:\web\saadla\dist     | 3003         | 127.0.0.1:3003          |

## Caddyfile Structure

The `Caddyfile` (`C:\web\Caddyfile`) contains:
- All 3 site configurations
- Reverse proxy for `/api/*` to respective backend ports
- SPA fallback to `index.html`
- Automatic HTTPS via Let's Encrypt

## Updating a Site

After making changes to your code:

```powershell
# Rebuild and redeploy all sites
./deploy/deploy-multi-site.ps1

# Or redeploy just one site
./deploy/deploy-multi-site.ps1 -Site piwpiwsnap
```

## Backend API Setup

Each site proxies `/api/*` to a local backend. Make sure your backend servers are running:

```powershell
# Example: Start backend for piwpiwsnap on port 3001
cd C:\backend\piwpiwsnap
node server.js

# Example: Start backend for piwpiw on port 3002
cd C:\backend\piwpiw
node server.js

# Example: Start backend for saadla on port 3003
cd C:\backend\saadla
node server.js
```

**Tip:** Use a process manager like `pm2` or `NSSM` to run backends as Windows services.

## Managing Caddy

```powershell
# Check Caddy status
& C:\web\caddy\caddy.exe status

# Reload configuration (after editing Caddyfile)
& C:\web\caddy\caddy.exe reload --config C:\web\Caddyfile

# Stop Caddy
& C:\web\caddy\caddy.exe stop

# Start Caddy
& C:\web\caddy\caddy.exe start

# View Caddy service logs (Windows Event Viewer)
Get-EventLog -LogName Application -Source Caddy -Newest 50
```

## Log Files

- **piwpiwsnap.site**: `C:\web\piwpiwsnap\caddy.log`
- **piwpiw.site**: `C:\web\piwpiw\caddy.log`
- **saadla.site**: `C:\web\saadla\caddy.log`

View logs:
```powershell
Get-Content C:\web\piwpiwsnap\caddy.log -Tail 50
```

## Troubleshooting

### Sites Not Working After Deployment

1. **Check DNS**: Ensure all 3 domains resolve to your VPS IP
   ```powershell
   nslookup piwpiwsnap.site
   nslookup piwpiw.site
   nslookup saadla.site
   ```

2. **Check Caddy is Running**:
   ```powershell
   & C:\web\caddy\caddy.exe status
   ```

3. **Check Firewall**:
   ```powershell
   netsh advfirewall firewall show rule name="Caddy HTTP"
   netsh advfirewall firewall show rule name="Caddy HTTPS"
   ```

4. **Test Locally**:
   ```powershell
   # Test if site files exist
   Test-Path C:\web\piwpiwsnap\dist\index.html
   Test-Path C:\web\piwpiw\dist\index.html
   Test-Path C:\web\saadla\dist\index.html
   ```

5. **Check Caddyfile Syntax**:
   ```powershell
   & C:\web\caddy\caddy.exe validate --config C:\web\Caddyfile
   ```

6. **Restart Caddy**:
   ```powershell
   & C:\web\caddy\caddy.exe stop
   & C:\web\caddy\caddy.exe start
   ```

### HTTPS Certificate Issues

- Caddy needs ports 80 and 443 open to obtain Let's Encrypt certificates
- DNS must resolve to your VPS before certificates can be issued
- Check logs for certificate errors: `Get-Content C:\web\piwpiwsnap\caddy.log`

### API Not Working

- Verify backend is running on the correct port
- Check API calls in browser DevTools (Network tab)
- Test backend directly: `curl http://127.0.0.1:3001/api/test`

## Migration from Single Site

If you previously had only `piwpiwsnap.site` working:

1. ✅ Updated `Caddyfile` with all 3 sites
2. ✅ Run `./deploy/deploy-multi-site.ps1` to deploy all sites
3. ✅ Ensure DNS A records exist for all domains
4. ✅ Start backend APIs on ports 3001, 3002, 3003

## Adding More Sites

To add a 4th site:

1. Edit `deploy/Caddyfile` - add new site block
2. Edit `deploy/deploy-multi-site.ps1` - add site to `$Sites` hashtable
3. Redeploy: `./deploy/deploy-multi-site.ps1`

## Support

- **Caddy Docs**: https://caddyserver.com/docs/
- **Let's Encrypt**: https://letsencrypt.org/
