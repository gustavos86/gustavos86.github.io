---
title: Git commands to remember
date: 2024-07-19 14:30:00 -0700
categories: [GIT]
tags: [git]     # TAG names should always be lowercase
---

**NOTE:** Always proceed with caution.

## Add code to the last commit (not adding a new commit)

```bash
git add .
git commit --amend --no-edit
```

## Undo last commit

The `--soft` flag makes sure that the changes in undone revisions are preserved.
After running the command, you'll find the changes as uncommitted local modifications in your working copy.

```bash
git reset --soft HEAD~1
```

If you don't want to keep these changes, simply use the `--hard` flag

```bash
git reset --hard HEAD~1
```

## Add Tags

These are also known as "releases" in the GuiHub GUI

```bash
git tag -a "v0.0.1" -m "First release of this module - example"
git push --follow-tags
```

## References

- [https://www.git-tower.com/learn/git/faq/undo-last-commit](https://www.git-tower.com/learn/git/faq/undo-last-commit)