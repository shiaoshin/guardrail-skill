# Secret Scanning for Vibe Coders

## Why This Exists

Claude cannot inspect your codebase for hardcoded secrets. It sees what you paste into the conversation — not your files on disk, your git history, or the code your AI tool just generated. That blind spot is real, and it needs to be closed by tooling you own.

An API key committed to GitHub is typically scraped and exploited within **5 minutes**. Automated bots continuously monitor public repos. One slip = unexpected charges, data breach, or both.

This reference gives you options ranked by setup effort, from zero-config cloud tools to a custom pre-commit hook Claude can build for you right now.

---

## Option 1: Zero Setup — GitHub Secret Scanning (Free, Immediate)

If your repo is on GitHub, turn this on now:
1. Repo → **Settings** → **Security** → **Secret scanning** → Enable
2. Also enable **Push protection** — this blocks pushes that contain known secret patterns before they land

GitHub scans for 200+ secret types (AWS, OpenAI, Stripe, Twilio, etc.) automatically. If a secret is detected in a push, GitHub alerts you and (with push protection) blocks the commit.

**Limitation**: Only works on GitHub. Only catches patterns GitHub knows. Doesn't scan local files before commit.

---

## Option 2: Pre-Commit Hook — Blocks Leaks Before They Happen

A pre-commit hook runs automatically every time you run `git commit`. If it finds a secret pattern, the commit is rejected and nothing reaches your repo.

### Using `detect-secrets` (Python — recommended)

```bash
# Install
pip install detect-secrets

# Initialize a baseline (tells the tool what's already in your repo so it doesn't spam you)
detect-secrets scan > .secrets.baseline

# Install the pre-commit hook
pip install pre-commit
```

Create `.pre-commit-config.yaml` in your project root:
```yaml
repos:
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.4.0
    hooks:
      - id: detect-secrets
        args: ['--baseline', '.secrets.baseline']
```

```bash
# Activate the hook
pre-commit install

# Test it manually
pre-commit run --all-files
```

Now every `git commit` automatically checks for secrets. Commit blocked if any found.

---

### Using `gitleaks` (Go binary — no Python needed)

```bash
# macOS
brew install gitleaks

# Linux
curl -sSfL https://raw.githubusercontent.com/gitleaks/gitleaks/main/scripts/install.sh | sh

# Windows: download from https://github.com/gitleaks/gitleaks/releases
```

Add to `.pre-commit-config.yaml`:
```yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.0
    hooks:
      - id: gitleaks
```

Or run manually at any time:
```bash
# Scan your working directory
gitleaks detect --source . --verbose

# Scan your entire git history
gitleaks detect --source . --log-opts="--all" --verbose
```

---

## Option 3: Ask Claude to Build You a Custom Script

If you want something lightweight without external dependencies, Claude can generate a custom script tailored to your project's specific secret patterns. Use this prompt:

```
"Build me a secret scanning script for this project. It should:
1. Scan all files in the project directory (excluding node_modules, .git, dist, .next)
2. Flag any line that looks like it contains a hardcoded secret: API keys, tokens, 
   passwords, connection strings, private keys
3. Know that my project uses [OpenAI / Stripe / Supabase / etc.] — flag those key patterns specifically
4. Output file name, line number, and the flagged line
5. Exit with a non-zero code if anything is found (so it can be used in CI)
Save it as scripts/scan-secrets.sh (or .py if you prefer Python)"
```

Then wire it into a pre-commit hook:
```bash
# .git/hooks/pre-commit (create this file, make it executable)
#!/bin/sh
bash scripts/scan-secrets.sh
if [ $? -ne 0 ]; then
  echo "⚠️  GUARDRAIL: Potential secrets found. Commit blocked."
  echo "Review the output above and move secrets to .env before committing."
  exit 1
fi
```

```bash
chmod +x .git/hooks/pre-commit
```

---

## Option 4: CI/CD Pipeline Scanning — Catch What Slips Through

Even with local hooks, someone might bypass them (`git commit --no-verify`). Add a pipeline check as a second layer.

### GitHub Actions workflow

Create `.github/workflows/secret-scan.yml`:
```yaml
name: Secret Scanning

on:
  push:
    branches: [main, staging]
  pull_request:

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history for git log scanning

      - name: Run Gitleaks
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

This blocks merges to `main` if secrets are detected in the diff.

---

## Option 5: Scan Your Existing Git History

If you haven't been scanning until now, your history might already contain leaked secrets. Run this once:

```bash
# With gitleaks — scans entire commit history
gitleaks detect --source . --log-opts="--all" --verbose --report-format json --report-path leaks-report.json

# Review the report
cat leaks-report.json
```

If it finds anything:
1. Rotate the key immediately (assume it's already been seen)
2. Check billing/usage on that service
3. Clean history with BFG or `git filter-repo` (won't help if repo was ever public — focus on rotation)

---

## What Gets Detected

These tools catch the most dangerous patterns:
- `sk-...` → OpenAI keys
- `pk_live_...` / `sk_live_...` → Stripe keys
- `AKIA...` → AWS access keys
- `eyJ...` → JWT tokens (base64-encoded)
- `postgres://user:password@...` → DB connection strings with credentials
- `ghp_...` / `ghs_...` → GitHub tokens
- Private key blocks (`-----BEGIN RSA PRIVATE KEY-----`)
- `.env` file content patterns (KEY=value pairs)

---

## Recommended Setup by Experience Level

| Level | Recommended setup | Time to set up |
|---|---|---|
| Complete beginner | GitHub Secret Scanning (Option 1) | 2 minutes |
| Comfortable with terminal | `gitleaks` pre-commit hook (Option 2) | 10 minutes |
| Want custom coverage | Ask Claude to build a script (Option 3) | 5 minutes |
| Team / production project | All of the above + CI/CD pipeline (Option 4) | 30 minutes |

**Minimum bar for any project with real API keys**: Option 1 + `.env` in `.gitignore`. Everything above that is defense in depth.
