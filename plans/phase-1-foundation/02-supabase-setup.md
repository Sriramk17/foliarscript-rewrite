# Phase 1.2: Supabase Setup

> **Goal**: Supabase project with auth, database, and storage configured
> **Time**: ~1 hour
> **Dependencies**: Phase 1.1 complete

---

## What We're Setting Up

1. **Supabase Project** - Cloud instance
2. **Authentication** - Email/password with email verification
3. **Database** - PostgreSQL with our schema
4. **Storage** - Bucket for file uploads (CSV/PDF)

---

## Steps

### 1. Create Supabase Project

1. Go to [supabase.com](https://supabase.com)
2. Create new project
3. Choose region closest to users (likely US East)
4. Save the credentials:
   - Project URL: `https://xxxxx.supabase.co`
   - Anon Key: `eyJhbGc...`
   - Service Role Key: `eyJhbGc...` (keep secret!)

### 2. Configure Authentication

In Supabase Dashboard → Authentication → Providers:

- [x] Email enabled
- [x] Confirm email enabled
- [ ] Phone disabled
- [ ] All OAuth providers disabled (for now)

In Authentication → URL Configuration:
- Site URL: `http://localhost:3000` (dev) → update for prod
- Redirect URLs: `http://localhost:3000/auth/callback`

### 3. Create Environment Files

**Frontend** (`frontend/.env.local`):

```env
NEXT_PUBLIC_SUPABASE_URL=https://xxxxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGc...
```

**Backend** (`backend/.env`):

```env
SUPABASE_URL=https://xxxxx.supabase.co
SUPABASE_KEY=eyJhbGc...  # Service role key
SUPABASE_JWT_SECRET=your-jwt-secret

# Stripe (placeholder for now)
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...

# App
APP_ENV=development
CORS_ORIGINS=http://localhost:3000
```

### 4. Create Supabase Client (Frontend)

```typescript
// frontend/src/lib/supabase/client.ts
import { createBrowserClient } from '@supabase/ssr'

export function createClient() {
  return createBrowserClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  )
}
```

```typescript
// frontend/src/lib/supabase/server.ts
import { createServerClient, type CookieOptions } from '@supabase/ssr'
import { cookies } from 'next/headers'

export async function createClient() {
  const cookieStore = await cookies()

  return createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll() {
          return cookieStore.getAll()
        },
        setAll(cookiesToSet) {
          try {
            cookiesToSet.forEach(({ name, value, options }) =>
              cookieStore.set(name, value, options)
            )
          } catch {
            // Server Component - ignore
          }
        },
      },
    }
  )
}
```

### 5. Create Supabase Client (Backend)

```python
# backend/app/core/supabase.py
from supabase import create_client, Client
from app.core.config import settings

supabase: Client = create_client(
    settings.SUPABASE_URL,
    settings.SUPABASE_KEY
)

def get_supabase() -> Client:
    return supabase
```

```python
# backend/app/core/config.py
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    SUPABASE_URL: str
    SUPABASE_KEY: str
    SUPABASE_JWT_SECRET: str

    STRIPE_SECRET_KEY: str = ""
    STRIPE_WEBHOOK_SECRET: str = ""

    APP_ENV: str = "development"
    CORS_ORIGINS: str = "http://localhost:3000"

    class Config:
        env_file = ".env"

settings = Settings()
```

### 6. Create Storage Bucket

In Supabase Dashboard → Storage:

1. Create bucket: `uploads`
2. Set to **private** (we'll use signed URLs)
3. Add policy for authenticated users:

```sql
-- Allow authenticated users to upload
CREATE POLICY "Users can upload files"
ON storage.objects FOR INSERT
TO authenticated
WITH CHECK (bucket_id = 'uploads');

-- Allow users to read their own files
CREATE POLICY "Users can read own files"
ON storage.objects FOR SELECT
TO authenticated
USING (bucket_id = 'uploads' AND auth.uid()::text = (storage.foldername(name))[1]);
```

### 7. Create Auth Middleware (Frontend)

```typescript
// frontend/src/middleware.ts
import { createServerClient, type CookieOptions } from '@supabase/ssr'
import { NextResponse, type NextRequest } from 'next/server'

export async function middleware(request: NextRequest) {
  let response = NextResponse.next({
    request: {
      headers: request.headers,
    },
  })

  const supabase = createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll() {
          return request.cookies.getAll()
        },
        setAll(cookiesToSet) {
          cookiesToSet.forEach(({ name, value, options }) => {
            request.cookies.set(name, value)
            response.cookies.set(name, value, options)
          })
        },
      },
    }
  )

  const { data: { user } } = await supabase.auth.getUser()

  // Protected routes
  if (!user && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url))
  }

  // Redirect logged-in users away from auth pages
  if (user && (request.nextUrl.pathname === '/login' || request.nextUrl.pathname === '/signup')) {
    return NextResponse.redirect(new URL('/dashboard', request.url))
  }

  return response
}

export const config = {
  matcher: ['/((?!_next/static|_next/image|favicon.ico|api).*)'],
}
```

---

## Verification Checklist

- [ ] Supabase project created and accessible
- [ ] Environment variables set in both frontend and backend
- [ ] `createClient()` works without errors
- [ ] Can see Supabase connection in browser console (no errors)
- [ ] Storage bucket created

---

## Files Created

```
frontend/
├── .env.local                    # Supabase credentials
├── src/
│   ├── lib/
│   │   └── supabase/
│   │       ├── client.ts         # Browser client
│   │       └── server.ts         # Server client
│   └── middleware.ts             # Auth middleware

backend/
├── .env                          # All backend secrets
└── app/
    └── core/
        ├── config.py             # Settings from env
        └── supabase.py           # Supabase client
```

---

## Next Step

→ `plans/phase-1-foundation/03-schema-design.md`
