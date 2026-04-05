# Agent Workflow Guide

**Purpose**: Step-by-step instructions for AI agents using Sensible Analytics workflows

---

## 🚀 Quick Start

1. Load the `sensible-standards` skill: `skill(name="sensible-standards")`
2. Check current branch: `git status`
3. Always work on a feature branch

---

## 📋 Daily Workflow

### 1. Start of Session
```bash
git fetch origin
git pull origin main
```

### 2. Create Worktree for Task
```bash
git worktree add -b feat/your-feature ../project-feature main
cd ../project-feature
```

### 3. Make Changes
```bash
# Make your changes
npm run lint && npm run test && npm run build  # Validate
git add . && git commit -m "feat(scope): description"
```

### 4. End of Session
```bash
git push origin feat/your-feature
gh pr create --title "feat: description" --body "What it does"
```

### 5. Post-Merge Cleanup
```bash
git worktree remove ../project-feature
git push origin --delete feat/your-feature
```

---

## 🔧 GitHub Agentic Workflows (gh-aw)

### Setup
```bash
# Install gh-aw (if not already installed)
gh extension install github/gh-aw

# Initialize for a repo
gh aw init

# List available workflows
gh aw list
```

### Key Commands

| Command | Purpose |
|---------|---------|
| `gh aw init` | Set up new workflow in repo |
| `gh aw new <name>` | Create new workflow |
| `gh aw compile` | Compile markdown workflows to Actions YAML |
| `gh aw run <workflow>` | Execute a workflow |
| `gh aw logs <workflow>` | View execution logs |
| `gh aw audit <run-id>` | Debug failed runs |

---

## 📝 AI Agent Workflow Pattern

```markdown
---                    # Trigger section
on:
  issues:
    types: [opened]
---
# Agent workflow for Sensible Analytics

## Task
Analyze the issue and:
1. Add appropriate labels
2. Identify duplicates
3. Create follow-up tasks if needed
4. Assign to relevant team

## AI Disclosure
- AI-Generated: Yes
- Model: [e.g., Claude Sonnet 4.6]
- Platform: OpenCode
- Human Oversight: Fully reviewed
```

---

## 🔒 Safe Outputs

gh-aw supports validated write operations:
```markdown
safe-outputs:
  add-comment:
  create-issue:
    labels: [ agent-task ]
```

---

## 🧹 Post-Merge Cleanup

After PR is merged:
```bash
git checkout main
git pull origin main
git worktree remove ../project-feature
git branch -d feat/your-feature
git push origin --delete feat/your-feature
```

---

*Last updated: 2026-04-05*
