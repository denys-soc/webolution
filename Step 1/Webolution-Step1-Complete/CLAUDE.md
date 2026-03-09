# WEBOLUTION – Project Instructions for Claude Code

## What is this project?

Webolution is a fully automated pipeline that finds businesses with bad websites, analyzes their content, generates an improved website preview, and sells it to them via personalized outreach. Built by SocialCooks.

## Architecture Overview

- **Event-driven microservices** communicating via Redis Streams (BullMQ)
- **17-step pipeline**: Scrape → Validate → Enrich → Crawl → Review → Competitor → Score → Plan → Polish → Build → QA → Deploy → Outreach Prep → Email → Payment → Onboard → Post-Launch
- **Parallel industry pipelines**: Each industry (coaches, friseure, etc.) runs its own pipeline with industry-specific configs (JSON profiles)
- **Admin Backend (Operations Center)**: Central control panel with 10 modules — Pipeline Dashboard, Lead Explorer, Lead Detail View, Manual Pipeline Controls, QA & Review Center, Website Preview & Compare, Outreach Monitor, Revenue & Conversion, Configuration Center, System Health & Costs
- **Monorepo** with Turborepo

## Tech Stack

- **Language**: TypeScript everywhere (Node.js for services, Next.js for dashboards)
- **Pipeline Workers**: Node.js + BullMQ (Redis-based job queues)
- **Web Crawling**: Playwright (headless Chromium)
- **AI**: Anthropic Claude API (claude-sonnet-4-20250514) for content extraction, quality review, content polishing, rebuild planning
- **Database**: PostgreSQL via Prisma ORM
- **Cache / Event Bus**: Redis (Upstash) with BullMQ for job queues
- **Object Storage**: Cloudflare R2 (S3-compatible)
- **Preview Hosting**: Vercel (wildcard domain *.webolution.de)
- **Website Generation**: Next.js + Tailwind CSS templates
- **Email Outreach**: Instantly.ai API
- **Payment**: Stripe (Checkout + Customer Portal)
- **Monitoring**: Sentry (errors) + Prometheus/Grafana (metrics)
- **Admin Backend**: Next.js App Router + tRPC + shadcn/ui + TanStack Table + Recharts + SSE for real-time
- **CI/CD**: GitHub Actions

## Project Structure

```
webolution/
├── CLAUDE.md                    # This file
├── turbo.json                   # Turborepo config
├── package.json                 # Root package.json (workspaces)
├── packages/
│   ├── db/                      # Prisma schema + client + migrations
│   │   ├── prisma/
│   │   │   └── schema.prisma    # Database schema (see docs/DATABASE.md)
│   │   └── src/
│   │       └── index.ts         # Prisma client export
│   ├── shared/                  # Shared types, utils, constants
│   │   └── src/
│   │       ├── types/           # Lead, ContentPackage, QualityReport, etc.
│   │       ├── events/          # Event type definitions + emitters
│   │       ├── industry/        # Industry profile loader + types
│   │       └── utils/           # Logger, cost-tracker, confidence-scorer
│   └── queue/                   # BullMQ queue definitions + worker base class
│       └── src/
│           ├── queues.ts        # All queue definitions
│           └── base-worker.ts   # Abstract PipelineWorker class
├── services/
│   ├── scraper/                 # Step 1: Lead Scraping
│   ├── validator/               # Step 2: Data Validation & Cleaning
│   ├── enricher/                # Step 3: Lead Enrichment
│   ├── crawler/                 # Step 4: Website Crawl & Content Extraction
│   ├── reviewer/                # Step 5: Website Quality Review
│   ├── competitor/              # Step 6: Competitor Benchmarking
│   ├── scorer/                  # Step 7: Lead Scoring
│   ├── planner/                 # Step 8: Rebuild Planning
│   ├── polisher/                # Step 9: Content Polish & Fact Verification
│   ├── builder/                 # Step 10: Website Assembly & Build
│   ├── qa/                      # Step 11: QA & Testing
│   ├── deployer/                # Step 12: Preview Deployment
│   ├── outreach-prep/           # Step 13: Outreach Preparation
│   ├── outreach-sender/         # Step 14: Email Outreach
│   ├── payment/                 # Step 15: Conversion & Payment
│   ├── onboarding/              # Step 16: Managed Onboarding
│   └── customer-success/        # Step 17: Post-Launch Monitoring
├── apps/
│   ├── admin/                   # Admin Dashboard (Next.js)
│   ├── customer/                # Customer Dashboard (Next.js)
│   ├── pricing/                 # Personalized Pricing Pages (Next.js)
│   └── preview/                 # Preview Website Templates (Next.js)
├── templates/                   # Industry-specific website templates
│   ├── coach-landing/
│   ├── coach-authority/
│   ├── salon-gallery/
│   └── salon-barber/
├── profiles/                    # Industry Profile JSON configs
│   ├── coaches.json
│   └── friseure.json
├── docs/                        # Specification documents
│   ├── SPEC.md                  # Condensed full pipeline spec
│   ├── DATABASE.md              # Complete database schema
│   ├── EVENTS.md                # Event catalog
│   ├── INDUSTRY-PROFILES.md     # Industry profile details
│   └── docx/                    # Original .docx spec files (reference only)
└── scripts/                     # Dev/ops scripts
    ├── seed.ts                  # Seed DB with test data
    └── test-pipeline.ts         # Run 1 lead through entire pipeline
```

