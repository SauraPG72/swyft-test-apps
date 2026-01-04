# Multi-Lender Quoting Calculator

## The Problem

Finance brokers work with dozens of lenders, each calculating loan repayments differently. While the underlying math is compound interest, lenders differ in:

- Interest calculation method (daily vs monthly compounding)
- Payment timing (arrears vs advance)
- Fee structures (establishment fees, monthly fees, PPSR)
- **Commission structures** (four distinct models - see below)

Currently, brokers either:
1. Use each lender's individual calculator (slow, repetitive)
2. Use spreadsheets that are error-prone and hard to maintain
3. Use expensive third-party tools that still don't match lender calculations

## Your Task

Build a **full-stack web application** that allows finance brokers to:

1. Calculate loan quotes for multiple lenders
2. Manage their own lender configurations
3. Compare quotes side-by-side

### Tech Stack Requirements

- **Frontend**: Next.js (React)
- **Backend/Database**: Supabase
- **Authentication**: Google OAuth (via Supabase Auth)

---

## Mathematical Specifications

### Variables

| Variable | Description |
|----------|-------------|
| P | Principal (asset price) |
| L | Lender establishment fee |
| O | Origination fee |
| B | Brokerage/commission amount |
| F | Total financed amount |
| r | Annual interest rate (decimal) |
| n | Number of payment periods |
| R | Periodic repayment amount |
| BV | Balloon/residual value |

### Calculation Methods

#### 1. Monthly Compounding (Most Lenders)

**Financed Amount:**
```
F = P + L + O + B
```
Where L, O are included only if financed (not paid upfront), and B is the capitalised brokerage.

**Interest Rates:**
| Type | Formula | Example (8.5% p.a.) |
|------|---------|---------------------|
| Monthly Rate | i = r / 12 | 0.007083 (0.7083%) |
| Daily Rate | d = r / 365 | 0.000233 (0.0233%) |

#### 2. Payment Formulas

| | **No Balloon** | **With Balloon** |
|---|----------------|------------------|
| **Arrears** | R = F × [i(1+i)^n] / [(1+i)^n - 1] | R = [F - BV/(1+i)^n] × [i(1+i)^n] / [(1+i)^n - 1] |
| **Advance** | R = R_arrears / (1 + i) | R = R_arrears_balloon / (1 + i) |

**Notes:**
- **Arrears**: First payment due one period after settlement
- **Advance**: First payment due at settlement (reduces effective principal by one payment)

#### 3. Daily Compounding (Autopay/MoneyMe)

Some lenders calculate interest daily using actual/365 day count:

```
Daily Rate = r / 365
Interest Per Period = Balance × Daily Rate × actual_days_in_period
```

Note: Each payment period has variable interest based on actual calendar days. First payment typically includes +1 day (settlement day counts).

### Fee Structures

**Establishment Fee:**
- Fixed dollar amount (e.g., $495, $550, $990)
- Can be financed (added to loan) or paid upfront

**Origination Fee:**
- Fixed dollar amount (e.g., $790)
- Can be financed (added to loan) or paid upfront

**PPSR Fee:**
- Personal Property Securities Register fee
- Usually $6-8, one-time
- Can be financed or paid upfront

**Monthly Fees:**
- Charged separately from repayment
- Total monthly = R + monthly_fee

---

## Commission Structures

### 1. Capitalised Brokerage
Commission is added to the loan principal. Customer pays interest on it.
```
NAF = Finance Amount + Financed Fees
Commission = Commission% × NAF
Total Financed = NAF + Commission  ← Interest charged on this
Comparison Rate = Rate on NAF that produces same PMT (always higher than base rate)
```

### 2. Commission Overs
Commission is paid from rate differential (markup above base rate).
```
Base PMT = PMT(NAF, Base Rate, Term)
Final PMT = PMT(NAF, Contract Rate, Term)
Hiring Difference = (Final PMT × Term) - (Base PMT × Term)
Commission = MAX(Base Commission, 75% × Hiring Difference × 1.10)
```

