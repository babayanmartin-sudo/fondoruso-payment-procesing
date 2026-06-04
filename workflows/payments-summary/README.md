# Fondo Ruso: Payments Summary

Runs weekly (Monday 9:00) to export all unprocessed payment records from NocoDB into a CSV file and email it as an attachment. After a successful send, each exported record is marked as processed.

## Flow

```
Weekly Trigger → Fetch Unprocessed Payments → Has Records?
  → Build CSV → Send CSV by Email → Expand Record IDs → Mark as Sent
```

## Configuration

Replace the following placeholders before importing:

| Placeholder | Description |
|---|---|
| `YOUR_NOCODB_PROJECT_ID` | NocoDB project/base ID |
| `YOUR_NOCODB_PAYMENTS_TABLE_ID` | NocoDB table ID for the payments table |
| `YOUR_NOCODB_API_TOKEN` | NocoDB API token (`xc-token`) |
| `YOUR_SMTP_CREDENTIAL_ID` | n8n SMTP credential ID |
| `YOUR_ALERT_EMAIL` | Recipient email address for the CSV |
| `YOUR_WORKFLOW_ID` | n8n workflow ID (set after first import) |

## CSV Format

UTF-8 with BOM, semicolon-delimited, compatible with CoinKeeper import:

```
SUM;;Взносы;;YYYY-MM-DD HH:MM:SS;Member Name;
```

- **SUM** — extracted from the `Payment` field (1800, 2500, 3600, or 5000)
- **Object** — always `Взносы`
- **Date** — timestamp of export run
- **Comment** — member's `Name`

## Processed flag

Field `ck0jfgt6xcunro2` in NocoDB is set to `1` after successful export. Records with this field `= 1` are excluded from future runs.
