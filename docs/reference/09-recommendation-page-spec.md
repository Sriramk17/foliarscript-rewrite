# Recommendation Page - Technical Specification

> Complete implementation spec for the FoliarScript Recommendations Page.

---

## Overview

The Recommendations Page displays nutrient recommendations based on lab test results. Users can:

- View nutrient test results (N, P, K, Zn, Fe, Mn, Cu, B, etc.)
- Adjust concentration levels using interactive sliders
- See real-time recalculation of application amounts
- View split application schedules when amounts exceed limits
- Download recommendations as PDF

---

## Data Structures

### Nutrient Interface

```typescript
interface Nutrient {
  // Identity
  name: string;              // Short code: "N", "P", "K", "Zn", etc.
  fullName: string;          // Display name: "Nitrogen", "Phosphorus", etc.

  // Lab Results
  testValue: number;         // Measured value from lab test
  category: 'Sufficient' | 'Low';

  // Slider State
  sliderValue: number;       // Current slider position (0-100)
  displayedValue: string;    // Formatted value shown in tooltip (e.g., "25.00" or "--")
  isInputActive: boolean;    // True when user is editing the tooltip input
  rawInputValue?: string;    // Raw string value during editing

  // Calculation Values (from API)
  totalAmount: number;       // Total fl oz to apply
  maxAmount: number;         // Backend-provided max amount
  concentration: number;     // Backend-provided max concentration
  defaultRate: number;       // Current concentration rate
  maxAmountToApply: number;  // Calculated max per single application

  // For recalculation (store originals)
  originalTotalAmount?: number;
  originalDefaultRate?: number;

  // Split Applications
  splitAmounts: number[];    // Array of amounts per application [5.0, 5.0, 2.5]
  hasExcess: boolean;        // True if totalAmount > 3 * maxAmountToApply
}
```

### Nutrient Name Mapping

```typescript
const NUTRIENT_NAME_MAP: Record<string, string> = {
  'N': 'Nitrogen',
  'P': 'Phosphorus',
  'K': 'Potassium',
  'Mg': 'Magnesium',
  'Ca': 'Calcium',
  'S': 'Sulfur',
  'Fe': 'Iron',
  'Mn': 'Manganese',
  'Cu': 'Copper',
  'B': 'Boron',
  'Zn': 'Zinc',
  'Mo': 'Molybdenum'
};
```

### Nutrient Units

```typescript
const NUTRIENT_UNITS: Record<string, string> = {
  'Nitrogen': '%',
  'Phosphorus': '%',
  'Potassium': '%',
  'Magnesium': '%',
  'Calcium': '%',
  'Sulfur': '%',
  'Zinc': 'PPM',
  'Iron': 'PPM',
  'Manganese': 'PPM',
  'Copper': 'PPM',
  'Boron': 'PPM',
  'Molybdenum': 'PPM'
};
```

### Producer Details

```typescript
interface ProducerDetails {
  name: string;        // Client name
  crop: string;        // Crop type (e.g., "Corn", "Soybean")
  stage: string;       // Growth stage (e.g., "V6", "R1")
  field_name: string;  // Field name
  sample_id: string;   // Lab sample ID
  testdate: string;    // Date of test
}
```

---

## API Response

### Endpoint

```
GET /api/recommendations?script_id={id}

Response:
{
  "data": {
    "producer_details": {
      "name": "John Doe",
      "crop": "Corn",
      "stage": "V6",
      "field_name": "North Field",
      "sample_id": "LAB-2024-001",
      "testdate": "2024-01-15"
    },
    "recommendations": {
      "N": {
        "test_val": 2.85,
        "recommended_amount": 12.5,
        "max_apply_amount": 5.0,
        "max_conc": 50,
        "default_conc": 25
      },
      "P": {
        "test_val": 0.32,
        "recommended_amount": 0,
        "max_apply_amount": 3.0,
        "max_conc": 40,
        "default_conc": 20
      }
      // ... other nutrients
    }
  }
}
```

### Field Mapping

| API Field | Nutrient Property | Description |
|-----------|-------------------|-------------|
| `test_val` | `testValue` | Lab measurement |
| `recommended_amount` | `totalAmount` | Total fl oz needed |
| `max_apply_amount` | `maxAmount` | Max amount constant |
| `max_conc` | `concentration` | Max concentration constant |
| `default_conc` | `defaultRate`, `sliderValue` | Starting concentration |

