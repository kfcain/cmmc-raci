---
name: export
description: Package the viewer.html and generate a Markdown RACI table with CMMC control cross-walk for SSP embedding or client deliverables.
---

# cmmc-raci:export — Export Deliverable Package

## Purpose
Verifies `viewer.html` is current, generates a Markdown RACI table grouped by capability category, appends a CMMC control cross-walk section, and writes `manifest.json` with file hashes and timestamps.

## Preconditions
- `~/cmmc-raci-data/{client-slug}/raci.json` must exist
- `~/cmmc-raci-data/{client-slug}/exports/viewer.html` should exist (if not, run `cmmc-raci:view` first)

## Reference Files
- `skills/export/references/markdown-table-template.md` — section header and table format reference
- `skills/init/references/capability-areas.json` — CMMC control mappings

## What to Do

### Step 1 — Verify viewer
Check that `exports/viewer.html` exists. If not, inform the user and offer to run `cmmc-raci:view` first.

### Step 2 — Build Markdown table
For each of the 9 capability categories, emit:

```markdown
## {Category Name}

| Capability Area | CISO | ISSO | SysAdmin | Network Eng | Cloud Admin | HR | Legal | IT Director |
|---|---|---|---|---|---|---|---|---|
| Endpoint Detection & Response (EDR) | A | C | R | — | C | — | — | I |
| Anti-Malware / Antivirus | A | C | R | — | — | — | — | I |
```

Rules:
- Rows = capabilities in this category
- Columns = roles in the order they appear in `raci.json`
- Cell value = R, A, C, or I. Empty = `—`
- Bold the R cells in Markdown: `**R**`

### Step 3 — Build CMMC cross-walk section
After the RACI table, append:

```markdown
---

## CMMC 800-171 Control Cross-Walk

| Capability Area | CMMC Controls |
|---|---|
| Endpoint Detection & Response (EDR) | 3.14.1, 3.14.2, 3.14.6, 3.14.7 |
| Anti-Malware / Antivirus | 3.14.2 |
...
```

Include all 35 capabilities sorted by category.

### Step 4 — Write Markdown file
Write to `~/cmmc-raci-data/{client-slug}/exports/raci-matrix.md`.

Add a header block at the top:
```markdown
# Information Security Responsibility Matrix
**Organization:** {Org Name}
**Version:** {version from raci.json}
**Last Updated:** {last_updated formatted as YYYY-MM-DD}

**Legend:** R = Responsible · A = Accountable · C = Consulted · I = Informed · — = Not Assigned
```

### Step 5 — Write manifest.json
Write `~/cmmc-raci-data/{client-slug}/exports/manifest.json`:
```json
{
  "client": "{Org Name}",
  "exported_at": "{ISO timestamp}",
  "files": [
    { "path": "exports/viewer.html", "purpose": "Interactive HTML responsibility matrix viewer" },
    { "path": "exports/raci-matrix.md", "purpose": "Markdown RACI table with CMMC cross-walk for SSP embedding" }
  ]
}
```

### Step 6 — Confirm to user
Print:
```
Export complete for {Org Name}

  exports/viewer.html       — interactive browser viewer
  exports/raci-matrix.md   — Markdown table + CMMC cross-walk (embed in SSP)
  exports/manifest.json    — file inventory

The Markdown table is ready to paste into an SSP or present to a C3PAO.
```

## What This Skill Does NOT Do
- Does not regenerate the viewer (run `cmmc-raci:view` to refresh it)
- Does not modify `raci.json`

## Notes for Claude
- Column order in the Markdown table must match the role order in `raci.json` for consistency
- The cross-walk section comes AFTER the RACI table, not before
- Do not include version numbers or dates in document footers (Workstreet brand rule)
