# Sensible Analytics - Agent Instructions

**ALL Sensible Analytics repos follow these rules. Read this first.**

## ⚠️ CRITICAL: Load Sensible Standards First

Before any work, you MUST load the Sensible Analytics standards:

1. **Check local skill** (preferred):
   ```
   ~/.superset/hooks/opencode/skills/sensible-standards/SKILL.md
   ```

2. **If not available locally**, read:
   ```
   ~/Business/sensibleAnalytics/.github/AGENTS.md
   ```

## 📋 Branch Protection

**ALL Sensible repos have protected branches.**
- ❌ NEVER push to main/master directly
- ✅ ALWAYS create feature branch + PR

## 🏷️ Branch Names

| Prefix | Use For |
|--------|---------|
| `feat/` | New features |
| `fix/` | Bug fixes |
| `docs/` | Documentation |
| `refactor/` | Refactoring |
| `test/` | Tests |
| `chore/` | Maintenance |

## 🔀 Workflow

1. Create feature branch: `git checkout -b feat/your-feature`
2. Make changes & commit
3. Push & create PR: `gh pr create --title "feat: description" --body "..."`
4. Wait for CI → Merge

## 📋 Available Skills

This repo has access to engineering skills from `~/.config/opencode/skills/`.

### Intent → Skill Mapping

| Task Type | Skills to Use |
|-----------|---------------|
| New feature | `spec-driven-development` → `planning-and-task-breakdown` → `incremental-implementation` |
| Planning | `planning-and-task-breakdown` |
| Bug fix | `debugging-and-error-recovery` |
| Code review | `code-review-and-quality` |
| Refactoring | `code-simplification` |
| API design | `api-and-interface-design` |
| UI work | `frontend-ui-engineering` |
| Security | `security-and-hardening` |
| Performance | `performance-optimization` |
| Git/Version | `git-workflow-and-versioning` |
| CI/CD | `ci-cd-and-automation` |
| Deploy | `shipping-and-launch` |

### Core Rules

- If a task matches a skill, you MUST invoke it using the `skill` tool
- Never implement directly if a skill applies
- Always follow skill instructions exactly
- Verification is non-negotiable - provide evidence (tests pass, build works)

## 🧪 Testing Standards

- Python: `ruff check` + `pytest`
- TypeScript: `npm run lint` + `npm run build`

## 📦 Package Manager Patterns

| Type | Manager | Lock File |
|------|---------|-----------|
| npm | npm | package-lock.json |
| Python | pip | requirements.txt |
| Rust | cargo | Cargo.lock |

## 🎯 DSPy Component Design (For AI modules)

When building AI-powered components, read:
- `.github/docs/standards/dspy-component-template.md`
- `.github/docs/standards/smart-gl-dspy-breakdown.md`

Quick pattern:
```python
class ComponentInput(BaseModel):
    required_field: str
    
class ComponentOutput(BaseModel):
    result: str
    confidence: float = Field(ge=0.0, le=1.0)
    requires_review: bool
```

---

*Last updated: 2026-04-18*