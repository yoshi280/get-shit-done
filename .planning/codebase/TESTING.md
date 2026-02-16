# Testing Patterns

**Analysis Date:** 2026-02-16

## Test Framework

**Runner:**
- Node.js built-in `node:test` module (native test runner, no external dependency)
- Version: Node.js 18+ (uses `require('node:test')`)
- Config: No separate config file; tests run directly via `node gsd-tools.test.js`

**Assertion Library:**
- Node.js built-in `node:assert` module
- Methods: `assert.ok()`, `assert.strictEqual()`, `assert.deepStrictEqual()`
- No external testing framework (Jest, Vitest, Mocha, etc.) used

**Run Commands:**
```bash
node /Users/thelorax/.config/opencode/get-shit-done/bin/gsd-tools.test.js    # Run all tests
```

Note: No dedicated test runner script, watch mode, or coverage tools configured.

## Test File Organization

**Location:**
- Co-located with source: `gsd-tools.test.js` in same directory as `gsd-tools.js`
- Path: `/Users/thelorax/.config/opencode/get-shit-done/bin/gsd-tools.test.js`
- Test discovery: Manual (not automatic discovery)

**Naming:**
- Pattern: `[module].test.js` (e.g., `gsd-tools.test.js`)
- One test file per source file

**Structure:**
```
bin/
├── gsd-tools.js           # Main CLI tool (2286 lines)
└── gsd-tools.test.js      # Integration tests (1193 lines)
```

## Test Structure

**Suite Organization:**
```javascript
const { test, describe, beforeEach, afterEach } = require('node:test');
const assert = require('node:assert');

describe('history-digest command', () => {
  let tmpDir;

  beforeEach(() => {
    tmpDir = createTempProject();  // Setup per test
  });

  afterEach(() => {
    cleanup(tmpDir);               // Teardown per test
  });

  test('empty phases directory returns valid schema', () => {
    const result = runGsdTools('history-digest', tmpDir);
    assert.ok(result.success, `Command failed: ${result.error}`);
    // ... assertions
  });

  test('nested frontmatter fields extracted correctly', () => {
    // Arrange
    const phaseDir = path.join(tmpDir, '.planning', 'phases', '01-foundation');
    fs.mkdirSync(phaseDir, { recursive: true });
    fs.writeFileSync(path.join(phaseDir, '01-01-SUMMARY.md'), summaryContent);

    // Act
    const result = runGsdTools('history-digest', tmpDir);

    // Assert
    assert.ok(result.success, `Command failed: ${result.error}`);
    const digest = JSON.parse(result.output);
    assert.deepStrictEqual(digest.phases['01'].provides.sort(), [...]);
  });
});
```

**Patterns:**

1. **Describe blocks:**
   - One describe per command/feature
   - Examples: `'history-digest command'`, `'phases list command'`, `'phase next-decimal command'`
   - Total: 9+ describe blocks in test file

2. **Setup/Teardown:**
   ```javascript
   beforeEach(() => {
     tmpDir = createTempProject();  // Create temp dir with .planning/phases structure
   });

   afterEach(() => {
     cleanup(tmpDir);               // Delete temp dir recursively
   });
   ```

3. **Assertion Patterns:**
   - Success check first: `assert.ok(result.success, '...')`
   - Structural assertions: `assert.deepStrictEqual(obj, expected, 'message')`
   - Strict equality: `assert.strictEqual(count, 3, 'message')`
   - Existence checks: `assert.ok(digest.phases['01'], 'Phase 01 should exist')`
   - Array checks: `assert.ok(digest.decisions.some(d => d.decision === '...'))`

## Mocking

**Framework:**
- No mocking library (Jest mocks, Sinon, etc.)
- Manual mocking via file system setup in test fixtures
- Subprocess execution: `execSync()` runs actual CLI in isolation

**Patterns:**
```javascript
function runGsdTools(args, cwd = process.cwd()) {
  try {
    const result = execSync(`node "${TOOLS_PATH}" ${args}`, {
      cwd,
      encoding: 'utf-8',
      stdio: ['pipe', 'pipe', 'pipe'],
    });
    return { success: true, output: result.trim() };
  } catch (err) {
    return {
      success: false,
      output: err.stdout?.toString().trim() || '',
      error: err.stderr?.toString().trim() || err.message,
    };
  }
}
```

This approach:
- Executes actual CLI in subprocess with test directory
- Captures stdout/stderr
- Treats command as black box (integration-style testing)
- Allows testing real file I/O and git commands