## Coding Conventions

### General
- TypeScript strict mode everywhere
- ESM modules (type: "module" in package.json)
- Use `pnpm` as package manager
- Prettier for formatting (no semicolons, single quotes, 2-space indent)
- All environment variables in `.env` files, loaded via `dotenv`
- Never commit secrets. Use `.env.example` with placeholder values

### Services (Pipeline Workers)
Every service follows the same pattern:

```typescript
// services/{name}/src/worker.ts
import { BaseWorker } from '@webolution/queue'
import { db } from '@webolution/db'
import { EventType } from '@webolution/shared'

export class MyWorker extends BaseWorker {
  readonly inputEvent = EventType.LEAD_ENRICHED
  readonly outputEvent = EventType.CRAWL_COMPLETED

  async process(job: Job<LeadEnrichedPayload>): Promise<CrawlResult> {
    const lead = await db.lead.findUnique({ where: { id: job.data.leadId } })
    const profile = await this.loadIndustryProfile(lead.industryTag)

    // ... processing logic ...

    return {
      leadId: lead.id,
      confidence: calculatedConfidence,
      // ... result data
    }
  }
}
```

### Key Patterns
- **Idempotency**: Every worker checks if lead was already processed before starting
- **Confidence scoring**: Every output includes a confidence score (0-1). Below threshold → manual review queue
- **Cost tracking**: Every API call is logged with cost in cents to the audit_log table
- **Error handling**: RetryableError (auto-retry with backoff), PermanentError (mark lead as failed), ConfidenceError (route to manual review)
- **Industry profiles**: All industry-specific logic comes from JSON config, never hardcoded

### Database
- Prisma ORM for all DB operations
- JSONB columns for semi-structured data (firm_profile, brand_assets, quality_reports, etc.)
- UUID primary keys
- Always include created_at/updated_at timestamps
- See docs/DATABASE.md for complete schema

### AI / Claude API Usage
- Model: claude-sonnet-4-20250514 for all AI tasks (fast + good enough for extraction/review)
- Use structured output (JSON mode) for all extractions
- Always include industry-specific context from profile in system prompt
- Track token usage and cost per call
- Temperature: 0 for extraction/fact-check, 0.3 for content generation, 0.7 for creative polish

### Frontend (Dashboards)
- Next.js App Router
- Tailwind CSS for styling
- shadcn/ui for components
- tRPC for type-safe API calls between frontend and backend

## Key Reference Documents

### Quick Reference (condensed, for orientation)
- `docs/SPEC.md` – Condensed 17-step pipeline (1 page per step)
- `docs/DATABASE.md` – Full Prisma schema (copy-paste ready)
- `docs/EVENTS.md` – Event catalog with BullMQ queue definitions
- `docs/INDUSTRY-PROFILES.md` – Coaches + Friseure JSON configs

### Full Specifications (complete, for implementation details)
Read the relevant full spec BEFORE implementing any step:
- `docs/STEPS-1-4.md` – **Full spec: Scraping, Validation, Enrichment, Crawl & Extraction** — All sub-operations, error handling tables, confidence scoring formulas, industry-specific parameters, KPIs
- `docs/STEPS-5-8.md` – **Full spec: Quality Review, Competitor Benchmarking, Lead Scoring, Rebuild Planning** — Calibration sets, visual scoring prompts, weakness classification, scoring formulas, template decision trees, legal checklists
- `docs/STEPS-9-12.md` – **Full spec: Content Polish, Website Build, QA & Testing, Preview Deployment** — Banned phrase lists, tone-of-voice matrix, fact-verification workflow, performance budgets, QA test suite (40+ tests), preview sales layer, behavior tracking
- `docs/STEPS-13-17.md` – **Full spec: Outreach Prep, Email Outreach, Payment, Onboarding, Post-Launch** — Email validation, hyper-personalization variables, 5-touch sequence with behavioral branching, objection handling, DNS switch provider matrix, rollback plan, upsell triggers
- `docs/INDUSTRY-RANKING-FULL.md` – **Full industry analysis** — 10-dimension scoring for all 6 industries, complete pilot profiles for Coaches + Friseure
- `docs/TECH-ARCHITECTURE.md` – **Full technical architecture** — Service landscape, event bus, state machine, database schema, infrastructure costs, API cost model per step, unit economics, worker pattern, error categorization, scaling strategy, monitoring, security, CI/CD, MVP roadmap
- `docs/ADMIN-BACKEND.md` – **Full admin backend spec** — 10 modules (Pipeline Dashboard, Lead Explorer, Lead Detail with 7 tabs, Manual Pipeline Controls with per-step actions, QA & Review Center, Website Preview & Compare, Outreach Monitor, Revenue Dashboard, Configuration Center, System Health). Includes URL structure, tRPC router definitions, real-time events, DB extensions, build order

