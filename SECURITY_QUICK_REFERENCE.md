# 🔒 Security Quick Reference for Developers

## ⚠️ CRITICAL: Before Every Commit

Run this checklist:

```bash
□ git diff --cached --name-only  # Review what you're committing
□ grep -r "sk-" . --include="*.js" --include="*.ts" --include="*.py"  # Check for API keys
□ grep -r "postgresql://" . --include="*.env*"  # Check for DB strings
□ ls -la | grep "\.env"  # Ensure .env is not staged
□ cat .gitignore | grep "\.env"  # Verify .env is ignored
```

## 🚨 NEVER Commit These

```
❌ .env files (any variation)
❌ API keys (sk-*, AKIA*, hf_*)
❌ Database connection strings
❌ Private keys (*.pem, id_rsa)
❌ AI agent tokens
❌ JWT signing secrets
❌ Webhook secrets
❌ AWS/Azure/GCP credentials
❌ Log files with sensitive data
```

## ✅ ALWAYS Do These

```bash
# 1. Use environment variables
echo "OPENAI_API_KEY=$OPENAI_API_KEY" >> .env

# 2. Copy .env.example to .env
cp .env.example .env

# 3. Add to .gitignore
echo ".env" >> .gitignore

# 4. Use .env.example for documentation
cat .env.example  # Shows what variables are needed

# 5. Sanitize logs before committing
rm *.log
```

## 🔍 Quick Secret Detection

### Install Tools

```bash
# Gitleaks (fast secret scanner)
brew install gitleaks  # macOS
# or
docker pull zricethezav/gitleaks

# Detect Secrets (Yelp)
pip install detect-secrets
detect-secrets scan > .secrets.baseline

# TruffleHog
docker pull trufflesecurity/trufflehog:latest
```

### Run Checks

```bash
# Scan current directory
gitleaks detect --source . --verbose

# Scan git history
gitleaks detect --source . --no-git

# Scan specific file
gitleaks detect --source ./config.js

# Check specific patterns
grep -E "(sk-|AKIA|hf_|postgresql://)" . -r --include="*.js" --include="*.ts" --include="*.py"
```

## 🛠️ Setting Up Pre-Commit Hooks

```bash
# 1. Install pre-commit
pip install pre-commit
# or
brew install pre-commit

# 2. Create .pre-commit-config.yaml
cat > .pre-commit-config.yaml << 'EOF'
repos:
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.4.0
    hooks:
      - id: detect-secrets
        args: ['--baseline', '.secrets.baseline']
  
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.1
    hooks:
      - id: gitleaks
EOF

# 3. Install hooks
pre-commit install

# 4. Run on all files (first time)
pre-commit run --all-files

# 5. Now it runs automatically on every commit!
```

## 🆘 If You Accidentally Committed a Secret

### Step 1: DON'T PANIC

### Step 2: Revoke the Secret IMMEDIATELY

```bash
# Example: OpenAI key
curl https://api.openai.com/v1/keys \
  -H "Authorization: Bearer sk-your-key" \
  -X DELETE

# Or use the provider's dashboard to revoke
```

### Step 3: Remove from Git History

```bash
# Install BFG
curl -o bfg.jar https://repo1.maven.org/maven2/com/madgag/bfg/1.14.0/bfg-1.14.0.jar

# Remove file with secret
java -jar bfg.jar --delete-files config-with-secrets.js

# Or replace text
java -jar bfg.jar --replace-text secrets.txt

# Clean up
git reflog expire --expire=now --all
git gc --prune=now --aggressive

# Force push (DANGEROUS - coordinate with team!)
git push origin main --force
```

### Step 4: Rotate All Related Secrets

If one secret is exposed, rotate all related ones:
- Database passwords
- API keys for same service
- JWT secrets
- Session tokens

### Step 5: Audit Access Logs

Check service logs for unauthorized access:
- API provider dashboard
- Database logs
- Application logs
- Cloud provider logs

### Step 6: Notify Security Team

Email: security@sensibleanalytics.co