---

## Core Calculations

### 1. Category Determination

```typescript
const category = recommendedAmount === 0 ? 'Sufficient' : 'Low';
```

### 2. Max Amount Per Application

**Formula**: `maxAmountToApply = (maxAmount × concentration) / currentRate`

```typescript
function calculateMaxAmountToApply(
  maxAmount: number,
  concentration: number,
  displayedValue: number
): number {
  if (displayedValue === 0) displayedValue = 0.1; // Prevent division by zero
  return parseFloat(((maxAmount * concentration) / displayedValue).toFixed(2));
}
```

**Example**:
- `maxAmount = 5.0`, `concentration = 50`, `displayedValue = 25`
- Result: `(5.0 × 50) / 25 = 10.0 fl oz` per application

### 3. Update Total Amount on Concentration Change

**Formula**: `newTotalAmount = (originalTotalAmount × originalDefaultRate) / newRate`

This creates an **inverse relationship**: higher concentration = lower total volume needed.

```typescript
function updateTotalAmount(nutrient: Nutrient, newRate: number): void {
  if (newRate <= 0) {
    nutrient.totalAmount = 0;
    nutrient.splitAmounts = [];
    return;
  }

  // Store originals on first change
  if (!nutrient.originalTotalAmount) {
    nutrient.originalTotalAmount = nutrient.totalAmount;
    nutrient.originalDefaultRate = nutrient.defaultRate;
  }

  const newTotalAmount = parseFloat(
    ((nutrient.originalTotalAmount * nutrient.originalDefaultRate) / newRate).toFixed(2)
  );

  nutrient.totalAmount = newTotalAmount;
  nutrient.defaultRate = newRate;

  calculateApplicationTiming(nutrient);
}
```

**Example**:
- Original: `totalAmount = 12.5`, `defaultRate = 25`
- User changes rate to `50`
- New total: `(12.5 × 25) / 50 = 6.25 fl oz`

### 4. Split Application Calculation

**Logic**:
- If `totalAmount > maxAmountToApply`, split into multiple applications
- Each application gets `maxAmountToApply`, except last gets remainder
- If `totalAmount > 3 × maxAmountToApply`, flag as "excess"

```typescript
function calculateApplicationTiming(nutrient: Nutrient): void {
  nutrient.hasExcess = nutrient.totalAmount > (3 * nutrient.maxAmountToApply);

  if (nutrient.maxAmountToApply <= 0) {
    nutrient.splitAmounts = [];
    return;
  }

  const applications = Math.ceil(nutrient.totalAmount / nutrient.maxAmountToApply);
  nutrient.splitAmounts = [];

  let remainingAmount = nutrient.totalAmount;
  for (let i = 0; i < applications; i++) {
    if (remainingAmount > nutrient.maxAmountToApply) {
      nutrient.splitAmounts.push(nutrient.maxAmountToApply);
      remainingAmount -= nutrient.maxAmountToApply;
    } else {
      nutrient.splitAmounts.push(parseFloat(remainingAmount.toFixed(2)));
      remainingAmount = 0;
    }
  }
}
```

**Example**:
- `totalAmount = 12.5`, `maxAmountToApply = 5.0`
- Applications needed: `ceil(12.5 / 5.0) = 3`
- `splitAmounts = [5.0, 5.0, 2.5]`

---

## Slider Implementation

### Component Structure

```
    ┌──────────┐
    │  25.00   │  ← Editable tooltip input (positioned at slider value %)
    └────┬─────┘
         │
    ─────●────────────────  ← Range slider (0-100)
    0                   100
```

### JSX Structure (React/Next.js)

