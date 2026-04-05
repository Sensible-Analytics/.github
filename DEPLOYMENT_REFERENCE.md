# Sensible Analytics - Deployment & Infrastructure Reference

> Centralized documentation for all CI/CD, deployment platforms, and infrastructure services
> **Last Updated**: April 2026

---

## 🎯 Deployment Platforms

| Project | Platform | Service | Configuration |
|---------|----------|---------|---------------|
| **PropRoo** | Cloudflare Workers | Frontend + Functions | `wrangler.toml`, `.wrangler/` |
| **Folio** | Docker Hub | Container Registry | `docker-publish.yml` |
| **Qullamaggie Scanner** | GitHub Pages | Static Hosting | `deploy-frontend.yml` |
| **Rentroo** | Oracle Cloud + GCP | VM/Containers | `deploy.yml` (OCI), `deploy-gcp.yml` (GCP) |
| **Crewcircle** | Vercel | Frontend | `vercel.json` |
| **CardScannerApp** | Apple App Store / Google Play | Mobile | `ios-build.yml`, `android-build.yml` |

---

## 🔧 CI/CD Tools (GitHub Actions)

### Common Workflows

| Workflow | Purpose |
|----------|---------|
| `ci.yml` | Build, lint, test on every PR |
| `security-codeql.yml` | CodeQL security scanning |
| `dependabot-auto-merge.yml` | Auto-merge dependency updates |
| `release.yml` | Release automation |
| `stale.yml` | Auto-close stale issues/PRs |

### Project-Specific Workflows

| Project | Workflows |
|---------|-----------|
| PropRoo | `deploy-frontend.yml`, `etl-pipeline.yml`, `guardrails.yml` |
| Rentroo | `deploy.yml` (OCI), `deploy-gcp.yml` (GCP), `codeql-analysis.yml`, `roam.yml` |
| CardScannerApp | `ios-build.yml`, `android-build.yml`, `release.yml` |
| Folio | `docker-publish.yml`, `nightly.yml`, `pr-check.yml` |
| Qullamaggie | `deploy-frontend.yml` (GitHub Pages) |
| Video Analysis | `build.yml` |

---

## 🗄️ Database & Data Services

| Project | Service | Type |
|---------|---------|------|
| PropRoo | DuckDB WASM | Browser-local database |
| Rentroo | MongoDB Atlas | Document database |
| Rentroo | Redis (Upstash) | Caching/session |
| Folio | Local/Embedded | SQLite-like |

### Connection Patterns

```bash
# MongoDB Atlas (Rentroo)
mongodb+srv://<user>:<password>@cluster.mongodb.net/rentroo

# Redis (Upstash)
rediss://<user>:<password>@<host>.upstash.io:6379
```

---

## ☁️ Cloud Providers

| Provider | Used By | Purpose |
|----------|---------|---------|
| **Cloudflare** | PropRoo | Workers, Pages, KV |
| **Oracle Cloud** | Rentroo | Compute (OCI Free Tier) |
| **GCP** | Rentroo | VM, containers |
| **GitHub** | All | Container Registry (ghcr.io) |
| **Vercel** | Crewcircle, Qullamaggie | Frontend hosting |
| **GitHub Pages** | Qullamaggie | Static hosting |

---

## 📦 Container Registry

| Registry | Usage |
|----------|-------|
| **GHCR** (ghcr.io) | All Docker builds push here |
| **Docker Hub** | Folio publishes to `sensible-folio` |

---

## 🚀 Quick Deploy Commands

### PropRoo (Cloudflare)
```bash
npm install -g wrangler
wrangler deploy
wrangler dev
```

### Rentroo (Oracle Cloud)
```bash
# Deploy via Render Blueprint
render-blueprint render.yaml

# Or GCP
cd terraform && terraform apply
```

### Folio (Docker)
```bash
docker build -t sensible-folio:latest .
docker push ghcr.io/sensible-analytics/folio:latest
```

### Qullamaggie (GitHub Pages)
```bash
# Auto-deploys on push to main
# No manual command needed
```

### Crewcircle (Vercel)
```bash
vercel --prod
vercel dev
```

---

## 🔐 Required Secrets

| Secret | Used By | Purpose |
|--------|---------|---------|
| `OCI_TENANCY_OCID` | Rentroo | Oracle Cloud authentication |
| `OCI_USER_OCID` | Rentroo | Oracle Cloud authentication |
| `OCI_FINGERPRINT` | Rentroo | Oracle Cloud authentication |
| `CF_API_TOKEN` | PropRoo | Cloudflare deployment |
| `DOCKERHUB_*` | Folio | Docker Hub push |

---

## 📁 Key Configuration Files

| File | Project | Purpose |
|------|---------|---------|
| `wrangler.toml` | PropRoo | Cloudflare Workers config |
| `render.yaml` | Rentroo | Render Blueprint |
| `docker-compose.yml` | Multiple | Local dev environment |
| `vercel.json` | Crewcircle, Qullamaggie | Vercel config |
| `render.yaml` | Rentroo | Render deployment |

---

## 🧪 Local Development

### Docker Compose
```bash
# Most projects support local Docker
docker-compose up -d

# Or for specific services
docker compose -f docker-compose.yml up
```

### Python Projects
```bash
# Virtual environment
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Run
python -m uvicorn main:app --reload
```

### Node.js Projects
```bash
npm install
npm run dev
```

---

## 📋 Environment Variables

| Variable | Project | Description |
|----------|---------|-------------|
| `DATABASE_URL` | Rentroo | MongoDB connection |
| `REDIS_URL` | Rentroo | Redis connection |
| `NODE_ENV` | All | development/production |
| `PORT` | All | Server port |

---

## 🔗 Useful Links

- **Cloudflare Dashboard**: https://dash.cloudflare.com
- **Oracle Cloud Console**: https://console.oraclecloud.com
- **GCP Console**: https://console.cloud.google.com
- **MongoDB Atlas**: https://cloud.mongodb.com
- **Vercel Dashboard**: https://vercel.com/dashboard
- **GitHub Packages**: https://github.com/orgs/Sensible-Analytics/packages

---

*This document is maintained at `.github/DEPLOYMENT_REFERENCE.md` in the Sensible Analytics organization.*