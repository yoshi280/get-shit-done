# External Integrations

**Analysis Date:** 2026-02-16

## APIs & External Services

**Web Search:**
- Brave Search API - Web search for research agents
  - SDK/Client: Native `fetch()` to `https://api.search.brave.com/res/v1/web/search`
  - Auth: `BRAVE_API_KEY` environment variable or `~/.gsd/brave_api_key` file
  - Implementation: `gsd-tools.cjs` websearch command (lines 2118-2160)
  - Query parameters: `q` (query), `count` (limit), `freshness` (day|week|month)
  - Optional feature: Can be disabled if API key not provided
  - Used by: `gsd-phase-researcher`, `gsd-project-researcher` agents

**Package Registry:**
- npm Registry - Check for GSD updates
  - Command: `npm view get-shit-done-cc version`
  - Endpoint: Default npm registry (usually https://registry.npmjs.org)
  - Purpose: Compare installed version against latest published version
  - Implementation: `gsd-check-update.js` hook, executed at SessionStart
  - Runs in background process (non-blocking)
  - Timeout: 10 seconds
  - Result cached in `~/.claude/cache/gsd-update-check.json` (10 minute TTL)

## Version Control Integration

**Git:**
- Git command execution for SCM operations
  - Operations: `git check-ignore`, commit creation, status checking
  - Executed via: `child_process.execSync()` with escaped arguments
  - Safe execution: Arguments sanitized to prevent injection (alphanumeric + safe chars only)
  - Used by: Phase operations, commit docs, branching workflows
  - Implementation: `gsd-tools.cjs` git wrapper functions

**GitHub:**
- Repository hosting (not an active integration in code)
- Metadata: Repository URL in `package.json` → `https://github.com/glittercowboy/get-shit-done`
- Issues URL: `https://github.com/glittercowboy/get-shit-done/issues`

## Configuration Storage

**Local File System:**
- Configuration directory locations (runtime-specific):
  - Claude Code: `~/.claude/` (or `CLAUDE_CONFIG_DIR`)
  - OpenCode: `~/.config/opencode/` (respects `XDG_CONFIG_HOME`)
  - Gemini: `~/.gemini/` (or `GEMINI_CONFIG_DIR`)
  - Local project: `./.claude/`, `./.opencode/`, `./.gemini/` (if installing locally)

**Configuration Files:**
- `settings.json` - Hook and statusline registration (Claude/Gemini)
- `opencode.json` - Permission settings (OpenCode only)
- `get-shit-done/VERSION` - Current installed version
- `get-shit-done/CHANGELOG.md` - Version history
- `gsd-file-manifest.json` - File hashes for modification detection (SHA256)
- `gsd-local-patches/` - Backups of user-modified files
- `cache/gsd-update-check.json` - Cached update check results

## CLI Runtime Integration

**Claude Code:**
- Hook registration in `settings.json`:
  - StatusLine hook: Runs `gsd-statusline.js` for status display
  - SessionStart hook: Runs `gsd-check-update.js` for background update check
- Settings path: `~/.claude/settings.json`
- Frontmatter format: YAML with `allowed-tools:` array
- Tool names: Standard names (Read, Write, Bash, Grep, etc.)

**OpenCode:**
- Command structure: Flat namespace (`/gsd-help`, `/gsd-plan`, etc.)
- Implementation: Flattened from Claude structure (`gsd/help.md` → `command/gsd-help.md`)
- Frontmatter conversion: YAML `allowed-tools:` → `tools:` object with boolean values
- Tool names: Lowercase (read, write, bash, grep, question, skill, etc.)
- Permissions: Configured in `opencode.json` with `permission.read` and `permission.external_directory`
- XDG compliance: Respects `XDG_CONFIG_HOME` and `XDG_CONFIG_DIRS`

**Gemini CLI:**
- Frontmatter format: YAML with `tools:` array (Gemini-specific tool names)
- Tool name mapping: Built-in tools use snake_case (read_file, run_shell_command, etc.)
- File format: Agents converted to `.toml` files for Gemini compatibility
- Experimental feature: Requires `settings.experimental.enableAgents = true`
- Template variables: Escaped to prevent Gemini template validation errors (`${VAR}` → `$VAR`)

## Monitoring & Observability

**Update Checking:**
- Service: npm Registry polling
- Frequency: Once per session (SessionStart hook)
- Process: Background spawn (non-blocking)
- Cache: `~/.claude/cache/gsd-update-check.json`
- Display: Status shown in statusline if update available

**Logging:**
- None centralized (each command logs to stdout/stderr)
- Hooks write cache files for state persistence

## CI/CD & Deployment

**Publishing:**
- Package: Published to npm as `get-shit-done-cc`
- Registry: npm public registry
- Entry point: `bin/install.js`
- Included files: `bin/`, `commands/`, `get-shit-done/`, `agents/`, `hooks/dist/`, `scripts/`

**Local Installation:**
- Global: `npx get-shit-done-cc --claude --global`
- Local/Project: `npx get-shit-done-cc --claude --local`
- Uninstall: `npx get-shit-done-cc --claude --global --uninstall`

**Pre-publish Hook:**
- npm script: `prepublishOnly` runs `npm run build:hooks`
- Purpose: Ensures hooks bundled in `hooks/dist/` before package upload

## Environment Configuration

**Required env vars:**
- `BRAVE_API_KEY` - Optional but enables web search (can also use `~/.gsd/brave_api_key`)

**Optional env vars for configuration override:**
- `CLAUDE_CONFIG_DIR` - Override Claude config directory
- `OPENCODE_CONFIG_DIR` - Override OpenCode config directory
- `GEMINI_CONFIG_DIR` - Override Gemini config directory
- `XDG_CONFIG_HOME` - OpenCode respects standard XDG paths
- `HOME` - Used for home directory expansion

**Secrets location:**
- Brave API key: `process.env.BRAVE_API_KEY` or `~/.gsd/brave_api_key` file
- No other sensitive data stored in code or config (credentials handled by respective runtimes)

## Webhooks & Callbacks

**Incoming:** None detected

**Outgoing:**
- Status information: Statusline hook sends formatted status to IDE
- No external webhook calls from GSD system itself

## No Active Integrations For

**Database:** Not applicable (file-based system only)

**File Storage:** Local filesystem only (no cloud storage integration)

**Caching:** In-memory during execution, filesystem-based results cache (update check JSON)

**Authentication & Identity:**
- Handled by respective runtime (Claude Code, OpenCode, Gemini)
- GSD has no built-in auth system
- Brave Search auth via API key only

**Error Tracking:** None (stdout/stderr logging only)

---

*Integration audit: 2026-02-16*