```tsx
{nutrient.category === 'Low' && (
  <div className="relative w-full">
    {/* Editable Tooltip */}
    <input
      type="text"
      className="absolute -top-8 transform -translate-x-1/2 bg-[#ADD1FF] border border-white/30 rounded w-12 h-7 text-center text-xs text-[#000A3C] outline-none"
      style={{ left: `${nutrient.sliderValue}%` }}
      value={nutrient.rawInputValue ?? nutrient.displayedValue}
      onChange={(e) => onTooltipValueChange(e.target.value, nutrient)}
      onFocus={() => onInputFocus(nutrient)}
      onBlur={(e) => onInputBlur(nutrient, e)}
      inputMode="decimal"
    />

    {/* Range Slider */}
    <input
      type="range"
      min="0"
      max="100"
      value={nutrient.sliderValue}
      onChange={(e) => onSliderChange(e, nutrient)}
      className="w-full h-1.5 rounded-full appearance-none cursor-pointer"
      style={{
        background: `linear-gradient(to right, #0372FF 0%, #0372FF ${nutrient.sliderValue}%, #2D4A6E ${nutrient.sliderValue}%, #2D4A6E 100%)`
      }}
    />

    {/* Min/Max Labels */}
    <div className="flex justify-between text-sm text-white mt-1">
      <span>0</span>
      <span>100</span>
    </div>
  </div>
)}
```

### Event Handlers

```typescript
// On Slider Drag
function onSliderChange(event: React.ChangeEvent<HTMLInputElement>, nutrient: Nutrient) {
  const value = Number(event.target.value);
  nutrient.sliderValue = value;

  if (value === 0) {
    nutrient.displayedValue = "--";
    nutrient.totalAmount = 0;
    nutrient.splitAmounts = [];
    nutrient.rawInputValue = "";
  } else {
    nutrient.displayedValue = value.toFixed(2);
    nutrient.rawInputValue = value.toString();
    nutrient.maxAmountToApply = calculateMaxAmountToApply(
      nutrient.maxAmount,
      nutrient.concentration,
      value
    );
    updateTotalAmount(nutrient, value);
  }
}

// On Tooltip Focus
function onInputFocus(nutrient: Nutrient) {
  nutrient.isInputActive = true;
  if (nutrient.displayedValue === "--") {
    nutrient.rawInputValue = "";
  } else {
    nutrient.rawInputValue = parseFloat(nutrient.displayedValue).toString();
  }
}

// On Tooltip Change (real-time)
function onTooltipValueChange(newValue: string, nutrient: Nutrient) {
  nutrient.rawInputValue = newValue;

  if (!newValue) {
    nutrient.sliderValue = 0;
    nutrient.displayedValue = "--";
    nutrient.totalAmount = 0;
    nutrient.splitAmounts = [];
    return;
  }

  const parsed = parseFloat(newValue.replace(/[^\d.]/g, ''));
  if (!isNaN(parsed)) {
    const bounded = Math.min(100, Math.max(0, parsed));
    nutrient.sliderValue = bounded;

    if (parsed === 0) {
      nutrient.displayedValue = "--";
      nutrient.totalAmount = 0;
      nutrient.splitAmounts = [];
    } else {
      nutrient.displayedValue = bounded.toFixed(2);
      nutrient.maxAmountToApply = calculateMaxAmountToApply(
        nutrient.maxAmount,
        nutrient.concentration,
        bounded
      );
      updateTotalAmount(nutrient, bounded);
    }
  }
}

// On Tooltip Blur
function onInputBlur(nutrient: Nutrient) {
  nutrient.isInputActive = false;
  // Finalize value formatting
  if (!nutrient.rawInputValue?.trim()) {
    nutrient.displayedValue = "--";
    nutrient.sliderValue = 0;
    nutrient.totalAmount = 0;
    nutrient.splitAmounts = [];
  }
}
```

---

## Application Timeline Graphic

### Display Logic

```
IF splitAmounts.length === 0:
    Show "Sufficient" (green text)

ELSE IF splitAmounts.length === 1:
    Show single amount: "X.XX fl oz"

ELSE IF splitAmounts.length > 1:
    Show total amount + timeline graphic
    Limit displayed circles to 3 max
    If hasExcess === true, show warning message
