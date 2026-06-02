# Qualys VM Program Dashboard

A single HTML file that gives you everything you need to run a complete Vulnerability Management program using Qualys — from identifying and categorising vulnerabilities, to raising Jira tickets with evidence, to tracking remediation through revalidation.

No server. No installation. No dependencies. Open the file in a browser and start your VM program.

---

## The Problem This Solves

Running a VM program across multiple asset groups in Qualys is painful. You get large CSV exports with thousands of rows, no severity classification tied to your asset criticality, no SLA tracking, no way to quickly generate Jira tickets with the right scope, and no structured way to measure remediation progress.

This dashboard solves all of that in a single file.

---

## How a VM Cycle Works With This Tool

### Step 1 — Setup (Once)
Configure the dashboard to match your environment:
- Define your **asset tags** (parent tags from Qualys) and their criticality (ACS 4 or 5)
- Define **child tag mappings** — Qualys often assigns multiple sub-tags to assets; map them to your parent tags
- Set your **SLA targets** per severity (default: Critical=15d, High=30d, Medium=60d, Low=90d)
- All of this lives in a simple CSV config file — update it in Excel, load it in the dashboard

### Step 2 — Baseline Scan
At the start of each scan cycle (monthly or quarterly):
- Export Qualys VM reports as CSV for your agentic asset groups
- Export separate reports for agentless assets (Network Devices, Database servers)
- Upload each report to the corresponding upload zone in the dashboard
- Set the **scan period** (e.g. `May 2026`) and **SLA start date** (ticket creation date)

The dashboard immediately shows you:
- Total detections per tag, broken down by severity (Critical / High / Medium / Low)
- SLA breach status per tag per severity — On Track, At Risk, or Breached
- KPI summary cards and charts across your entire asset landscape

### Step 3 — Severity Classification
Severity is calculated per detection using your asset criticality (ACS) and Qualys Detection Score (QDS):

| QDS Range | ACS 5 — Critical Assets | ACS 4 — Non-Critical Assets |
|-----------|------------------------|-----------------------------|
| 90–100    | 🔴 Critical             | 🟠 High                     |
| 70–89     | 🟠 High                 | 🟡 Medium                   |
| 40–69     | 🟡 Medium               | 🟢 Low                      |
| 1–39      | 🟢 Low                  | 🟢 Low                      |

This ensures vulnerabilities are prioritised based on both exploitability (QDS) and the criticality of the affected asset (ACS) — not just raw CVSS scores.

### Step 4 — Raise Jira Tickets
For each tag, the dashboard generates ready-to-paste Jira queries. Click the **Jira** button on any tag row to get:

**Vulnerability Query** — filters detections by scan period and severity score range:
```
(((((vulnerabilities.firstFound:[2026-05-01 ... 2026-05-31]
AND (vulnerabilities.status:ACTIVE OR vulnerabilities.status:NEW)
AND vulnerabilities.typeDetected:Confirmed)))))
AND vulnerabilities.detectionScore:[90 ... 100]
```

**Asset Query** — scopes the ticket to the right assets:
- Agentic tags → `(tags.name:YOUR_TAG_NAME)`
- Network Devices → IP-based OR query (auto-extracted from your uploaded report)
- Database servers → `interfaces.address` query (auto-extracted from your uploaded report)

One ticket per tag per severity — up to 4 tickets per tag (Critical, High, Medium, Low).

### Step 5 — Attach Evidence
Download a per-tag evidence CSV with the **⬇ CSV** button. Each file contains every detection for that tag across all severities:

| Asset Name | IP Address | QID | Title | QDS | ACS | Severity | Status |
|------------|------------|-----|-------|-----|-----|----------|--------|

The file is sorted by severity so the engineer can filter in Excel and attach the relevant rows to each Jira ticket as proof of findings.

### Step 6 — Revalidation
After engineers remediate vulnerabilities:
- Export fresh Qualys reports for the same period
- Upload them to the **Revalidation** slots
- The dashboard shows what remains open and how many were fixed
- The gap between baseline and revalidation counts = remediated vulnerabilities
- SLA timers continue running — you can see which tickets are now at risk of breaching

Switch between **📌 Baseline** and **🔄 Revalidation** views at any time using the toggle in the filter bar.

---

## Getting Started

### Download
Download `VM_Dashboard_Qualys_offline.html` — all libraries (Chart.js, PapaParse, SheetJS) are embedded directly in the file. Works with zero internet connection, on any machine, forever.

### Open
Double-click the HTML file or drag it into Chrome. No installation required.

### Configure Your Tags
Edit `pml_config.csv` in Excel to define your asset tags and child mappings, then load it using the **⬆ Load Config** button in the dashboard header.

Config file format:

| Type | Name | ACS | Team | Env | Scan | ChildTag |
|------|------|-----|------|-----|------|----------|
| parent | Banking_Prod_Linux_Audit_Scope | 5 | Banking | Prod | Agentic | |
| parent | Network_Devices | 5 | Network | Prod | Agentless | |
| child | Banking_DC_Prod_ASG | | | | | Banking_Prod_Linux_Audit_Scope |
| child | Banking_Windows_Prod | | | | | Banking_Prod_Windows_Audit_Scope |
| ignore | Internal_Scan_Group | | | | | |

Save the config anytime with **⬇ Save Config** as CSV or Excel.

---

## Key Concepts

### Asset Criticality Score (ACS)
Every asset tag has an ACS of 4 (Non-Critical) or 5 (Critical). This drives severity classification — the same QDS score produces a higher severity on a critical asset than a non-critical one.

### Scan Period
Each upload is tied to a scan period (e.g. `May 2026` or `Q2 2026`). The dashboard tracks data per period so you can compare across cycles.

### SLA Start Date
The date Jira tickets were created. This is the reference point for all SLA countdown timers. Set it per period using the date picker in the header.

### Child Tag Mappings
Qualys assigns multiple tags to assets — your assets may appear under sub-group tags rather than the parent audit scope tag. The dashboard maps child tags to parent dashboard tags automatically. Any unrecognised tags are surfaced in a mapping modal on upload.

### Baseline vs Revalidation
- **Baseline** — the initial scan at the start of the cycle. SLA clock starts here.
- **Revalidation** — a fresh scan after remediation. Shows what was fixed and what remains.

---

## Architecture

| Property | Detail |
|----------|--------|
| File type | Single HTML file |
| File size | ~1.2MB (all libraries embedded) |
| Dependencies | None — Chart.js 4.4.0, PapaParse 5.4.1, SheetJS 0.18.5 all embedded |
| Storage | Browser localStorage (no server, no database) |
| Compatibility | Chrome, Firefox, Edge, Safari |

---

## Limitations

- Designed for single-user local use — not a multi-user web application
- Data is stored in browser localStorage — clearing browser data clears uploaded counts (re-upload CSVs to restore)
- Uploaded file objects (for evidence export) are held in memory only until page refresh

---

## Contributing

Contributions welcome. Possible extensions:
- PDF report generation per scan cycle
- Trend charts across multiple periods
- Additional agentless asset types
- Multi-user support via a lightweight backend

---

## License

MIT License — free to use, modify, and distribute.
