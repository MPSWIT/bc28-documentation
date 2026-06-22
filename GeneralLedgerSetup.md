# General Ledger Setup (Page 118, Table 98)

> **Source:** Base Application · Namespace `Microsoft.Finance.GeneralLedger.Setup`
> **PageType:** Card · **UsageCategory:** Administration
> **Table 98** is a singleton table (one record per company) — the system creates it automatically if it doesn't exist.

## Overview

The **General Ledger Setup** page is the central configuration hub for all financial accounting in Business Central. It controls posting date ranges, currency precision, VAT handling, dimension management, background posting, financial reporting defaults, payment tolerances, and journal template assignments. Changes here affect every financial transaction in the system.

---

## Fields by Section

### General

| Field | Type | Description | OnValidate Behavior |
|-------|------|-------------|---------------------|
| **Allow Posting From** | Date | Earliest date allowed for posting to the company books. | Clears the corresponding DateFormula if a date is set; calls `CheckAllowedPostingDates(0)` and `CheckDateRange()`. |
| **Allow Posting To** | Date | Latest date allowed for posting to the company books. | Same as above. |
| **Allow Posting From DateFormula** | DateFormula | Dynamic formula (relative to today) for earliest posting date. | — |
| **Allow Posting To DateFormula** | DateFormula | Dynamic formula (relative to today) for latest posting date. | — |
| **Allow Deferral Posting From** | Date | Earliest date for deferral postings. | Calls `CheckAllowedDeferralPostingDates(0)`. |
| **Allow Deferral Posting To** | Date | Latest date for deferral postings. | Calls `CheckAllowedDeferralPostingDates(0)`. |
| **VAT Reporting Date Usage** | Enum | Controls whether VAT Date is used: Disabled, Posting Date, or Document Date. | If set to **Disabled**, resets Default VAT Reporting Date to Posting Date. Logs feature telemetry. |
| **Default VAT Reporting Date** | Enum | The date used for VAT report period assignment (Posting Date or Document Date). | Visible only when VAT Reporting Date Usage ≠ Disabled. |
| **Register Time** | Boolean | Records the time users spend logged in (can be overridden per user in User Setup). | — |
| **Local Address Format** | Option | Print format: Post Code+City / City+Post Code / City+County+Post Code / Blank Line+Post Code+City. | — |
| **Local Cont. Addr. Format** | Option | Position of contact name: First / After Company Name / Last. | — |
| **Req.Country/Reg. Code in Addr.** | Boolean | If enabled, clearing Country/Region Code also clears Post Code, City, and County. | — |
| **Inv. Rounding Precision (LCY)** | Decimal | Interval for rounding invoice amounts in local currency (e.g., 0.05). | Validated against **Amount Rounding Precision** — must be a multiple of it. |
| **Inv. Rounding Type (LCY)** | Option | Nearest / Up / Down. | — |
| **Amount Rounding Precision (LCY)** | Decimal | Interval for rounding **all** amounts in LCY. Default: 0.01. | Re-rounds Inv. Rounding Precision to this precision; validates it's not a rounding error source. |
| **Amount Decimal Places (LCY)** | Text[5] | Display decimal places for amounts. Format `min:max` or fixed number. Default: `2:2`. | Validated by `CheckDecimalPlacesFormat()`. |
| **Unit-Amount Rounding Precision (LCY)** | Decimal | Rounding interval for unit prices. Default: 0.00001. | — |
| **Unit-Amount Decimal Places (LCY)** | Text[5] | Display decimal places for unit prices. Default: `2:5`. | Same format validation as Amount Decimal Places. |
| **Allow G/L Acc. Deletion Before** | Date | G/L accounts with entries **on or after** this date require confirmation before deletion. Only applies when **Block Deletion** = No. | — |
| **Block Deletion of G/L Accounts** | Boolean | Prevents deletion of G/L accounts that have ledger entries. Default: **Yes**. | — |
| **Check G/L Account Usage** | Boolean | Protects G/L accounts used in setup tables from being deleted. | — |
| **Mark Cr. Memos as Corrections** | Boolean | Automatically marks new credit memos as corrective entries for VAT/reporting compliance. | — |
| **Pmt. Disc. Excl. VAT** | Boolean | Calculate payment discount on amounts **excluding** VAT. | If enabled, forces **Adjust for Payment Disc.** = No. If disabled, forces **VAT Tolerance %** = 0. |
| **Adjust for Payment Disc.** | Boolean | Recalculates VAT when posting payments that trigger discounts. Default: **Yes**. | If enabled, forces **Pmt. Disc. Excl. VAT** = No and VAT Tolerance % = 0. If disabled, checks that no VAT Posting Setup has "Adjust for Payment Discount" enabled. |
| **Unrealized VAT** | Boolean | Enables cash-basis VAT (VAT recognized upon payment, not invoice). | If disabled, checks VAT Posting Setup and Tax Jurisdiction for active Unrealized VAT Type ≥ Percentage. Auto-sets **Prepayment Unrealized VAT** accordingly. |
| **Prepayment Unrealized VAT** | Boolean | Enables unrealized VAT on prepayments. Default: **Yes**. | Cannot disable if Unrealized VAT is enabled. |
| **Max. VAT Difference Allowed** | Decimal | Maximum VAT correction amount allowed in LCY. | Must be a multiple of Amount Rounding Precision; stored as absolute value. |
| **Tax Invoice Renaming Threshold** | Decimal | If a sales/service invoice exceeds this amount, the document is renamed to "Tax Invoice". | **Hidden** by default (Visible = false). |
| **VAT Rounding Type** | Option | Nearest / Up / Down — how VAT is rounded in LCY. | — |
| **Control VAT Period** | Enum | Block posting within closed / Warn for released / Block posting within closed / Warn when posting / Disabled. Controls posting behavior against VAT Return Periods. | Logs feature telemetry. |
| **Bank Account Nos.** | Code[20] | Number series for auto-assigning Bank Account numbers. | — |
| **Bill-to/Sell-to VAT Calc.** | Enum | Where the VAT Bus. Posting Group on orders comes from: Bill-to/Pay-to No. or Sell-to/Buy-from No. | — |
| **Print VAT specification in LCY** | Boolean | Prints an extra VAT specification in LCY on foreign-currency documents. | — |
| **Show Amounts** | Option | Amount Only / Debit/Credit Only / All Amounts — controls which amount columns appear in journals and ledger entries. | — |
| **Hide Payment Method Code** | Boolean | Hides the Payment Method Code field in sales/purchase documents. | — |
| **Hide Company Bank Account** | Boolean | Controls whether Company Bank Account can be updated on posted documents. | — |
| **Posting Preview Type** | Enum | Standard (grouped by entry type) or Extended (G/L + VAT detail). | — |
| **SEPA Non-Euro Export** | Boolean | Use SEPA export for non-EUR currencies. | — |
| **SEPA Export w/o Bank Acc. Data** | Boolean | Allow SEPA export with Bank Branch No. + Bank Account No. instead of IBAN + SWIFT. | — |
| **Journal Templ. Name Mandatory** | Boolean | Requires journal template/batch names on G/L transactions. | **Hidden** (BE localization). Toggles Journal Templates group visibility. |
| **Enable Data Check** | Boolean | Validates data entered in documents and journals while typing, showing messages in Document/Journal Check FactBox. | — |
| **Check Source Curr. Consistency** | Boolean | Validates that G/L entry source currency totals match during posting. | — |

