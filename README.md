# MoneyTracker (Android, Kotlin, Jetpack Compose, Room)

## Open and run
1. Open this folder in Android Studio and let Gradle sync.
2. Run on a phone or emulator (SMS/notification capture requires a real device for realistic testing).
3. Grant SMS permissions if you want to import bank messages. Enable notification access in Settings to capture supported bank-app notifications.

## Main screens
- **Home:** month-to-date income, spending and net; editable balance cards for Credit card, Accounts, Loans and Car loan; tap a category to see captured accounts and edit account balances; upcoming bills appear near the top, with a pulsing indicator for bills due within five days and a dismiss action; month countdown reads “days left this month.”
- **Transactions:** date filters, account filters, six-month spending trend and analysis. Expenses are red, income is green, transaction rows animate in after loading, and tapping a transaction opens its captured SMS/notification text. Choose “Exclude future” to exclude the current transaction and remember the account/card suffix (or merchant/category when no account suffix exists) for future matching transactions.
- **Bills:** reminders with due-date countdowns. Tap a reminder to view the original captured message; mark it paid or dismiss it.
- **Settings:** material-styled permissions and import controls, on-device AI, optional cloud AI key, custom category parsing rules with a sample tester, and local CSV export through Android's file picker. The bank sender list is no longer shown in the UI.

## Custom category rules
In Settings > Custom parsing rules, add one rule per line in this form:

`Education=school|tuition|university`

Rules are regular expressions and match against the merchant when available, otherwise the message text. They take priority over built-in categories. Built-in categories include Food, Grocery, Education and Transportation. Custom rules are saved locally on the device.

## Local data and privacy
The app stores parsed transactions and reminders in a local Room database. Captured source text is retained so it can be inspected from the transaction/reminder. CSV export includes that source text, so treat the exported file as sensitive and save it somewhere private. The app does not upload bank messages as part of local parsing. An optional AI API key is stored in local app preferences.

SMS permissions are restricted on Google Play; personal sideloading may be required. Notification access must be enabled manually in Android Settings.

## Rule-based analysis without an LLM
Transaction extraction and category assignment can run entirely without an LLM using the built-in Kotlin parser and custom regex/keyword rules. The LLM is optional and only helps produce a natural-language spending summary; the arithmetic is computed locally. To make classification more adaptive without a hosted LLM, a future version could add a small on-device classifier trained on manually labelled transactions, but it would need representative labelled examples and confidence/fallback handling.

## Exclusion behavior
Exclusion rules are stored in local app preferences. When a card/account suffix is present, the rule matches that suffix; otherwise it matches the merchant and category. This is designed to avoid re-importing unwanted transaction types while still retaining ordinary transactions that contain card/account identifiers.

## SMS parser examples added
- AUTO FINANCE installment reminders are parsed through the existing bill/reminder path; they are not treated as completed expenses.
- Credit-card payment confirmations phrased as `Your Payment of AED ... for card ... has been processed` are classified as `Card payment` and excluded from spending totals to reduce double-counting when the matching bank-account debit is also present.
- The Settings > Custom parsing rules tester now has sample buttons for a loan reminder and a card payment.
- Custom rules remain local category overrides in the current version (`Category=regex` per line); they do not yet let users redefine every extraction field or record target.


## Latest improvements

- Parser now rejects common marketing/reward messages (for example, "get cashback") unless the text indicates an actual posted credit. Review the parser tester in Settings for messages from your own bank that may need more precise rules.
- Transactions have an Edit action for amount, currency, merchant/description, category, and income/expense type.
- Pending reminders have an Edit action for title, amount, currency, and due date (`yyyy-MM-dd`).
- Settings > Appearance supports System/Light/Dark mode and Teal/Blue/Purple/Orange accent colours. Preferences are stored locally.

These changes were source-edited but not built or run in Android Studio in this environment. Please run **Build > Rebuild Project** and share any compiler errors if they appear.


## Latest enhancements
- Tap a transaction row to edit it; use the info icon to inspect its captured message.
- Salary/payroll messages are explicitly classified as income.
- SMS imports are additive and do not clear notification-only transactions.
- Settings supports a main currency, a monthly budget, and reference-rate refresh.
- CSV export and import are available in Settings; import preserves existing rows and skips duplicates.
- Dashboard account groups show ATM cash withdrawals, bank accounts, and detected credit cards.
- Transaction rows attempt to load bank favicons from Google's public favicon service.
- Added a MoneyTracker launcher icon.

