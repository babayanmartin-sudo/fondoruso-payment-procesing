# Payment Processor

Triggered when a member submits a payment screenshot in NocoDB. Reads the selected payment type, updates the member's due date in the members table, and sends an alert email if the payment type is unrecognised.

## Flow

```
NocoDB webhook (records.after.insert on payments table)
  → Extract member name + payment type
  → Valid payment type?
      YES → Find member row → Update due date (Взнос)
      NO  → Send alert email
```

## Trigger

**NocoDB webhook** — `records.after.insert` on the payments table.

The webhook is protected with HTTP Basic Auth (credential: **VPS Login**).

## Payment type mapping

The NocoDB form field `Payment` must contain one of these exact values:

| Payment field value | Due date set |
|---|---|
| `Полгода 1800` | `2027-01-31` |
| `Полгода 2500` | `2027-01-31` |
| `Год 3600` | `2027-07-31` |
| `Год 5000` | `2027-07-31` |

Any other value triggers the error email.

> **Note on encoding:** The NocoDB webhook delivers Cyrillic text as double-encoded Latin-1 bytes. The string comparisons in *Extract Submission Info* and the field name in *Update Due Date* intentionally use this same encoding — do not decode them.

## Configuration

After importing, replace the following placeholders in the workflow nodes:

| Placeholder | Where | Description |
|---|---|---|
| `YOUR_WEBHOOK_PATH` | New Payment Submission | UUID path of the n8n webhook |
| `YOUR_BASIC_AUTH_CREDENTIAL_ID` | New Payment Submission | n8n credential ID for webhook basic auth |
| `YOUR_NOCODB_PROJECT_ID` | Find Member, Update Due Date | NocoDB project/base ID (from the API URL) |
| `YOUR_NOCODB_MEMBERS_TABLE_ID` | Find Member, Update Due Date | NocoDB members table ID |
| `YOUR_NOCODB_API_TOKEN` | Find Member, Update Due Date | NocoDB API token (Settings → Team & Auth → API Tokens) |
| `YOUR_ALERT_EMAIL` | Send Error Email | Recipient for unrecognised payment alerts |
| `YOUR_SMTP_CREDENTIAL_ID` | Send Error Email | n8n credential ID for SMTP |

## NocoDB setup

- **Payments table** — must have a `Payment` field (Single Select) with the four options above, and a NocoDB webhook configured for `records.after.insert` pointing to this n8n webhook URL
- **Members table** — must have a `Взнос` date field that stores the membership due date
