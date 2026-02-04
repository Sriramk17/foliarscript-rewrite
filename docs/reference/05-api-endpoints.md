# FoliarScript - API Endpoints

## User Management

### Create User (Register)
```
POST /api/users

Request:
{
  "name": "John",
  "last_name": "Doe",
  "email": "john@example.com",
  "password": "securepass123",
  "type": "FOLIAR_AGRONOMIST",  // or "FOLIAR_GROWER"
  "address": "123 Farm Rd",      // optional
  "zip_code": "12345",           // optional
  "foliar_user_id": 5,           // optional, parent agronomist ID
  "added_by_user_id": 5          // optional, who is creating this user
}

Response:
{
  "id": 123
}
```

### Get Users
```
GET /api/users?foliar_user_id=5&type=FOLIAR_GROWER

Query params:
- userId: specific user ID
- foliar_user_id: get sub-users of this user
- type: filter by user type
- added_by_user_id: users added by this user

Response:
{
  "count": 10,
  "data": [
    {
      "id": 123,
      "name": "John",
      "last_name": "Doe",
      "email": "john@example.com",
      "type": "FOLIAR_GROWER",
      "total_test": 15,
      "total_growers_added": 0,
      "last_test_date": "2024-01-15"
    }
  ]
}
```

### Update User
```
PATCH /api/users?userId=123

Request:
{
  "name": "John Updated",
  "email": "newemail@example.com"
}
```

### Delete User (Soft)
```
DELETE /api/users?userId=123
```

---

## Script Management

### Create Script (Manual)
```
POST /api/scripts

Request:
{
  "foliar_user_id": 123,
  "payment_method": "regular",  // or "free"
  "field_name": "North Field",
  "crop": "corn",
  "stage": "v8",
  "sample_id": "SAMPLE-001",
  "lab_name": "AgriLab",
  "date": "2024-01-15",

  // Macronutrients (percentages)
  "nitrogen_percent": 3.2,
  "phosphorus_percent": 0.35,
  "potassium_percent": 2.1,
  "calcium_percent": 0.8,
  "magnesium_percent": 0.4,
  "sulfur_percent": 0.25,

  // Micronutrients (ppm)
  "zinc_ppm": 25,
  "iron_ppm": 120,
  "manganese_ppm": 45,
  "copper_ppm": 8,
  "boron_ppm": 15,

  // Optional: assign to grower
  "assign_to_grower": true,
  "grower_id": 456
}

Response:
{
  "id": 789,
  "payment_link": "https://payment...",
  "payment_id": 101
}
```

### Create Script (CSV/Excel Upload)
```
POST /api/scripts
Content-Type: multipart/form-data

Form fields:
- file: .csv or .xlsx file
- foliar_user_id: 123
- payment_method: "regular"
- assign_to_grower: true (optional)
- grower_id: 456 (optional)

Required CSV columns:
- Field_Name
- Crop_Name (corn, soybean)
- Stage
- Sample_ID
- Date (YYYY-MM-DD)
- Nitrogen, Phosphorus, Potassium, Calcium, Magnesium, Sulfur
- Zinc, Iron, Manganese, Copper, Boron
- Laboratory (optional)

Response:
{
  "message": "Imported Successfully",
  "ids": [1, 2, 3, 4, 5],
  "payment_link": "https://payment...",
  "payment_id": 123
}
```

### Get Scripts
```
GET /api/scripts?foliar_user_id=123

Query params:
- foliar_script_id: specific script(s), comma-separated
- foliar_user_id: scripts for user
- crop: filter by crop
- status: Pending or Completed
- stage: filter by stage
- field_id: filter by field
- date: filter by date

Response:
{
  "count": 25,
  "pendingTests": [
    {
      "created_at": "2024-01-15T10:00:00Z",
      "date": "2024-01-15",
      "foliar_script_ids": [1, 2, 3],
      "payment_id": 101,
      "payment_link": "https://...",
      "csv_filename": "upload.csv",
      "csv_url": "https://s3.../file.csv",
      "assigned_to_grower": false,
      "grower_id": null
    }
  ],
  "completedTests": [
    {
      "id": 789,
      "foliar_user_id": 123,
      "crop": "corn",
      "stage": "v8",
      "lab_name": "AgriLab",
      "date": "2024-01-15",
      "field_id": 5,
      "field_name": "North Field",
      "status": "Completed",
      "sample_id": "SAMPLE-001"
    }
  ]
}
```

### Update Script (Complete Test)
```
PATCH /api/scripts?foliar_script_id=789

Request:
{
  "status": "Completed",
  "payment_status": "success"
}

Response:
{
  "message": "Foliar script updated"
}

Side effect: When status = "Completed", automatically:
1. Runs LabTest calculation
2. Creates/updates Recommendation record
```

### Delete Script (Soft)
```
DELETE /api/scripts?foliar_script_id=789,790,791
```

---

## Recommendations

### Get Recommendations
```
GET /api/recommendations?foliar_script_id=789

Response:
{
  "count": 1,
  "data": [
    {
      "id": "rec_123",
      "crop": "corn",
      "foliar_script_id": 789,
      "stage": "V6_V16",
      "testdate": "01/15/2024",
      "producer_details": {
        "name": "John Doe",
        "email": "john@example.com",
        "field_name": "North Field",
        "sample_id": "SAMPLE-001",
        "labname": "AgriLab"
      },
      "recommendations": {
        "nitrogen_percent": 12.68,
        "phosphorus_percent": 0,
        "potassium_percent": 5.2,
        "calcium_percent": 0,
        "magnesium_percent": 0,
        "sulfur_percent": 0,
        "zinc_ppm": 8.5,
        "iron_ppm": 0,
        "manganese_ppm": 3.2,
        "copper_ppm": 0,
        "boron_ppm": 2.1
      }
    }
  ]
}
```

---

## Fields

### Get Fields
```
GET /api/fields?foliar_user_id=123

Response:
{
  "count": 5,
  "data": [
    {
      "id": 1,
      "name": "North Field",
      "area": 150.5,
      "foliar_user_id": 123
    }
  ]
}
```

Note: Fields are automatically created when a script is submitted with a new field_name.