### 3. Daily Interest with Rate Adjustment (Autopay/MoneyMe)
Daily interest accrual with monthly payments. Commission capitalised into starting principal.

**Rate Adjustment Formula:**
```
Contract Rate = Base Rate + (Commission% × 0.4)
```
For example: 4% commission adds 1.6% to the base rate (4 × 0.4 = 1.6).

**Daily Interest Calculation:**
```
Daily Rate = Contract Rate / 365
Interest = Balance × Daily Rate × Days in Period
First Payment: +1 day interest (settlement day counts)
```

**Example:**
- Base Rate: 7.35%
- Commission: 4%
- Rate Adjustment: 4 × 0.4 = 1.6%
- Contract Rate: 7.35% + 1.6% = 8.95%

### 4. Brokerage with Factor Rate (Loaded Commission)
Uses a loading factor to partially capitalise commission.
```
Loading = (NAF / (NAF + Commission)) × (1 - 0.4 × FinancierRate)
Amount Financed = NAF + Loading × Commission
Customer Rate = Rate on NAF that produces the calculated PMT
```

---

## Worked Example (Capitalised Brokerage)

**Inputs:**
- Principal Amount: $30,000
- Establishment Fee: $495 (financed)
- Origination Fee: $0
- Monthly Fee: $0
- Annual Rate: 8.5%
- Term: 60 months
- Payment Type: Arrears
- Residual/Balloon: $0
- Commission: 4%

**Step-by-Step Calculation:**

1. **NAF (Net Amount Financed)**
   ```
   NAF = $30,000 + $495 = $30,495.00
   ```

2. **Commission**
   ```
   Commission = $30,495.00 × 0.04 = $1,219.80
   ```

3. **Total Financed**
   ```
   Total Financed = $30,495.00 + $1,219.80 = $31,714.80
   ```

4. **Monthly Rate**
   ```
   i = 0.085 / 12 = 0.00708333
   ```

5. **Monthly Payment (Arrears, No Balloon)**
   ```
   R = $31,714.80 × [0.00708333 × (1.00708333)^60] / [(1.00708333)^60 - 1]
   R = $31,714.80 × [0.00708333 × 1.52699] / [0.52699]
   R = $31,714.80 × 0.02052
   R = $650.68
   ```

6. **Total Repayments**
   ```
   Total = $650.68 × 60 = $39,040.80
   ```

7. **Comparison Rate**
   The comparison rate is the rate that, when applied to NAF alone, produces the same payment.
   Solving: PMT($30,495, comparison_rate, 60) = $650.68
   ```
   Comparison Rate = 10.18% p.a.
   ```

**Summary:**
| Output | Value |
|--------|-------|
| Monthly Payment | $650.68 |
| Total Hiring Installments | $39,040.80 |
| Base Rate | 8.50% |
| Comparison Rate | 10.18% |
| Broker Commission | $1,219.80 |

---

## Application Requirements

### Authentication
- [ ] Google OAuth login via Supabase Auth
- [ ] Protected routes - only authenticated users can access the calculator
- [ ] User-specific lender configurations (each user has their own lenders)

### Core Calculator
- [ ] Input form for loan details (finance amount, term, rate, fees, balloon)
- [ ] Support for different interest calculation methods:
  - Monthly compounding
  - Daily interest calculation
- [ ] Support for payment timing:
  - Advance (first payment at settlement)
  - Arrears (first payment after one period)
- [ ] Output must match lender-generated quotes to the cent

### Quote Comparison & Export
This is a critical feature for broker workflow - the ability to build multiple quotes and export them for client communication.

#### Payment Frequency Options
- [ ] Support for multiple payment frequencies with simple conversion:
  - **Monthly**: Base payment amount
  - **Fortnightly**: Monthly / (26/12) = Monthly × 0.4615
  - **Weekly**: Monthly / (52/12) = Monthly × 0.2308
- [ ] Checkbox toggles to select which frequencies to display
- [ ] Format payment strings like: `$650.68 (monthly) OR $300.31 (fortnightly) OR $150.16 (weekly)`
- [ ] Include monthly fee notation: `(incl. $12.50 monthly fee)` or `(no monthly fees)`

