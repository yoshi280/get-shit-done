# Coding Conventions

**Analysis Date:** 2026-02-16

## Naming Patterns

**Files:**
- Scripts: PascalCase with descriptive names followed by optional suffix, e.g., `gsd-tools.js`, `gsd-tools.test.js`
- Test files: Named identically to source with `.test.js` suffix: `gsd-tools.test.js` for `gsd-tools.js`

**Functions:**
- Command handlers: `cmd` prefix followed by PascalCase, e.g., `cmdGenerateSlug()`, `cmdHistoryDigest()`, `cmdStateLoad()`
- Helper/utility functions: camelCase without prefix, e.g., `parseIncludeFlag()`, `safeReadFile()`, `loadConfig()`
- Internal implementations: `*Internal` suffix for private versions, e.g., `resolveModelInternal()`, `findPhaseInternal()`
- Command dispatching: Nested `subcommand` pattern for complex commands (state, phases, phase, init workflows)

**Variables:**
- camelCase for local variables and parameters: `configPath`, `phaseDir`, `tmpDir`, `summaryContent`
- Constants (non-mutable): UPPER_SNAKE_CASE, e.g., `MODEL_PROFILES`, `VALID_PROFILES`, `TOOLS_PATH`
- Loop variables: Single letter preferred (`i`, `f`) or short descriptive (`dir`)
- Temporary/working variables: Short names appropriate to scope (`obj`, `key`, `value`, `result`)
- Booleans: Prefix with `is` or verb form: `exists`, `success`, `raw`, `recursive`

**Types/Interfaces:**
- Object destructuring patterns: camelCase keys matching JSON structure (e.g., `update_available`, `installed`, `latest` in result objects)
- Return objects: Properties named with snake_case in some contexts (legacy config parsing), camelCase in modern code
- Schema fields: Flexible approach - snake_case for compatibility with YAML/JSON files, camelCase in JS

## Code Style

**Formatting:**
- No formal linter configured in codebase (no eslint/prettier config)
- Semicolons: Present at end of statements
- Indentation: 2 spaces (observed consistently)
- Line length: No enforced limit, varies by context (up to ~100 chars typical)

**Linting:**
- No ESLint or Prettier configuration detected
- Code follows manual conventions without automated enforcement

## Import Organization

**Order:**
1. Core Node.js modules: `const fs = require('fs');`, `const path = require('path');`
2. Built-in utilities: `const { execSync } = require('child_process');`, `const { spawn } = require('child_process');`
3. Node test framework: `const { test, describe, beforeEach, afterEach } = require('node:test');`
4. Standard assertions: `const assert = require('node:assert');`
5. Local utilities: `const os = require('os');` (standard library)
6. Constants/config: Model profiles, valid values
7. No imports from external packages detected (fully self-contained)

**Path Aliases:**
- Not used - imports are direct paths

## Error Handling

**Patterns:**
- Try-catch blocks around risky operations: file reads, git commands, JSON parsing
- Silent failure pattern: `catch` blocks without re-throw used when graceful degradation acceptable
- Informative catch: Some catches log to console or return error object with details
- Error output: Use `error(message)` function that writes to stderr and exits
- Return objects: Most functions return `{ success: boolean, [fields]: value }` or `{ error: string }` patterns
- Exit codes: Program calls `process.exit()` only in `error()` helper function

**Example patterns:**
```javascript
try {
  const raw = fs.readFileSync(configPath, 'utf-8');
  const parsed = JSON.parse(raw);
  // process parsed
} catch {
  return defaults;  // Silent fallback
}

try {
  const stats = fs.statSync(fullPath);
  const type = stats.isDirectory() ? 'directory' : 'file';
  return { exists: true, type };
} catch {
  return { exists: false, type: null };
}
```

## Logging

**Framework:** Plain `console` object (Node.js built-in)

**Patterns:**
- Logging used minimally - mostly within hooks and background processes
- Example: `gsd-check-update.js` uses no logging, relies on file writes for state
- Errors written to stderr via `process.stderr.write()` in `error()` function
- Output to stdout via `output()` function for command results
- No structured logging, no logger framework

## Comments

**When to Comment:**
- Function purpose: Brief JSDoc-style comment blocks above functions, e.g., `// Check for GSD updates in background`
- Complex logic: Comments explain YAML parsing stacks, frontmatter extraction, nested structures
- Configuration note: e.g., "VERSION file locations (check project first, then global)"
- Assumptions: `// Store path must be relative or absolute; if relative, join with cwd`

**JSDoc/TSDoc:**
- Not systematically used
- Basic doc comments on files (header comment explains purpose)
- No parameter or return type documentation

## Function Design

**Size:**
- Range: 20-150 lines typical
- Longer functions (>100 lines) for complex parsing (YAML extraction, frontmatter processing)
- Helper functions kept short (20-50 lines)

**Parameters:**
- Typically 1-4 parameters
- Common pattern: `(cwd, arg1, arg2, raw)` where `raw` is a boolean flag for output format
- Object parameters rare; positional parameters preferred
- Path-related functions require `cwd` as first parameter for reproducibility

**Return Values:**
- Object returns most common: `{ success, output }` or `{ field: value }`
- Void functions that call `output()` or `error()` to exit
- Error cases: Either throw (rare), return error object, or return defaults (silent)
- JSON output when `--raw` flag present; wrapped in output object otherwise

## Module Design

**Exports:**
- Main script: Single invocation via `main()` function
- No module.exports; scripts are executable CLI tools (shebang: `#!/usr/bin/env node`)
- All functions defined at module scope and called from main dispatcher

**Barrel Files:**
- Not applicable - monolithic CLI tool file structure

**Command Dispatcher Pattern:**
```javascript
function main() {
  const command = args[0];

  switch (command) {
    case 'command-name': {
      cmdCommandName(cwd, args[1], raw);
      break;
    }
    // ... more cases
    default:
      error(`Unknown command: ${command}`);
  }
}
```

This pattern is used consistently for:
- Top-level commands (state, phases, init, etc.)
- Subcommands (state load/get/patch, phases list, phase next-decimal, etc.)
- Initialization workflows (execute-phase, plan-phase, new-project, etc.)

---

*Convention analysis: 2026-02-16*
