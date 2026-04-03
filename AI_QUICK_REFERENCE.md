# AI Agent Quick Reference Card

Quick commands and standards for any AI agent working on Sensible Analytics repos.

## 🚀 Common Commands

```bash
# Create feature branch
git checkout -b feat/your-feature

# Stage and commit
git add . && git commit -m "feat: description"

# Create PR
gh pr create --title "feat: description" --body "What it does"

# Run tests
pnpm test

# Run linting
pnpm lint
```

## ❌ Never Do

- Push to `main`/`master` directly
- Commit secrets/API keys
- Leave `.env` files in git
- Use `any` type in TypeScript
- Skip tests for "quick fixes"

## ✅ Always Do

- Use PR workflow
- Write tests for new features
- Run lint before commit
- Use environment variables for secrets
- Put docs in `.doc/` folder

## 📁 Required Folder Structure

```
repo/
├── .doc/           # Intermediate docs (REQUIRED)
│   ├── planning/
│   ├── specs/
│   └── notes/
├── src/            # Source code
├── tests/          # Test files
├── .github/        # CI/CD configs
└── docs/           # Public docs
```

## 🌍 Environment Variables

Always use these patterns:

```bash
# Development
export NODE_ENV=development

# API Keys (never hardcode)
export OPENAI_API_KEY="sk-..."
export DATABASE_URL="postgresql://..."
```

## 🔐 Security First

Before ANY commit:
1. Check no keys in diff
2. Verify env vars are external
3. Run security scan if available

---

**For full context, see:** `.github/AI_AGENT_SYSTEM_PROMPT.md`