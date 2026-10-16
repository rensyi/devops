# 04 · Conflicts & undo

## Goal

You've caused and resolved a merge conflict, you know which undo command fits which situation, and you've rescued "lost" commits with the reflog. All of it happens in a throwaway lab repo, and your own cheat sheet goes into `devops`.

## Why this matters at work

Conflicts happen weekly on any team, and panicking through one is how people delete a colleague's work. "Undo" is worse: `reset --hard` on a shared branch, followed by a force push, has wiped out other people's commits at real companies. Knowing *which* undo is safe *where* is what earns you trust with production repos.

## Concepts

**Merge conflicts** happen when two branches change the *same lines* (or one edits a file the other deleted). Git can't pick a winner, so it writes both versions into the file:

```
<<<<<<< HEAD
the version on the branch you're on
=======
the version from the branch you're merging in
>>>>>>> other-branch
```

You edit the file to the correct final content, delete the markers, `git add` it, and commit. `git merge --abort` is always available to back out.

**The three undo commands:**

| Command | What it changes | Safe on pushed/shared history? |
|---|---|---|
| `git restore <file>` | Throws away changes in the working tree (`--staged` unstages instead) | Yes. It doesn't touch history. |
| `git reset <commit>` | Moves the branch label back. `--soft` keeps changes staged, `--mixed` (the default) keeps them unstaged, `--hard` **discards** them. | **No.** It rewrites history. |
| `git revert <commit>` | Adds a *new* commit that undoes an old one | **Yes.** This is the safe way to undo on `main`. |

**The reflog** is Git's private diary of everywhere `HEAD` has pointed. "Lost" commits are usually still there, just unlabeled. `git reflog` finds them, and `git reset --hard <hash>` or a new branch brings them back. It only covers your machine and expires after about 90 days.

Docs: [Resolving conflicts](https://git-scm.com/docs/git-merge#_how_to_resolve_conflicts) · [Undoing things](https://git-scm.com/book/en/v2/Git-Basics-Undoing-Things) · [git reflog](https://git-scm.com/docs/git-reflog)

## Install

Nothing new today.

## Steps

**Part A: the lab.** Make it outside your real repo: `mkdir -p ~/sandbox && cd ~/sandbox && git init git-lab && cd git-lab`.

1. **Set up a conflict.** Create `menu.txt` with three lines (e.g. three dishes) and commit it on `main`. Create a branch `chef-a` and change line 2, then commit. Switch back to `main`, create `chef-b`, change **the same line 2** differently, and commit.

2. **Collide.** On `chef-b`, merge `chef-a`. Read Git's message. Run `git status` and notice what it says about the file. Open `menu.txt` in VS Code (`code menu.txt`) and look at the markers. VS Code also offers "Accept Current / Incoming / Both" buttons, and you should know what each does.

3. **Resolve it by hand.** Write the line you actually want, remove all markers, `git add`, and `git commit`. Look at `git log --oneline --graph --all`: you have a merge commit with two parents.

4. **Restore.** Edit `menu.txt`, then throw the edit away with `git restore`. Stage an edit, then unstage it with `git restore --staged`.

5. **Revert.** Pick an older commit and `git revert` it. Look at the log: history only grew.

6. **Reset.** Make two throwaway commits. Undo them with `git reset --soft HEAD~2` and check `git status`. Redo them, then try `--mixed`. Notice the difference in where the changes end up.

**Part B: your cheat sheet.** Back in `~/devops`, on branch `04-conflicts-and-undo`:

7. Create `notes/git-undo.md`, a one-page cheat sheet **in your own words**. For each situation, write which command you'd use and why:
   - "I edited a file and want the original back"
   - "I staged the wrong file"
   - "My last commit isn't pushed and it's wrong"
   - "A bad commit is already on `main`"
   - "I lost commits after a reset"

   You'll come back to this sheet. It's yours, not AI's.

## Check

| Run | You should see |
|---|---|
| (lab) `git log --oneline --graph --all` | a merge commit joining `chef-a` and `chef-b`, plus a `Revert "..."` commit |
| (lab) `grep -rn '<<<<<<<\|>>>>>>>' .` | nothing, so no leftover markers |
| (lab) `git status` | `nothing to commit, working tree clean` |
| (devops) `cat notes/git-undo.md` | your five situations, answered |

## Gate before merge

### Explain

Put these in the PR description.

1. What's the difference between `reset` and `revert`, and which one would you use on `main` and why?
2. In the conflict markers, which half was `HEAD`, and why?
3. The reflog saved you once. When would it *not* be able to?

### Break

In the lab, on `main`, make sure you have at least two commits you care about, then:

```bash
git reset --hard HEAD~2
```

They're gone from `git log`. Recover them **using only what `git reflog` shows you**: find the entry from just before the reset and get back to it. Then confirm with `git log` that both commits have returned.

### Review

Advice found in a team chat:

> "Pushed a bad commit to main? Easy: `git reset --hard HEAD~1 && git push --force`. Done."

What happens to the teammates who already pulled? What should the advice have been? (Ask Claude for the answer once you've decided.)

## Commit

```bash
git switch main && git pull
git switch -c 04-conflicts-and-undo
# write notes/git-undo.md, tick lesson 04 in lessons/README.md
git add -A && git commit -m "docs: add git undo cheat sheet"
git push -u origin HEAD
gh pr create --web
gh pr merge --squash --delete-branch
```

## Stretch

- Set `git config --global merge.conflictStyle zdiff3` and redo the conflict. The markers now also show the *original* line (the common ancestor), which makes most conflicts much easier to judge.
- Try `git stash` and `git stash pop` to park unfinished work while you switch branches.

## If you're stuck

| Symptom | Likely cause |
|---|---|
| Unsure whether you're mid-merge | `git status` says `You have unmerged paths`. Finish the merge (fix, `add`, `commit`) or bail out (`git merge --abort`). |
| Vim opened for the merge commit message | The default message is fine. Type `:wq` and Enter. |
| `HEAD detached at ...` | You checked out a commit, not a branch. If you want to keep work from here: `git switch -c rescue`. Otherwise: `git switch main`. |
| `git revert` on a merge commit errors with `is a merge but no -m option` | A merge has two parents, so Git needs to know which side to keep. Use `-m 1` (keep the main-line parent). |