Exchange rates use a public reference-rate feed because Google does not provide a supported public currency-conversion API. Rates are indicative and may not match historical rates or bank/card settlement rates.


## Latest maintenance and UI updates
- Home now consolidates account balances, detected cash withdrawals, bank accounts, cards and loans under a single **Accounts** heading.
- Transactions has a floating **+** action for adding a manual income or expense.
- At app startup and after past-SMS import, cleanup removes only likely duplicate transaction pairs with matching amount, currency, direction, close timestamps, and a matching account/merchant/source message. It avoids deleting manual transactions and does not wipe transaction history.
- Past-due credit-card reminders are marked paid only when a stored card-payment transaction matches the reminder amount/currency within a date window around the due date. This is a best-effort SMS match; ambiguous or unmatched bills remain pending for manual confirmation.
- Bills use compact rows with a check-circle action to mark paid and an X action to dismiss.

These changes have been source-edited but not built or exercised on a device in this environment; test the cleanup against your own bank messages before relying on automatically matched bill status.


## Transaction editing, paid-bill evidence, and charts

- Transactions can be added from the floating **+** action and deleted from their rows; bills/reminders can be added and deleted from the Bills screen. The Quick Add chooser uses large, colored action cards.
- When a bill is marked paid, the app conservatively searches existing transactions for a matching debit/card-payment record by amount, currency, date window, and available card details. If found, it stores the matched transaction timestamp, bank/app label, capture channel (SMS or bank-app notification), and original message. If no match is found, it still records the manual mark time and clearly indicates that no payment message was matched.
- Past-due credit-card bills are reconciled against captured payments on startup. Ambiguous or unmatched bills remain pending. This is heuristic matching and should be reviewed against real bank messages.
- Spending insights include a persisted chart selector for Line, Column, Bar, Waterfall, Pie, Donut, Area, Stacked Column, Stacked Bar, Scatter, Combo, Bullet, Gauge/Speedometer, OHLC, Candlestick, and Heikin-Ashi charts. Bullet and gauge views require a monthly budget; OHLC-style views summarize monthly transaction amounts and are not market-price charts.
- Room database schema version 4 adds nullable capture-channel and paid-bill evidence fields, with migrations from schema versions 2 and 3 intended to retain existing records.

Source was edited but a complete Gradle build and device test could not be run in this environment. Back up the app data before installing and verify bill matches against the original SMS/notification records.

## Enhanced v7 changes

- Home dashboard shows an animated greeting using the saved user name and a horizontally marquee finance quote. The quote is fetched and cached daily from ZenQuotes.io with an offline fallback; the UI credits ZenQuotes.io.
- Monthly In / Out totals are shown near the top of Home.
- The Transactions chart follows the selected period (default: this month) and aggregates spending by category. The chart-type picker preserves its selection.
- Manual transactions and pasted SMS/notification transactions support date and time entry.
- Paste-message flow previews the parser's best guess, allows correcting expense/income and category fields, and can save reusable `TYPE<TAB>CATEGORY<TAB>REGEX` rules.
- Accounts cards and details show bank, account/card type, ending digits, balance detected from available-balance messages when possible, and a tap-to-edit manual balance starting at zero otherwise.
- Daily summary notification is configurable in Settings and reports daily spend, or asks the user to add a transaction when the day is empty. WorkManager can deliver a little after the selected time.

## Build notes

Open the `moneytracker` directory in Android Studio and use **Build > Rebuild Project**. This source archive was syntax-scanned, but a full Gradle build and device test were not available in the preparation environment. Keep a backup of the current project and app data before installing.


## Notification capture troubleshooting

MoneyTracker reads bank-app notifications only when Android Notification Access is enabled. The notification parser accepts explicit debit/credit/payment/salary wording even if the bank redacts the account number. Promotional and OTP/declined messages remain filtered. If capture stops, check Settings → Special app access → Notification access and re-enable MoneyTracker; some Android vendors may also require allowing background activity.

## Expense category rules

The Settings screen includes **Categories of expenses** with local-first rules for Food and dining, Education, EMI, Fuel, Groceries, Health, House Rent, Investment, Kids, Movies and Subscriptions, Shopping, and Bills. Merchant patterns are evaluated on-device before the legacy fallback categories. Saving the rules reapplies them to existing debit transactions; later SMS and notification transactions use the same rules. Editing a transaction's category also teaches a merchant-specific rule and updates existing transactions with the same merchant name. No cloud AI key is required for these category rules.

