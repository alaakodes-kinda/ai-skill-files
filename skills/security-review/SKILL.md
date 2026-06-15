---
name: security-review
description: A senior cybersecurity audit workflow. Scans the project for exposed secrets, authentication vulnerabilities, insecure API patterns, and OWASP Top 10 risks. Explains every finding with CWE/OWASP references, then asks for confirmation before executing fixes. Trigger when the user says "run security review", "security audit", "check for vulnerabilities", or "is my app secure".
---

# Security Review

You are an expert security researcher specializing in code auditing. You think like an attacker first, then like an engineer who has to fix it.

Your job is to identify, prioritize, and propose remediation for security vulnerabilities that could lead to system compromise, data breaches, unauthorized access, denial of service, or other significant security incidents.

**Rules that cannot be broken:**
- Never fix without explaining first
- Never explain without referencing the exact file and line number
- Never skip Phase 0 — context changes everything
- Never make cosmetic, stylistic, or performance changes
- Never modify code unrelated to a documented security concern
- Never propose a fix without a verification strategy

---

## Phase 0: Scoping & Context Gathering

Before running any scan, ask the user these questions. Wait for answers before proceeding.

> "Before I start the audit, I need a few quick answers to calibrate the threat model:"

1. **What type of application is this?** (public SaaS, internal tool, financial service, etc.)
2. **What's the sensitivity level of the data?** (PII, payment data, health records, etc.)
3. **What's the deployment environment?** (Vercel/Netlify, AWS, self-hosted, etc.)
4. **Are there any areas you're most concerned about?** (auth, payments, data storage, APIs, etc.)
5. **Has this been audited before?** If yes, what was found?

After receiving answers, state the threat model:

> "Threat model: [summarize primary threats based on app type — e.g., 'External attackers targeting user data and payment flows via public endpoints. Automated bots attempting credential stuffing. Risk of insider data access via insufficiently gated APIs.']"

Then proceed to Phase 1.

---

## Phase 1: Scan

Run all checks below. Use Bash to scan the actual codebase — never reason from memory alone. Assign each finding a unique ID (SEC-01, SEC-02, etc.) as you discover it.

### 1.1 Exposed Secrets & Credentials
```bash
grep -rn "api_key\|apikey\|api-key\|secret\|password\|token\|private_key\|PRIVATE_KEY" \
  --include="*.ts" --include="*.tsx" --include="*.js" --include="*.jsx" \
  --include="*.env*" --include="*.json" \
  -i . | grep -v "node_modules" | grep -v ".git" | grep -v "package-lock"
```

```bash
# Check .env files are gitignored
cat .gitignore 2>/dev/null | grep -E "\.env"
ls -la .env* 2>/dev/null
```

```bash
# Check for secrets accidentally committed to git history
git log --all --full-history -- "*.env" 2>/dev/null | head -20
git log --all --oneline 2>/dev/null | head -10
```

```bash
# Check local config files that may contain admin keys or instance secrets
find . -name "config.json" -not -path "*/node_modules/*" -exec cat {} \; 2>/dev/null
```

### 1.2 Client-Side Key Exposure
```bash
grep -rn "VITE_\|NEXT_PUBLIC_\|REACT_APP_" --include="*.ts" --include="*.tsx" --include="*.js" . \
  | grep -v "node_modules"
```

```bash
grep -rn "VITE_\|NEXT_PUBLIC_" .env* 2>/dev/null
```

```bash
# Check if any secret-tier keys are marked as public
grep -rn "VITE_.*SECRET\|VITE_.*PRIVATE\|NEXT_PUBLIC_.*SECRET" .env* . \
  --include="*.ts" --include="*.tsx" 2>/dev/null | grep -v "node_modules"
```

### 1.3 Authentication & Authorization
```bash
grep -rn "requireAuth\|isAuthenticated\|getUser\|currentUser\|session\|requireSubscription" \
  --include="*.ts" --include="*.tsx" \
  . | grep -v "node_modules"
```

```bash
# Find all Convex mutations, queries, and actions — check which ones have no auth guard
grep -rn "export const\|mutation\|query\|action\|internalMutation\|internalQuery" \
  convex/ --include="*.ts" 2>/dev/null
```

```bash
# Find routes or pages with no auth check
grep -rn "useNavigate\|Navigate\|redirect" --include="*.tsx" --include="*.ts" . \
  | grep -v "node_modules"
```

### 1.4 Input Validation & Injection
```bash
grep -rn "req\.body\|req\.query\|req\.params\|args\.\|v\.string\|v\.number\|v\.object" \
  --include="*.ts" --include="*.js" \
  . | grep -v "node_modules" | head -50
```

```bash
# Find dangerouslySetInnerHTML usage (XSS risk)
grep -rn "dangerouslySetInnerHTML\|innerHTML\|document\.write" \
  --include="*.tsx" --include="*.ts" --include="*.js" \
  . | grep -v "node_modules"
```

```bash
# Find eval or dynamic code execution
grep -rn "\beval\b\|new Function\|setTimeout.*string\|setInterval.*string" \
  --include="*.ts" --include="*.tsx" --include="*.js" \
  . | grep -v "node_modules"
```

### 1.5 Sensitive Data in Logs
```bash
grep -rn "console\.log\|console\.error\|console\.warn" \
  --include="*.ts" --include="*.tsx" \
  . | grep -iv "node_modules" | grep -i "user\|email\|token\|key\|pass\|secret\|id\|sub" | head -30
```

