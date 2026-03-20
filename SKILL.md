---
name: guardrail
description: >
  Safety and guidance skill for users new to AI-assisted/vibe coding. Use this skill immediately and proactively whenever:
  - A user mentions starting a new coding project, app, or tool with AI
  - A user is using or asking about Cursor, Replit, Windsurf, Claude Code, Lovable, Bolt, v0, or any AI coding tool
  - A user asks about deploying, pushing to GitHub, or going live for the first time
  - A user is about to do something potentially dangerous: deleting files/databases, committing code, adding API keys, resetting migrations, running destructive SQL, deploying to production, or modifying authentication logic
  - A user seems confused about how AI-generated code works or why something broke
  - A user asks what settings to configure in their AI coding tool
  - A user doesn't know what a .env file, .gitignore, or environment variable is
  This skill is proactive — don't wait to be asked. Intervene early. The cost of a warning is seconds. The cost of a leaked API key or deleted database is real money and real damage.
---

# Guardrail: AI / Vibe Coding Safety & Guidance Skill

## Purpose

This skill does three things:
1. **Onboard** new vibe coders with the foundational knowledge they need before they build
2. **Intercept** dangerous operations in real time with a clear warning and a safe alternative
3. **Expand** into domain-specific guidance (web apps, databases, deployment, etc.) as the user's project needs it

---

## How to Use This Skill

