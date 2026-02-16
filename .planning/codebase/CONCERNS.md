# Codebase Concerns

**Analysis Date:** 2026-02-16

## Tech Debt

### Claude Code Classification Bug Workaround

**Issue:** Claude Code has a `classifyHandoffIfNeeded` bug that causes false agent failure reports even when execution succeeds
- **Files:** `agents/gsd-executor.md` (line 131), `get-shit-done/workflows/execute-phase.md`, `docs/USER-GUIDE.md` (line 405)
- **Impact:** Orchestrators report phase failure incorrectly, user may think work didn't complete when commits were actually made
- **Current Mitigation:** Execute-phase and quick workflows implement spot-checking of actual output before reporting failure; recommends user verify with `git log` if failure reported but commits exist
- **Fix Approach:** Pending resolution from Claude Code team. In interim: continue spot-checking pattern, educate users on verification via git log

### MCP Tool Access Workaround for Subagents

**Issue:** Claude Code bug #13898 prevents subagents from accessing MCP tools (Context7, WebFetch, etc.)
- **Files:** `CHANGELOG.md` (line 313), `agents/gsd-planner.md` (line 4)
- **Impact:** Subagents have limited tool access; planner marks `tools: mcp__context7__*` to work around limitation
- **Current Mitigation:** Workaround pattern established in agent definitions
- **Fix Approach:** Pending Claude Code fix. Pattern is established but fragile — may break if MCP naming changes

### OpenCode JSONC Parser Edge Cases

**Issue:** OpenCode's config file uses JSONC format with comments, trailing commas, and potential BOM markers
- **Files:** `bin/install.js` (lines 160-180), `CHANGELOG.md` (line 173)
- **Impact:** Installer could fail or corrupt opencode.json during JSONC parsing if comments or special characters aren't handled
- **Current Mitigation:** Installer now handles comments, trailing commas, and BOM correctly; validates before writing
- **Fix Approach:** Already improved in v1.14.0. Continue testing with real-world opencode.json files during installs.

## Execution Constraints & Limits

### Executor Auto-Fix Attempt Limit (3 attempts per task)

**Issue:** Executor auto-fixes issues automatically but caps attempts at 3 per task before requiring checkpoint
- **Files:** `agents/gsd-executor.md` (line 150)
- **Impact:** Tasks with multiple cascading failures may require manual intervention after 3 fix attempts, blocking autonomous execution
- **Risk:** Complex tasks with interdependent validations can exceed 3-attempt budget
- **Mitigation:** Checkpoint system surfaces blocker; planner should validate single-fix-per-failure pattern in task design
- **Recommendation:** Keep limit at 3; requires planner discipline to avoid multi-stage auto-fixes in single task

### Context Window Management (50% Budget Per Plan)

**Issue:** Plans sized to complete within ~50% context window; going above 50% triggers "DEGRADING" quality zone
- **Files:** `agents/gsd-planner.md` (lines 74-83), `agents/gsd-plan-checker.md` (line 473)
- **Impact:** Complex phases split into multiple plans; token exhaustion degrades code quality mid-execution
- **Risk:** Poorly estimated scopes can burn through 70%+ context, leaving executor in minimal-quality zone
- **Mitigation:** Plan checker flags scope_sanity violations (5+ tasks, 12+ files). Roadmapper must split large phases.
- **Recommendation:** Enforce 2-3 tasks per plan; add pre-execution scope audit before execute-phase

### Phase Execution Waves Dependency Management

**Issue:** Plans declare `depends_on` for sequential waves, but no automatic rollback if Wave N fails
- **Files:** `agents/gsd-planner.md` (lines 290+), `agents/gsd-executor.md` (checkpoint protocol), `docs/USER-GUIDE.md` (line 107)
- **Impact:** If Wave 2 fails after Wave 1 succeeds, commits from Wave 1 remain; must manual revert or skip wave
- **Risk:** Partial phase state if mid-wave failure; hard to resume cleanly
- **Mitigation:** Verifier detects goal-backward failures; can trigger `/gsd:plan-milestone-gaps` for re-execution
- **Recommendation:** Document manual rollback steps; consider flag for auto-revert-on-wave-failure