### Dimensions

| Field | Type | Description | OnValidate Behavior |
|-------|------|-------------|---------------------|
| **Global Dimension 1 Code** | Code[20] | Primary global dimension for analysis (e.g., Department). **Not editable** — use Change Global Dimensions action. | Auto-sets **Shortcut Dimension 1 Code** to the same value. |
| **Global Dimension 2 Code** | Code[20] | Secondary global dimension (e.g., Project). **Not editable** — use Change Global Dimensions action. | Auto-sets **Shortcut Dimension 2 Code** to the same value. |
| **Shortcut Dimension 1 Code** | Code[20] | Quick-access dimension 1, directly editable on journals/lines. | **Not editable** (mirrors Global Dim 1). |
| **Shortcut Dimension 2 Code** | Code[20] | Quick-access dimension 2. | **Not editable** (mirrors Global Dim 2). |
| **Shortcut Dimension 3 Code** | Code[20] | Quick-access dimension 3. | Calls `UpdateDimValueGlobalDimNo()` to update Dimension Set Entry global dimension numbers. |
| **Shortcut Dimension 4 Code** | Code[20] | Quick-access dimension 4. | Same as above. |
| **Shortcut Dimension 5 Code** | Code[20] | Quick-access dimension 5. | Same as above. |
| **Shortcut Dimension 6 Code** | Code[20] | Quick-access dimension 6. | Same as above. |
| **Shortcut Dimension 7 Code** | Code[20] | Quick-access dimension 7. | Same as above. |
| **Shortcut Dimension 8 Code** | Code[20] | Quick-access dimension 8. | Same as above. |

