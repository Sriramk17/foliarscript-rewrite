# FoliarScript - Architecture Recommendations

## Suggested Tech Stack

| Layer | Recommendation |
|-------|----------------|
| API Framework | FastAPI (Python) or NestJS (Node) |
| Validation | Pydantic (Python) or class-validator (Node) |
| Database | PostgreSQL with proper indexes |
| ORM | SQLAlchemy (Python) or Prisma (Node) |
| File Storage | S3 or compatible |
| Queue | Redis + Celery/BullMQ for async |
| Config | Environment variables + config files |

---

## Suggested Module Structure

```
/src
├── /api
│   ├── /users
│   ├── /scripts
│   ├── /recommendations
│   └── /fields
├── /core
│   ├── /models           # Database entities
│   ├── /schemas          # Request/Response DTOs
│   └── /services         # Business logic
├── /recommendation_engine
│   ├── model.py          # Threshold model data
│   ├── calculator.py     # LabTest calculation logic
│   └── stage_mapping.py  # Stage normalization
├── /file_processing
│   ├── csv_parser.py
│   └── validators.py
└── /config
    └── settings.py
```

---

## Key Improvements to Consider

1. **Separate Recommendation Service**: Make it a microservice for scalability
2. **Model Versioning**: Store model version with each recommendation
3. **Audit Trail**: Track all changes to scripts
4. **API Versioning**: Use `/api/v1/` prefix
5. **Rate Limiting**: Protect endpoints
6. **Caching**: Cache model data, stage mappings
7. **Error Codes**: Structured error responses with codes
8. **Pagination**: Proper cursor-based pagination

---

## Database Indexes (Recommended)

```sql
-- FoliarUser
CREATE INDEX idx_foliar_user_email ON foliar_user(email) WHERE deleted_at IS NULL;
CREATE INDEX idx_foliar_user_type ON foliar_user(type) WHERE deleted_at IS NULL;
CREATE INDEX idx_foliar_user_parent ON foliar_user(foliar_user_id) WHERE deleted_at IS NULL;

-- FoliarScript
CREATE INDEX idx_foliar_script_user ON foliar_script(foliar_user_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_foliar_script_status ON foliar_script(status) WHERE deleted_at IS NULL;
CREATE INDEX idx_foliar_script_crop ON foliar_script(crop) WHERE deleted_at IS NULL;
CREATE INDEX idx_foliar_script_date ON foliar_script(date) WHERE deleted_at IS NULL;

-- Field
CREATE INDEX idx_field_user ON field(foliar_user_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_field_name ON field(name, foliar_user_id) WHERE deleted_at IS NULL;

-- Recommendation
CREATE UNIQUE INDEX idx_recommendation_script ON recommendation(foliar_script_id);
```

---

## Error Response Format

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid crop type",
    "details": [
      {
        "field": "crop",
        "message": "Must be one of: corn, soybean",
        "received": "wheat"
      }
    ]
  }
}
```

### Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `VALIDATION_ERROR` | 400 | Request validation failed |
| `NOT_FOUND` | 404 | Resource not found |
| `UNAUTHORIZED` | 401 | Authentication required |
| `FORBIDDEN` | 403 | Insufficient permissions |
| `DUPLICATE_ENTRY` | 409 | Resource already exists |
| `PAYMENT_REQUIRED` | 402 | Payment needed |
| `INTERNAL_ERROR` | 500 | Server error |

---

## API Response Envelope

```json
{
  "success": true,
  "data": { ... },
  "meta": {
    "count": 25,
    "page": 1,
    "per_page": 10,
    "total_pages": 3
  }
}
```

---

## Environment Variables

```bash
# Database
DATABASE_URL=postgresql://user:pass@host:5432/foliarscript

# AWS S3
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_REGION=us-east-1
S3_BUCKET=foliarscript-uploads

# Stripe (Payments)
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=

# Application
APP_ENV=production
APP_SECRET_KEY=
JWT_SECRET=
JWT_EXPIRY=86400

# Redis (optional, for caching/queues)
REDIS_URL=redis://localhost:6379
```