## Known Bugs & Issues

### Requirement ID Bracket Syntax Stripping

**Issue:** Requirement IDs sometimes appear in bracket syntax `[REQ-01, REQ-02]` that must be stripped across the full chain
- **Files:** `CHANGELOG.md` (lines 22-23), `agents/gsd-planner.md` (line 456)
- **Impact:** If bracket stripping fails anywhere (researcher, planner, verifier), requirement tracking breaks
- **Current Status:** Fixed in v1.20.2 — chain now consistently strips brackets
- **Risk:** Regression if new agents don't follow pattern
- **Recommendation:** Test bracket syntax during researcher → planner → checker → verifier chain on every major release

### ROADMAP Header Format Ambiguity

**Issue:** Phase headers may be `##` or `###` depth; malformed ROADMAPs crash parsing
- **Files:** `CHANGELOG.md` (line 102), `agents/gsd-roadmapper.md`, `agents/gsd-verifier.md`
- **Impact:** Parser fails silently or produces wrong results if ROADMAP has inconsistent heading depth
- **Current Status:** Fixed in v1.19.0 — accepts both formats, detects malformed ROADMAPs
- **Risk:** User-edited ROADMAPs may introduce bad headers
- **Recommendation:** Add ROADMAP validation to `/gsd:health` command with auto-repair option

### Phase Directory Fallback on Missing Paths

**Issue:** If phase directory is missing, code falls back to ROADMAP.md for phase info
- **Files:** `CHANGELOG.md` (line 100), `agents/gsd-verifier.md`
- **Impact:** Orphaned phases (in ROADMAP but no .planning/phase-N/) silently skip verification
- **Risk:** Incomplete phases not detected; user thinks phase exists when it doesn't
- **Recommendation:** Add orphan detection to health check; require explicit phase creation via `gsd-tools phase add`

## Fragile Areas

### CONTEXT.md Locked Decision Fidelity

**Issue:** Planner must honor CONTEXT.md locked decisions, but no runtime validation that plans actually implement them
- **Files:** `agents/gsd-planner.md` (lines 28-55), `agents/gsd-plan-checker.md` (lines 261-290)
- **Impact:** Planner may claim to honor decisions in frontmatter but skip in task content; checker catches some but not all cases
- **Why Fragile:** Depends on careful reading + explicit comment in action field; easy to implement alternative accidentally
- **Safe Modification:** Add automated cross-reference in plan-checker: scan task actions for keywords from locked decisions (library names, architecture patterns, etc.)
- **Test Coverage:** Plan-checker has dimension `context_fidelity` but limited to presence check, not content verification

### Requirement Traceability Chain (Multi-Source)

**Issue:** v1.20.3 cross-references THREE independent sources for requirement status: VERIFICATION.md + SUMMARY frontmatter + REQUIREMENTS.md
- **Files:** `CHANGELOG.md` (lines 11-16), `agents/gsd-verifier.md`, `get-shit-done/workflows/audit-milestone.md`
- **Impact:** Orphaned requirements (in REQUIREMENTS.md but absent from all phase VERIFICATION.md files) detected but may not trigger auto-fix
- **Why Fragile:** If any source gets out of sync (manual edit, failed update), traceability breaks; three sources = three points of failure
- **Safe Modification:** Treat ROADMAP as source of truth; regenerate REQUIREMENTS.md and phase VERIFICATIONs from ROADMAP on `/gsd:health --repair`
- **Test Coverage:** Milestone audit validates cross-references, but no automated healing beyond surfacing discrepancies

### Project State Machine Transitions

