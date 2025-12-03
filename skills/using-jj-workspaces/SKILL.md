---
name: using-jj-workspaces
description: Use when starting feature work that needs isolation from current workspace or before executing implementation plans - creates isolated jj workspaces with smart directory selection and safety verification
---

# Using Jujutsu Workspaces

## Overview

Jujutsu workspaces create isolated working copies sharing the same repository, allowing work on multiple features simultaneously without switching.

**Core principle:** Systematic directory selection + safety verification = reliable isolation.

**Announce at start:** "I'm using the using-jj-workspaces skill to set up an isolated workspace."

## Directory Selection Process

Follow this priority order:

### 1. Check Existing Directories

```bash
# Check in priority order
ls -d .workspaces 2>/dev/null     # Preferred (hidden)
ls -d workspaces 2>/dev/null      # Alternative
```

**If found:** Use that directory. If both exist, `.workspaces` wins.

### 2. Check CLAUDE.md

```bash
grep -i "workspace.*director" CLAUDE.md 2>/dev/null
```

**If preference specified:** Use it without asking.

### 3. Ask User

If no directory exists and no CLAUDE.md preference:

```
No workspace directory found. Where should I create workspaces?

1. .workspaces/ (project-local, hidden)
2. ~/.jj-workspaces/<project-name>/ (global location)

Which would you prefer?
```

## Safety Verification

### For Project-Local Directories (.workspaces or workspaces)

**MUST verify .gitignore before creating workspace:**

```bash
# Check if directory pattern in .gitignore
grep -q "^\.workspaces/$" .gitignore || grep -q "^workspaces/$" .gitignore
```

**If NOT in .gitignore:**

Per Matthew's rule "Fix broken things immediately":
1. Add appropriate line to .gitignore
2. Commit the change
3. Proceed with workspace creation

**Why critical:** Prevents accidentally committing workspace contents to repository.

### For Global Directory (~/.jj-workspaces)

No .gitignore verification needed - outside project entirely.

## Creation Steps

### 1. Detect Project Name

```bash
project=$(basename "$(jj workspace root)")
```

### 2. Create Workspace

```bash
# Determine full path
case $LOCATION in
  .workspaces|workspaces)
    path="$LOCATION/$WORKSPACE_NAME"
    ;;
  ~/.jj-workspaces/*)
    path="~/.jj-workspaces/$project/$WORKSPACE_NAME"
    ;;
esac

# Create workspace
jj workspace add "$path"
cd "$path"
```

### 3. Run Project Setup

Auto-detect and run appropriate setup:

```bash
# Node.js
if [ -f package.json ]; then npm install; fi

# Rust
if [ -f Cargo.toml ]; then cargo build; fi

# Python
if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
if [ -f pyproject.toml ]; then poetry install; fi

# Go
if [ -f go.mod ]; then go mod download; fi
```

### 4. Verify Clean Baseline

Run tests to ensure workspace starts clean:

```bash
# Examples - use project-appropriate command
npm test
cargo test
pytest
go test ./...
```

**If tests fail:** Report failures, ask whether to proceed or investigate.

**If tests pass:** Report ready.

### 5. Report Location

```
Workspace ready at <full-path>
Tests passing (<N> tests, 0 failures)
Ready to implement <feature-name>
```

## Quick Reference

| Situation | Action |
|-----------|--------|
| `.workspaces/` exists | Use it (verify .gitignore) |
| `workspaces/` exists | Use it (verify .gitignore) |
| Both exist | Use `.workspaces/` |
| Neither exists | Check CLAUDE.md → Ask user |
| Directory not in .gitignore | Add it immediately + commit |
| Tests fail during baseline | Report failures + ask |
| No package.json/Cargo.toml | Skip dependency install |

## Common Mistakes

**Skipping .gitignore verification**
- **Problem:** Workspace contents get tracked, pollute jj status
- **Fix:** Always grep .gitignore before creating project-local workspace

**Assuming directory location**
- **Problem:** Creates inconsistency, violates project conventions
- **Fix:** Follow priority: existing > CLAUDE.md > ask

**Proceeding with failing tests**
- **Problem:** Can't distinguish new bugs from pre-existing issues
- **Fix:** Report failures, get explicit permission to proceed

**Hardcoding setup commands**
- **Problem:** Breaks on projects using different tools
- **Fix:** Auto-detect from project files (package.json, etc.)

## Example Workflow

```
You: I'm using the using-jj-workspaces skill to set up an isolated workspace.

[Check .workspaces/ - exists]
[Verify .gitignore - contains .workspaces/]
[Create workspace: jj workspace add .workspaces/auth]
[Run npm install]
[Run npm test - 47 passing]

Workspace ready at /Users/matthew/myproject/.workspaces/auth
Tests passing (47 tests, 0 failures)
Ready to implement auth feature
```

## Red Flags

**Never:**
- Create workspace without .gitignore verification (project-local)
- Skip baseline test verification
- Proceed with failing tests without asking
- Assume directory location when ambiguous
- Skip CLAUDE.md check

**Always:**
- Follow directory priority: existing > CLAUDE.md > ask
- Verify .gitignore for project-local
- Auto-detect and run project setup
- Verify clean test baseline

## Integration

**Called by:**
- **brainstorming** (Phase 4) - REQUIRED when design is approved and implementation follows
- Any skill needing isolated workspace

**Pairs with:**
- **finishing-a-development-branch** - REQUIRED for cleanup after work complete
- **executing-plans** or **subagent-driven-development** - Work happens in this workspace
