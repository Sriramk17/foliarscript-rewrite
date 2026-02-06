# PDF Export Template - Technical Specification

> Complete implementation spec for generating the FoliarScript Recommendations PDF.

---

## Overview

The PDF template generates a downloadable report containing:

- AgWise logo and "Foliar Script" title
- Producer details (name, crop, stage, field, sample ID, test date)
- Nutrient recommendations table with test values, concentrations, and application amounts
- Split application timeline graphics for nutrients requiring multiple applications
- Excess warnings for nutrients exceeding maximum limits

---

## How It Works

1. A hidden HTML template (`#htmlData`) is rendered in DOM with `display: none`
2. User clicks "Download PDF"
3. Template is temporarily shown
4. `html2canvas` captures the HTML as an image
5. `jsPDF` converts the image to a multi-page PDF
6. PDF is saved and opened in a new tab
7. Template is hidden again

---

## Data Structures

### Producer Details Array

```typescript
interface ProducerDataItem {
  question: string;
  answer: string;
}

// Populated from API response
const data: ProducerDataItem[] = [
  { question: 'Name', answer: formatAnswer(details.name) },
  { question: 'Crop', answer: formatAnswer(details.crop) },
  { question: 'Stage', answer: formatAnswer(details.stage) },
  { question: 'Field', answer: formatAnswer(details.field_name) },
  { question: 'Sample ID', answer: formatAnswer(details.sample_id) },
  { question: 'Test Date', answer: formatAnswer(details.testdate) }
];

function formatAnswer(answer: any): string {
  return (answer === null || answer === undefined || answer === 'null') ? '--' : answer;
}
```

### Nutrient Data

Uses the same `Nutrient` interface as the recommendations page:

```typescript
interface Nutrient {
  name: string;              // "N", "P", "K", etc.
  fullName: string;          // "Nitrogen", "Phosphorus", etc.
  testValue: number;         // Lab test result
  displayedValue: string;    // Concentration value (e.g., "25.00" or "--")
  totalAmount: number;       // Total fl oz to apply
  maxAmountToApply: number;  // Max per single application
  splitAmounts: number[];    // Array of split amounts [5.0, 5.0, 2.5]
  hasExcess: boolean;        // True if exceeds 3x max application
}
```

---

## Visual Layout

```
┌─────────────────────────────────────────────────────────────────┐
│  [AgWise Logo]                              Foliar Script       │
├─────────────────────────────────────────────────────────────────┤
│  Foliar Recommendations, based on your Test Results.           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ Name     │ John Doe    │ Field     │ North Field        │   │
│  │ Crop     │ Corn        │ Sample ID │ LAB-2024-001       │   │
│  │ Stage    │ V6          │ Test Date │ January 15, 2024   │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ Nutrient │ Test Value │ Concentration │ Total App Amount │   │
│  ├──────────┼────────────┼───────────────┼──────────────────┤   │
│  │ Nitrogen │ 2.85 %     │ 25.00 %       │ 12.50 fl oz      │   │
│  │          │            │               │                  │   │
│  │          │            │               │  ①───②───③       │   │
│  │          │            │               │  5.00 5.00 2.50  │   │
│  │          │            │               │  1st  2nd  3rd   │   │
│  ├──────────┼────────────┼───────────────┼──────────────────┤   │
│  │ Phosph.  │ 0.32 %     │ 20.00 %       │ Sufficient       │   │
│  ├──────────┼────────────┼───────────────┼──────────────────┤   │
│  │ Potassium│ 1.85 %     │ 30.00 %       │ 8.00 fl oz       │   │
│  │          │            │               │                  │   │
│  │          │            │               │  ①───②           │   │
│  │          │            │               │  4.00 4.00       │   │
│  │          │            │               │  1st  2nd        │   │
│  └──────────┴────────────┴───────────────┴──────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Timeline Visualization

### 3 Applications

```
     ┌─────┐          ┌─────┐          ┌─────┐
     │  1  │──────────│  2  │──────────│  3  │
     └─────┘  After   └─────┘  After   └─────┘
              1 week           1 week

    5.00 fl oz      5.00 fl oz      2.50 fl oz
  1st Application  2nd Application  3rd Application
```

### 2 Applications

```
     ┌─────┐                    ┌─────┐
     │  1  │────────────────────│  2  │
     └─────┘     After 1 week   └─────┘

    4.00 fl oz                  4.00 fl oz
  1st Application              2nd Application