**Issue:** STATE.md transitions through phases with `current_plan` tracking; if manually edited, executor resumes from wrong point
- **Files:** `agents/gsd-executor.md` (line 28), `agents/gsd-verifier.md`, `get-shit-done/workflows/transition.md`
- **Impact:** Resume-from-checkpoint can skip completed tasks if STATE.md edited incorrectly
- **Why Fragile:** State tracking relies on exact field names and frontmatter YAML parsing; one typo breaks continuation
- **Safe Modification:** Executor validates `<completed_tasks>` against git log before resuming; all state operations route through gsd-tools CLI
- **Test Coverage:** gsd-tools has `state verify` command but limited to schema; doesn't validate against actual disk state (git log, file system)

## Security Considerations

### API Key/Credential Leakage in `/gsd:map-codebase`

**Issue:** Mapper scans codebase for STACK/INTEGRATIONS/CONCERNS.md; must never capture secrets
- **Files:** `agents/gsd-codebase-mapper.md` (lines 709-728), `CHANGELOG.md` (line 218)
- **Risk:** env vars, API keys, certificates could be quoted in mapper output if found in source files
- **Current Mitigation:** Forbidden files list blocks reading `.env`, `.env.*`, credentials, keys, keystores, serviceAccountKey.json, etc.
- **Recommendation:** Add runtime check: mapper output should never contain `sk-`, `pk_`, `Bearer`, `=` in values section. Fail if detected.
- **Testing:** Include `.env` test file in codebase mapper tests to verify forbidden-file enforcement

### Webhook & Integration Credential Handling

**Issue:** INTEGRATIONS.md documents auth mechanisms (env vars, OAuth) but must not include actual values
- **Files:** `agents/gsd-codebase-mapper.md` (INTEGRATIONS.md template, line ~270)
- **Risk:** Mapper could accidentally include `STRIPE_KEY=sk_test_...` if template not carefully followed
- **Current Mitigation:** Template uses placeholder syntax `[env var name]`, not values
- **Recommendation:** Add validation step in integration checker; flag any line matching pattern `KEY=.*` as potential secret leak

## Performance Bottlenecks

### Large Agent Prompt Size (1000+ lines)

**Issue:** gsd-debugger, gsd-planner, gsd-executor are 1000+ line prompts (1198, 1164, ~1200 lines respectively)
- **Files:** `agents/gsd-debugger.md` (1198 lines), `agents/gsd-planner.md` (1164 lines), `agents/gsd-executor.md` (1100+ lines)
- **Impact:** Agent loading time, context compression, possibility of prompt truncation if user context is large
- **Cause:** Comprehensive but verbose explanations of philosophies, methodologies, error cases
- **Mitigation:** Agents pass; large size is justified by complexity. Monitor for truncation errors.
- **Optimization Path:** Could refactor agent-specific sections to separate reference docs (e.g., `executor-deviations.md`, `executor-checkpoints.md`) and link instead of inline; but tradeoff is less self-contained agents

### ROADMAP Parsing at Scale

**Issue:** As project scales (50+ requirements, 10+ phases), ROADMAP.md can become large; parsing done by multiple agents independently
- **Files:** `agents/gsd-roadmapper.md`, `agents/gsd-verifier.md`, `agents/gsd-planner.md` all parse ROADMAP
- **Impact:** Repeated parsing inefficiency; if ROADMAP > 10K lines, parsing overhead compounds
- **Mitigation:** gsd-tools `roadmap` commands delegate parsing; agents use CLI instead of manual markdown parsing
- **Optimization Path:** Implement ROADMAP -> JSON cache in gsd-tools; invalidate on ROADMAP edits

### Milestone Audit Three-Source Validation