#### Quotes Log Table
- [ ] "Add Quote to Log" button to save current calculation
- [ ] Table displaying all saved quotes with columns:
  - Date added
  - Finance amount
  - Asset description
  - Term (months)
  - Payment (with selected frequencies)
  - Payment type (Advance/Arrears)
  - Residual/Balloon
  - Base rate (toggleable visibility)
  - Comparison rate (toggleable visibility)
  - Lender/establishment fee
  - Origination fee
  - Monthly fee
  - Commissions (toggleable visibility)
  - Total hiring installments (toggleable visibility)
  - Notes field (editable)
  - Delete action
- [ ] "Clear All Quotes" button to reset the log

#### Display Toggles
Toggle switches to show/hide specific fields in both the quotes table and email export:
- [ ] Base Rate toggle (show/hide)
- [ ] Comparison Rate toggle (show/hide)
- [ ] Commissions toggle (show/hide) - hide for client-facing quotes
- [ ] Total Hiring Installments toggle (show/hide)

#### Email-Ready Quote Export
- [ ] Formatted quote display section optimized for copy/paste into emails
- [ ] Group quotes by Finance Amount and Asset for cleaner presentation
- [ ] Format structure:
  ```
  Finance Amount: $30,000.00
  Asset: 2024 Toyota Hilux SR5

  Term: 60 months
  Repayments: $650.68 (monthly) OR $300.31 (fortnightly)
  Residual: NIL

  Lender Fee: $495 (financed)
  Origination Fee: $790 (payable at settlement)
  Comparison Rate: 10.18%
  ```
- [ ] "Copy Quote to Clipboard" button that copies formatted text
- [ ] Respect toggle settings (hide commission, rates, etc. based on toggles)

#### Total Hiring Installments Calculation
```
Total Hiring = (Monthly Payment × Term) + Balloon + Upfront Fees
```
Where upfront fees are any fees marked as "payable at settlement" rather than financed.

### Lender Management
- [ ] Preset lender choices pre-configured
- [ ] Allow users to add their own custom lenders with:
  - Lender name and logo (image upload via Supabase Storage)
  - Establishment/lender fee (financed or upfront option)
  - Origination fee (financed or upfront option)
  - Monthly account fee
  - Commission calculation method
- [ ] Edit and delete custom lenders
- [ ] Store lender configurations per user in Supabase

## Technical Requirements

### Must Have
- [ ] Next.js application with proper routing
- [ ] Supabase integration for database and authentication
- [ ] Google OAuth working correctly
- [ ] Core calculation engine matching specifications above
- [ ] Support for all 4 commission models
- [ ] CRUD operations for custom lenders
- [ ] Input validation with helpful error messages
- [ ] Test suite validating against known correct outputs

### Should Have
- [ ] Responsive design (mobile-friendly)
- [ ] Side-by-side comparison of multiple lenders
- [ ] Commission comparison showing broker earnings per lender
- [ ] Ability to save and retrieve quotes

### Nice to Have
- [ ] PDF/print-friendly quote output
- [ ] Amortization schedule generation and display
- [ ] Target commission calculator (reverse calculate required rate)
- [ ] Quote history per user
- [ ] Dark mode support

### Amortization Schedule
The application should be able to generate and display a full amortization schedule showing:
- Payment number and date
- Opening balance
- Interest for the period
- Principal paid
- Any fees (monthly fee, sliding fee)
- Closing balance

For daily interest lenders, the schedule must account for:
- Actual calendar days between payments
- Weekend/holiday adjustments for payment dates
- The +1 day interest on first payment (settlement day counts)

## Database Schema (Suggested)

