# Phase 1.3: Database Schema Design

> **Goal**: All database tables created with proper indexes and RLS
> **Time**: ~1 hour
> **Dependencies**: Phase 1.2 complete

---

## Schema Overview

```
┌─────────────────┐
│   auth.users    │  ← Supabase managed
└────────┬────────┘
         │
         │ 1:1
         ▼
┌─────────────────┐
│    profiles     │  ← Extended user data
└────────┬────────┘
         │
         │ 1:M
         ▼
┌─────────────────┐     ┌─────────────────┐
│    clients      │     │     fields      │
└────────┬────────┘     └────────┬────────┘
         │                       │
         │ M:1                   │ M:1
         ▼                       ▼
┌──────────────────────────────────────────┐
│               scripts                     │
└────────────────────┬─────────────────────┘
                     │
                     │ 1:1
                     ▼
┌──────────────────────────────────────────┐
│           recommendations                 │
└──────────────────────────────────────────┘
```

---

## SQL Migration

Run this in Supabase SQL Editor:

```sql
-- ============================================
-- PROFILES TABLE (extends auth.users)
-- ============================================
CREATE TABLE public.profiles (
    id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
    name TEXT NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Auto-create profile on user signup
CREATE OR REPLACE FUNCTION public.handle_new_user()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO public.profiles (id, name)
    VALUES (NEW.id, COALESCE(NEW.raw_user_meta_data->>'name', 'User'));
    RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE TRIGGER on_auth_user_created
    AFTER INSERT ON auth.users
    FOR EACH ROW EXECUTE FUNCTION public.handle_new_user();

-- RLS for profiles
ALTER TABLE public.profiles ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can view own profile"
    ON public.profiles FOR SELECT
    USING (auth.uid() = id);

CREATE POLICY "Users can update own profile"
    ON public.profiles FOR UPDATE
    USING (auth.uid() = id);


-- ============================================
-- CLIENTS TABLE
-- ============================================
CREATE TABLE public.clients (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
    name TEXT NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_clients_user_id ON public.clients(user_id);

-- RLS for clients
ALTER TABLE public.clients ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can CRUD own clients"
    ON public.clients FOR ALL
    USING (auth.uid() = user_id);


-- ============================================
-- FIELDS TABLE
-- ============================================
CREATE TABLE public.fields (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
    name TEXT NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE(user_id, name)  -- Field names unique per user
);

CREATE INDEX idx_fields_user_id ON public.fields(user_id);

-- RLS for fields
ALTER TABLE public.fields ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can CRUD own fields"
    ON public.fields FOR ALL
    USING (auth.uid() = user_id);


-- ============================================
-- SCRIPTS TABLE
-- ============================================
CREATE TYPE script_status AS ENUM ('draft', 'unpaid', 'paid');
CREATE TYPE crop_type AS ENUM ('corn', 'soybean');

CREATE TABLE public.scripts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
    client_id UUID REFERENCES public.clients(id) ON DELETE SET NULL,
    field_id UUID REFERENCES public.fields(id) ON DELETE SET NULL,

    -- Metadata
    crop crop_type NOT NULL,
    stage TEXT NOT NULL,
    sample_id TEXT NOT NULL,
    lab_name TEXT,
    test_date DATE NOT NULL,
    status script_status DEFAULT 'draft',

    -- Macronutrients (percentages)
    nitrogen DECIMAL(5,3),
    phosphorus DECIMAL(5,3),
    potassium DECIMAL(5,3),
    calcium DECIMAL(5,3),
    magnesium DECIMAL(5,3),
    sulfur DECIMAL(5,3),

    -- Micronutrients (ppm)
    zinc DECIMAL(7,2),
    iron DECIMAL(7,2),
    manganese DECIMAL(7,2),
    copper DECIMAL(7,2),
    boron DECIMAL(7,2),
    molybdenum DECIMAL(7,3),

    -- File tracking
    source_file_url TEXT,
    source_filename TEXT,

    -- Payment
    stripe_payment_intent_id TEXT,
    stripe_checkout_session_id TEXT,
    paid_at TIMESTAMPTZ,

    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_scripts_user_id ON public.scripts(user_id);
CREATE INDEX idx_scripts_status ON public.scripts(status);
CREATE INDEX idx_scripts_field_id ON public.scripts(field_id);
CREATE INDEX idx_scripts_created_at ON public.scripts(created_at DESC);

-- RLS for scripts
ALTER TABLE public.scripts ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can CRUD own scripts"
    ON public.scripts FOR ALL
    USING (auth.uid() = user_id);


-- ============================================
-- RECOMMENDATIONS TABLE
-- ============================================
CREATE TABLE public.recommendations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    script_id UUID NOT NULL UNIQUE REFERENCES public.scripts(id) ON DELETE CASCADE,
    model_version TEXT NOT NULL DEFAULT '2.0.0',

    -- Calculated amounts (oz/acre)
    nitrogen DECIMAL(7,2) DEFAULT 0,
    phosphorus DECIMAL(7,2) DEFAULT 0,
    potassium DECIMAL(7,2) DEFAULT 0,
    calcium DECIMAL(7,2) DEFAULT 0,
    magnesium DECIMAL(7,2) DEFAULT 0,
    sulfur DECIMAL(7,2) DEFAULT 0,
    zinc DECIMAL(7,2) DEFAULT 0,
    iron DECIMAL(7,2) DEFAULT 0,
    manganese DECIMAL(7,2) DEFAULT 0,
    copper DECIMAL(7,2) DEFAULT 0,
    boron DECIMAL(7,2) DEFAULT 0,
    molybdenum DECIMAL(7,2) DEFAULT 0,

    -- Additional calculation data (stored for PDF/display)
    calculation_data JSONB,

    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_recommendations_script_id ON public.recommendations(script_id);

-- RLS for recommendations (via script ownership)
ALTER TABLE public.recommendations ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can view recommendations for own scripts"
    ON public.recommendations FOR SELECT
    USING (
        EXISTS (
            SELECT 1 FROM public.scripts
            WHERE scripts.id = recommendations.script_id
            AND scripts.user_id = auth.uid()
        )
    );


-- ============================================
-- UPDATED_AT TRIGGER
-- ============================================
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER profiles_updated_at
    BEFORE UPDATE ON public.profiles
    FOR EACH ROW EXECUTE FUNCTION update_updated_at();

CREATE TRIGGER clients_updated_at
    BEFORE UPDATE ON public.clients
    FOR EACH ROW EXECUTE FUNCTION update_updated_at();

CREATE TRIGGER fields_updated_at
    BEFORE UPDATE ON public.fields
    FOR EACH ROW EXECUTE FUNCTION update_updated_at();

CREATE TRIGGER scripts_updated_at
    BEFORE UPDATE ON public.scripts
    FOR EACH ROW EXECUTE FUNCTION update_updated_at();

CREATE TRIGGER recommendations_updated_at
    BEFORE UPDATE ON public.recommendations
    FOR EACH ROW EXECUTE FUNCTION update_updated_at();
```

