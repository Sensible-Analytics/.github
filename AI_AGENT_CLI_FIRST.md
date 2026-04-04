# AI Agent CLI-First Guidelines

> **Purpose**: AI agents should prefer CLI/terminal over UI dashboards
> **Audience**: AI agents working in Sensible Analytics repositories
> **Last Updated**: April 2026

---

## Core Principle

**Use CLI tools for all operations where possible.** UI dashboards are for humans, not AI agents.

---

## Deployment Status

| Instead of | Use |
|------------|-----|
| Vercel dashboard | `vercel --prod` CLI, check via `vercel inspect` |
| Netlify dashboard | `netlify deploy --prod` |
| Railway dashboard | `railway status` |
| AWS console | AWS CLI / `aws` commands |
| Heroku dashboard | `heroku ps` / `heroku logs` |

### Vercel Example

```bash
# Check deployment status
vercel ls

# Get deployment URL
vercel inspect <deployment-id>

# View logs
vercel logs <deployment-url>
```

---

## Database Operations

| Instead of | Use |
|------------|-----|
| Neon console | `psql` with connection string |
| pgAdmin | `psql` / `pg_dump` commands |
| MongoDB Atlas UI | `mongosh` CLI |

---

## GitHub Operations

| Instead of | Use |
|------------|-----|
| GitHub PR UI | `gh pr list`, `gh pr view`, `gh pr merge` |
| GitHub Issues UI | `gh issue list`, `gh issue create` |
| GitHub Actions UI | `gh run list`, `gh run view` |

### GitHub CLI Examples

```bash
# List open PRs
gh pr list --repo Sensible-Analytics/PropRoo

# Check workflow runs
gh run list --repo Sensible-Analytics/PropRoo

# View deployment status
gh api repos/Sensible-Analytics/PropRoo/deployments
```

---

## Cloud Operations

| Instead of | Use |
|------------|-----|
| GCP Console | `gcloud` CLI |
| AWS Console | `aws` CLI |
| Azure Portal | `az` CLI |
| Docker Hub UI | `docker` CLI |

---

## Why CLI-First

1. **Scriptable** — Can be automated, UI cannot
2. **Reproducible** — Same result every time
3. **Audit trail** — Commands logged in shell history
4. **Faster** — No browser loading, no click fatigue
5. **AI-optimized** — Text output easy to parse

---

## When to Use UI

- Only when CLI doesn't exist (rare)
- Visual inspection for layout/design verification
- Actions requiring 2FA that CLI can't handle

---

## Quick Reference

```bash
# Always prefer these over UI:
vercel --help              # Deployment
gh --help                  # GitHub
aws --help                 # AWS
gcloud --help              # GCP
railway --help             # Railway
heroku --help              # Heroku
kubectl --help             # Kubernetes
docker --help              # Docker
```

---

*AI agents should never need to open a browser for routine operations.*