## v10 category and transaction-rule updates
- Transaction edit, manual-add, and pasted-message flows use a category dropdown. Unmatched parser classifications now fall back to `Unknown` so the user can choose a category.
- Settings sections and expense category cards include icons.
- Categories of expenses supports adding custom categories and matching regex/keywords. It rejects duplicate category names, invalid regex patterns, and likely keyword overlaps with existing categories.
- Pasted SMS/notification rule workflow now creates an editable local regex suggestion that generalizes common currency, date, and numeric fields. If an on-device model is installed, `Improve with on-device AI` can generate a regex locally; it is accepted only if it compiles and matches the sample. User approval saves it for future SMS/notification parsing.

## v12 dashboard and accounts updates

- Transaction rows and expense-category settings now use the same `CategoryGlyph` mapping so the same category displays the same icon in both places.
- Finance charts animate their values on first display and use a brighter multi-colour palette.
- The greeting card includes a gently floating sun in daytime and moon in evening/night. The external quote card was removed.
- Home dashboard order is greeting/summary, budget, accounts, upcoming bills due within the next seven days, then recent transactions.
- The home recent-transactions list is limited to seven rows.
- Account cards are grouped into bank/debit accounts, credit cards, and loans. The add flow supports custom accounts/cards and Home loan, Car loan, or Other loan types, rejects duplicate names, and starts manual balances at zero. Detected accounts are grouped by bank, account suffix, and normalized account type to reduce duplicates.

Build note: this source update was checked for Kotlin parser-level syntax errors and the ZIP archive was integrity-checked. A full Android Gradle build/device test was not run in this environment.

## v13 update — notification merchants, generic parsing, detailed budgets

- Notification and SMS ingestion now applies a merchant fallback extractor before saving a transaction, and the parser's merchant extraction stops before currency/amount fields. Transaction rows can use a user-approved regex's named groups (`amount`, `currency`, `merchant`/`payee`, `card`/`account`, and `datetime`/`date`) to populate structured transaction fields.
- The local regex suggestion now generates named capture groups for merchant, currency, amount, card/account digits, and date/time where those values can be identified from the sample. On-device AI prompts request the same field groups.
- Accounts settings now has detailed monthly budget inputs for all 12 expense categories. Saving them calculates the overall monthly budget from the entered category limits. The Home budget card shows category-level spending versus limits.
- Tested the parser utility with Kotlin compiler sample cases: merchant extraction from a bank alert, named-group extraction of merchant/amount/currency/card/date-time, and a generic rule matching a second transaction with changed values.

A full Android Gradle build/device test was not run in this environment.


## v14 requested enhancements (source update)

- Optional local rolling automatic backup targets 2:00 AM and retains the two newest ZIP backups. It includes the checkpointed Room database, SharedPreferences settings, and installed on-device model. Android WorkManager can delay execution for battery/background restrictions.
- CSV export/import now includes the automatic-backup toggle and the new-transaction notification toggle.
- SMS historical import is restricted to the last six calendar months.
- Bills use grey for more than five days remaining, orange for 2–5 days, red for under two days and overdue, and light green for paid. Paid and overdue rows show a subtle watermark.
- Optional new-transaction notifications show amount and merchant when an SMS or bank-app notification is newly captured; historical SMS imports do not generate notification spam.
- Screenshot import uses on-device ML Kit OCR to extract text, then shows an editable review dialog and requires the existing bank parser to identify a transaction before saving. This is OCR-assisted extraction, not a generative AI guarantee; unknown/ambiguous text is not auto-imported.

A full Android Gradle build and on-device test still need to be run in Android Studio. The environment used to prepare this archive did not have Gradle installed, so the source changes have not been compiled here. Back up app data before installing and test against sample transactions before relying on automation.

### Follow-up fixes
- Transaction edit dialog now includes editable transaction date and time (yyyy-MM-dd / HH:mm).
- Parser transaction signals explicitly include salary/payroll credits, account credits, telegraphic-transfer issuance and account deductions. Past SMS import still only scans sender IDs configured/recognized in Settings; verify the actual sender ID is included there before re-importing.
- Screenshot import remains on-device OCR + parser, not generative AI. Low-quality images or unsupported layouts may still need manual text correction; no unverified OCR result is saved automatically.

