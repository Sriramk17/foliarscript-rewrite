# FoliarScript - Validation Rules

## Script Creation Validation

```python
VALIDATION_RULES = {
    # Required fields
    "field_name": {"required": True, "type": "string"},
    "sample_id": {"required": True, "type": "string"},
    "date": {
        "required": True,
        "format": "YYYY-MM-DD",
        "max": "today",  # Cannot be in future
    },
    "crop": {
        "required": True,
        "values": ["corn", "soybean"]
    },
    "stage": {
        "required": True,
        "dependent_on": "crop",  # Must be valid for selected crop
    },

    # Nutrient validation
    "nutrients": {
        "at_least_one": True,  # At least one nutrient must be provided
        "numeric": True,
        "min": 0,  # No negative values
    }
}
```

---

## Stage Validation by Crop

### Corn Valid Stages
```
ve, v1, v2, v3, v4, v5, v6, v7, v8, v9, v10, v11, v12, v13, v14, v15, v16, vt, r1, r2, r3, r4, r5
```

### Soybean Valid Stages
```
vc, v1, v2, v3, v4, v5, v6, v7, v8, v9, v10, v11, v12, v13, v14, v15, v16, v17, v18, v19, v20, r1, r2, r3, r4, r5
```

---

## CSV Column Normalization

Columns are normalized before processing:
- Strip whitespace
- Replace non-breaking spaces
- Replace spaces with underscores
- Convert to lowercase

```python
def normalize_column(col):
    return col.strip().replace("\xa0", " ").replace(" ", "_").lower()

# Example: "Field Name" → "field_name"
```

---

## File Upload Processing

### Supported Formats
- CSV (.csv)
- Excel (.xlsx, .xls)

### Excel Template Structure
- Sheet name: "Bulk Import Template"
- Header row: Row containing "Field_Name" column
- Data rows: Below header

### Processing Flow

```
1. DETECT FORMAT
   └── CSV vs Excel

2. FIND HEADER ROW
   └── Scan rows for "Field_Name" column

3. NORMALIZE COLUMNS
   └── Clean column names

4. VALIDATE EACH ROW
   ├── Required fields present
   ├── Date format valid
   ├── Crop/Stage valid combination
   ├── At least one nutrient
   └── No negative nutrients

5. IF ERRORS → Return validation errors

6. UPLOAD FILE TO S3
   └── Add timestamp prefix to filename

7. CREATE RECORDS
   ├── Create Fields (if new)
   └── Create FoliarScripts

8. CREATE PAYMENT
   └── Generate payment link

9. RETURN SUCCESS
   └── IDs, payment link
```

---

## Status Transitions

### Script Status
```
Pending ─────────────────→ Completed
           (payment + lab test done)
```

### Payment Status
```
Pending ─────────────────→ Completed
              (payment received)
```

### Recommendation Generation
```
Script created (Pending)
      │
      ▼
Script updated (status=Completed)
      │
      ▼
LabTest.calculate() runs
      │
      ▼
Recommendation record created/updated
```
