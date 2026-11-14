# MoneyTracker – V49 follow-up changes

Implemented in this source snapshot:

1. Account and card removal now has confirmation. Detected bank/card accounts can be hidden from the Accounts list and card selectors without deleting historical transactions.
2. Transaction add/edit forms now offer the saved credit cards from Accounts as dropdown selections. Card-payment entries still require a selected card.
3. A user-selected folder is used for manual ZIP backups, automatic ZIP backups, CSV export, and CSV import. Automatic and manual ZIP backups share one retention rule (latest two); CSV exports also retain only the latest two. Selecting a folder removes legacy internal auto-backup ZIPs and prunes older MoneyTracker files in the selected folder.
4. Daily spending reminder controls are compact and use a clock-style time picker.
5. The supplied HDFC Bank A/c XX9565 balance is loaded once as INR 25,348.33. Future HDFC available-balance messages are parsed as account balances, not expenses.
6. Emirates NBD, ENBD, and ENBDX are normalized to the same bank identity.

## Validation

- The Kotlin bank parser compiled independently and the supplied HDFC message was tested: it resolves to `BalanceInfo(bank=HDFC Bank, account=9565, currency=INR, amount=25348.33)`.
- `ENBDX` sender recognition was tested successfully.
- A full Android Gradle build was not run in this workspace because Gradle and the Android SDK are not installed here. Open the `moneytracker` folder in Android Studio or run `gradle :app:assembleDebug` in an Android build environment to produce the APK.


# V50 follow-up implementation

Changes in this working snapshot:

1. ZIP backup now includes every app SharedPreferences XML file (including capture/notification preferences), the Room database, and the installed on-device model file when present.
2. Added ZIP restore from a user-selected file. Restore validates the database and settings entries, replaces the database and saved preferences, restores the model when included, and asks the user to reopen the app.
3. The custom parser screen now shows editable amount, currency, merchant, date, and last-four account fields after a sample is pasted. Currency recognition covers AED/DHS, INR/Rs., USD, EUR, GBP and common symbols. Rs. amounts normalize to INR in the parser.
4. Bills & Payments now has top-level Bills and Credit Card Payments filters; the existing Pending/Paid/All filters remain under Bills.
5. A downward swipe on Home or Transactions runs maintenance and exchange-rate refresh, then reloads the displayed data.

Validation: `BankParser.kt` compiled independently with Kotlin 2.x and the Kotak sample `Sent Rs.349.00 from Kotak Bank A/c X8023 to Jio Prepaid Recharge on 07-11-26...` was parsed as INR 349.00, debit, merchant Jio Prepaid Recharge, account ending 8023, and 7 Nov 2026. A full Android `assembleDebug` build could not be run here because this environment has no Gradle installation or Android SDK. Please open the project in Android Studio and run :app:assembleDebug before installing.


# V52 follow-up implementation

1. **Pull-to-Refresh on Home and Transactions:** Wrapped the Home (`Dashboard`) and Transactions (`Transactions`) scrollable lists in Material 3 `PullToRefreshBox` so swiping down from the top displays the pull-to-refresh indicator, imports recent SMS messages (when permission is granted), runs duplicate/bill reconciliation maintenance, refreshes exchange rates, and reloads UI state.
2. **Automatic Bill Icon Search & Display:** Added `BillIconAvatar` next to each bill in `BillRow` (shown in both Bills & Payments and Home upcoming bills). It resolves known regional/global billers, utilities, telecoms, banks, and subscriptions, dynamically queries Clearbit company autocomplete / Google Favicons for unknown billers, and falls back to semantic Material 3 vector icons and themed containers when offline or for generic reminders.


# V53 follow-up implementation

1. **Duplicate Bill Prevention (`BillPayments.kt`, `Capture.kt`, `Maintenance.kt`, `MainActivity.kt`):**
   - Added `BillPayments.isDuplicateBill()` and `BillPayments.insertIfUnique()`, checking normalized biller keys, card last-4 digits, amounts, due dates (±3 days), and raw text before inserting.
   - Added `deduplicateBills()` to `TransactionMaintenance.run()` so duplicate bills are cleaned up automatically.
