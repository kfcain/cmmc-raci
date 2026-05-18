---
name: init
description: Scaffold a new Information Security Responsibility Matrix for a client. Run this first to create the raci.json from the standard 8-role template.
---

# cmmc-raci:init — Initialize Client Responsibility Matrix

## Purpose
Creates a new `raci.json` data file for a client, pre-populated with 8 standard CMMC roles and default R/A/C/I assignments across all 35 infosec capability areas. The user can add or remove roles at initialization. After init, run `cmmc-raci:describe` to review and refine assignments interactively.

## Preconditions
None. This is the entry point.

## Reference Files
- `skills/init/references/raci-template.json` — 8 standard roles + all default assignments
- `skills/init/references/capability-areas.json` — 35 capability areas across 9 categories

## What to Do

### Step 1 — Get client name
Use AskUserQuestion (free text / "Other") to ask: "What is the organization's name for this responsibility matrix?"

Derive `client-slug`: lowercase the org name, replace spaces and special characters with hyphens, strip leading/trailing hyphens.

### Step 2 — Show default role roster
Display the 8 standard roles from `raci-template.json`:
1. CISO / Security Director
2. Information System Security Officer (ISSO)
3. Systems Administrator
4. Network Engineer
5. Cloud Administrator
6. HR Director
7. Legal / Contracts
8. IT Director / CIO

Ask: "Does this role roster fit the organization? You can drop roles that don't apply or note roles to add."
Accept free text — parse their answer to identify which roles to drop (exact match or close match on title/department).

### Step 3 — Build raci.json
1. Start from `raci-template.json`
2. Remove any roles the user said to drop
3. Remove all assignments referencing dropped role IDs
4. Set:
   - `id`: `raci-{client-slug}-{current year}`
   - `client`: org name as provided
   - `version`: "0.1.0"
   - `last_updated`: current ISO 8601 timestamp
   - `coverage_check.capabilities_without_R`: compute — any capability ID with no R assignment among remaining roles
   - `coverage_check.capabilities_without_A`: compute — any capability ID with no A assignment among remaining roles
   - `coverage_check.last_validated`: current ISO 8601 timestamp

### Step 4 — Create data store directory
Create `~/cmmc-raci-data/{client-slug}/` and `~/cmmc-raci-data/{client-slug}/exports/` if they do not already exist.

### Step 5 — Write raci.json
Write the completed `raci.json` to `~/cmmc-raci-data/{client-slug}/raci.json`.

### Step 6 — Confirm to user
Print:
```
Matrix scaffolded for {Org Name}
  Data: ~/cmmc-raci-data/{client-slug}/raci.json
  Roles: {N} roles, {M} assignments across 35 capability areas

Next step: run cmmc-raci:describe to review and refine assignments, or
           run cmmc-raci:view to generate the HTML viewer immediately.
```

## What This Skill Does NOT Do
- Does not ask about assignments (that is `cmmc-raci:describe`)
- Does not generate the viewer (that is `cmmc-raci:view`)
- Does not add person names/emails (that is done in `cmmc-raci:describe`)

## Notes for Claude
- If `raci.json` already exists for this client-slug, warn the user before overwriting
- The client-slug must be filesystem-safe: only lowercase letters, numbers, hyphens
- Always compute coverage_check at init time so the user knows immediately if dropping a role caused a gap
