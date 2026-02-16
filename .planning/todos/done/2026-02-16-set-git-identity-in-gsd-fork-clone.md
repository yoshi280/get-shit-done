---
created: 2026-02-16T08:03:53.094Z
title: Set git identity in GSD fork clone
area: tooling
files:
  - ~/get-shit-done/.git/config
---

## Context

Forked `gsd-build/get-shit-done` to `yoshi280/get-shit-done`. Commits in the clone at `~/get-shit-done` used auto-detected identity (`thelorax@Mac.lan`). Need to set proper name/email.

## Action

```bash
cd ~/get-shit-done
git config user.name "Your Name"
git config user.email "your@email.com"
```

Optionally amend the two existing commits to fix authorship.