Include:
- What was exposed
- When it happened
- Actions taken
- Impact assessment

## 📋 Environment Variable Template

### Good Example

```javascript
// config.js - GOOD ✅
const config = {
  openaiApiKey: process.env.OPENAI_API_KEY,
  databaseUrl: process.env.DATABASE_URL,
  jwtSecret: process.env.JWT_SECRET,
};
```

### Bad Example

```javascript
// config.js - BAD ❌
const config = {
  openaiApiKey: 'sk-abc123...',  // NEVER DO THIS
  databaseUrl: 'postgresql://user:pass@localhost/db',  // NEVER DO THIS
};
```

## 🔐 Secure Code Patterns

### Loading Environment Variables

```javascript
// Node.js
require('dotenv').config();

const apiKey = process.env.OPENAI_API_KEY;
if (!apiKey) {
  throw new Error('OPENAI_API_KEY is required');
}
```

```python
# Python
import os
from dotenv import load_dotenv

load_dotenv()

api_key = os.getenv('OPENAI_API_KEY')
if not api_key:
    raise ValueError('OPENAI_API_KEY is required')
```

### Sanitizing Logs

```javascript
// GOOD ✅
console.log('API request made', { 
  endpoint: '/api/users',
  // DON'T log: apiKey, password, token
});

// BAD ❌
console.log('API request', { apiKey: config.apiKey });  // NEVER!
```

```python
# GOOD ✅
import logging

# Redact sensitive fields
class SensitiveDataFilter(logging.Filter):
    def filter(self, record):
        record.msg = record.msg.replace(os.getenv('API_KEY'), '[REDACTED]')
        return True

logger.addFilter(SensitiveDataFilter())
```

## 🔔 Common Mistakes

### 1. Hardcoded Credentials in Examples

```javascript
// ❌ BAD - Don't do this even in examples
const example = {
  apiKey: 'sk-demo123...'  
};

// ✅ GOOD
const example = {
  apiKey: process.env.API_KEY || 'your-api-key-here'
};
```

### 2. Logging Full Objects

```javascript
// ❌ BAD
console.log('Config:', config);  // May include secrets

// ✅ GOOD
console.log('Config:', {
  port: config.port,
  env: config.env,
  // Explicitly list safe fields
});
```

### 3. Comments with Secrets

```javascript
// ❌ BAD
// API Key: sk-abc123...
const client = new OpenAI();

// ✅ GOOD
// Initialize OpenAI client (API key from env)
const client = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});
```

### 4. Test Files with Real Keys

```javascript
// ❌ BAD - test/api.test.js
const apiKey = 'sk-real-production-key';  // NO!

// ✅ GOOD - test/api.test.js
const apiKey = process.env.TEST_API_KEY || 'sk-test-fake-key';
```

## 🚀 CI/CD Security

### GitHub Actions Safe Practices

```yaml
# ✅ GOOD
jobs:
  deploy:
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy
        env:
          API_KEY: ${{ secrets.API_KEY }}  # Use secrets
        run: |
          echo "Deploying with API key..."
          deploy --key "$API_KEY"
```

```yaml
# ❌ BAD
jobs:
  deploy:
    steps:
      - run: |
          deploy --key "sk-abc123..."  # NEVER hardcode!
```

### Docker Safe Practices

```dockerfile
# ✅ GOOD - Use build args
ARG API_KEY
ENV API_KEY=$API_KEY

# ❌ BAD - Hardcoded
ENV API_KEY=sk-abc123...
```

## 📞 Emergency Contacts

| Issue | Contact |
|-------|---------|
| Secret exposed | security@sensibleanalytics.co |
| Security question | #security Slack channel |
| Incident response | +61-XXX-XXX-XXX |

## 📚 Additional Resources

- [GitHub Secret Scanning](https://docs.github.com/en/code-security/secret-scanning)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Mozilla Web Security](https://infosec.mozilla.org/guidelines/web_security)

---

**Remember: When in doubt, DON'T commit it. Ask first!**

**Security is everyone's responsibility.** 🛡️
