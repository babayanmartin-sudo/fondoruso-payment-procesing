# Payments Summary

Runs weekly (Monday 9:00) to export all unprocessed payment records from NocoDB as a UTF-8 CSV file and email it as an attachment. After a successful send, each exported record is marked as processed (`Send = 1`).

## Flow

```
Weekly Trigger → Get many rows (NocoDB) → Build CSV → Send CSV by Email
  → Expand Record IDs → Mark as Sent
```

## CSV Format

UTF-8 with BOM, semicolon-delimited, compatible with CoinKeeper import:

```
SUM;;Взносы;;YYYY-MM-DD HH:MM:SS;Member Name;
```

- **SUM** — extracted from the `Payment` field (1800, 2500, 3600, or 5000)
- **Object** — always `Взносы`
- **Date** — timestamp of the export run
- **Comment** — member's `Name`

The filename includes the reporting period: `fondoruso_payments_YYYY-MM-DD_to_YYYY-MM-DD.csv`

## Processed flag

`Send` field in NocoDB is set to `1` after successful export. Records with `Send = 1` are excluded from future runs via the NocoDB node filter `(Send,neq,1)`.

## Configuration

Replace the following placeholders before importing:

| Placeholder | Where | Description |
|---|---|---|
| `YOUR_NOCODB_WORKSPACE_ID` | Get many rows | NocoDB workspace ID (from the dashboard URL) |
| `YOUR_NOCODB_PROJECT_ID` | Get many rows, Mark as Sent | NocoDB project/base ID |
| `YOUR_NOCODB_PAYMENTS_TABLE_ID` | Get many rows, Mark as Sent | NocoDB payments table ID |
| `YOUR_NOCODB_CREDENTIAL_ID` | Get many rows | n8n NocoDB API Token credential ID |
| `YOUR_NOCODB_API_TOKEN` | Mark as Sent | NocoDB API token for the PATCH request |
| `YOUR_SMTP_CREDENTIAL_ID` | Send CSV by Email | n8n SMTP credential ID |
| `YOUR_PRIMARY_EMAIL` | Send CSV by Email | Primary recipient for the weekly CSV |
| `YOUR_CC_EMAIL` | Send CSV by Email | CC recipient |
| `YOUR_WORKFLOW_ID` | — | n8n workflow ID (update after first import) |
