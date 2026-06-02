# Payment Processor

Triggered when a member submits a payment form in NocoDB. Reads the selected payment type and updates the member's due date in the members table. Sends an alert email if the payment type is unrecognised.

## Flow

```
NocoDB form submission (webhook)
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
| `Полгода (1800 или 2500)` | `2027-01-31` |
| `Год (3600 или 5000)` | `2027-07-31` |

Any other value triggers the error email.

## Configuration

After importing, replace the following placeholders in the workflow nodes:

| Placeholder | Where | Description |
|---|---|---|
| `YOUR_WEBHOOK_PATH` | New Payment Submission | UUID path of the webhook (from n8n webhook node) |
| `YOUR_BASIC_AUTH_CREDENTIAL_ID` | New Payment Submission | n8n credential ID for webhook basic auth |
| `YOUR_NOCODB_PROJECT_ID` | Find Member, Update Due Date | NocoDB project ID (found in the API URL) |
| `YOUR_NOCODB_TABLE_ID` | Find Member, Update Due Date | NocoDB members table ID |
| `YOUR_NOCODB_API_TOKEN` | Find Member, Update Due Date | NocoDB API token (Settings → Team & Auth → API Tokens) |
| `YOUR_ALERT_EMAIL` | Send Error Email | Email address to receive error alerts |
| `YOUR_SMTP_CREDENTIAL_ID` | Send Error Email | n8n credential ID for SMTP |

## NocoDB setup

- Payments table must have a `Payment` field (Single Select) with options:
  - `Полгода (1800 или 2500)`
  - `Год (3600 или 5000)`
- Members table must have a `Взнос` date field
- A NocoDB webhook must point to this n8n webhook URL on `records.after.insert`
