# Authentication & Authorization for Vibe Coders

## Why This Is the Highest-Risk Feature to Vibe Code

Auth is deceptively hard. AI generates auth that passes all your manual tests, then fails when an attacker probes it systematically. Real case: Base44 (vibe coding platform) had publicly accessible Swagger UIs that let any user access any private app with just the `app_id` value.

**Rule: Never write custom authentication. Always use a battle-tested library.**

---

## Auth Library Decision Guide

| Need | Recommended | Why |
|---|---|---|
| Next.js with social login | **Clerk** or **NextAuth/Auth.js** | Clerk: easiest setup, hosted; NextAuth: self-hosted, more control |
| Supabase project | **Supabase Auth** | Built in, RLS integrates automatically |
| Any backend | **Better Auth** | Framework-agnostic, modern, well-maintained |
| Need full control | **Lucia** | Lightweight, forces you to understand the flow |
| Enterprise (SSO/SAML) | **Auth0** or **WorkOS** | Handles SAML, SCIM, enterprise requirements |

Tell AI: *"Use [library] for authentication. Do not write JWT handling, session management, or password hashing manually."*

---

## Authentication vs. Authorization — AI Gets This Wrong

**Authentication**: Who are you? (login, session)
**Authorization**: What are you allowed to do? (permissions, ownership)

AI almost always gets authentication right and misses authorization. After building any feature with user data:

```
"Review all endpoints in this feature. For each one, confirm that:
1. The user is authenticated (logged in)
2. The user is authorized to access THIS SPECIFIC resource (not just any resource of this type)
For example: a user should be able to view /invoices/123 only if invoice 123 belongs to them."
```

---

## What Good Auth Code Looks Like

**Session check (server-side — Next.js example):**
```typescript
// ✅ Server-side auth check
const session = await getServerSession(authOptions)
if (!session) redirect('/login')

// ✅ Ownership check — not just "is logged in"
const invoice = await db.invoice.findFirst({
  where: { id: params.id, userId: session.user.id } // user can only get their own
})
if (!invoice) notFound()
```

**What AI sometimes generates instead:**
```typescript
// ❌ Client-side only check — bypassable
if (typeof window !== 'undefined' && !localStorage.getItem('token')) {
  router.push('/login') // useless — server still serves the data
}

// ❌ Auth but no ownership check
const invoice = await db.invoice.findFirst({
  where: { id: params.id } // any logged-in user can get any invoice
})
```

---

## Critical Auth Patterns to Verify

**Password storage**: Demand bcrypt or Argon2. Never MD5, SHA1, or plaintext.
```
"Confirm that passwords are hashed with bcrypt (cost factor 12+) or Argon2. Never store plaintext passwords."
```

**JWT secrets**: If AI generates JWT handling, the secret must be long, random, and in an env var.
```
"The JWT_SECRET must come from an environment variable and be at least 32 random characters."
```

**Session invalidation**: After password change or logout, old sessions/tokens must be invalid.

**CSRF protection**: If using session cookies, ensure CSRF protection is enabled. Modern auth libraries handle this — custom code often doesn't.

**Rate limiting on login**: Brute-force attacks try thousands of passwords. Your login endpoint needs rate limiting.
```
"Add rate limiting to the login endpoint: maximum 5 attempts per IP per 15 minutes."
```

---

## Roles & Permissions

If your app has admin users, different subscription tiers, or any access levels:

1. Define roles explicitly in your database (`user`, `admin`, `moderator`)
2. Store role in the database — not in the client-side session alone
3. Check role server-side on every privileged action

```
"Add role-based access control. Admin users have role='admin' in the database. 
Check this server-side before any admin action. Never trust role claims from the client."
```

---

## Pre-Launch Auth Checklist

- [ ] Using a battle-tested auth library, not custom code
- [ ] Passwords hashed with bcrypt/Argon2 (if handling passwords)
- [ ] JWT secret in environment variable, 32+ chars
- [ ] Every protected route checks auth server-side
- [ ] Every resource fetch verifies ownership (not just auth)
- [ ] Rate limiting on login/signup endpoints
- [ ] CSRF protection enabled
- [ ] OAuth redirect URIs locked to your domain (not wildcards)
- [ ] Admin routes protected by role check, not just login check
- [ ] Sessions invalidated on logout
