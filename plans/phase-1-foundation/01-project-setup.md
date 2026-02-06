# Phase 1.1: Project Setup

> **Goal**: Scaffolded Next.js + FastAPI projects with proper structure
> **Time**: ~2 hours
> **Dependencies**: None

---

## What We're Building

```
foliarscript-rewrite/
├── frontend/                 # Next.js 15 app
│   ├── src/
│   │   ├── app/             # App router pages
│   │   ├── components/      # UI components
│   │   │   └── ui/          # shadcn/ui components
│   │   ├── lib/             # Utilities
│   │   └── styles/          # Global CSS
│   ├── public/              # Static assets
│   └── package.json
│
├── backend/                  # FastAPI app
│   ├── app/
│   │   ├── api/             # Route handlers
│   │   │   └── v1/          # Versioned endpoints
│   │   ├── core/            # Config, security
│   │   ├── models/          # Pydantic models
│   │   ├── services/        # Business logic
│   │   └── main.py          # Entry point
│   ├── tests/
│   └── requirements.txt
│
├── docs/                     # Already exists
├── plans/                    # Already exists
└── CLAUDE.md
```

---

## Steps

### 1. Create Next.js App

```bash
cd /home/user/foliarscript-rewrite
npx create-next-app@latest frontend --typescript --tailwind --eslint --app --src-dir --import-alias "@/*"
```

**Options to select:**
- TypeScript: Yes
- ESLint: Yes
- Tailwind CSS: Yes
- `src/` directory: Yes
- App Router: Yes
- Import alias: `@/*`

### 2. Install UI Dependencies

```bash
cd frontend
npm install @radix-ui/react-dialog @radix-ui/react-select @radix-ui/react-tabs @radix-ui/react-checkbox @radix-ui/react-tooltip @radix-ui/react-dropdown-menu @radix-ui/react-separator @radix-ui/react-progress @radix-ui/react-slider @radix-ui/react-toast
npm install class-variance-authority clsx tailwind-merge tailwindcss-animate
npm install lucide-react
npm install @supabase/supabase-js @supabase/ssr
```

### 3. Setup shadcn/ui

```bash
npx shadcn@latest init
```

**Options:**
- Style: Default
- Base color: Slate
- CSS variables: Yes

Then add core components:

```bash
npx shadcn@latest add button card input label dialog select tabs toast
```

### 4. Configure Tailwind

Replace `tailwind.config.ts` with our design system config (see `docs/reference/08-design-system.md`).

### 5. Add Global CSS Variables

Update `src/styles/globals.css` with CSS variables from design system.

### 6. Create FastAPI Backend

```bash
cd /home/user/foliarscript-rewrite
mkdir -p backend/app/{api/v1,core,models,services}
mkdir -p backend/tests
touch backend/app/__init__.py
touch backend/app/main.py
touch backend/requirements.txt
```

### 7. Backend Requirements

```txt
# backend/requirements.txt
fastapi>=0.109.0
uvicorn[standard]>=0.27.0
pydantic>=2.5.0
pydantic-settings>=2.1.0
python-multipart>=0.0.6
httpx>=0.26.0
supabase>=2.3.0
stripe>=7.0.0
python-jose[cryptography]>=3.3.0
passlib[bcrypt]>=1.7.4
```

### 8. Create FastAPI Entry Point

```python
# backend/app/main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI(
    title="FoliarScript API",
    version="2.0.0",
    docs_url="/api/docs",
    redoc_url="/api/redoc",
)

# CORS for local development
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

@app.get("/api/health")
async def health_check():
    return {"status": "healthy", "version": "2.0.0"}
```

### 9. Add Font (Poppins)

Update `frontend/src/app/layout.tsx`:

```tsx
import { Poppins } from 'next/font/google'

const poppins = Poppins({
  subsets: ['latin'],
  weight: ['400', '500', '600', '700'],
  variable: '--font-poppins',
})

export default function RootLayout({ children }) {
  return (
    <html lang="en" className={poppins.variable}>
      <body className="font-sans bg-background text-foreground">
        {children}
      </body>
    </html>
  )
}
```

### 10. Create lib/utils.ts

```typescript
// frontend/src/lib/utils.ts
import { type ClassValue, clsx } from "clsx"
import { twMerge } from "tailwind-merge"

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```

---

## Verification Checklist

- [ ] `cd frontend && npm run dev` → App runs on localhost:3000
- [ ] `cd backend && uvicorn app.main:app --reload` → API runs on localhost:8000
- [ ] Visit http://localhost:8000/api/health → Returns JSON
- [ ] Visit http://localhost:8000/api/docs → Swagger UI works
- [ ] Tailwind styles apply (dark background visible)
- [ ] Poppins font loads

---

## Files Created

```
frontend/
├── src/
│   ├── app/
│   │   ├── layout.tsx      # Root layout with font
│   │   ├── page.tsx        # Home page (placeholder)
│   │   └── globals.css     # Design system variables
│   ├── components/
│   │   └── ui/             # shadcn components
│   └── lib/
│       └── utils.ts        # cn() utility
├── tailwind.config.ts      # Custom config
└── package.json

backend/
├── app/
│   ├── __init__.py
│   ├── main.py             # FastAPI entry
│   ├── api/v1/             # (empty for now)
│   ├── core/               # (empty for now)
│   ├── models/             # (empty for now)
│   └── services/           # (empty for now)
├── tests/
└── requirements.txt
```

---

## Next Step

→ `plans/phase-1-foundation/02-supabase-setup.md`
