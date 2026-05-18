---
name: describe
description: Interactively review and refine R/A/C/I assignments category by category. Also collects person names and emails for each role. Run after cmmc-raci:init.
---

# cmmc-raci:describe — Interactive Assignment Review

## Purpose
Walks through all 9 capability categories with the user, confirming or adjusting R/A/C/I assignments for each. Optionally collects person names and contact info per role. Ends with a coverage check ensuring every capability has at least one R and one A.

## Preconditions
`~/cmmc-raci-data/{client-slug}/raci.json` must exist (run `cmmc-raci:init` first).

## Reference Files
- `skills/describe/references/family-guidance.json` — context text for each capability
- `skills/init/references/capability-areas.json` — canonical capability list

## What to Do

### Step 1 — Identify client
Ask: "Which client's matrix are you reviewing?" Show a list of directories found under `~/cmmc-raci-data/` if multiple exist.

Read `~/cmmc-raci-data/{client-slug}/raci.json` and `capability-areas.json`.

### Step 2 — Category-by-category review
For each of the 9 categories (endpoint, identity, vuln, network, cloud, data, monitoring, ir, governance, physical):

1. Display the current assignments for that category as a compact table:
   ```
   ENDPOINT SECURITY
   ─────────────────────────────────────────────────────────────────
   Capability                     CISO  ISSO  SysAdm  NetEng  Cloud  HR  Legal  ITDir
   Endpoint Detection & Response   A     C     R       ─       C     ─    ─     I
   Anti-Malware / Antivirus        A     C     R       ─       ─     ─    ─     I
   Mobile Device Management        A     C     R       ─       ─     I    ─     I
   Device Encryption               A     R     R       ─       C     ─    ─     I
   ```

2. Use AskUserQuestion: "Does the **{Category Name}** section look right, or do you need to make changes?"
   Options: "Looks good", "Make changes"

3. If "Make changes": ask them to describe what to change in plain text (e.g. "Move EDR responsible from SysAdmin to Cloud Admin" or "Add Legal as Consulted for Device Encryption"). Apply the changes to `raci.json` and redisplay the updated table, then ask again.

4. After user confirms a category, write the updated assignments to `raci.json` before moving to the next category.

### Step 3 — Collect person info (optional)
After all categories are confirmed, ask: "Would you like to add person names and contact info to each role? This appears in the viewer's role cards and inspector panel."

If yes: for each role, ask for person_name, person_email, and person_title. Use AskUserQuestion in a loop. Write updates to `raci.json` after each role.

If no: skip.

### Step 4 — Coverage check
Compute:
- `capabilities_without_R`: capability IDs where no assignment has raci="R"
- `capabilities_without_A`: capability IDs where no assignment has raci="A"

Update `coverage_check` in `raci.json` with current timestamp.

### Step 5 — Summary
Print:
```
Review complete for {Org Name}

Capabilities reviewed: 35
Roles: {N}
Total assignments: {M}

Coverage:
  ✓ All 35 capabilities have at least one Responsible (R)
  ✓ All 35 capabilities have at least one Accountable (A)

[OR if gaps:]
  ⚠ {N} capabilities have no Responsible (R): {list}
  ⚠ {N} capabilities have no Accountable (A): {list}

Next step: run cmmc-raci:view to generate the HTML viewer.
```

## What This Skill Does NOT Do
- Does not generate the viewer (that is `cmmc-raci:view`)
- Does not add new capability areas (those are defined in capability-areas.json)

## Notes for Claude
- Always show the table before asking — never make the user imagine the current state
- Work category by category, not capability by capability — the user should only drill down if they say "make changes"
- After every write to `raci.json`, update `last_updated` timestamp
- When applying plain-text changes, confirm your interpretation before writing
