# Option B: Bank Statement Analysis Tool

## The Problem

When assessing loan applications, brokers analyse bank statements to understand a customer's financial position. The current industry tool (illion BankStatements) provides HTML/JSON exports but has significant UX limitations:

1. **Vertical-only layout**: Multiple accounts stacked vertically. Matching interbank transfers (e.g., finding where a $1,000 credit came from) requires manual Ctrl+F hunting.

2. **Poor categorisation**: Transactions often mislabelled. Genuine business income tagged as "Other Credit". No way to correct this.

3. **Limited analysis**: Basic income/expense summaries. No trend analysis, no cash flow timing, no irregular payment detection.

4. **No interactivity**: Static display. Can't filter, can't compare periods, can't annotate for underwriters.

## Your Task

Build a bank statement analysis tool that:

1. **Displays multiple accounts side-by-side** - Horizontal layout showing all accounts simultaneously
2. **Links related transactions** - Automatically detect and highlight interbank transfers
3. **Allows re-categorisation** - User can click and tag line items
4. **Provides meaningful analysis** - Different analysis lenses for Consumer vs Commercial applications
5. **Supports deal management** - Create deals and upload bank statements to track assessments

## Core Features

### 1. File Upload & Parsing
- Upload illion HTML or JSON bank statement exports
- Parse and display transaction data
- Support multiple file uploads per deal

### 2. Multi-Account Side-by-Side View
- Display all accounts horizontally in columns
- Synchronised scrolling for comparing transactions across accounts
- Visual highlighting of related transactions (interbank transfers)

### 3. Transaction Tagging System
Users should be able to click on transactions and tag them with categories:

**Consumer Tags:**
- Primary Income / Salary
- Centrelink Benefits
- Loan Repayments
- Debt Collections
- Gambling
- ATM Withdrawals
- Large Transfers
- Recurring Expenses

**Commercial Tags:**
- Genuine Revenue
- Interbank Transfers
- Loan Repayments
- ATO Payments
- Owner Drawings
- Wages Paid
- Debt Collections
- Significant Expenses (5-digit+)

### 4. Analysis Lens - Consumer vs Commercial

The tool should support two analysis modes with different assessment criteria:

#### Consumer Analysis Lens
| Question | What to Look For |
|----------|------------------|
| Primary Income | Salary/wage frequency, amount, employer consistency |
| Are declared expenses true? | Match against application declarations |
| Any income subsidies not declared? | Centrelink, Family Benefits, Carers payments |
| Signs of good conduct? | Regular repayments, savings behaviour |
| Any debt collections? | Recovery agencies, collection notices |
| Bad account conduct? | Gambling, excessive withdrawals, dishonours, days in negative |
| Arrears, knockbacks, days in negative? | Payment failures, negative balance periods |
| Any accounts not declared? | Hidden savings, undisclosed accounts |
| Large payments to/from? | Significant transfers requiring explanation |
| Consistent/recurring expenses? | International transfers, insurances, loan repayments |

#### Commercial Analysis Lens
| Question | What to Look For |
|----------|------------------|
| Revenue Per Month? | Total business income, breakdown by source |
| Genuine revenue and days? | Real trading income vs interbank transfers |
| Last Month Revenue? | Most recent month's performance |
| Any interbank transfers inflating credits? | Circular transfers between related accounts |
| Any 5-digit or significant expenses/debts? | Large payments, supplier invoices |
| Most significant debtor? | Largest customer paying the business |
| Any debt collections? | Recovery agencies indicating credit issues |
| Bad account conduct? | ATM withdrawals, gambling, dishonours |
| Arrears, knockbacks, days in negative? | Cash flow stress indicators |
| Current Loans? | Existing finance commitments |
| ATO payments? | Tax obligations, payment patterns |
| Savings taken, owner drawings? | Money withdrawn from business |

### 5. Deal Management
- Create a "Deal" to group related bank statement analyses
- Upload multiple statements to a single deal
- Track assessment progress per deal
- Generate summary reports

## Business Value

The goal is to significantly reduce **TTR (Time to Write)** - the time it takes a broker to assess bank statements and write up the loan application. By providing:

- Side-by-side account comparison
- Quick tagging and categorisation
- Structured analysis checklists
- Pre-formatted assessment outputs

## Technical Requirements