```sql
-- Users table (handled by Supabase Auth)

-- Lenders table
CREATE TABLE lenders (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES auth.users(id),
  name TEXT NOT NULL,
  logo_path TEXT,  -- Path to file in Supabase Storage bucket
  is_preset BOOLEAN DEFAULT false,
  establishment_fee DECIMAL(10,2),
  establishment_fee_financed BOOLEAN DEFAULT true,
  origination_fee DECIMAL(10,2),
  origination_fee_financed BOOLEAN DEFAULT true,
  ppsr_fee DECIMAL(10,2) DEFAULT 6.00,
  ppsr_fee_financed BOOLEAN DEFAULT true,
  monthly_fee DECIMAL(10,2),
  commission_model TEXT CHECK (commission_model IN ('capitalised', 'overs', 'daily_interest', 'loaded')),
  commission_config JSONB,  -- e.g., {"rate": 0.04} or {"base": 110, "overs_percent": 0.75}
  interest_calculation TEXT CHECK (interest_calculation IN ('monthly', 'daily')),
  payment_timing TEXT CHECK (payment_timing IN ('advance', 'arrears')),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Quotes table (optional - for saving quotes)
CREATE TABLE quotes (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES auth.users(id),
  lender_id UUID REFERENCES lenders(id),
  inputs JSONB NOT NULL,
  outputs JSONB NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Quotes log table (for quote comparison feature)
CREATE TABLE quote_logs (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES auth.users(id),
  session_id TEXT,  -- Group quotes by session
  lender_id UUID REFERENCES lenders(id),
  asset_description TEXT,
  finance_amount DECIMAL(12,2),
  term_months INTEGER,
  payment_monthly DECIMAL(10,2),
  payment_type TEXT CHECK (payment_type IN ('advance', 'arrears')),
  residual_value DECIMAL(12,2),
  base_rate DECIMAL(5,4),
  comparison_rate DECIMAL(5,4),
  commission_amount DECIMAL(10,2),
  total_hiring DECIMAL(12,2),
  notes TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);
```

## Supabase Storage Configuration

Lender logos are stored using Supabase Storage. Follow these steps to configure the storage bucket:

### 1. Create the Storage Bucket
```sql
-- Run in Supabase SQL Editor
INSERT INTO storage.buckets (id, name, public)
VALUES ('lender-logos', 'lender-logos', true);
```

### 2. Set Up Row Level Security (RLS) Policies
```sql
-- Allow authenticated users to upload to their own folder
CREATE POLICY "Users can upload their own logos"
ON storage.objects FOR INSERT
TO authenticated
WITH CHECK (
  bucket_id = 'lender-logos' AND
  (storage.foldername(name))[1] = auth.uid()::text
);

-- Allow authenticated users to update/delete their own logos
CREATE POLICY "Users can manage their own logos"
ON storage.objects FOR UPDATE
TO authenticated
USING (
  bucket_id = 'lender-logos' AND
  (storage.foldername(name))[1] = auth.uid()::text
);

CREATE POLICY "Users can delete their own logos"
ON storage.objects FOR DELETE
TO authenticated
USING (
  bucket_id = 'lender-logos' AND
  (storage.foldername(name))[1] = auth.uid()::text
);

-- Allow public read access for displaying logos
CREATE POLICY "Anyone can view logos"
ON storage.objects FOR SELECT
TO public
USING (bucket_id = 'lender-logos');
```

### 3. Storage Usage in Application
```javascript
// Upload logo (store in user's folder)
async function uploadLenderLogo(userId, file) {
  const fileExt = file.name.split('.').pop();
  const fileName = `${Date.now()}.${fileExt}`;
  const filePath = `${userId}/${fileName}`;

  const { data, error } = await supabase.storage
    .from('lender-logos')
    .upload(filePath, file, {
      cacheControl: '3600',
      upsert: false
    });

  if (error) throw error;
  return filePath;
}

// Get public URL for display
function getLenderLogoUrl(logoPath) {
  const { data: { publicUrl } } = supabase.storage
    .from('lender-logos')
    .getPublicUrl(logoPath);
  return publicUrl;
}

// Delete logo when lender is deleted
async function deleteLenderLogo(logoPath) {
  const { error } = await supabase.storage
    .from('lender-logos')
    .remove([logoPath]);
  if (error) throw error;
}
```

