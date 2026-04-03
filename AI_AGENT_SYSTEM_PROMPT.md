# AI Agent System Prompts - Sensible Analytics Organization

This file contains organization-wide prompts that should be used when working with any AI agent (OpenCode, Cursor, GitHub Copilot, etc.) on Sensible Analytics repositories.

## 🔑 Organization Context

**Industry:** Regulated industries (Healthcare, Financial Services, Property Management)
**Location:** Sydney, Australia
**Standards:** HIPAA, GDPR, SOC 2, Australian Privacy Act

## 🤖 Universal AI Agent Instructions

You are working on a Sensible Analytics repository. All our projects follow these organization-wide standards:

### 1. Branch Protection
- NEVER push directly to `main` or `master`
- All changes MUST go through Pull Requests
- Branch naming: `feat/`, `fix/`, `docs/`, `refactor/`, `test/`, `chore/`

### 2. Security-First Mindset
- NEVER commit API keys, secrets, or credentials
- All secrets MUST use environment variables
- Follow the AI_AGENT_KEYS_POLICY.md guidelines
- Report any security concerns immediately

### 3. Document Organization
- Intermediate documents go in `.doc/` folder
- Use folders: `.doc/planning/`, `.doc/specs/`, `.doc/notes/`
- NEVER leave loose documents in root

### 4. Testing Requirements
- All new features MUST have tests
- Run tests before submitting PR
- Maintain >80% test coverage

### 5. Type Safety
- Use TypeScript for all new code
- NEVER use `any` type
- Enable strict mode in tsconfig

### 6. Code Quality
- Run linting before commits
- Follow existing code patterns
- Write descriptive commit messages

## 📋 Per-Project Type Standards

### Next.js/TypeScript Projects
- Use App Router (`app/` directory)
- Server components by default
- Use React Server Components for data fetching
- Style with Tailwind CSS

### Python Projects
- Use type hints everywhere
- Follow PEP 8
- Use pydantic for data validation
- Include docstrings for all public functions

### Database Projects
- All schemas MUST have migrations
- Include rollback scripts
- Use foreign key constraints
- Add indexes for frequently queried columns

## 🔒 Security Checklist (Run Before Every PR)

- [ ] No API keys or secrets in code
- [ ] No hardcoded credentials
- [ ] Environment variables documented
- [ ] SQL injection prevented
- [ ] XSS protection in place
- [ ] CSRF tokens implemented
- [ ] Input validation on all endpoints

## 📝 Commit Message Format

```
type(scope): description

[optional body]
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

---

**Remember:** You represent Sensible Analytics. Write code that would pass security audit for HIPAA/GDPR compliance.