## Build Order (MVP)

Follow this order when building:

### Phase 1: Foundation (Week 1-2)
1. Monorepo setup (Turborepo + pnpm workspaces)
2. `packages/db` – Prisma schema + migrations
3. `packages/shared` – Types, event definitions, industry profile types
4. `packages/queue` – BullMQ setup + BaseWorker class
5. `apps/admin` – **Admin Backend Sprint 1**: Pipeline Dashboard (Modul 1) + Lead Explorer (Modul 2) + Lead Detail View basics (Modul 3). See `docs/ADMIN-BACKEND.md`

### Phase 2: Ingestion (Week 3-4)
6. `services/scraper` – Google Maps API scraping
7. `services/validator` – URL checks, dedup, blacklist
8. `services/enricher` – Google Places details, impressum parsing
9. `apps/admin` – **Admin Backend Sprint 2**: Manual Pipeline Controls (Modul 4) + full Lead Detail View tabs (Modul 3). Campaign launcher for batch scraping

### Phase 3: Analysis (Week 5-6)
10. `services/crawler` – Playwright crawling + Claude extraction
11. `services/reviewer` – Lighthouse + Claude Vision quality review
12. `services/competitor` – Quick competitor analysis
13. `services/scorer` – Score calculation with industry weights

### Phase 4: Generation (Week 7-8)
14. `services/planner` – Claude-based rebuild planning
15. `services/polisher` – Content polish + fact verification + banned phrases
16. `templates/` – First 4 industry templates (Next.js + Tailwind)
17. `services/builder` – Template engine + content injection
18. `services/qa` – Automated test suite
19. `apps/admin` – **Admin Backend Sprint 3**: QA & Review Center (Modul 5) + Website Preview & Compare (Modul 6). Rebuild plan approval workflow

### Phase 5: Delivery (Week 9-10)
20. `services/deployer` – Vercel preview deployment
21. `services/outreach-prep` – Email personalization + PDF generation
22. `services/outreach-sender` – Instantly.ai integration
23. `apps/admin` – **Admin Backend Sprint 4**: Outreach Monitor (Modul 7) + Configuration Center (Modul 9)

### Phase 6: Conversion (Week 11-12)
24. `apps/pricing` – Personalized pricing pages
25. `services/payment` – Stripe checkout + webhooks
26. `services/onboarding` – Onboarding form + manual process
27. `apps/customer` – Customer dashboard
28. `apps/admin` – **Admin Backend Sprint 5**: Revenue & Conversion (Modul 8) + System Health & Costs (Modul 10)

## Environment Variables

```env
# Database
DATABASE_URL=postgresql://...

# Redis
REDIS_URL=redis://...

# Anthropic
ANTHROPIC_API_KEY=sk-ant-...

# Google
GOOGLE_MAPS_API_KEY=...
GOOGLE_PLACES_API_KEY=...

# Vercel
VERCEL_API_TOKEN=...
VERCEL_TEAM_ID=...

# Stripe
STRIPE_SECRET_KEY=sk_...
STRIPE_WEBHOOK_SECRET=whsec_...

# Instantly.ai
INSTANTLY_API_KEY=...

# NanoBanana (Image Generation)
NANOBANANA_API_KEY=...

# NeverBounce (Email Validation)
NEVERBOUNCE_API_KEY=...

# Object Storage (Cloudflare R2)
R2_ACCOUNT_ID=...
R2_ACCESS_KEY_ID=...
R2_SECRET_ACCESS_KEY=...
R2_BUCKET_NAME=webolution-assets

# Monitoring
SENTRY_DSN=...
```

## Important Rules

1. **NEVER fabricate content**: All generated website text must be traceable to extracted data. No hallucinated facts. This is the #1 quality rule.
2. **Industry profile drives everything**: Every industry-specific decision comes from the JSON profile, not from hardcoded logic.
3. **Confidence-based routing**: Every output has a confidence score. Low confidence → manual review queue, never auto-proceed.
4. **Cost tracking**: Log every API call with cost to audit_log. Budget caps per lead.
5. **Idempotency**: Every service can be safely retried. Check for existing results before processing.
6. **MX records are sacred**: During onboarding, NEVER modify MX records. Only A/CNAME records for website pointing.
7. **German language**: All generated content, emails, and UI text for end customers is in German. Code, comments, and internal docs in English.

