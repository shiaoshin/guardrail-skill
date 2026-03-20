# Guardrail — AI/Vibe Coding Safety Skill

A safety skill for AI-assisted developers that protects against the most common and costly mistakes: leaked secrets, open databases, rogue AI agents, and deploying code that looks right but isn't.

Works with Claude Code, Cursor, Codex CLI, VS Code (Copilot), Gemini CLI, Windsurf, Goose, Amp, and 20+ other agents that support the [Agent Skills open standard](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills).

---

## What It Does

**Onboards** new vibe coders with 6 non-negotiables before any code gets written — version control, secrets management, AI output review, scope control, secret scanning, and planning discipline.

**Intercepts** dangerous operations in real time with a clear `⚠️ GUARDRAIL` warning, the specific risk in plain language, and a safe alternative offered immediately.

**Expands** into domain-specific guidance as the project grows — loading the right reference file at the moment it's relevant, not upfront.

---

## Installation

### One command (recommended)
```bash
npx skills add shiaoshin/guardrail-skill
```
Installs to `~/.agent/skills/`, the universal path recognized by Claude Code, Cursor, Codex, Gemini CLI, and most other agents.

### Claude.ai
1. Download `guardrail.skill` from [Releases](../../releases)
2. Go to **Settings → Capabilities → Skills**
3. Upload the `.skill` file

### Manual install

Clone or download this repo, then copy the `guardrail` folder to the right path for your tool:

| Agent | Personal (all projects) | Project-level (repo) |
|---|---|---|
| Claude Code | `~/.claude/skills/` | `.claude/skills/` |
| Cursor | `~/.cursor/skills/` | `.cursor/skills/` |
| Codex CLI | `~/.codex/skills/` | `.agents/skills/` |
| Gemini CLI | `~/.gemini/skills/` | `.gemini/skills/` |
| VS Code / Copilot | `~/.github/skills/` | `.github/skills/` |
| Windsurf | `~/.codeium/skills/` | `.agents/skills/` |
| **Universal (any agent)** | `~/.agent/skills/` | `.agents/skills/` |

> ⚠️ **Gemini CLI note:** Progressive loading of `references/` subdirectories has a [known compatibility issue](https://github.com/google-gemini/gemini-cli/issues/15895). Core `SKILL.md` functionality works; fix is pending upstream.

---

## Skill Structure

```
guardrail/
├── SKILL.md                    # Core instructions + danger interception logic
└── references/
    ├── git-basics.md           # Git workflow and commands for vibe coders
    ├── web-app.md              # OWASP Top 10, input validation, auth libraries
    ├── database.md             # Supabase RLS, migrations, destructive ops
    ├── deployment.md           # Env vars, staging, CI/CD, billing alerts
    ├── auth.md                 # Auth libraries, JWT, authorization vs authentication
    └── secret-scanning.md     # Pre-commit hooks, gitleaks, GitHub scanning, custom scripts
```

Reference files load on demand — only when the user's project enters that domain.

---

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

---

## Who It's For

Designers, marketers, non-technical founders, and anyone using vibe coding tools (Cursor, Replit, Lovable, Bolt, v0, Claude Code) to build their first real project. Experienced developers won't need it — but their junior teammates will.

---

## Background

Built in response to documented patterns in the vibe coding ecosystem:
- 45% of AI-generated code introduces security vulnerabilities (Veracode, 2025)
- Replit's AI agent deleted a production database despite explicit instructions not to (SaaStr, 2025)
- 170 of 1,645 Lovable-generated apps had auth vulnerabilities exposing user data (2025)
- API keys scraped from public GitHub repos within minutes of being pushed

---

## License

MIT — use, modify, and redistribute freely. Attribution appreciated.

## Author

Built by **Luke Lin** with AI assistance.  
Part of a consulting practice focused on automation, design, and web for small and mid-sized businesses.
