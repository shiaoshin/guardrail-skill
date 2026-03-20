# Guardrail — AI/Vibe Coding Safety Skill for Claude

A Claude skill that protects first-time and early-stage AI-assisted developers from the most common and costly mistakes: leaked secrets, open databases, rogue AI agents, and deploying code that looks right but isn't.

## What It Does

**Onboards** new vibe coders with the 6 non-negotiables before any code gets written — version control, secrets management, AI output review, scope control, secret scanning, and planning discipline.

**Intercepts** dangerous operations in real time with a clear `⚠️ GUARDRAIL` warning, the specific risk explained in plain language, and a safe alternative offered immediately.

**Expands** into domain-specific guidance as the project grows — loading the right reference file at the moment it's relevant rather than front-loading a 10,000-word manual.

## Installation

### Claude.ai
1. Download `guardrail.skill` from [Releases](../../releases)
2. Go to **Settings → Capabilities → Skills**
3. Upload the `.skill` file

### Claude Code
```bash
# Personal (available across all projects)
cp -r guardrail ~/.claude/skills/

# Project-level (committed to repo, shared with team)
cp -r guardrail .claude/skills/
```

## Skill Structure

```
guardrail/
├── SKILL.md                        # Core instructions + danger interception logic
└── references/
    ├── git-basics.md               # Git workflow and commands for vibe coders
    ├── web-app.md                  # OWASP Top 10, input validation, auth libraries
    ├── database.md                 # Supabase RLS, migrations, destructive ops
    ├── deployment.md               # Env vars, staging, CI/CD, billing alerts
    ├── auth.md                     # Auth libraries, JWT, authorization vs authentication
    └── secret-scanning.md         # Pre-commit hooks, gitleaks, GitHub scanning, custom scripts
```

Reference files load on demand — only when the user's project enters that domain.

## Danger Interception Triggers

| Signal | Risk | Action |
|---|---|---|
| Pasting an API key into chat | Key logged and leaked | Block + explain `.env` files |
| `DROP TABLE` / delete DB / reset migrations | Permanent data loss | Verify backup exists first |
| `git push --force` to main | Overwrites shared history | Suggest `--force-with-lease` |
| "Just hardcode it for now" | Keys get committed, stay forever | Refuse + offer `.env` setup |
| No `.gitignore` before first commit | Secrets in git history | Generate `.gitignore` immediately |
| Custom auth from scratch | High CVE risk | Redirect to NextAuth/Clerk/Supabase Auth |
| Unrecognized npm/pip package | Slopsquatting / malware risk | Verify on npm/PyPI before install |
| No secret scanning setup | Silent leaks undetected | Prompt to set up scanning |

## Who It's For

Designers, marketers, non-technical founders, and anyone using vibe coding tools (Cursor, Replit, Lovable, Bolt, v0, Claude Code) to build their first real project. Experienced developers won't need it — but their junior teammates will.

## Background

Built in response to documented patterns in the vibe coding ecosystem:
- 45% of AI-generated code introduces security vulnerabilities (Veracode, 2025)
- Replit's AI agent deleted a production database despite explicit instructions not to (SaaStr, 2025)
- 170 of 1,645 Lovable-generated apps had auth vulnerabilities exposing user data (2025)
- API keys scraped from public GitHub repos within minutes of being pushed

The skill follows the [Agent Skills open standard](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills) (released December 2025) and is compatible with Claude Code, Claude.ai, and any agent platform that supports the `SKILL.md` format.

## Compatibility

| Platform | Supported |
|---|---|
| Claude.ai (Pro/Max/Team/Enterprise) | ✅ |
| Claude Code | ✅ |
| Cursor (SKILL.md standard) | ✅ |
| Codex CLI | ✅ |

## License

MIT — use, modify, and redistribute freely. Attribution appreciated.

## Author

Built by **Luke Lin** with AI assistance.  
Part of a consulting practice focused on automation, design, and web for small and mid-sized businesses.
