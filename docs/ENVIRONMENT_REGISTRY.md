# Environment Registry

> **⚠️ MUST UPDATE when any environment, deployment, or tech stack changes**
> This document tracks ALL environments - update on EVERY architecture PR

## How to Use

1. **Before submitting a PR**: Update this document if your changes affect any environment
2. **In PR description**: Reference affected environments using the Environment column
3. **CI checks**: Validates environment consistency

---

## Production Environments

| Service | URL | Platform | Database | Tech Stack | Owner |
|---------|-----|----------|----------|------------|-------|
| **PropRoo Frontend** | https://proproo.sensibleanalytics.co | Cloudflare Workers | DuckDB WASM | React 19, Vite, Tailwind 4 | @rprabhat |
| **PropRoo Backend** | https://proproo-api.sensibleanalytics.co | Cloudflare Workers | DuckDB | Python FastAPI | @rprabhat |
| **Folio App** | https://folio.sensibleanalytics.co | Tauri/Docker | Local SQLite | Rust, React | @rprabhat |
| **Qullamaggie Scanner** | https://qullamaggie.sensibleanalytics.co | GitHub Pages | N/A (static) | React, Python | @rprabhat |
| **Rentroo Landlord** | https://landlord.rentroo.sensibleanalytics.co | Oracle Cloud | MongoDB Atlas | Node.js, React | @rprabhat |
| **Rentroo Tenant** | https://tenant.rentroo.sensibleanalytics.co | Oracle Cloud | MongoDB Atlas | Node.js, React | @rprabhat |
| **Rentroo API** | https://api.rentroo.sensibleanalytics.co | Oracle Cloud/GCP | MongoDB + Redis | Node.js microservices | @rprabhat |
| **Crewcircle** | https://crewcircle.sensibleanalytics.co | Vercel | TBD | React | @rprabhat |
| **CardScanner App** | App Store/Play Store | Mobile (iOS/Android) | Local storage | React Native | @rprabhat |

---

## Staging Environments

| Service | URL | Platform | Database | Tech Stack |
|---------|-----|----------|----------|-------------|
| PropRoo Staging | https://staging.proproo.sensibleanalytics.co | Cloudflare Workers | DuckDB (staging) | Same as prod |
| Rentroo Staging | https://staging.rentroo.sensibleanalytics.co | Oracle Cloud | MongoDB (staging) | Same as prod |

---

## Development Environments

| Service | Local URL | Database | Notes |
|---------|-----------|----------|-------|
| PropRoo Frontend | http://localhost:5173 | DuckDB WASM | `npm run dev` |
| PropRoo Backend | http://localhost:8000 | DuckDB local | `uvicorn main:app --reload` |
| Folio | http://localhost:1420 | Local | `cargo tauri dev` |
| Qullamaggie | http://localhost:3000 | In-memory | `python -m uvicorn` |
| Rentroo | http://localhost:8080 | MongoDB local | Docker compose |
| Crewcircle | http://localhost:3000 | TBD | `npm run dev` |

---

## Database Services

| Database | Used By | Connection |
|----------|---------|------------|
| **MongoDB Atlas** | Rentroo | `mongodb+srv://...` |
| **DuckDB WASM** | PropRoo (browser) | Client-side |
| **PostgreSQL** | Folio (Docker) | `postgres://...` |
| **Redis** | Rentroo cache | Upstash / Redis OSS |
| **Local Storage** | CardScanner | Device local |

---

## External Services

| Service | Provider | Purpose |
|---------|----------|---------|
| **Email** | SendGrid/Postmark | Transactional email |
| **SMS** | Twilio | Notifications |
| **Storage** | Cloudflare R2 / S3 | File storage |
| **Auth** | Clerk / Custom | Authentication |
| **Analytics** | Plausible / Mixpanel | Usage analytics |

---

## Updating This Document

### When to Update

Update IMMEDIATELY when your PR:
- Changes a deployment URL
- Adds/removes a database
- Switches to a different service
- Updates tech stack (new framework, language version)
- Adds new environment
- Changes environment configuration

### Format

```markdown
| Service | URL | Platform | Database | Tech Stack |
|---------|-----|----------|----------|------------|
| MyService | https://... | Cloudflare | PostgreSQL | Node.js 20 |
```

### PR Description Reference

In your PR, reference affected environments:

```markdown
## Environment Changes
- ✅ No environment changes
- OR
- ⚠️ PropRoo - updated backend URL
- ⚠️ Rentroo - added new Redis cache
```

---

## Auto-Validation

This document is validated in CI:

```yaml
# .github/workflows/env-consistency.yml
- name: Validate environment URLs
  run: |
    # Check URLs are valid format
    # Check databases are documented
    # Verify no hardcoded secrets
```

---

*Last updated: April 2026*
*Maintained by: All developers - update on every architecture PR!*