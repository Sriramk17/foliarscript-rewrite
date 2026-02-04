# FoliarScript - Recommendation Engine

This is the heart of FoliarScript - the AI/ML logic that calculates how much fertilizer to apply.

## Stage Mapping

Maps raw growth stage inputs to normalized stage codes:

```python
STAGE_MAP = {
    "corn": {
        # Early vegetative stages → VE_V5
        "ve": "VE_V5", "v1": "VE_V5", "v2": "VE_V5",
        "v3": "VE_V5", "v4": "VE_V5", "v5": "VE_V5",

        # Mid vegetative stages → V6_V16
        "v6": "V6_V16", "v7": "V6_V16", "v8": "V6_V16",
        "v9": "V6_V16", "v10": "V6_V16", "v11": "V6_V16",
        "v12": "V6_V16", "v13": "V6_V16", "v14": "V6_V16",
        "v15": "V6_V16", "v16": "V6_V16",

        # Tasseling/early reproductive → VT_R2
        "vt": "VT_R2", "r1": "VT_R2", "r2": "VT_R2",

        # Late reproductive → R3_R5
        "r3": "R3_R5", "r4": "R3_R5", "r5": "R3_R5"
    },
    "soybean": {
        # Early vegetative → VC_V5
        "vc": "VC_V5", "v1": "VC_V5", "v2": "VC_V5",
        "v3": "VC_V5", "v4": "VC_V5", "v5": "VC_V5",

        # Mid vegetative → V6_V20
        "v6": "V6_V20", "v7": "V6_V20", "v8": "V6_V20",
        "v9": "V6_V20", "v10": "V6_V20", "v11": "V6_V20",
        "v12": "V6_V20", "v13": "V6_V20", "v14": "V6_V20",
        "v15": "V6_V20", "v16": "V6_V20", "v17": "V6_V20",
        "v18": "V6_V20", "v19": "V6_V20", "v20": "V6_V20",

        # Reproductive stages (individual)
        "r1": "R1", "r2": "R2", "r3": "R3",

        # Late reproductive → R4_R5
        "r4": "R4_R5", "r5": "R4_R5"
    }
}
```

---

## Default Concentrations

Standard fertilizer product concentrations (percentage of nutrient in product):

```python
DEFAULT_CONCENTRATION = {
    "N": 10,      # Nitrogen - 10%
    "P": 19,      # Phosphorus - 19%
    "K": 24,      # Potassium - 24%
    "Mg": 2.5,    # Magnesium - 2.5%
    "Ca": 10,     # Calcium - 10%
    "S": 17,      # Sulfur - 17%
    "Fe": 4.5,    # Iron - 4.5%
    "Mn": 8,      # Manganese - 8%
    "Cu": 8,      # Copper - 8%
    "B": 10,      # Boron - 10%
    "Z": 9,       # Zinc - 9%
    "Moly": 3     # Molybdenum - 3%
}
```

---

## Maximum Application Limits

Safety limits for maximum nutrient application per acre:

```python
MAX_APPLY = {
    "N":    {"concentrate": 21,   "value": 96},   # Max 96 oz/acre
    "P":    {"concentrate": 19,   "value": 128},  # Max 128 oz/acre
    "K":    {"concentrate": 24,   "value": 192},  # Max 192 oz/acre
    "Mg":   {"concentrate": 2.5,  "value": 32},   # Max 32 oz/acre
    "Ca":   {"concentrate": 10,   "value": 96},   # Max 96 oz/acre
    "S":    {"concentrate": 17,   "value": 96},   # Max 96 oz/acre
    "Fe":   {"concentrate": 4.5,  "value": 32},   # Max 32 oz/acre
    "Mn":   {"concentrate": 8,    "value": 72},   # Max 72 oz/acre
    "Cu":   {"concentrate": 8,    "value": 32},   # Max 32 oz/acre
    "B":    {"concentrate": 10,   "value": 72},   # Max 72 oz/acre
    "Z":    {"concentrate": 9,    "value": 32},   # Max 32 oz/acre
    "Moly": {"concentrate": 3,    "value": 16}    # Max 16 oz/acre
}
```

---

## Calculation Algorithm

### Core Formula

```python
def calculate(nutrient, nutrient_value, crop, stage):
    """
    Calculate recommended application amount for a single nutrient.

    Formula:
    1. If nutrient_value >= low threshold → return 0 (no deficiency)
    2. Otherwise:
       - change = hit - nutrient_value (how much to increase)
       - std = change_percentage / (rate * concentrate)
       - amount_to_apply = change / (std * default_concentration[nutrient])

    Returns:
        float: Recommended oz/acre to apply (0 if sufficient)
    """
    # Get model parameters for this crop/nutrient/stage
    model_data = MODEL[crop][nutrient][stage]
    low = model_data["ranges"]["low"]
    hit = model_data["hit"]
    rate = model_data["apply_assumption"]["rate"]
    concentrate = model_data["apply_assumption"]["concentrate"]
    change_percentage = model_data["apply_assumption"]["change_percentage"]

    # If change_percentage is "na", no recommendation available
    if change_percentage == "na":
        return 0

    # If nutrient is sufficient (>= low threshold), no application needed
    if nutrient_value >= low:
        return 0

    # Calculate deficiency and required application
    std = change_percentage / (rate * concentrate)
    change = hit - nutrient_value
    amount_to_apply = change / (std * DEFAULT_CONCENTRATION[nutrient])

    return amount_to_apply
```

### Example Calculation

```python
# Example: Corn at V8 stage with nitrogen test result of 3.2%

# Calculation breakdown for Nitrogen:
# 1. Stage v8 maps to V6_V16
# 2. Model for corn/N/V6_V16:
#    - low: 3.8, hit: 3.85
#    - rate: 48, concentrate: 5, change_percentage: 1.23
# 3. Since 3.2 < 3.8 (low), crop is deficient
# 4. std = 1.23 / (48 * 5) = 0.005125
# 5. change = 3.85 - 3.2 = 0.65
# 6. amount = 0.65 / (0.005125 * 10) = 12.68 oz/acre

# Result: {"N": 12.68}
```

---

## LabTest Class Structure

```python
class LabTest:
    """
    Calculate nutrient application recommendations based on lab test results.

    Args:
        crop: Crop type ('corn' or 'soybean')
        stage: Growth stage (e.g., 'v6', 'r1')
        N, P, K, etc.: Nutrient test values (-1 means not tested)
    """

    def __init__(self, crop, stage, N=-1, P=-1, K=-1, Mg=-1, Ca=-1, S=-1,
                 Fe=-1, Mn=-1, Cu=-1, B=-1, Z=-1, Moly=-1):
        self.crop = crop.lower()
        self.stage = STAGE_MAP[crop][stage.lower()]  # Normalize stage

        # Store test values
        self.N_value = N
        self.P_value = P
        # ... etc

    def calculate(self, nutrient, nutrient_value, is_ppm):
        # Core calculation logic
        pass

    def report(self):
        """
        Generate complete recommendation report for all nutrients.

        Returns:
            dict: {nutrient_code: recommended_amount, ...}
        """
        results = {}

        # Macronutrients (percentages)
        if self.N_value != -1:
            results["N"] = self.calculate("N", self.N_value, False)
        # ... etc for all nutrients

        return results
```
