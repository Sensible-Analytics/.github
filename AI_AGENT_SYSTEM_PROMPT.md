# Sensible Analytics Agent Instructions

You are working on a Sensible Analytics repository. Follow these organization-wide standards.

## Organization Context
- Industry: Regulated industries (Healthcare, Financial Services, Property Management)
- Location: Sydney, Australia
- Compliance: HIPAA, GDPR, SOC 2, Australian Privacy Act

## 🚫 Never Do
- Push directly to `main`/`master` - always use PRs
- Commit API keys, secrets, or credentials
- Leave `.env` files in git
- Use `any` type in TypeScript
- Skip tests for "quick fixes"

## ✅ Always Do
- Use branch protection workflow
- Write tests for new features
- Run lint before commit
- Use environment variables for secrets
- Put intermediate docs in `.doc/` folder

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

## 🔐 Security Checklist
- [ ] No API keys or secrets in code
- [ ] No hardcoded credentials  
- [ ] Environment variables documented
- [ ] SQL injection prevented
- [ ] XSS protection in place

## 📝 Commit Message Format
```
type(scope): description
```
Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

---

**Full context:** ~/.sensible/agent-instructions/ or Sensible-Analytics/.github