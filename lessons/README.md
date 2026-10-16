# DevOps Learning Path

One lesson per sitting, 1–2 hours each. By the end, a commit you push travels by itself through tests, a container build, a security scan, a staging deploy, an approval, and a production deploy on AWS, and you can explain and debug every step of it.

The full reasoning behind this path is in [`docs/superpowers/specs/2026-10-16-devops-learning-path-design.md`](../docs/superpowers/specs/2026-10-16-devops-learning-path-design.md).

## How to use these lessons

**One lesson, one sitting, one merge.** Each lesson ends with something working and a merge into `main`. That merge is your green tile, and it's earned.

**Work in Ubuntu, in `~/devops`.** Not in a Windows folder. Lesson 00 explains why.

**AI is allowed, like at a real job.** Use it to write code and config. Before you merge, though, every lesson makes you pass a three-part gate:

| Gate | What you do | Why |
|---|---|---|
| **Explain** | Answer the lesson's questions in your PR description, in your own words. | If you can't explain it, you can't defend it in a review or an interview. |
| **Break** | Introduce the fault the lesson names, then diagnose it from the error output alone, and fix it. | Production fails in ways nobody planned. Reading errors calmly is the job. |
| **Review** | Find the planted mistake in the sample config. | This is the skill AI makes *more* valuable: catching what's wrong in code you didn't write. |

**Stuck?** Each lesson ends with common errors. Still stuck, or want to check your Review answer? Ask Claude in chat. Reference solutions are never stored in this repo, so the only code here is code you merged.

**The daily loop** (from Lesson 03 on; Lesson 03 teaches it):

```bash
git switch main && git pull
git switch -c NN-short-title
# ...do the lesson...
git add -A && git commit -m "..."
git push -u origin HEAD
gh pr create --web          # opens the PR form with your template: write your Explain answers here
gh pr merge --squash --delete-branch
```

Tick the box below when the lesson is merged.

## Roadmap

### Stage 0 · Setup
- [ ] 00 · Workspace: Ubuntu, VS Code connected to it, and an SSH key GitHub trusts.
- [ ] 01 · First commit: this repo cloned into `~/devops`, your first push, and a green tile you can see.

### Stage 1 · Git & GitHub
- [ ] 02 · Git basics: what working tree, staging area, and commit really are; good messages; a `.gitignore` that protects secrets.
- [ ] 03 · Branches & pull requests: install `gh` and run the branch → PR → merge loop you'll use every day.
- [ ] 04 · Conflicts & undo: cause a merge conflict on purpose, resolve it, and know when to use revert, reset, and restore.
- [ ] 05 · Protect main: a ruleset so nothing reaches `main` without a PR, plus a PR template.
- [ ] 06 · History & tags: rebase vs merge, squash merges, and semantic version tags.

### Stage 2 · Linux & shell
- [ ] 07 · Filesystem & permissions: find your way around Linux and understand `rwx`, owners, and `chmod`.
- [ ] 08 · Processes & ports: see what's running, stop it, and find what's listening on a port.
- [ ] 09 · Pipes & environment: chain `grep`, `sed`, `awk`, `jq`; exit codes; `PATH` and environment variables.
- [ ] 10 · Bash scripting: a safe script (`set -euo pipefail`) that checks if a URL is healthy, linted with shellcheck.
- [ ] 11 · Make & packages: `apt` and a Makefile that becomes the front door to the project.

### Stage 3 · The app
- [ ] 12 · App skeleton: a FastAPI link shortener built with AI help, configured from environment variables, with a `/health` endpoint.
- [ ] 13 · Postgres & migrations: a real database and migrations treated as a deploy step.
- [ ] 14 · Tests & lint: `make test` and `make lint` pass from a fresh clone.

### Stage 4 · Containers
- [ ] 15 · Container basics: images, containers, layers; run, exec, logs, inspect.
- [ ] 16 · First Dockerfile: your app in an image, and a review of an AI-written Dockerfile.
- [ ] 17 · Production image: multi-stage, slim, non-root, with a healthcheck.
- [ ] 18 · Compose: app + Postgres with one command, data that survives restarts, secrets kept out of Git.
- [ ] 19 · Container debugging: diagnose crashed, killed, and unreachable containers.

### Stage 5 · Continuous Integration
- [ ] 20 · First workflow: GitHub Actions runs lint and tests on every PR.
- [ ] 21 · CI with Postgres & caching: real database in CI, faster runs.
- [ ] 22 · Publish to GHCR: build and push a tagged image on every merge to `main`.
- [ ] 23 · Pipeline security: image scanning, Dependabot, push protection, pinned actions.
- [ ] 24 · Required checks: nothing merges unless CI is green; read a failed run like a pro.

### Stage 6 · Kubernetes (local)
- [ ] 25 · Kubernetes & kind: a real cluster on your laptop and your first `kubectl` session.
- [ ] 26 · Deployment & Service: your app running as replicas that heal themselves.
- [ ] 27 · Config, secrets & Postgres: ConfigMaps, Secrets, and a database with persistent storage.
- [ ] 28 · Probes, resources & rollouts: health probes, CPU and memory limits, rolling update and rollback.
- [ ] 29 · Ingress: reach the app by hostname through a maintained ingress controller.
- [ ] 30 · Helm chart: package everything so a deploy is one command.
- [ ] 31 · CI smoke test: every PR spins up a throwaway cluster and proves the chart deploys.

### Stage 7 · Observability
- [ ] 32 · App metrics: a `/metrics` endpoint with rate, errors, and duration.
- [ ] 33 · Prometheus & Grafana: a monitoring stack scraping your app.
- [ ] 34 · Dashboards & alerts: a dashboard and an alert rule, both as code.
- [ ] 35 · Structured logs: JSON logs you can actually search.

### Stage 8 · Cloud & Infrastructure as Code
- [ ] 36 · AWS account safety: MFA, a non-root user, a $5 budget alert, and the AWS CLI. **Nothing in AWS before this.**
- [ ] 37 · Terraform basics: plan, apply, destroy a harmless S3 bucket.
- [ ] 38 · Remote state: Terraform state stored safely in S3 with locking.
- [ ] 39 · Network & EC2: a server with Docker, reachable via SSM, no SSH port open.
- [ ] 40 · Modules & plan review: reusable Terraform, and spotting the plan that would destroy something.
- [ ] 41 · Manual EC2 deploy: run your image on AWS by hand, then tear it all down.

### Stage 9 · Continuous Delivery
- [ ] 42 · GitHub OIDC to AWS: CI gets AWS access with no stored keys.
- [ ] 43 · Deploy job: merges deploy to production after your approval.
- [ ] 44 · Rollback & releases: deploy any version, roll back under pressure.

### Stage 10 · Capstone
- [ ] 45 · End to end: the full path from PR to production, with an architecture diagram and README.
- [ ] 46 · Incident drill: break production, get alerted, roll back, write the postmortem.
- [ ] 47 · Teardown & portfolio: clean up, then turn this repo into an interview story.

### Optional
- [ ] 48 · Ansible: configure the EC2 server with Ansible instead of a startup script.
- [ ] 49 · EKS vs EC2: what managed Kubernetes would cost and when it's worth it.
- [ ] 50 · uv: swap pip for uv and measure the difference.