**What to Mock:**
- Not applicable - no mocking framework used
- Instead: Create real temporary files/directories for isolation

**What NOT to Mock:**
- File system operations - tests use real fs
- Command execution - tests run actual CLI in subprocess
- Git operations - tests use real git commands (in subprocess)

## Fixtures and Factories

**Test Data:**
```javascript
function createTempProject() {
  const tmpDir = fs.mkdtempSync(path.join(require('os').tmpdir(), 'gsd-test-'));
  fs.mkdirSync(path.join(tmpDir, '.planning', 'phases'), { recursive: true });
  return tmpDir;
}

function cleanup(tmpDir) {
  fs.rmSync(tmpDir, { recursive: true, force: true });
}
```

**Dynamic fixture creation:**
```javascript
// Within test
const phaseDir = path.join(tmpDir, '.planning', 'phases', '01-foundation');
fs.mkdirSync(phaseDir, { recursive: true });

const summaryContent = `---
phase: "01"
name: "Foundation Setup"
dependency-graph:
  provides:
    - "Database schema"
tech-stack:
  added:
    - "prisma"
---

# Summary content here
`;

fs.writeFileSync(path.join(phaseDir, '01-01-SUMMARY.md'), summaryContent);
```

**Location:**
- Not in separate fixtures directory
- Inline in test functions via factory functions and string templates
- Temporary files created in OS tmpdir (prefixed `gsd-test-`)

## Coverage

**Requirements:**
- No coverage tool configured
- No coverage requirements enforced
- Coverage tracking: Not present

**View Coverage:**
- Not available - no coverage tool in use

## Test Types

**Unit Tests:**
- Scope: Individual command functions (e.g., history-digest, phases list)
- Approach: Integration-style - commands run in subprocess against real file system
- Tests filesystem isolation via temporary directories
- Does NOT test individual internal functions in isolation

**Integration Tests:**
- Actual scope: All tests are integration tests
- They exercise: CLI parsing, config loading, file I/O, YAML/frontmatter parsing, git interactions
- Subprocess isolation ensures no test interference
- Real workflow patterns tested (e.g., extracting nested YAML, handling malformed files)

**E2E Tests:**
- Framework: Not used (no Playwright, Cypress, etc.)
- CLI is the E2E boundary - tests exercise full CLI behavior

## Common Patterns

**Async Testing:**
- Not used - all tests are synchronous
- `execSync()` waits for subprocess completion synchronously

**Error Testing:**
```javascript
test('malformed SUMMARY.md skipped gracefully', () => {
  const phaseDir = path.join(tmpDir, '.planning', 'phases', '01-test');
  fs.mkdirSync(phaseDir, { recursive: true });

  // Write valid summary
  fs.writeFileSync(
    path.join(phaseDir, '01-01-SUMMARY.md'),
    `---\nphase: "01"\n---\nValid content`
  );

  // Write malformed summary
  fs.writeFileSync(
    path.join(phaseDir, '01-02-INVALID.md'),
    `Invalid frontmatter without closing ---`
  );

  // Command succeeds despite malformed file
  const result = runGsdTools('history-digest', tmpDir);
  assert.ok(result.success, `Command should succeed despite malformed files: ${result.error}`);

  // Malformed file is skipped
  const digest = JSON.parse(result.output);
  assert.ok(digest.phases['01'], 'Phase 01 should exist');
  // Only valid summary is processed
});
```

**Backward Compatibility Testing:**
```javascript
test('flat provides field still works (backward compatibility)', () => {
  // Test that old YAML format still works
  const summaryContent = `---
phase: "01"
provides:
  - "Old style array"
---
`;
  fs.writeFileSync(path.join(phaseDir, '01-01-SUMMARY.md'), summaryContent);

  const result = runGsdTools('history-digest', tmpDir);
  assert.ok(result.success);
  const digest = JSON.parse(result.output);
  assert.deepStrictEqual(digest.phases['01'].provides, ['Old style array']);
});
```

## Test Statistics

- **Total test suites:** 9 describe blocks
- **Total test cases:** ~50+ individual tests
- **Test file lines:** 1193 lines
- **Approximate test-to-code ratio:** ~1:2 (1193 test lines to 2286 source lines)
- **Coverage areas:**
  - Command parsing and dispatch
  - File system operations
  - YAML/frontmatter extraction (nested fields, backward compatibility)
  - Phase directory listing and sorting
  - State snapshot extraction
  - Summary extraction with field filtering

---

*Testing analysis: 2026-02-16*