---

## TypeScript Types (Generate from Supabase)

After running migration, generate types:

```bash
npx supabase gen types typescript --project-id YOUR_PROJECT_ID > frontend/src/lib/database.types.ts
```

Or manually create:

```typescript
// frontend/src/lib/database.types.ts
export type ScriptStatus = 'draft' | 'unpaid' | 'paid'
export type CropType = 'corn' | 'soybean'

export interface Profile {
  id: string
  name: string
  created_at: string
  updated_at: string
}

export interface Client {
  id: string
  user_id: string
  name: string
  created_at: string
  updated_at: string
}

export interface Field {
  id: string
  user_id: string
  name: string
  created_at: string
  updated_at: string
}

export interface Script {
  id: string
  user_id: string
  client_id: string | null
  field_id: string | null
  crop: CropType
  stage: string
  sample_id: string
  lab_name: string | null
  test_date: string
  status: ScriptStatus
  nitrogen: number | null
  phosphorus: number | null
  potassium: number | null
  calcium: number | null
  magnesium: number | null
  sulfur: number | null
  zinc: number | null
  iron: number | null
  manganese: number | null
  copper: number | null
  boron: number | null
  molybdenum: number | null
  source_file_url: string | null
  source_filename: string | null
  stripe_payment_intent_id: string | null
  stripe_checkout_session_id: string | null
  paid_at: string | null
  created_at: string
  updated_at: string
}

export interface Recommendation {
  id: string
  script_id: string
  model_version: string
  nitrogen: number
  phosphorus: number
  potassium: number
  calcium: number
  magnesium: number
  sulfur: number
  zinc: number
  iron: number
  manganese: number
  copper: number
  boron: number
  molybdenum: number
  calculation_data: Record<string, any> | null
  created_at: string
  updated_at: string
}
```

---

## Pydantic Models (Backend)

```python
# backend/app/models/schemas.py
from pydantic import BaseModel
from datetime import date, datetime
from typing import Optional, Literal
from uuid import UUID

ScriptStatus = Literal['draft', 'unpaid', 'paid']
CropType = Literal['corn', 'soybean']

class ProfileBase(BaseModel):
    name: str

class Profile(ProfileBase):
    id: UUID
    created_at: datetime
    updated_at: datetime

class ClientCreate(BaseModel):
    name: str

class Client(ClientCreate):
    id: UUID
    user_id: UUID
    created_at: datetime
    updated_at: datetime

class FieldCreate(BaseModel):
    name: str

class Field(FieldCreate):
    id: UUID
    user_id: UUID
    created_at: datetime
    updated_at: datetime

class ScriptCreate(BaseModel):
    client_id: Optional[UUID] = None
    field_id: Optional[UUID] = None
    crop: CropType
    stage: str
    sample_id: str
    lab_name: Optional[str] = None
    test_date: date
    nitrogen: Optional[float] = None
    phosphorus: Optional[float] = None
    potassium: Optional[float] = None
    calcium: Optional[float] = None
    magnesium: Optional[float] = None
    sulfur: Optional[float] = None
    zinc: Optional[float] = None
    iron: Optional[float] = None
    manganese: Optional[float] = None
    copper: Optional[float] = None
    boron: Optional[float] = None
    molybdenum: Optional[float] = None

class Script(ScriptCreate):
    id: UUID
    user_id: UUID
    status: ScriptStatus
    source_file_url: Optional[str] = None
    source_filename: Optional[str] = None
    stripe_payment_intent_id: Optional[str] = None
    paid_at: Optional[datetime] = None
    created_at: datetime
    updated_at: datetime

class Recommendation(BaseModel):
    id: UUID
    script_id: UUID
    model_version: str
    nitrogen: float
    phosphorus: float
    potassium: float
    calcium: float
    magnesium: float
    sulfur: float
    zinc: float
    iron: float
    manganese: float
    copper: float
    boron: float
    molybdenum: float
    calculation_data: Optional[dict] = None
    created_at: datetime
    updated_at: datetime
```

---

## Verification Checklist

- [ ] All tables created in Supabase
- [ ] RLS policies active
- [ ] Indexes created
- [ ] Auto-profile creation works (test signup)
- [ ] TypeScript types generated
- [ ] Pydantic models created

---

## Next Step

→ `plans/phase-1-foundation/04-auth-flow.md`
