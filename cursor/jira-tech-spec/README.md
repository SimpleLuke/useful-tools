# jira-tech-spec (Cursor skill)

Cursor skill that drafts the Jira **Technical specification** field from your branch diff plus pasted Jira text, using a fixed team template (Why, behaviour, config impact, code pointers, release note).

## Install for the team

### Option A: Project skill (recommended for OMS repos)

Copy this folder into your application repo:

```text
your-oms-repo/
└── .cursor/
    └── skills/
        └── jira-tech-spec/
            └── SKILL.md
```

Commit `.cursor/skills/jira-tech-spec/SKILL.md`. Everyone who opens that repo in Cursor gets the skill.

### Option B: Personal skill (any repo)

```bash
mkdir -p ~/.cursor/skills/jira-tech-spec
cp SKILL.md ~/.cursor/skills/jira-tech-spec/SKILL.md
```

## When to use

- You are about to move a Jira to **dev completed** (or QA).
- **Technical specification** would otherwise be `N/A`.
- You want a short **release note** product can reuse.

Run it **once per ticket**, on your feature branch, with the MR diff still local.

## How to invoke

The skill is **not** auto-run (`disable-model-invocation: true`). In Cursor Agent chat, on your branch, type something like:

```text
/jira-tech-spec OMS-1234

Summary: Allow partial cancel for client ACME
Description: ACME needs to cancel remaining quantity on partially filled orders.
Config: order.partialCancel.enabled = true for ACME
```

You can omit the Jira key if it is in the branch name (e.g. `feature/OMS-1234-partial-cancel`) or in recent commits.

### What the agent does

1. Runs `git log` / `git diff --stat` against `develop`, `main`, or `master` (or the base branch you name).
2. Reads only the files that matter for the change.
3. Asks **at most two** short questions if the Jira does not explain **why** (the diff alone cannot).
4. Returns one block of text to paste into Jira.

### After you get the output

1. Read and edit (especially anything marked `[DEV TO CONFIRM: …]`).
2. Paste into Jira **Technical specification**.
3. Move the ticket to **dev completed**.

Use a cheaper/faster model for this task if your team has a choice—it is structured writing, not deep coding.

## Example (end to end)

### Setup

- Branch: `feature/OMS-1234-partial-cancel`
- MR targets `develop`
- Jira **Config** already filled for the fix

### Chat message

```text
/jira-tech-spec OMS-1234

Summary: Allow partial cancel for client ACME
Description: ACME needs to cancel remaining quantity on partially filled orders.
Config: order.partialCancel.enabled = true for ACME
```

### Agent may ask

```text
Two quick questions before I write it:
1. Why does ACME need partial cancel—is it compliance, trading workflow, or something else?
2. Why a per-client flag instead of enabling for everyone?
```

### You reply

```text
Their desk must release unfilled qty at market close per compliance. Other clients expect full cancel only, so opt-in.
```

### Agent output (paste into Jira)

```text
Why this change exists
- ACME's compliance policy requires releasing the unfilled quantity of partially filled orders at market close.
- Made opt-in per client because other clients' workflows expect cancel to apply to the whole order only.

Behaviour before → after
- Before: cancel on a partially filled order was rejected; only fully unfilled orders could be cancelled.
- After: when enabled for the client, cancel releases the remaining quantity and keeps the filled part booked.

Config / client impact
- order.partialCancel.enabled (default false). Enabled for ACME only, for the compliance reason above.
- Other clients: no change.

Code pointers
- OrderCancelService.cancel – partial-cancel branch behind the config flag
- CancelValidator – allows partially filled orders when the flag is on

Release note
Clients can now be set up to cancel the remaining quantity of a partially filled order while keeping what has already been filled. This is off by default and currently enabled for ACME.
```

Later, someone runs `git blame` on `OrderCancelService`, opens **OMS-1234**, and reads the Why without pinging the original author.

## Chore tickets (only valid `N/A`)

For dependency bumps, renames, formatting, or test-only work with **no behaviour change**:

```text
/jira-tech-spec OMS-1300

Summary: Bump library X for CVE-2026-1234
```

Expected output:

```text
N/A – chore: bump library X to 2.3 for security fix; no behaviour change
```

Anything that changes runtime behaviour, config handling, client logic, or data must use the full template—not `N/A`.

## Tips

| Tip | Why |
| --- | --- |
| Paste Jira **Summary**, **Description**, and **Config** | MCP/Jira fetch is often disabled; paste is reliable |
| Name the MR base branch if not `develop`/`main`/`master` | Diff scope stays correct |
| Answer the Why questions in one short reply | Stops invented business intent in the spec |
| Match Jira field help text to the same headings | Devs without Cursor can fill the field by hand |

## Files in this folder

| File | Purpose |
| --- | --- |
| `SKILL.md` | Skill definition (copy into `.cursor/skills/jira-tech-spec/`) |
| `README.md` | Team usage guide (this file) |