```

---

## Color Specifications

| Element | Color | Hex Code |
|---------|-------|----------|
| Header background | Dark Navy | `#2C334B` |
| Header text | White | `#FFFFFF` |
| Table header background | Dark Navy | `#2C334B` |
| Table header text | White | `#FFFFFF` |
| Answer cell background | White | `#FFFFFF` |
| Answer cell text | Black | `#000000` |
| Stepper circle (completed) | Blue | `#005ED6` |
| Stepper circle (default) | Gray | `#CCCCCC` |
| Stepper connector line | Gray/Blue | `#CCCCCC` / `#005ED6` |
| Excess warning text | Dark Red | `#7A0000` |
| Border color | Black | `#000000` |

---

## HTML Template Structure (React/Next.js)

```tsx
{/* PDF Template Container - Hidden by default */}
<div className="pdf-wrapper">
  <div id="htmlData" className="hidden">

    {/* Header */}
    <div className="flex justify-between items-center px-8 pt-4">
      <img
        src="/images/agwise-logo-black.png"
        alt="AgWise"
        className="w-[86px] h-[32px]"
      />
      <h1 className="text-lg font-semibold">Foliar Script</h1>
    </div>

    {/* Subtitle */}
    <div className="pl-8 text-sm font-semibold">
      Foliar Recommendations, based on your Test Results.
    </div>

    {/* Producer Details Table (4-column) */}
    <table className="pdf-details-table">
      <tbody>
        <tr>
          {/* Left: First 3 items (Name, Crop, Stage) */}
          <td className="bg-[#2C334B] text-white">
            {data.slice(0, 3).map(item => (
              <div key={item.question}>{item.question}</div>
            ))}
          </td>
          <td className="bg-white text-black">
            {data.slice(0, 3).map(item => (
              <div key={item.question}>{item.answer}</div>
            ))}
          </td>

          {/* Right: Last 3 items (Field, Sample ID, Test Date) */}
          <td className="bg-[#2C334B] text-white">
            {data.slice(3).map(item => (
              <div key={item.question}>{item.question}</div>
            ))}
          </td>
          <td className="bg-white text-black">
            {data.slice(3).map(item => (
              <div key={item.question}>{item.answer}</div>
            ))}
          </td>
        </tr>
      </tbody>
    </table>

    {/* Nutrients Table */}
    <table className="pdf-nutrients-table">
      <thead>
        <tr className="bg-[#2C334B] text-white">
          <th>Nutrient</th>
          <th>Test Value</th>
          <th>Concentration</th>
          <th>Total Application Amount</th>
        </tr>
      </thead>
      <tbody>
        {nutrients.filter(n => n.totalAmount !== -1).map(nutrient => (
          <tr key={nutrient.name}>
            <td>{nutrient.fullName}</td>
            <td>{nutrient.testValue?.toFixed(2) ?? '—'} {NUTRIENT_UNITS[nutrient.fullName]}</td>
            <td>{nutrient.displayedValue} %</td>
            <td>
              {/* Total Amount */}
              <div className="text-center mb-4">
                {nutrient.totalAmount === 0
                  ? 'Sufficient'
                  : `${nutrient.totalAmount.toFixed(2)} fl oz`}
              </div>

              {/* Timeline (if multiple applications) */}
              {nutrient.splitAmounts.length > 1 && (
                <>
                  <div className="stepper-wrapper">
                    {nutrient.splitAmounts.slice(0, 3).map((amount, j) => (
                      <div key={j} className="stepper-item">
                        <div className="step-counter bg-[#005ED6]">
                          {j + 1}
                        </div>
                        <div className="step-name">
                          <b>
                            {j === 2 && nutrient.splitAmounts.length > 3
                              ? nutrient.maxAmountToApply
                              : amount} fl oz
                          </b>
                          <br />
                          <small>{j + 1}{getOrdinalSuffix(j + 1)} Application</small>
                        </div>

                        {/* Separator */}
                        {j < Math.min(nutrient.splitAmounts.length, 3) - 1 && (
                          <div className="step-separator">After 1 week</div>
                        )}
                      </div>
                    ))}
                  </div>

                  {/* Excess Warning */}
                  {nutrient.hasExcess && (
                    <p className="text-[#7A0000] text-xs mt-2">
                      This nutrient deficiency exceeds the maximum recommended application
                      and cannot be resolved only with foliar application.
                    </p>
                  )}
                </>
              )}
            </td>
          </tr>
        ))}
      </tbody>
    </table>

  </div>
</div>
```

---

## CSS Styles