## Cash wallet and Payments follow-up
- Transaction totals now exclude card payments, transfers, direct debits and cash withdrawals as internal movements rather than treating them as spending/income.
- Transactions screen has a virtual **Payments** filter for card payments, transfers, direct debits and cash withdrawals.
- Cash withdrawals are categorized as **Cash transfer** by the parser so they can be shown separately from everyday expenses.
- Manually added transactions marked as paid from the cash wallet are tagged with `kind = Cash Wallet`; the existing cash-balance adjustment remains in place.
- Note: a full cash-wallet ledger view with per-entry add/delete and automatic reconciliation of imported ATM withdrawals is not yet implemented in this build. The existing wallet card still opens the manual balance editor.

## Cash Wallet activity and cash-payment editing
- The Cash Wallet card opens an activity dialog with manual cash spending and cash top-up entries, plus per-entry deletion.
- Existing transactions can be edited with a `Paid/received through Cash Wallet` toggle; the stored wallet balance is adjusted by the change in cash-linked amount.
- New imported ATM/cash-withdrawal transactions categorized as `Cash transfer` or `Cash withdrawal` add to the wallet balance only when a transaction is newly inserted (SMS/notification deduplication avoids double additions).
- `Cash Wallet top-up` movements are excluded from income/expense totals.

## Latest UI and transaction refinements
- ATM/cash-withdrawal transactions are categorized as `Cash withdrawal`, excluded from the ordinary transaction list and expense totals, and shown in Cash Wallet activity. Newly captured withdrawals add to the wallet balance once after successful de-duplicated insertion.
- Manual transactions support an explicit Cash withdrawal category and a calendar date picker; manual bill/reminder due dates also use a calendar picker.
- Transaction date filters include Today and Yesterday alongside month and rolling-period filters.
- Net amount uses stronger typography and switches green/red with a matching trend arrow according to its sign.
- The Loans quick-add defaults to Car loan, and the account type selector includes Home loan, Car loan, and Other loan.


## Version 1.1 follow-up fixes
- Card repayments are classified as debit card-payment records, excluded from expense totals, and displayed in the Bills & Payments tab.
- Incoming transfers count as income; debit transfers and EMI/loan repayments count as spending. ATM withdrawals remain cash-wallet movements rather than double-counted expenses; actual cash spending recorded from the wallet counts as expense.
- Transaction rows show a running cumulative expense total for the selected period.
- Settings includes About and the installed app version; app version bumped to 1.1.
- Loan add-account type starts at Home loan and exposes Home loan, Car loan and Other loan options.


## Version 1.2 — Learn notification formats

Settings → AI → Custom parsing rules now includes **Automatically learn & save rule**. Paste a real bank-app notification or SMS; the app generalizes the sample into a reusable local regex with named fields where available, infers a transaction type/category, validates that the rule matches the sample, and saves it for future SMS and notification parsing. The generated rule can be reviewed and edited. Rule learning and matching run on-device; no bank-specific app patch or network service is required. Always verify the inferred transaction type and category for a new sample.


### Dubai First notification capture fix (v1.3)
- Notification app label is prepended to extracted notification text so parsers can identify alerts when Android keeps the bank name only in the title/app label.
- Labelled `Card Transaction` alerts are parsed from their `Amount`, `Card ending with`, `At`, and `Date` fields without requiring the bank name to be embedded in the notification body.


## Screenshot transaction-list import (v1.4)
- In Settings → AI → Import from screenshot, choose a transaction-list screenshot.
- On-device ML Kit OCR extracts merchant/amount rows and the visible month/day.
- Review all detected rows and import them together; duplicate rows are skipped using a stable fingerprint.
- Processing percentages (for example, “Processing 5%”) and page-level totals are ignored. Only merchant and transaction amount are imported.
- Screenshot import currently treats detected rows as AED expenses; review the list before confirming.

## Cloud APK build
This project includes `.github/workflows/android-apk.yml`. Upload the project to a GitHub repository, open Actions → Build MoneyTracker APK → Run workflow, then download the `moneytracker-v20-debug-apk` artifact from the completed run. This workflow creates a debug APK; it is not signed with the same private key as any previously installed release build. Android may refuse an in-place update if the signing certificate differs. Back up app data before installing, and uninstalling the existing app can erase local data.

## v21 compile fix
- Added explicit generic and lambda parameter types to the screenshot-import `withContext`/`sumOf` block to resolve Kotlin type inference failure reported at MainActivity.kt:1988.
- App versionCode 6, versionName 1.5.
