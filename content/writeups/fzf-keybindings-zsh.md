---
title: "Fixing fzf Keybindings in Zsh"
date: 2026-02-25
summary: "Restore Ctrl-R/Ctrl-T/Alt-C in Zsh by installing full fzf scripts."
tags: ["linux", "zsh", "fzf", "cli", "writeup"]
---

## Problem

`fzf` was installed, but `Ctrl-R`, `Ctrl-T`, and `Alt-C` did not work in Zsh.
Cause: system package had the binary, not full shell integration scripts.

## Solution

1. Optional: remove distro `fzf`:

```bash
sudo apt remove fzf
```

2. Install official `fzf` with scripts:

```bash
git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
~/.fzf/install --all
```

3. Reload shell:

```bash
source ~/.zshrc
```

## Result

`fzf` keybindings and completion work normally in Zsh.
