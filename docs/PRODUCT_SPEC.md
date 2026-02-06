# FoliarScript Product Specification

> **Version**: 2.0 (Rebuild)
> **Last Updated**: 2026-02-06

---

## Executive Summary

FoliarScript is a foliar nutrient analysis and recommendation platform for agriculture. Users submit plant tissue lab results, and the system calculates precise fertilizer application recommendations based on crop type, growth stage, and current nutrient levels.

### Goals for Rebuild

1. **Simplify the data model** - Remove overly complex hierarchy
2. **Modernize the stack** - FastAPI + Next.js + Supabase
3. **Improve UX** - Mobile-first, responsive design
4. **Streamline input** - PDF parsing + flexible CSV mapping

---

## User Model

### Simplified Hierarchy

```
User
├── Clients (just names - records belonging to user)
├── Fields (just names - for grouping scripts)
└── Scripts
      ├── Nutrient values
      └── Recommendation (generated on payment)
```

**Key Decisions:**
- Everyone is a "User" - no explicit Agronomist/Grower distinction
- Clients are just name records (no login, no email)
- A client cannot be shared/transferred between users
- Fields are just names for organization + history tracking

### User Entity

```
User
├── id                 : uuid (PK, from Supabase Auth)
├── email              : string (unique, from Supabase Auth)
├── name               : string
├── created_at         : timestamp
└── updated_at         : timestamp
```

### Client Entity (Record Only)

```
Client
├── id                 : uuid (PK)
├── user_id            : uuid (FK → User)
├── name               : string (required)
├── created_at         : timestamp
└── updated_at         : timestamp
```

---

## Core Flows

### 1. Manual Script Creation

```
User fills wizard form:
  Step 1: Metadata (client, field, crop, stage, sample_id, date, lab)
  Step 2: Macronutrients (N, P, K, Ca, Mg, S)
  Step 3: Micronutrients (Zn, Fe, Mn, Cu, B)
  Step 4: Review & Submit
       ↓
Script created (status: draft)
       ↓
User proceeds to payment
       ↓
Payment completes (Stripe)
       ↓
Recommendation calculated & stored
       ↓
User views recommendation
```

### 2. PDF Upload Flow

```
User uploads PDF from lab
       ↓
PDF parsing API extracts values
       ↓
Pre-filled form shown to user
       ↓
User corrects any errors (human-in-the-loop)
       ↓
Continue with normal submission flow
```

### 3. CSV Upload Flow

```
User uploads any CSV format
       ↓
Column mapping UI shows detected columns
       ↓
User maps columns to our fields
       ↓
User corrects missing/invalid values
       ↓
Multiple scripts created
       ↓
Single payment for all scripts
```

### 4. View Recommendations

```
User selects completed script
       ↓
Recommendation page shows:
  - Producer details bar
  - Nutrient table with test values
  - Concentration sliders (for Low nutrients)
  - Calculated application amounts
  - Split application timeline (if needed)
       ↓
User can adjust concentration (recalculates in real-time)
       ↓
User can download PDF report
```

---

## Script States

```
draft ──────────→ unpaid ──────────→ paid (completed)
   (created)      (submitted)       (payment success)
        │              │
        └──────────────┴──── Can edit before payment
                             Can delete anytime (no refund)
```

| State | Description |
|-------|-------------|
| `draft` | Script created, not yet submitted |
| `unpaid` | Submitted, awaiting payment |
| `paid` | Payment complete, recommendation available |

---

## Data Entities

### Script

```
Script
├── id                    : uuid (PK)
├── user_id               : uuid (FK → User)
├── client_id             : uuid (FK → Client, nullable)
├── field_id              : uuid (FK → Field, nullable)
│
├── # Metadata
├── crop                  : enum ['corn', 'soybean', ...]
├── stage                 : string (growth stage code)
├── sample_id             : string (user-provided from lab)
├── lab_name              : string (free text)
├── test_date             : date
├── status                : enum ['draft', 'unpaid', 'paid']
│
├── # Macronutrients (percentages, nullable = not tested)
├── nitrogen              : float
├── phosphorus            : float
├── potassium             : float
├── calcium               : float
├── magnesium             : float
├── sulfur                : float
│
├── # Micronutrients (ppm, nullable = not tested)
├── zinc                  : float
├── iron                  : float
├── manganese             : float
├── copper                : float
├── boron                 : float
├── molybdenum            : float (optional)
│
├── # File tracking (for CSV uploads)
├── source_file_url       : string (S3 URL)
├── source_filename       : string
│
├── # Payment
├── payment_intent_id     : string (Stripe)
├── payment_status        : string
├── paid_at               : timestamp
│
├── created_at            : timestamp
└── updated_at            : timestamp
```

