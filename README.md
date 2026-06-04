# Fondo Ruso — n8n Automation

This repository contains n8n workflow definitions for the **Fondo Ruso** membership management system.

## Stack

| Component | Tool |
|---|---|
| Database & forms | [NocoDB](https://dashboard.fondoruso.ru) |
| Automation | [n8n](https://n8n.babayma.com) (self-hosted) |
| Email | SMTP via `noreply@fondoruso.ru` |

## Repository structure

```
workflows/
├── payment-processor/
│   ├── workflow.json   — importable n8n workflow
│   └── README.md       — setup & configuration guide
└── payments-summary/
    ├── workflow.json   — importable n8n workflow
    └── README.md       — setup & configuration guide
```

## Workflows

| Workflow | Trigger | Purpose |
|---|---|---|
| [payment-processor](workflows/payment-processor/) | NocoDB webhook (new payment record) | Validates payment type and updates member due date |
| [payments-summary](workflows/payments-summary/) | Weekly schedule (Monday 9:00) | Exports unprocessed payments as CSV and emails for import |

## How to import a workflow

1. Open n8n → **Workflows** → **Import from file**
2. Select `workflow.json` from the relevant folder
3. Follow the configuration steps in that workflow's `README.md`
4. Activate the workflow

## Credentials required

All workflows use credentials stored in n8n. You will need to create these once in your n8n instance:

| Credential | Type | Used by |
|---|---|---|
| VPS Login | HTTP Basic Auth | Webhook authentication |
| SMTP - Fondo Ruso | SMTP | Error alert emails |

NocoDB API tokens are configured directly in each workflow's HTTP Request nodes.

## Adding a new workflow

1. Create a folder under `workflows/` with a short kebab-case name
2. Export the workflow from n8n as JSON and save it as `workflow.json`
3. Add a `README.md` describing the trigger, logic, and required configuration
4. Replace any secrets with `YOUR_*` placeholders before committing
