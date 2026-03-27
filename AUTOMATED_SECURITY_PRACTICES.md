# Automated Security Practices for Sensible Analytics

## 🚨 Critical Security Risks in "Vibe Coded" Repos

When developing quickly with AI assistance, these security mistakes are common:

- ✅ **Hardcoded API keys** in source code
- ✅ **AI agent tokens** committed to git
- ✅ **Database credentials** in config files
- ✅ **.env files** accidentally committed
- ✅ **AWS/Azure/GCP keys** in scripts
- ✅ **Private keys** (SSH, TLS) in repos
- ✅ **Webhook secrets** exposed
- ✅ **JWT signing keys** in code

## 🛡️ Defense in Depth Strategy

Implement these **6 layers** of automated protection:

```
┌─────────────────────────────────────────────────────────────┐
│  LAYER 1: Developer Workstation (Pre-commit)               │
│  └─ Git hooks scan before every commit                      │
├─────────────────────────────────────────────────────────────┤
│  LAYER 2: CI/CD Pipeline (GitHub Actions)                  │
│  └─ Scan every PR and push                                  │
├─────────────────────────────────────────────────────────────┤
│  LAYER 3: Repository Protection (GitHub Native)            │
│  └─ Secret scanning, push protection                        │
├─────────────────────────────────────────────────────────────┤
│  LAYER 4: Code Review (Mandatory)                          │
│  └─ CODEOWNERS review required                              │
├─────────────────────────────────────────────────────────────┤
│  LAYER 5: Monitoring (Ongoing)                             │
│  └─ Continuous scanning + alerting                          │
├─────────────────────────────────────────────────────────────┤
│  LAYER 6: Incident Response (When Breach Happens)          │
│  └─ Automated rotation, revocation                          │
└─────────────────────────────────────────────────────────────┘
```

---

## LAYER 1: Pre-Commit Hooks (Developer Workstation)

### Installation (Per Repository)

Create `.pre-commit-config.yaml` in each repo:

```yaml
repos:
  # Secret Detection
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.4.0
    hooks:
      - id: detect-secrets
        args: ['--baseline', '.secrets.baseline']
        exclude: package.lock.json|yarn.lock|package-lock.json

  # Additional Secret Scanner
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.1
    hooks:
      - id: gitleaks

  # General file checks
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: check-added-large-files
      - id: check-case-conflict
      - id: check-merge-conflict
      - id: detect-private-key
      - id: end-of-file-fixer
      - id: trailing-whitespace
```

### Installation Script

```bash
#!/bin/bash
# install-hooks.sh

echo "Installing pre-commit hooks..."

# Install pre-commit
pip install pre-commit

# Install hooks
pre-commit install

# Run initial scan
pre-commit run --all-files

echo "✓ Pre-commit hooks installed"
echo "Scans will run automatically on every commit"
```

---

## LAYER 2: CI/CD Security Scanning

### GitHub Actions Security Workflow

```yaml
# .github/workflows/security-scan.yml
name: Security Scan

on:
  push:
    branches: [main, master]
  pull_request:
    branches: [main, master]
  schedule:
    - cron: '0 0 * * *'  # Daily

permissions:
  contents: read
  security-events: write

jobs:
  # 1. Secret Detection
  secret-detection:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history for better detection

      - name: Run Gitleaks
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          GITLEAKS_ENABLE_SUMMARY: true

      - name: Run TruffleHog
        uses: trufflesecurity/trufflehog@main
        with:
          path: ./
          base: main
          head: HEAD
          extra_args: --debug --only-verified

  # 2. Dependency Vulnerability Scan
  dependency-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          format: 'sarif'
          output: 'trivy-results.sarif'

      - name: Upload Trivy results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'

  # 3. Code Security Analysis
  code-security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Initialize CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: javascript,python
          queries: security-extended,security-and-quality

      - name: Autobuild
        uses: github/codeql-action/autobuild@v3

      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v3

  # 4. Infrastructure as Code Scan
  iac-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Checkov
        uses: bridgecrewio/checkov-action@master
        with:
          directory: .
          framework: all
          output_format: sarif
          output_file_path: reports/checkov.sarif

  # 5. Container Scan (if Dockerfile exists)
  container-scan:
    runs-on: ubuntu-latest
    if: hashFiles('**/Dockerfile') != ''
    steps:
      - uses: actions/checkout@v4

      - name: Build image
        run: docker build -t app:${{ github.sha }} .

      - name: Run Trivy container scan
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'app:${{ github.sha }}'
          format: 'sarif'
          output: 'trivy-container-results.sarif'

      - name: Upload results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-container-results.sarif'
```

