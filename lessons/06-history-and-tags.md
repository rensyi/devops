# 06 · History & tags

## Goal

You can choose between merge, rebase, and squash on purpose, keep a branch up to date by rebasing, and the repo has its first release: `v0.1.0`.

## Why this matters at work

Readable history is a debugging tool. `git log` and `git bisect` are how you find the commit that broke production, and that's much easier on a clean history than a tangle of "merge main into feature" commits. Tags matter even more: in Stage 9, every production deploy will be a specific version, and "roll back to v1.3.2" only works if v1.3.2 means something.

## Concepts

**Three ways to combine branches.** Picture `main` gaining commits while your branch works:

| | What `main` looks like after | Good for |
|---|---|---|
| **Merge** | Both histories kept, joined by a merge commit | Preserving exactly what happened |
| **Rebase** | Your commits *replayed* on top of the latest `main`: linear, with new commit hashes | Keeping your branch current before a PR |
| **Squash** | Your whole branch becomes one commit | One PR = one commit on `main` (this repo) |

**The rule of rebasing:** rebase only commits *nobody else has*. Rebasing creates new commits, so after rebasing a branch you already pushed, you must force-push. Use `git push --force-with-lease`, which refuses if someone else pushed in the meantime. Plain `--force` silently overwrites them.

**Semantic Versioning:** `MAJOR.MINOR.PATCH`

| Bump | When | Example |
|---|---|---|
| PATCH | Bug fix, nothing changes for users | `1.4.2 → 1.4.3` |
| MINOR | New feature, nothing breaks | `1.4.2 → 1.5.0` |
| MAJOR | Something that worked before breaks | `1.4.2 → 2.0.0` |

Before `1.0.0`, anything may change. Starting at `0.1.0` says "early, unstable".

**Tags** label a commit forever. **Annotated** tags (`git tag -a`) store who, when, and a message; use those for releases. A GitHub **Release** is a tag plus notes plus optional files.

Docs: [git rebase](https://git-scm.com/book/en/v2/Git-Branching-Rebasing) · [SemVer](https://semver.org/) · [git tag](https://git-scm.com/book/en/v2/Git-Basics-Tagging) · [gh release](https://cli.github.com/manual/gh_release_create)

## Install

Nothing new today.

## Steps

**Part A: the lab** (`~/sandbox/git-lab` from Lesson 04).

1. **Make two diverged branches.** From `main`, create `feature` and make two commits there. Switch to `main` and make two *different* commits touching *different* files.

2. **Merge it.** Make a copy of the situation first (`git branch feature-copy feature`). Merge `feature` into `main` and look at `git log --oneline --graph --all`.

3. **Rebase it.** Undo the merge (`git reset --hard HEAD~1`; it's the lab, nothing is shared). Switch to `feature-copy` and run `git rebase main`. Look at the graph again. Compare the hashes of the rebased commits to the originals: they're new commits.

4. **Fast-forward.** On `main`, run `git merge feature-copy`. The message says *fast-forward*, so no merge commit was needed. Why?

**Part B: in `devops`**, on branch `06-history-and-tags`.

5. Create `notes/git-history.md` in your own words: merge vs rebase vs squash, when you'd use each, and what `--force-with-lease` protects against.

6. Ship it through the daily loop. Your PR should now open with your template from Lesson 05.

7. **Cut the release** after merging, on an up-to-date `main`: create an **annotated** tag `v0.1.0` with a message like "Stage 0–1 complete: repo, workflow, and protections", and push the tag. Tags don't go up with a normal `git push`, so look up how to push one tag. Then create a GitHub release from it with `gh release create`, letting GitHub generate the notes from your merged PRs.

## Check

| Run | You should see |
|---|---|
| (lab) `git log --oneline --graph feature-copy` | a straight line: your rebased commits sitting on top of `main`'s |
| (devops) `git describe --tags` | `v0.1.0` |
| (devops) `git show v0.1.0 --no-patch` | `tag v0.1.0`, `Tagger: rensyi`, and your message |
| `gh release view v0.1.0` | the release, with notes listing your PRs |

## Gate before merge

### Explain

1. When would you rebase, and when would you merge? Give one situation for each.
2. What exactly does `--force-with-lease` check that `--force` doesn't?
3. You fix a typo in an error message. Then you remove an API endpoint. Starting from `1.4.2`, what versions do those two releases get?

### Break

Get stuck mid-rebase on purpose. In the lab, make `main` and a new branch `clash` change **the same line** of the same file. On `clash`, run `git rebase main`.

1. Read the output and `git status`. Which commit is Git trying to apply, and what is it asking you to do?
2. Abort once with `git rebase --abort` and confirm everything is as before.
3. Rebase again, and this time resolve the conflict and finish with `git rebase --continue`.

Explain why the rebase stopped on one specific commit, rather than conflicting once for the whole branch like a merge does.

### Review

Release notes from a team's changelog:

```
v1.4.3 (patch)
- Renamed the `url` field in the API response to `long_url` for clarity
- Fixed typo in 404 error message
```

Which number is wrong, and what should it have been? Who gets hurt if it ships like this? (Ask Claude for the answer once you've decided.)

## Commit

```bash
git switch main && git pull
git switch -c 06-history-and-tags
# write notes/git-history.md, tick lesson 06
git add -A && git commit -m "docs: add notes on history and versioning"
git push -u origin HEAD
gh pr create --web
gh pr merge --squash --delete-branch
# then step 7: tag + release on main
```

That's **Stage 1 done**. The repo is protected, every change is reviewed, and you've cut a release.

## Stretch

- `git bisect`: in the lab, make ten commits where one of them "breaks" a file (e.g. adds the word `BUG`). Use `git bisect start`, `git bisect bad`, `git bisect good <first-hash>`, then `git bisect run sh -c '! grep -q BUG file.txt'` to have Git find the culprit by binary search. Work out why the `!` is needed: what exit code does `bisect run` treat as "good"?
- Read your release on GitHub. Generated notes come from PR titles, which is one more reason to write good ones.

## If you're stuck

| Symptom | Likely cause |
|---|---|
| Push rejected after a rebase: `Updates were rejected because the tip of your current branch is behind` | You rewrote the branch's commits. On *your own* feature branch, `git push --force-with-lease`. Never on `main` (the ruleset blocks it anyway). |
| `fatal: tag 'v0.1.0' already exists` | It's already made locally. Check with `git show v0.1.0`. To redo it before pushing: `git tag -d v0.1.0`. |
| The tag doesn't appear on GitHub | Tags aren't pushed by default. Push it by name: `git push origin v0.1.0`. |
| `git describe` says `fatal: No names found` | The tag isn't on your local `main`. Run `git fetch --tags`, or check you tagged the right commit. |
