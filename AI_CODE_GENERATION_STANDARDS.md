# AI Code Generation Standards

> **Purpose**: Engineering standards for AI agents generating production-ready code at Sensible Analytics.
> **Audience**: AI agents, developers, and reviewers working with AI-assisted code generation.
> **Last Updated**: April 2026

---

## 1. Engineering Standards for AI-Generated Code

### Context Engineering Over Vibe Coding

The industry has moved from unstructured "vibe coding" to **context engineering** — carefully structuring how AI models are used to get consistently excellent results. Context engineering means providing the AI with your project's architectural decisions, coding standards, existing patterns, and specific constraints before it writes a single line.

### Agentic Workflow Architecture

Production-ready agentic workflows have five sequential components:

| Component | Description |
|-----------|-------------|
| **Task Input** | Structured description with acceptance criteria, scope constraints, context |
| **Context Assembly** | Agent pulls from codebase, docs, tickets, business intent |
| **Sandbox Execution** | Isolated environment with full codebase access |
| **PR Output** | Quality directly proportional to input quality |
| **Review** | AI review pass first, then human review for architectural decisions |

### Configuration as Code

Treat AI CLI configuration like production code — version control it, review it in PRs, keep it consistent across the team.

```markdown
# CLAUDE.md structure example
.claude/
  skills/
    payment-processing.md    # Domain knowledge
    auth-patterns.md         # Authentication conventions
    database-migrations.md   # Schema change patterns
    testing-strategy.md      # Coverage targets, what to mock
```

**Configuration Rules:**
- Keep to ~100-150 custom rules focused on what the model can't infer
- Use `file:line` references instead of code snippets (snippets go stale)
- Delegate code style to linters, not LLMs
- Describe the "why" not just the "what"

---

## 2. Test Coverage Requirements

### Coverage Thresholds

AI-generated code requires **higher** test coverage than human-written code, not lower:

| Coverage Type | Minimum | Target for Critical Paths |
|---------------|---------|---------------------------|
| **Line coverage** | 80% | 90-100% |
| **Branch coverage** | 75% | 85%+ |

> **Critical**: AI code contains 1.7x more issues than human-written code. Comprehensive tests are your only reliable defense against hidden failures.

### Test Types That Catch AI Bugs

AI-generated code exhibits predictable failure patterns. Tailor testing approaches:

**Integration tests over unit tests** — AI excels at isolated functions but struggles with:
- Database queries that lock under concurrent access
- API calls assuming synchronous responses
- File operations failing across different OS environments

**Property-based testing for edge cases** — Generate hundreds of randomized inputs to validate invariants

**Error path testing** — AI optimizes for happy path. Explicitly test:
- Network failures (timeouts, DNS failures, TLS errors)
- Resource exhaustion (OOM, disk full, connection pool depletion)
- Invalid input (null values, type mismatches, size violations)
- Concurrent access (race conditions, deadlocks)

### Coverage Enforcement in CI/CD

```javascript
// jest.config.js — fail builds below threshold
{
  "coverageThreshold": {
    "global": {
      "branches": 75,
      "functions": 80,
      "lines": 80,
      "statements": 80
    }
  }
}
```

**Differential Requirements:**
- New AI-generated files: 90%+ coverage required
- Modified files: coverage increase on changed lines
- Critical paths (auth, payments): maintain existing high coverage

---

## 3. Performance Optimization Guidelines

### Common AI Performance Pitfalls

AI-generated code produces **8x more excessive I/O operations** than human-written equivalents.