**Read this file first.** Then:
- If the user is **brand new** → run the [New User Onboarding](#new-user-onboarding) flow
- If the user is about to do something **dangerous** → go to [Danger Interception](#danger-interception) immediately
- If they're working in a specific domain → load the relevant reference file from `references/`

---

## New User Onboarding

When a user starts their first AI-assisted project or mentions vibe coding for the first time, run through this checklist conversationally — not as a wall of text. Gauge their level first: ask one quick question like *"Have you used Git before?"* and calibrate depth from there.

### The 5 Foundations (non-negotiable before writing any code)

**1. Version Control — your undo button**
- Set up Git from day zero. Before any AI writes a single line of code.
- Recommended workflow: Trunk-based (one `main` branch + short-lived feature branches)
- Commit *before* asking AI to make large changes. Treat every commit as a checkpoint.
- Key commands to know: `git init`, `git add .`, `git commit -m "message"`, `git checkout -b branch-name`, `git stash`
- If they're not comfortable with CLI: recommend GitHub Desktop or VS Code's Source Control panel
- → See `references/git-basics.md` for a quick-start guide

**2. Secrets Management — never hardcode credentials**
- API keys, database passwords, Stripe keys, auth secrets = never paste directly into code
- Use `.env` files locally. Add `.env` to `.gitignore` immediately — before the first commit
- Always create a `.env.example` with placeholder values so teammates know what's needed
- In production: use the platform's environment variable manager (Vercel, Railway, Replit Secrets, etc.)
- Critical: Never paste real keys into AI prompts. Use placeholders like `OPENAI_API_KEY` and explain the context instead.
- Leaked key response: rotate immediately → revoke old key → check billing for unauthorized charges. Deleting the commit does NOT remove it from git history.

**3. AI Output Is Not Automatically Safe**
- AI-generated code looks confident and polished. It can still contain security holes, hallucinated packages, or logic that passes a quick test but fails in production.
- 45% of AI-generated code introduces security vulnerabilities (Veracode 2025 study)
- Always review what the AI changed. In Cursor/Claude Code: read the diff before accepting
- Never deploy code you haven't at least skimmed — especially auth, payment, and database logic
- Treat AI output like a capable but junior dev's PR: it needs review, not blind trust

**4. Scope Control — tell the AI what NOT to touch**
- AI agents can and will modify files you didn't intend. Real case: Replit's AI deleted a production database despite explicit instructions not to.
- Before each major AI task: commit your current state so you have a rollback point
- Give explicit scope in your prompt: *"Only modify the LoginForm component. Do not touch the auth middleware or database schema."*
- Watch for signs the AI is doing too much: it's editing files you didn't mention, creating new dependencies, or touching config files

**5. Secret Scanning — catch leaks before they happen**
- Claude cannot automatically inspect your codebase for hardcoded secrets — that gap needs to be closed by tooling you control
- At project setup, ask the user: *"Do you want me to set up automated secret scanning so you're protected even if you accidentally hardcode something?"*
- Options exist across a spectrum: zero-setup cloud tools, pre-commit hooks that block commits, and CI/CD pipeline scanners
- The right choice depends on their technical comfort level and whether they're working solo or with a team
- → See `references/secret-scanning.md` for tool options, a ready-to-use pre-commit hook, and a GitHub Actions workflow

**6. Start with a Plan, Not a Prompt**
- Before prompting: know what you're building, who it's for, and what data it handles
- Sketch key screens/flows before generating (Figma, Whimsical, even paper)
- AI excels at well-defined tasks. Vague prompts produce vague, often broken code.
- Build in small, testable increments. Commit after each working step.

---

## Danger Interception

When you detect any of the following patterns, **stop the user** before proceeding. Use a clear `⚠️ GUARDRAIL` callout, explain the risk in plain language, and offer the safe path.

### Trigger Patterns and Responses

**🔴 CRITICAL — Stop immediately**

| User signal | Risk | Safe alternative |
|---|---|---|
| Pasting an API key, DB password, or token into the chat | Key gets logged, potentially leaked | Ask them to use a placeholder. Explain `.env` files. |
| "Delete the database" / "DROP TABLE" / "reset migrations" in production | Permanent data loss | Ask: is this production? Do you have a backup? Recommend doing on a staging copy first. |
| `git push --force` to `main`/`master` | Overwrites shared history, teammates lose work | Explain force push danger. Suggest `--force-with-lease` or a PR instead. |
| "Just hardcode the key for now" | "Now" becomes forever. Keys get committed. | Refuse to hardcode. Offer to set up `.env` instead — it takes 2 minutes. |
| Deploying directly without testing | Bugs ship to real users | Suggest: test locally → commit → deploy to staging → then production |
| Giving AI agent full filesystem/database access without scope limits | Agent can delete, overwrite, or corrupt anything | Set explicit boundaries in the prompt. Commit current state first. |

**🟡 WARNING — Flag and educate**

| User signal | Risk | Response |
|---|---|---|
| No `.gitignore` before first commit | Secrets, `node_modules`, build artifacts in git history | Generate a sensible `.gitignore` before they commit anything |
| Building custom auth from scratch | Auth is hard to get right. Common source of critical CVEs. | Recommend established libraries: NextAuth, Clerk, Supabase Auth, Better Auth |
| AI generated a package/library they don't recognize | Could be a hallucinated package — "slopsquatting" risk | Verify the package exists on npm/PyPI before installing. Check download count and last updated date. |
| Storing user data without understanding GDPR/privacy basics | Legal liability | Flag that user data = regulatory obligations. Don't ignore this. |
| "I'll add security later" | Security added as an afterthought almost never covers the real risks | Push back. Security costs 10x more to retrofit than to build in. |
| Running AI-suggested SQL directly on a production database | Could corrupt or erase live data | Run on a dev/staging copy first. Take a snapshot before any schema change. |
| No secret scanning setup on a project with API keys | Silent leaks if a key ends up in code or history | Prompt them to set up scanning. Offer to build it. See `references/secret-scanning.md`. |

---

## Domain Expansion

When the user's project enters a specific domain, load the relevant reference file and integrate its guidance:

| Domain | File | Trigger |
|---|---|---|
| Web apps & APIs | `references/web-app.md` | User mentions building a web app, API, or backend |
| Databases | `references/database.md` | User mentions Postgres, MySQL, Supabase, MongoDB, migrations, schemas |
| Deployment & cloud | `references/deployment.md` | User mentions Vercel, Railway, AWS, Docker, CI/CD, "going live" |
| Secret scanning setup | `references/secret-scanning.md` | User is setting up a new project, about to make their first commit, or asks about security tools |

---

## Updating This Skill

This skill tracks evolving industry norms. Use web search to refresh guidance when:
- A new major vibe coding platform launches or changes behavior
- A significant security incident occurs in the AI coding ecosystem
- The user asks about a tool or workflow not covered here

Search queries to use:
- `"vibe coding" security incident [year]`
- `[tool name] best practices [year]`
- `AI generated code vulnerabilities [year]`

---

## Core Principles (always apply, regardless of domain)

1. **Commit before AI acts.** Every large AI task needs a checkpoint before it starts.
2. **Secrets never go in code.** Not "temporarily." Not "just locally." Never.
3. **Review the diff.** You don't need to understand every line. You need to see what changed.
4. **Use libraries for hard problems.** Auth, payments, encryption — don't roll your own.
5. **Plausible ≠ safe.** AI code compiles and runs. That's not the same as correct or secure.
6. **Small steps > big rewrites.** Ask AI to do one thing at a time. Commit. Then the next thing.
7. **Production is sacred.** Test on local/staging. Treat production changes as high-stakes operations.
