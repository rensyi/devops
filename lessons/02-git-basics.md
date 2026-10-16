# 02 · Git basics

## Goal

You can say exactly what state every file is in (modified, staged, committed, or ignored), and the repo has a `.gitignore` that blocks secrets before any secret exists.

## Why this matters at work

The most common way a company leaks credentials is a developer running `git add .` with a `.env` file in the folder. Once a secret is pushed to a public repo, bots find it within minutes. Deleting it afterwards doesn't help, because it's in the history, so the key has to be revoked. A good `.gitignore` written on day one, plus knowing what you're staging, prevents a very expensive afternoon.

## Concepts

**The three areas.** Every change moves left to right:

```
working tree  --git add-->  staging area (index)  --git commit-->  repository (history)
```

- **Working tree:** the files you see and edit.
- **Staging area:** the exact snapshot your *next* commit will contain. You choose what goes in, even parts of a file (`git add -p`).
- **Repository:** the permanent history.

Two commands show the gaps between them:

| Command | Shows the difference between |
|---|---|
| `git diff` | working tree and staging |
| `git diff --staged` | staging and the last commit |

**Commit messages** are for the person debugging at 3am a year from now (often you). This repo uses the [Conventional Commits](https://www.conventionalcommits.org/) style:

```
type: short summary in the imperative, under ~50 chars

Optional body: WHY this change, not what (the diff shows what).
```

The common types are `feat`, `fix`, `docs`, `chore`, `ci`, `refactor`, and `test`. "Imperative" means writing the summary as a command: `add`, not `added` or `adds`.

**`.gitignore`** tells Git which untracked files to pretend don't exist. Key rules:

| Pattern | Matches |
|---|---|
| `name` | That name, in any folder |
| `dir/` | Folders only |
| `*.log` | A wildcard |
| `!pattern` | Un-ignores something matched earlier |
| A line starting with `#` | A comment. **Only at the start of a line**, never after a pattern. |

`.gitignore` does **not** affect files Git already tracks. Once a file is committed, ignoring it later changes nothing.

Docs: [gitignore](https://git-scm.com/docs/gitignore) · [Pro Git ch. 2](https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository)

## Install

Nothing new today.

## Steps

1. **Rewrite `README.md`.** It's nearly empty. Write a short real one: what this repo is (your DevOps learning path), how it's organized (`lessons/`, `docs/`), and a link to `lessons/README.md`. AI can draft it, but make the words yours.

2. **Look before you stage.** Run `git status` and then `git diff`. Then stage the README with `git add -p` and answer `y` or `n` per chunk, so you see exactly what you're staging. Run `git diff` again (empty?) and `git diff --staged` (your change?). Commit with a `docs:` message.

3. **Write `.gitignore`.** It must ignore:
   - Python: `__pycache__/`, `.venv/`, `*.pyc`, `.pytest_cache/`, `.ruff_cache/`
   - Secrets: `.env` and any `.env.*`, **but not** `.env.example`. That file will hold safe placeholder values everyone can see.
   - Terraform: `.terraform/`, and state files `*.tfstate` and `*.tfstate.*`
   - Claude's scratch folder: `.superpowers/`

   Group the patterns with comment lines so it reads like documentation.

4. **Prove it works.** Create empty files to test with: `.env`, `.env.production`, `.env.example`, and `app.pyc`. Run `git status --short --ignored`. Ignored files show with `!!`, and files Git would add show with `??`. You want everything flagged `!!` *except* `.env.example` and `.gitignore`. When it's right, delete the test files and commit `.gitignore` with a `chore:` message.

5. **Read your history** with `git log --oneline -5`. Two new, well-named commits, each doing one thing. Don't push yet: the Break comes first.

## Check

| Run | You should see |
|---|---|
| `touch .env && git status --short` | `.env` does **not** appear |
| `touch .env.example && git status --short` | `?? .env.example` (not ignored) |
| `git ls-files \| grep -c '^\.env$'` | `0` (`.env` has never been tracked) |
| `git log --oneline -3` | your `docs:` and `chore:` commits on top |

Clean up the test files afterwards (`rm .env .env.example`).

## Gate before merge

### Explain

1. Describe the three areas and which command moves a change between each.
2. You ran `git add -p` and said `n` to one chunk. Where is that chunk now, and will it be in your commit?
3. Why write `.gitignore` *before* any secrets exist, rather than when you first need it?

### Break

Do this on top of your two local commits, before pushing.

1. Create `config.txt` containing `API_KEY=fake123`, then add and commit it. You've just "leaked" a secret.
2. "Fix" it the way many people try first: add `config.txt` to `.gitignore`, then append a line to `config.txt`. Run `git status`.
3. Git still reports `config.txt` as modified. Explain why from what `git status` shows you.
4. Repair it properly. Since you haven't pushed, the leak only exists on your machine. Use `git reset HEAD~1` to undo the commit while keeping your files. Delete `config.txt`, take the line back out of `.gitignore`, and confirm `git log` and `git status` look like they did before step 1.

Then answer: if you *had* pushed this to a public repo, why would removing it in a new commit not be enough, and what must you do with the key?

### Review

A teammate's `.gitignore`:

```gitignore
# Python
__pycache__/
.venv/

# Secrets
.env  # never commit this!
```

They're confident `.env` is protected. Are they right? Test your answer in a scratch folder (`git init`, create `.env`, run `git status`) before asking Claude.

## Commit

Today you still push straight to `main`:

```bash
git push
```

From Lesson 03 on, every change goes through a branch and a pull request.

## Stretch

- Set a **global** ignore file for things that are about *your* machine, not the project (editor backups, `.DS_Store`): `git config --global core.excludesFile ~/.gitignore_global`.
- Fix a typo in your last commit message with `git commit --amend`. Note: only safe because it isn't pushed yet. Why does that matter?

## If you're stuck

| Symptom | Likely cause |
|---|---|
| A file is in `.gitignore` but still shows in `git status` | It's already tracked. `.gitignore` only hides *untracked* files. Untrack it with `git rm --cached <file>`. |
| `git commit` opened a strange editor (Vim) | You left off `-m`. Type `:wq` and Enter to save, or `:q!` to abort. To use VS Code instead: `git config --global core.editor "code --wait"`. |
| `git add -p` says `No changes` | The file is untracked (brand new). Use `git add <file>` first, or `git add -N <file>` and then `-p`. |
| A `!pattern` line doesn't un-ignore a file | Order matters, so the `!` line must come *after* the pattern it overrides. You also can't un-ignore a file inside an ignored folder. |
