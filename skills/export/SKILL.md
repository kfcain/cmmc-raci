---
name: export
description: Package the viewer.html and generate a Markdown responsibility table with CMMC control cross-walk for SSP embedding or client deliverables.
---

# cmmc-raci:export — Export Deliverable Package

## Purpose
Verifies `viewer.html` is current, generates a Markdown responsibility table grouped by capability category (Primary/Backup model), appends a CMMC control cross-walk section, and writes `manifest.json`.

## Preconditions
- `~/cmmc-raci-data/{client-slug}/raci.json` must exist
- `~/cmmc-raci-data/{client-slug}/exports/viewer.html` should exist (if not, run `cmmc-raci:view` first)

## Reference Files
- `skills/export/references/markdown-table-template.md` — section and table format reference
- `skills/init/references/capability-areas.json` — CMMC control mappings

## What to Do

### Step 1 — Verify viewer
Check that `exports/viewer.html` exists. If not, inform the user and offer to run `cmmc-raci:view` first.

### Step 2 — Build Markdown responsibility table
For each of the 10 capability categories, emit:

```markdown
## Endpoint Security

| Capability Area | Primary | Backup | Access Methods | Frequency |
|---|---|---|---|---|
| Endpoint Detection & Response (EDR) | Systems Administrator | ISSO | ON-PREM, VPN | Daily |
| Anti-Malware / Antivirus | Systems Administrator | ISSO | ON-PREM, VPN | Weekly |
| Mobile Device Management (MDM) | Systems Administrator | ISSO | CLOUD, VPN | Weekly |
| Device Encryption | Systems Administrator | ISSO | ON-PREM, VPN | Monthly |
```

Rules:
- Rows = capabilities in this category (40 total across 10 categories)
- Primary / Backup columns show the role title (or person_name if set in raci.json)
- Access Methods = comma-joined list of methods (e.g., "ON-PREM, VPN")
- Frequency = the value from raci.json
- If backup_role_id is null, Backup cell = `—`
- If access_methods is empty, Access Methods cell = `—`

### Step 3 — Build CMMC cross-walk section
After the responsibility table, append:

```markdown
---

## CMMC 800-171 Control Cross-Walk

| Capability Area | CMMC Controls |
|---|---|
| Endpoint Detection & Response (EDR) | 3.14.1, 3.14.2, 3.14.6, 3.14.7 |
| Anti-Malware / Antivirus | 3.14.2 |
...
```

Include all 40 capabilities grouped by category.

### Step 4 — Write Markdown file
Write to `~/cmmc-raci-data/{client-slug}/exports/raci-matrix.md`.

Add a header block at the top:
```markdown
# Information Security Responsibility Matrix
**Organization:** {Org Name}
**Last Updated:** {last_updated from raci.json, formatted as YYYY-MM-DD}
```

### Step 5 — Write manifest.json
Write `~/cmmc-raci-data/{client-slug}/exports/manifest.json`:
```json
{
  "client": "{Org Name}",
  "exported_at": "{ISO timestamp}",
  "files": [
    { "path": "exports/viewer.html", "purpose": "Interactive HTML responsibility matrix viewer" },
    { "path": "exports/raci-matrix.md", "purpose": "Markdown responsibility table with CMMC cross-walk for SSP embedding" }
  ]
}
```

### Step 6 — Confirm to user
```
Export complete for {Org Name}

  exports/viewer.html       — interactive browser viewer (3 tabs)
  exports/raci-matrix.md   — Markdown table + CMMC cross-walk (embed in SSP)
  exports/manifest.json    — file inventory

The Markdown table is ready to paste into an SSP or present to a C3PAO.
```

## What This Skill Does NOT Do
- Does not regenerate the viewer (run `cmmc-raci:view` to refresh)
- Does not modify `raci.json`

## Notes for Claude
- Use role title (not ID) in the table; use person_name if it is set, falling back to role title
- The cross-walk section comes AFTER the responsibility table, not before
- Do not include version numbers in document footers (Workstreet brand rule)