> **Important:** When the page closes and any Shortcut Dimension (3-8) was modified, a `SessionSettings.RequestSessionUpdate(false)` is triggered to refresh dimension visibility across the session.

### Currency

| Field | Type | Description | OnValidate Behavior |
|-------|------|-------------|---------------------|
| **LCY Code** | Code[10] | ISO 3-letter local currency code (e.g., EUR, USD). | Cannot be changed if G/L Entries exist. Auto-fills Symbol and Description if blank. |
| **Local Currency Symbol** | Text[10] | Currency symbol for display (e.g., $). | — |
| **Local Currency Description** | Text[60] | Descriptive name (e.g., "US Dollar"). | — |
| **EMU Currency** | Boolean | Indicates if LCY is an EMU (Eurozone) currency. | — |

### Background Posting

| Field | Type | Description | OnValidate Behavior |
|-------|------|-------------|---------------------|
| **Post with Job Queue** | Boolean | Post documents in the background via job queue. | If disabled, also disables **Post & Print with Job Queue**. |
| **Post & Print with Job Queue** | Boolean | Post **and print** in the background. | If enabled, also enables **Post with Job Queue**. |
| **Job Queue Category Code** | Code[10] | Category for organizing posting job queue entries. | — |
| **Notify On Success** | Boolean | Sends a notification when background posting completes. | — |
| **Report Output Type** | Enum | Output format for scheduled reports (PDF, Excel, Word, Print). | On SaaS, prevents Print and forces PDF. |

### Reporting

| Field | Type | Description | OnValidate Behavior |
|-------|------|-------------|---------------------|
| **Additional Reporting Currency** | Code[10] | Currency for parallel accounting/reporting (e.g., group reporting currency). | Confirmation dialog. If changed to non-blank: opens **Adjust Add. Reporting Currency** batch job. If changed to blank: confirmation. Changes also delete Analysis Views. |
| **VAT Exchange Rate Adjustment** | Enum | How VAT accounts are adjusted during exchange rate adjustments. | — |
| **Acc. Receivables Category** | Integer | G/L Account Category for AR accounts. | — |
| **Acc. Payables Category** | Integer | G/L Account Category for AP accounts. | — |

**Financial Reports Subsection:**

| Field | Description |
|-------|-------------|
| **Balance Sheet Report** | Default Financial Report for Balance Sheet. |
| **Income Statement Report** | Default Financial Report for Income Statement. |
| **Cash Flow Statement Report** | Default Financial Report for Cash Flow Statement. |
| **Retained Earnings Report** | Default Financial Report for Retained Earnings. |
| **Row Definition for Balance Sheet** | Default row definition. |
| **Row Definition for Income Statement** | Default row definition. |
| **Row Definition for Cash Flow Statement** | Default row definition. |
| **Row Definition for Retained Earnings** | Default row definition. |
| **Column Definition for Balance Sheet** | Default column definition. |
| **Column Definition for Net Change** | Default column definition. |
| **Default View by** | Default period type for Financial Reports. |
| **Default Negative Amount Format** | Default negative number format. |
| **Default Company Logo Position** | Default logo placement. |
| **Default Status** | Default Financial Report status. |

### Application

| Field | Type | Description | OnValidate Behavior |
|-------|------|-------------|---------------------|
| **Appln. Rounding Precision** | Decimal | Allowed rounding difference when applying entries across currencies. | — |
| **Pmt. Disc. Tolerance Warning** | Boolean | Show warning when an application occurs between the Pmt. Disc. Date and Tolerance Date. | — |
| **Pmt. Disc. Tolerance Posting** | Option | Post tolerance to Payment Tolerance Accounts or Payment Discount Accounts. | — |
| **Payment Discount Grace Period** | DateFormula | Days a payment can be late and still receive discount. | Confirmation: if accepted, calls `PaymentToleranceMgt.CalcGracePeriodCVLedgEntry()` to recalculate grace periods on open customer/vendor entries. |
| **Payment Tolerance Warning** | Boolean | Show message when a payment is within tolerance, letting user choose how to process. | — |
| **Payment Tolerance Posting** | Option | Post tolerance to Payment Tolerance Accounts or Payment Discount Accounts. | — |
| **Payment Tolerance %** | Decimal | Default % a payment can differ from the invoice. | **Not editable** — use Change Payment Tolerance action. |
| **Max. Payment Tolerance Amount** | Decimal | Default max amount a payment can differ. | **Not editable** — use Change Payment Tolerance action. |
| **App. Dimension Posting** | Option | Dimension source for Realized Gain/Loss application entries. | — |

