# 05 · Protect main

## Goal

`main` can only change through a pull request: direct pushes, force pushes, and deletion are blocked by a GitHub ruleset. Every new PR opens with a template that has a place for your Explain / Break / Review answers.

## Why this matters at work

`main` is what gets deployed. If anyone can push to it directly, then testing, review, and approvals are all optional, and eventually someone skips them on a Friday evening. Protection rules turn "please don't" into "you can't". In Stage 5 you'll add "CI must pass" to the same ruleset, and that's the backbone of every CI/CD setup you'll ever work on.

## Concepts

**Rulesets** are GitHub's current way to protect branches and tags. Older repos use "branch protection rules", which are similar but less flexible. You'll see both at work. A ruleset has:

- **Enforcement:** `Active`, or `Disabled`.
- **Targets:** which branches it applies to, e.g. the default branch.
- **Rules:** what's required or blocked.
- **Bypass list:** who is exempt. Anyone on it isn't protected *from themselves*.

**Solo-repo detail:** GitHub never lets you approve your own PR. If you require 1 approval on a repo with one person, you can never merge. So for this repo: require a PR, with **0** required approvals. The value is that every change has a reviewed diff and, later, green CI, not the second pair of eyes.

**PR templates:** `.github/pull_request_template.md` pre-fills every new PR's description. GitHub reads it from the **default branch**, so it starts working for the PR *after* the one that adds it.

Docs: [About rulesets](https://docs.github.com/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets) · [PR templates](https://docs.github.com/communities/using-templates-to-encourage-useful-issues-and-pull-requests/creating-a-pull-request-template-for-your-repository)

## Install

Nothing new today.

## Steps

1. **Branch** `05-protect-main` from a fresh `main`.

2. **Write `.github/pull_request_template.md`.** It needs these sections, and you decide the wording and the HTML-comment hints inside each:
   - **What changed**
   - **Why**
   - **Explain:** your answers to the lesson's Explain questions
   - **Break:** what you broke, the key error line, and how you diagnosed it
   - **Review:** your answer to the Review question
   - **Checklist:** e.g. "I read the Files changed tab", "Lesson ticked in `lessons/README.md`"

3. **Ship it through the loop** (see Commit). Notice that *this* PR's form doesn't use the template yet. Can you explain why from the Concepts?

4. **Create the ruleset** after merging. On GitHub: **Settings → Rules → Rulesets → New ruleset → New branch ruleset**:
   - Name: `protect-main`. Enforcement: **Active**.
   - Bypass list: **leave empty**. (The Review question below explains why.)
   - Target branches: **Include default branch**.
   - Rules: **Restrict deletions**; **Require a pull request before merging** with required approvals **0**; **Block force pushes**.
   - If you see **Allowed merge methods**, allow only **Squash**, to match the daily loop.

5. **Confirm it from the terminal.** Use `gh api repos/rensyi/devops/rulesets` to list rulesets. Then add `--jq` to print just each ruleset's name and enforcement. You're reading GitHub's REST API directly, and `gh api` is how you'll automate GitHub later.

## Check

| Run | You should see |
|---|---|
| `gh api repos/rensyi/devops/rulesets --jq '.[] \| "\(.name) \(.enforcement)"'` | `protect-main active` |
| `git ls-files .github` | `.github/pull_request_template.md` |
| Lesson 06's `gh pr create --web` (next time) | the form pre-filled with your template |
| The Break below | a rejected direct push |

## Gate before merge

This PR merges *before* the ruleset exists, so write the Explain answers in the PR now. Do the Break after step 4.

### Explain

1. Why is required approvals set to 0 here, and what would be different on a team?
2. What does the bypass list do, and why leave yourself off it?
3. Why does the template not apply to the PR that adds it?

### Break

After the ruleset is active, try to sneak past it:

```bash
git switch main && git pull
git commit --allow-empty -m "chore: direct push test"
git push
```

Read the rejection carefully: find the error code and the rules it lists. Explain which rule fired. Then remove the local commit so `main` matches GitHub again: `git reset --hard origin/main`. Why is a hard reset fine here when Lesson 04 warned against it?

### Review

A teammate set up their repo's ruleset like this and says `main` is "fully protected":

```
Ruleset: protect-main        Enforcement: Active
Target:  Default branch
Bypass list: Repository admin
Rules:
  [x] Require a pull request before merging (required approvals: 1)
  [x] Block force pushes
  [x] Restrict deletions
```

They're the only admin, and they push most of the commits. What's the real level of protection? (Ask Claude for the answer once you've decided.)

## Commit

```bash
git switch main && git pull
git switch -c 05-protect-main
# write .github/pull_request_template.md, tick lesson 05
git add -A && git commit -m "chore: add pull request template"
git push -u origin HEAD
gh pr create --web
gh pr merge --squash --delete-branch
# now step 4 (ruleset), then the Break
```

## Stretch

- Save the ruleset as code: `gh api repos/rensyi/devops/rulesets/<id> > docs/ruleset-protect-main.json` (find the id from step 5). Settings that live only in a web UI are hard to review or recreate. Version controlling them is the start of "everything as code".
- Add a second ruleset that blocks deleting tags matching `v*`. You'll start tagging releases in Lesson 06.

## If you're stuck

| Symptom | Likely cause |
|---|---|
| No **Rulesets** menu | Check the URL is `github.com/rensyi/devops/settings/rules`. Rulesets work on public repos on the free plan. |
| `gh pr merge` fails with `Pull request ... is not mergeable` / review required | Required approvals is above 0. Edit the ruleset and set it to 0. |
| The direct push in the Break **succeeded** | Enforcement isn't Active, the target isn't the default branch, or you're on the bypass list. Check all three, and use `git revert` on that commit via a PR. |
| `gh api` returns `404` | Typo in `rensyi/devops`, or `gh` is logged in to a different account (`gh auth status`). |
