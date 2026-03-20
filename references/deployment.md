# Deployment Safety for Vibe Coders

## The "It Works on My Machine" Problem

Local success means almost nothing about production behavior. AI-generated deployment configs (CI/CD workflows, Dockerfiles, cloud IAM, Terraform) are a documented source of dangerous defaults — they "work" while embedding security and cost risks.

---

## Deployment Readiness Checklist

Before going live with anything:

**Security**
- [ ] All secrets in environment variables, not code
- [ ] HTTPS enforced (not optional redirect, actually enforced)
- [ ] No debug endpoints or admin panels publicly accessible
- [ ] Error pages don't expose stack traces to users
- [ ] CORS locked to your actual domain(s)
- [ ] Auth required on everything that should require auth

**Stability**
- [ ] App tested locally against a copy of production data structure
- [ ] Database migrations tested on a staging environment first
- [ ] Rollback plan exists (git tag, previous deploy snapshot)
- [ ] Environment variables set in the deployment platform, not uploaded as files

**Cost**
- [ ] Set billing alerts on cloud providers before you deploy
- [ ] Understand what scales (and costs) with traffic
- [ ] Review any AI-suggested cloud service tiers — AI often defaults to over-provisioned setups

---

## Platform-Specific Guardrails

### Vercel
- Environment variables: Project Settings → Environment Variables. Never commit `.env.production`.
- Preview deployments use preview env vars — keep them separate from production values
- Functions have a 10s timeout on free tier — AI often generates logic that silently fails here
- Bandwidth limits on free tier — set an alert before you go viral

### Railway
- Has built-in secrets management — use it, don't use `.env` files in the repo
- Generates a public URL by default — intentional, but be aware everything is live immediately
- Watch the usage dashboard. Unexpected traffic or runaway processes = surprise bills.

### Replit
- "Always on" costs money — don't enable it until you actually need it
- Replit Secrets manager for env vars — don't put them in `main.py` or `index.js`
- Replit's AI agent has file system access by default — watch what it changes during a session
- Export/backup your Repl regularly if the project matters

### AWS / GCP / Azure (if AI suggests these)
- Red flag: AI suggesting managed cloud services to a beginner without cost discussion
- IAM roles: demand least-privilege. Never use root credentials. Never create IAM users with `AdministratorAccess` for your app.
- Set billing alerts/budgets *before* provisioning anything
- AI-generated Terraform/CDK/CloudFormation should be reviewed by someone who understands it before applying

---

## CI/CD Guardrails

AI-generated GitHub Actions, GitLab CI, and similar configs are a common source of hidden risks:

**Things to verify in any AI-generated workflow file:**
- Secrets referenced as `${{ secrets.NAME }}` not hardcoded
- Production deployments only trigger on `main`, not on every branch push
- No `--force` flags on git operations in CI
- Permissions scoped to what the workflow actually needs
- Third-party Actions pinned to a specific commit SHA, not just `@latest` (supply chain attack vector)

**Safe pattern for reviewing:**
```
"Review this GitHub Actions workflow for security issues. 
Check: Are secrets handled safely? Does production only deploy from main? 
Are third-party actions pinned to commit SHAs?"
```

---

## Staging Environments

If you have real users, you need a staging environment. This is not optional once you're past hobby-project stage.

**Minimum viable staging:**
- Separate deployment (e.g., `staging.yourapp.com` vs `yourapp.com`)
- Separate database (no production data in staging)
- Separate environment variables
- Always test migrations, schema changes, and major features in staging first

**Telling AI about your environments:**
```
"This project has three environments: local (my machine), staging (staging.app.com), 
and production (app.com). Each has its own database and environment variables. 
When you write deployment-related code, respect these boundaries."
```

---

## Runaway Costs — A Common Vibe Coding Incident

AI doesn't think about cost. Several real incidents involve vibe-coded apps racking up $500-$5,000+ cloud bills within hours of launch due to:
- No rate limiting → DDoS / spam fills database, hits API limits
- AI-suggested over-provisioned services
- Infinite retry loops in AI-generated code hitting paid APIs
- Accidental public S3 buckets serving as CDN (and you're paying egress)

**Before launch:**
1. Set a billing alert at $10, $50, $100 on every cloud account
2. Review every external API call AI generated — understand what costs money per call
3. Add rate limiting to any public-facing endpoints that call paid APIs
4. Know how to shut everything down quickly if costs spike
