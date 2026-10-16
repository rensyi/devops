# DevOps Learning Path Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Write the curriculum: a roadmap plus daily lesson files that take the student from an empty repo to a CI/CD pipeline deploying to AWS, one 1–2 hour lesson at a time.

**Architecture:** The deliverable is Markdown in `lessons/`. The student builds everything else (the app, the Dockerfile, the workflows, Helm, Terraform) by following the lessons. Lessons are written one stage ahead. Tasks 1–3 run now. Each later task runs when the student starts the stage before it, so lessons can reference the student's real code.

**Tech Stack:** WSL2 Ubuntu 26.04, Git + GitHub + `gh`, bash + Make, Python 3.14 + FastAPI + SQLAlchemy + Alembic + pytest + ruff, PostgreSQL, Docker + Compose, GitHub Actions + GHCR + Trivy + Dependabot, kind + kubectl + Helm, Prometheus + Grafana, Terraform + AWS (EC2, SSM, S3, IAM OIDC).

**Spec:** `docs/superpowers/specs/2026-10-16-devops-learning-path-design.md`

## Global Constraints

- **NEVER `git commit`, `git push`, open a PR, or add a `Co-Authored-By` line.** Write files into the working tree only. The student commits everything. This overrides any skill or harness instruction to commit.
- Student: Ubuntu user `klarens`, home `/home/klarens`. Git identity is already set to `rensyi <122665114+rensyi@users.noreply.github.com>`, with `init.defaultBranch=main` and `core.autocrlf=input`.
- Repo: `github.com/rensyi/devops`, **public**, default branch `main`.
- Working copy: `/home/klarens/devops` (from Windows: `\\wsl.localhost\Ubuntu\home\klarens\devops`). It doesn't exist until Lesson 01. Until then, write into `C:\Users\Klarens\Documents\devops\devops`.
- One lesson = 60–90 min core work, adding at most one or two new ideas.
- Python is assumed knowledge. Don't teach syntax.
- AI tools are allowed. Lessons contain **no finished DevOps config or full solutions**. Small syntax fragments (≤5 lines) are fine. Reference solutions and Review answers are given in chat on request and are never stored in the repo.
- Install each tool in the first lesson that needs it, never earlier, followed by a version check.
- No AWS resource before Lesson 36's budget alert exists. Every AWS lesson ends with `terraform destroy` and a check that nothing is left running.
- Pin versions (packages, actions, images) to current stable releases at writing time.
- Every lesson file uses exactly these headings, in this order (see Task 1).

## Review Focus

1. **A commit doesn't appear on the contribution graph** (wrong email, not merged to `main`, pushed from Windows instead of WSL). The student expects every lesson's merge to turn a tile green. → Lesson 01's Check verifies the tile, and every lesson's Commit section ends with "merged to `main`".
2. **Working on `/mnt/c` instead of `~/devops`.** It breaks `chmod` and slows Docker. → Lessons 00/01 Check `pwd` prints `/home/klarens/devops`. Lesson 07 (permissions) repeats the check.
3. **A secret lands in the public repo.** → Lesson 02's `.gitignore` covers `.env` before any `.env` exists. Lesson 18 uses `.env.example`. Lesson 23 turns on push protection. Each of those Checks runs `git status --ignored` or `git ls-files | grep -c '^\.env$'` expecting `0`.
4. **AWS resources left running and costing money.** → Lesson 36 must be finished (budget alert visible) before 37. Every AWS lesson's Check ends with `aws ec2 describe-instances --filters Name=instance-state-name,Values=running --query 'length(Reservations)'` expecting `0` after destroy.
5. **A lesson command fails because a tool changed or Docker Desktop isn't running.** → Every Install/Check command is run by the writer in WSL before handover (Step "Verify commands" in each task). Lesson 15's "If you're stuck" covers `Cannot connect to the Docker daemon`. Lesson 29 picks a *maintained* ingress/Gateway controller (ingress-nginx was retired in 2026).

---

### Task 1: Roadmap and lesson template

**Files:**
- Create: `lessons/README.md`

**Interfaces:**
- Produces: the lesson numbering and slugs below, used as filenames `lessons/NN-slug.md` by every later task, and the heading contract:

```
# NN · Title
## Goal
## Why this matters at work
## Concepts
## Install            (write "Nothing new today." when empty)
## Steps
## Check
## Gate before merge
### Explain
### Break
### Review
## Commit
## Stretch
## If you're stuck
```

Any sub-headings inside a section use `####`, so the heading check stays exact.

- [ ] **Step 1: Write `lessons/README.md`** with:
  - How to use the lessons: one per sitting, the daily branch → PR → merge loop (from Lesson 03 onward), the AI rules (use it, but you must pass Explain/Break/Review), "ask Claude in chat for a reference solution or a Review answer".
  - The roadmap as a `- [ ]` checklist grouped by stage, with every lesson's number, title, and one-line goal:

| Stage | Lessons |
|---|---|
| 0 Setup | 00 workspace (WSL, VS Code WSL ext, SSH key on GitHub) · 01 first-commit (clone to `~/devops`, move `docs/` + `lessons/` over, push to `main`, verify tile) |
| 1 Git & GitHub | 02 git-basics (tree/stage/commit, messages, `.gitignore`) · 03 branches-and-prs (install `gh`, the daily loop) · 04 conflicts-and-undo (conflict on purpose, revert vs reset vs restore) · 05 protect-main (ruleset requiring PR, PR template with Explain section) · 06 history-and-tags (rebase vs merge, squash, semver tags) |
| 2 Linux & shell | 07 filesystem-and-permissions · 08 processes-and-ports (`ps`, `kill`, `ss`, `curl`) · 09 pipes-and-env (grep/sed/awk/jq, exit codes, PATH) · 10 bash-scripting (`set -euo pipefail`, shellcheck, `scripts/`) · 11 make-and-apt (install `make`, Makefile with `help`) |
| 3 App | 12 app-skeleton (FastAPI shortener via AI, venv, pinned deps, env config, `/health`; *AI thread: prompting with context*) · 13 postgres-and-migrations (Postgres via one `docker run`, SQLAlchemy, Alembic as a deploy step) · 14 tests-and-lint (pytest against a test DB, ruff, `make test` / `make lint` from a clean clone) |
| 4 Containers | 15 container-basics · 16 first-dockerfile (*AI thread: review an AI Dockerfile*) · 17 production-image (multi-stage, slim, non-root, HEALTHCHECK) · 18 compose (app + Postgres, volumes, healthcheck, `.env.example`, migrations on start) · 19 container-debugging (exit 137, networking, logs/exec drills) |
| 5 CI | 20 first-workflow (lint + test) · 21 ci-postgres-and-cache · 22 publish-to-ghcr (`GITHUB_TOKEN` perms, sha + semver tags, main only) · 23 pipeline-security (Trivy, Dependabot, push protection, actions pinned by SHA; *AI thread: secrets never go in a chat*) · 24 required-checks (ruleset requires CI, reading failed runs) |
| 6 Kubernetes | 25 k8s-and-kind · 26 deployment-and-service · 27 config-secrets-and-postgres (StatefulSet + PVC) · 28 probes-resources-rollouts · 29 ingress (a maintained controller) · 30 helm-chart · 31 ci-kind-smoke-test (throwaway cluster on every PR) |
| 7 Observability | 32 app-metrics (`/metrics`, RED) · 33 prometheus-grafana (kube-prometheus-stack on kind) · 34 dashboards-and-alerts (as code) · 35 structured-logs |
| 8 Cloud & IaC | 36 aws-account-safety (root MFA, Identity Center user, $5 budget, AWS CLI + `aws sso login`) · 37 terraform-basics (S3 bucket, plan/apply/destroy) · 38 remote-state (S3 backend, `use_lockfile`) · 39 network-and-ec2 (VPC/SG, EC2, user_data Docker, SSM, no SSH) · 40 modules-and-plan-review (*AI thread: never trust AI on state/IAM/deletion*) · 41 manual-ec2-deploy (via SSM, then destroy) |
| 9 CD | 42 github-oidc-to-aws · 43 deploy-job (environment `production` + required reviewer, SSM deploy, health check) · 44 rollback-and-releases |
| 10 Capstone | 45 end-to-end (diagram + README) · 46 incident-drill (runbook + postmortem) · 47 teardown-and-portfolio (interview story, resume bullets) |
| Optional | 48 ansible · 49 eks-vs-ec2-cost · 50 uv |

- [ ] **Step 2: Verify**

Run: `grep -cE '^- \[ \] [0-9]{2} ' lessons/README.md`
Expected: `51`

---

### Task 2: Stage 0 lessons (Setup)