2. **Live Current-Month Cash Wallet with Negative Balance Support (`Entities.kt`, `MainActivity.kt`, `Capture.kt`):**
   - Removed cached `Prefs.saveCashBalance` updates; Cash Wallet now loads only current month transactions (`monthRange(0)`) and computes balance live from withdrawals/top-ups minus cash spending.
   - Deleting a cash withdrawal or recording cash spending without a withdrawal dynamically updates the balance and supports negative balances.
3. **Notification Navigation & Morning 3-Day Bill Reminder (`Reminders.kt`, `AutoBackup.kt`, `Prefs.kt`, `MainActivity.kt`):**
   - Tapping transaction or daily spending notifications opens the **Transactions** tab; tapping bill reminder notifications opens the **Bills & Payments** tab.
   - Added `BillReminderWorker` for pending bills due within 3 days, with an enable/disable toggle and morning time picker in **Settings → Appearance**.


# V54 follow-up implementation

1. **Transactions Search with Amount & Date Range (`MainActivity.kt`):**
   - Expanded the floating search bar in Transactions to support Min/Max amount range inputs, inline range expressions (e.g., `50-200`, `>100`, `<500`), and quick access to the Custom Date Range picker.
2. **Monthly Budget Category Glance Note & Inline Budget Edit (`MainActivity.kt`):**
   - Tapping any category under **Category budgets** on the Home screen now opens a monthly glance note listing all transactions for that category in the current month.
   - The top-right badge displays `Used / Set` (`Spent / Budget`) and tapping it opens an inline editor to update that category's monthly budget and recalculate the total budget immediately.
3. **Bills Countdown Badge Moved to Far Right (`MainActivity.kt`):**
   - Moved the reminder days / status badge (`Days`, `Today`, `Late`, `Paid`) to the far right end of `BillRow` for a cleaner layout.
4. **Accounts Settings Currency Dropdown & Live Exchange Rate Display (`MainActivity.kt`):**
   - Replaced the plain text currency field in **Settings → Accounts** with an `ExposedDropdownMenuBox` populated from currencies detected across accounts, transactions, and bills.
   - Added a dedicated card displaying live exchange rates (`1 X = Y Main` and `1 Main = Z X`) for all detected account currencies.


# V55 follow-up implementation

1. **Automatic Bill & Card Payment Matching with Confirmation Notification (`BillPayments.kt`, `AutoBackup.kt`, `Capture.kt`, `MainActivity.kt`, `Maintenance.kt`):**
   - Whenever a transaction is captured (SMS or bank notification), pasted, manually added, or edited, `BillPayments.reconcileTransactionWithBills()` checks if the transaction amount/currency matches any pending bill or credit card payment.
   - Matching bills are automatically marked `PAID` with linked transaction evidence (and next month's recurring bill is scheduled when applicable), and a confirmation notification (`TransactionNotice.showBillPaidConfirmation` / `showCardPaymentConfirmation`) is sent that deep-links to **Bills & Payments**.
2. **Account & Card Name Editing with On-the-Fly Bank Icon Pulling (`MainActivity.kt`, `Prefs.kt`):**
   - Both custom and detected accounts/cards now support editing their display name alongside their balance.
   - Added `BankIconAvatar`, `knownBanksCatalog`, and `searchBankLogoOnTheFly()`: choosing a bank from the dropdown or typing a bank name in Add/Edit Account immediately pulls and previews the bank's logo/icon on the fly.
3. **All 4 Additional Transactions Enhancements (`MainActivity.kt`, `Prefs.kt`):**
   - **Multi-Select Bulk Actions (`Select` chip):** Select multiple transactions to bulk-assign categories or delete in one tap.
   - **Merchant Grouping (`Merchants` chip):** Groups expenses in the selected period by merchant, showing visit count, average spend, and total spent.
   - **Recurring Expense Detection (`Recurring` chip):** Automatically detects recurring merchants across months and offers a one-tap `+ Bill reminder` button.
   - **Personal Transaction Notes:** Added a personal note/warranty/memo field in `EditTransactionDialog` and displayed it inline on `TxRow`.


# V56 compile-fix

- Moved `val categoryBudgets = remember(budgetRefresh, showBudgetEditor, monthReviewOpen) { Prefs.categoryBudgets(ctx) }` below the declarations of `budgetRefresh`, `showBudgetEditor`, and `monthReviewOpen` in `Dashboard` (`MainActivity.kt`) to fix the forward-reference compile error at line 990.
