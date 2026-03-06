---
description: Send code changes from the current session to OpenAI Codex CLI for review. Codex reviews the diff for bugs, security, performance, and quality.
---

# Codex Code Review (Iterative)

Review code changes made in the current Claude Code session by sending them to OpenAI Codex. Claude and Codex iterate until the code passes review. Max 5 rounds.

---

## When to Invoke

- When the user runs `/codex-review:code` after making code changes
- When the user wants Codex to review their implementation before committing

## Agent Instructions

When invoked, perform the following:

### Step 1: Generate Session ID

```bash
REVIEW_ID=$(uuidgen | tr '[:upper:]' '[:lower:]' | head -c 8)
```

Use this for all temp file paths: `/tmp/claude-code-${REVIEW_ID}.md` and `/tmp/codex-code-review-${REVIEW_ID}.md`.

### Step 2: Gather Changed Files

Identify all files changed in the current session:

1. Run `git diff` (unstaged) and `git diff --cached` (staged) to get the full diff of changes.
2. Run `git diff --name-only` and `git diff --cached --name-only` to get the list of changed files.
3. Also check `git status` for any new untracked files that were created in this session.

If there are no changes detected, ask the user which files or changes they want reviewed.

### Step 3: Build Review Context

Write a review document to `/tmp/claude-code-${REVIEW_ID}.md` containing:

```markdown
# Code Review Request

## Changed Files
- [list of changed files with brief description of what each does]

## Intent
[Summarize the purpose of these changes based on the conversation context -- what was the user trying to accomplish?]

## Diff
[Full git diff output]

## New Files (if any)
[Full content of any newly created files not captured in the diff]
```

### Step 4: Initial Review (Round 1)

Run Codex CLI in non-interactive mode:

```bash
codex exec \
  -m gpt-5.3-codex \
  -s read-only \
  -o /tmp/codex-code-review-${REVIEW_ID}.md \
  "Review the code changes described in /tmp/claude-code-${REVIEW_ID}.md. Focus on:
1. Bugs - Logic errors, off-by-one, null/undefined issues, race conditions
2. Security - Injection, auth issues, secrets exposure, OWASP top 10
3. Performance - N+1 queries, unnecessary allocations, missing indexes
4. Error handling - Uncaught exceptions, missing validation, silent failures
5. Readability - Unclear naming, overly complex logic, missing context

For each issue found, specify:
- File and line number (or range)
- Severity: CRITICAL / WARNING / SUGGESTION
- What the problem is
- How to fix it

If the code is solid and ready to ship, end with: VERDICT: APPROVED
If changes are needed, end with: VERDICT: REVISE"
```

**Capture the Codex session ID** from the output line that says `session id: <uuid>`. Store this as `CODEX_SESSION_ID`.

**Notes:**

- Use `-m gpt-5.3-codex` as the default model. If the user specifies a different model (e.g., `/codex-review:code o4-mini`), use that instead.
- Use `-s read-only` so Codex can read the codebase for context but cannot modify anything.
- Use `-o` to capture the output to a file for reliable reading.
- Do NOT pipe the command through `tail` or any other filter — let the full output be visible so the user can see Codex's progress.
- Set a timeout of at least 5 minutes (300000ms) for the Bash tool call, as Codex reviews can take a while.

### Step 5: Read Review & Check Verdict

1. Read `/tmp/codex-code-review-${REVIEW_ID}.md`
2. Present Codex's review to the user:

```
## Codex Code Review -- Round N (model: gpt-5.3-codex)

[Codex's feedback here, organized by severity]
```

3. Check the verdict:
   - If **VERDICT: APPROVED** -> go to Step 8 (Done)
   - If **VERDICT: REVISE** -> go to Step 6 (Fix & Re-submit)
   - If no clear verdict but no actionable issues -> treat as approved
   - If max rounds (5) reached -> go to Step 8 with remaining concerns noted

### Step 6: Fix the Code

Based on Codex's feedback:

1. **Apply fixes** -- address each CRITICAL and WARNING issue Codex raised. Actually edit the files to fix the problems.
2. For SUGGESTION items, apply them if they're clearly beneficial; otherwise note them for the user.
3. **Summarize** what you fixed:

```
### Fixes Applied (Round N)
- [File:line] [What was fixed and why]
```

4. Inform the user: "Sending updated code back to Codex for re-review..."

### Step 7: Re-submit to Codex (Rounds 2-5)

Update the review document with the new diff:

1. Regenerate the diff (`git diff` + `git diff --cached`) and update `/tmp/claude-code-${REVIEW_ID}.md`

Resume the existing Codex session:

```bash
codex exec resume ${CODEX_SESSION_ID} \
  "I've fixed the code based on your feedback. The updated diff is in /tmp/claude-code-${REVIEW_ID}.md.

Here's what I changed:
[List the specific fixes applied]

Please re-review. If the code is now solid and ready to ship, end with: VERDICT: APPROVED
If more changes are needed, end with: VERDICT: REVISE"
```

**Note:** `codex exec resume` does NOT support `-o` flag. Read the Codex response directly from stdout. Do NOT pipe through `tail` or any filter — show full output. Set a timeout of at least 5 minutes (300000ms).

Then go back to **Step 5** (Read Review & Check Verdict).

**Important:** If `resume ${CODEX_SESSION_ID}` fails, fall back to a fresh `codex exec` call with context about prior rounds.

### Step 8: Present Final Result

Once approved (or max rounds reached):

```
## Codex Code Review -- Final (model: gpt-5.3-codex)

**Status:** Approved after N round(s)

**Files reviewed:**
- [list of files]

**Issues found and fixed:** X critical, Y warnings, Z suggestions

[Final Codex feedback]

---
**Code has been reviewed and approved by Codex. Ready to commit.**
```

If max rounds were reached without approval:

```
## Codex Code Review -- Final (model: gpt-5.3-codex)

**Status:** Max rounds (5) reached -- not fully approved

**Remaining concerns:**
[List unresolved issues with file:line references]

---
**Codex still has concerns. Review the remaining items and decide whether to proceed.**
```

### Step 9: Cleanup

```bash
rm -f /tmp/claude-code-${REVIEW_ID}.md /tmp/codex-code-review-${REVIEW_ID}.md
```

## Rules

- Claude **actively fixes the code** based on Codex feedback between rounds -- not just relaying messages
- Default model is `gpt-5.3-codex`. Accept model override from user arguments (e.g., `/codex-review:code o4-mini`)
- Always use read-only sandbox mode for Codex -- Codex reviews but never writes files
- Max 5 review rounds to prevent infinite loops
- Show the user each round's feedback and fixes so they can follow along
- Prioritize CRITICAL issues over WARNINGs over SUGGESTIONs
- If Codex CLI is not installed or fails, inform the user and suggest `npm install -g @openai/codex`
- If a fix would contradict the user's explicit requirements, skip it and note it for the user
- Do NOT commit changes -- leave that to the user
