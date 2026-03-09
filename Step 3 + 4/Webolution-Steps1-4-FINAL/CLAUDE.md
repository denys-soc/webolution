# WEBOLUTION – Project Instructions for Claude Code

## What is this project?

Webolution is a fully automated pipeline that finds businesses with bad websites, analyzes their content, generates an improved website preview, and sells it to them via personalized outreach. Built by SocialCooks.

## Architecture Overview

- **Event-driven microservices** communicating via Redis Streams (BullMQ)
- **17-step pipeline with gates**: Scrape+Check+Score (Step 1) → **Gate 1** (operator review) → Intelligence+Routing (Step 2) → Enrich+Crawl **PARALLEL** (Steps 3+4) → Review+Competitor (Steps 5+6 parallel) → Score (Step 7) → Gate 3 → Plan → Polish → Gate 3.5 → Build → QA → Gate 4 → Deploy → Outreach Prep → Gate 5 → Email → Payment → Onboard → Post-Launch
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
│   │       ├── types/           # Lead, Step1 types, Step2 types, events, industry profiles
│   │       ├── constants.ts     # BROWSER_USER_AGENT, AGGREGATOR_DOMAINS, PARKING_PATTERNS, etc.
│   │       ├── url.ts           # normalizeDomain()
│   │       ├── detection.ts     # detectBookingWidget(), extractCopyrightYear(), etc.
│   │       ├── phone.ts         # normalizePhone()
│   │       ├── business-name.ts # normalizeBusinessName(), extractLegalForm()
│   │       ├── events.ts        # emitEvent(), event type definitions
│   │       ├── industry-profile.ts  # loadIndustryProfile(), IndustryProfile type
│   │       ├── r2.ts            # uploadToR2(), deleteFromR2()
│   │       ├── cost-tracking.ts # logApiCost()
│   │       └── logger.ts        # pino logger setup
│   └── queue/                   # BullMQ queue definitions + worker base class
│       └── src/
│           ├── queues.ts        # All queue definitions
│           └── base-worker.ts   # Abstract PipelineWorker class
├── services/
│   ├── scraper/                 # Step 1: Find Businesses With Bad Websites (Scrape+Check+Score)
│   ├── intelligence/            # Step 2: Lead Intelligence & Pipeline Routing
│   ├── enricher/                # Step 3: Lead Enrichment (runs PARALLEL with Step 4)
│   ├── crawler/                 # Step 4: Website Crawl & Content Extraction (runs PARALLEL with Step 3)
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
│   ├── STEP-1-FINAL.md          # ★ THE Step 1 spec (replaces Step 1 in STEPS-1-4.md)
│   ├── STEP-2-FINAL.md          # ★ THE Step 2 spec (replaces Step 2 in STEPS-1-4.md)
│   ├── STEPS-3-4-FINAL.md       # ★ THE Steps 3+4 spec (replaces Steps 3+4 in STEPS-1-4.md)
│   ├── SPEC.md                  # Condensed full pipeline spec (overview only)
│   ├── DATABASE.md              # Base database schema (extended by STEP-1 and STEP-2)
│   ├── EVENTS.md                # Event catalog (base, extended by each step)
│   ├── INDUSTRY-PROFILES.md     # Industry profile details
│   ├── STEPS-1-4.md             # OLD specs (Steps 1-2 superseded by FINAL files, Steps 3-4 still valid)
│   ├── STEPS-5-8.md             # Full specs Steps 5-8
│   ├── STEPS-9-12.md            # Full specs Steps 9-12
│   ├── STEPS-13-17.md           # Full specs Steps 13-17
│   ├── INDUSTRY-RANKING-FULL.md # Full industry analysis
│   ├── INDUSTRY-WEBSITE-PLAYBOOK.md # ★ 22 industries, weaknesses, detection, arguments
│   ├── TECH-ARCHITECTURE.md     # Full technical architecture
│   ├── ADMIN-BACKEND.md         # Admin backend spec (10 modules)
│   ├── QA-FINDINGS.md           # Critical QA findings
│   ├── WEBSITE-BLUEPRINTS.md    # What "perfect" looks like per industry
│   ├── WEBSITE-QUALITY-DETECTION.md # OLD (superseded by STEP-1-FINAL.md)
│   └── SETUP-GUIDE.md           # Setup instructions
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
- Model: claude-sonnet-4-20250514 for all AI tasks (content extraction, quality review, content polish, rebuild planning)
- Model: claude-haiku-4-5-20251001 for lightweight AI tasks (Step 1 screenshot visual analysis — cheaper, faster)
- Use structured output (JSON mode) for all extractions
- Always include industry-specific context from profile in system prompt
- Track token usage and cost per call via logApiCost()
- Temperature: 0 for extraction/fact-check, 0.3 for content generation, 0.7 for creative polish

