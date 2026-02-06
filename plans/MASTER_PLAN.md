# FoliarScript Rebuild - Master Plan

> **Owner**: Sriram (Product/Business) + Claude (Engineering)
> **Target**: Production-ready MVP
> **Approach**: Ship fast, iterate faster

---

## Philosophy

```
Build the simplest thing that works → Ship → Learn → Iterate
```

**Our competitive advantage**: We're rebuilding a working product with known requirements. No discovery phase. No ambiguity. Pure execution.

---

## Phase Overview

| Phase | Name | Duration | Focus |
|-------|------|----------|-------|
| 1 | Foundation | Week 1 | Auth, DB schema, project setup |
| 2 | Core | Week 2-3 | Scripts, recommendations, payments |
| 3 | Features | Week 4 | PDF/CSV upload, field history |
| 4 | Polish | Week 5 | Mobile UX, testing, deployment |

---

## Phase 1: Foundation

**Goal**: Get a user from signup to empty dashboard

```
📁 plans/phase-1-foundation/
├── 01-project-setup.md      # Next.js + FastAPI scaffolding
├── 02-supabase-setup.md     # Auth, DB, storage config
├── 03-schema-design.md      # Database tables + migrations
└── 04-auth-flow.md          # Signup, login, session handling
```

### Deliverables
- [ ] Next.js 15 app with Tailwind + shadcn/ui
- [ ] FastAPI backend with proper structure
- [ ] Supabase project with auth configured
- [ ] Database schema migrated
- [ ] User can signup, login, see empty dashboard

### When Done
User signs up → verifies email → lands on dashboard saying "No scripts yet"

---

## Phase 2: Core

**Goal**: Complete script lifecycle (create → pay → view recommendation)

```
📁 plans/phase-2-core/
├── 01-script-wizard.md      # Multi-step form implementation
├── 02-recommendation-engine.md  # Port calculation logic
├── 03-stripe-integration.md     # Payment flow
├── 04-recommendation-page.md    # Interactive display + sliders
└── 05-api-endpoints.md          # All CRUD operations
```

### Deliverables
- [ ] Script creation wizard (4 steps)
- [ ] Recommendation calculation engine (Python)
- [ ] Stripe checkout integration
- [ ] Recommendation page with sliders
- [ ] All API endpoints working

### When Done
User creates script → pays → sees interactive recommendation with sliders

---

## Phase 3: Features

**Goal**: Input flexibility + history tracking

```
📁 plans/phase-3-features/
├── 01-pdf-upload.md         # PDF parsing integration
├── 02-csv-mapping.md        # Flexible CSV import
├── 03-field-history.md      # Timeline + charts
├── 04-pdf-export.md         # Download recommendations
└── 05-client-management.md  # Add/manage clients
```

### Deliverables
- [ ] PDF upload with parsing + correction UI
- [ ] CSV upload with column mapping
- [ ] Field history with line charts
- [ ] PDF export of recommendations
- [ ] Client management (add, list, select)

### When Done
User can upload PDF/CSV → correct values → see field trends over time

---

## Phase 4: Polish

**Goal**: Production-ready, mobile-optimized

```
📁 plans/phase-4-polish/
├── 01-mobile-optimization.md    # Responsive testing
├── 02-error-handling.md         # Edge cases + UX
├── 03-testing-strategy.md       # Critical path tests
├── 04-deployment.md             # Vercel + AWS setup
└── 05-monitoring.md             # Logging + alerts
```

### Deliverables
- [ ] Mobile/tablet responsive across all pages
- [ ] Error states and loading states
- [ ] E2E tests for critical paths
- [ ] Production deployment
- [ ] Basic monitoring

### When Done
Ship it. Real users. Real feedback.

---

## Development Workflow

### Daily Rhythm

```
Morning:
  1. Review tasks/todo.md
  2. Pick highest-impact item
  3. Build it

Afternoon:
  1. Test what you built
  2. Commit + push
  3. Update todo.md

End of day:
  1. Demo to yourself (screen record if significant)
  2. Update tasks/lessons.md if you learned something
```

### When to Use What

| Scenario | Tool | Why |
|----------|------|-----|
| Writing code, debugging | **Claude Code** | Full context, file access, can run tests |
| Planning architecture | **Claude Code (Plan mode)** | Can explore codebase while planning |
| Quick questions | **Claude.ai** | Faster for conceptual questions |
| Research (docs, APIs) | **Claude Code + WebSearch** | Can fetch and apply immediately |
| Reviewing PRs | **Claude Code** | Can read files, understand context |
| Pair programming | **Claude Code** | Real-time iteration |

