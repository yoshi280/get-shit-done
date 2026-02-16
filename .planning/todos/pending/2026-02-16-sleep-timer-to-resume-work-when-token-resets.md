---
created: 2026-02-16T07:48:44.816Z
title: Sleep timer to resume work when token resets
area: tooling
files:
  - claude-resume.sh
---

## Problem

When Claude Code token limits reset (e.g., at 1am), there's no automated way to restart a session and restore GSD project context. Currently requires manual intervention to launch claude and run `/gsd:resume-work` after the reset window.

## Solution

A shell script (`~/claude-resume.sh`) was created that:
1. Accepts a target time (HH:MM) and project directory
2. Sleeps until the specified time
3. Launches `claude` in a tmux session
4. Sends `/gsd:resume-work` after TUI initialization

Potential improvements:
- Detect claude TUI readiness instead of fixed 10s sleep
- Support `launchd`/`cron` scheduling as an alternative to long sleep
- Add retry logic if claude fails to start
- Notification (terminal-notifier or similar) when session is ready