---

## LAYER 3: GitHub Native Protection

### Organization-Level Settings

```bash
#!/bin/bash
# enable-org-security.sh

ORG="Sensible-Analytics"

echo "Enabling organization-level security features..."

# Enable secret scanning for all repos
gh api -X PATCH "orgs/$ORG" \
  -f "security_and_analysis[secret_scanning][status]=enabled"

# Enable push protection
gh api -X PATCH "orgs/$ORG" \
  -f "security_and_analysis[secret_scanning_push_protection][status]=enabled"

# Enable dependency graph
gh api -X PATCH "orgs/$ORG" \
  -f "security_and_analysis[dependency_graph][status]=enabled"

# Enable Dependabot alerts
gh api -X PATCH "orgs/$ORG" \
  -f "security_and_analysis[dependabot_alerts][status]=enabled"

# Enable Dependabot security updates
gh api -X PATCH "orgs/$ORG" \
  -f "security_and_analysis[dependabot_security_updates][status]=enabled"

echo "✓ Organization security features enabled"
```

### Repository-Level Settings

```bash
#!/bin/bash
# enable-repo-security.sh REPO_NAME

REPO=$1
ORG="Sensible-Analytics"

if [ -z "$REPO" ]; then
    echo "Usage: ./enable-repo-security.sh <repo-name>"
    exit 1
fi

echo "Enabling security for $ORG/$REPO..."

# Enable secret scanning
gh api -X PUT "repos/$ORG/$REPO/secret-scanning"

# Enable push protection
gh api -X PUT "repos/$ORG/$REPO/secret-scanning/push-protection"

# Enable vulnerability alerts
gh api -X PUT "repos/$ORG/$REPO/vulnerability-alerts"

# Enable automated security fixes
gh api -X PUT "repos/$ORG/$REPO/automated-security-fixes"

echo "✓ Security enabled for $REPO"
```

---

## LAYER 4: Mandatory Code Review

### Branch Protection Rules

```bash
#!/bin/bash
# configure-branch-protection.sh REPO_NAME

REPO=$1
ORG="Sensible-Analytics"
BRANCH="main"  # or "master"

if [ -z "$REPO" ]; then
    echo "Usage: ./configure-branch-protection.sh <repo-name>"
    exit 1
fi

echo "Configuring branch protection for $ORG/$REPO/$BRANCH..."

gh api -X PUT "repos/$ORG/$REPO/branches/$BRANCH/protection" \
  -H "Accept: application/vnd.github+json" \
  -f "required_status_checks[strict]=true" \
  -f "required_status_checks[contexts][]=Security Scan" \
  -f "required_status_checks[contexts][]=secret-detection" \
  -f "required_pull_request_reviews[required_approving_review_count]=1" \
  -f "required_pull_request_reviews[dismiss_stale_reviews]=true" \
  -f "required_pull_request_reviews[require_code_owner_reviews]=true" \
  -f "required_pull_request_reviews[required_conversation_resolution]=true" \
  -f "enforce_admins=true" \
  -f "required_linear_history=true" \
  -f "allow_force_pushes=false" \
  -f "allow_deletions=false" \
  -f "required_conversation_resolution=true" \
  -f "required_signatures=false"

echo "✓ Branch protection configured for $REPO"
```

---

## LAYER 5: Environment Security

### .env.example Template

Create this file in EVERY repository:

