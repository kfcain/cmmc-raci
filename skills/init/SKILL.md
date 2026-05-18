---
name: init
description: Scaffold a new Information Security Responsibility Matrix for a client. Run this first to create the raci.json from the standard 8-role template.
---

# cmmc-raci:init — Initialize Client Responsibility Matrix

## Purpose
Creates a new `raci.json` data file for a client, pre-populated with 8 standard CMMC roles and default Primary/Backup assignments across all 40 infosec capability areas. The user can add or remove roles at initialization. After init, run `cmmc-raci:describe` to review and refine assignments interactively.

## Preconditions
None. This is the entry point.

## Reference Files
- `skills/init/references/raci-template.json` — 8 standard roles + all default assignments
- `skills/init/references/capability-areas.json` — 40 capability areas across 10 categories

## What to Do

### Step 1 — Get client name
Use AskUserQuestion (free text / "Other") to ask: "What is the organization's name for this responsibility matrix?"

Derive `client-slug`: lowercase the org name, replace spaces and special characters with hyphens, strip leading/trailing hyphens.

### Step 2 — Show default role roster
Display the 8 standard roles from `raci-template.json`:
1. Chief Information Security Officer (CISO)
2. Information System Security Officer (ISSO)
3. Systems Administrator
4. Network Engineer
5. Cloud Administrator
6. HR Director
7. Legal / Contracts
8. IT Director / CIO

Ask: "Does this role roster fit the organization? You can drop roles that don't apply or note any roles to add."

Accept free text — parse their answer to identify which roles to drop (match on title or common abbreviation). If the user wants to add roles, ask for the title and department for each new role, generate a kebab-case ID (e.g., `role-cto`), and append them to the roles array.

When a role with default assignments as primary or backup is dropped, reassign those capabilities to the closest remaining role (e.g., ISSO responsibilities shift to CISO; SysAdmin primary shifts to IT Director).

### Step 3 — Build raci.json
1. Start from `raci-template.json`
2. Apply role additions and removals per Step 2
3. Re-assign any orphaned primary/backup references to appropriate remaining roles
4. Set:
   - `schema_version`: "0.2.0"
   - `id`: `raci-{client-slug}-{current year}`
   - `client`: org name as provided
   - `version`: "0.1.0"
   - `last_updated`: current ISO 8601 timestamp

The assignments array has one record per capability (40 total), each with:
- `capability_id` — from capability-areas.json
- `primary_role_id` — role that owns execution of this capability
- `backup_role_id` — role that covers when primary is unavailable (null if no backup assigned)
- `tool_description` — null at init; filled during describe
- `access_methods` — array of "VPN", "ON-PREM", "VDI", "CLOUD"; inherited from template defaults
- `frequency` — inherited from template defaults
- `notes` — null

### Step 4 — Create data store directory
Create `~/cmmc-raci-data/{client-slug}/` and `~/cmmc-raci-data/{client-slug}/exports/` if they do not already exist.

### Step 5 — Write raci.json
Write the completed `raci.json` to `~/cmmc-raci-data/{client-slug}/raci.json`.

### Step 6 — Confirm to user
Print:
```
Matrix scaffolded for {Org Name}
  Data: ~/cmmc-raci-data/{client-slug}/raci.json
  Roles: {N} roles
  Capabilities: 40 across 10 categories — all have Primary assigned

Next step: run cmmc-raci:describe to refine assignments and add tool/access details, or
           run cmmc-raci:view to generate the HTML viewer immediately.
```

## What This Skill Does NOT Do
- Does not ask about individual assignments (that is `cmmc-raci:describe`)
- Does not generate the viewer (that is `cmmc-raci:view`)
- Does not add person names/emails (that is done in `cmmc-raci:describe`)

## Notes for Claude
- If `raci.json` already exists for this client-slug, warn the user before overwriting
- The client-slug must be filesystem-safe: only lowercase letters, numbers, hyphens
- When dropping a role, never leave a capability with no primary_role_id — always reassign
- New custom roles should get IDs prefixed with `role-` followed by a kebab-case slug