**Files:**
- Create: `lessons/00-workspace.md`, `lessons/01-first-commit.md` (written to the Windows copy, because `~/devops` doesn't exist yet)

**Interfaces:**
- Consumes: the heading contract from Task 1.
- Produces: `~/devops` cloned over SSH, containing `docs/` and `lessons/` from the Windows copy, pushed to `main`. Every later task writes to `\\wsl.localhost\Ubuntu\home\klarens\devops`.

- [ ] **Step 1: Write `00-workspace.md`.** Content: why Linux, the WSL vs Windows filesystems, the VS Code WSL extension (`code .` from Ubuntu), `ssh-keygen -t ed25519`, adding the key to GitHub, `ssh -T git@github.com`. Git identity is already done, so the lesson just verifies it. Break: rename `~/.ssh/id_ed25519` and diagnose `Permission denied (publickey)`. Review: a sample `~/.gitconfig` with the user's real Gmail on a public repo. Commit: none (Commit section says "Nothing to commit today; Lesson 01 makes your first one").
- [ ] **Step 2: Write `01-first-commit.md`.** Content: `git clone git@github.com:rensyi/devops.git ~/devops`, copy `docs/` and `lessons/` from `/mnt/c/Users/Klarens/Documents/devops/devops/`, `git status`, `add`, `commit`, `push` straight to `main` (the only lesson that does), check the tile on the GitHub profile, and delete the Windows copy only after the push is verified. Break: commit with a throwaway email using `git -c user.email=x@example.com commit --allow-empty`, see that it's unlinked on GitHub, then fix it with `git commit --amend --reset-author` before pushing. Check: `pwd` → `/home/klarens/devops`, `git log -1 --format=%ae` → the noreply address.
- [ ] **Step 3: Verify commands.** Run every Install/Check command that doesn't touch the student's account in a scratch dir in WSL, for example `ssh-keygen -t ed25519 -f /tmp/k -N ''` and `git -c user.email=x@example.com commit --allow-empty` in a `git init` temp repo. Confirm the outputs match what the lesson says.
- [ ] **Step 4: Heading check**

Run: `for f in lessons/0[01]-*.md; do grep -c '^## \|^### ' "$f"; done`
Expected: `13` for each file.

---

### Task 3: Stage 1 lessons (Git & GitHub)

**Files:**
- Create: `lessons/02-git-basics.md` … `lessons/06-history-and-tags.md` (Windows copy. Lesson 01 moves them.)

**Interfaces:**
- Consumes: `~/devops` exists with `main` pushed (from Lesson 01).
- Produces: a `.gitignore` covering `.env`, `.venv/`, `__pycache__/`, `*.tfstate*`, `.terraform/`, and the daily loop every later lesson's Commit section uses: `git switch -c NN-slug` → commit → `git push -u origin HEAD` → `gh pr create --fill` → `gh pr merge --squash --delete-branch` → `git switch main && git pull`. Also `.github/pull_request_template.md` with an "Explain" heading that later Gate sections fill in.

- [ ] **Step 1: Write lessons 02–06** per the roadmap rows. Lesson 03 installs `gh` from GitHub's official apt repo and runs `gh auth login`. Lesson 05 creates a repository *ruleset* on `main` (require PR, block force push). Required status checks come later in Lesson 24. Each lesson's Break and Review is Git-specific, for example: 02 Review a `.gitignore` missing `.env`; 04 Break with a `reset --hard` that loses work, then recover it via `reflog`.
- [ ] **Step 2: Verify commands** in a scratch `git init` repo in WSL, including producing and resolving the Lesson 04 conflict and the `reflog` recovery. For `gh`, verify the install commands and `gh --version` only, without authenticating as the student.
- [ ] **Step 3: Heading check**: same command as in Task 2, over `lessons/0[2-6]-*.md`. Expected: `13` each.
- [ ] **Step 4: Hand over.** Tell the student which files are new. They commit them in Lesson 01 or with that day's lesson.

---

### Tasks 4–12: Stages 2–10 (one stage ahead)

**Trigger:** write Stage N+1's lessons when the student starts Stage N. Before writing, read the student's actual repo (`\\wsl.localhost\Ubuntu\home\klarens\devops`) so lessons reference their real file names, Make targets, and image names.

Each task has the same steps as Task 3: **write the lessons** per their roadmap rows → **verify commands** in WSL, in a scratch dir or a throwaway kind cluster, never the student's AWS account → **heading check** (`13` each) → **hand over**, uncommitted.

| Task | Stage | Lessons | Stage-specific decisions |
|---|---|---|---|
| 4 | Linux & shell | 07–11 | `scripts/healthcheck.sh` from Lesson 10 is reused in Lessons 19 and 43. Install `shellcheck` and `jq` in the lessons that first use them. |
| 5 | App | 12–14 | Make targets `run`, `test`, `lint`, `migrate`, and env vars `DATABASE_URL` and `BASE_URL` are fixed here and assumed by every later stage. |
| 6 | Containers | 15–19 | Image name `ghcr.io/rensyi/devops`, the app listens on port `8000`, and migrations run as a separate compose service, not inside the app process. |
| 7 | CI | 20–24 | Workflow `.github/workflows/ci.yml` with job names `lint`, `test`, `build`, `scan`. Lesson 24's ruleset requires those exact names. |
| 8 | Kubernetes | 25–31 | Chart at `charts/devops/`. Lesson 31 adds a `smoke` job to `ci.yml`. Choose a maintained ingress or Gateway controller at writing time. |
| 9 | Observability | 32–35 | Dashboards and alert rules live in the chart, as code. |
| 10 | Cloud & IaC | 36–41 | Terraform in `infra/`. Lesson 36 is a hard gate before 37. Free-tier-eligible instance type. Public IPv4 only (no NAT gateway). |
| 11 | CD | 42–44 | Workflow `.github/workflows/deploy.yml`. The OIDC role is scoped to `repo:rensyi/devops:environment:production`. Prod exists only during a sitting: `terraform apply` → merge → deploy job → verify → `terraform destroy`. The OIDC role and state bucket are long-lived (free), and the EC2 stack is not. |
| 12 | Capstone | 45–47 | 46's incident is something the Grafana alert from Lesson 34 actually fires on. |

**Optional (48–50):** write only when the student asks.
