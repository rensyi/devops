# DevOps Learning Path — Design

Date: 2026-10-16
Status: reviewed, awaiting student approval

## Intent

The student is a 4th-year CS student. They plan to start as a software developer and grow into a DevOps engineer. They want to learn, from the beginning, the workflow that takes a developer's code to production safely, and to automate the repetitive work around it so the people running the system don't have to do it by hand.

The repo also exists to keep their GitHub contribution graph active. That only works if every commit is real learning. Each day's work should produce something they wrote, understand, and can explain in an interview.

They have used Python, Git, and Docker before, but always with an AI assistant doing much of the work. They want to learn DevOps from the beginning. Python itself is assumed knowledge.

**AI-era stance (agreed with the student):** AI can write most of the code, and the student may use it the way a working engineer would. What makes them employable is what AI doesn't replace: debugging a failure from its error output, catching what's dangerous in AI-generated config, reasoning about systems, cost, and risk, and passing live troubleshooting interviews without AI. The bar is: **understand and be able to debug every line you merge, whoever wrote it.**

## Goals

1. By the end, the student can explain and demonstrate, without help, how a commit travels from their laptop to a running production deployment on AWS. That covers tests, container build, security scan, deploy, monitoring, and rollback.
2. A steady learning curve: each lesson builds on the last and adds only one or two new ideas.
3. One finishable lesson per sitting (1–2 hours a day), ending in a commit the student makes themselves.
4. They work in a Linux environment (WSL2 Ubuntu) the way a real DevOps engineer does.
5. Habits from real workplaces are built in from day one: branches, pull requests, CI checks, least-privilege access, no secrets in Git, tearing down cloud resources.
6. Interview-ready debugging and review skills. The student can diagnose a broken pipeline, container, cluster, or Terraform run from its error output, and can spot the dangerous line in a config they didn't write.

## Non-goals

- A feature-rich app. The app is deliberately boring. It is the thing being shipped, not the thing being learned.
- Covering every DevOps tool. Breadth comes later. Depth on one coherent stack comes first.
- Certification prep.
- Managed Kubernetes on AWS (EKS). Its control plane alone costs about $70/month. One lesson explains this tradeoff instead.

## Anti-goals (failure even if it "works")

- **Claude commits, pushes, or appears as a co-author or contributor.** Never. No `Co-Authored-By` lines, no commits, no pushes, no PRs opened by Claude. Claude writes lesson files into the working tree, and the student reviews and commits everything.
- **Filler commits** made only to turn a tile green.
- **Merging what you can't explain.** AI-generated code is fine. A lesson fails if the student can finish it without being able to explain and debug what they merged. The Explain / Break / Review gate (see Lesson format) is what enforces this.
- **Python tutorials.** Teaching Python syntax or web-framework basics wastes lessons the DevOps stages need.
- **Surprise cloud bills.** No AWS resource is created before the cost guardrails exist.
- **Hardcoded secrets** anywhere in the repo, even "temporarily".

## Constraints

- **Machine:** Windows 11, 16 GB RAM, about 145 GB free. WSL2 Ubuntu 26.04 is installed. Docker Desktop 29.7 is installed and reachable from Ubuntu.
- **Already installed:** Git (Windows and Ubuntu), Python 3.14 (Ubuntu), Docker, kubectl (via Docker Desktop), VS Code, curl, Node.
- **Missing (install in the lesson that first needs it, never all at once):** `python3-pip`, `python3-venv`, `make`, `gh`, `kind`, `helm`, `terraform`, AWS CLI, and optionally `ansible`. Installing and verifying each tool is part of its lesson.
- **Git identity:** Ubuntu has no `user.name` or `user.email` set. Windows Git uses `c.redoma26@gmail.com`, and the initial commit used `122665114+rensyi@users.noreply.github.com`. A commit only counts on the GitHub graph if its email is verified on the account. Lesson 1 must set Ubuntu's identity to an email the student confirms is verified (the noreply address always works) and verify that a test commit shows up on the graph.
- **Contribution graph rules:** commits count when they land on the default branch (`main`). Work done on a feature branch counts once its PR is merged. Every lesson therefore ends with branch → commit → push → PR → merge.
- **Time:** 1–2 hours a day. A lesson's core fits in about 60–90 minutes, and optional "stretch" items can fill the rest.
- **Budget:** $0 until the AWS stage. On AWS, aim for $0 and accept at most about $5/month, enforced by a budget alert.
- **Repo:** `github.com/rensyi/devops`, branch `main`. The student owns all commits. It is **public** (confirmed 2026-10-16). That enables branch protection, required checks, and environment approval gates on the free plan, and the repo doubles as a portfolio piece. Because it's public, secret hygiene is taught before the first secret ever exists.