### Recommendation

```
Recommendation
├── id                    : uuid (PK)
├── script_id             : uuid (FK → Script, unique)
├── model_version         : string (for tracking)
│
├── # Calculated amounts (oz/acre, 0 = sufficient)
├── nitrogen              : float
├── phosphorus            : float
├── potassium             : float
├── calcium               : float
├── magnesium             : float
├── sulfur                : float
├── zinc                  : float
├── iron                  : float
├── manganese             : float
├── copper                : float
├── boron                 : float
├── molybdenum            : float
│
├── created_at            : timestamp
└── updated_at            : timestamp
```

### Field

```
Field
├── id                    : uuid (PK)
├── user_id               : uuid (FK → User)
├── name                  : string (required)
├── created_at            : timestamp
└── updated_at            : timestamp
```

---

## Payment

### Model
- **Pricing**: Flat rate per script (regardless of crop)
- **Bulk**: Same price each, no volume discount
- **Blocking**: Must pay before seeing recommendations
- **Provider**: Stripe

### Failed Payment Handling
- Keep script in `unpaid` state
- User can retry payment later

### Deletion After Payment
- Allowed (user's choice)
- No refund (they already saw recommendations)

---

## Dashboard

### Primary View (for User)

```
┌─────────────────────────────────────────────────────────────┐
│  FoliarScript                          [User Name] [Logout] │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Quick Stats                                                │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐                      │
│  │   15    │  │    3    │  │    5    │                      │
│  │ Scripts │  │ Clients │  │ Unpaid  │                      │
│  └─────────┘  └─────────┘  └─────────┘                      │
│                                                             │
│  [+ Create New Script]  ← Primary action                    │
│                                                             │
│  ─────────────────────────────────────────────────────────  │
│                                                             │
│  Unpaid Scripts (Needs Attention)                           │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Sample-001 │ Corn │ V6 │ North Field │ [Pay Now]    │    │
│  │ Sample-002 │ Soy  │ R2 │ South Field │ [Pay Now]    │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
│  Recent Scripts                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Sample-003 │ Corn │ V8 │ East Field │ Paid │ [View] │    │
│  │ Sample-004 │ Soy  │ R1 │ West Field │ Paid │ [View] │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Script Creation Wizard

### Step Flow

```
Step 1: Basic Info
├── Client (dropdown + "Add New")
├── Field (dropdown + "Add New")
├── Crop (dropdown: corn, soybean, ...)
├── Growth Stage (dropdown, filtered by crop)
├── Sample ID (text input)
├── Test Date (date picker, max: today)
└── Laboratory (text input)

Step 2: Macronutrients
├── Nitrogen (%) - optional
├── Phosphorus (%) - optional
├── Potassium (%) - optional
├── Calcium (%) - optional
├── Magnesium (%) - optional
└── Sulfur (%) - optional
    (At least one nutrient required)

Step 3: Micronutrients
├── Zinc (ppm) - optional
├── Iron (ppm) - optional
├── Manganese (ppm) - optional
├── Copper (ppm) - optional
├── Boron (ppm) - optional
└── Molybdenum (ppm) - optional

Step 4: Review & Submit
├── Summary of all entered data
├── Edit buttons for each section
└── [Submit & Pay] button
```

---

## Field History

### Features Required

1. **Timeline View**: All tests on this field, chronologically
2. **Line Charts**: Nutrient levels over time
3. **Side-by-Side Comparison**: Select two tests to compare

### History Table

| Date | Sample ID | Crop | Stage | N | P | K | ... | Status |
|------|-----------|------|-------|---|---|---|-----|--------|
| 2024-01-15 | SAMPLE-003 | Corn | V8 | 3.2% | 0.35% | 2.1% | ... | Paid |
| 2024-02-01 | SAMPLE-005 | Corn | VT | 3.5% | 0.38% | 2.3% | ... | Paid |

---

## Recommendations Page

### Layout

```
┌─────────────────────────────────────────────────────────────┐
│  Producer: John Doe | Corn | V6 | North Field | 01/15/2024  │
├─────────────────────────────────────────────────────────────┤
│  Legend: ● Sufficient  ● Low                    [Download]  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌───────────┬────────────┬───────────────┬────────────────┐│
│  │ Nutrient  │ Test Value │ Concentration │ Application    ││
│  ├───────────┼────────────┼───────────────┼────────────────┤│
│  │ Nitrogen  │ [3.20 %]   │ ──●────────── │ 12.50 fl oz    ││
│  │           │   Low      │   [25.00]     │ ┌─┐            ││
│  │           │            │   0      100  │ │1│→ 5.0 fl oz ││
│  │           │            │               │ └─┘            ││
│  │           │            │               │  │ After 1 wk  ││
│  │           │            │               │ ┌─┐            ││
│  │           │            │               │ │2│→ 5.0 fl oz ││
│  │           │            │               │ └─┘            ││
│  │           │            │               │  │ After 1 wk  ││
│  │           │            │               │ ┌─┐            ││
│  │           │            │               │ │3│→ 2.5 fl oz ││
│  │           │            │               │ └─┘            ││
│  ├───────────┼────────────┼───────────────┼────────────────┤│
│  │ Phosphorus│ [0.35 %]   │      --       │ Sufficient     ││
│  │           │ Sufficient │               │                ││
│  └───────────┴────────────┴───────────────┴────────────────┘│
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Interaction Rules

1. **Slider only shows for "Low" nutrients**
2. **Slider adjusts concentration (0-100%)**
3. **Total amount recalculates inversely**: higher concentration = lower volume
4. **Split into weekly applications** if exceeds max per application
5. **Maximum 3 applications shown**; excess shows warning
6. **Concentration = 0 displays "--" and 0 amount**

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| Frontend | Next.js 15, React 19, TypeScript |
| Styling | Tailwind CSS, shadcn/ui, Radix UI |
| Backend | FastAPI (Python) |
| Database | PostgreSQL (via Supabase) |
| Auth | Supabase Auth (email/password) |
| Payments | Stripe |
| File Storage | AWS S3 |
| PDF Parsing | External API (provided) |

---

## Authentication

- **Provider**: Supabase Auth
- **Method**: Email/password only (for now)
- **Registration**: Open, email verification handled by Supabase
- **No approval process**

---

## Notifications (MVP)

- **Signup confirmation**: Handled by Supabase
- **No other emails** in MVP

---

## Crops & Extensibility

- **Current**: Corn, Soybean
- **Future**: More crops planned
- **Model data**: Stored/configured to support adding crops

---

## Business Rules

1. **Recommendations never recalculate** - model version locked at creation
2. **Scripts editable until payment**
3. **Scripts deletable anytime** (no refund post-payment)
4. **At least one nutrient required** per script
5. **Null nutrients = not tested** (skip in calculations)
6. **Sample ID is user-provided** (from lab)
7. **Laboratory is free text** entry
8. **Field auto-created** if new name entered

---

## Out of Scope (for MVP)

- [ ] Consulting group hierarchy
- [ ] Grower login/accounts
- [ ] Multiple agronomists per grower
- [ ] Field GPS/acreage
- [ ] Social login
- [ ] Email notifications (beyond Supabase)
- [ ] Data migration (handled separately)
- [ ] Bulk discounts
- [ ] Subscription pricing

---

## Open Items

| Item | Status | Notes |
|------|--------|-------|
| PDF template design | **Done** | See `docs/reference/10-pdf-template-spec.md` |
| PDF parsing API details | Pending | User will provide API endpoint details |
| CSV mapping service | Pending | User will provide details |
| Stripe configuration | Pending | User will provide when we reach that stage |
| Model data for new crops | Future | Currently corn + soybean |
