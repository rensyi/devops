# 03 · Branches & pull requests

## Goal

You have the GitHub CLI installed and logged in, and you've shipped one change through the full loop: branch → commit → push → pull request → squash merge. You'll repeat that loop every day from now on.

## Why this matters at work

On a team, nobody pushes to `main`. Every change arrives as a pull request so that CI can test it, a teammate can review it, and there's a record of *why* it happened. That record is what you'll search six months later when something breaks. Doing this alone feels like ceremony, but it's the same muscle memory you'll need on day one of a job, and it's where CI (Stage 5) and deploy approvals (Stage 9) plug in later.

## Concepts

**A branch is just a movable label pointing at a commit.** It's a 41-byte file. Creating one is instant and free. `HEAD` is the label for "where you are now".

**Pull request (PR):** a request to merge your branch into `main`. On GitHub it's also where discussion, CI results, and review happen.

**Merge strategies** (Lesson 06 goes deeper):

| Strategy | What lands on `main` |
|---|---|
| Merge commit | All your commits, plus a merge commit |
| **Squash** | One commit containing all your changes. **This repo's default:** one lesson = one commit on `main`. |
| Rebase | Your commits replayed on top, with no merge commit |

**Upstream / tracking branch:** the link between your local branch and its copy on GitHub. You set it once with `git push -u`, and after that a bare `git push` and `git pull` know where to go.

**`gh`** is GitHub's official CLI. It's PRs, issues, and Actions from your terminal, and you'll use it heavily in CI lessons.

Docs: [About pull requests](https://docs.github.com/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests) · [gh manual](https://cli.github.com/manual/)

## Install

**GitHub CLI**, from GitHub's own apt repository. The version in Ubuntu's default repos lags behind. Follow the **Debian/Ubuntu** section of the [official install guide](https://github.com/cli/cli/blob/trunk/docs/install_linux.md#debian).

Before you run it, read what it does. Each line is one of four things:
1. Making sure `wget` exists.
2. Downloading GitHub's **signing key** into `/etc/apt/keyrings`, so apt can verify that packages really come from GitHub.
3. Adding GitHub's repository to `/etc/apt/sources.list.d/`.
4. Running `apt update` and installing `gh`.

You'll see this pattern again for Terraform, Helm, and others. It needs `sudo`, so it will ask for your Ubuntu password.

Check it:

```bash
gh --version
```

## Steps

1. **Log in:** run `gh auth login` and choose **GitHub.com** → **SSH** → **skip** uploading a key (GitHub already has yours from Lesson 00) → **Login with a web browser**. If the browser doesn't open from WSL, copy the URL it prints into your Windows browser. Confirm with `gh auth status`.

2. **Start from a fresh `main`:** `git switch main && git pull`.

3. **Branch:** `git switch -c 03-branches-and-prs`. Run `git branch` and see the `*` move.

4. **Make the change:** in `lessons/README.md`, tick the boxes for lessons 00–03 (`- [ ]` → `- [x]`). This is your progress tracker from now on.

5. **Commit, then push with upstream:** `git push -u origin HEAD`. Read the output: GitHub even prints a link to open a PR.

6. **Open the PR:** `gh pr create --web` opens the PR form in your browser. Write a real title and a description that says what changed and why. Put your **Explain** answers (below) in the description. Create it.

7. **Read your own PR like a reviewer.** Open the **Files changed** tab. Every PR you open from now on gets this one-minute look before you merge.

8. **Squash-merge from the terminal:** `gh pr merge --squash --delete-branch`. It merges on GitHub, deletes the branch on GitHub and locally, and switches you back to `main`. Then `git pull` and `git log --oneline -3`: one new commit, titled after your PR.

## Check

| Run | You should see |
|---|---|
| `gh auth status` | `Logged in to github.com account rensyi` |
| `gh pr list --state merged` | your PR |
| `git branch` | only `* main` |
| `git log -1 --format='%an %s'` | `rensyi` and your PR title |
| Your GitHub profile | today's tile is green |

## Gate before merge

### Explain

Write these in the PR description in step 6.

1. A branch is "just a label pointing at a commit". Prove it: run `git rev-parse HEAD` and `cat .git/refs/heads/03-branches-and-prs` on your branch, and explain the output.
2. Why bother with a PR when you're the only person on the repo?
3. Your branch had N commits. How many land on `main` with a squash merge, and what happens to the originals?

### Break

On a new practice branch, push **without** `-u`:

```bash
git switch -c break-03
git commit --allow-empty -m "chore: test push"
git push
```

Read the error from top to bottom. What is Git missing, and what two fixes does it offer? Then clean up without pushing: `git switch main` and `git branch -D break-03`. Why does `-D` need to be uppercase here?

### Review

A new hire's end-of-day routine:

```bash
git switch main
# ...edit a few files...
git add -A
git commit -m "update"
git push
```

It works every time. List what's wrong with it, starting with the biggest problem. (Ask Claude for the answer once you've decided.)

## Commit

This is the daily loop, and it's the same every lesson from here:

```bash
git switch main && git pull
git switch -c 03-branches-and-prs
git add -A && git commit -m "docs: track progress for lessons 00-03"
git push -u origin HEAD
gh pr create --web
gh pr merge --squash --delete-branch
```

## Stretch

- Make a better log view: `git config --global alias.lg "log --oneline --graph --decorate --all"`, then run `git lg`.
- Explore `gh pr view`, `gh pr diff`, and `gh repo view --web`. Anything you can click on GitHub, `gh` can do from the terminal, which makes it scriptable.

## If you're stuck

| Symptom | Likely cause |
|---|---|
| `gh: command not found` after install | The apt step failed. Scroll up for the first red line. `apt update` errors usually mean a typo in the sources line. |
| `gh auth login` browser step hangs | WSL can't open your Windows browser. Copy the URL and one-time code it prints and open the URL manually. |
| `gh pr create` says `no commits between main and ...` | You forgot to commit, or committed on `main`. Check `git log --oneline -3` and `git branch`. |
| `gh pr merge` says the PR isn't mergeable | `main` changed since you branched. Run `git pull origin main` on your branch, push, and try again. |
| Tile not green after a merge | Check the squashed commit's author on GitHub. If it's not linked to your profile, check GitHub → Settings → Emails. |