### Must Have
- [ ] Parse illion JSON/HTML format (sample provided)
- [ ] Multi-column account display (side-by-side)
- [ ] Transaction tagging with click-to-tag functionality
- [ ] Consumer vs Commercial analysis mode toggle
- [ ] Basic income/expense summary per account
- [ ] Search and filter functionality

### Should Have
- [ ] Automatic interbank transfer detection (matching amounts/dates)
- [ ] Visual linking of related transactions across accounts
- [ ] Income consistency analysis (regular vs irregular)
- [ ] Deal management system
- [ ] Export summary for loan applications
- [ ] Responsive design for different screen sizes

### Nice to Have
- [ ] Trend visualisation (charts/graphs)
- [ ] Anomaly detection (unusual transactions, gambling patterns)
- [ ] Multiple statement upload and merge
- [ ] Annotation system for broker notes
- [ ] Direct comparison against lending criteria

## Resources Provided

```
resources/
├── statements/           # Sample illion exports (HTML + JSON)
│   ├── a-t/             # Consumer - 3 bank accounts (ANZ, Bendigo, QCCU)
│   ├── t-k-pty-ltd/     # Commercial - 1 bank account (CBA)
│   ├── h-c-pty-ltd/     # Commercial - 1 bank account (Westpac)
│   └── j-v-w/           # Consumer - 1 bank account (ING)
└── test-cases.md        # Scenarios to validate against
```

## Sample Data Analysis Notes

The following are example assessments for each sample statement, demonstrating the kind of analysis the tool should facilitate:

---

### A-T (Consumer - 3 Accounts)

**Account Summary:** ANZ Transaction, Bendigo Home Loan, QCCU Savings

#### Assessment

| Question | Analysis |
|----------|----------|
| **Primary Income** | Monthly average: $7,375.30 from AUSTRALIAN COUNT (fortnightly pay cycle) |
| **Are declared expenses true?** | To be verified against application |
| **Any income subsidies not declared?** | No subsidies detected |
| **Signs of good conduct?** | Yes - consistent loan repayments being made |
| **Any debt collections?** | No debt collections found |
| **Bad account conduct?** | No gambling, no excessive withdrawals |
| **Arrears/knockbacks/days in negative?** | None |
| **Undeclared accounts?** | Account 64277377 receives transfers - possible additional savings |
| **Large payments to/from** | $6,530.00 transfer to SAV 64277377 - may be savings, not expense |
| **Recurring expenses** | FMC Direct Debit: $1,765.86/month, Yamaha Motor Finance: $276.30/month |

**Notes:** Finances spread across 3 banks. Credits from other accounts make reconciliation complex.

---

### T-K-PTY-LTD (Commercial - 1 Account)

**Account Summary:** CBA Business Transaction Account

#### Assessment

| Question | Analysis |
|----------|----------|
| **Revenue Per Month** | To be calculated from transactions |
| **Genuine revenue and days?** | To be verified |
| **Last Month Revenue** | To be calculated |
| **Interbank transfers?** | Check for circular transfers |
| **5-digit expenses?** | Review large payments |
| **Most significant debtor?** | Identify largest customer |
| **Debt collections?** | Check for recovery agencies |
| **Account conduct?** | Review for red flags |
| **Current Loans?** | Identify existing finance |
| **ATO payments?** | Check tax payment patterns |
| **Owner drawings?** | Identify personal withdrawals |

---

### H-C-PTY-LTD (Commercial - 1 Account)

**Account Summary:** Westpac Business One Plus - 180 days - 0 days negative - $102k available

#### Assessment

| Question | Analysis |
|----------|----------|
| **Revenue Per Month** | Hume City Council: $180,294 avg pm, Downer: $10,431 avg pm, Other: $85,009 avg pm |
| **Genuine revenue (last 30 days)** | Hume City Council: $264,754, Downer: $6,479, Other: $1,738 |
| **Last Month Revenue** | Oct: Hume City Council $235,087, Downer $6,479, Other $3,388 |
| **Interbank transfers?** | Check RS Trading and Hume Contracting NAB transfers |
| **5-digit expenses** | Barling's Wood ($15k Jun, $10k Aug), Bluedog Fences ($18k Oct), Lokan Nominees ($13k Jun, $16k Aug), Rent $2,166 pm, Yard Rent $6,555 pm |
| **Most significant debtor** | HUME CITY COUNCIL |
| **Debt collections** | Recoveries Corp $1,888 (Jun), ARMA $484 (Jul) |
| **ATO payments** | $7,467 avg pm - latest $7,071 (Sep for Jun quarter) |
| **Owner drawings** | RS Trading transfers: $75k-$150k multiple payments, Hume Contracting NAB: $90k-$125k transfers |