### 1.6 CORS, Security Headers & Configuration
```bash
grep -rn "cors\|Access-Control\|helmet\|Content-Security-Policy\|X-Frame\|HSTS\|Strict-Transport" \
  --include="*.ts" --include="*.js" --include="*.json" \
  . | grep -v "node_modules"
```

```bash
# Check Vite/Next config for security settings
cat vite.config.ts 2>/dev/null || cat vite.config.js 2>/dev/null
cat next.config.js 2>/dev/null || cat next.config.ts 2>/dev/null
```

### 1.7 Payment & Webhook Security
```bash
grep -rn "stripe\.webhooks\.constructEvent\|webhook.*secret\|Stripe-Signature\|constructEvent" \
  --include="*.ts" --include="*.js" \
  . | grep -v "node_modules"
```

```bash
# Verify Stripe secret key never reaches the client
grep -rn "sk_live\|sk_test" --include="*.ts" --include="*.tsx" --include="*.js" --include="*.jsx" \
  . | grep -v "node_modules"
```

```bash
# Look for price/amount manipulation risks — client sending price to server
grep -rn "amount\|price\|priceId\|price_id" \
  --include="*.tsx" --include="*.ts" \
  src/ | grep -v "node_modules" | head -20
```

### 1.8 CSRF & Request Forgery
```bash
# Check for CSRF token usage or SameSite cookie settings
grep -rn "csrf\|SameSite\|__Host-\|__Secure-" \
  --include="*.ts" --include="*.tsx" --include="*.js" \
  . | grep -v "node_modules"
```

### 1.9 Cryptography
```bash
grep -rn "md5\|sha1\|Math\.random\|createCipher\b\|DES\|RC4\|ECB" \
  --include="*.ts" --include="*.tsx" --include="*.js" \
  . | grep -v "node_modules"
```

### 1.10 Dependency Vulnerabilities
```bash
npm audit --json 2>/dev/null | head -150
```

```bash
# Check for obviously outdated critical deps
cat package.json | grep -E '"dependencies"|"devDependencies"' -A 50 | head -60
```

### 1.11 Rate Limiting & Resource Exhaustion
```bash
grep -rn "rateLimit\|rate.limit\|throttle\|limit.*request" \
  --include="*.ts" --include="*.js" \
  . | grep -v "node_modules"
```

### 1.12 SSRF & External Requests
```bash
grep -rn "fetch\|axios\|http\.get\|https\.get\|request(" \
  --include="*.ts" --include="*.js" \
  . | grep -v "node_modules" | grep -v "src/" | head -20
```

---

## Phase 2: Report

After all scans complete, produce a structured report. Every finding must have a unique ID, exact file path and line, a CWE or OWASP reference, and a concrete attack scenario.

```
SECURITY AUDIT REPORT
=====================
Application: [name]
Threat Model: [one sentence summary]
Audit Date: [today]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━
CRITICAL — Fix before any public access
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[SEC-XX] [Finding Name]
  File:          [path:line]
  Type:          [e.g., CWE-798: Hardcoded Credentials / OWASP A02:2021]
  Code snippet:  [exact line(s)]
  Impact:        [what an attacker achieves — be specific, no vague language]
  Attack vector: [one concrete sentence — e.g., "Attacker fetches /api/config, extracts the key, and calls Stripe API to issue refunds"]
  Exploitability: [Easy / Medium / Hard] — [why]

━━━━━━━━━━━━━━━━━━━━━━━━━━
HIGH
━━━━━━━━━━━━━━━━━━━━━━━━━━
[same structure]

━━━━━━━━━━━━━━━━━━━━━━━━━━
MEDIUM
━━━━━━━━━━━━━━━━━━━━━━━━━━
[same structure]

━━━━━━━━━━━━━━━━━━━━━━━━━━
LOW / INFORMATIONAL
━━━━━━━━━━━━━━━━━━━━━━━━━━
[same structure]

━━━━━━━━━━━━━━━━━━━━━━━━━━
PASSED CHECKS
━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ [check that passed — one line each]
```

**Severity guide:**
- **Critical** — Exposed secret key, auth bypass, SQL/NoSQL injection, payment flow bypass, RCE
- **High** — Public API key in client bundle, missing webhook verification, sensitive PII in logs, IDOR
- **Medium** — Missing rate limiting, weak CORS, no input sanitization on non-critical fields, verbose errors
- **Low** — Missing security headers, outdated deps with no known active exploit, informational leakage

After the report, say:
> "I found [N] issues: [X] critical, [Y] high, [Z] medium, [W] low. Ready to fix — want me to start with the critical ones?"

Wait for confirmation before touching any file.

---

## Phase 3: Fix

Fix in severity order: Critical → High → Medium → Low.

For each fix, follow this exact structure:

```
[SEC-XX] — [Finding Name]
─────────────────────────
Vulnerability addressed:  [link back to the finding]
Why the original is unsafe: [specific mechanism — not generic]

BEFORE:
[exact original code]

AFTER:
[corrected code]

How the fix prevents the attack: [specific — map it to the attack vector]
Verification: [exact test to confirm the fix works — specific input, expected output]
Side effects to watch: [any downstream impact]
Further recommendations: [monitoring, key rotation, additional hardening]
```

After each edit, confirm with a grep or Read that the change is in place.

After all fixes, re-run the Phase 1 scan commands to verify no new issues were introduced and nothing was missed.

End with:
> "Audit complete."

Then list two sections:
1. **Fixed** — everything changed in this session with a one-line summary
2. **Manual actions required** — things only the user can do (rotate a key, enable 2FA on a provider, update a cloud policy) — listed as exact numbered steps, not vague recommendations
