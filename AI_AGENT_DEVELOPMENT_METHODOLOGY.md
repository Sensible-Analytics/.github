# AI Agent Development Methodology

**Purpose**: Standardize how AI agents develop code at Sensible Analytics
**Audience**: AI agents working on Sensible Analytics projects

---

## 🔴 Core Rules

1. **ALWAYS create feature branches** - never commit to main/master
2. **ALWAYS use PR workflow** - review before merge
3. **ALWAYS validate before commit** - lint + test + build
4. **ALWAYS use worktrees** for parallel tasks

---

## 🔀 Git Workflow (Trunk-Based Development)

### 1. Worktree Isolation (Recommended)
```bash
git worktree add -b fix/issue-422 ../project-issue-422 main
```

### 2. Small, Scoped Commits
- Commit after EVERY success
- One logical change per commit
- Use checkpoint commits before risky changes

### 3. Commit Message Format
```
<type>(<scope>): <subject>

Co-Authored-By: Claude <noreply@anthropic.com>
```

### 4. Pre-Commit Validation (MANDATORY)
```bash
npm run lint && npm run test && npm run build
```

### 5. PR Size Rule
- Target: < 200 lines changed
- If larger, split into multiple PRs

---

## 🤖 Architecture Patterns

### Coordinator-Worker Pattern (Recommended)
- One coordinator agent
- Multiple specialized worker agents
- Worktree isolation per task

### Supervisor-Subagent Pattern
- For complex multi-step tasks
- Clear handoff protocols

---

## ✅ Required Standards

- ✅ Use Pydantic for input/output validation
- ✅ Use dependency injection
- ✅ Add retry middleware
- ✅ Enable LangSmith tracing
- ✅ Test with TestModel override
- ✅ Implement human-in-the-loop for critical actions

---

## 🏗️ Architecture Tests

Each project MUST have architecture tests to enforce:
- Clean layer separation (presentation → application → domain)
- Hexagonal architecture (ports & adapters)
- No circular dependencies
- File size limits

---

## 🔧 GitHub Agentic Workflows (gh-aw)

Use `gh-aw` for AI agent task automation:

```bash
# Install
gh extension install github/gh-aw

# Initialize
gh aw init

# Run workflow
gh aw run my-workflow
```

---

*Last updated: 2026-04-05*
