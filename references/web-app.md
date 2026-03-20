# Web App Security for Vibe Coders

## The Reality Check

45% of AI-generated code contains security vulnerabilities (Veracode 2025).
AI tools are good at avoiding *generic* flaws, but struggle with context-specific security decisions.
You can't outsource security judgment entirely to the AI — you need to know the basics.

---

## The Non-Negotiables

### 1. Never Build Custom Auth

Auth is where the most critical vulnerabilities live. AI will generate auth code that *looks* correct and *almost* works, with subtle flaws that only surface under attack.

**Use instead:**
- **NextAuth / Auth.js** — Next.js standard, supports many providers
- **Clerk** — hosted, easiest to set up, great DX
- **Supabase Auth** — if you're already on Supabase
- **Better Auth** — newer, framework-agnostic
- **Lucia** — lightweight, if you want more control

Tell the AI: *"Use [Clerk/NextAuth/etc.] for authentication. Do not write custom auth logic."*

### 2. Never Store Secrets in Code

See `git-basics.md` and the main SKILL.md. This also applies to:
- Never log secrets (check your console.log statements)
- Never return secrets in API responses
- Never put secrets in client-side code (anything with `NEXT_PUBLIC_` prefix in Next.js is public)

### 3. Validate All User Input

AI often skips input validation. Unvalidated input = SQL injection, XSS, command injection.

Ask AI explicitly: *"Add input validation and sanitization to all form fields and API endpoints. Use parameterized queries for all database operations."*

Common patterns to verify:
- Form inputs sanitized before DB writes
- API endpoints validate request body shape
- File uploads check type and size limits
- URL parameters don't go directly into queries

### 4. Use HTTPS Everywhere

All major deployment platforms (Vercel, Railway, Netlify, Render) give you HTTPS automatically. Never launch without it. Never use `http://` URLs in production code.

### 5. Authentication vs. Authorization

AI frequently implements authentication (who are you?) but forgets authorization (what are you allowed to do?).

After AI generates user-facing features, ask: *"Review this code and check that every endpoint verifies the logged-in user has permission to access or modify this specific resource. Not just that they're logged in."*

---

## Common AI-Generated Web App Vulnerabilities

| Vulnerability | What it looks like | How to catch it |
|---|---|---|
| **Hardcoded secrets** | `const apiKey = "sk-..."` in source | Grep for strings that look like keys |
| **Client-side auth checks** | `if (user.role === 'admin')` in frontend JS | Auth checks must always be on the server |
| **SQL injection** | String concatenation in queries | Demand parameterized queries |
| **Missing rate limiting** | No throttle on login, signup, or public APIs | Add rate limiting middleware on sensitive routes |
| **Exposed internal APIs** | Swagger UI publicly accessible | Check that debug/admin routes are not in production |
| **CORS misconfiguration** | `cors({ origin: '*' })` on an API with auth | Lock CORS to your actual frontend domain |
| **Insecure direct object references** | `/api/invoices/123` with no ownership check | Every resource fetch must verify ownership |

---

## Pre-Launch Security Checklist

Before sharing or deploying any web app with real users:

- [ ] No secrets in code or git history
- [ ] `.env` is in `.gitignore`
- [ ] Using an auth library, not custom auth
- [ ] All user inputs validated server-side
- [ ] Database uses parameterized queries (ORM handles this if using Prisma, Drizzle, SQLAlchemy, etc.)
- [ ] HTTPS enabled in production
- [ ] Admin/debug routes protected or removed
- [ ] CORS locked to your domain
- [ ] Rate limiting on auth endpoints
- [ ] Error messages don't leak stack traces or internal paths to users

---

## Quick AI Prompts That Help

```
"Review this code for OWASP Top 10 vulnerabilities and list any issues you find."

"Add server-side input validation to all API routes. Use Zod for schema validation."

"Check every API endpoint and confirm that resource ownership is verified before returning data."

"Are there any secrets, tokens, or credentials in this code that should be in environment variables instead?"
```