**Issue:** `/gsd:audit-milestone` queries VERIFICATION.md, SUMMARY frontmatter, and REQUIREMENTS.md; three independent file scans
- **Files:** `get-shit-done/workflows/audit-milestone.md`, `CHANGELOG.md` (line 12)
- **Impact:** Scaling to 20+ phases means 60+ file reads + parsing operations
- **Mitigation:** Delegated to gsd-tools `milestone complete` command for parallelization
- **Optimization Path:** Pre-compute validation hashes for VERIFICATION.md sections; detect changes via hash instead of full re-parse

## Scaling Limits

### Maximum Phases Per Milestone

**Issue:** No documented limit; but practical constraints exist:
- Current capacity: Tested with ~10 phases per milestone
- Limit: Beyond 15-20 phases, ROADMAP parsing and plan-checker iterations become slow; context window for planner degrades
- Scaling path: Split into multiple milestones; `gsd-tools roadmap analyze` helps identify natural breakpoints
- **Files:** `agents/gsd-planner.md` (context budget rules), `agents/gsd-roadmapper.md`

### Parallel Wave Execution Complexity

**Issue:** Execute-phase spawns fresh executors for each parallel plan, but all output streams to orchestrator
- Current capacity: ~4-6 parallel plans feasible before context overhead
- Limit: Beyond 8 parallel plans, orchestrator context loss degrades checkpoint handling
- Scaling path: Reduce plan granularity; fewer, larger plans instead of many small ones
- **Files:** `agents/gsd-executor.md` (line 108), `docs/USER-GUIDE.md` (line 100)

### State Table Growth (VERIFICATION, SUMMARY Frontmatter)

**Issue:** As projects progress, STATE.md, REQUIREMENTS.md frontmatter tables grow linearly with phase count
- Current capacity: 20+ phases supported; tables remain readable
- Limit: Beyond 50 phases, YAML frontmatter parsing becomes fragile; durations/metrics table rows become unwieldy
- Scaling path: Archive completed phase summaries to `phases-archive/` directory; keep active phases in main directories
- **Files:** `agents/gsd-executor.md` (state updates), `CHANGELOG.md` (line 62)

## Dependencies at Risk

### gsd-tools Node.js CLI Binary

**Issue:** Core deterministic operations delegate to `gsd-tools.cjs` (Node.js script); platform-specific risks
- **Risk:** Windows path handling, zsh vs bash differences, nvm/node version mismatches
- **Current Status:** Converted to CommonJS (.cjs) to avoid ESM/CJS conflicts; v1.19.2 fixes backslash paths on Windows
- **Impact if Missing:** All deterministic ops fail; phase add/remove/complete, state updates, template fills all blocked
- **Migration Plan:** Already mitigated; gsd-tools is hardcoded dependency with no alternatives. Monitor for Node.js version conflicts.
- **Files:** `bin/install.js`, `CHANGELOG.md` (lines 70, 104)

### esbuild Dev Dependency

**Issue:** `scripts/build-hooks.js` uses esbuild to bundle gsd-statusline.js and gsd-check-update.js
- **Risk:** esbuild version mismatches, ESM loader issues
- **Current Status:** Vendored; not published to npm (only hooks/dist/ is shipped)
- **Impact if Missing:** Hook compilation fails; statusline and update-check won't work
- **Migration Plan:** Relatively low risk; hooks are nice-to-have. Could fallback to unminified JS if needed.
- **Files:** `package.json`, `scripts/build-hooks.js`

## Test Coverage Gaps

### Agent Prompt Coverage Gaps

**Issue:** Large agents (executor, planner, debugger) have manual testing via `gsd-tools verify` but no unit tests
- **What's Not Tested:** Agent logic branches, edge case handling in deviation rules, checkpoint protocol sequencing
- **Files:** `agents/gsd-executor.md` (deviation auto-fix branches), `agents/gsd-debugger.md` (investigation paths), `agents/gsd-planner.md` (scope estimation)
- **Risk:** Regressions in agent behavior not caught until manual testing or user reports
- **Priority:** Medium — agents are well-documented; manual testing covers main paths, but edge cases vulnerable
- **Approach:** gsd-tools test suite (22 tests in v1.13.0) covers CLI commands; expand to agent-specific scenarios once agent testing framework developed