### 4. Accepted File Types
- PNG, JPG, JPEG, SVG, WebP
- Maximum file size: 2MB recommended
- Recommended dimensions: 200x80px (maintains aspect ratio)

## Resources Provided

```
resources/
├── lender-configs.json            # Lender calculation parameters
├── test-cases.json                # Input/output pairs for validation
├── traditional/                   # Traditional (capitalised brokerage) calculator
│   ├── traditional-quoting-tool.html
│   └── traditional-test-cases.csv # 3 test cases with amortisation schedules
├── pepper/                        # Pepper Money specific resources
│   ├── pepper-calculator.html     # Loaded/factor rate calculator
│   ├── test-case-1-w-e.csv           # 60-month loan @ 12.34%, 2% commission
│   ├── test-case-2-h-y-c-pty-ltd.csv # 60-month loan @ 7.49%, 4% commission
│   └── test-case-3-g-i-i-pty-ltd.csv # 60-month loan @ 7.75%, 4.08% commission
├── branded/                       # Branded Financial Services (commission overs)
│   ├── branded-calculator.html    # Commission overs calculator
│   ├── test-case-1-r-f-pty-ltd.csv   # 60-month loan @ 10.04%, $735 commission
│   ├── test-case-2-r-b-pty-ltd.csv   # 60-month loan @ 9.95%, $1,380 commission
│   └── test-case-3-z-h-g.csv         # 60-month loan @ 13.40%, $1,254 commission
└── autopay/                       # Autopay/MoneyMe specific resources
    ├── autopay-daily-interest-calculator.html  # Daily interest calculator
    ├── test-case-1-84month.csv    # 84-month loan @ 12.15%, 5% commission
    └── test-case-2-60month.csv    # 60-month loan @ 8.95%, 4% commission
```

### Traditional Lender Test Cases (Capitalised Brokerage)

The `resources/traditional/` folder contains test cases for the capitalised brokerage model with full amortisation schedules:

**Common Details (All Test Cases):**
| Field | Value |
|-------|-------|
| Asset | New Vehicle |
| Finance Amount | $75,000.00 |
| Lender Fee | $450.00 (financed) |
| Origination Fee | $0.00 (waived) |
| NAF | $75,450.00 |
| Base Rate | 6.75% p.a. |
| Payment Mode | Arrears |

**Test Case 1: 60 months, No Balloon**
| Field | Value |
|-------|-------|
| Commission | 2.89% = $2,181.95 (GST exc) |
| Broker Receives | $2,400.15 (inc GST) |
| Total Financed | $77,631.95 |
| Comparison Rate | 7.99% p.a. |
| Monthly Payment | $1,520.50 |
| Term | 60 months |
| Balloon | $0 |

**Test Case 2: 60 months, 30% Balloon**
| Field | Value |
|-------|-------|
| Commission | 3.00% = $2,265.00 (GST exc) |
| Broker Receives | $2,491.50 (inc GST) |
| Total Financed | $77,715.00 |
| Comparison Rate | 7.79% p.a. |
| Monthly Payment | $1,207.58 |
| Term | 60 months |
| Balloon | $22,500 (30%) |

**Test Case 3: 84 months, No Balloon**
| Field | Value |
|-------|-------|
| Commission | 3.00% = $2,265.00 (GST exc) |
| Broker Receives | $2,491.50 (inc GST) |
| Total Financed | $77,715.00 |
| Comparison Rate | 7.69% p.a. |
| Monthly Payment | $1,157.69 |
| Term | 84 months |
| Balloon | $0 |

**Traditional Capitalised Brokerage Formula:**

```
NAF = Finance Amount + Lender Fee + Origination Fee (if financed)
Commission = NAF × Commission Rate (GST exclusive)
Total Financed = NAF + Commission
PMT (Arrears) = Total Financed × [r(1+r)^n] / [(1+r)^n - 1]
PMT (with Balloon) = (Total Financed - Balloon/(1+r)^n) × [r(1+r)^n] / [(1+r)^n - 1]
```