```bash
# .env.example
# Copy this file to .env and fill in your actual values
# NEVER commit .env to git!

# ============================================
# AI/ML API Keys
# ============================================
OPENAI_API_KEY=sk-your-openai-key-here
ANTHROPIC_API_KEY=sk-ant-your-anthropic-key-here
HUGGINGFACE_API_KEY=hf_your_huggingface_token
COHERE_API_KEY=your-cohere-key

# ============================================
# Database Credentials
# ============================================
DATABASE_URL=postgresql://user:password@localhost:5432/dbname
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_ANON_KEY=your-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key

# ============================================
# Cloud Providers
# ============================================
# AWS
AWS_ACCESS_KEY_ID=AKIA...
AWS_SECRET_ACCESS_KEY=your-secret-key
AWS_REGION=ap-southeast-2

# Azure
AZURE_CLIENT_ID=your-client-id
AZURE_CLIENT_SECRET=your-client-secret
AZURE_TENANT_ID=your-tenant-id

# GCP
GOOGLE_APPLICATION_CREDENTIALS=path/to/service-account.json

# ============================================
# Authentication
# ============================================
JWT_SECRET=your-jwt-secret-min-32-chars
NEXTAUTH_SECRET=your-nextauth-secret
NEXTAUTH_URL=http://localhost:3000

# OAuth Providers
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret
GITHUB_CLIENT_ID=your-github-client-id
GITHUB_CLIENT_SECRET=your-github-client-secret

# ============================================
# External APIs
# ============================================
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
SENDGRID_API_KEY=SG.xxx
TWILIO_ACCOUNT_SID=ACxxx
TWILIO_AUTH_TOKEN=your-auth-token

# ============================================
# Application Settings
# ============================================
NODE_ENV=development
PORT=3000
LOG_LEVEL=debug
```

### .gitignore Template

```gitignore
# ============================================
# SECURITY - NEVER COMMIT THESE FILES
# ============================================

# Environment files
.env
.env.local
.env.*.local
.env.development
.env.test
.env.production
!.env.example

# Secret keys
*.pem
*.key
*.p12
*.pfx
*.crt
*.cer
*.der
id_rsa
id_rsa.pub
id_dsa
id_dsa.pub
id_ecdsa
id_ecdsa.pub
id_ed25519
id_ed25519.pub

# AI/ML model files that might contain embeddings
*.gguf
*.bin
models/
embeddings/

# Database files
*.sqlite
*.sqlite3
*.db

# Log files that might contain sensitive data
*.log
logs/
log/

# Temporary files
tmp/
temp/
*.tmp
*.temp

# OS files
.DS_Store
Thumbs.db

# IDE files (might contain local paths)
.vscode/settings.json
.idea/
*.sublime-project
*.sublime-workspace

# Dependencies
node_modules/
__pycache__/
*.pyc
*.pyo
*.pyd
.Python
*.so

# Build outputs
dist/
build/
out/
.next/

# Coverage reports
coverage/
.nyc_output/

# Secret scanning baselines (if generated)
.secrets.baseline

# Local development files
.docker/data/
volumes/
```

---

## LAYER 6: Incident Response

### Secret Rotation Playbook

```markdown
# SECRET COMPROMISE RESPONSE PLAYBOOK

## Immediate Actions (Within 1 Hour)

1. **Identify the exposed secret**
   - Note the repository, file, and line number
   - Determine what service the secret belongs to

2. **Revoke the secret IMMEDIATELY**
   - Log into the service dashboard
   - Revoke/regenerate the key
   - Document the time of revocation

3. **Assess impact**
   - Check service logs for unauthorized access
   - Determine what data/systems were accessible
   - Check for any unauthorized actions taken

4. **Notify team**
   - Alert engineering lead
   - Document in incident log

## Short-term Actions (Within 24 Hours)

1. **Update all environments**
   - Replace secret in production
   - Replace secret in staging
   - Update CI/CD secrets
   - Update developer environments

2. **Audit access logs**
   - Review service logs thoroughly
   - Check for suspicious activity
   - Export logs for analysis

3. **Clean git history (if needed)**
   ```bash
   # Use BFG Repo-Cleaner
   bfg --delete-files YOUR-FILE-WITH-SECRET
   bfg --replace-text secrets.txt
   ```

4. **Communicate**
   - If customer data affected: legal review
   - If compliance violation: notify compliance team
   - Update security incident tracker

## Long-term Actions (Within 1 Week)

1. **Root cause analysis**
   - How did the secret get committed?
   - Why didn't pre-commit hooks catch it?
   - Process gaps identified

2. **Implement preventive measures**
   - Update developer training
   - Enhance automated scanning
   - Improve code review checklist

3. **Update documentation**
   - Document lessons learned
   - Update security runbooks
   - Share with team

## Post-Incident Review

Questions to answer:
- [ ] How was the secret exposed?
- [ ] Why didn't automated tools prevent it?
- [ ] Was there unauthorized access?
- [ ] What data was at risk?
- [ ] What preventive measures were implemented?
```

---

## 🔍 Common Patterns to Detect

### AI Agent Keys

