---
name: view
description: Generate the interactive HTML responsibility matrix viewer from raci.json. Opens in any browser with no server required.
---

# cmmc-raci:view — Generate Interactive HTML Viewer

## Purpose
Reads `raci.json` and injects its data into `viewer.template.html` to produce a self-contained, interactive HTML file. The viewer shows a matrix grid of capability areas vs. roles with R/A/C/I badges, clickable role cards, and an inspector panel displaying CMMC control mappings and contact info.

## Preconditions
`~/cmmc-raci-data/{client-slug}/raci.json` must exist.

## Reference Files
- `skills/view/viewer.template.html` — the HTML template to inject data into
- `skills/init/references/capability-areas.json` — capability area details for inspector panel

## What to Do

### Step 1 — Identify client
If not already known from context, ask which client. Read `~/cmmc-raci-data/{client-slug}/raci.json` and `capability-areas.json`.

### Step 2 — Build the D object
Construct the following JavaScript object:

```javascript
const D = {
  client: "<org name>",
  version: "<version>",
  last_updated: "<ISO timestamp>",
  roles: [
    { id: "role-ciso", title: "Chief Information Security Officer (CISO)", department: "Executive / Security", person_name: null, person_email: null, person_title: null },
    // ... all roles from raci.json
  ],
  categories: [
    {
      id: "endpoint",
      name: "Endpoint Security",
      capabilities: [
        { id: "cap-edr", name: "Endpoint Detection & Response (EDR)", description: "...", cmmc_controls: ["3.14.1", "3.14.2", "3.14.6", "3.14.7"] },
        // ... all capabilities in this category from capability-areas.json
      ]
    },
    // ... all 9 categories
  ],
  assignments: [
    { capability_id: "cap-edr", role_id: "role-ciso", raci: "A", notes: null },
    // ... all assignments from raci.json
  ],
  coverage: {
    capabilities_without_R: [],
    capabilities_without_A: []
  }
};
```

The `categories` array must be built by joining `capability-areas.json` category structure with the `description` and `cmmc_controls` fields from that same file.

### Step 3 — Inject into template
1. Read `skills/view/viewer.template.html`
2. Find the line: `const D = { /* RACI_DATA_PLACEHOLDER */ };`
3. Replace it with the full `const D = { ... };` block constructed in Step 2
4. Replace `<title>RACI Viewer</title>` with `<title>{Org Name} — CMMC Responsibility Matrix</title>`

### Step 4 — Write output
Write the result to `~/cmmc-raci-data/{client-slug}/exports/viewer.html`.

### Step 5 — Confirm to user
Print:
```
Viewer generated for {Org Name}
  Output: ~/cmmc-raci-data/{client-slug}/exports/viewer.html

Open in any browser. No server required.
```

## What This Skill Does NOT Do
- Does not open the browser automatically
- Does not modify `raci.json`

## Notes for Claude
- The `const D = { /* RACI_DATA_PLACEHOLDER */ };` line is the ONLY injection point — do not replace any other part of the template
- The D object must be valid JavaScript (not JSON) — strings are fine, but do not use JSON.stringify wrapping
- The categories array in D must follow the exact structure shown above — the viewer JavaScript depends on it
- If coverage_check has gaps, they are passed in D.coverage so the viewer can highlight them
