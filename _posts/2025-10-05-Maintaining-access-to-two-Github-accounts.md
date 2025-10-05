---
title: Maintaining Access to Two GitHub accounts
date: 2025-10-05 09:00:00 -0700
categories: [GIT]
tags: [git]     # TAG names should always be lowercase
---

I have 2 different GitHub accounts and I need to maintain write access to both from may macOS laptop.
Here what I did:

- Generate a two SSH keys, one per GitHub account

GitHub does not access the same SSH key for more than one account.

```
ssh-keygen -t ed25519 -C "gustavoserrano86@gmail.com" -f ~/.ssh/id_ed25519_github_gustavos86
ssh-keygen -t ed25519 -C "h.serranog@uniandes.edu.co" -f ~/.ssh/id_ed25519_github_hserranog
```

- Add each `.pub` key to the corresponding GitHub account

Go to GitHub → Settings → SSH and GPG keys → New SSH key

- Configure `~/.ssh/config` to differentiate them

```
# gustavos86 account
Host github.com-gustavos86
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_github_gustavos86
    IdentitiesOnly yes

# hserranog account
Host github.com-hserranog
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_github_hserranog
    IdentitiesOnly yes
```

- Update repo remotes to use the correct host

```
git remote set-url origin git@github.com-gustavos86:gustavos86/gustavos86.github.io.git
```

Make sure to update `some-repo.git` with the corresponding repo name

```
git remote set-url origin git@github.com-hserranog:hserranog/some-repo.git
```