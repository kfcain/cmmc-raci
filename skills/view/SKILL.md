---
name: view
description: Generate the interactive HTML responsibility matrix viewer from raci.json. Opens in any browser with no server required.
---

# cmmc-raci:view — Generate Interactive HTML Viewer

## Purpose
Reads `raci.json` and injects its data into `viewer.template.html` to produce a self-contained, interactive HTML file. The viewer shows a three-tab layout: Responsibility Matrix (category filter chips + Primary/Backup table), By Person (role cards with counts and responsibility lists), and Personnel Directory.

## Preconditions
`~/cmmc-raci-data/{client-slug}/raci.json` must exist.

## Reference Files
- `skills/view/viewer.template.html` — the HTML template to inject data into
- `skills/init/references/capability-areas.json` — capability area details (names, descriptions, CMMC controls)

## What to Do

### Step 1 — Identify client
If not already known from context, ask which client. Read `~/cmmc-raci-data/{client-slug}/raci.json` and `skills/init/references/capability-areas.json`.

### Step 2 — Build the D object
Construct the following JavaScript object (valid JS, not JSON-stringified):

```javascript
const D = {
  client: "<org name from raci.json>",
  version: "<version from raci.json>",
  last_updated: "<ISO timestamp from raci.json>",
  roles: [
    {
      id: "role-ciso",
      title: "Chief Information Security Officer (CISO)",
      department: "Executive / Security",
      person_name: null,
      person_email: null,
      person_title: null
    }
    // ... all roles from raci.json, in order
  ],
  categories: [
    {
      id: "endpoint",
      name: "Endpoint Security",
      capabilities: [
        {
          id: "cap-edr",
          name: "Endpoint Detection & Response (EDR)",
          description: "Deployment, configuration, and ongoing monitoring of EDR tooling...",
          cmmc_controls: ["3.14.1", "3.14.2", "3.14.6", "3.14.7"]
        }
        // ... all capabilities in this category from capability-areas.json
      ]
    }
    // ... all 10 categories, preserving order from capability-areas.json
  ],
  assignments: [
    {
      capability_id: "cap-edr",
      primary_role_id: "role-sysadmin",
      backup_role_id: "role-isso",
      tool_description: null,
      access_methods: ["ON-PREM", "VPN"],
      frequency: "Daily",
      notes: null
    }
    // ... all assignments from raci.json (one record per capability)
  ]
};
```

Key rules:
- `categories` is built from `capability-areas.json` (includes full capability details: name, description, cmmc_controls)
- `assignments` is copied directly from `raci.json` — primary/backup model, one flat record per capability
- `roles` is copied directly from `raci.json`
- Null values use JS `null` (not Python `None` or JSON string "null")

### Step 3 — Inject into template
1. Read `skills/view/viewer.template.html`
2. Find the exact line: `const D = { /* RACI_DATA_PLACEHOLDER */ };`
3. Replace that entire line with the full `const D = { ... };` block from Step 2

### Step 4 — Write output
Create `~/cmmc-raci-data/{client-slug}/exports/` if it does not exist.
Write the result to `~/cmmc-raci-data/{client-slug}/exports/viewer.html`.

### Step 5 — Confirm to user
```
Viewer generated for {Org Name}
  Output: ~/cmmc-raci-data/{client-slug}/exports/viewer.html

Open in any browser — no server required.
Tabs: Responsibility Matrix | By Person | Personnel Directory
```

## What This Skill Does NOT Do
- Does not open the browser automatically
- Does not modify `raci.json`

## Notes for Claude
- The injection point `const D = { /* RACI_DATA_PLACEHOLDER */ };` is the ONLY line to replace — do not touch any other part of the template
- The D object must be syntactically valid JavaScript
- `categories` must follow the exact structure above — the viewer JS iterates `D.categories[i].capabilities` to build each matrix section
- `assignments` is a flat array (one record per capability), not one record per role-capability pair