### Journal Templates (conditional, visible when Journal Templ. Name Mandatory = Yes)

| Field | Description |
|-------|-------------|
| **Adjust ARC Jnl. Template Name** | Journal template for Adjustment of Additional Reporting Currency. |
| **Adjust ARC Jnl. Batch Name** | Journal batch for ARC adjustments. |
| **Apply Jnl. Template Name** | Journal template for applying customer/vendor entries. |
| **Apply Jnl. Batch Name** | Journal batch for applying entries. |
| **Job WIP Jnl. Template Name** | Journal template for Project WIP to G/L. |
| **Job WIP Jnl. Batch Name** | Journal batch for Project WIP. |
| **Bank Acc. Recon. Template Name** | Journal template for bank account reconciliation. |
| **Bank Acc. Recon. Batch Name** | Journal batch for bank reconciliation. |

> All batch fields validate that a template is selected first (via `TestField`).

### Payroll Transaction Import (hidden)

| Field | Description |
|-------|-------------|
| **Payroll Trans. Import Format** | Data Exchange Definition for importing payroll files into General Journal. |

---

## Actions

### Processing

#### Change Global Dimensions

Opens the **Change Global Dimensions** page (Page 577), which allows replacing one or both global dimensions (e.g., switching from "Department/Project" to "Cost Center/Division").

**How it works:**
1. The page shows current Global Dimension 1/2 codes and lets you select new ones.
2. You can choose **Sequential** (run in current session, blocks other users) or **Parallel** (background job queue) processing.
3. The process updates every table containing dimension fields — G/L Entries, Customer Ledger Entries, Vendor Ledger Entries, Item Ledger Entries, FA Ledger Entries, Bank Account Ledger Entries, all posted sales/purchase documents, jobs, resources, inventory, assemblies, reminders, finance charges, and more.
4. The parallel mode has a **Prepare** step that populates a log with affected tables, then a **Start** step that runs the update via job queue.
5. After a parallel start, the user must sign out and back in to ensure no locks conflict with the background job.
6. A **Reset** action allows canceling a prepared change before execution.

**Permissions required:** TableData Dimension = Modify, plus read/write on Change Global Dim. Log Entry.

#### Change Payment Tolerance

Runs the **Change Payment Tolerance** report (Report 34), which updates payment tolerance values.

**How it works:**
1. A request page lets you choose:
   - **All Currencies** — apply the same tolerance to LCY and all foreign currencies
   - **Currency Code** — target a single currency or LCY (blank = LCY)
   - **Payment Tolerance %** — new percentage
   - **Max. Pmt. Tolerance Amount** — new maximum amount
2. When executed, the report updates:
   - **Currency** table — Payment Tolerance % and Max. Payment Tolerance Amount for each selected currency
   - **General Ledger Setup** — the global default values
3. After updating setup values, it asks: *"Do you want to change all open entries for every customer and vendor that are not blocked?"*
4. If confirmed, it iterates through all **Customer Ledger Entries** (open invoices/credit memos) and **Vendor Ledger Entries**, recalculating each entry's `Max. Payment Tolerance` based on `Remaining Amount × Payment Tolerance % / 100`, capped by the max amount. Entries are only updated if the customer/vendor doesn't have **Block Payment Tolerance** enabled.

**Permissions required:** Read/Modify on Currency, Cust. Ledger Entry, Vendor Ledger Entry, General Ledger Setup.

### Navigation Actions

These actions open related setup pages:

