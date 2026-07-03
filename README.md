# SCA License Compliance Workflow

An automated **Software Composition Analysis (SCA) License Compliance** pipeline built in [n8n](https://n8n.io). It reads a project's Software Bill of Materials (SBOM), scans each dependency's license, compares it against a company license policy, automatically creates a **Jira ticket** for any restricted license found, and generates a compliance report.

This removes the need to manually review every open-source dependency in a project for license risk.

---

## How It Works

```
Trigger (Schedule / Manual)
        │
        ▼
  Receive SBOM
        │
        ▼
Scan Licenses (Trivy / FOSSology)
        │
        ▼
Compare with Company License Policy
        │
        ▼
Build Compliance Summary
        │
        ▼
 Restricted Licenses Found?
   │             │
  Yes            No
   │             │
   ▼             │
Create Jira      │
  Ticket         │
   │             │
   └──────┬──────┘
          ▼
   Generate Report
```

| Step | Node | Purpose |
|---|---|---|
| 1 | **Schedule Trigger** | Runs the workflow automatically on a set interval |
| 2 | **Receive SBOM** | Loads the project's dependency list (package + version) |
| 3 | **Scan with Trivy (Simulated)** | Resolves each package's license — swap for a real Trivy/FOSSology call in production |
| 4 | **Compare with Company Policy** | Flags each license as `Allowed` or `Restricted` against an approved list |
| 5 | **Build Compliance Summary** | Aggregates results into one report object |
| 6 | **Restricted Licenses Found? (IF)** | Branches based on whether any violations exist |
| 7 | **Create Jira Ticket** | Automatically opens a ticket describing the violation and recommended action |
| 8 | **Generate Report** | Produces the final compliance report |

---

## Repository Contents

```
├── workflow/
│   └── SCA_License_Compliance_Workflow.json   # Importable n8n workflow
├── docs/
│   └── SCA_License_Compliance_Report.docx     # Full implementation write-up
└── README.md
```

---

## Getting Started

### 1. Import the workflow into n8n

1. Open your n8n instance
2. Click **+ New Workflow** → **Import from File**
3. Select `workflow/SCA_License_Compliance_Workflow.json`

### 2. Configure the allowed license policy

Open the **Compare with Company Policy** node and edit the `allowedLicenses` array to match your organization's approved list:

```js
const allowedLicenses = ["MIT", "Apache-2.0", "BSD", "BSD-3-Clause"];
```

### 3. Connect Jira

1. Generate an Atlassian API token: `https://id.atlassian.com/manage-profile/security/api-tokens`
2. In n8n, create a credential of type **Jira SW Cloud API** (email + API token + your Atlassian site domain)
3. Attach the credential to the **Create Jira Ticket** node
4. Update the `project.key` and `issuetype.name` in that node's request body to match your Jira Software project

> **Note:** Jira Cloud requires the ticket `description` field in **Atlassian Document Format (ADF)**, not plain text. The node in this repo is already formatted correctly — see the JSON body for reference if you need to replicate it elsewhere.

### 4. Replace the mock data sources for production use

This workflow ships with **simulated** SBOM data and a **simulated** license scan so it runs standalone out of the box. Before relying on it for real compliance decisions:

- Replace **Receive SBOM** with a real source (HTTP Request, Read Binary File, or a GitHub Trigger tied to your CI/CD pipeline)
- Replace **Scan with Trivy (Simulated)** with a real scanner call — an `Execute Command` node running the Trivy CLI, or an HTTP Request to a hosted scanning service

### 5. Set your schedule

Open the **Schedule Trigger** node and set a sensible interval (e.g. daily) rather than a short testing interval.

### 6. Publish

Click **Publish**, then confirm the workflow shows the **Published** (active) status so it runs automatically going forward.

---

## Example Violation → Jira Ticket

**Input package:**
```json
{ "package": "Library XYZ", "version": "2.1", "license": "GPL-3.0" }
```

**Resulting Jira ticket:**
> **Title:** License Compliance Violation - Sample Project
> **Description:** Project: Sample Project. Restricted packages found: 1. Library XYZ@2.1 uses GPL-3.0 (Restricted). Recommended Action: Replace the flagged libraries with approved alternatives.

---

## Roadmap / Recommendations

- [ ] Add duplicate-ticket prevention (check for an existing open ticket before creating a new one)
- [ ] Swap in a real SBOM source tied to CI/CD
- [ ] Swap in a real Trivy/FOSSology scan
- [ ] Finalize the allowed-license list with Legal/Security

---

## License

This project is released under the [MIT License](LICENSE).
