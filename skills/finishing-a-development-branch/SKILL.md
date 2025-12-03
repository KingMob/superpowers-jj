---
name: finishing-a-development-branch
description: Use when implementation is complete, all tests pass, and you need to decide how to integrate the work - guides completion of development work by presenting structured options for merge, PR, or cleanup
---

# Finishing a Development Branch

## Overview

Guide completion of development work by presenting clear options and handling chosen workflow.

**Core principle:** Verify tests → Present options → Execute choice → Clean up.

**Announce at start:** "I'm using the finishing-a-development-branch skill to complete this work."

## The Process

### Step 1: Verify Tests

**Before presenting options, verify tests pass:**

```bash
# Run project's test suite
npm test / cargo test / pytest / go test ./...
```

**If tests fail:**
```
Tests failing (<N> failures). Must fix before completing:

[Show failures]

Cannot proceed with merge/PR until tests pass.
```

Stop. Don't proceed to Step 2.

**If tests pass:** Continue to Step 2.

### Step 2: Determine Base Change ID

```bash
# Find common ancestor change ID with main/master
jj log -r 'fork_point(bookmarks(regex:"main|master") | @)' -T 'change_id' --no-graph
```

Or ask: "This bookmark split from main - is that correct?"

### Step 3: Present Options

Present exactly these 4 options:

```
Implementation complete. What would you like to do?

1. Merge back to <base-bookmark> locally
2. Push and create a Pull Request
3. Keep the branch as-is (I'll handle it later)
4. Discard this work

Which option?
```

**Don't add explanation** - keep options concise.

### Step 4: Execute Choice

#### Option 1: Merge Locally

```bash
# Fetch latest
jj git fetch

# Create merge commit (use @ for current change, or @- if @ is empty)
jj new <base-bookmark> @

# Verify tests on merged result
<test command>

# If tests pass, move base bookmark to the merge and delete feature bookmark
jj bookmark move <base-bookmark> --to @
jj bookmark delete <feature-bookmark>
```

Then: Cleanup workspace (Step 5)

#### Option 2: Push and Create PR

```bash
# Create bookmark and push
jj bookmark create <bookmark-name>
jj git push --bookmark <bookmark-name>

# Create PR
gh pr create --title "<title>" --body "$(cat <<'EOF'
## Summary
<2-3 bullets of what changed>

## Test Plan
- [ ] <verification steps>
EOF
)"
```

Then: Cleanup workspace (Step 5)

#### Option 3: Keep As-Is

Report: "Keeping bookmark <name>. Workspace preserved at <path>."

**Don't cleanup workspace.**

#### Option 4: Discard

**Confirm first:**
```
This will permanently delete:
- Bookmark <name>
- All commits: <commit-list>
- Workspace at <path>

Type 'discard' to confirm.
```

Wait for exact confirmation.

If confirmed:
```bash
# Abandon the changes (actually removes them from history)
jj abandon @::
jj new <base-bookmark>
jj bookmark delete <feature-bookmark>
```

Then: Cleanup workspace (Step 5)

### Step 5: Cleanup Workspace

**For Options 1, 2, 4:**

Check if in workspace:
```bash
jj workspace list
```

If yes:
```bash
jj workspace forget <workspace-name>
rm -r <workspace-path>
```

**For Option 3:** Keep workspace.

## Quick Reference

| Option | Merge | Push | Keep Workspace | Cleanup Bookmark |
|--------|-------|------|----------------|------------------|
| 1. Merge locally | ✓ | - | - | ✓ |
| 2. Create PR | - | ✓ | ✓ | - |
| 3. Keep as-is | - | - | ✓ | - |
| 4. Discard | - | - | - | ✓ (force) |

## Common Mistakes

**Skipping test verification**
- **Problem:** Merge broken code, create failing PR
- **Fix:** Always verify tests before offering options

**Open-ended questions**
- **Problem:** "What should I do next?" → ambiguous
- **Fix:** Present exactly 4 structured options

**Automatic workspace cleanup**
- **Problem:** Remove workspace when might need it (Option 2, 3)
- **Fix:** Only cleanup for Options 1 and 4

**No confirmation for discard**
- **Problem:** Accidentally delete work
- **Fix:** Require typed "discard" confirmation

## Red Flags

**Never:**
- Proceed with failing tests
- Merge without verifying tests on result
- Delete work without confirmation
- Force-push without explicit request

**Always:**
- Verify tests before offering options
- Present exactly 4 options
- Get typed confirmation for Option 4
- Clean up workspace for Options 1 & 4 only

## Integration

**Called by:**
- **subagent-driven-development** (Step 7) - After all tasks complete
- **executing-plans** (Step 5) - After all batches complete

**Pairs with:**
- **using-jj-workspaces** - Cleans up workspace created by that skill
