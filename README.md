# wyrd-automation-n8n

## Loom Update

Loom walkthrough: [Watch on Loom](https://www.loom.com/share/b8eed3c4f6a14c20a74539eeddf0e750)

This repository contains two n8n workflow exports:

- `Weekly CRM Report.json`
- `Lead Follow-up System (Corrected).json`

## 1) Weekly CRM Report

### What this workflow does

This workflow generates and emails a weekly CRM summary.

### How it works

1. `Schedule Trigger`
   - Runs weekly (configured for day `5` at hour `17` in n8n schedule settings).

2. `Compute Week Range`
   - Calculates start-of-week (Monday 00:00) and current timestamp.

3. Fetch data from CRM endpoints (HTTP Request nodes):
   - `Fetch Leads`
   - `Fetch Deals`
   - `Fetch Tasks`
   - `Fetch Activites` (typo in node name, but functional)

4. Manual date filtering per stream:
   - `Manual Filtering Leads`
   - `Manual Filtering Deals`
   - `Manual Filtering Tasks`
   - `Manual Filtering Activities`
   - These code nodes keep only records in the current week window.

5. `Merge` (4 inputs)
   - Combines all filtered streams into one list.

6. `Process Weekly Summary`
   - Groups data into:
     - new leads
     - moved deals
     - completed tasks
     - activities
   - Also computes count totals.

7. `Format Weekly Report`
   - Builds a text summary message.

8. `Send a message` (Gmail node)
   - Sends the weekly report by email.

## 2) Lead Follow-up System (Corrected)

### What this workflow does

This workflow accepts new leads via webhook, classifies them, sends the right welcome message, logs lead data to CRM, waits, then sends a follow-up email.

Flow:

1. `Receive Lead` (Webhook `POST /lead-intake`)
2. `Parse & Classify Lead`
3. `Route by Lead Type` (`Organic` / `Paid` / fallback)
4. Welcome email:
   - `Send Welcome - Organic` for organic leads
   - `Send Welcome - Paid` for paid leads
5. `Prepare Follow-up`
6. `Wait 24h`
7. `Send Follow-up Email`
8. `Log to CRM` (fallback route from switch)

### What was corrected vs the earlier assignment version

Based on your earlier JSON, these are the important fixes in the corrected export:

1. Fixed switch matching for lead type case:
   - Earlier rules checked `organic` / `paid`
   - Classifier outputs `Organic` / `Paid`
   - Corrected switch now matches `Organic` / `Paid`

2. Fixed follow-up recipient field:
   - Earlier `Send Follow-up Email` used `{{ $json.userEmail }}`
   - Parsed data stores the email in `{{ $json.email }}`
   - Corrected node now uses `{{ $json.email }}`

3. Added paid-branch follow-up path:
   - Earlier only `Send Welcome - Organic` connected to `Prepare Follow-up`
   - Corrected workflow also connects `Send Welcome - Paid` to `Prepare Follow-up`
   - Result: paid leads also enter the follow-up sequence

4. Simplified wait configuration:
   - Earlier wait used `resume: specificTime` with `resumeTime: {{ $json.followUpTime }}`
   - Corrected wait node uses a fixed `amount: 24` delay in the wait node

## How to import

1. Open n8n.
2. Go to **Workflows** -> **Import from File**.
3. Select a `.json` file from this repository.
4. Review node credentials, endpoint URLs, and field mappings.
5. Save and activate.