## ⚠️ CRITICAL: QA Findings (Read FIRST)

**Before implementing ANY step, read `docs/QA-FINDINGS.md`.**

This document contains 35 prioritized findings from comprehensive QA across all specs. Key blockers that change the architecture:

1. **CMS Layer required** – websites need a headless CMS, not just static builds (affects Step 10, 17, customer dashboard)
2. **Pipeline must STOP at gates** – step specs say "auto-trigger next step" but gates require "wait for approval." All step specs must be implemented with gate-awareness
3. **Gate 4 BEFORE deploy** – not after. Operator approves before website goes public
4. **Gate 3.5 (Content Approval)** – new gate between polish and build. Operator approves texts before they become a website
5. **Cost estimation BEFORE every costly step** – operator sees "this will cost ~€X" before approving
6. **No competitor names in outreach emails** – legal risk (UWG §6). Use anonymous: "Ein Mitbewerber in Ihrer Nähe"
7. **Content Ethics Policy** – explicit rules for what AI may/may not generate. Must be in every content-generating prompt
8. **Pricing configurable per industry** – not hardcoded
9. **Progressive gate automation** – MVP: all manual. Scale: configurable auto-approve thresholds

The QA findings also list required DB schema additions, event catalog additions, and industry profile additions. Cross-reference when building each component.

## 🎯 Website Blueprints (Read BEFORE designing templates)

**`docs/WEBSITE-BLUEPRINTS.md`** defines what a PERFECT website looks like for each pilot industry.

This is the foundation for:
- Template design (what to build)
- Content strategy (what to write)  
- QA criteria (what to check)
- Rebuild planning (what to aim for)
- Outreach arguments (what to highlight in the before/after)

Covers per industry: visitor personas, page-by-page structure, visual design direction (colors, typography, imagery, layout), conversion elements, what makes it go from "good" to "incredible", and cross-industry quality standards.

**Read this BEFORE building any template or writing any content-generation prompt.**

## 🔍 Website Quality Detection (Read BEFORE implementing Step 1-2)

**`docs/WEBSITE-QUALITY-DETECTION.md`** answers the most fundamental question: How do we know a website is "bad enough" to rebuild?

Defines: 5 weakness categories (Invisible, Repulsive, Passive, Trustless, Outdated), a Quick-Quality-Check as new Step 1.5 (€0.005/lead, <15 seconds), industry-specific weakness weights, the 10 strongest sales arguments derived from weaknesses, and how the quick-check changes the entire pipeline flow (60% cost savings by filtering before expensive steps).

**This fundamentally changes the pipeline**: Scrape → Validate → QUICK-QUALITY-CHECK → only bad websites enter the expensive pipeline. Good websites are archived immediately.

## 📖 Industry Website Playbook (THE foundation document)

**`docs/INDUSTRY-WEBSITE-PLAYBOOK.md`** is the MOST IMPORTANT reference document in this project.

It defines for EACH industry (sorted by priority): what makes a website GOOD, what makes it BAD (12 specific weaknesses per industry with detection methods and outreach arguments), the weakness-score formula, industry-specific legal risks, and the decision framework (Should we build for this lead?).

This document DRIVES:
- Quality Detection (Step 1.5): which weaknesses to check for
- Scoring (Step 7): weakness weights per industry  
- Rebuild Planning (Step 8): what "good" looks like as the target
- Content Polish (Step 9): what to write and what NOT to write
- QA (Step 11): what to test against
- Outreach (Steps 13-14): which arguments to use per detected weakness

**Read this BEFORE implementing ANY step.**

## 🚀 Step 1 Final Spec (THE definitive Step 1 document)

**`docs/STEP-1-FINAL.md`** replaces the Step 1 section in STEPS-1-4.md AND the WEBSITE-QUALITY-DETECTION.md document.

This is a complete, self-contained spec for Step 1: "Find Businesses With Bad Websites."
Covers: the 3-phase flow (Scrape → Check → Enrich+Score), Google Places API (New) with correct endpoints and pricing, Quick-Score calculation with industry-specific weights, 40+ technical detection checks (booking widgets, parking pages, SPAs, cookie banners, etc.), campaign management UX, error handling, duplikat detection, idempotency/resume, cost circuit breakers, calibration mode, screenshot retention, events, Track B, and parallelization strategy.

**Read this INSTEAD of the old Step 1 section in STEPS-1-4.md.**
