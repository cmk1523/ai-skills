---
name: cc-statement-extractor
description: Extract structured data from credit card statements (PDF or image) into a JSON file. Use when the user provides a credit card statement and wants transactions, balances, fees, interest, rewards, or other fields extracted to JSON.
---

# Credit Card Statement Extractor

Extracts all key information from a credit card statement into one normalized JSON file per statement.

## Workflow

1. **Read the input.**
   - Text-based PDF: use `pdfplumber` or `pdftotext -layout`.
   - Scanned PDF or image: use OCR or read the page visually.
   - Multi-column layouts (e.g., Amex) need care. Check that columns were not interleaved.
2. **Identify the issuer and card product** from the header or logo.
3. **Extract each section** listed under "Fields to extract".
4. **Normalize** using the rules under "Normalization rules".
5. **Validate** using the checks under "Validation".
6. **Write the output** to `{issuer}_{last4}_{YYYY-MM}.json` (one file per statement).
7. **Report** a short summary to the user: period, new balance, transaction count, and any validation warnings.

## Fields to extract

### 1. Issuer and account
- Issuer name, card product name, card network (Visa, Mastercard, Amex, Discover)
- Account holder name, billing address
- Account number (**last 4 digits only**)
- Statement date, billing period start and end, days in cycle
- Payment due date

### 2. Account summary
- Previous balance
- Payments, other credits
- Purchases, cash advances, balance transfers
- Fees charged, interest charged
- New balance
- Credit limit, available credit, cash advance limit, available cash

### 3. Payment info
- Minimum payment due and due date
- Late fee amount and penalty APR
- Minimum-payment warning (months to payoff, total cost)
- 3-year payoff amount and total cost

### 4. Transactions
For each line item:
- Transaction date, post date
- Raw description and a normalized merchant name
- Amount
- Type: `purchase | payment | credit | fee | interest | cash_advance | balance_transfer`
- Category (if the issuer provides one)
- Cardholder (for authorized users)
- Foreign currency amount, currency code, exchange rate (if present)

### 5. Fees and interest
- Itemized fees: annual, late, foreign transaction, over-limit, returned payment
- Interest charge table: balance type, APR, APR type (fixed or variable), balance subject to interest, interest charged
- Promotional APR and its expiration date
- Year-to-date totals for fees and interest

### 6. Rewards
- Program name and unit (points, miles, cash back)
- Beginning balance, earned (base and bonus), redeemed, adjustments, ending balance

### 7. Miscellaneous
- Promotional offers and deferred-interest plans, with expiration dates
- Installment plans (e.g., Amex Plan It, Chase My Chase Plan)
- Notable notices (change in terms, etc.)

## Output schema

```json
{
  "issuer": "string",
  "card_product": "string",
  "network": "Visa|Mastercard|Amex|Discover|null",
  "account_last4": "string",
  "cardholder_name": "string",
  "billing_address": "string|null",
  "statement": {
    "period_start": "YYYY-MM-DD",
    "period_end": "YYYY-MM-DD",
    "statement_date": "YYYY-MM-DD",
    "days_in_cycle": 30,
    "currency": "USD"
  },
  "summary": {
    "previous_balance": 0.0,
    "payments": 0.0,
    "credits": 0.0,
    "purchases": 0.0,
    "cash_advances": 0.0,
    "balance_transfers": 0.0,
    "fees": 0.0,
    "interest": 0.0,
    "new_balance": 0.0,
    "credit_limit": 0.0,
    "available_credit": 0.0,
    "cash_advance_limit": 0.0
  },
  "payment": {
    "minimum_due": 0.0,
    "due_date": "YYYY-MM-DD",
    "late_fee": 0.0,
    "penalty_apr": 0.0
  },
  "transactions": [
    {
      "transaction_date": "YYYY-MM-DD",
      "post_date": "YYYY-MM-DD|null",
      "description": "string",
      "merchant_normalized": "string|null",
      "amount": 0.0,
      "type": "purchase|payment|credit|fee|interest|cash_advance|balance_transfer",
      "category": "string|null",
      "cardholder": "string|null",
      "foreign": { "amount": 0.0, "currency": "EUR", "fx_rate": 1.08 }
    }
  ],
  "fees": [
    { "date": "YYYY-MM-DD", "type": "string", "amount": 0.0 }
  ],
  "interest_charges": [
    {
      "balance_type": "purchases|cash_advances|balance_transfers",
      "apr": 0.0,
      "apr_type": "fixed|variable",
      "balance_subject": 0.0,
      "interest": 0.0
    }
  ],
  "rewards": {
    "program": "string|null",
    "unit": "points|miles|cash",
    "beginning": 0,
    "earned": 0,
    "redeemed": 0,
    "adjustments": 0,
    "ending": 0
  },
  "year_to_date": { "fees": 0.0, "interest": 0.0 },
  "promotions": [
    { "description": "string", "expires": "YYYY-MM-DD" }
  ],
  "validation": {
    "balance_reconciles": true,
    "transactions_sum_matches_summary": true,
    "warnings": []
  }
}
```

## Normalization rules

- **Sign convention:** purchases, fees, interest, and cash advances are **positive**. Payments and credits are **negative**. Apply this regardless of how the issuer prints them.
- **Dates:** output ISO `YYYY-MM-DD`. If transaction lines omit the year (e.g., `11/03`), infer it from the statement period, and handle the Dec/Jan boundary correctly.
- **Amounts:** numbers, not strings. Strip `$`, commas, and parentheses. Treat `(12.34)` and `12.34-` as negative.
- **Missing data:** use `null`. Never guess or invent values.
- **Percentages:** store APRs as plain percent numbers (e.g., `24.99`, not `0.2499`).
- **Merchant normalization:** strip store numbers, location codes, and processor prefixes (e.g., `AMZN MKTP US*2K4XY` becomes `Amazon Marketplace`). Always keep the raw text in `description`.
- **Privacy:** never write the full card number. Only `account_last4`.

## Validation

Run these checks and record results in `validation`:

1. **Balance reconciles:**
   `previous_balance + purchases + cash_advances + balance_transfers + fees + interest - payments - credits = new_balance` (tolerance: 0.01)
2. **Transactions match summary:** summing transactions by type matches the corresponding summary totals.
3. **Date sanity:** all transaction dates fall within (or just before) the billing period, and `due_date` is after `statement_date`.
4. **Available credit:** `credit_limit - new_balance` is roughly `available_credit`.

If any check fails, set the flag to `false` and add a human-readable message to `warnings`. Do not silently "fix" numbers to make them reconcile. Re-check the extraction instead.

## Issuer notes

- **Chase:** a transaction table with "Account Activity" sections. Rewards box on page 1.
- **Amex:** multi-column layout, so extract carefully. Card member sections may repeat per authorized user.
- **Citi:** the "Account Summary" box is on page 1, and the interest table is near the end.
- **Capital One:** the transaction list is grouped by cardholder.
- **Bank of America:** the "Payment Information" box is at the top right.

Add observed quirks here as new statements are processed.

## Privacy and handling

- Statements contain sensitive personal and financial data. Write output only to the location the user specifies (or the current workspace).
- Do not send statement contents to external services.
- Remind the user to keep output JSON files out of version control (add to `.gitignore`).