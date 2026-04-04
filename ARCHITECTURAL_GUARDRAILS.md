# Architectural Guardrails for AI-Generated Code

> **Purpose**: Enforce architectural constraints on AI-generated code at Sensible Analytics
> **Audience**: AI agents, developers, and CI/CD pipelines
> **Last Updated**: April 2026

---

## 1. Tool Selection

| Use Case | Recommended Tool | Why |
|----------|-----------------|-----|
| **Test-based enforcement** | ArchUnitTS | Integrates with Jest/Vitest |
| **ESLint-native** | eslint-plugin-boundaries | Real-time feedback during dev |
| **CI/CD gate** | dependency-cruiser | Fast CLI-first |
| **Hexagonal architecture** | ArchUnitTS | Built-in presets |

### Installation

```bash
# ArchUnitTS (test-based)
npm install --save-dev archunitts

# eslint-plugin-boundaries (ESLint)
npm install --save-dev eslint-plugin-boundaries

# dependency-cruiser (CLI)
npm install --save-dev dependency-cruiser
```

---

## 2. Separation of Concerns: Clean Architecture Layers

### Layer Structure

```
src/
├── presentation/    # UI, Controllers, HTTP handlers
├── application/     # Use cases, Application services
├── domain/          # Entities, Value objects, Domain services
└── infrastructure/  # Database, External services, Adapters
```

### Dependency Rules (STRICT)

| Layer | Can Depend On |
|-------|---------------|
| `presentation` | application, domain |
| `application` | domain |
| `domain` | NOTHING external |
| `infrastructure` | domain |

---

## 3. ArchUnitTS Configuration

### Clean Architecture Tests

```typescript
// tests/architecture/clean-layers.test.ts
import { projectFiles } from 'archunit';

describe('Clean Architecture Layers', () => {
  it('domain should not depend on presentation', async () => {
    const rule = projectFiles()
      .inFolder('src/domain/**')
      .shouldNot()
      .dependOnFiles()
      .inFolder('src/presentation/**');
    await expect(rule).toPassAsync();
  });

  it('domain should not depend on infrastructure', async () => {
    const rule = projectFiles()
      .inFolder('src/domain/**')
      .shouldNot()
      .dependOnFiles()
      .inFolder('src/infrastructure/**');
    await expect(rule).toPassAsync();
  });

  it('application should not depend on presentation', async () => {
    const rule = projectFiles()
      .inFolder('src/application/**')
      .shouldNot()
      .dependOnFiles()
      .inFolder('src/presentation/**');
    await expect(rule).toPassAsync();
  });

  it('presentation should not depend on infrastructure', async () => {
    const rule = projectFiles()
      .inFolder('src/presentation/**')
      .shouldNot()
      .dependOnFiles()
      .inFolder('src/infrastructure/**');
    await expect(rule).toPassAsync();
  });
});
```

### No Circular Dependencies

```typescript
// tests/architecture/circular-deps.test.ts
import { projectFiles } from 'archunit';

it('should not have circular dependencies', async () => {
  const rule = projectFiles().inFolder('src/**').should().haveNoCycles();
  await expect(rule).toPassAsync();
});
```

---

## 4. Hexagonal Architecture: Ports & Adapters

### Directory Structure

```
src/
├── core/
│   ├── domain/           # Entities, business logic
│   └── ports/            # Interfaces (driven & driving)
├── adapters/
│   ├── primary/          # REST, GraphQL, CLI (driving)
│   └── secondary/        # Database, External APIs (driven)
└── config/               # DI wiring
```

### Port Naming Convention

- Interfaces: `*.port.ts`
- Adapters: `*.adapter.ts`

### ArchUnitTS Hexagonal Rules

```typescript
// tests/architecture/hexagonal.test.ts
import { projectFiles } from 'archunit';

describe('Hexagonal Architecture', () => {
  it('core domain should not depend on adapters', async () => {
    const rule = projectFiles()
      .inFolder('src/core/domain/**')
      .shouldNot()
      .dependOnFiles()
      .inFolder('src/adapters/**');
    await expect(rule).toPassAsync();
  });

  it('ports should not depend on adapters', async () => {
    const rule = projectFiles()
      .inFolder('src/core/ports/**')
      .shouldNot()
      .dependOnFiles()
      .inFolder('src/adapters/**');
    await expect(rule).toPassAsync();
  });

  it('secondary adapters should not depend on primary', async () => {
    const rule = projectFiles()
      .inFolder('src/adapters/secondary/**')
      .shouldNot()
      .dependOnFiles()
      .inFolder('src/adapters/primary/**');
    await expect(rule).toPassAsync();
  });
});
```

