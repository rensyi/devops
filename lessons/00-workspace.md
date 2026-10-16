# 00 · Workspace

## Goal

You work in Ubuntu from VS Code, and GitHub trusts your machine through an SSH key.

## Why this matters at work

Nearly every server, container, and CI runner you'll touch runs Linux. On your first day, someone will hand you access to a Linux box, and they'll expect you to be comfortable in a terminal, not hunting for a GUI. They'll also expect you to authenticate to GitHub with a key, never a password. Today you set up the environment you'll use for every lesson after this one.

## Concepts

**WSL (Windows Subsystem for Linux)** runs a real Ubuntu next to Windows. It has its *own* filesystem:

| Path inside Ubuntu | What it is |
|---|---|
| `/home/klarens` (also written `~`) | Your Linux home. **All work happens here.** |
| `/mnt/c/...` | Your Windows `C:` drive, reached through a translation layer. |

Working under `/mnt/c` *seems* fine until it isn't: file permissions don't behave like Linux, Docker is slow at reading those files, and Windows line endings sneak into scripts. So: Linux files live in Linux. From Windows, your Linux home is visible at `\\wsl.localhost\Ubuntu\home\klarens` if you ever need it.

**VS Code + the WSL extension.** VS Code's window stays on Windows, but the editor's "brain" (terminal, extensions, file access) runs *inside* Ubuntu. That's the same model as editing files on a remote server, which you'll do on the job.

**SSH keys** come in pairs:
- The **private key** (`~/.ssh/id_ed25519`) never leaves your machine. Anyone holding it *is* you.
- The **public key** (`~/.ssh/id_ed25519.pub`) is safe to share. You give it to GitHub, and GitHub uses it to check that whoever connects holds the matching private key.

`ed25519` is the modern key type: short, fast, and what GitHub recommends.

Docs: [WSL](https://learn.microsoft.com/windows/wsl/about) · [VS Code WSL](https://code.visualstudio.com/docs/remote/wsl) · [GitHub SSH keys](https://docs.github.com/authentication/connecting-to-github-with-ssh)

## Install

**VS Code WSL extension.** In VS Code on Windows, open Extensions (`Ctrl+Shift+X`), search **WSL** (publisher: Microsoft), and install it.

Check it from Ubuntu (open the **Ubuntu** app from the Start menu):

```bash
code --version
```

The first time, this may install a small VS Code server inside Ubuntu. You should then see a version number like `1.141.0`.

## Steps

1. **Get oriented.** In the Ubuntu terminal, find out who you are, where you are, and what's in your home folder (`whoami`, `pwd`, `ls -la`). Then peek at `/mnt/c/Users/Klarens` to see your Windows files from the Linux side.

2. **Open VS Code from Ubuntu.** Run `code ~` from Ubuntu. Look at the bottom-left corner of the VS Code window: it should say **WSL: Ubuntu**. Open VS Code's terminal (`` Ctrl+` ``) and run `pwd`. It's a Linux terminal.

3. **Confirm your Git identity.** You set this already. Run `git config --global --list` and confirm `user.name` and `user.email` are there, with the noreply email.

4. **Create an SSH key.** Use `ssh-keygen` with type `ed25519` and a comment (`-C`) that labels the key, e.g. `"klarens-laptop-wsl"`. Accept the default file location. A passphrase is recommended: it protects the key if your laptop is ever compromised. (Stretch shows how to avoid typing it on every push.)

5. **Give GitHub the public half.** Print the **`.pub`** file and copy the whole line. On GitHub: **Settings → SSH and GPG keys → New SSH key**, key type **Authentication Key**, title it after your laptop, and paste.

6. **Test the connection** with `ssh -T git@github.com`. The first time, SSH shows GitHub's fingerprint and asks if you trust it. Before typing `yes`, compare it to the one GitHub publishes ([GitHub's SSH key fingerprints](https://docs.github.com/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints)). For ed25519 it is `SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU`. Checking this prevents a man-in-the-middle attack.

## Check

| Run | You should see |
|---|---|
| `whoami` | `klarens` |
| `git config --global user.email` | `122665114+rensyi@users.noreply.github.com` |
| `ls -l ~/.ssh` | `id_ed25519` with `-rw-------` (only you can read it) and `id_ed25519.pub` with `-rw-r--r--` |
| `ssh -T git@github.com` | `Hi rensyi! You've successfully authenticated, but GitHub does not provide shell access.` |
| VS Code bottom-left | `WSL: Ubuntu` |

`ssh -T` exits with code 1 even when it worked (check with `echo $?`). That's because GitHub refuses to open a shell. The greeting is what proves success.

## Gate before merge

There's no pull request yet (that starts in Lesson 03), so answer these in a scratch note or out loud. Out loud counts: interviewers ask exactly these.

### Explain

1. Which half of the key pair did you give GitHub, and why is it safe to share it?
2. Why is the private key's permission `-rw-------`, and what do you think SSH does if it's readable by others?
3. Why do we keep project files in `~` and not under `/mnt/c`?

### Break

Rename your private key so SSH can't find it:

```bash
mv ~/.ssh/id_ed25519 ~/.ssh/id_ed25519.bak
```

Run `ssh -T git@github.com` again and read the error. Then run it with `-v` (verbose) and find the lines showing **which key files SSH tried**. Explain the failure from that output, then put the key back with `mv` in the other direction and confirm the greeting returns.

### Review

A classmate is about to push their first commits to a **public** repo. This is their `~/.gitconfig`:

```ini
[user]
    name = jdelacruz
    email = jdelacruz.personal@gmail.com
[init]
    defaultBranch = main
[core]
    autocrlf = input
```

Everything works. So what's the problem, and what would you tell them to change *before* the first push? (Ask Claude for the answer once you've decided.)

## Commit

Nothing to commit today. Your first commit happens in Lesson 01.

## Stretch

- **Stop retyping your passphrase.** Start an agent for the session with `eval "$(ssh-agent -s)"`, then add your key with `ssh-add ~/.ssh/id_ed25519`. You'll type the passphrase once, and pushes stop asking. Look up `keychain` (an apt package) if you want this to survive terminal restarts.
- Run `ssh -vT git@github.com` on a *working* key and read the whole handshake. Find the line where your key is accepted.

## If you're stuck

| Symptom | Likely cause |
|---|---|
| `code: command not found` in Ubuntu | VS Code wasn't added to PATH, or the terminal was open during install. Close and reopen Ubuntu. Still missing? Reinstall VS Code with "Add to PATH" checked. |
| `Permission denied (publickey)` | GitHub doesn't have this key's `.pub`, or SSH can't find the private key. Run `ssh -vT git@github.com` and check which files it offered. |
| `Host key verification failed` | You answered `no` to the fingerprint prompt. Run the test again and answer `yes` after comparing the fingerprint. |
| `ssh: Could not resolve hostname github.com` | No network inside WSL (a VPN can cause this). Test with `ping -c 3 github.com`. |
| `WARNING: UNPROTECTED PRIVATE KEY FILE!` | Permissions on the private key are too open. Fix with `chmod 600 ~/.ssh/id_ed25519`. |