```css
/* PDF Container */
.pdf-wrapper {
  /* Hidden template styles */
  #htmlData {
    width: 595pt; /* A4 width in points */
    margin: 0 auto;
    font-family: 'Poppins', sans-serif;
    background: white;
  }

  #htmlData.hidden {
    display: none;
  }
}

/* Producer Details Table */
.pdf-details-table {
  width: 740px;
  margin: 10pt 20pt;
  border-radius: 3px;
  border: 1px solid black;
  border-collapse: collapse;
  font-size: 12px;
}

.pdf-details-table td {
  padding: 15px;
  border: 1px solid black;
}

/* Nutrients Table */
.pdf-nutrients-table {
  width: 740px;
  margin: 10pt 20pt;
  border-collapse: collapse;
  font-size: 12px;
  border: 1px solid black;
}

.pdf-nutrients-table th {
  background-color: #2C334B;
  color: white;
  font-weight: 600;
  padding: 10px;
}

.pdf-nutrients-table td {
  border-top: 1px solid #2C334B;
  border-bottom: 1px solid #2C334B;
  font-weight: bold;
  padding: 10px;
}

/* Stepper/Timeline */
.stepper-wrapper {
  display: flex;
  justify-content: space-between;
  margin: 20px 0 15px;
}

.stepper-item {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  flex: 1;
}

.step-counter {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 30px;
  height: 30px;
  border-radius: 50%;
  background: #005ED6;
  color: white;
  font-size: 12pt;
  margin-bottom: 6px;
  z-index: 5;
}

.step-name {
  text-align: center;
  line-height: normal;
}

.step-name small {
  font-size: 10px;
}

/* Connector lines between steps */
.stepper-item::after {
  content: "";
  position: absolute;
  border-bottom: 2px solid #005ED6;
  width: 100%;
  top: 15px;
  left: 50%;
  z-index: 2;
}

.stepper-item:last-child::after {
  content: none;
}

.step-separator {
  position: absolute;
  font-size: 10px;
  width: 70px;
  margin-top: -7px;
  text-align: center;
}
```

---

## PDF Generation Function

```typescript
import html2canvas from 'html2canvas';
import jsPDF from 'jspdf';

async function generatePDF(details: ProducerDetails): Promise<void> {
  const element = document.getElementById('htmlData')!;

  // Temporarily show the template
  element.style.display = 'block';

  const options = {
    useCORS: true,
    logging: false,
    imageTimeout: 2000,
    removeContainer: true,
    allowTaint: true,
    scale: 2, // Higher quality
  };

  try {
    const canvas = await html2canvas(element, options);
    const imgData = canvas.toDataURL('image/png');

    // A4 dimensions in mm
    const imgWidth = 210;
    const pageHeight = 297;
    const imgHeight = (canvas.height * imgWidth) / canvas.width;

    const doc = new jsPDF({
      orientation: 'portrait',
      unit: 'mm',
      format: 'a4',
    });

    // Handle multi-page content
    let heightLeft = imgHeight;
    let position = 0;

    // First page
    doc.addImage(imgData, 'PNG', 0, position, imgWidth, imgHeight);
    heightLeft -= pageHeight;

    // Additional pages
    while (heightLeft > 0) {
      position = heightLeft - imgHeight;
      doc.addPage();
      doc.addImage(imgData, 'PNG', 0, position, imgWidth, imgHeight);
      heightLeft -= pageHeight;
    }

    // Save with producer name
    const fileName = `${details.name}_foliar_script.pdf`;
    doc.save(fileName);

    // Also open in new tab
    const pdfUrl = URL.createObjectURL(doc.output('blob'));
    window.open(pdfUrl);

  } finally {
    // Hide template again
    element.style.display = 'none';
  }
}
```

---

## Helper Functions

```typescript
function getOrdinalSuffix(n: number): string {
  if (n % 10 === 1 && n % 100 !== 11) return "st";
  if (n % 10 === 2 && n % 100 !== 12) return "nd";
  if (n % 10 === 3 && n % 100 !== 13) return "rd";
  return "th";
}

function formatAnswer(answer: any): string {
  return (answer === null || answer === undefined || answer === 'null')
    ? '--'
    : String(answer);
}
```

---

## Dependencies

```bash
npm install html2canvas jspdf
```

---

## Required Assets

| Asset | Path | Dimensions |
|-------|------|------------|
| AgWise Logo (Black) | `/images/agwise-logo-black.png` | 86px × 32px |

---

## Implementation Notes

1. **Template Must Be Hidden**: The `#htmlData` div must have `display: none` by default

2. **Width Matters**: PDF container is set to `595pt` (A4 width) for proper scaling

3. **Max 3 Circles**: Even if more than 3 split applications, only show 3 circles. For the 3rd, display `maxAmountToApply` instead of actual split amount

4. **Multi-Page Handling**: Content exceeding one A4 page automatically adds pages

5. **File Naming**: PDF filename pattern: `{producer_name}_foliar_script.pdf`

6. **Hide After Capture**: `element.style.display = 'none'` must be called AFTER the promise resolves

7. **CORS for Images**: Ensure logo image has CORS enabled or is a local asset
