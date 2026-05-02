# repo-review

A comprehensive, read-only repository housekeeping skill. Produces a chat-only summary of findings across five dimensions — no report file, no automatic fixes.

## Who / When

- Anyone who wants to audit a repo for cleanliness and consistency
- Triggered by prompts like: "review this repo", "housekeeping", "audit repository", "check repo health"

## Workflow

Run these phases in order. Each phase can use parallel subagent tasks where sensible.

### Phase 1: Repo structure overview

1. Read all top-level files: `README*`, `AGENTS.md`, `CONTRIBUTING*`, `CODEOWNERS`, any root config files (`package.json`, `pyproject.toml`, `Cargo.toml`, `Makefile`, `go.mod`, `Gemfile`, etc.)
2. Enumerate the full directory tree (every file). Classify each file:
   - **Config** — build, lint, test, CI, git, editor
   - **Source** — application/library code
   - **Test** — test files/fixtures
   - **Doc** — markdown, docs, changelogs
   - **Asset** — images, fonts, binaries, large data
   - **Generated** — build artifacts, lockfiles, codegen output
   - **Unknown** — can't classify; flag for manual review
3. Note the real entrypoints, package boundaries, and monorepo structure if applicable.

### Phase 2: Git hygiene

1. Check if `.gitignore` exists. If missing, flag it.
2. Compare `.gitignore` rules against:
   - Common patterns missing for the detected tech stack (e.g., `node_modules/` for Node, `__pycache__/` for Python)
   - Files that are tracked but match an ignore pattern (should never happen; flag)
   - Untracked files that should be ignored (not covered by `.gitignore`)
3. Check for stalled/untracked files: temp files (`*.tmp`, `*.bak`, `*.orig`, `*~`, `.DS_Store`), editor swap files, OS metadata.
4. Look for large files (binary blobs, archives) that should be in `.gitignore` or managed with LFS.
5. If git repo: list untracked files, check for stale branches.

### Phase 3: Dead code / unused files

1. Flag files that appear to be one-off scripts, experiments, or leftovers:
   - Files named with patterns like `test-*`, `temp-*`, `scratch*`, `debug*`, `deprecated*`
   - Files with no imports/references from any other file in the repo
   - Duplicate or near-duplicate files
   - Orphaned config files pointing to non-existent tools
2. Check for dead symlinks.

### Phase 4: Config consistency

1. Find all config files for lint, test, build, formatter, typecheck, and codegen.
2. Cross-reference:
   - Do the tools configured in CI match local config?
   - Are there configs for tools not installed (orphaned config)?
   - Are there tools installed without config?
   - Do scripts in `package.json` / `Makefile` / task runner reference commands that exist?
3. Flag missing or broken toolchain links.

### Phase 5: Documentation health

1. Check README for:
   - Setup instructions match actual config
   - Commands shown work (static check — don't execute)
2. Check AGENTS.md / CLAUDE.md / `.cursor/rules/` / copilot instructions:
   - Commands documented match real config
   - No stale references to removed files or tools
3. Flag missing documentation (no README, no AGENTS.md, no CONTRIBUTING).

### Phase 6: Dependency audit

1. Identify all dependency manifests (`package.json`, `requirements.txt`, `Cargo.toml`, `go.mod`, etc.).
2. For each:
   - Flag unused dependencies (declared but never imported).
   - Flag missing dependencies (imported but not declared).
   - Flag outdated lockfiles (manifest newer than lockfile, or no lockfile).
   - Flag version pinning issues (loose ranges, git SHAs without hashes).
3. Note: dependency checking is static only — grep imports vs declarations.

## Output format

Stream findings directly in chat. No report file is written.

Structure the output as:

```
## Repo Review — <repo-name>

### Overview
- <N> files, <M> directories
- Detected tech stack: <languages/frameworks>
- Entrypoints: <list>

### Git Hygiene
- <finding> ⚠️
- <finding> ✅

### Dead Code / Unused Files
- <finding>

### Config Consistency
- <finding>

### Documentation Health
- <finding>

### Dependency Audit
- <finding>
```

Use severity markers:
- `❌` Critical / broken
- `⚠️` Warning / should fix
- `💡` Suggestion / nice to have
- `✅` Good / no issues

## Edge cases

- **Empty repo**: Report "nothing to review" and stop.
- **No `.gitignore`**: Flag it; still do full review on untracked files.
- **Monorepo**: Treat each package as a sub-unit for dependency audit; flag package boundary issues.
- **Very large repos**: Sample rather than exhaustively scanning every leaf — note when sampling was used.
- **No config files**: Skip Config Consistency phase but note it.
- **Binary / large assets**: Flag in Git Hygiene but don't attempt to diff or analyze contents.
- **Symlinks**: Follow and verify target exists; flag dead ones.
- **Not a git repo**: Skip git-specific checks (`.gitignore`, branches, tracked/untracked) but still do dead code, config, docs, and dependency phases.

## Anti-patterns (do NOT do)

- Do NOT run `npm install`, `pip install`, or any dependency installation.
- Do NOT execute lint, test, build, or typecheck commands.
- Do NOT modify any files.
- Do NOT write a REVIEW.md or any report file unless explicitly asked.
- Do NOT commit anything.