---

## 5. ESLint Boundaries Configuration

```javascript
// eslint.config.js or .eslintrc.js
import boundaries from "eslint-plugin-boundaries";

export default [
  {
    plugins: { boundaries },
    settings: {
      "boundaries/elements": [
        { type: "controller", pattern: "src/presentation/**/*controller*" },
        { type: "service", pattern: "src/application/**/*service*" },
        { type: "repository", pattern: "src/infrastructure/**/*repository*" },
        { type: "domain", pattern: "src/domain/**" },
        { type: "port", pattern: "src/core/ports/**" },
        { type: "adapter", pattern: "src/adapters/**" }
      ]
    },
    rules: {
      "boundaries/element-types": [2, {
        default: "disallow",
        rules: [
          // Domain: no external dependencies
          { from: { type: "domain" }, disallow: { to: { type: "*" } } },
          
          // Controllers: can use services, domain
          { from: { type: "controller" }, allow: { to: { type: ["service", "domain", "port"] } } },
          
          // Services: can use repositories, domain, ports
          { from: { type: "service" }, allow: { to: { type: ["repository", "domain", "port"] } } },
          
          // Repositories: can use ports
          { from: { type: "repository" }, allow: { to: { type: ["port"] } } },
          
          // Adapters: must implement ports
          { from: { type: "adapter" }, allow: { to: { type: ["port", "domain"] } } }
        ]
      }]
    }
  }
];
```

---

## 6. CI/CD Integration

### GitHub Actions

```yaml
# .github/workflows/architecture.yml
name: Architecture Validation

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  architecture:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm test -- --testPathPattern="architecture"
      - run: npx depcruise --config .dependency-cruiser.json src
```

### Fail Build on Violations

```yaml
- name: Fail on critical violations
  run: npx depcruise --config .dependency-cruiser.json --fail-on critical src
```

---

## 7. CLAUDE.md Integration

Add to every project's `CLAUDE.md`:

```markdown
# Architecture Rules

## Clean Architecture Layers
- `src/presentation` → `application`, `domain`
- `src/application` → `domain`
- `src/domain` → NO dependencies
- `src/infrastructure` → `domain`

## Hexagonal Structure
- Ports in `src/core/ports/` (interface suffix)
- Adapters in `src/adapters/` (adapter suffix)
- Domain in `src/core/domain/` (no external deps)

## Forbidden Patterns
- NO direct infrastructure imports in presentation
- NO domain entities in adapters
- NO circular dependencies

## Before Generating Code
Run architecture tests first:
npm test -- --testPathPattern="architecture"
```

---

## 8. Quick Reference Card

```
ARCHITECTURE GUARDRAILS CHECKLIST
==================================
LAYERS:
[ ] presentation → application, domain only
[ ] application → domain only
[ ] domain → no external dependencies

HEXAGONAL:
[ ] core/ports/*.port.ts define interfaces
[ ] adapters/*.adapter.ts implement ports
[ ] domain has no adapter imports

ESLINT:
[ ] boundaries plugin configured
[ ] rules enforced in CI/CD

TESTS:
[ ] ArchUnitTS layer tests pass
[ ] No circular dependencies
[ ] File size limits enforced
==================================
Violations → Block merge
```

---

## Related Documentation

- [AI_CODE_GENERATION_STANDARDS.md](./AI_CODE_GENERATION_STANDARDS.md)
- [AI_AGENT_DEVELOPMENT_METHODOLOGY.md](./AI_AGENT_DEVELOPMENT_METHODOLOGY.md)
- [AGENT_WORKFLOW.md](./AGENT_WORKFLOW.md)

---

*These guardrails ensure AI-generated code maintains structural integrity across all Sensible Analytics projects.*