**N+1 Query Pattern (AI's Favorite Mistake)**
```typescript
// AI loves this pattern (N+1 queries) — AVOID
const users = await prisma.user.findMany();
const usersWithPosts = await Promise.all(
  users.map(async (user) => ({
    ...user,
    posts: await prisma.post.findMany({ where: { authorId: user.id } }),
  }))
);

// What it should be (1 query)
const usersWithPosts = await prisma.user.findMany({
  include: { posts: true },
});
```

### Performance Checklist

| Category | Check |
|----------|-------|
| **Queries** | No N+1 patterns — use `include`/`join` instead of loops |
| **Database** | Appropriate indexes, paginated for large datasets |
| **Async** | `Promise.all` for independent operations |
| **UI** | No blocking operations in request handlers |
| **Payloads** | No over-fetching, reasonable response sizes |

---

## 4. Making Architectural Decisions Visible

### The Explain-It Test

Can you explain every line to a teammate without saying "the AI wrote it"? If any section makes you pause, that section needs rewriting. **You own every line that ships, regardless of who generated it.**

### Documentation Requirements

AI-generated code should include:

1. **Why decisions were made** — not what the code does (that's obvious from reading it)
2. **Business context** — the reason behind a particular approach
3. **Assumptions** — what the code assumes about inputs, state, environment
4. **Failure modes** — what happens when things go wrong and how the system recovers

### Architecture Decision Records (ADRs)

Maintain ADRs for significant decisions:

```markdown
# ADR-001: Chose PostgreSQL over MongoDB for user data

## Status
Accepted

## Context
We need to store user authentication data with complex relational queries.

## Decision
PostgreSQL with Prisma ORM.

## Consequences
- Positive: Strong consistency, complex query support
- Negative: Requires migration management, schema versioning

## Generated Code Alignment
AI-generated data access follows patterns in src/services/UserService.ts:1-45
```

---

## 5. Code Review Standards for AI-Generated Code

### Why AI Code Review Is Different

AI-generated code is **always well-formatted** — this tricks reviewers into skimming. The problems hide in untested edge cases, error paths, and assumptions.

> **Warning**: 72% of developers report they review AI code less carefully than human-written code.

### Red Flags in AI-Generated Code

| What You See | What You Miss |
|--------------|---------------|
| Clean formatting | Null/undefined not handled in critical paths |
| Reasonable names | Race conditions in async operations |
| Error handling with try/catch | Wrong assumptions about your data model |
| Correct output for simple tests | Generic error handling that swallows specifics |

### The 10-Point AI Code Review Checklist

**Critical (Check Every PR):**
- [ ] No hardcoded secrets, API keys, or credentials
- [ ] Authentication check present on all protected endpoints
- [ ] Authorization check present (not just authentication)
- [ ] User input validated and sanitized before use
- [ ] No SQL/NoSQL injection vectors (parameterized queries only)
- [ ] Error responses don't leak internal details
- [ ] Sensitive data not logged

**Architecture (Check When Adding New Files):**
- [ ] Follows existing project error handling pattern
- [ ] Uses established data access layer
- [ ] Imports use project aliases (@/) not deep relative paths
- [ ] No new dependencies added without team approval
- [ ] File placed in correct directory per project structure
- [ ] No duplicate logic (search for similar functions)

**Edge Cases (Check When Touching Business Logic):**
- [ ] Null/undefined inputs handled explicitly
- [ ] Empty arrays and empty strings considered
- [ ] Boundary conditions tested (exactly at limit, one above, one below)
- [ ] Network failure handling present
- [ ] Concurrent access considered (race conditions)
- [ ] Errors propagated correctly (not swallowed)
- [ ] Partial failure scenarios handled

**Performance (Check When Touching Data Layer):**
- [ ] No N+1 query patterns
- [ ] Database queries use appropriate indexes
- [ ] Large datasets paginated
- [ ] Promise.all used for independent async operations
- [ ] No blocking operations in request handlers
- [ ] Response payload size reasonable

### Team Review Practices

| Practice | Implementation |
|----------|---------------|
| **Label AI-generated PRs** | Use tags like `ai-assisted` or `copilot-generated` |
| **Require boundary tests** | Every AI function needs null/empty/error tests |
| **Rotate reviewers** | Prevent familiarity bias |
| **Monthly architecture reviews** | Catch drift before it compounds |
| **Track AI bug rates separately** | Measure AI vs human code quality |

---

## 6. Performance Integration for AI Agents

### The Seven-Component Prompt Framework

Effective system prompts must include:

1. **Role Definition** - Identity, expertise, persona
2. **Core Instructions** - Primary task with priority ordering
3. **Constraints/Guardrails** - Positive directives (do X) over negative (don't do Y)
4. **Output Format** - Explicit structure markers
5. **Tool Instructions** - When/why/how to use each tool
6. **Few-Shot Examples** - 1-3 input/output pairs
7. **Error Handling** - Fallback behavior

### Performance-Specific Instructions

Include explicit performance requirements in constraints:

```xml
<constraints>
- Target O(n log n) or better for list operations
- Use streaming for responses over 1000 tokens
- Cache repeated computations
- Lazy-load large data structures
- Respond to API calls within 200ms where applicable
</constraints>
```

### Model Tiering Strategy

Route tasks to appropriate model tiers based on complexity:

| Task Type | Model | Use Case |
|-----------|-------|----------|
| Quick Fix | Haiku | Typos, formatting, simple edits |
| Standard Dev | Sonnet | Feature implementation, bug fixes |
| Complex Analysis | Opus | Architecture decisions, refactoring |

---

## 7. Benchmarking AI-Generated Code

### Key Metrics for AI Code Health

| Metric | Healthy Target | Warning Range |
|--------|---------------|---------------|
| AI Code Share % | 40-50% | <40% or >50% |
| Cycle Time Lift | 18-24% faster | <18% |
| Tool Effectiveness | Cursor: 46%, Copilot: 32% | Varies by use case |
| Defect Risk | <5% increase | >5% increase |

### The COMPASS Multi-Dimensional Benchmark

Evaluate AI code across three independent dimensions:

| Dimension | What It Measures |
|-----------|------------------|
| **Correctness** | Does code pass tests? |
| **Efficiency** | Does it scale under load? |
| **Quality** | Is code maintainable? |

> **Key Finding**: High correctness ≠ efficient code. Models with highest correctness often have poor efficiency scores.

---

## Quick Reference Card

```
AI CODE GENERATION QUICK CHECK
===============================
ENGINEERING:
[ ] Context structured before coding
[ ] Configuration as code followed

TESTING:
[ ] 90%+ coverage on new AI files
[ ] Integration + error path tests included

PERFORMANCE:
[ ] No N+1 query patterns
[ ] Appropriate indexes present

ARCHITECTURE:
[ ] Explain-it test passed
[ ] ADRs maintained for decisions

REVIEW:
[ ] Auth + Authorization verified
[ ] Input validation checked
[ ] Edge cases tested
[ ] No hardcoded secrets
===============================
If any "no": request changes before merge.
```

---

## Related Documentation

- [AI_AGENT_DEVELOPMENT_METHODOLOGY.md](./AI_AGENT_DEVELOPMENT_METHODOLOGY.md)
- [AI_AGENT_SYSTEM_PROMPT.md](./AI_AGENT_SYSTEM_PROMPT.md)
- [AGENT_WORKFLOW.md](./AGENT_WORKFLOW.md)
- [AI_QUICK_REFERENCE.md](./AI_QUICK_REFERENCE.md)

---

*These standards ensure AI-generated code at Sensible Analytics meets production quality requirements while maintaining security, performance, and maintainability.*