| Action | Opens | Description |
|--------|-------|-------------|
| **Accounting Periods** | Page "Accounting Periods" | Set up fiscal year accounting periods. |
| **Dimensions** | Page "Dimensions" | Set up all dimensions (Department, Project, etc.). |
| **User Setup** | Page "User Setup" | Restrict user posting permissions. |
| **Cash Flow Setup** | Page "Cash Flow Setup" | Configure cash flow forecast accounts. |
| **Bank Export/Import Setup** | Page "Bank Export/Import Setup" | Set up formats for bank statement import and payment export. |
| **General Posting Setup** | Page "General Posting Setup" | Set up Gen. Business × Gen. Product posting group combinations to G/L accounts. |
| **Gen. Business Posting Groups** | Page "Gen. Business Posting Groups" | Trade-type posting groups (Domestic, EU, Export, etc.). |
| **Gen. Product Posting Groups** | Page "Gen. Product Posting Groups" | Item-type posting groups (Retail, Wholesale, etc.). |
| **VAT Posting Setup** | Page "VAT Posting Setup" | Set up VAT Bus. × VAT Prod. posting group combinations to G/L accounts. |
| **VAT Business Posting Groups** | Page "VAT Business Posting Groups" | Trade-type VAT posting groups. |
| **VAT Product Posting Groups** | Page "VAT Product Posting Groups" | Item-type VAT posting groups. |
| **VAT Report Setup** | Page "VAT Report Setup" | Number series and options for VAT reports. |
| **Bank Account Posting Groups** | Page "Bank Account Posting Groups" | Posting groups for bank account transactions. |
| **General Journal Templates** | Page "General Journal Templates" | Templates and batches for general journals. |
| **VAT Statement Templates** | Page "VAT Statement Templates" | VAT statement report templates. |

---

## Page Triggers (Codeunit-less Logic)

### OnOpenPage
- Resets and retrieves the singleton record; inserts it if it doesn't exist.
- Stores the original record in `xGeneralLedgerSetup` for change detection.
- Sets journal template visibility based on "Journal Templ. Name Mandatory".
- Initializes Financial Report defaults feature flag.

### OnClosePage
- If any Shortcut Dimension (3-8) was modified, requests a session update via `SessionSettings.RequestSessionUpdate(false)` to refresh dimension visibility across all open pages.

### OnValidate: Additional Reporting Currency (page-level)
- Shows a confirmation dialog before changing or clearing the Additional Reporting Currency.
- Warning when clearing: *"Future G/L entries will be posted in LCY only. Deleting ARC does not affect already posted entries."*
- Warning when changing: *"A batch job will open to recalculate already posted entries in the new currency. Entries will be deleted in Analysis Views."*

### OnValidate: Payment Discount Grace Period (page-level)
- Asks for confirmation, then calls `PaymentToleranceMgt.CalcGracePeriodCVLedgEntry()` to update grace periods on all open customer/vendor entries.

---

## Promoted Actions (Action Bar)

The action bar groups actions into categories:

| Category | Actions |
|----------|---------|
| **Process** | Change Payment Tolerance, Change Global Dimensions |
| **Posting** | General Posting Setup, Gen. Business Posting Groups, Gen. Product Posting Groups |
| **General** | Accounting Periods, Dimensions, User Setup, Cash Flow Setup |
| **VAT** | VAT Statement Templates, VAT Posting Setup, VAT Business Posting Groups, VAT Product Posting Groups, VAT Report Setup |
| **Bank** | Bank Export/Import Setup, Bank Account Posting Groups |
| **Journal Templates** | General Journal Templates |

---

## Key Validation Rules (Table Triggers Summary)

- **Allow Posting From/To:** Cross-validates date ranges and clears DateFormula when a fixed date is entered.
- **Unrealized VAT:** Cannot disable if VAT Posting Setup or Tax Jurisdiction has Unrealized VAT Type ≥ Percentage.
- **Prepayment Unrealized VAT:** Cannot disable while Unrealized VAT is enabled.
- **Pmt. Disc. Excl. VAT ⇄ Adjust for Payment Disc.:** These are mutually exclusive; one forces the other off.
- **LCY Code:** Cannot change after G/L Entries exist.
- **Additional Reporting Currency:** Confirmation required; when changed, triggers batch job for recalculation and deletes Analysis Views.
- **Amount Rounding Precision:** Must be non-zero; re-rounds Invoice Rounding Precision.
- **Shortcut Dimensions 3-8:** On change, update Dimension Set Entry global dimension numbers via `UpdateDimValueGlobalDimNo()`.
- **Report Output Type:** On SaaS, PDF is the only allowed output type (Print is blocked). -->