**Important - Commission GST:**
- Commission shown is **GST exclusive**
- Broker receives Commission + 10% GST (e.g., $2,181.95 + $218.20 = $2,400.15)
- The GST component is **NOT capitalised** - only the ex-GST amount is added to the loan
- GST is paid by the lender to the broker separately

**Comparison Rate:**
The comparison rate is the rate that, when applied to NAF alone, produces the same payment as applying the base rate to Total Financed. This represents the "true cost" to the customer including the capitalised commission.

### Pepper Money Test Cases

The `resources/pepper/` folder contains test cases derived from real Pepper Money loan contracts:

**Test Case 1 - W-E:**
| Field | Value |
|-------|-------|
| Finance Amount | $34,400.00 |
| Origination Fee | $790.00 (financed) |
| NAF | $35,190.00 |
| Commission | 2% = $703.80 |
| Base Rate | 12.34% p.a. |
| Comparison Rate | 13.19% p.a. |
| Monthly Payment | $795.36 |
| Payment Mode | Advance |
| Term | 60 months |
| Balloon | 0% |
| Total Interest | $12,531.60 |

**Test Case 2 - H-Y-C Pty Ltd:**
| Field | Value |
|-------|-------|
| Finance Amount | $50,480.00 |
| Lender Fee | $499.00 (financed) |
| PPSR Fee | $6.00 (financed) |
| NAF | $50,985.00 |
| Commission | 4% = $2,039.40 |
| Base Rate | 7.49% p.a. |
| Comparison Rate | 8.55% p.a. |
| Monthly Payment | $1,052.89 |
| Payment Mode | Advance |
| Term | 60 months |
| Balloon | 0% |
| Total Interest | $12,188.40 |

**Test Case 3 - G-I-I Pty Ltd:**
| Field | Value |
|-------|-------|
| Asset | New 2023 Isuzu Rigid Truck |
| Asset Price | $301,271.28 |
| Deposit | $158,271.28 |
| Finance Amount | $143,000.00 |
| Lender Fee | $499.00 (financed) |
| Origination Fee | $990.00 (financed) |
| PPSR Fee | $6.00 (financed) |
| NAF | $144,495.00 |
| Commission | 4.08% = $5,895.40 |
| Base Rate | 7.75% p.a. |
| Comparison Rate | 9.40% p.a. |
| Monthly Payment | $3,003.91 |
| Payment Mode | Advance |
| Term | 60 months |
| Balloon | 0% |
| Total Interest | $35,739.60 |

**Pepper's Loading Factor Formula:**

Pepper uses a unique "loading factor" that partially capitalises the commission:

```
NAF = Finance Amount + All Financed Fees
Commission = NAF × Commission Rate
Loading = (NAF / (NAF + Commission)) × (1 - 0.4 × Financier Rate)
Amount Financed = NAF + (Loading × Commission)
```

The 0.4 loading factor was reverse-engineered from Pepper's portal. At typical rates, this results in only ~93% of the commission being added to the amount financed.

**Payment Calculation:**
```
PMT (Arrears) = Amount Financed × [r(1+r)^n] / [(1+r)^n - 1]
PMT (Advance) = PMT Arrears / (1 + Monthly Rate)
```

**Comparison Rate:**
The comparison rate is the rate that, when applied to NAF alone, produces the same payment. This represents the "true cost" to the customer.

**Important - Pepper Commission Structure:**

Unlike capitalised brokerage models where commission is added directly to the principal, **Pepper's commission is serviced by the interest margin**. The commission is NOT added to the principal balance. Instead:
- The customer pays a higher effective interest rate
- The rate margin (difference between base rate and effective rate) covers the broker's commission
- This is why the "Comparison Rate" is always higher than the "Base Rate"

For example, in Test Case 2 (H-Y-C Pty Ltd):
- Base Rate: 7.49% p.a.
- Effective Rate (derived from amortization): ~8.55% p.a.
- The ~1.06% rate differential services the 4% commission over the loan term

### Branded Financial Services Test Cases

