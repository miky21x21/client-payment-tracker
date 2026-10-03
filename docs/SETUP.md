# Setup guide

Prerequisites: a Google account, and n8n (self-hosted or cloud) with Google Sheets and Gmail OAuth credentials.

## 1. Spreadsheet
1. Upload `template/client-payment-tracker-template.xlsx` to Google Drive → **Open with Google Sheets** → **File → Save as Google Sheets**.
2. Delete the sample **Payments** tab (the form will create the real one).
3. **Format → Theme → Customize → Font: Arial** so new tabs and rows match.
4. Format the **Clients → email** column as **Plain text** (Format → Number → Plain text) so addresses containing `+` are accepted.
5. Replace the sample clients, services and subscriptions with your own. Keep the grey formula columns.

## 2. Staff payment form
1. In the sheet: **Tools → Create a new form**, titled *Record a Payment*.
2. Questions, in this order:

| # | Question | Type | Required |
|---|---|---|---|
| 1 | Service reference | Dropdown — paste the values of **Subscriptions → form_label** | Yes |
| 2 | Payment date | Date | Yes |
| 3 | Payment mode | Dropdown: Cash, Bank, UPI | Yes |
| 4 | Amount received | Short answer, validation: number > 0 | Yes |
| 5 | UTR / Reference | Short answer | No |
| 6 | Note | Short answer | No |

3. **Settings → Responses → Collect email addresses: Verified.**
4. **Publish** the form.
5. Rename the new responses tab to **Payments**. Expected columns: `Timestamp | Email Address | Service reference | Payment date | Payment mode | Amount received | UTR / Reference | Note`.
6. Do **not** add formulas or columns to the Payments tab — the **Payment_links** tab does all calculations.

## 3. n8n — Recurring Charges
Schedule Trigger (daily 06:00) → Google Sheets *Get rows* (Subscriptions) → Filter → Edit Fields *Build charge* → Remove Duplicates → Google Sheets *Append* (Charges) → Google Sheets *Update* (Subscriptions, match on `sub_id`).

- **Filter (AND):** `active` = `Y`; `next_due_date` not empty; `{{ DateTime.fromFormat($json.next_due_date,'dd-MMM-yyyy') <= $now.startOf('day') }}` is true.
- **Build charge:**
  - `charge_id` = `{{ 'CH-' + $json.sub_id + '-' + DateTime.fromFormat($json.next_due_date,'dd-MMM-yyyy').toFormat('yyyyMMdd') }}`
  - `amount` = `{{ Number(String($json.fee).replace(/[^0-9.]/g,'')) }}`
  - `charge_date` = due date as `yyyy-MM-dd`
  - `new_next_due` = due date + 1 month (Monthly) / + 1 year (Yearly) / blank (One-time)
  - `new_active` = `N` for One-time, otherwise `Y`
- **Remove Duplicates:** *Remove items processed in previous executions*, dedupe on `{{ $json.charge_id }}`.
- Both Sheets write nodes: **Options → Cell Format → Let Google Sheets format**.
- Workflow settings: **Timezone → your local timezone**; **Error workflow → your alert workflow**.

## 4. n8n — Payment Confirmation
Google Sheets Trigger (Row Added, Payments, every minute) → Edit Fields *Parse* → Wait 10 s → Get rows (Subscriptions, `sub_id`) → Get rows (Clients, `client_id`) → Gmail *Send*.

- `sub_id` = `{{ $json['Service reference'].slice(0,6) }}`
- `pay_date` = `{{ DateTime.fromFormat($json['Payment date'],'M/d/yyyy').toFormat('dd MMM yyyy') }}`
- Gmail: **Email type HTML**, **Append n8n Attribution off**. The body shows only the amount, date, mode, reference and the remaining balance for that one service.

## 5. Go live
1. **Publish** every workflow (edits only take effect after publishing again).
2. Test with **Execute workflow** from the trigger — never "Execute step" on a node that writes to the sheet.
3. Use your own `you+client@gmail.com` addresses for test clients, never realistic-looking third-party addresses.
