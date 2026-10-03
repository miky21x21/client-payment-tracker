# Lessons learned

Every real problem hit while building this, and the fix.

## Google Sheets
| Problem | Cause | Fix |
|---|---|---|
| Formulas silently switched to `Payments_old!` | Renaming a tab rewrites every formula that references it | Rename, then Find & Replace with *Also search within formulas* |
| Formula showed as plain text | Leading spaces before `=` | `=` must be the first character |
| `#REF!` on every helper row | ARRAYFORMULA collided with values below it | One array formula per column, in row 1 with the header built in (`={"header";ARRAYFORMULA(...)}`) |
| "Formulas are not supported in a table header row" | The form's responses tab was a Sheets *table* | Leave the form tab untouched; do calculations on a separate **Payment_links** tab |
| `#VALUE!` from SUMIFS | Ranges from two tabs had different sizes | Bring the amount into the helper tab so all ranges come from one tab |
| Dashboard still showed old totals | Formulas were correct; the form tab hadn't replaced the sample tab yet | Finish the swap before judging the numbers |
| `+` email addresses rejected | A cell starting with or containing `+` can be parsed as a formula | Format the column as Plain text (or prefix with `'`) |
| New rows looked different from old ones | Formatting had been applied to cells, not columns; new tabs used the theme's serif font | Format whole columns; set the theme font |
| A count showed as "₹5" | Currency format applied to a count | Format → Number → Number |

## n8n
| Problem | Cause | Fix |
|---|---|---|
| The same charge appeared 2–3 times | "Execute step" on the Append node replays the previous input | Test only from the trigger; add a Remove Duplicates guard keyed on a deterministic `charge_id` |
| Dates written as text (`2026-09-30`) | Values sent RAW | Cell Format → *Let Google Sheets format* |
| Schedule ran on New York time | n8n default timezone | Workflow Settings → Timezone → Asia/Kolkata |
| Fixes "didn't work" | Edits change the draft; the trigger runs the published version | Publish again after every change |
| HTML tags appeared in the email | Email type was Text | Email type → HTML |
| n8n footer in emails | Attribution is on by default | Add option *Append n8n Attribution* → off |
| Sheets Trigger asked for a new credential | Each Google node type has its own credential | Reuse the same OAuth client ID/secret |

## Data protection
| Problem | Lesson |
|---|---|
| Test confirmation emails were addressed to realistic third-party Gmail addresses | Test data must never point at real people. Use `you+name@gmail.com` (delivered to yourself) or the reserved `example.com` domain |
