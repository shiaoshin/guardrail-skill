# Database Safety for Vibe Coders

## The Highest-Risk Area in Vibe Coding

Databases are where the most catastrophic vibe coding incidents happen:
- Replit's AI agent deleted a production database despite explicit "don't touch" instructions
- Lovable-generated apps exposed entire user tables via misconfigured access permissions
- Tea/Sapphos breach: database had public access enabled, no app exploit needed

AI agents treat databases as mutable. Your data is irreplaceable. These two facts require deliberate guardrails.

---

## Before You Start: Database Rules

**Always establish these before AI touches your database:**

1. **Never run destructive operations on production without a backup**
2. **Use separate databases for development and production** — not the same instance
3. **Migrations go through version control** — never modify schema directly via a GUI without a migration file
4. **Principle of least privilege** — your app's DB user should only have SELECT/INSERT/UPDATE/DELETE on its own tables, not DROP TABLE or CREATE DATABASE

---

## Dangerous Operations — Intercept These

When AI suggests any of these, **pause and verify** before running:

| Command/Action | Risk Level | Safe Protocol |
|---|---|---|
| `DROP TABLE` | 🔴 CRITICAL | Backup first. Confirm this is dev/staging. |
| `DELETE FROM table` without WHERE | 🔴 CRITICAL | This wipes the entire table. Always add a WHERE clause. |
| Schema migration on production | 🟡 HIGH | Test on a copy first. Know how to roll back. |
| `TRUNCATE` | 🔴 CRITICAL | Same as DELETE without WHERE — permanent |
| Disabling Row Level Security (RLS) | 🔴 CRITICAL | In Supabase/Postgres: RLS is what keeps users from reading each other's data |
| Changing a column type | 🟡 HIGH | Can cause data loss if incompatible types. Backup first. |
| Granting broad DB permissions | 🟡 HIGH | `GRANT ALL ON ALL TABLES` is almost never correct |

---

## Supabase-Specific Guardrails

Supabase is the most common DB for vibe coders. Critical defaults:

**Row Level Security (RLS):**
- Supabase exposes your database via an API. Without RLS, *anyone* can read/write any row.
- **Always enable RLS on every table before any data goes in**
- Ask AI: *"Add appropriate RLS policies to this table. Users should only be able to read and write their own rows."*
- Check: Go to Table Editor → your table → Policies. If it says "RLS disabled" next to any table with user data, fix it now.

**Public vs. Service Role keys:**
- `SUPABASE_ANON_KEY` → safe for client-side. Respects RLS.
- `SUPABASE_SERVICE_ROLE_KEY` → bypasses ALL security. Never use in client-side code. Server only.

**Storage buckets:**
- Default is public. If storing user-uploaded files, set the bucket to private and generate signed URLs.

---

## Using ORMs (Recommended)

ORMs (Prisma, Drizzle, SQLAlchemy, ActiveRecord) automatically handle parameterized queries, which eliminates SQL injection. If AI offers to write raw SQL queries, push back:

*"Use Prisma/Drizzle for all database operations instead of raw SQL."*

Benefits:
- SQL injection protection by default
- Schema defined in code (version controlled)
- Migrations tracked automatically
- Type safety in TypeScript projects

---

## Backup Discipline

Before any schema change, migration, or data operation on production:

```bash
# PostgreSQL
pg_dump -U username -d database_name > backup_$(date +%Y%m%d_%H%M%S).sql

# Supabase: use the dashboard → Settings → Database → Backups
# Or: Supabase CLI → supabase db dump
```

Most managed platforms (Supabase, Railway, PlanetScale) have point-in-time recovery — know where to find it before you need it.

---

## Pre-Deployment Database Checklist

- [ ] Dev and production are separate databases
- [ ] RLS enabled on all user-data tables (Supabase)
- [ ] No `SUPABASE_SERVICE_ROLE_KEY` in client-side code
- [ ] Database user has minimum required permissions
- [ ] All queries use ORM or parameterized statements (no string concatenation)
- [ ] Migrations tracked in version control
- [ ] Backup exists before any schema migration
- [ ] Sensitive fields (passwords) hashed — never stored plaintext
- [ ] Storage buckets set to private if containing user uploads