```

### Visual Structure

```
┌─────────────────────────────────────────────┐
│              12.50 fl oz                    │
│                                             │
│    ┌───┐                                    │
│    │ 1 │ ────────  5.00 fl oz               │
│    └───┘           1st Application          │
│      │                                      │
│  After                                      │
│  1 week                                     │
│      │                                      │
│    ┌───┐                                    │
│    │ 2 │ ────────  5.00 fl oz               │
│    └───┘           2nd Application          │
│      │                                      │
│  After                                      │
│  1 week                                     │
│      │                                      │
│    ┌───┐                                    │
│    │ 3 │ ────────  2.50 fl oz               │
│    └───┘           3rd Application          │
└─────────────────────────────────────────────┘
```

### Circle Colors (Gradient)

```css
.circle-1 { background: rgba(0, 94, 214, 1); }    /* #005ED6 - darkest */
.circle-2 { background: rgba(56, 136, 237, 1); }  /* #3888ED - medium */
.circle-3 { background: rgba(124, 177, 245, 1); } /* #7CB1F5 - lightest */
```

### Ordinal Suffix Helper

```typescript
function getOrdinalSuffix(n: number): string {
  if (n % 10 === 1 && n % 100 !== 11) return "st";
  if (n % 10 === 2 && n % 100 !== 12) return "nd";
  if (n % 10 === 3 && n % 100 !== 13) return "rd";
  return "th";
}
```

---

## Test Value Badge

### Styling

```tsx
<span
  className={cn(
    "px-3 py-2 rounded text-sm font-semibold min-w-[60px] text-center",
    nutrient.category === 'Sufficient'
      ? "bg-[rgba(14,155,21,0.5)] text-[#10CF18]"
      : "bg-[rgba(220,123,12,0.28)] text-[#FF9982]"
  )}
>
  {formatValue(nutrient.testValue)} {NUTRIENT_UNITS[nutrient.fullName]}
</span>
```

---

## Page Header

### Producer Details Bar

```tsx
<div className="flex items-center gap-2 text-sm text-muted-foreground">
  <span className="capitalize">{details.name}</span>
  <span>|</span>
  <span className="capitalize">{details.crop}</span>
  <span>|</span>
  <span className="uppercase">{details.stage}</span>
  <span>|</span>
  <span className="capitalize">{details.field_name}</span>
  <span>|</span>
  <span>{formatDate(details.testdate)}</span>
</div>
```

### Legend Bar

```tsx
<div className="flex items-center gap-6">
  <div className="flex items-center gap-2">
    <div className="w-3 h-3 rounded-full bg-[#11C819]" />
    <span>Sufficient</span>
  </div>
  <div className="flex items-center gap-2">
    <div className="w-3 h-3 rounded-full bg-[#FF9822]" />
    <span>Low</span>
  </div>
</div>
```

---

## Table Structure

| Column | Width | Content |
|--------|-------|---------|
| Nutrient | ~2/12 | Full nutrient name |
| Test Value | ~3/12 | Badge with value + unit |
| Concentration | ~3/12 | Slider (if Low) or empty |
| Total Application | ~4/12 | Amount or timeline graphic |

---

## PDF Export

### Dependencies

```bash
npm install html2canvas jspdf
```

### Implementation

```typescript
import html2canvas from 'html2canvas';
import jsPDF from 'jspdf';

async function generatePDF(details: ProducerDetails) {
  const element = document.getElementById('pdf-content')!;

  const canvas = await html2canvas(element, {
    useCORS: true,
    logging: false,
    scale: 2, // Higher quality
  });

  const imgData = canvas.toDataURL('image/png');
  const imgWidth = 210; // A4 width in mm
  const pageHeight = 297; // A4 height in mm
  const imgHeight = (canvas.height * imgWidth) / canvas.width;

  const doc = new jsPDF({
    orientation: 'portrait',
    unit: 'mm',
    format: 'a4',
  });

  let heightLeft = imgHeight;
  let position = 0;

  // First page
  doc.addImage(imgData, 'PNG', 0, position, imgWidth, imgHeight);
  heightLeft -= pageHeight;

  // Additional pages if needed
  while (heightLeft > 0) {
    position = heightLeft - imgHeight;
    doc.addPage();
    doc.addImage(imgData, 'PNG', 0, position, imgWidth, imgHeight);
    heightLeft -= pageHeight;
  }

  const fileName = `${details.name}_foliar_script.pdf`;
  doc.save(fileName);
}
```

---

## Edge Cases

| Scenario | Behavior |
|----------|----------|
| Concentration = 0 | Display "--", totalAmount = 0, no applications |
| Empty input on blur | Treat as 0 |
| Value > 100 | Clamp to 100 |
| Value < 0 | Clamp to 0 |
| totalAmount = 0 | Show "Sufficient" |
| splitAmounts > 3 | Show only first 3 circles, use maxAmountToApply for 3rd |
| hasExcess = true | Show red warning message |
| testValue = null | Display "—" |

---

## Excess Warning Message

```tsx
{nutrient.hasExcess && (
  <p className="text-red-500 text-sm mt-4 text-center">
    *This nutrient deficiency exceeds the maximum recommended application
    and cannot be resolved only with foliar application.
  </p>
)}
```
