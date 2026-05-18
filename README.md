# cmmc-raci-v1 — CMMC Information Security Responsibility Matrix

A Claude Code plugin that builds an interactive responsibility matrix mapping 35 infosec capability areas to organizational roles, with CMMC NIST 800-171 control cross-walks baked into every capability.

## What It Produces

- **Interactive HTML viewer** — matrix grid of capability areas vs. roles with R/A/C/I badges, clickable role cards, inspector panel showing CMMC controls and contact info. Opens in any browser, no server required.
- **Markdown RACI table** — formatted for SSP embedding or client deliverables, with a CMMC control cross-walk section.

## Capability Areas (35 total, 9 categories)

| Category | Capabilities |
|---|---|
| Endpoint Security | EDR, Anti-Malware, MDM, Device Encryption |
| Identity & Access | IAM, MFA, PAM, Directory Services |
| Vulnerability Management | Vulnerability Scanning, Pen Testing, Patch Management |
| Network Security | Firewall, Segmentation, VPN, Wireless, DNS/DHCP |
| Cloud Security | IaaS/PaaS, SaaS Config, CASB, Cloud Backup |
| Data Security | Data Classification, DLP, Encryption at Rest, Encryption in Transit, Media Sanitization |
| Monitoring & Operations | SIEM, Log Management, Security Operations |
| Incident Response | IR Planning, IR Handling, Business Continuity/DR |
| Governance & Compliance | InfoSec Policy, Risk Assessment, Security Awareness Training, CMMC Prep, Supply Chain Risk, Config/Change Mgmt |
| Physical Security | Physical Access, Facility Security, Equipment Disposal |

## Skills

```
cmmc-raci:init      — Scaffold a new matrix for a client (pick from 8 standard roles)
cmmc-raci:describe  — Interactive category-by-category assignment review
cmmc-raci:view      — Generate the interactive HTML viewer
cmmc-raci:export    — Package viewer + Markdown table for deliverables
```

## Workflow

```
cmmc-raci:init      → creates ~/cmmc-raci-data/<client-slug>/raci.json
cmmc-raci:describe  → refine assignments and collect person names/emails
cmmc-raci:view      → generates ~/cmmc-raci-data/<client-slug>/exports/viewer.html
cmmc-raci:export    → generates raci-matrix.md + manifest.json
```

## Data Store

```
~/cmmc-raci-data/
└── <client-slug>/
    ├── raci.json          ← source of truth
    └── exports/
        ├── viewer.html    ← interactive matrix viewer
        ├── raci-matrix.md ← Markdown table + CMMC cross-walk
        └── manifest.json  ← file inventory + timestamps
```

## Default Roles (template)

1. CISO / Security Director — Accountable on all 35 capabilities
2. ISSO — Responsible for governance, monitoring, IR, IAM, data security
3. Systems Administrator — Responsible for endpoint, patch, config, directory
4. Network Engineer — Responsible for firewall, segmentation, VPN, wireless
5. Cloud Administrator — Responsible for cloud infra, SaaS, CASB, backups
6. HR Director — Responsible for training and personnel functions
7. Legal / Contracts — Consulted on governance, supply chain, CMMC prep
8. IT Director / CIO — Accountable on infrastructure and BC/DR

All roles and assignments are fully editable. Add or remove roles at `init` time; reassign capabilities anytime via `describe`.
