# Payment Processor

Triggered when a member submits a payment screenshot in NocoDB. Reads the selected payment type, updates the member's due date, and — if the member's status is `Заявка` — also sets their status to `FR` and records the join date. Sends an alert email if the payment type is unrecognised.

## Flow

```
NocoDB webhook (records.after.insert on payments table)
  → Extract member name + payment type
  → Valid payment type?
      YES → Find Member & Build Payload → Update Due Date
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

## New member promotion

If the member's `Статус` field is `Заявка` at the time of payment, the PATCH also sets:

| Field | Value |
|---|---|
| `Статус` | `FR` |
| `Дата вступления` | `2026-08-01` |

## Node: Find Member & Build Payload

This Code node replaces a separate HTTP Request + Code node pair. It:
1. Looks up the member row in the members table by `Name`
2. Builds the PATCH payload conditionally based on `Статус`
3. Passes `memberId` and `payload` to `Update Due Date`

Using a single Code node avoids cross-node `$()` references, which are unreliable in n8n 2.20's task runner.

## Configuration

After importing, replace the following placeholders in the workflow nodes:

| Placeholder | Where | Description |
|---|---|---|
| `YOUR_WEBHOOK_PATH` | New Payment Submission | UUID path of the n8n webhook |
| `YOUR_BASIC_AUTH_CREDENTIAL_ID` | New Payment Submission | n8n credential ID for webhook basic auth |
| `YOUR_NOCODB_PROJECT_ID` | Find Member & Build Payload, Update Due Date | NocoDB project/base ID (from the API URL) |
| `YOUR_NOCODB_MEMBERS_TABLE_ID` | Find Member & Build Payload, Update Due Date | NocoDB members table ID |
| `YOUR_NOCODB_API_TOKEN` | Find Member & Build Payload, Update Due Date | NocoDB API token (Settings → Team & Auth → API Tokens) |
| `YOUR_ALERT_EMAIL` | Send Error Email | Recipient for unrecognised payment alerts |
| `YOUR_SMTP_CREDENTIAL_ID` | Send Error Email | n8n credential ID for SMTP |

## NocoDB setup

- **Payments table** — must have a `Payment` field (Single Select) with the four options above, and a NocoDB webhook configured for `records.after.insert` pointing to this n8n webhook URL
- **Members table** — must have a `Взнос` date field, a `Статус` text/select field, and a `Дата вступления` date field
