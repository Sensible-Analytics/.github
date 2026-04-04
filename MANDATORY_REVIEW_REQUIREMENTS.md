# Mandatory Reviewer Requirements

> **Purpose**: Enforce mandatory code review for all PRs in Sensible Analytics org
> **Audience**: Developers, AI agents, repository maintainers
> **Last Updated**: April 2026

---

## 1. Organization Ruleset (Enterprise)

### Create Org-Wide Ruleset

1. Go to **Organization Settings → Repository → Rulesets**
2. Click **New ruleset**
3. Configure:

| Setting | Value |
|---------|-------|
| **Name** | `Sensible-Analytics Default Protection` |
| **Target** | All repositories |
| **Status** | Active |

### Required Rules

```yaml
rules:
  - type: require_code_owner_reviews
  - type: required_reviewers
    min: 1
  - type: delete_head_branches
  - type: required_signatures
```

### Conditions

```yaml
conditions:
  - type: ref_name
    pattern: "main"
  - type: ref_name
    pattern: "release/*"
```

---

## 2. Repository-Level Branch Protection

### Required Settings

| Setting | Required Value |
|---------|---------------|
| **Require pull request** | ✅ Yes |
| **Require approvals** | 1 (minimum) |
| **Require review from Code Owners** | ✅ Yes |
| **Dismiss stale approvals** | ✅ Yes |
| **Do not allow bypassing** | ✅ Yes |
| **Require status checks** | ✅ CI passing |
| **Require signed commits** | For production |

### Apply via GitHub CLI

```bash
# For each repo, apply branch protection
gh api repos/Sensible-Analytics/{repo}/protection \
  -X PUT \
  -H "Accept: application/vnd.github.v3+json" \
  -d '{
    "required_status_checks": null,
    "enforce_admins": true,
    "required_review_requests": [
      {
        "type": "ReviewRequest",
        "dismiss_stale_reviews": true,
        "required_approving_review_count": 1,
        "require_code_owner_reviews": true
      }
    ],
    "restrictions": null,
    "require_signed_commits": false
  }'
```

---

## 3. CODEOWNERS File

### Standard CODEOWNERS for Sensible Analytics

Create `.github/CODEOWNERS` in each repository:

```text
# Default: Senior Engineers
*                   @sensible-analytics/lead

# Frontend (React/TypeScript)
frontend/src/       @sensible-analytics/frontend
src/components/    @sensible-analytics/frontend
*.tsx              @sensible-analytics/frontend

# Backend (API, Services)
src/api/           @sensible-analytics/backend
src/services/      @sensible-analytics/backend

# Infrastructure
src/infrastructure/  @sensible-analytics/devops
src/database/        @sensible-analytics/devops
Terraform/           @sensible-analytics/devops

# Security-sensitive code
src/auth/           @sensible-analytics/security
src/payment/        @sensible-analytics/security
/security/          @sensible-analytics/security

# Documentation
docs/              @sensible-analytics/docs

# GitHub configurations
.github/           @sensible-analytics/lead
CODEOWNERS         @sensible-analytics/lead
```

---

## 4. Team Structure

### Required Teams

| Team | Purpose | Members |
|------|---------|---------|
| `@sensible-analytics/lead` | Senior engineers, org admins | Full access |
| `@sensible-analytics/frontend` | UI/UX reviews | React specialists |
| `@sensible-analytics/backend` | API reviews | Backend developers |
| `@sensible-analytics/devops` | Infrastructure | DevOps engineers |
| `@sensible-analytics/security` | Security-critical code | Security leads |
| `@sensible-analytics/docs` | Documentation | Technical writers |

### Create Teams

```bash
# Create teams via GitHub CLI
gh api --method POST orgs/Sensible-Analytics/teams \
  -f name="frontend" \
  -f description="Frontend reviews" \
  -f privacy="closed"

gh api --method POST orgs/Sensible-Analytics/teams \
  -f name="backend" \
  -f description="Backend reviews" \
  -f privacy="closed"

# ... etc for other teams
```

---

## 5. Review Workflow for AI Agents

### Required Steps Before Merge

For AI agents (and humans), every PR must:

1. **✅ Pass all CI checks** — Tests, linting, type checking
2. **✅ Get minimum 1 approval** — From CODEOWNER
3. **✅ Pass architecture tests** — ArchUnitTS, boundaries
4. **✅ No architectural violations** — Clean layers, no cycles
5. **✅ Address all review comments** — Resolve or respond

### AI Agent PR Template

```markdown
## PR Description
<!-- What does this PR do? -->

## Architecture Impact
- [ ] Architecture tests pass
- [ ] No layer violations
- [ ] No circular dependencies

## Review Checklist
- [ ] CODEOWNER approval obtained
- [ ] CI checks passing
- [ ] Tests added/updated
- [ ] Documentation updated

## Notes for Reviewers
<!-- Any context reviewers should know -->
```

---

## 6. Enforcement Actions

### When Review Requirements Not Met

| Condition | Action |
|-----------|--------|
| No approvals | Block merge |
| CODEOWNER not reviewed | Block merge |
| CI failing | Block merge |
| Architecture violations | Block merge |
| Stale review | Request re-review |

### GitHub Branch Protection API

```bash
# Check protection status
gh api repos/Sensible-Analytics/{repo}/protection

# Enable protection
gh api -X PUT repos/Sensible-Analytics/{repo}/protection \
  -F require_reviewers=true \
  -F required_approving_review_count=1 \
  -F require_code_owner_reviews=true \
  -F dismiss_stale_reviews=true \
  -F require_up_to_date_branches=true
```

---

## 7. Quick Reference Card

```
MANDATORY REVIEW CHECKLIST
===========================
BEFORE MERGE:
[ ] 1+ approval from CODEOWNER
[ ] CI checks passing
[ ] Architecture tests pass
[ ] No review comments pending

CODEOWNER RESPONSIBILITIES:
- Review within 24 hours
- Test the changes locally if needed
- Verify no security issues
- Check architecture compliance

BLOCK IF:
- No CODEOWNER approval
- CI failing
- Architecture violations found
- Security concerns not addressed
===========================
```

---

## Related Documentation

- [ARCHITECTURAL_GUARDRAILS.md](./ARCHITECTURAL_GUARDRAILS.md)
- [AI_CODE_GENERATION_STANDARDS.md](./AI_CODE_GENERATION_STANDARDS.md)
- [AGENT_WORKFLOW.md](./AGENT_WORKFLOW.md)

---

*All code changes in Sensible Analytics require review before merging. This ensures quality, security, and architectural compliance.*