The `resources/branded/` folder contains test cases for the commission overs model:

**Test Case 1 - R-F Pty Ltd:**
| Field | Value |
|-------|-------|
| Asset Price | $20,853.31 |
| Deposit | $2,083.81 |
| Finance Amount | $18,769.50 |
| Establishment Fee | $550.00 (financed) |
| Origination Fee | $990.00 (financed) |
| PPSR Fee | $6.00 (financed) |
| NAF | $20,315.50 |
| Base Rate | 8.54% p.a. |
| Contract Rate | 10.04% p.a. |
| Commission | $735.02 (overs) |
| Net Payment | $428.46 |
| Monthly Fee | $8.00 |
| Gross Payment | $436.46 |
| Payment Mode | Advance |
| Term | 60 months |
| Balloon | 0% |

**Test Case 2 - R-B Pty Ltd:**
| Field | Value |
|-------|-------|
| Asset Price | $104,865.09 |
| Deposit | $25,000.00 |
| Finance Amount | $79,865.09 |
| Establishment Fee | $550.00 (financed) |
| Origination Fee | $990.00 (financed) |
| PPSR Fee | $6.00 (financed) |
| NAF | $81,411.09 |
| Base Rate | 9.25% p.a. |
| Contract Rate | 9.95% p.a. |
| Commission | $1,380.43 (overs) |
| Net Payment | $1,713.53 |
| Monthly Fee | $8.00 |
| Gross Payment | $1,721.53 |
| Payment Mode | Advance |
| Term | 60 months |
| Balloon | 0% |

**Test Case 3 - Z-H-G:**
| Field | Value |
|-------|-------|
| Asset Price | $35,000.00 |
| Deposit | $3,500.00 |
| Finance Amount | $31,500.00 |
| Establishment Fee | $650.00 (financed) |
| Origination Fee | $990.00 (financed) |
| PPSR Fee | $6.00 (financed) |
| NAF | $33,146.00 |
| Base Rate | 11.90% p.a. |
| Contract Rate | 13.40% p.a. |
| Commission | $1,254.20 (overs) |
| Net Payment | $752.57 |
| Monthly Fee | $8.00 |
| Gross Payment | $760.57 |
| Payment Mode | Advance |
| Term | 60 months |
| Balloon | 0% |

**Branded's Commission Overs Formula:**

Branded uses a "commission overs" model where broker earns from the rate markup:

```
NAF = Finance Amount + All Financed Fees
Base PMT = PMT(NAF, Base Rate, Term, Balloon)
Final PMT = PMT(NAF, Contract Rate, Term, Balloon)
Hiring Difference = (Final PMT × Term) - (Base PMT × Term)
Overs Amount = Hiring Difference × 75% × 1.10 (GST)
Commission = MAX($110 base, Overs Amount)
```

**Key Difference from Capitalised Brokerage:**
- Commission is NOT added to the principal
- Customer pays the Contract Rate (not a hidden rate)
- Broker "dials up" the rate above base to earn commission
- Higher markup = higher commission, but also higher customer payment

**Payment Calculation:**
```
PMT (Arrears) = (NAF - PV_balloon) × [r(1+r)^n] / [(1+r)^n - 1]
PMT (Advance) = PMT Arrears / (1 + Monthly Rate)
```

### Autopay Test Cases

The `resources/autopay/` folder contains real amortization schedules from the lender for validation:

**Test Case 1 (84 months):**
| Field | Value |
|-------|-------|
| Asset/Finance Amount | $58,179.70 |
| Establishment Fee | $550.00 |
| Origination Fee | $1,490.00 |
| NAF | $60,219.70 |
| Commission | 5% = $3,010.99 |
| Base Rate | 10.15% |
| Contract Rate | 12.15% (base + 2%) |
| Monthly Payment | $1,060.74 |
| Monthly Fee | $12.50 |
| Sliding Fee | $12.50 (first payment) |

