---
created: 2026-02-16T08:03:53.094Z
title: Periodic upstream sync for GSD fork
area: tooling
files:
  - ~/get-shit-done
---

## Context

Fork `yoshi280/get-shit-done` tracks `gsd-build/get-shit-done` (upstream). Two branches exist:

- `feat/selectable-research-dimensions` — PR #610 submitted upstream
- `fork/multi-provider-routing` — fork-only, modifies `model-profiles.md` which upstream also actively develops

## Action (run periodically)

```bash
cd ~/get-shit-done
git fetch upstream
git checkout main
git merge upstream/main
git push origin main

# Rebase fork-only branch (expect conflicts on model-profiles.md)
git checkout fork/multi-provider-routing
git rebase main
git push origin fork/multi-provider-routing --force-with-lease
```

## Notes

- If PR #610 is merged, delete `feat/selectable-research-dimensions` branch
- If PR #610 is rejected, rebase it onto latest main and keep it in the fork
- `model-profiles.md` will likely conflict on rebase — resolve by keeping fork additions on top of upstream changes
