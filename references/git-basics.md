# Git Basics for Vibe Coders

## Why Git Matters More for Vibe Coding

Traditional coding is incremental — you write a function, test it, write another. Easy to undo.

Vibe coding is different. You prompt AI and get 200 lines of changes across 10 files in 30 seconds. Without Git, one bad AI response can silently destroy days of work with no recovery path.

**Git = your undo button, your backup, your safety net.**

---

## Minimum Setup (do this before anything else)

```bash
# In your project folder
git init
git add .
git commit -m "Initial commit"
```

That's it. You now have version control.

---

## The Vibe Coder's Git Workflow

### Trunk-based (recommended for solo/small teams)

```
main (always working, always deployable)
  └── feature/add-login     ← AI works here
  └── feature/payment-page  ← one feature per branch
```

**Routine:**
1. Start work → `git checkout -b feature/what-youre-building`
2. AI makes changes → review the diff in VS Code Source Control or `git diff`
3. It works → `git add . && git commit -m "feat: describe what you built"`
4. Merge to main → `git checkout main && git merge feature/what-youre-building`
5. Something broke → `git checkout main` (your branch is expendable, main is safe)

---

## Essential Commands

```bash
git status                        # What changed?
git diff                          # Line-by-line what changed
git add .                         # Stage all changes
git commit -m "message"           # Save checkpoint
git log --oneline                 # See history
git checkout -b branch-name       # New branch
git checkout main                 # Back to safety
git stash                         # Temporarily shelve changes
git stash pop                     # Restore shelved changes
git reset --hard HEAD~1           # Undo last commit (destructive — use carefully)
```

---

## Before Every Major AI Task

```bash
git add . && git commit -m "checkpoint before AI refactor"
```

This gives you a one-command escape if the AI makes a mess:
```bash
git reset --hard HEAD~1
```

---

## .gitignore — Set This Up Before Your First Commit

Create `.gitignore` in your project root:

```gitignore
# Secrets — NEVER commit these
.env
.env.local
.env.production
.env*.local

# Dependencies (install locally with npm install / pip install)
node_modules/
__pycache__/
venv/
.venv/

# Build output
.next/
dist/
build/
*.pyc

# System junk
.DS_Store
Thumbs.db

# IDE files
.cursor/
.vscode/settings.json
```

**Critical:** Add `.gitignore` and commit it *before* you add any `.env` files or `node_modules`.

---

## If You Accidentally Committed a Secret

1. **Rotate the key immediately** — generate a new one in the provider's dashboard
2. **Revoke the old key** — treat it as compromised
3. Check your billing for unauthorized usage
4. Don't just delete the commit — the key is in git history and already public if pushed

To clean history (advanced — do this after rotating):
```bash
# Remove a file from all git history
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch .env" \
  --prune-empty --tag-name-filter cat -- --all
```
Or use [BFG Repo Cleaner](https://rtyley.github.io/bfg-repo-cleaner/) — simpler.

---

## No-CLI Options

- **VS Code**: Source Control panel (Ctrl+Shift+G) — visual add, commit, branch, diff
- **GitHub Desktop**: Full GUI, great for beginners
- **Cursor**: Has built-in Git panel — use it after every AI session
