# FoliarScript - Business Overview

## What is FoliarScript?

FoliarScript is a **foliar (leaf tissue) nutrient analysis and recommendation platform** for agriculture. It helps growers and agronomists optimize crop nutrition through:

1. **Lab Test Management** - Submit plant tissue samples for nutrient analysis
2. **AI-Powered Recommendations** - Calculate optimal fertilizer application amounts based on test results
3. **Multi-tenant User Hierarchy** - Agronomists manage multiple growers
4. **Bulk Operations** - CSV/Excel upload for batch test submissions

## Core Problem Solved

Farmers need to know:
- Which nutrients their crops are deficient in
- How much fertilizer to apply to reach optimal nutrient levels

FoliarScript automates this by taking lab test results and calculating precise application recommendations based on crop type, growth stage, and current nutrient levels.

## User Roles

| Role | Description |
|------|-------------|
| `FOLIAR_AGRONOMIST` | Professional who manages multiple growers, creates scripts, processes bulk uploads |
| `FOLIAR_GROWER` | Farmer who views their test results and recommendations |

## Key Business Flows

### 1. Script Creation Flow
```
Agronomist/Grower → Submit test data (manual or CSV) → Pay → Wait for results
```

### 2. Recommendation Flow
```
Lab completes test → Status = "Completed" → AI calculates recommendations → User views results
```

### 3. Agronomist-Grower Flow
```
Agronomist creates grower account → Creates scripts on behalf of grower →
Optionally assigns payment to grower → Grower pays and views results
```

## Key Business Rules

1. **Field Auto-Creation**: If field_name doesn't exist for user, create it
2. **Soft Deletes**: All entities use `deleted_at` for soft deletion
3. **Payment Gating**: Recommendations only visible after payment
4. **Grower Assignment**: Agronomist can assign payment responsibility to grower
5. **CSV Batch**: Multiple scripts can be created from one CSV upload
6. **Stage Mapping**: Raw stages normalize to grouped stages for calculations
7. **Deficiency Threshold**: Only recommend application if below "low" threshold
8. **Zero Recommendations**: Return 0 if nutrient is sufficient (>= low)
