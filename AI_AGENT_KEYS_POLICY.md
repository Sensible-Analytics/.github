# AI Agent Keys Security Policy

## 🚨 CRITICAL: AI Agent Keys Are High-Value Targets

AI agent API keys (OpenAI, Anthropic, Hugging Face, etc.) are:
- ✅ **Expensive** - Can rack up $1000s in usage if stolen
- ✅ **Powerful** - Can generate content, access data, execute code
- ✅ **Untraceable** - Hard to audit what was done with stolen keys
- ✅ **Instantly exploitable** - No cooldown period

## 🛡️ Zero-Tolerance Policy

### Automatic Blocking

Any commit containing AI agent keys will be:
1. **Automatically rejected** by push protection
2. **Flagged for security review**
3. **Reported to security team**
4. **Key immediately revoked**

### Disciplinary Actions

| Violation | Action |
|-----------|--------|
| Accidental commit (caught by automation) | Mandatory security training |
| Repeated violations | Revoke commit access |
| Intentional exposure | Immediate termination consideration |

---

## 📋 AI Agent Key Management

### 1. Key Generation

**NEVER use production keys for:**
- ❌ Local development
- ❌ Testing environments
- ❌ CI/CD pipelines
- ❌ Shared development servers

**ALWAYS use separate keys:**
- ✅ Development keys (rate-limited, low budget)
- ✅ Staging keys (moderate limits)
- ✅ Production keys (strictest controls)

### 2. Key Storage

**Approved Storage Methods:**

```bash
# 1. Environment Variables (Local Dev)
export OPENAI_API_KEY="sk-..."

# 2. GitHub Secrets (CI/CD)
# Settings > Secrets and variables > Actions

# 3. Secret Management Services
# - AWS Secrets Manager
# - Azure Key Vault
# - Google Secret Manager
# - HashiCorp Vault
# - Doppler
```

**FORBIDDEN Storage Methods:**

```bash
❌ Hardcoded in source code
❌ Committed .env files
❌ Slack/Teams messages
❌ Email
❌ README files
❌ Configuration files in git
❌ Docker images
❌ Browser bookmarks
```

### 3. Key Rotation Schedule

| Environment | Rotation Frequency | Method |
|-------------|-------------------|---------|
| Production | Monthly | Automated |
| Staging | Quarterly | Manual |
| Development | As needed | Manual |

### 4. Key Monitoring

**All AI agent keys must have:**
- ✅ Usage alerts (>$50/day triggers alert)
- ✅ Rate limiting enabled
- ✅ IP allowlisting (production only)
- ✅ Activity logging
- ✅ Automatic revocation on anomaly

---

## 🔍 Detection Patterns

### Keys We Monitor

| Provider | Pattern Example | Risk Level |
|----------|----------------|------------|
| **OpenAI** | `sk-abc123...` (48 chars) | 🔴 Critical |
| **Anthropic** | `sk-ant-...` | 🔴 Critical |
| **Hugging Face** | `hf_...` (34 chars) | 🔴 Critical |
| **Cohere** | 40 char string | 🟠 High |
| **Replicate** | `r8_...` | 🟠 High |
| **Pinecone** | UUID format | 🟠 High |
| **SerpAPI** | `serp_...` | 🟡 Medium |
| **Weather API** | Various | 🟢 Low |

### Detection Methods

1. **Pre-commit hooks** - Local scanning
2. **GitHub Secret Scanning** - Server-side detection
3. **CI/CD pipeline** - Build-time checks
4. **Nightly scans** - Repository-wide audit
5. **Real-time monitoring** - Usage anomaly detection

---

## 🚀 Secure Development Workflow

### Step 1: Setup Local Environment

```bash
# 1. Clone repository
git clone https://github.com/Sensible-Analytics/PROJECT.git
cd PROJECT

# 2. Copy environment template
cp .env.example .env

# 3. Request development keys from DevOps
# DO NOT use production keys!

# 4. Add keys to .env (never commit this file)
echo "OPENAI_API_KEY=sk-dev-your-key-here" >> .env

# 5. Verify .env is ignored
git check-ignore .env  # Should output: .env

# 6. Install pre-commit hooks
pip install pre-commit
pre-commit install
```

### Step 2: Development

```javascript
// ✅ GOOD: Load from environment
const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

// ❌ BAD: Never do this
const openai = new OpenAI({
  apiKey: 'sk-abc123...'  // HARDCODED = SECURITY INCIDENT
});
```

### Step 3: Testing

```javascript
// Use test keys or mocks in tests
const mockOpenAI = {
  chat: {
    completions: {
      create: jest.fn().mockResolvedValue({
        choices: [{ message: { content: 'test' } }]
      })
    }
  }
};
```

### Step 4: Commit

```bash
# Before committing, run security checks
gitleaks detect --source . --verbose

# Review your changes
git diff --cached

# Check for any secrets
grep -r "sk-" . --include="*.js" --include="*.ts"

# Commit
git commit -m "feat: add AI feature"
```

---

## 🆘 Incident Response

### Level 1: Key Exposed in Commit (Caught by Automation)

**Timeline: Immediate**