**Test Case 2 (60 months):**
| Field | Value |
|-------|-------|
| Asset/Finance Amount | $84,724.86 |
| Establishment Fee | $490.00 |
| Origination Fee | $490.00 |
| NAF | $85,704.86 |
| Commission | 4% = $3,428.19 |
| Base Rate | 7.35% |
| Contract Rate | 8.95% (base + 1.6%) |
| Monthly Payment | $1,767.42 |
| Monthly Fee | $12.50 |
| Sliding Fee | $12.50 (first payment) |

Your amortization schedule generator should match these CSV files exactly (to the cent) when given the same inputs.

### Amortization Schedule Responsibilities

**Autopay/MoneyMe:**
The CSV amortization schedules are provided for your convenience because daily interest calculations have nuances that are difficult to replicate:
- Interest accrues daily using actual/365 day count
- Payment dates can shift due to weekends and holidays
- The first payment includes +1 day interest (settlement day counts)
- Each period has variable interest based on actual calendar days

Use the provided Autopay CSVs as your validation reference. Your schedule should match these exactly.

**Pepper, Branded, and Traditional Lenders:**
For these lenders, the calculators provide the repayment amount and comparison rate calculations, but **you are responsible for creating the amortization schedule logic**. The provided CSVs show what the correct amortization should look like - your implementation must produce matching results.

Key considerations:
- **Traditional:** Full amortisation schedules provided in `traditional-test-cases.csv`. Commission IS capitalised (added to principal). Use base rate on Total Financed. Includes balloon payment example.
- **Pepper:** First payment in ADVANCE mode is pure principal (no interest). Commissions are serviced by interest, not added to principal. Use the comparison rate for amortization.
- **Branded:** Use the Contract Rate (NOT Base Rate) for interest calculations. Standard compound interest amortization with monthly compounding. Commissions paid from overs are NOT added to principal.
- **Advance mode:** First payment due at settlement reduces principal before interest begins accruing (first payment is pure principal).
- **Arrears mode:** First payment due one period after settlement includes full period's interest.

## Validation

Your calculator passes validation when:

1. All test cases in `test-cases.json` produce matching outputs (to the cent)
2. Given the same inputs as a lender PDF quote, your output matches exactly
3. Commission calculations match what the broker actually receives

## Rounding Rules

1. **Intermediate calculations**: Maintain full precision
2. **Final payment**: Round to nearest cent
3. **Commission**: Round to nearest cent
4. **Some lenders**: Round up payment to whole dollar

## Common Pitfalls

1. **Using simple interest**: Auto finance always uses compound
2. **Wrong period rate**: Must match payment frequency
3. **Forgetting fees in finance amount**: Check if fees are financed or upfront
4. **Rounding too early**: Keep precision until final result
5. **Advance vs arrears confusion**: Payment differs by factor of (1+i)

## Key Challenges

1. **Precision**: Financial calculations require exact decimal handling (use Decimal.js or similar)
2. **Commission Logic**: Each lender's commission model is subtly different
3. **Rate Relationships**: Understanding base rate vs customer rate vs comparison rate
4. **Edge Cases**: Balloon payments, advance vs arrears, fee financing options
5. **State Management**: Handling user-specific lender configurations

## Getting Started

1. Set up a Supabase project and configure Google OAuth
2. Create a Storage bucket for lender logos
3. Study the reference calculators in `resources/`
4. Start with the traditional capitalised model (simplest)
5. Add authentication and user-specific lender storage
6. Build the lender management UI
7. Add remaining commission models
8. Validate against provided test cases frequently

## Evaluation Criteria

1. **Functionality**: Does the calculator produce correct results?
2. **Code Quality**: Is the code well-organized and maintainable?
3. **UX/UI**: Is the application intuitive and easy to use?
4. **Authentication**: Is the auth flow secure and properly implemented?
5. **Database Design**: Is the schema well-designed for the use case?

## Questions to Consider

- How will you handle floating-point precision (hint: financial libraries)?
- How will you structure the code to easily add new commission models?
- How will you validate user inputs before calculation?
- How will you handle image uploads and storage for lender logos?
- What happens when a user wants to update a preset lender's fees?
