# FoliarScript - Data Models

## Entity Relationship Overview

```
FoliarUser (1) ←────────→ (M) FoliarScript
     │
     ├── foliar_user (self) ← Grower points to Agronomist
     └── added_by_user (self) ← Who created this user

FoliarUser (1) ←────────→ (M) Field
FoliarUser (1) ←────────→ (M) FoliarPayments

FoliarScript (1) ←───────→ (1) Recommendation
FoliarScript (M) ←───────→ (1) Field
FoliarScript (M) ←───────→ (1) Laboratory
```

---

## FoliarUser

The core user entity with self-referential hierarchy.

```
FoliarUser
├── id                      : integer (PK)
├── user_id                 : integer (FK → Auth User)
├── name                    : string (required)
├── last_name               : string
├── email                   : string (unique)
├── type                    : enum ['FOLIAR_AGRONOMIST', 'FOLIAR_GROWER']
├── foliar_user_id          : integer (FK → self, parent agronomist)
├── added_by_user_id        : integer (FK → self, who created this user)
├── address                 : string
├── zip_code                : string
├── is_active               : boolean (default: false)
├── is_password_set         : boolean (default: false)
├── activation_token        : string
├── activation_uid          : string
├── activate_expiry         : datetime
├── last_login              : datetime
├── deleted_at              : datetime (soft delete)
├── created_at              : datetime
└── updated_at              : datetime
```

**Hierarchy Logic:**
- `foliar_user_id` → Points to parent agronomist (for growers managed by agronomist)
- `added_by_user_id` → Points to user who created this account

---

## FoliarScript

The main test script entity containing lab test results.

```
FoliarScript
├── id                      : integer (PK)
├── foliar_user_id          : integer (FK → FoliarUser)
├── field_id                : integer (FK → Field)
├── laboratory_id           : integer (FK → Laboratory, optional)
│
├── # TEST METADATA
├── crop                    : string ['corn', 'soybean']
├── stage                   : string (growth stage code)
├── sample_id               : string
├── lab_name                : string
├── date                    : date
├── status                  : enum ['Pending', 'Completed']
│
├── # MACRONUTRIENTS (percentages, 0-100)
├── nitrogen_percent        : float
├── phosphorus_percent      : float
├── potassium_percent       : float
├── calcium_percent         : float
├── magnesium_percent       : float
├── sulfur_percent          : float
│
├── # MICRONUTRIENTS (ppm values)
├── zinc_ppm                : float
├── iron_ppm                : float
├── manganese_ppm           : float
├── copper_ppm              : float
├── boron_ppm               : float
├── molybdenum_ppm          : float (optional, currently disabled)
├── aluminum_ppm            : float (optional)
├── sodium_ppm              : float (optional)
│
├── # FILE TRACKING
├── csv_url                 : string (S3 URL if uploaded via CSV)
├── csv_filename            : string (original filename)
├── email_key               : string (for email handler tracking)
│
├── # PAYMENT (can be separate service)
├── payment_method          : enum ['regular', 'free']
├── payment_status          : string
├── payment_transaction_id  : string
├── payment_link            : string
├── payment_initiated_at    : datetime
├── payment_completed_at    : datetime
│
├── deleted_at              : datetime (soft delete)
├── created_at              : datetime
└── updated_at              : datetime
```

---

## Recommendation

Stores AI-calculated nutrient recommendations.

```
Recommendation
├── id                      : integer (PK)
├── foliar_script_id        : integer (FK → FoliarScript, unique)
├── crop                    : string
├── date                    : date
├── model_version           : string
├── access_link             : string
│
├── # RECOMMENDED APPLICATION AMOUNTS (oz/acre to apply)
├── nitrogen_percent        : float
├── phosphorus_percent      : float
├── potassium_percent       : float
├── calcium_percent         : float
├── magnesium_percent       : float
├── sulfur_percent          : float
├── zinc_ppm                : float
├── iron_ppm                : float
├── manganese_ppm           : float
├── copper_ppm              : float
├── boron_ppm               : float
├── molybdenum_ppm          : float
├── aluminum_ppm            : float
│
├── created_at              : datetime
└── updated_at              : datetime
```

---

## Field

Geographic field/location entity.

```
Field
├── id                      : integer (PK)
├── foliar_user_id          : integer (FK → FoliarUser)
├── name                    : string (required)
├── area                    : float (acres)
├── coordinates             : json (geolocation data)
├── status                  : string
├── deleted_at              : datetime (soft delete)
├── created_at              : datetime
└── updated_at              : datetime
```

---

## Laboratory

Lab reference data.

```
Laboratory
├── id                      : integer (PK)
├── name                    : string
├── email                   : string
├── deleted_at              : datetime (soft delete)
├── created_at              : datetime
└── updated_at              : datetime
```

---

## FoliarPayments

Payment tracking entity.

```
FoliarPayments
├── id                      : integer (PK)
├── foliar_user_id          : integer (FK → FoliarUser)
├── status                  : enum ['Pending', 'Completed']
├── grower_id               : integer (assigned grower for payment)
├── pending_by_grower       : boolean (true if grower should pay)
├── foliar_script_ids       : json array [int, int, ...]
├── payment_link            : string (Stripe URL)
├── deleted_at              : datetime (soft delete)
├── created_at              : datetime
└── updated_at              : datetime
```
