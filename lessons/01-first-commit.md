# 01 · First commit

## Goal

This repo lives in `~/devops`, your first commit is on GitHub, and a green tile for it is visible on your profile.

## Why this matters at work

Your first ticket at a new job usually starts with "clone the repo". Knowing exactly where code lives, which remote it talks to, and how your commits get attributed to you sounds trivial until a commit shows up under the wrong name in an audit, or you spend an afternoon editing a copy nobody uses.

## Concepts

**Clone vs. copy.** `git clone` doesn't just copy files. It copies the *entire history* and remembers where it came from, as a **remote** named `origin`. Copying a folder gets you files. Cloning gets you a repository.

**SSH vs. HTTPS remote URLs.**

| URL | How you authenticate |
|---|---|
| `git@github.com:rensyi/devops.git` | Your SSH key from Lesson 00 |
| `https://github.com/rensyi/devops.git` | A personal access token |

You'll use SSH.

**Commit → push.** A commit is saved only on your machine. `git push` sends your new commits to `origin`, and until then GitHub knows nothing about them.

**How GitHub counts a contribution.** A commit turns a tile green when all of these are true:
- its author email is verified on your account (the noreply address always is)
- it's on the repo's **default branch** (`main`)
- the repo isn't a fork

Lesson 03 onward uses pull requests, and a PR's commits count once it's merged into `main`.

**Where the lesson files come from.** Claude wrote `docs/` and `lessons/` into the old Windows folder (`C:\Users\Klarens\Documents\devops\devops`). Today you move them into your Linux clone and commit them. That's real work: you're adopting the project's plan into the repo.

Docs: [git clone](https://git-scm.com/docs/git-clone) · [Why contributions don't show up](https://docs.github.com/account-and-profile/setting-up-and-managing-your-github-profile/managing-contribution-settings-on-your-profile/why-are-my-contributions-not-showing-up-on-my-profile)

## Install

Nothing new today.

## Steps

1. **Clone** `git@github.com:rensyi/devops.git` into `~/devops`, then `cd` into it. Run `ls -la`. Notice the `.git` folder: that's the whole repository, and everything else is just the current snapshot.

2. **Look at the remote** with `git remote -v`. Both lines should show the `git@github.com:` URL.

3. **Bring the plan over.** Copy the `docs` and `lessons` folders from `/mnt/c/Users/Klarens/Documents/devops/devops/` into `~/devops/` (`cp -r`). Copy *only* those two folders. The Windows folder also has a hidden `.superpowers` folder, which is Claude's scratch space and doesn't belong in your repo.

4. **Watch Git notice.** Run `git status`. The two folders show as *untracked*. Stage them with `git add`, run `git status` again, and see how the wording changes. That's the staging area, and you'll dig into it in Lesson 02.

5. **Commit** with a message that says what and why, e.g. `docs: add DevOps learning path design, plan, and lessons`. Then run `git log --oneline` and see two commits: the original one and yours.

6. **Push.** Today only, push straight to `main` with `git push`. From Lesson 03 on, everything goes through a pull request.

7. **See it on GitHub.** Open `github.com/rensyi/devops` and check the files are there. Then open your profile (`github.com/rensyi`) and find today's tile on the contribution graph. It usually appears within a few minutes.

8. **Open the project in VS Code** with `code .` from `~/devops`.

9. **Retire the Windows copy.** Only after step 7 checks out, delete `C:\Users\Klarens\Documents\devops\devops` so you never edit the wrong copy. From now on, `~/devops` is the only working copy. (When you work with Claude Code, start it from `~/devops` inside Ubuntu.)

## Check

| Run (in `~/devops`) | You should see |
|---|---|
| `pwd` | `/home/klarens/devops` |
| `git log -1 --format='%an <%ae>'` | `rensyi <122665114+rensyi@users.noreply.github.com>` |
| `git status` | `Your branch is up to date with 'origin/main'.` and `nothing to commit, working tree clean` |
| `ls` | `README.md  docs  lessons` |
| Your GitHub profile | A green tile for today |

## Gate before merge

### Explain

1. What does `git clone` give you that copying the folder wouldn't?
2. After `git commit` but before `git push`, what does GitHub know about your commit?
3. Name the three conditions for a commit to count on your contribution graph.

### Break

This fault is **silent**: nothing errors, the commit just quietly doesn't count. That makes it worth seeing once.

With a clean working tree, make an empty commit as a stranger:

```bash
git -c user.email=someone@example.com commit --allow-empty -m "test: who am I?"
```

Now use `git log` with a format showing author name and email for the last 2 commits, and spot the difference. If this were pushed, which condition from Concepts would it fail, and what would the tile do? Remove it **without pushing** using `git reset --soft HEAD~1`, then confirm `git status` says you're up to date with `origin/main` again.

### Review

This is how a teammate set up their copy:

```bash
cd /mnt/c/Users/jdelacruz/Documents
git clone git@github.com:team/payments-api.git
cd payments-api && code .
```

The clone works and the editor opens. What will hurt them later, and why? (Ask Claude for the answer once you've decided.)

## Commit

Done in steps 4–6. This is the one lesson that pushes straight to `main`.

## Stretch

Look inside a commit. Run `cat .git/HEAD`, then `git cat-file -p HEAD`, then use `git cat-file -p` on the `tree` hash it prints. You've just walked Git's object database by hand. A commit is a snapshot pointer plus metadata, not a diff.

## If you're stuck

| Symptom | Likely cause |
|---|---|
| `Permission denied (publickey)` on clone or push | The SSH key isn't set up. Go back to Lesson 00's Check. |
| `fatal: destination path '.../devops' already exists` | You already cloned. `cd ~/devops` and continue. |
| `cp: cannot stat '/mnt/c/...'` | Typo in the path. Use Tab to autocomplete each folder name. |
| Push rejected with `GH007: Your push would publish a private email address` | The commit used a private email. Set the noreply email (Lesson 00's Check table), then fix the commit with `git commit --amend --reset-author --no-edit`. |
| No tile after 10 minutes | Check the commit email (`git log -1 --format=%ae`) and confirm the commit is on `main` on GitHub. |