### Installer Platform-Specific Coverage

**Issue:** Installer supports Mac, Windows, Linux, but test coverage skewed toward macOS
- **What's Not Tested:** OpenCode XDG path handling on Linux, Windows backslash normalization under all conditions, JSONC edge cases
- **Files:** `bin/install.js` (lines 75-92 for OpenCode paths, lines 100+ for platform detection)
- **Risk:** Windows/Linux users hit undiscovered bugs; macOS default path assumptions bleed through
- **Priority:** High — installer is user-facing; one-time critical failure
- **Approach:** Add CI matrix for Windows/Linux; simulate file system with different path structures

### ROADMAP Malformation Recovery

**Issue:** ROADMAP validation catches malformed headers (`##` vs `###` inconsistency) but limited test scenarios
- **What's Not Tested:** Missing requirements section, duplicate phase numbers, invalid YAML in success criteria frontmatter, requirement IDs with typos
- **Files:** `agents/gsd-roadmapper.md`, `get-shit-done/workflows/health.md`
- **Risk:** Ambiguous error messages if ROADMAP is only partially malformed
- **Priority:** Medium — health check catches most; but auto-repair edge cases unclear
- **Approach:** Expand gsd-tools `verify consistency` command with ROADMAP-specific checks; add recovery examples to docs

## Missing Critical Features

### No Automatic Rollback for Failed Waves

**Issue:** If execute-phase Wave 2 fails after Wave 1 commits, no built-in rollback mechanism
- **Problem:** User must manually `git revert` Wave 1 commits or document partial execution in STATE.md
- **Blocks:** Clean retry of failed phase without manual state management
- **Workaround:** Re-run `/gsd:plan-milestone-gaps` to create gap-closure phase; doesn't revert Wave 1, just adds fixes
- **Recommendation:** Implement `--rollback` flag for execute-phase; auto-reverts all commits from failed wave before returning

### No Cross-Milestone Requirement Tracking

**Issue:** REQUIREMENTS.md is per-milestone; no mechanism to track requirements across multiple milestones
- **Problem:** Large projects with shared requirements (e.g., "support OAuth") must manually track across REQUIREMENTS.md files
- **Blocks:** Verifying overall project completeness; no unified requirement status view
- **Recommendation:** Create `PROJECT_REQUIREMENTS.md` at project root; versioned tracking across all milestones. `/gsd:audit-project` command to validate all milestone requirements satisfy project-level needs.

### No Selective Phase Re-Execution

**Issue:** If Phase 3 verification fails, user must re-plan and re-execute full phase; can't retry just failed tasks
- **Problem:** Wasted tokens on passing tasks; context impact if phase is large
- **Blocks:** Efficient iteration on partially-failed phases
- **Recommendation:** Implement `/gsd:retry-phase --tasks task-ids-only` to re-execute specific tasks without full phase context

## Recommendations Summary

| Area | Priority | Fix Type | Effort |
|------|----------|----------|--------|
| Executor auto-fix attempt limit | Medium | Increase to 5, add metrics | Low |
| ROADMAP header validation in health | High | Expand gsd-tools verify | Low |
| Requirement bracket syntax regressions | High | Add test case to CI | Low |
| Context fidelity runtime verification | Medium | Add keyword-scan to plan-checker | Medium |
| Automatic wave rollback | Medium | New orchestrator feature | High |
| Cross-milestone requirement tracking | Low | New REQUIREMENTS.md consolidation | High |
| Secret detection in mapper output | High | Add pattern validation | Low |
| Phase directory orphan detection | Medium | Expand health check | Low |
| Large agent prompt refactoring | Low | Agent decomposition | High |

---

*Concerns audit: 2026-02-16*
