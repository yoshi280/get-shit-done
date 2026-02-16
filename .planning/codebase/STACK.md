# Technology Stack

**Analysis Date:** 2026-02-16

## Languages

**Primary:**
- JavaScript (Node.js) - CLI tools, installation scripts, hooks, and core GSD system

**Secondary:**
- Markdown - All agent and command definitions (Claude Code/OpenCode/Gemini native format)
- YAML - Frontmatter in markdown files for metadata

## Runtime

**Environment:**
- Node.js 16.7.0 or higher (specified in `package.json` engines field)

**Package Manager:**
- npm (lockfile: `package-lock.json` present)

## Frameworks

**Core:**
- Vanilla Node.js - No framework dependencies used. System relies on pure Node.js standard library

**Build/Dev:**
- esbuild ^0.24.0 - Bundler for pre-build preparation (dev dependency only)

**Runtime Hooks:**
- Child process execution (`child_process.spawn`, `execSync`) - For background update checks and hook execution
- File system operations (`fs` module) - Central for all file I/O operations
- Path handling (`path` module) - Cross-platform path resolution

## Key Dependencies

**Zero Production Dependencies:**
- `package.json` contains empty `dependencies: {}` object
- All functionality uses only Node.js built-in modules

**Dev Dependencies (Build-time only):**
- `esbuild` ^0.24.0 - Used in `scripts/build-hooks.js` to prepare hooks for distribution

## Built-in Modules Used

**Core utilities:**
- `fs` - File system operations (read, write, copy, delete, manifest generation)
- `path` - Path resolution (tilde expansion, cross-platform paths)
- `crypto` - SHA256 hashing for file manifests and local patch detection
- `os` - OS detection (home directory, platform-specific paths)
- `child_process` - Execute git commands and npm version checks
- `readline` - Interactive prompts during installation

**Key patterns:**
- `execSync()` - Git operations, npm version checks
- `spawn()` - Background update check hook (non-blocking)
- SHA256 hashing - Track modified GSD files between updates (local patch persistence)

## Configuration

**Environment Variables:**

*Required for features:*
- `BRAVE_API_KEY` - Enables web search via Brave Search API for researchers
  - Alternative location: `~/.gsd/brave_api_key` file
  - Used by: `/gsd:websearch` command, researcher agents

*Runtime selection (optional):*
- `CLAUDE_CONFIG_DIR` - Override Claude Code config location (default: `~/.claude`)
- `OPENCODE_CONFIG_DIR` - Override OpenCode config location (checked before XDG_CONFIG_HOME)
- `XDG_CONFIG_HOME` - OpenCode respects XDG Base Directory spec
- `GEMINI_CONFIG_DIR` - Override Gemini config location (default: `~/.gemini`)

**Configuration Files Created:**
- `.planning/config.json` - GSD project configuration (model profiles, workflow settings)
- `package.json` (in config dir) - Forces CommonJS mode to prevent module resolution issues
- `gsd-file-manifest.json` - SHA256 hashes of installed files for update detection
- `gsd-local-patches/` directory - Backups of user-modified GSD files before updates

## Installation & Distribution

**Installation Entry Point:**
- `bin/install.js` - 1,816 lines, handles all installation workflows
  - Interactive or scripted installation
  - Multi-runtime support (Claude Code, OpenCode, Gemini)
  - Global or local (per-project) installation
  - Frontmatter conversion between runtime formats
  - Settings.json hook registration

**Build Process:**
- `scripts/build-hooks.js` - Copies compiled hooks to `hooks/dist/` directory
- Hooks are bundled with package on publish (included in `files` array)

**Installed Files Structure:**
- `bin/install.js` → `{configDir}/` (entry point)
- `get-shit-done/` → `{configDir}/get-shit-done/` (skill/core system)
- `commands/gsd/` → `{configDir}/commands/gsd/` (Claude/Gemini) or `{configDir}/command/gsd-*` (OpenCode flat)
- `agents/` → `{configDir}/agents/` (distributed agents)
- `hooks/dist/` → `{configDir}/hooks/` (statusline, update check)
- `CHANGELOG.md` → `{configDir}/get-shit-done/CHANGELOG.md` (version history)
- `VERSION` file created with current npm package version

## Platform Requirements

**Development:**
- Node.js 16.7.0+ installed locally
- npm (comes with Node.js)
- Git (for installation verification and version checks)

**Production (Runtime):**
- Node.js 16.7.0+ on user's machine
- Claude Code, OpenCode, or Gemini CLI installed
- Git (for GSD's internal version control operations)

**Supported Platforms:**
- macOS (primary development target)
- Linux (explicitly supported via cross-platform path handling)
- Windows (path normalization handled in `bin/install.js`)

## Version Management

**Current Version:**
- `1.20.3` (from `package.json`)

**Version Files:**
- `VERSION` file written to config directory on each install
- Update checking via `gsd-check-update.js` hook
  - Compares local VERSION against `npm view get-shit-done-cc version`
  - Caches result in `~/.claude/cache/gsd-update-check.json` (Claude) or equivalent per runtime
  - Runs background check at SessionStart (non-blocking)

**Changelog:**
- `CHANGELOG.md` (61KB) - Full release history and breaking changes
- Copied to `{configDir}/get-shit-done/CHANGELOG.md` on install

## Special Technical Notes

**Zero External Dependencies Philosophy:**
- Intentional design decision to avoid npm dependency bloat
- All file operations, path handling, and process management use Node.js built-ins
- Reduces attack surface and installation time

**Multi-Runtime Support:**
- Single codebase supports Claude Code, OpenCode, and Gemini CLI
- Frontmatter conversion (`convertClaudeToOpencodeFrontmatter()`, `convertClaudeToGeminiAgent()`)
- Tool name mapping for platform differences (AskUserQuestion → question in OpenCode)
- Config directory abstraction handles XDG Base Directory spec for OpenCode

**Local Patch Persistence:**
- SHA256 manifest system tracks user modifications to GSD files
- On update, modified files backed up to `gsd-local-patches/` with metadata
- Allows safe updates without losing user customizations
- Manifest stored as `gsd-file-manifest.json`

---

*Stack analysis: 2026-02-16*
