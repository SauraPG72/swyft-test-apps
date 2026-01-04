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
3. **Allows re-categorisation** - User can correct transaction categories
4. **Provides meaningful analysis** - Cash flow patterns, income consistency, expense ratios

## Technical Requirements

### Must Have
- [ ] Parse illion JSON/HTML format (sample provided)
- [ ] Multi-column account display (side-by-side)
- [ ] Transaction categorisation with ability to override
- [ ] Basic income/expense summary per account
- [ ] Search and filter functionality

### Should Have
- [ ] Automatic interbank transfer detection (matching amounts/dates)
- [ ] Visual linking of related transactions across accounts
- [ ] Income consistency analysis (regular vs irregular)
- [ ] Export summary for loan applications
- [ ] Responsive design for different screen sizes

### Nice to Have
- [ ] Trend visualisation (charts/graphs)
- [ ] Anomaly detection (unusual transactions)
- [ ] Multiple statement upload and merge
- [ ] Annotation system for broker notes
- [ ] Direct comparison against lending criteria

## Resources Provided

```
resources/
├── statements/       # Sample illion exports (HTML + JSON)
├── sample-data.json  # Example data structure
└── test-cases.md     # Scenarios to validate against

reference/
└── (current tool screenshots for reference)
```

## Data Format

illion provides bank statements in structured format:

```json
{
  "accounts": [
    {
      "accountName": "Business Transaction",
      "bsb": "063-000",
      "accountNumber": "12345678",
      "openingBalance": 5432.10,
      "closingBalance": 8765.43,
      "transactions": [
        {
          "date": "2024-01-15",
          "description": "PAYMENT FROM CUSTOMER ABC",
          "amount": 1500.00,
          "balance": 6932.10,
          "category": "Income - Business",
          "tags": ["recurring", "verified"]
        }
      ]
    }
  ]
}
```

## Key Challenges

1. **Layout complexity**: Multiple accounts must be viewable simultaneously while remaining readable
2. **Transfer matching**: Same amount transferred between accounts may have different dates (banking delays), different descriptions
3. **Category accuracy**: Initial categories are often wrong; system should learn from corrections
4. **Performance**: Statements can have 1000+ transactions; must remain responsive

## Validation Scenarios

Your tool passes validation when a broker can:

1. Open a statement with 4+ accounts and view all side-by-side
2. Find a $1,000 transfer between accounts in under 5 seconds
3. Re-categorise a transaction and see updated summaries
4. Generate a summary showing total income by category

## Getting Started

1. Review sample data structure in resources/
2. Study the test cases to understand expected functionality
3. Build the parser first, then the display, then analysis
4. Test with increasingly complex scenarios

## Questions to Consider

- How will you handle statements with 10+ accounts?
- What's your matching algorithm for interbank transfers?
- How will you persist category overrides?
- What summary metrics are most valuable for loan assessment?
