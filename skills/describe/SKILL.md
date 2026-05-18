---
name: describe
description: Interactively review and refine Primary/Backup assignments category by category, and collect tool descriptions, access methods, frequency, and person contact info. Run after cmmc-raci:init.
---

# cmmc-raci:describe — Interactive Assignment Review

## Purpose
Walks through all 10 capability categories with the user, confirming or adjusting Primary/Backup assignments, tool descriptions, access methods, and frequency for each. Optionally collects person names and contact info per role. Every capability must end with a Primary assigned.

## Preconditions
`~/cmmc-raci-data/{client-slug}/raci.json` must exist (run `cmmc-raci:init` first).

## Reference Files
- `skills/describe/references/family-guidance.json` — 1-2 sentence context per capability (for the inspector panel)
- `skills/init/references/capability-areas.json` — canonical capability list

## What to Do

### Step 1 — Identify client
Ask: "Which client's matrix are you reviewing?" Show a list of directories found under `~/cmmc-raci-data/` if multiple exist.

Read `~/cmmc-raci-data/{client-slug}/raci.json` and `capability-areas.json`.

Build a role lookup map: `{ roleId → role }` for display.

### Step 2 — Category-by-category review
For each of the 10 categories (endpoint, identity, vuln, network, cloud, data, monitoring, ir, governance, physical):

1. Display the current assignments for that category as a compact table:

   ```
   ENDPOINT SECURITY
   ────────────────────────────────────────────────────────────────────────
   Capability                     Primary          Backup           Freq
   Endpoint Detection & Response  SysAdmin         ISSO             Daily
   Anti-Malware / Antivirus       SysAdmin         ISSO             Weekly
   Mobile Device Management       SysAdmin         ISSO             Weekly
   Device Encryption              SysAdmin         ISSO             Monthly
   ```

   Show abbreviated role names (first word of title or person_name if set).

2. Use AskUserQuestion: "Does the **{Category Name}** section look right?"
   Options: "Looks good", "Make changes"

3. If "Make changes": ask them to describe what to change in plain text — for example:
   - "Move EDR primary to Cloud Admin"
   - "Add backup for Device Encryption — Network Engineer"
   - "Change patch management frequency to Monthly"
   - "Add tool description for SIEM: Microsoft Sentinel"
   Apply the changes to the assignments in memory, redisplay the updated table, then ask again.

4. After the user confirms a category, write the updated assignments to `raci.json` before moving on.

### Step 3 — Access methods review (optional, per-capability)
After categories are confirmed, ask: "Would you like to review access methods for any capabilities? (e.g., flag which use VPN, CLOUD, ON-PREM, or VDI)"

If yes: present capabilities that currently have no access methods set (empty array) or where the default may not fit. Accept free-text adjustments. Valid values: "VPN", "ON-PREM", "VDI", "CLOUD".

If no: skip.

### Step 4 — Collect person info (optional)
Ask: "Would you like to add person names and contact info to each role? This shows in the viewer's By Person cards and Personnel Directory."

If yes: for each role, collect `person_name`, `person_email` (optional), and `person_title` (optional). Write to `raci.json` after each role entry.

If no: skip.

### Step 5 — Coverage check and summary
Verify every capability has a non-null `primary_role_id`. List any gaps.

Update `last_updated` timestamp in `raci.json`.

Print:
```
Review complete for {Org Name}

Capabilities reviewed: 40 (10 categories)
Roles: {N}

Coverage:
  ✓ All 40 capabilities have a Primary assigned
  [OR: ⚠ {N} capabilities missing Primary: {list}]

Capabilities with tool descriptions: {count} / 40

Next step: run cmmc-raci:view to generate the HTML viewer.
```

## What This Skill Does NOT Do
- Does not generate the viewer (that is `cmmc-raci:view`)
- Does not add new capability areas (those are defined in capability-areas.json)

## Notes for Claude
- Always show the table first — never make the user reconstruct the current state in their head
- Work category by category; only drill into individual capabilities when the user says "make changes"
- After every write to `raci.json`, update `last_updated` timestamp
- Confirm your interpretation of plain-text changes before writing (e.g., "Setting EDR primary to role-cloudadmin — correct?")
- Access method values are strictly: "VPN", "ON-PREM", "VDI", "CLOUD" — reject any other values
- Frequency values are strictly: "Daily", "Weekly", "Monthly", "Quarterly", "Annually", "As Needed", "24/7 On-Call"
