---
title: "git log --oneline --graph for Visual Branch History"
date: 2025-09-20
tags: [git, command-line, productivity]
---

Was trying to understand the merge history of llmswap before rebasing. Regular `git log` was just a wall of text - impossible to see the branch structure.

Discovered `--oneline --graph` flags:

```bash
# Regular log - hard to follow
git log
commit a1b2c3d Author: Me Date: ... Long message...

# Visual log - see the structure!
git log --oneline --graph

* a1b2c3d (HEAD -> main) Add caching feature
* | b2c3d4e Merge pull request #12
|\|
| * c3d4e5f Fix provider error handling
| * d4e5f6g Update tests
|/
* e5f6g7h Initial commit
```

The graph shows:
- `*` commits
- `|` branch lines
- `/` and `\` merges
- Branch relationships

I added an alias to my `.gitconfig`:

```bash
git config --global alias.lg "log --oneline --graph --all --decorate"

# Now just:
git lg
```

Makes understanding complex merge histories so much easier. Can see exactly where branches diverged and merged back.
