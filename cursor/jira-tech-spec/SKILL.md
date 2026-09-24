---
name: jira-tech-spec
description: Drafts the Jira "Technical specification" field for the current change from the branch diff and pasted Jira text, using the team's fixed Why template. Use when a developer finishes a Jira, moves it to "dev completed", or asks to fill in the technical specification or release note for a ticket.
disable-model-invocation: true
---

# Jira Technical Specification

Produce paste-ready text for the Jira **Technical specification** field. The reader is a developer years from now who found this ticket via `git blame` and needs to know **why** the code is like this.

## Inputs

1. **Jira key** – from the user's message, otherwise from the branch name (`git branch --show-current`) or the latest commit message.
2. **Jira text** – summary, description, and the **Config** field, pasted by the user. If not provided, ask for it once. Do not try to fetch Jira; MCP is not available.
3. **Code change** – from git (see Step 1).

## Workflow

### Step 1: Read the change (keep it cheap)

Find the base branch: use the MR target branch if the user names it, otherwise the first that exists of `origin/develop`, `origin/main`, `origin/master`.

```bash
git log --oneline <base>..HEAD
git diff --stat <base>...HEAD
```

Then read the diff only for files with real logic changes. Skip generated files, lockfiles, formatting-only changes, and test fixtures unless they are the point of the change. Do not explore the wider codebase unless a changed line cannot be understood without one referenced definition.

### Step 2: Decide if this is a chore

If the change has **no behaviour change** (dependency bump, rename, formatting, logging text, test-only), output:

```
N/A – chore: <one-line reason, e.g. "bump library X to 2.3 for CVE; no behaviour change">
```

Stop there. Any change to runtime behaviour, config handling, client-specific logic, or data is **not** a chore.

### Step 3: Get the Why from the developer

The diff shows *what* changed, not *why*. If the Jira text does not clearly state the business reason, ask the developer **at most two short questions** in one message, then wait. Good questions:

- "What problem did this fix or what did the business ask for?"
- "Why this approach, e.g. why a config flag instead of changing the default?"
- "Which clients or flows does this affect?"

If the developer says they don't know, write `[DEV TO CONFIRM: …]` in that spot. **Never invent business intent.**

### Step 4: Write the specification

Use this exact template. Keep the whole thing under ~200 words. Plain text, `-` bullets, no tables, no code blocks.

```
Why this change exists
- <business reason / problem, in one or two lines>
- <why this approach, if non-obvious>

Behaviour before → after
- Before: <old behaviour>
- After: <new behaviour>

Config / client impact
- <config key(s) added/changed, default value, which clients enable/disable it and why>
- or: None – see Config field

Code pointers
- <main class/function/file and its role, 1–3 bullets>

Release note
<2–3 non-jargon sentences a product manager can reuse as is>
```

Rules:

- **Why** is the most important section. Write business language, not "added a null check".
- **Config / client impact**: do not repeat the Jira **Config** field word for word. Add only the intent (why a client uses X over Y) and anything the Config field is missing.
- **Code pointers**: name stable things (class, method, config key), not line numbers.
- **Release note**: no class names, no ticket jargon, describes the effect for users or ops.
- Leave out empty sections except **Config / client impact** (write `None – see Config field`).

### Step 5: Hand back

Output the finished text in one block the developer can copy, then one line:

"Review, edit anything marked DEV TO CONFIRM, paste into Technical specification, then move the ticket to dev completed."

Do not regenerate or polish unless asked.
