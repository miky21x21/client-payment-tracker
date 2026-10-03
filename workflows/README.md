# n8n workflows

Exports of the three workflows used by this project:

| File | Workflow |
|---|---|
| `recurring-charges.json` | Daily billing of due subscriptions |
| `payment-confirmation.json` | Client confirmation email on each new payment |
| `error-alerts.json` | Email alert when any workflow fails |

## Importing
n8n → **Create workflow → ⋯ → Import from File**, then:
1. Re-select your own Google Sheets, Google Sheets Trigger and Gmail credentials on each node.
2. Point every Google Sheets node at your copy of the spreadsheet (exports contain `YOUR_SHEET_ID` as a placeholder).
3. Set the workflow timezone and error workflow, then **Publish**.
