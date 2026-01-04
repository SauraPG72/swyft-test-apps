# Bank Statement Test Cases

## Overview

Each test case provides bank statement data for a different fictional person/business. Test cases have varying complexity to evaluate how well the tool handles real-world scenarios.

## Test Case 1: Simple Single Account

**Files:** `statements/case-1-simple.html`, `statements/case-1-simple.json`
**Period:** 90 days
**Accounts:** 1 (Business transaction account)
**Transactions:** ~50

**Scenario:**
A sole trader with one business account. Regular weekly income from 3-4 clients, standard operating expenses.

**Expected Analysis:**
- Average weekly income: ~$4,500
- Regular income sources: 3 identified
- Major expense categories: Rent, Utilities, Materials
- Income consistency: High (weekly pattern)

---

## Test Case 2: Personal + Business Mix

**Files:** `statements/case-2-mixed.html`, `statements/case-2-mixed.json`
**Period:** 90 days
**Accounts:** 3 (1 business transaction, 1 personal, 1 credit card)
**Transactions:** ~200

**Scenario:**
Small business owner who sometimes mixes personal and business transactions. Some business income deposited to personal account. Credit card used for both.

**Expected Analysis:**
- Clear separation of business vs personal income
- Identification of business expenses on personal accounts
- Credit card business usage percentage
- Warning flags for mixed usage

**Interbank Transfers to Detect:**
- 4 transfers between business and personal
- 2 credit card payments from business account

---

## Test Case 3: Complex Business Structure

**Files:** `statements/case-3-complex.html`, `statements/case-3-complex.json`
**Period:** 90 days
**Accounts:** 6 (2 operating, 1 tax savings, 1 equipment reserve, 2 personal)
**Transactions:** ~500

**Scenario:**
Established business with structured account management. Regular transfers between accounts for tax, equipment reserves. Multiple income streams.

**Expected Analysis:**
- Operating account cash flow patterns
- Tax savings discipline (regular percentage transfers)
- Equipment reserve usage
- Net business income after internal transfers

**Interbank Transfers to Detect:**
- Weekly transfers to tax account (~12)
- Monthly transfers to equipment reserve (3)
- Irregular transfers to personal (5)
- Equipment purchases from reserve (2)

---

## Test Case 4: Irregular Income Pattern

**Files:** `statements/case-4-irregular.html`, `statements/case-4-irregular.json`
**Period:** 180 days (extended period)
**Accounts:** 2 (business, savings)
**Transactions:** ~150

**Scenario:**
Project-based business with lumpy income. Large payments every 4-6 weeks, sometimes significant gaps. Tests ability to identify genuine income vs one-off payments.

**Expected Analysis:**
- Identification of project-based income pattern
- Average monthly income accounting for timing
- Largest income gap duration
- Sufficient runway analysis

---

## Validation Checklist

For each test case, verify:

- [ ] All accounts display correctly side-by-side
- [ ] Transaction count matches expected
- [ ] Opening/closing balances are correct
- [ ] Categories are assigned to all transactions
- [ ] Interbank transfers are linked
- [ ] Summary totals are accurate
- [ ] Search finds expected transactions
- [ ] Category overrides persist and update summaries

## Edge Cases to Handle

1. **Same-day, same-amount transactions** - Multiple $100 transactions on same day, ensure correct linking
2. **Delayed transfers** - Transfer out on Friday, received Monday - still linked
3. **Split transactions** - $5000 transferred in two parts ($3000 + $2000)
4. **Description mismatches** - "TRANSFER TO SAVINGS" vs "TFR FR TRANSACTION" for same transfer
5. **Negative balance periods** - Account goes into overdraft, correct balance handling
6. **Foreign currency** - Some transactions in USD, handled correctly

## Performance Requirements

- 500 transactions: Page load < 2 seconds
- 2000 transactions: Page load < 5 seconds
- Search results: < 200ms response time
- Category update: < 100ms UI update