### Commit Strategy

```bash
# Feature work
feat: add script creation wizard step 1
feat: implement recommendation calculation engine

# Fixes
fix: slider tooltip positioning on mobile

# Chores
chore: add missing dependencies
chore: update schema migration

# Always include session link
https://claude.ai/code/session_XXXXX
```

---

## Speed Multipliers

### 1. Patterns Library
Pre-built patterns Claude Code can reference:

```
📁 patterns/
├── api-endpoint.md      # FastAPI endpoint pattern
├── react-form.md        # Form with validation pattern
├── supabase-query.md    # Database query pattern
└── component.md         # React component pattern
```

### 2. Templates
Copy-paste starting points:

```
📁 templates/
├── api-route.py         # FastAPI route template
├── page.tsx             # Next.js page template
├── component.tsx        # React component template
└── test.py              # Pytest test template
```

### 3. Context Files
Every phase has context files that give Claude Code everything it needs:

- **What** we're building (requirements)
- **How** to build it (patterns)
- **Where** files go (structure)
- **Why** decisions were made (rationale)

---

## Skills to Build (Future)

Custom Claude Code skills that could 10x speed:

| Skill | Trigger | What It Does |
|-------|---------|--------------|
| `/new-endpoint` | Creating API | Scaffolds FastAPI endpoint + tests |
| `/new-page` | Creating page | Scaffolds Next.js page + components |
| `/new-component` | Creating component | Scaffolds React component + stories |
| `/migrate` | Schema change | Creates migration + updates types |
| `/test` | After feature | Runs tests, reports failures |
| `/deploy-preview` | Before merge | Deploys to preview URL |

---

## Automation Ideas

### GitHub Actions

```yaml
# On every push:
- Lint (ESLint + Ruff)
- Type check (TypeScript + mypy)
- Test (Jest + Pytest)
- Preview deploy (Vercel)

# On merge to main:
- Production deploy
- Notify Slack/Discord
```

### Pre-commit Hooks

```bash
# Before every commit:
- Format code (Prettier + Black)
- Lint
- Type check
- Run affected tests
```

### Database Sync

```bash
# On schema change:
- Generate migration
- Update TypeScript types (from Supabase)
- Update Pydantic models
```

---

## Risk Mitigation

| Risk | Mitigation |
|------|------------|
| Stripe integration complexity | Use Stripe Checkout (hosted), not custom |
| PDF parsing accuracy | Human-in-the-loop correction UI |
| Mobile UX | Test on real devices early (Phase 1) |
| Data migration | Handle separately, after MVP stable |
| Performance at scale | Not a concern at 100 users, optimize later |

---

## Success Metrics

### MVP Launch Criteria

- [ ] User can complete full flow (signup → pay → view recommendation)
- [ ] Works on mobile Safari + Chrome
- [ ] PDF export generates correctly
- [ ] No critical bugs
- [ ] Under 3s page load

### Post-Launch (Week 1)

- [ ] 5 real users complete a script
- [ ] Collect feedback on UX pain points
- [ ] Zero payment failures
- [ ] Uptime > 99%

---

## Quick Reference

### Tech Stack

| Layer | Choice | Docs |
|-------|--------|------|
| Frontend | Next.js 15 | nextjs.org/docs |
| UI | shadcn/ui | ui.shadcn.com |
| Backend | FastAPI | fastapi.tiangolo.com |
| Database | Supabase (Postgres) | supabase.com/docs |
| Auth | Supabase Auth | supabase.com/docs/auth |
| Payments | Stripe | stripe.com/docs |
| Storage | AWS S3 | docs.aws.amazon.com/s3 |

### Key Files

| Purpose | Location |
|---------|----------|
| Project instructions | `CLAUDE.md` |
| Product requirements | `docs/PRODUCT_SPEC.md` |
| Current tasks | `tasks/todo.md` |
| Lessons learned | `tasks/lessons.md` |
| Phase plans | `plans/phase-X-*/` |
| Design system | `docs/reference/08-design-system.md` |
| Recommendation logic | `docs/reference/03-recommendation-engine.md` |
| Nutrient model | `docs/reference/04-nutrient-model-data.md` |

---

## Next Action

**Start with Phase 1, Task 1**: Project Setup

```bash
# Read the context
cat plans/phase-1-foundation/01-project-setup.md

# Then execute
```

Let's build this thing.
