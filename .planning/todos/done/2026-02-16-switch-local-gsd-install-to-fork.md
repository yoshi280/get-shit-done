---
created: 2026-02-16T08:03:53.094Z
title: Switch local GSD install to fork
area: tooling
files:
  - ~/get-shit-done/bin/install.js
  - ~/.config/opencode/get-shit-done/VERSION
---

## Context

Local GSD is installed via `npx get-shit-done-cc@latest` which overwrites customizations on every update. The fork at `~/get-shit-done` now has both the upstream PR branch and the fork-only model routing branch.

## Action

Replace npm-based install with fork-based install:

```bash
# Option A: Run installer from fork
node ~/get-shit-done/bin/install.js --opencode --global

# Option B: npm link
cd ~/get-shit-done && npm link
```

Verify: `cat ~/.config/opencode/get-shit-done/VERSION` should match fork version.

## Notes

- After switching, `npx get-shit-done-cc@latest` should NOT be used (it would overwrite fork changes)
- Use `git pull` in the fork repo instead for updates
- The fork-only `fork/multi-provider-routing` branch needs to be merged or cherry-picked into whatever branch the installer runs from