### Frontend (Dashboards)
- Next.js App Router
- Tailwind CSS for styling
- shadcn/ui for components
- tRPC for type-safe API calls between frontend and backend

## Key Reference Documents

### Quick Reference (condensed, for orientation)
- `docs/SPEC.md` – Condensed 17-step pipeline (1 page per step) — overview only, NOT for implementation
- `docs/DATABASE.md` – Base Prisma schema (extended by Step 1 and Step 2 final specs)
- `docs/EVENTS.md` – Event catalog with BullMQ queue definitions (base, extended by each step)
- `docs/INDUSTRY-PROFILES.md` – Coaches + Friseure JSON configs

### ★ DEFINITIVE Step Specs (read THESE for implementation)
- **`docs/STEP-1-FINAL.md`** – Step 1: "Find Businesses With Bad Websites" — 10,600 words, complete, self-contained, replaces old Step 1
- **`docs/STEP-2-FINAL.md`** – Step 2: "Lead Intelligence & Pipeline Routing" — 15,400 words, complete, self-contained, replaces old Step 2. Includes all TypeScript interfaces, shared infrastructure code, recommended package structure
- **`docs/STEPS-3-4-FINAL.md`** – Steps 3+4: "Gap-Fill Enrichment & Full Website Crawl" — 11,600 words, complete, self-contained, replaces old Steps 3+4. Steps run IN PARALLEL. Includes: competitor finding from own DB (free!), enrichment plan execution, Playwright crawling with CMS-specific strategies (WordPress/Jimdo/Wix/Squarespace), AI content extraction with industry-specific prompts, brand asset extraction, SEO analysis, responsive design testing, content completeness scoring, post-crawl merge orchestrator

### Full Specifications (complete, for implementation details)
Read the relevant full spec BEFORE implementing any step:
- `docs/STEPS-1-4.md` – ⚠️ **FULLY SUPERSEDED** — Steps 1-2 replaced by STEP-1-FINAL.md + STEP-2-FINAL.md. Steps 3-4 replaced by STEPS-3-4-FINAL.md. This file is kept for historical reference only.
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
6. `services/scraper` – Step 1: Google Places API (New) Text Search, HTTP Quick-Check, Lighthouse, Quick-Score, AI Vision for gray-zone leads. See `docs/STEP-1-FINAL.md`
7. `services/intelligence` – Step 2: Suppression checks, Freshness check, Impressum scrape, data normalization, email validation+guessing, intelligence calculation, enrichment plan+pipeline routing. Phase 1 (per-lead) + Phase 2 (batch). See `docs/STEP-2-FINAL.md`
8. `services/enricher` – Step 3: Follow enrichment plan from Step 2. LinkedIn, portal profiles, email finding. Runs PARALLEL with Step 4
9. `services/crawler` – Step 4: Playwright crawling + Claude extraction. Uses crawlStrategy and priorityPages from Step 2. Runs PARALLEL with Step 3
10. `apps/admin` – **Admin Backend Sprint 2**: Gate 1 approval screen (with bulk-approve, reject reasons, manual overrides, Quick Intel mode), Pipeline progress view, Step 2 results tab in Lead Detail