```bash
# 1. Revoke key immediately
# Log into provider dashboard and delete/disable

# 2. Generate new key
# Create new development key

# 3. Update .env locally
echo "OPENAI_API_KEY=sk-new-key" > .env

# 4. Document incident
# Email: security@sensibleanalytics.co
```

### Level 2: Key Exposed and Used by Attacker

**Timeline: Within 1 hour**

```bash
# 1. REVOKE KEY IMMEDIATELY
# Provider dashboard > API Keys > Revoke

# 2. Check usage logs
# Provider dashboard > Usage
# Look for:
#   - Unusual spike in requests
#   - Requests from unknown IPs
#   - Unusual model usage patterns

# 3. Assess damage
# - Cost incurred
# - Data potentially accessed
# - Actions taken by attacker

# 4. Generate new key with restrictions
# - IP allowlisting
# - Usage limits
# - Restricted models

# 5. Notify security team
# Email: security@sensibleanalytics.co
# Subject: [INCIDENT] AI Key Compromised
# Include:
#   - Key that was exposed
#   - When it was exposed
#   - Usage statistics
#   - Actions taken
```

### Level 3: Production Key Compromised

**Timeline: Immediate - Emergency Response**

```bash
# 1. REVOKE ALL KEYS IMMEDIATELY
# All environments affected

# 2. Emergency deployment
# Deploy with new keys

# 3. Customer notification (if applicable)
# Legal review required

# 4. Full forensic audit
# Preserve all logs

# 5. Post-mortem within 24 hours
```

---

## 📊 Monitoring & Alerting

### Usage Alerts

| Metric | Threshold | Action |
|--------|-----------|--------|
| Daily spend | >$50 | Alert to Slack |
| Hourly requests | >1000/min | Alert + Rate limit |
| Error rate | >10% | Alert + Review |
| New IP address | Any | Log + Alert |
| After-hours usage | 10pm-6am | Alert + Review |

### Dashboards

Monitor these dashboards daily:
- OpenAI Usage Dashboard
- Anthropic Console
- Hugging Face Usage
- Internal cost tracking

---

## 🎓 Training Requirements

### Mandatory for All Developers

**Before accessing AI agent keys:**

1. **Complete Security Training**
   - Key management best practices
   - Incident response procedures
   - Hands-on exercise: Detect secrets in code

2. **Pass Security Quiz**
   - 10 questions, 100% required
   - Covers all topics in this document

3. **Acknowledge Policy**
   - Sign acknowledgment form
   - Reviewed annually

4. **Shadow Period**
   - First 2 weeks: Pair with senior developer
   - All commits reviewed before push

### Refresher Training

- **Quarterly**: 30-minute review
- **After incident**: Immediate retraining
- **Policy updates**: Mandatory review

---

## 🔐 Key Generation Request Process

### For New Projects

1. **Submit Request**
   ```
   To: devops@sensibleanalytics.co
   Subject: [KEY REQUEST] Project Name
   
   Environment: Development/Staging/Production
   Provider: OpenAI/Anthropic/Hugging Face/etc
   Use case: Brief description
   Expected usage: Requests/month, estimated cost
   Requestor: Your name
   Manager approval: Attached
   ```

2. **Review Process**
   - DevOps reviews (1 business day)
   - Security review (if production)
   - Manager approval verified

3. **Key Delivery**
   - Secure channel only (never email)
   - 1Password or similar
   - Expires in 90 days if unused

4. **Documentation**
   - Key registered in inventory
   - Owner assigned
   - Rotation scheduled

---

## 📞 Contact Information

| Role | Contact | When to Contact |
|------|---------|----------------|
| **Security Team** | security@sensibleanalytics.co | Any key exposure |
| **DevOps** | devops@sensibleanalytics.co | Key requests, rotation |
| **Emergency** | +61-XXX-XXX-XXX | Active breach |
| **Manager** | Your direct manager | Policy violations |

---

## ✅ Developer Checklist

Before using AI agent keys:

- [ ] Completed security training
- [ ] Passed security quiz
- [ ] Pre-commit hooks installed
- [ ] .env.example copied to .env
- [ ] .env added to .gitignore
- [ ] Development key obtained (not production)
- [ ] Code review process understood
- [ ] Incident response procedure reviewed

---

## 🚫 Prohibited Practices

### NEVER DO THESE

1. ❌ Share keys via Slack, Teams, email
2. ❌ Store keys in password managers without 2FA
3. ❌ Use production keys for development
4. ❌ Share personal API keys with team
5. ❌ Check keys into any repository (even private)
6. ❌ Hardcode keys in scripts or notebooks
7. ❌ Log keys or responses containing keys
8. ❌ Take screenshots of keys
9. ❌ Store keys in cloud storage (Google Drive, Dropbox)
10. ❌ Use keys on personal devices without approval

---

**Violations of this policy may result in:**
- Immediate key revocation
- Loss of commit access
- Disciplinary action up to termination
- Legal action (if intentional malice)

**Questions? Contact: security@sensibleanalytics.co**

**Last Updated:** March 28, 2026  
**Version:** 1.0  
**Next Review:** June 28, 2026