```yaml
# Patterns to watch for
patterns:
  # OpenAI
  - regex: 'sk-[a-zA-Z0-9]{48}'
    provider: OpenAI
  
  # Anthropic
  - regex: 'sk-ant-[a-zA-Z0-9_-]+'
    provider: Anthropic
  
  # Hugging Face
  - regex: 'hf_[a-zA-Z]{34}'
    provider: HuggingFace
  
  # Cohere
  - regex: '[a-zA-Z0-9]{40}'
    provider: Cohere
  
  # Replicate
  - regex: 'r8_[a-zA-Z0-9]{40}'
    provider: Replicate
  
  # Pinecone
  - regex: '[a-f0-9]{8}-[a-f0-9]{4}-[a-f0-9]{4}-[a-f0-9]{4}-[a-f0-9]{12}'
    provider: Pinecone
```

### Database Connections

```yaml
patterns:
  # PostgreSQL
  - regex: 'postgresql://[^:]+:[^@]+@'
    type: PostgreSQL
  
  # MongoDB
  - regex: 'mongodb(\+srv)?://[^:]+:[^@]+@'
    type: MongoDB
  
  # Redis
  - regex: 'redis://:[^@]+@'
    type: Redis
  
  # MySQL
  - regex: 'mysql://[^:]+:[^@]+@'
    type: MySQL
```

### Cloud Provider Keys

```yaml
patterns:
  # AWS
  - regex: 'AKIA[0-9A-Z]{16}'
    type: AWS Access Key
  
  - regex: '[0-9a-zA-Z/+]{40}'
    type: AWS Secret Key
  
  # Azure
  - regex: '[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}'
    type: Azure Key
  
  # GCP
  - regex: 'AIza[0-9A-Za-z_-]{35}'
    type: Google API Key
```

---

## 🚀 Implementation Checklist

### For Each Repository

- [ ] Add `.pre-commit-config.yaml`
- [ ] Add `.env.example` with all required variables
- [ ] Update `.gitignore` with security patterns
- [ ] Add security scanning GitHub Actions workflow
- [ ] Enable secret scanning in repo settings
- [ ] Enable push protection
- [ ] Configure branch protection rules
- [ ] Add SECURITY.md
- [ ] Train developers on security practices

### Organization Level

- [ ] Enable secret scanning for all repos
- [ ] Enable push protection org-wide
- [ ] Set up security alerts
- [ ] Create incident response team
- [ ] Document secret rotation procedures
- [ ] Schedule security training

---

## 📊 Security Metrics to Track

Monitor these KPIs:

| Metric | Target | Tool |
|--------|--------|------|
| Secrets committed | 0 | Gitleaks, GitHub Secret Scanning |
| High-severity vulnerabilities | 0 | Dependabot, Trivy |
| Security scan failures | 0 | GitHub Actions |
| Mean time to patch | < 7 days | Dependabot |
| Failed push protection blocks | Track | GitHub |
| Security training completion | 100% | Manual |

---

## 🎓 Developer Training

### Security Checklist for Developers

Before every commit, ask:

- [ ] Did I check for hardcoded passwords/API keys?
- [ ] Is `.env` in `.gitignore`?
- [ ] Did I run `detect-secrets` locally?
- [ ] Are there any `.log` files with sensitive data?
- [ ] Are database connection strings using environment variables?
- [ ] Are AI agent keys in environment variables?
- [ ] Did I review the diff before committing?

### Common Mistakes to Avoid

1. ❌ **DON'T**: Commit `.env` files
   ✅ **DO**: Use `.env.example` and document variables

2. ❌ **DON'T**: Hardcode API keys in source code
   ✅ **DO**: Use environment variables

3. ❌ **DON'T**: Log sensitive data for debugging
   ✅ **DO**: Sanitize logs before committing

4. ❌ **DON'T**: Share credentials in Slack/Teams
   ✅ **DO**: Use secret management tools

5. ❌ **DON'T**: Use production keys in development
   ✅ **DO**: Separate dev/staging/prod credentials

---

## 📚 Additional Resources

- [GitHub Secret Scanning](https://docs.github.com/en/code-security/secret-scanning)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [SANS Securing Web Applications](https://www.sans.org/web-application-security/)
- [Mozilla Web Security Guidelines](https://infosec.mozilla.org/guidelines/web_security)

---

**Remember**: Security is not a one-time setup. It's a continuous process of monitoring, detection, and improvement.

**Questions?** Contact security@sensibleanalytics.co