### Phase 3: Analysis (Week 5-6)
10. `services/crawler` – Step 4: Playwright full-website crawling with CMS-specific strategies, AI content extraction (Sonnet), brand asset extraction, SEO analysis, responsive testing. Runs PARALLEL with Step 3. See `docs/STEPS-3-4-FINAL.md`
11. `services/reviewer` – Step 5: Lighthouse + Claude Vision quality review
12. `services/competitor` – Step 6: Competitor analysis (uses leads from OWN DB + screenshots). Can start EARLY after Step 3 finds competitor IDs
13. `services/scorer` – Step 7: Score calculation with industry weights

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

# Google (single key for Places API New + PageSpeed Insights)
GOOGLE_API_KEY=...

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
R2_ENDPOINT=https://<account-id>.r2.cloudflarestorage.com
R2_ACCESS_KEY_ID=...
R2_SECRET_ACCESS_KEY=...
R2_BUCKET=webolution
R2_PUBLIC_URL=https://assets.webolution.de

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

## 🚀 Step 2 Final Spec (THE definitive Step 2 document)

**`docs/STEP-2-FINAL.md`** replaces the Step 2 section in STEPS-1-4.md completely.

This is a complete, self-contained spec for Step 2: "Lead Intelligence & Pipeline Routing."
Covers: 2-phase architecture (Phase 1: per-lead with 7 sub-steps, Phase 2: batch per campaign), suppression checks (cooling period, bounce, spam, competitor, non-profit, review crisis), website freshness checks, Impressum scraping with 6-level fallback chain, data normalization (phone E.164, firm name, legal form), email validation + SMTP verification + email guessing, intelligence calculation (data consistency, decision maker confidence, content readiness, personalization level, ROI), pipeline routing (5 route types: FULL, ANALYSIS_ONLY, PHONE_ONLY, NURTURE, FAST_TRACK), enrichment plan as waterfall, outreach channel determination, batch operations (geo-clustering, template diversification, outreach wave assignment, campaign insights, cross-campaign dedup), state machine with valid transitions, auto-approve rules, all TypeScript interfaces, shared infrastructure functions (emitEvent, loadIndustryProfile, R2, cost tracking, logging), and recommended package structure.

**Step 2 dispatches to Steps 3 + 4 in PARALLEL (not sequential like the old spec).**

**Read this INSTEAD of the old Step 2 section in STEPS-1-4.md.**

## 🚀 Steps 3+4 Final Spec (THE definitive Steps 3+4 document)

**`docs/STEPS-3-4-FINAL.md`** replaces the Steps 3+4 sections in STEPS-1-4.md completely.

This is a complete, self-contained spec for Steps 3+4 running IN PARALLEL:
- **Step 3 "Gap-Fill Enrichment"** (3-15 sec, €0.00-0.07): Executes the enrichment plan from Step 2. Finds competitors from our OWN database for FREE (other leads in same city+industry). LinkedIn via Google Search (MVP). Portal existence checks (Treatwell, Booksy). Northdata only for GmbH/UG. Email ranking. Outreach channel updates. Can DISQUALIFY leads (insolvent companies) and CANCEL running Step 4 crawls.
- **Step 4 "Full Website Crawl & Content Extraction"** (30-300 sec, €0.05-0.15): Playwright crawling with CMS-specific strategies. Priority page crawling with early exit. Full-page screenshots. Image download + classification. Brand asset extraction (logo, colors, fonts, design direction). AI content extraction with industry-specific Sonnet prompts. Boilerplate removal (Mozilla Readability). SEO analysis. Responsive design testing. Content completeness scoring.
- **Orchestrator Merge** (after both complete): Merges content brief with REAL data. Updates pipeline route if content too thin (FULL→NURTURE). Dispatches to Steps 5+6.

Includes: 4 Prisma models (ContentPackage, ContentPage, ContentImage + Lead extensions), CMS strategies for WordPress/Jimdo/Wix/Squarespace, full AI extraction prompts for Coaches+Friseure, complete example walkthrough (Sandra K.), all helper functions (image classification, boilerplate removal, SEO analysis, responsive test, brand extraction).

**Read this INSTEAD of the old Steps 3+4 sections in STEPS-1-4.md.**
