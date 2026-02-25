---
title: "Fixing fzf Keybindings in Zsh"
date: 2026-02-25
summary: "Why Ctrl-R/Ctrl-T/Alt-C fail with the system fzf package, and how to restore full keybindings with the official installer."
tags: ["linux", "zsh", "fzf", "cli", "writeup"]
---

## Problem

You have `fzf` installed on your system:

```bash
fzf --version
# 0.60 (devel)
```

However, keybindings like `Ctrl-R` (history search), `Ctrl-T` (file completion), and `Alt-C` (directory change) do not work in Zsh.

Upon inspection:

```bash
ls -l ~/.fzf.zsh
# ls: cannot access '/home/user/.fzf.zsh': No such file or directory
```

And the system-installed binary is located at:

```bash
which fzf
# /usr/bin/fzf
```

Root cause:

* The APT (or system) version of `fzf` often only installs the binary, not the shell scripts needed for keybindings and completion.
* Therefore, sourcing `~/.fzf.zsh` or `/usr/share/fzf/install` fails because the scripts are missing.

---

## Solution

### Step 1: Remove the system `fzf` (optional but recommended)

To avoid conflicts with the manual version:

```bash
sudo apt remove fzf
```

### Step 2: Install `fzf` manually via Git

Clone the official repository:

```bash
git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
```

Run the installer with all options:

```bash
~/.fzf/install --all
```

What `--all` does:

* Enables keybindings (`Ctrl-R`, `Ctrl-T`, `Alt-C`)
* Enables fuzzy completion
* Updates your `~/.zshrc` automatically

### Step 3: Reload Zsh

After installation, restart your terminal or run:

```bash
source ~/.zshrc
```

### Step 4: Test Keybindings

* `Ctrl-R` -> Search command history
* `Ctrl-T` -> Fuzzy file finder
* `Alt-C` -> Fuzzy change directory

They should now work immediately.

---

## Notes

* This method ensures you always have the latest `fzf` version and full functionality.
* No conflicts with old system-installed binaries.
* If using `oh-my-zsh`, make sure the `fzf` source line is after `source $ZSH/oh-my-zsh.sh` in `~/.zshrc`.