## Decided how

### Student's decisions
- Start DevOps from the beginning. Python is assumed.
- AI tools are allowed for writing code and config, as at a real job. Every lesson ends with an Explain / Break / Review gate before merge.
- Claude acts as a guide: lessons give the goal, concepts, hints, and checks. Lessons don't contain finished DevOps config. The student produces it, with or without AI, and Claude gives a full reference solution in chat when asked.
- Make the repo public.
- Include a real cloud deployment near the end.
- Lesson and design files are written into the repo but left uncommitted. The student commits them.

### Claude's decisions (marked so the student can override)
- **Cloud:** AWS, because it appears in the most job postings and the concepts carry over to other clouds.
- **App:** a small Python web API (FastAPI) backed by PostgreSQL. It is a link shortener, chosen because it has an obvious database table, simple endpoints, and real data you don't want to lose across deploys.
- **Stack, in the order it's introduced:** Git + GitHub → Linux shell + Make → Python, venv, pytest, ruff → Docker + Docker Compose → GitHub Actions + GHCR + Trivy + Dependabot → kind + kubectl + Helm → Prometheus + Grafana → Terraform + AWS → (optional) Ansible.
- **AWS deployment shape:** Terraform provisions one small free-tier-eligible EC2 instance (plus its network and security group). The instance runs the app and Postgres with Docker Compose. GitHub Actions deploys to it by assuming an AWS role through OIDC (no long-lived access keys), using SSM rather than open SSH. This is cheap, realistic, and fully destroyable.
- **Kubernetes stays local:** on kind, where it is free. It acts as a "staging" environment, and AWS is "production". GitHub's runners can't reach the laptop, so staging works in two parts. CI spins up a throwaway kind cluster inside the job (`helm install` + smoke test) on every PR. The student deploys to their laptop's kind cluster by hand with `helm upgrade`. **No self-hosted runner on the laptop:** on a public repo, a stranger's PR could run code on the student's machine.
- **Production is ephemeral by default.** Every AWS sitting is apply → deploy → verify → destroy. That keeps the $5 cap safe whether or not the account still has free-tier credits (new accounts get a time-limited credit plan, not the old 12-month free tier). The capstone is the one session where prod stays up long enough for the incident drill, and it still ends with a destroy. The account-hardening lesson works for both new and existing accounts.
- **Working copy lives inside WSL.** Lesson 0 has the student clone the repo to `~/devops` in Ubuntu and work only there. Working on `/mnt/c` breaks `chmod`, slows Docker bind mounts, and invites line-ending bugs. From then on, Claude writes lesson files into that clone (`\\wsl.localhost\Ubuntu\home\<user>\devops`). This Windows copy is retired once the student moves the design doc over.
- **Solutions are not stored in the repo.** That removes the temptation to look and keeps the student's code the only code.
- **Lessons are written one stage ahead.** The full roadmap (every lesson's title and one-line goal) is written up front. Full lesson content is written for the current stage and the next. Later lessons must reference the student's actual code, which won't exist until they write it.

## The curriculum

About 45 lessons over roughly 9–11 weeks at one lesson per day. Stage boundaries are fixed. The exact lesson split inside a stage is left to the builder.

| # | Stage | What the student can do at the end |
|---|---|---|
| 0 | **Setup** | Work in WSL Ubuntu from VS Code. Git identity set and verified on the graph. SSH key on GitHub. |
| 1 | **Git & GitHub workflow** | Branch, commit well, open and merge PRs, resolve a merge conflict, use `.gitignore`, protect `main`. |
| 2 | **Linux & shell** | Navigate the filesystem, understand permissions and processes, use environment variables and pipes, write a small bash script and a Makefile. |
| 3 | **The app** (about 3 lessons, AI-generated code encouraged) | A FastAPI link shortener backed by Postgres, using SQLAlchemy and Alembic migrations, configured from environment variables, with a `/health` endpoint, pytest tests, and ruff linting, all run via `make`. The lessons cover only what's DevOps-relevant about the app: 12-factor config, health endpoints, migrations as a deploy step, and making tests runnable in CI. |
| 4 | **Containers** | Write a multi-stage Dockerfile with a non-root user and `.dockerignore`. Run app + Postgres in Docker Compose with volumes, healthchecks, and an `.env` file kept out of Git. |
| 5 | **Continuous Integration** | A GitHub Actions pipeline on every PR: lint → test (with a Postgres service) → build image → Trivy scan → push to GHCR on merge. Dependabot enabled. Branch protection requires CI to pass. |
| 6 | **Kubernetes (local)** | Run the app on kind with a Deployment, Service, ConfigMap, Secret, probes, and Ingress, then package it as a Helm chart. Do a rolling update and a rollback. |
| 7 | **Observability** | Expose a `/metrics` endpoint. Install Prometheus + Grafana on kind, build a dashboard, write an alert rule, use structured logs. |
| 8 | **Cloud & IaC (AWS)** | Secure the AWS account first (root MFA, IAM Identity Center user, $5 budget alert). Learn Terraform basics, keep remote state in S3, provision the EC2 environment, and practice `terraform destroy` as a daily habit. |
| 9 | **Continuous Delivery** | A GitHub Actions deploy job: OIDC → AWS role → SSM deploy of the new image tag. A manual approval gate for production. A rollback drill. |
| 10 | **Capstone** | Walk the full path end to end: PR → CI → staging (kind) → approval → prod (AWS). Write a runbook and an architecture diagram in the README. Do a mock "incident" (break prod, detect it on the dashboard, roll back). Final teardown. |
| — | *Running thread: AI as a DevOps tool* | Woven into existing lessons, not a separate stage: prompting for infra code with the right context, reviewing AI output for security and cost traps, never pasting secrets or real config into a chat, and when not to trust it (anything touching state, IAM, or deletion). |
| — | *Optional* | Ansible to configure the EC2 host. A one-lesson EKS-vs-EC2 cost discussion. Swap pip for uv. |

## Lesson format

Each lesson is one Markdown file, `lessons/NN-short-title.md`, containing:

1. **Goal:** one sentence about what exists or works at the end.
2. **Why this matters at work:** a short, concrete workplace scenario.
3. **Concepts:** only what's needed for this lesson, explained plainly, with links to official docs.
4. **Install** (only when this lesson introduces a tool), followed by a version check.
5. **Steps:** what to do, described as tasks and hints, not code to paste. Small syntax examples are fine. Full working solutions are not.
6. **Check:** commands to run, plus what correct output looks like.
7. **Gate before merge:**
   - **Explain:** questions the student answers in the PR description in their own words (what each part does, and why).
   - **Break:** one deliberate fault to introduce, then diagnose from the error output alone and fix. The lesson names the fault, not the diagnosis.
   - **Review:** a short sample config with one planted mistake to find. This trains reviewing AI output. The answer comes from Claude in chat, not from the file.
8. **Commit:** the branch name, a suggested commit message, the PR, and the merge.
9. **Stretch** (optional): one or two extras for days with more time.
10. **If you're stuck:** common errors and what they mean.

`lessons/README.md` holds the roadmap as a checklist the student ticks off.

## Left to the builder

- The exact lesson split and count within each stage, as long as each lesson fits in 1–2 hours and adds no more than one or two new ideas.
- The repo layout for the student's code (`app/`, `tests/`, `infra/`, `charts/`, etc.), introduced as the lessons need it.
- Specific package and action versions (use current stable versions and pin them).
- The wording, examples, and workplace scenarios in each lesson.