**Additional Notes - Related CBA Account:**
- Ignite Limited wages: $28,540 avg pm
- WB Hunter wages: $3,689 avg pm
- Shift Financial: $431.18 pw (large settlement payments in May-Jul)
- BMW Finance: $2,767 pm
- MB Finance: $5,257 pm
- Arteva Funding: $5,548 pm
- ATM withdrawals: $2,800 avg pm

---

### J-V-W (Consumer - 1 Account)

**Account Summary:** ING Transaction Account - 0 days in negative

#### Assessment

| Question | Analysis |
|----------|----------|
| **Primary Income** | Comsol wages: $1,957.54, Centrelink Family Benefits: $1,673.96, Centrelink Pension: $1,084.99, Carers Benefit: $682.71, 2x DH: $167.03 |
| **Are declared expenses true?** | Yes |
| **Income subsidies not declared?** | No - opposite issue: Centrelink not visible in this account |
| **Signs of good conduct?** | Yes - strong savings behaviour |
| **Debt collections?** | None |
| **Bad account conduct?** | Minimal - $200 ATM (one-time $600) |
| **Arrears/days in negative?** | None |
| **Undeclared accounts?** | None found |
| **Large payments** | Adyen $18,835.95 (10 Dec) - requires explanation |
| **Non-SACC Loans** | Mortgage: $1,500, Plenti PL: $301.65 |

**Note:** Additional account BSB 633 123 / Account 196622807 shows Cleanaway salary $947 but Centrelink/Comsol/Child Support not visible there.

---

## Data Format

illion provides bank statements in structured format:

```json
{
  "dataVersion": 20170401,
  "reference": "XXXX",
  "submissionTime": "2025-12-27T08:34:31",
  "bankData": {
    "bankName": "ANZ",
    "bankSlug": "anz",
    "bankAccounts": [
      {
        "id": 0,
        "accountType": "transaction",
        "accountHolder": "Account Holder",
        "accountName": "Account Name",
        "bsb": "XXXXXX",
        "accountNumber": "XXXXXXXXX",
        "currentBalance": "0.86",
        "availableBalance": "0.86",
        "transactions": [
          {
            "date": "2025-12-22",
            "text": "PAYMENT DESCRIPTION",
            "amount": -199,
            "balance": "0.86",
            "type": "General Payment",
            "tags": [{"thirdParty": "Merchant Name"}]
          }
        ]
      }
    ]
  }
}
```

## Key Challenges

1. **Layout complexity**: Multiple accounts must be viewable simultaneously while remaining readable
2. **Transfer matching**: Same amount transferred between accounts may have different dates (banking delays), different descriptions
3. **Category accuracy**: Initial categories are often wrong; tagging system should allow corrections
4. **Performance**: Statements can have 1000+ transactions; must remain responsive
5. **Analysis context**: Consumer vs Commercial require different assessment approaches

## Validation Scenarios

Your tool passes validation when a broker can:

1. Open a statement with 4+ accounts and view all side-by-side
2. Find a $1,000 transfer between accounts in under 5 seconds
3. Tag a transaction (e.g., "Genuine Revenue" or "Gambling") with a single click
4. Switch between Consumer and Commercial analysis modes
5. Generate a summary showing analysis against the assessment checklist
6. Create a deal and upload multiple statements to it

## Getting Started

1. Review sample data structure in `resources/statements/`
2. Study the test cases to understand expected functionality
3. Build the parser first, then the display, then analysis
4. Implement tagging system
5. Add Consumer vs Commercial analysis modes
6. Test with the provided sample data

## Questions to Consider

- How will you handle statements with 10+ accounts?
- What's your matching algorithm for interbank transfers?
- How will you persist tags and category overrides?
- What summary metrics are most valuable for loan assessment?
- How will you differentiate Consumer vs Commercial analysis workflows?
