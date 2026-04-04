# Sensible Analytics Agent Workflow

**ALL branches (main/master) are PROTECTED.** You MUST follow this workflow for every change.

---

## 🚫 NEVER Do

- ❌ Push directly to `main` or `master`
- ❌ Commit directly to protected branches
- ❌ Use `git push --force` on protected branches

## ✅ Required Workflow

### Step 1: Create Feature Branch
```bash
git checkout -b feat/your-feature-name
# or
git checkout -b fix/issue-description
```

### Step 2: Make Changes & Commit
```bash
git add .
git commit -m "feat: descriptive commit message"
```

### Step 3: Push Branch
```bash
git push origin feat/your-feature-name
```

### Step 4: Create Pull Request
```bash
gh pr create --title "feat: Add new feature" --body "Description of changes"
```

### Step 5: Wait for CI Checks
- Run any tests required
- Ensure linting passes
- Wait for all checks to complete

### Step 6: Merge After Review
```bash
gh pr merge --squash --delete-branch
```

---

## 📋 Branch Naming Conventions

| Prefix | Use For |
|--------|---------|
| `feat/` | New features |
| `fix/` | Bug fixes |
| `docs/` | Documentation |
| `refactor/` | Code refactoring |
| `test/` | Test additions |
| `chore/` | Maintenance |

---

## 🔄 Quick Reference

```bash
# Start new work
git checkout -b feat/new-feature

# Make changes
git add . && git commit -m "feat: add feature"

# Push and create PR
git push origin feat/new-feature
gh pr create --title "feat: add feature" --body "What it does"

# After approved
gh pr merge --squash --delete-branch

# Back to main
git checkout main && git pull
```

---

## 📁 Document Organization

- Intermediate documents → `.doc/` folder
- Use: `.doc/planning/`, `.doc/specs/`, `.doc/notes/`
- NEVER leave loose docs in root

---

**This workflow applies to ALL Sensible Analytics repos.**
See: `~/.sensible/agent-instructions/` for local reference.