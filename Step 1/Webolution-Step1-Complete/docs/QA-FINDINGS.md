# WEBOLUTION QA FINDINGS – Definitive List
# "Was würde im echten Betrieb scheitern?"

> This document is the result of 3 comprehensive QA passes across all 17 pipeline steps, admin backend, tech architecture, database schema, industry profiles, and business model. It identifies every weakness, missing piece, and required improvement.
>
> **For Claude Code**: Read this BEFORE and DURING implementation. Every finding tagged with a step/module reference tells you what to fix or add when building that component.

---

## SEVERITY LEVELS

- **🔴 BLOCKER**: The machine CANNOT function without this. Must be solved before MVP launch.
- **🟠 CRITICAL**: The machine functions but produces BAD RESULTS without this. Must be solved within first 2 weeks of operation.
- **🟡 IMPORTANT**: Significantly improves quality, conversion, or operability. Should be solved within first month.
- **🔵 IMPROVEMENT**: Makes things better but machine works without it. Solve when capacity allows.
- **⚪ FUTURE**: Not needed for MVP. Document for later phases.

---

## CATEGORY 1: STRUCTURAL ARCHITECTURE PROBLEMS

### 🔴 BLOCKER: No CMS Layer for Live Websites

**Affects**: Step 10, Step 17, Admin Backend, Customer Dashboard
**Problem**: Generated websites are static Next.js builds. After go-live, there is NO WAY to change content (text, images, prices, opening hours, team members) without a full code rebuild. At 50+ customers, this is untenable. Every "change opening hours" request requires a developer-level rebuild.
**Required Fix**: Integrate a Headless CMS (Sanity, Strapi, or Payload CMS – self-hostable) into the template engine. Website content is stored in the CMS, templates read from it at build time (SSG) or runtime (SSR/ISR). Content changes = CMS update + incremental rebuild (seconds, not minutes). The customer dashboard gets a simple content editor that writes to the CMS. Admin backend gets a full content editor per customer.
**Impact on specs**: Step 10 (Build) architecture changes fundamentally. Step 17 (Post-Launch) content update process changes. Customer Dashboard gets editing capability. Database needs cms_content table or CMS is external service. Tech Architecture needs CMS Service added.

### 🔴 BLOCKER: Pipeline Does NOT Actually Stop at Gates

**Affects**: All 17 Steps, Orchestrator, State Machine, Event Catalog
**Problem**: The pipeline specs (STEPS-1-4.md through STEPS-13-17.md) describe an AUTOMATIC pipeline where each step triggers the next via events. The Admin Backend (ADMIN-BACKEND.md) describes 5 Approval Gates where the pipeline stops and waits. These two documents CONTRADICT each other. The pipeline specs say "lead.enriched event triggers crawler automatically." The backend says "Gate 2: you decide which enriched leads to crawl." Both cannot be true.
**Required Fix**: ALL 17 step specs must be rewritten to include gate-awareness. After each gate-relevant step, the pipeline emits a "lead.awaiting_gate_X" event and the lead enters a "awaiting_approval" state. The pipeline does NOT continue until an "lead.gate_X_approved" event is received (from admin backend). The State Machine needs gate states. The Event Catalog needs approval events.
**New States**: awaiting_gate_1 (after validation), awaiting_gate_2 (after enrichment), awaiting_gate_3 (after scoring), awaiting_gate_3_5 (after content polish – NEW GATE), awaiting_gate_4 (after QA – BEFORE deploy, not after), awaiting_gate_5 (after outreach prep)
**New Events**: lead.gate1_approved, lead.gate1_rejected, lead.gate2_approved, etc.
**Gate config**: Each gate has a mode (MANUAL or AUTO with threshold). Stored in DB automation_rules table. Orchestrator checks mode before routing.

### 🔴 BLOCKER: Gate 4 is AFTER Deploy – Must Be BEFORE

**Affects**: Step 11, Step 12, Admin Backend Module 2
**Problem**: Current flow: Build → QA → Deploy → Gate 4 (you review). This means the website goes LIVE on webolution.de BEFORE you approve it. If it has errors, it's already public. Must be: Build → QA → Gate 4 (you review and approve) → THEN Deploy.
**Required Fix**: Reverse order of Step 12 (Deploy) and Gate 4. QA passes → lead enters awaiting_gate_4 → you review in admin backend (full preview via local/staging URL, NOT public) → you approve → THEN deploy to public webolution.de subdomain.
**Implication**: Need a way to preview the website WITHOUT deploying publicly. Options: localhost preview in admin iframe (complex), or Vercel Preview Deployments with password protection (simpler – Vercel supports this natively).

### 🔴 BLOCKER: No Content Approval Gate (Gate 3.5)

**Affects**: Step 9, Step 10, Admin Backend
**Problem**: Polished content goes directly into the build without your approval. You see the content only AFTER the website is built (in Gate 4). But by then, rebuilding means re-running Steps 9+10 (expensive, slow). You should see and approve/edit the CONTENT before it becomes a website.
**Required Fix**: Add Gate 3.5 between Step 9 (Content Polish) and Step 10 (Build). Pipeline stops after polishing. You see: all section texts with source references, all images with labels (own/stock/nanobana/placeholder), meta tags, CTA texts. You can: approve all → build starts, edit specific texts inline → save → build starts with edited content, reject → re-polish with notes.
**New State**: awaiting_content_approval (between polished and building)
**New Event**: lead.content_approved

### 🔴 BLOCKER: No Cost Estimation Before Costly Steps

**Affects**: All Steps with API costs (3, 4, 5, 6, 8, 9, 13), Admin Backend (all gates), Orchestrator
**Problem**: Costs are tracked AFTER they occur (audit_log.cost_cents). But the operator never sees costs BEFORE approving. At Gate 2, approving 30 leads for analysis should show: "Estimated cost: €7.80 (30 × €0.26 avg for crawl+review+competitor)." At Gate 3, approving 15 leads for rebuild should show: "Estimated cost: €3.90." This is essential for budget control.
**Required Fix**: 
1. CostEstimator service that calculates expected cost per lead per step based on historical averages (from audit_log). Falls back to configured defaults when no history.
2. Every Gate Approval Screen shows: "Estimated cost for approving X leads: €Y" with breakdown per step.
3. Campaign creation shows total estimated budget: "200 leads × avg €0.80/lead through full pipeline = ~€160"
4. Per-lead cost display in Lead Detail: "This lead has cost €1.23 so far. Next step estimated: €0.12"
**New service**: CostEstimator (reads from audit_log aggregates, provides estimates via tRPC)
**New DB fields**: leads.cumulative_cost (running total), campaigns.budget, campaigns.budget_spent, campaigns.budget_remaining

### 🔴 BLOCKER: No Margin/Pricing Configuration

**Affects**: Step 15 (Payment), Admin Backend (Config Center), Business Model
**Problem**: Pricing is hardcoded in the spec (Starter €990, Professional €1.990, Premium €3.490). But: Different industries may need different prices. You need to control margins. There's no way to: set prices per industry, define minimum margin targets, see profitability per lead before deciding to rebuild, offer custom discounts, or ensure pricing covers costs.
**Required Fix**:
1. pricing_config table: packages with prices per industry, customizable in Config Center
2. margin_target per industry and per campaign (e.g., "min 85% margin for coaches")
3. Break-even display at Gate 3: "This lead cost €1.50 so far. Full pipeline est: €2.20. At Starter (€990) = 99.8% margin. ✓"
4. Dynamic package recommendation: "Based on firm size + industry, recommend Professional for this lead"
5. Discount/coupon system for referrals, promotions, negotiations
**New DB tables**: pricing_configs (industry_id, package_name, price_onetime, price_monthly, active), discount_codes (code, type, value, max_uses, expires_at)

---

## CATEGORY 2: CONTENT TRUTH & QUALITY

### 🔴 BLOCKER: No Explicit Content Ethics Policy for AI Prompts

**Affects**: Step 9, Step 8, all Claude API calls that generate customer-facing text
**Problem**: The "golden rule: nothing invented" is stated but not implemented as concrete prompt instructions. The boundary between "reformulating facts" and "subtle exaggeration" is not defined. Example: From "founded 2008" the AI might write "Mit über 15 Jahren Erfahrung" (technically derived from facts) + "betreut das erfahrene Team Mandanten" (value judgment "erfahren" + assumption "betreut"). What's allowed? What's not?
**Required Fix**: Define explicit Content Ethics Policy as a system prompt component loaded for ALL content-generating Claude calls:
```
STRICTLY ALLOWED:
- Restating extracted facts in better sentence structure
- Combining facts from different sources into coherent text
- Adding factual context: "founded 2008" → "seit 2008 in München" (if location confirmed)
- Formatting improvements: bullet points, headings, structure

ALLOWED WITH SOURCE-TAG:
- Logical derivations: "since 2008" → "über 15 Jahre Erfahrung" (math, tag: derived_from:founding_year)
- Aggregations: "Rechtsgebiete: Familienrecht, Erbrecht, Mietrecht" → "drei Schwerpunktbereiche" (tag: derived_from:services_count)

STRICTLY FORBIDDEN:
- Value judgments: "erfahren", "kompetent", "führend", "renommiert", "erstklassig"  
- Unverifiable claims: "hunderte zufriedene Kunden", "höchste Qualität"
- Activity assumptions: "wir kümmern uns", "wir betreuen", "wir begleiten" (unless extracted)
- Superlatives without source: "bester", "modernste", "umfassendste"
- Emotional appeals not from source: "Ihr Wohlbefinden liegt uns am Herzen"
- ANY number/statistic not directly from extraction
```
This policy must be: stored in DB (configurable in Config Center), loaded into every content-generating prompt, checked post-generation by a validation pass.

### 🟠 CRITICAL: Fact-Check Has No "Uncertain" State

**Affects**: Step 9
**Problem**: Every fact is binary: confirmed or hallucination. But many facts are UNCERTAIN: Website says "geöffnet bis 18:00", Google says "geöffnet bis 19:00". Which is true? Current spec: "Google has priority." But Google can be wrong too. 
**Required Fix**: Three-state fact-checking: CONFIRMED (2+ sources agree), UNCERTAIN (sources conflict or only 1 source), FABRICATED (no source at all). UNCERTAIN facts get a yellow flag in the Content Approval Gate. You decide: use it, remove it, or mark it with "Stand: [date]". FABRICATED facts are auto-removed, no exception.

### 🟠 CRITICAL: No Staleness Detection for Extracted Content

**Affects**: Step 4, Step 9, Step 12
**Problem**: Content is extracted once and used as-is, even weeks later. Opening hours change. Team members leave. Prices increase. A preview showing wrong info destroys credibility.
**Required Fix**: 
1. Content timestamp on every extracted field (when was this information last confirmed?)
2. Before deploying preview (Gate 4): auto-check critical fields against Google Business API (opening hours, phone, status). If changed: flag for review.
3. On preview page: subtle "Stand: März 2026" indicator. Not prominent, but CYA for accuracy.
4. For prices: NEVER show prices on preview unless operator explicitly confirms they're current (checkbox in Gate 3.5)

### 🟠 CRITICAL: Testimonials Are Not Verified

**Affects**: Step 4, Step 9
**Problem**: Testimonials are extracted from the old website and reused. But many KMU websites have fake testimonials ("Maria S., zufriedene Kundin"). We republish them and become complicit. Google Reviews are more trustworthy but have usage restrictions.
**Required Fix**:
1. Extracted testimonials get a "source: website" tag. They are used but with attribution "Quelle: [website-url]"
2. Google Reviews are embedded via a widget or displayed with "Google Reviews" branding (per Google ToS)
3. If no testimonials and no Google Reviews (or <4.0 rating): testimonial section is OMITTED, not filled with fake ones
4. Operator can flag specific testimonials as "suspicious" in Gate 3.5 → they get removed

### 🟡 IMPORTANT: NanoBanana Image Quality Consistency

**Affects**: Step 9, Step 10
**Problem**: Multiple NanoBanana images per website may have inconsistent styles (different lighting, color temperature, composition). This makes the website look visually incoherent.
**Required Fix**: 
1. Define a "Visual Style Prompt Prefix" per website that's prepended to ALL NanoBanana calls for that lead. E.g., "Warm lighting, soft shadows, minimal composition, muted earth tones, professional photography style."
2. The style prefix is derived from: the lead's brand colors + industry profile visual benchmark + template variant.
3. After generation: AI Vision check that all images on the website share a coherent visual style. Flag if inconsistent.

### 🟡 IMPORTANT: No Curated Stock Photo Library as NanoBanana Fallback

**Affects**: Step 9
**Problem**: AI-generated images still have subtle tells in 2026. Stock photos (properly licensed) look more natural.
**Required Fix**: Build/license a curated library of ~200 photos per pilot industry (coaches: office shots, laptop work, 1:1 conversations; friseure: salon interiors, styling tools, hair close-ups). Use these BEFORE NanoBanana. Priority chain: Own photos → Curated stock → NanoBanana → Omit image.

---

## CATEGORY 3: CONVERSION & BUSINESS MODEL

### 🟠 CRITICAL: Cold Email Conversion Rate Assumption is Optimistic

**Affects**: Business model, Unit Economics
**Problem**: The spec assumes 6% conversion (preview visit → payment). For COLD outreach to businesses that never heard of you, selling a €990+ product, this is extremely optimistic. Industry benchmarks for cold email B2B SaaS are 1-3% reply rate, not purchase rate. Even with a stunning preview, 6% is aspirational.
**Required Fix**: 
1. Model three scenarios: Optimistic (6%), Realistic (2-3%), Pessimistic (1%). Calculate unit economics for each.
2. At 2% conversion: CAC = 55 leads × €0.80 avg = €44 / 0.02 = ~€2,200. That's MORE than the Starter package (€990). Need Professional+ to be profitable. This fundamentally changes the business model.
3. Pre-launch validation: Before building the full machine, run a MANUAL test: Create 10 previews by hand, send outreach manually, measure actual conversion. This costs ~€100 and 2 days of work. It validates the entire business model before investing months of development.
**Impact**: If conversion is <2%, the Starter package at €990 may not be viable. Consider: higher prices, or focus only on Professional/Premium, or add a "free consultation" step that increases conversion through personal contact.

### 🟠 CRITICAL: No Pre-Launch Validation Plan

**Affects**: Entire project
**Problem**: We're building a 17-step automated machine without validating the core hypothesis: "Will businesses buy a website from a cold email with a preview link?" This should be tested BEFORE building automation.
**Required Fix**: Add a "Phase 0: Validation" before MVP:
1. Manually scrape 50 coaches + 50 friseure in one city
2. Manually crawl and analyze their websites (using Claude, but manually orchestrated)
3. Manually build 10 preview websites (using templates, but hand-assembled)
4. Manually write and send outreach emails (personal email, not Instantly)
5. Measure: Open rate, preview visit rate, reply rate, conversion rate
6. If conversion >1%: Build the machine. If <1%: Iterate on the approach (pricing, targeting, outreach) before automating.
Total cost: ~€50-100 in API costs + 2 weeks of manual work. Could save months of wasted development.

### 🟡 IMPORTANT: No ROI Calculator on Pricing Page

**Affects**: Step 15 (Payment)
**Problem**: Lead sees "€990" and thinks "that's expensive." They don't think "one new client per month = €X revenue, website pays for itself in Y weeks."
**Required Fix**: Industry-specific ROI calculator on pricing page:
- Coaches: "1 neuer Klient/Monat à €150/Session × 10 Sessions = €1.500 Mehr-Umsatz/Monat. Website amortisiert sich in 3 Wochen."
- Anwälte: "1 neuer Mandant/Quartal à €3.000 Honorar = €12.000/Jahr. Website amortisiert sich in 1 Monat."  
- Friseure: "5 neue Stammkunden à €50/Monat = €250 Mehr-Umsatz/Monat. Website amortisiert sich in 4 Monaten."
Numbers configurable per industry in Config Center.

### 🟡 IMPORTANT: No Value-Anchor on Pricing Page

**Affects**: Step 15
**Problem**: €990 in isolation feels expensive. Next to "€5.000-15.000 (typical agency price)" it feels like a steal.
**Required Fix**: Add anchor comparison: "Eine Webdesign-Agentur berechnet €5.000-15.000 für eine vergleichbare Website. Webolution: ab €990." With source/disclaimer that agency prices are industry averages.

### 🟡 IMPORTANT: No Phone/Consultation Flow Defined

**Affects**: Step 15, Admin Backend
**Problem**: Lead books a Calendly call. Then what? No call script. No objection handling for verbal conversation. No CRM-style pre-call brief. No post-call follow-up process. No outcome tracking (interested → follow-up in 1 week → converted).
**Required Fix**: 
1. Pre-call brief: Admin auto-shows lead detail + preview URL + score + key arguments when a Calendly booking comes in
2. Call script per industry (not word-for-word, but talking points + objection responses)
3. Post-call logging: Outcome dropdown (Converted, Follow-up, Not interested, No-show) + notes
4. Follow-up automation: "Follow-up in X days" → reminder in Action Required inbox
**New DB table**: call_logs (lead_id, user_id, scheduled_at, duration_minutes, outcome, notes, follow_up_date)

---

## CATEGORY 4: OPERATOR CONTROL & UX

### 🟠 CRITICAL: No Speed-Review Mode for Gate Queues

**Affects**: Admin Backend, all Gate screens
**Problem**: After a weekend, 200+ leads may be waiting at Gate 1. Reviewing them one-by-one in a table is tedious. At scale (1000+ leads/week), the operator becomes the bottleneck at every gate.
**Required Fix**: Speed Review Mode for every gate: Keyboard-driven (→ approve, ← reject, ↓ skip, Space = expand details). Auto-loads next lead after action. Shows only the critical info needed for decision (not full detail). Counter: "47/200 reviewed, 38 approved, 6 rejected, 3 skipped." Smart pre-selection: "Approve all with Confidence >0.8?" → one click approves the obvious ones, leaves the borderline ones for manual review.

### 🟠 CRITICAL: No Undo for Gate Decisions

**Affects**: Admin Backend
**Problem**: Accidentally approved a bad lead at Gate 3 → it goes into the rebuild pipeline → costs money. No way to undo.
**Required Fix**: 
1. Every gate decision is logged in approval_logs with timestamp and user
2. "Undo" button available for 10 minutes after decision (while lead hasn't moved to next step yet)
3. After 10 minutes or if next step already started: "Revert" button that moves lead back to previous gate (may lose work done in current step)

### 🟠 CRITICAL: AI Prompts Not Editable in Backend

**Affects**: Steps 4, 5, 8, 9, Admin Backend Config Center
**Problem**: AI prompts for extraction, review, planning, polishing are in code. If you notice "the AI keeps missing opening hours for dentists" after 50 leads, you have to change code. Should be editable in Config Center.
**Required Fix**: 
1. Store AI prompts in DB table: ai_prompts (step, industry_tag, prompt_template, version, active)
2. Config Center: Prompt Editor with version history, diff view, rollback
3. A/B testing: Run two prompt versions in parallel, compare output quality
4. Prompt variables: {{industry_name}}, {{required_fields}}, {{banned_phrases}} are injected from industry profile at runtime

### 🟡 IMPORTANT: No Lead Pause / Hold Feature

**Affects**: Admin Backend, Orchestrator
**Problem**: You want to pause a specific lead ("wait, I know this person, let me call them first") while the rest of the campaign continues. No per-lead pause exists.
**Required Fix**: Per-lead "Hold" state. Lead exits the queue and enters "held" state. Stays there until manually released. Visible in Lead Detail and in a "Held Leads" tab in Lead Explorer.

### 🟡 IMPORTANT: No Pre-Rebuild Instructions

**Affects**: Step 8, Gate 3, Admin Backend  
**Problem**: At Gate 3, you approve a lead for rebuild. But you can't give instructions: "Use coach-authority template" or "Emphasize the podcast" or "Don't include prices, they're outdated." The AI creates the plan with zero input from you.
**Required Fix**: Gate 3 approval includes an optional "Instructions" text field. Whatever you type is injected into the rebuild planning prompt as operator instructions. The rebuild plan then shows: "Operator requested: [your instructions]" and the AI incorporates them.

### 🟡 IMPORTANT: No Notification on Behavioral Triggers

**Affects**: Step 14, Admin Backend
**Problem**: Behavioral triggers (preview visited 3x, CTA clicked without purchase) currently trigger AUTOMATIC follow-up actions. The operator should be NOTIFIED and DECIDE.
**Required Fix**: Every behavioral trigger creates an entry in the Action Required inbox: "Lead X visited preview 3x in 24h. Suggested action: Phone call. [Approve auto-action] [Call manually] [Ignore]". Configurable per trigger: auto-execute vs. ask operator.

---

## CATEGORY 5: LEGAL & COMPLIANCE

### 🔴 BLOCKER: UWG Cold Email Legality Not Resolved

**Affects**: Steps 13-14, entire outreach model
**Problem**: B2B cold email in Germany under UWG §7 Abs. 2 Nr. 3 requires "mutmaßliche Einwilligung" (presumed consent). This is legally ambiguous. A competitor or unhappy lead could file an Abmahnung (cease-and-desist with legal fees, typically €1.000-5.000). At scale (500+ emails/month), the risk of at least one complaint is significant.
**Required Fix**:
1. BEFORE launch: Consult a lawyer specialized in UWG/Wettbewerbsrecht. Get a written legal opinion on: our specific email template, our targeting method, our opt-out process.
2. In the email: Make the business relevance extremely explicit ("Wir haben uns Ihre Website als [Branche] in [Stadt] angesehen" = clear Geschäftsbezug)
3. Keep meticulous records: for every email sent, document the public source of the contact data (Impressum URL), the business relevance justification, the timestamp, the opt-out mechanism
4. Set a complaint threshold: if >0.3% of recipients complain to ISP/legal → immediately pause outreach and review approach
5. Consider alternative first-touch: Instead of cold email with preview link, first email could be JUST the analysis report (no sales pitch). Second touch introduces the preview. This positions the first email as "helpful information" rather than "advertising."

### 🔴 BLOCKER: No AGB (Terms of Service)

**Affects**: Step 15 (Payment)
**Problem**: Selling a service without AGB is legally negligent in Germany. Must define: scope of service, liability limitations, intellectual property rights, cancellation terms, payment terms, warranty, applicable law, jurisdiction.
**Required Fix**: Have a lawyer draft AGB specifically for: website creation service, hosting service, managed DNS transfer. Key clauses: "Website content is based on publicly available information. Accuracy is not guaranteed. Customer is responsible for verifying content accuracy before use." This limits liability for content errors.

### 🟠 CRITICAL: Preview Websites Need Their Own Impressum

**Affects**: Step 12
**Problem**: Every website accessible under webolution.de is a Telemedium (TMG §5) and needs an Impressum. Currently not specified whose Impressum appears on the preview.
**Required Fix**: Preview footer includes: "Dies ist eine Vorschau, erstellt von [SocialCooks GmbH / entity], [address]. Kein Angebot von [Lead-Firmenname]." Links to SocialCooks Impressum and Datenschutzerklärung. This is separate from any Impressum link that belongs to the lead's content.

### 🟠 CRITICAL: Competitor Name Usage in Emails May Be Problematic

**Affects**: Steps 6, 13, 14
**Problem**: Outreach emails mention competitors by name: "Kanzlei Schmidt hat Online-Termine." This could be considered vergleichende Werbung (comparative advertising, §6 UWG). Comparative advertising in Germany is allowed only if: objective, verifiable, not misleading, not denigrating. "Kanzlei Schmidt hat Online-Termine – Sie nicht" could be seen as denigrating the lead. And using the competitor's name without consent is risky.
**Required Fix**: 
1. Do NOT use competitor names in outreach emails. Instead: "Ein Mitbewerber in Ihrer Straße bietet bereits Online-Termine an." (anonymous comparison)
2. Competitor names are used ONLY in the internal analysis and on the preview's comparison section (which the lead sees voluntarily by clicking a button, not in unsolicited email)
3. In the preview comparison section: use competitor screenshots with attribution but no negative framing. "So sieht ein vergleichbarer Auftritt in Ihrer Region aus." Not: "So viel besser ist Ihr Konkurrent."

### 🟠 CRITICAL: No Data Processing Agreement (AVV) Strategy

**Affects**: All steps using third-party APIs
**Problem**: GDPR requires Auftragsverarbeitungsverträge (AVV) with all processors handling personal data. We send personal data (names, emails, addresses) to: Google (Places API), Anthropic (Claude API), NeverBounce, Instantly.ai, Vercel, Stripe, NanoBanana. Each needs an AVV.
**Required Fix**: 
1. Document all third-party processors in a data processing register
2. Ensure each has a GDPR-compliant DPA/AVV (most major providers have standard ones – but they must be signed)
3. For Claude API: personal data in prompts must be handled per Anthropic's data processing terms. Review whether Anthropic's terms cover our use case.
4. Minimize personal data sent to AI: send firm name + URL + extracted content, but NOT personal email addresses or phone numbers in Claude prompts (not needed for content generation)

### 🟡 IMPORTANT: No Trademark Search for "Webolution"

**Affects**: Brand, Domain, Legal
**Problem**: "Webolution" might already be registered as a trademark. If so, we risk a trademark infringement claim.
**Required Fix**: Search DPMA (Deutsches Patent- und Markenamt) register + EUIPO for "Webolution" before launch. If taken: choose alternative name. If free: register it. Cost: ~€300 for DE trademark registration.

---

## CATEGORY 6: TECHNICAL RELIABILITY

### 🔴 BLOCKER: No Staging Environment Defined

**Affects**: Tech Architecture, CI/CD
**Problem**: The spec mentions "Deploy to Staging" in CI/CD but no staging environment is actually defined or budgeted. Without staging, every code change is tested on production. One bug can break the pipeline for all leads.
**Required Fix**: Define staging environment: separate DB (or same DB with staging schema), separate Redis instance (or prefixed queues), separate Vercel project for staging previews. Budget: ~€50-100/month additional.

### 🔴 BLOCKER: No Database Backup & Disaster Recovery Plan

**Affects**: Tech Architecture
**Problem**: The database contains ALL lead data, content packages, quality reports, customer data. If the DB is lost, EVERYTHING is lost. No backup strategy defined.
**Required Fix**: 
1. Daily automated backups (Supabase/Neon include this, but verify retention period – typically 7 days)
2. Weekly backup export to separate storage (S3/R2) for 90-day retention
3. Point-in-time recovery capability (Supabase Pro has this)
4. Disaster recovery runbook: "If DB is lost, here's how to restore"
5. Test backup restore quarterly

### 🟠 CRITICAL: No Circuit Breaker for External APIs

**Affects**: Tech Architecture, all services calling external APIs
**Problem**: If Google API goes down, every worker retries independently → 50 parallel retry cascades → potential rate limit ban when API comes back.
**Required Fix**: Implement Circuit Breaker pattern at the shared library level (packages/shared):
- CLOSED (normal): requests go through
- OPEN (after N failures): all requests immediately fail without calling API, for X seconds
- HALF-OPEN: after cooldown, one request goes through. If success: CLOSED. If fail: OPEN again.
Use a library like `opossum` for Node.js. One circuit breaker per external API, shared across all workers via Redis state.

### 🟠 CRITICAL: No Timezone Handling Defined

**Affects**: All timestamps, outreach scheduling, opening hours
**Problem**: Opening hours are in local time (Europe/Berlin). Outreach sending times are in local time. Database timestamps are... undefined. If stored in UTC but displayed without conversion: "Send email at 10:00" becomes "Send at 10:00 UTC = 11:00 CET in winter, 12:00 CEST in summer."
**Required Fix**: 
1. All database timestamps in UTC
2. All user-facing displays converted to Europe/Berlin
3. Outreach scheduling explicitly in Europe/Berlin timezone
4. Opening hours stored with timezone indicator
5. DST handling: test specifically for March and October transitions

### 🟡 IMPORTANT: Playwright Screenshot Consistency

**Affects**: Step 4, Step 6, Step 12
**Problem**: Chromium version changes with Playwright updates. Screenshots from today may look subtly different from screenshots in 3 months. For before/after comparison, both screenshots should be under identical conditions.
**Required Fix**: Pin Playwright + Chromium version in package.json. Update deliberately (not automatically) with visual regression testing. Docker image with fixed Chromium version for consistent screenshots.

### 🟡 IMPORTANT: No Health Check Endpoints

**Affects**: Tech Architecture, Monitoring
**Problem**: No standard health check endpoint defined for services. Kubernetes/Docker health checks, load balancers, and monitoring all need /health endpoints.
**Required Fix**: Every service exposes GET /health returning: { status: "ok", service: "scraper", version: "1.0.0", uptime: 3600, queue_depth: 5, last_processed: "2026-03-08T14:23:00Z" }. Monitoring polls these endpoints.

---

## CATEGORY 7: SCALING PREPARATIONS

### 🟡 IMPORTANT: Automation Paradox (Gates vs. Scale)

**Affects**: Admin Backend, entire operation model
**Problem**: 5 manual gates × 1000 leads/week = the operator reviews 5000 decisions per week = 1000/day = 125/hour = 1 every 30 seconds for 8 hours straight. That's not possible.
**Required Fix**: Design the gate system for PROGRESSIVE automation:
1. MVP: All gates MANUAL (you review everything, <100 leads/week)
2. Growth: Gate 1+2 on AUTO (confidence-based), Gate 3-5 MANUAL (200-500 leads/week)
3. Scale: Gate 1-3 on AUTO, Gate 4+5 MANUAL (500-2000 leads/week)
4. Full Scale: Gate 1-4 on AUTO, Gate 5 MANUAL for high-value leads only (2000+ leads/week)
Each gate's AUTO threshold is configurable. Smart pre-filtering: "Show me only the leads that AUTO would have rejected" (review the edge cases, not the obvious ones).

### 🟡 IMPORTANT: No Multi-Region Scraping Strategy

**Affects**: Step 1, Campaigns
**Problem**: Spec focuses on single-city campaigns. But at scale, you want to run across 50+ cities simultaneously. Deduplication becomes complex (coach in Munich may also appear in Augsburg search).
**Required Fix**: 
1. Cross-city dedup: global dedup, not per-campaign
2. Region hierarchy: City → Bundesland → National. A lead belongs to ONE city (based on address), even if found in multiple city searches
3. Market saturation tracking: "In München, 80% of coaches already scraped. Switch to Augsburg."

### 🔵 IMPROVEMENT: No A/B Testing Framework in Pipeline

**Affects**: Steps 8-14
**Problem**: A/B test MANAGER exists in admin backend spec, but the PIPELINE doesn't support it. There's no mechanism for: "50% of coaches get template A, 50% get template B. Track which converts better."
**Required Fix**: 
1. A/B test assignments stored on lead record: leads.ab_test_assignments (JSONB: { "template_test": "variant_a", "subject_line_test": "variant_b" })
2. Pipeline services check for active A/B tests and use the assigned variant
3. Reporting: conversion rate per variant with statistical significance

---

## CATEGORY 8: DATABASE SCHEMA ADDITIONS NEEDED

All findings above require the following schema additions:

```
// New tables
pricing_configs   (id, industry_tag, package_name, price_onetime, price_monthly, is_default, active)
discount_codes    (id, code, type, value, max_uses, uses_count, expires_at, created_at)
approval_logs     (id, lead_id, gate, decision, user_id, notes, cost_estimate, created_at)
call_logs         (id, lead_id, user_id, scheduled_at, duration_min, outcome, notes, follow_up_date)
ai_prompts        (id, step, industry_tag, prompt_template, version, active, created_at)
content_versions  (id, lead_id, field_path, old_value, new_value, changed_by, created_at)

// New fields on existing tables
leads.cumulative_cost       Decimal   -- running cost total for this lead
leads.approval_gate         Integer?  -- which gate is this lead waiting at (1-5, null = not waiting)
leads.is_vip                Boolean   -- priority flag
leads.held                  Boolean   -- paused by operator
leads.manual_instructions   Text?     -- operator instructions for rebuild planning
leads.predicted_package     String?   -- which package will this lead likely buy
leads.ab_test_assignments   JSONB?    -- active A/B test variant assignments

campaigns.budget            Integer?  -- budget in cents
campaigns.budget_spent      Integer   -- running total spent
campaigns.margin_target     Decimal?  -- minimum margin percentage

industry_profiles.config    JSONB     -- add: cost_limits, margin_targets, pricing_config_id, prompt_ids

// Extend audit_log
audit_log.estimated_cost    Integer?  -- pre-estimated cost (vs actual cost_cents)
```

---

## CATEGORY 9: INDUSTRY PROFILE ADDITIONS NEEDED

Each industry profile JSON needs these additional sections:

```json
{
  "costLimits": {
    "maxCostPerLead": 300,        // cents
    "maxEnrichmentCost": 30,      // cents  
    "maxCrawlCost": 20,           // cents
    "maxRebuildCost": 100         // cents
  },
  "marginTargets": {
    "minimumMarginPercent": 85,
    "recommendedPackage": "professional"
  },
  "pricingConfig": {
    "starter": { "onetime": 99000, "monthly": 0 },
    "professional": { "onetime": 199000, "monthly": 4900 },
    "premium": { "onetime": 349000, "monthly": 9900 }
  },
  "roiCalculator": {
    "avgNewClientsPerMonth": 3,
    "avgRevenuePerClient": 15000,
    "paybackPeriodWeeks": 3
  },
  "contentEthics": {
    "allowDerivedFacts": true,
    "requireSourceTag": true,
    "bannedValueJudgments": ["erfahren", "kompetent", "führend", ...],
    "industrySpecificRestrictions": ["keine Heilversprechen"]  // for doctors
  },
  "outreachLimits": {
    "maxLeadsPerWeekPerCity": 50,   // prevent market saturation
    "coolingPeriodDays": 365,
    "maxSequencesPerDay": 20
  },
  "callScript": {
    "introduction": "...",
    "keyArguments": ["...", "..."],
    "objectionHandling": { "tooExpensive": "...", "noTime": "...", "alreadyHaveAgency": "..." }
  }
}
```

---

## CATEGORY 10: EVENT CATALOG ADDITIONS NEEDED

New events required for gates, costs, and operator control:

```
// Gate events
lead.awaiting_gate_1          -- after validation, waiting for approval
lead.awaiting_gate_2          -- after enrichment
lead.awaiting_gate_3          -- after scoring  
lead.awaiting_content_approval -- after polish (Gate 3.5)
lead.awaiting_gate_4          -- after QA (BEFORE deploy)
lead.awaiting_gate_5          -- after outreach prep
lead.gate_approved            -- payload: { gate, userId, notes }
lead.gate_rejected            -- payload: { gate, userId, reason }

// Cost events
cost.estimated                -- before a costly step: { leadId, step, estimatedCents }
cost.incurred                 -- after a step: { leadId, step, actualCents }
campaign.budget_warning       -- campaign at 80% budget
campaign.budget_exceeded      -- campaign over budget → pipeline paused

// Operator control events  
lead.held                     -- operator paused this lead
lead.released                 -- operator resumed this lead
lead.vip_set                  -- lead marked as VIP
lead.instructions_set         -- operator added rebuild instructions
lead.content_edited           -- operator edited polished content
```

---

## PRIORITY SUMMARY

### Must fix BEFORE building (design-level changes):
1. 🔴 CMS layer for live websites
2. 🔴 Pipeline gates alignment (specs vs backend)
3. 🔴 Gate 4 before deploy (not after)
4. 🔴 Content approval gate (3.5)
5. 🔴 Cost estimation service
6. 🔴 Margin/pricing configuration
7. 🔴 Content ethics policy for AI prompts
8. 🔴 UWG legal opinion before launch
9. 🔴 AGB/Terms of Service
10. 🔴 Staging environment
11. 🔴 Database backup strategy

### Must fix BEFORE first outreach:
12. 🟠 Fact-check uncertain state
13. 🟠 Content staleness detection
14. 🟠 Testimonial verification
15. 🟠 Cold email conversion reality check (pre-launch validation!)
16. 🟠 Preview impressum
17. 🟠 No competitor names in emails
18. 🟠 AVV with all data processors
19. 🟠 Speed review mode for gates
20. 🟠 Undo for gate decisions
21. 🟠 AI prompts editable in config
22. 🟠 Circuit breaker for APIs
23. 🟠 Timezone handling

### Fix in first month of operation:
24. 🟡 NanoBanana style consistency
25. 🟡 Stock photo library
26. 🟡 ROI calculator on pricing page
27. 🟡 Value anchor on pricing page
28. 🟡 Phone/consultation flow
29. 🟡 Lead hold/pause feature
30. 🟡 Pre-rebuild instructions
31. 🟡 Behavioral trigger notifications
32. 🟡 Trademark search
33. 🟡 Progressive gate automation design
34. 🟡 Screenshot consistency
35. 🟡 Health check endpoints

---

## QA ROUND 4: FUNDAMENTAL GAPS (20 additional findings)

These findings go DEEPER than the previous rounds. They address the actual engineering reality of building and operating this system.

### 🔴 BLOCKER: Website Generation is a Black Box (F36)
**Affects**: Step 10 (Build), entire template system
**Problem**: HOW does a JSON content package become a working Next.js website? Does Claude generate React components? Does JSON map to template slots? What's the data flow? This is the hardest engineering challenge and has zero technical spec.
**Required**: Define the exact data flow: PolishedContent JSON → Template Engine reads JSON → maps to component props → Next.js renders pages → static export. Define the content-to-component mapping schema. Define how navigation is generated, how internal links work, how empty sections are handled.

### 🔴 BLOCKER: No Feature-Parity Analysis (F37)
**Affects**: Step 5, Step 8, conversion rate
**Problem**: If the old website has a Calendly booking, Treatwell widget, or chat – and our preview doesn't – the comparison shows a DOWNGRADE. Lead will never buy.
**Required**: Step 5 must detect functional features (booking tools, chat widgets, CRM forms, e-commerce). Step 8 must plan how to replicate or integrate them (Calendly embed, Treatwell embed). If we can't replicate: transparent communication in the preview. Add "feature_parity_check" to the QualityReport schema.

### 🔴 BLOCKER: SEO Transition Will Harm Customers (F38)
**Affects**: Step 16 (Onboarding), customer satisfaction
**Problem**: Old URLs are indexed by Google. New website has different URL structure. Without 301 redirects, customer loses Google rankings. This is business-damaging and will cause refunds.
**Required**: Step 4 must map ALL old URLs. Step 10 must generate matching URL structure OR a redirect map. Step 16 must configure 301 redirects from every old URL to the corresponding new URL. Vercel supports _redirects file or next.config.js redirects.

### 🔴 BLOCKER: AI Prompts Not Specified (F39)
**Affects**: Steps 4, 5, 8, 9 – all AI-dependent steps
**Problem**: Prompts are the #1 quality determinant and they don't exist in the spec. Without versioned, tested prompt templates, output quality is random.
**Required**: Create a `/prompts` directory in the repo with: extraction.md (Step 4), review.md (Step 5), competitor-analysis.md (Step 6), rebuild-planning.md (Step 8), content-polish.md (Step 9), visual-qa.md (Step 11). Each includes: system prompt, few-shot examples, output JSON schema, industry-specific variations. Prompts are stored in DB (ai_prompts table) and editable in Config Center.

### 🔴 BLOCKER: No End-to-End Pipeline Test (F40)
**Affects**: Tech Architecture, CI/CD, quality assurance
**Problem**: No automated test pushes a fake lead through all 17 steps to verify the output. Any code change could silently break the pipeline.
**Required**: E2E test script that: creates a test lead with known data → runs through all steps (with mocked external APIs) → asserts: website builds, content matches input, no hallucinations, QA passes, preview deploys. Runs on every CI build. Blocks deployment if failing.

### 🔴 BLOCKER: Vercel Deployment Mechanics Undefined (F41)
**Affects**: Step 12 (Deploy), scaling
**Problem**: Is each preview a separate Vercel project? Or one monorepo with dynamic routes? How is the built Next.js app uploaded? Git push or API upload? Wildcard subdomain configuration?
**Required**: Define deployment strategy. Recommended: Vercel Deploy API with tarball upload (no git repo per preview). One Vercel project with wildcard domain. Each preview is a separate deployment with unique alias (firma-name.webolution.de). Configuration documented.

### 🔴 BLOCKER: Email Deliverability Setup Missing (F42)
**Affects**: Step 14 (Outreach) – emails will land in spam without this
**Problem**: No SPF, DKIM, DMARC configuration mentioned for sender domains. Without these, 80%+ of emails go to spam.
**Required**: For each sender domain: SPF record allowing Instantly.ai sending IPs, DKIM key pair configured in DNS + Instantly, DMARC policy (start with p=none, move to p=quarantine). Custom tracking domain for link tracking (avoiding Instantly's default tracking domain which is flagged by spam filters). Document the complete DNS setup per sender domain.

### 🟠 CRITICAL: Preview → Production Transition Undefined (F43)
**Affects**: Step 12, Step 16
**Problem**: After purchase, how does the preview become the production website? Does the sales banner get removed? Does the webolution.de subdomain redirect to the customer's domain? What about the transition period during DNS propagation?
**Required**: Define transition flow: Payment → remove sales banner from preview → preview becomes temporary production → DNS switch → customer domain points to same deployment → webolution.de subdomain 301-redirects to customer domain → done.

### 🟠 CRITICAL: Image Rights of Extracted Images (F44)
**Affects**: Step 4, Step 9, Step 16, legal
**Problem**: We extract images from the lead's old website and republish them. The lead may not own the rights (photographer retains rights, unlicensed stock photos). On the preview (our domain): possibly covered by Zitatrecht. On the customer's live domain: customer needs the rights.
**Required**: 1) In preview: watermark all non-logo images as "Vorschau" 2) In onboarding: include clause "Please confirm you own the rights to all images on your current website" 3) In AGB: liability for image rights rests with the customer 4) Offer: "We can replace images with licensed stock or custom photography" (upsell)

### 🟠 CRITICAL: Business Math Requires Multi-City from Day 1 (F45)
**Affects**: MVP planning, business model
**Problem**: A single city (e.g., Augsburg, 300k) has ~20-80 coaches. After filtering: 2-8 full pipeline leads per city. At 6% conversion: 0-0.5 customers per city. Need 10+ cities from the start to get meaningful results.
**Required**: MVP plan must target 5-10 cities simultaneously, not 1 pilot city. This changes the scraping campaign strategy (parallel multi-city) and the admin backend (city-level reporting). Rewrite the MVP roadmap to reflect this.

### 🟠 CRITICAL: No Reply-Response Process (F46)
**Affects**: Step 14, conversion rate
**Problem**: Lead replies "What does it cost?" and nobody answers for 12 hours because you're asleep. Interest dies. No response SLA, no pre-written templates, no escalation.
**Required**: 1) Response SLA: max 4h during business hours (8-18 CET) 2) Pre-written response templates per reply type in Config Center 3) Notification: immediate push notification (Slack + email + in-app) when lead replies 4) If no response after 2h: auto-send acknowledgment "Danke für Ihre Nachricht – wir melden uns innerhalb weniger Stunden bei Ihnen"

### 🟠 CRITICAL: No Website-Freshness-Check Before Outreach (F47)
**Affects**: Step 13, Step 14, reputation
**Problem**: We crawled the website 3 weeks ago. Since then, the lead relaunched. We send "Your website is outdated" to someone with a brand-new website. We look like amateurs.
**Required**: Before outreach (Step 13): quick re-check of homepage (HTTP request + screenshot comparison with stored screenshot). If visual similarity <70%: flag "Website may have changed since analysis." Operator decides: re-analyze or skip outreach.

### 🟠 CRITICAL: AI Model Regression Detection (F48)
**Affects**: All AI-dependent steps, quality over time
**Problem**: Claude model updates can subtly change output quality. No regression testing against known outputs.
**Required**: Monthly regression test: run all prompts against 10 test inputs from the calibration set. Compare outputs with stored "known good" outputs. Score drift. If drift >threshold: alert + freeze AI-dependent steps until reviewed.

### 🟡 IMPORTANT: Template Multi-Page Mechanics Undefined (F49)
**Affects**: Step 10
**Problem**: Navigation generation, internal linking, routing for multi-page templates (3-5 pages) not specified. How are pages that would be empty handled?
**Required**: Define: navigation is auto-generated from page_structure in rebuild plan. Pages with no content are excluded from navigation. Internal links use Next.js Link component. Breadcrumbs auto-generated. 404 page included.

### 🟡 IMPORTANT: No "Agency Already Working" Detection (F50)
**Affects**: Step 3, Step 14, reputation
**Problem**: Lead already has an agency working on a relaunch. Our email "your website is bad" offends the agency and the lead.
**Required**: In Step 4: check Wayback Machine for recent changes. If significant change in last 6 months: flag "possible active agency." In outreach: softer tone or skip.

### 🟡 IMPORTANT: Customer Post-Purchase Communication Gap (F51)
**Affects**: Step 16, customer satisfaction
**Problem**: Customer pays €990-3.490 and then hears nothing for 24-48 hours until the website is live. Creates anxiety.
**Required**: Define post-purchase communication flow: Immediately: confirmation email + "Here's what happens next (3 steps)". Hour 1: onboarding form link. When form submitted: "Received! Starting setup, estimated completion: [date]". During DNS switch: "Switching your domain now, 15-60 minutes." When live: "Your new website is live! [link]"

### 🟡 IMPORTANT: No Pre-Purchase Customization Path (F52)
**Affects**: Step 15, conversion rate
**Problem**: Lead says "I like it but want the hero photo changed before I buy." Our process: pay first, changes after. Lead doesn't trust us enough to pay without seeing the change.
**Required**: Allow 1-2 small changes before purchase. In Gate 5 outreach or as reply to interested leads: "We can adjust 1-2 things in the preview before you decide." This removes friction. Limit: text changes and image swaps only, no structural changes. Track pre-purchase customization cost and factor into margin.

### 🟡 IMPORTANT: Cross-Channel Stalking Perception (F53)
**Affects**: Step 14, reputation
**Problem**: Email → Instagram DM → LinkedIn without acknowledgment looks like automated stalking. Lead feels harassed.
**Required**: Each channel switch must reference the previous attempt: "Ich hatte Ihnen letzte Woche eine Email geschickt – falls die untergegangen ist..." Only 1 channel switch per lead (not email → Instagram → LinkedIn → letter). Max 2 channels total.

### 🟡 IMPORTANT: Vendor Dependency Strategy Missing (F54)
**Affects**: Tech Architecture, business continuity
**Problem**: Critical dependency on 5+ vendors. If one changes pricing or shuts down, a core pipeline component breaks.
**Required**: Document Plan B for each: Instantly→Smartlead, Vercel→Netlify/Cloudflare Pages, NanoBanana→DALL-E API/Flux, NeverBounce→ZeroBounce, Google Places→OpenStreetMap+Scraping. Not implemented, just documented and evaluated annually.

### 🔵 IMPROVEMENT: No Quick Mockup at Gate 3 (F55)
**Affects**: Admin Backend, Gate 3 decision quality
**Problem**: At Gate 3 you decide which leads to rebuild, but you don't see what the result would look like. You decide blind based on scores.
**Required**: "Quick Preview" button at Gate 3: generates a 10-second mockup (template + brand colors + hero text, no full build). Shows roughly what the website WOULD look like. Helps decision: "Yes, this will look great" vs "Hmm, not enough content for a good preview."

---

## QA ROUND 5: MACHINE-LEVEL WEAKNESSES (18 findings)

These address what would actually BREAK or UNDERPERFORM in real-world operation.

### 🔴 BLOCKER: Content Volume Problem – Not Enough Raw Material (F56)
**Affects**: Steps 8-10, content quality, conversion rate
**Problem**: A typical solo coach has a Jimdo site with 200 words. Building a 3-5 page website from 200 words is impossible without inventing content – violating the golden rule.
**Required**: 1) Minimum content threshold per template (e.g., coach-landing needs: name + coaching type + 1 method sentence + 1 contact method). Below threshold: lead goes to nurture, not rebuild. 2) Content-Gap-Report visible at Gate 3: "Missing: about text, testimonials, team photos, prices. Website will be thin." 3) Template selection driven by AVAILABLE CONTENT, not firm size. 200 words = single page max. 500+ = 3-page. 1000+ = full multi-page.

### 🔴 BLOCKER: No Template Creation Plan (F57)
**Affects**: Step 10, entire output quality
**Problem**: WHO designs and builds the 4 pilot templates? No designer specified, no design requirements, no budget, no timeline. Templates are the quality CEILING of the entire system.
**Required**: 1) Hire experienced web designer for template creation. Budget: €2,000-4,000 per template. 2) Template Quality Standard document: design principles, micro-interactions, performance, accessibility (BFSG). 3) Each template must be tested with 5 real content packages before production use. 4) Template iteration process based on conversion data.

### 🔴 BLOCKER: Cold Start – Machine Launches Without Data (F58)
**Affects**: All steps, quality, scoring accuracy
**Problem**: Day 1: no calibration sets, no scoring calibration, no conversion data, no prompt refinements, no email baselines, no testimonials. First 50-100 leads are experiments but spec treats them as production.
**Required**: Explicit "Learning Phase" plan: Weeks 1-2 create calibration sets. Weeks 3-4 scrape+score 50 leads, manually validate scoring. Weeks 5-6 build 20 websites, manually review each, iterate prompts. Weeks 7-8 deploy 10 previews, send 10 outreach emails MANUALLY. Week 9+ begin gradual automation.

### 🔴 BLOCKER: AI Has No Quality Floor (F59)
**Affects**: Steps 4, 5, 8, 9 – all AI-dependent steps
**Problem**: All critical outputs are AI-generated. If Claude's quality degrades (model update, load issues), there's no safety net. Confidence scores are self-assessed by the same AI – a hallucinating AI rates itself as confident.
**Required**: Deterministic post-AI validation checks (NOT AI-based): Extraction: firm_name must appear in crawled HTML (string match). Polish: every sentence must contain ≥1 word from original extraction (fuzzy match). Planning: every "NEW_FROM_DATA" section must have a source_reference pointing to an extracted field. These rule-based checks catch worst-case AI failures regardless of AI self-assessment.

### 🟠 CRITICAL: "Too Good To Be True" Perception Not Addressed (F60)
**Affects**: Step 14 (Outreach), open rates, conversion
**Problem**: Unknown company emails "we built you a free website" = SCAM perception. Most recipients will ignore or report.
**Required**: 1) Anti-scam positioning in email 1: explain WHO we are and WHY we do this before anything else. 2) Inline screenshot of the new website directly in the email (not just a link). 3) Company information visible: real name, real address, real phone number. 4) No urgency/scarcity in email 1 – pure value and transparency.

### 🟠 CRITICAL: Cookie Consent for Embedded Services (F61)
**Affects**: Step 10 (Build), legal compliance (TTDSG)
**Problem**: Generated websites will embed Google Maps, possibly YouTube, Google Fonts – all require cookie consent under TTDSG.
**Required**: Architectural decision: avoid ALL cookie-setting external services. Google Maps → OpenStreetMap embed. Google Fonts → self-hosted fonts. YouTube → thumbnail with external link. Social feeds → static screenshots. This improves both privacy AND performance.

### 🟠 CRITICAL: BFSG Accessibility Is Now Mandatory (F62)
**Affects**: Step 10 (Build), templates, legal
**Problem**: BFSG (Barrierefreiheitsstärkungsgesetz) is law since June 2025. Websites offering services to consumers must be accessible. Not optional.
**Required**: BFSG/WCAG AA compliance built into every template from the start: keyboard navigation, proper ARIA labels, sufficient contrast (including check against brand colors – if brand colors fail contrast: override), alt texts for all images, no color-only indicators. Contrast checker must be part of the brand-keep/upgrade decision.

### 🟠 CRITICAL: DNS Reality at German Hosters (F63)
**Affects**: Step 16 (Onboarding)
**Problem**: Many German hosters (Strato, All-Inkl) couple A-record and mail routing internally. Changing A-record CAN break email even without touching MX records. Some don't allow external A-records at all.
**Required**: 1) Provider compatibility database: which providers support external A-records cleanly? 2) Provider check during enrichment (WHOIS) – flag problematic providers at Gate 3 3) For incompatible providers: document domain transfer process as part of onboarding (adds 3-7 days) 4) Provider-specific step-by-step guides with screenshots

### 🟠 CRITICAL: Multi-Location Businesses Not Detected (F64)
**Affects**: Steps 1-3, deduplication, outreach
**Problem**: A firm with 3 offices appears as 3 separate leads. Gets 3 rebuilds and 3 outreach emails to the same person.
**Required**: Multi-location detection in Step 2: if multiple leads share firm name + website domain → merge into single lead with multiple addresses. Website template needs a locations page instead of single address in footer.

### 🟠 CRITICAL: No Minimum-Improvement-Threshold (F65)
**Affects**: Steps 11-12, conversion rate
**Problem**: If the new website scores only slightly better than the old one, the before/after comparison is unconvincing. A mediocre improvement is WORSE than none (shows we can't do much better).
**Required**: Step 11 QA must include: new website quality score vs old website quality score. If improvement <30 points: website NOT deployed. Lead goes to nurture instead of outreach. Only deploy previews where the difference is DRAMATIC and immediately visible.

### 🟡 IMPORTANT: Form Routing After Purchase (F66)
**Affects**: Step 16, customer satisfaction
**Problem**: Contact form sends to "info@firma.de" from enrichment. May not be the right address for inquiries.
**Required**: Onboarding form includes: "Where should contact form submissions be sent?" with option for multiple addresses. Test submission at go-live with customer confirmation.

### 🟡 IMPORTANT: "I Already Have an Agency" Handling (F67)
**Affects**: Steps 3-4, 14
**Problem**: 30-40% of KMUs have an existing web designer. Our email implicitly says "your designer did a bad job." Offensive to both lead and designer.
**Required**: Step 4: detect "Designed by XYZ" in footer. If found: flag as "has agency." Outreach sequence changes to: not criticizing the old website, but emphasizing ADDITIONAL features and modernization. Different tone entirely.

### 🟡 IMPORTANT: Social Media Dominant Leads (F68)
**Affects**: Steps 3, 7, 14
**Problem**: Leads with 2000+ Instagram followers and no/bad website may not want a website. Instagram IS their online presence. Our value prop doesn't resonate.
**Required**: Social-media-dominance check in enrichment. If Instagram >2000 followers AND website absent/minimal: different value proposition needed ("complement your Instagram with a Google-findable website") or deprioritize lead.

### 🟡 IMPORTANT: Customer Activation Tracking (F69)
**Affects**: Step 17, churn prevention
**Problem**: Customer buys website but never uses it (doesn't update Google Business, doesn't link from social media). 0 traffic. Sees no value. Churns.
**Required**: Post-go-live activation checklist: Google Business updated? Social media bio linked? >50 visitors in first 30 days? If not: proactive "How to get the most from your new website" guide.

### 🟡 IMPORTANT: Revenue Model Cashflow Gap (F70)
**Affects**: Business model, sustainability
**Problem**: At 60% Starter (€990 one-time, no MRR), avg MRR per customer is ~€29. Need 40+ customers for MRR to cover infra. At 2% conversion: 40 customers takes 10 months. Cashflow gap for months 1-10.
**Required**: 1) Model three pricing scenarios and their cashflow impact 2) Consider: no pure one-time package. Minimum monthly fee for all packages (e.g., Starter: €790 + €29/month) 3) Plan funding/runway for the cashflow gap period

### 🟡 IMPORTANT: Google Business Profile Dependency (F71)
**Affects**: Step 3 (Enrichment), scoring accuracy
**Problem**: Business Health Score is 35% dependent on Google data (rating 20%, reviews 15%). Businesses without Google profiles score artificially low.
**Required**: Fallback scoring when no Google Business Profile: use alternative signals (Provenexpert, portal ratings, Instagram engagement). "No Google profile" itself becomes a positive signal: "we can help with Google Business optimization too" = upsell.

### 🟡 IMPORTANT: No Local-SEO Quick-Win (F72)
**Affects**: Step 17, customer satisfaction, churn
**Problem**: SEO improvements take weeks/months. Customer sees no immediate effect after purchase.
**Required**: Include basic Google Business Profile optimization as part of every package (not upsell). Quick-win: complete description, add photos, correct categories, add services. Customer sees results within days. Differentiator: "Nicht nur Website, sondern komplette Online-Präsenz."

### 🔵 IMPROVEMENT: Machine Has No Aesthetic Judgment (F73)
**Affects**: Step 10, perceived quality
**Problem**: Template-filled websites are CORRECT but not REMARKABLE. They pass all QA checks but lack the design flair that makes someone say "wow." The before/after shows "bad → correct" not "bad → stunning."
**Required**: 1) Honest quality promise in outreach/pricing: "professional, modern website" not implied "agency quality" 2) Templates must be designed by an experienced designer (not AI-generated) 3) Each template needs 2-3 "surprise" details (unexpected color accent, asymmetric element, micro-animation) that elevate it beyond generic template look

---

## UPDATED PRIORITY SUMMARY (All Rounds Combined)

### 🔴 BLOCKERS (19 total – must fix before MVP):
F1-F11 (from rounds 1-3) + F36-F42 (round 4) + F56-F59 (round 5)
Key themes: CMS layer, gate alignment, gate ordering, content approval gate, cost estimation, pricing config, content ethics, UWG legal, AGB, staging, backups, website generation mechanics, feature parity, SEO transition, prompt specs, E2E test, Vercel deployment, email deliverability, content volume, template creation, cold start, AI quality floor

### 🟠 CRITICAL (20 total – must fix before first outreach):
F12-F23 (rounds 1-3) + F43-F48 (round 4) + F60-F65 (round 5)

### 🟡 IMPORTANT (16 total – fix in first month):
F24-F35 (rounds 1-3) + F49-F55 (round 4) + F66-F72 (round 5)

### 🔵 IMPROVEMENT (2 total):
F55 (round 4) + F73 (round 5)

---

## QA ROUND 6: WEBSITE QUALITY DEEP-DIVE (Steps 1-12)

Focus: What must EACH step deliver so the final website is an INCREDIBLE quality uplift?

### 🔴 BLOCKER: Steps 1-3 Collect Wrong Data for Great Websites (F74)
**Affects**: Steps 1, 3, and downstream Steps 8-10
**Problem**: We collect data for FINDING leads (name, url, rating) but not for BUILDING great websites. Missing: Instagram content (often the BEST photos), Google Review TEXTS (best testimonials), portal profile content (richer than impressum), YouTube/Vimeo presence (embeddable video), brand personality signals, target audience, USP.
**Required**:
Step 1 additions: Instagram handle + follower count + last 20 post image URLs, YouTube channel URL if exists, portal profile URLs (anwalt.de, jameda, etc.)
Step 3 additions: Instagram content quality assessment (AI quick-check on 5 images: professional/amateur/mixed), Google Review text extraction (top 5 reviews by length/quality), Portal profile full-text scrape (not just contact data), Brand personality analysis from existing website tone + Instagram aesthetic + review response style (formal/casual/warm/clinical), Target audience inference from services + pricing + location + content style, USP identification from: unique services, unique qualifications, unique approach, awards, publications
New fields on Lead/ContentPackage: instagram_photos[], google_review_texts[], portal_profile_texts{}, brand_personality{formality, warmth, energy, style_keywords[]}, target_audience{}, usp[]

### 🔴 BLOCKER: No Creative Design Briefing in Rebuild Plan (F75)
**Affects**: Step 8 (Planning), Step 10 (Build), visual quality
**Problem**: The rebuild plan defines WHAT to build (template, sections, content sources) but not HOW it should FEEL. A web designer gets "Sandra is an energetic business coach for women leaders. Brand: powerful but warm. Colors: warm earth tones with bold accent. Imagery: authentic, natural. Hero message: immediately clear what she does and for whom." We give: "template: coach-authority, sections: hero, about, services, blog, contact."
**Required**: Step 8 must generate a Creative Brief as part of the rebuild plan:
- Emotional design goal: "This website should radiate trust and feel modern"
- Hero concept: "Large portrait photo with overlay text, CTA immediately visible"
- Visual direction: warm/cool, light/dark, minimal/rich, photographic/illustrated
- Content hierarchy: what the visitor should see first, second, third
- Key message: the ONE thing a visitor should remember after 5 seconds
- Brand personality keywords: 3-5 adjectives that define the feel
The Creative Brief is generated by Claude using: brand personality (from Step 3), target audience, USP, industry benchmarks, competitor design levels. It becomes the steering document for Steps 9 (content tone) and 10 (template configuration).

### 🔴 BLOCKER: Content Polish Produces Correct but Not Compelling Copy (F76)
**Affects**: Step 9, conversion rate
**Problem**: Current spec produces factual, clean text. But not text that makes someone pick up the phone. Missing: problem-solution framing, emotional headlines, benefit-oriented writing, strategic social proof placement, persuasive micro-copy.
**Required**: Add Copywriting Framework to the polish prompt:
1) Problem-Solution-Proof-CTA structure per section: Address the visitor's problem → present the solution → prove it works → tell them what to do next
2) Headline hierarchy: every section gets a headline that communicates a BENEFIT, not a label. Not "Über uns" but "Seit 2008 Ihr Partner für Familienrecht in München." Not "Leistungen" but "Was wir für Sie tun können"
3) Social Proof aggregation: collect ALL trust signals (Google rating, years in business, certifications, memberships, review quotes, client count if verifiable) and place them strategically (hero area for strongest, footer for badges)
4) Micro-copy upgrade: buttons ("Jetzt Termin anfragen" not "Absenden"), form labels (conversational), footer (branded, not generic), 404 page (helpful, on-brand)
5) First-person vs third-person: match the lead's existing style. If they write "Ich bin Sandra und..." keep it. If "Die Kanzlei Müller bietet..." keep it. Don't force a style change.

### 🟠 CRITICAL: Templates Cannot Build Emotional Websites (F77)
**Affects**: Step 10 (Build)
**Problem**: Template slots produce CORRECT but GENERIC layouts. Missing: content-adaptive sections (hero changes based on photo availability), conditional rendering (testimonials carousel if 5+, single quote if 1, hidden if 0), micro-interactions (scroll reveals, hover effects, smooth scroll), dark mode support.
**Required**:
1) Content-adaptive template logic: IF portrait_photo EXISTS → full-bleed hero with overlay ELSE → color-bg hero with text only. IF testimonials.count >= 3 → carousel ELSE IF 1-2 → quote block ELSE → omit section. IF price_list EXISTS → structured table ELSE IF prices_on_request → CTA section ELSE → omit.
2) Mandatory micro-interactions in every template: scroll-reveal animations (sections fade in), button hover states (scale + shadow), smooth scroll for anchor navigation, image lazy loading with fade-in. These are 2026 baseline expectations.
3) Dark mode: @media (prefers-color-scheme: dark) support in every template. Auto-adapts colors while maintaining brand identity.
4) Responsive image art direction: hero image has different crop/focal point on mobile vs desktop (not just resized).

### 🟠 CRITICAL: QA Tests Technique Not Experience (F78)
**Affects**: Step 11, quality assurance
**Problem**: 40+ technical tests but no experiential tests. A website can pass all tests and still feel generic, unconvincing, or forgettable.
**Required**: Add 4 experiential QA tests:
1) Emotional Impact Test: AI reviews new website with prompt "Would you trust this business and contact them based on this website alone? Why or why not?" Score 1-10.
2) Before/After Impact Test: AI sees both old and new website. "Is the improvement dramatic enough that a business owner would pay €990 for this upgrade?" Yes/No with reasoning. If No → do not deploy.
3) 5-Second Test: AI sees ONLY the hero section (screenshot cropped to viewport). "What does this business do? Who is it for? What should I do next?" If AI can't answer all 3 → hero is not clear enough → revise.
4) Competitor Parity Test: AI sees new website alongside the BEST competitor from Step 6. "Is our website at least equal quality?" If competitor is clearly better → improve before deploy.

### 🟠 CRITICAL: Preview Experience Doesn't Tell a Story (F79)
**Affects**: Step 12, conversion rate
**Problem**: Preview has features (banner, toggle, analysis) but no emotional JOURNEY. Visitor should feel: Curiosity → Excitement → Urgency → Safety → Action.
**Required**: Redesign preview as guided experience:
1) Entry: "Wir haben Ihre Online-Präsenz analysiert" (curiosity)
2) Side-by-side: old screenshot next to new live site (excitement through contrast)
3) Improvement list: each improvement explained with before/after and WHY it matters in plain language (education)
4) Competitor context: "X von Y [Branche] in [Stadt] haben bereits [Feature]" (urgency)
5) Trust: guarantee + real team info + phone number (safety)
6) CTA: clear, simple, no-risk action (action)
Each section loads as visitor scrolls (progressive disclosure, not all at once).

### 🟠 CRITICAL: Step 5 Review Misses Conversion Signals (F80)
**Affects**: Step 5, outreach hooks, rebuild planning
**Problem**: Review measures technical quality (Lighthouse), visual quality (AI Vision), content quality (text metrics). Missing: conversion quality. Does the old website have: clear CTA? Trust signals? Mobile usability (as experience, not Lighthouse score)? Fast perceived load time?
**Required**: Add Conversion Score (0-100) to Quality Report:
- CTA presence + clarity (30%): is there a clear primary action? Above the fold? Specific wording?
- Trust signal density (25%): testimonials, ratings, certifications, years-in-business, photos of real people
- Mobile experience (25%): not Lighthouse mobile score, but: can a thumb reach the CTA? Is text readable without zooming? Does the page feel usable?
- Information scent (20%): can a visitor find what they need in <3 clicks? Is the navigation intuitive? Is key info (services, contact) immediately accessible?
Conversion Score becomes a key dimension in Step 7 (Scoring) and a primary improvement target in Step 8 (Planning).

### 🟡 IMPORTANT: Instagram Photos Not Used as Image Source (F81)
**Affects**: Steps 1, 4, 9, 10
**Problem**: A friseur with 2000 Instagram followers has professional photos on Instagram that are BETTER than anything on their website. We don't use them.
**Required**: Step 1 captures Instagram handle. Step 4 fetches last 20 Instagram post images (public API or scraping). Step 9 evaluates image quality and selects best images. Step 10 uses Instagram images (with placeholder watermark "With owner permission") before NanoBanana. After purchase, ask customer for image usage permission.

### 🟡 IMPORTANT: Google Review Texts as Premium Testimonials (F82)
**Affects**: Steps 1, 4, 9, 10
**Problem**: Google Review texts are the most AUTHENTIC testimonials available. "Herr Dr. Müller hat mir in einer schwierigen Scheidung geholfen" is better than any AI-generated or website-extracted testimonial. We capture the rating but not the text.
**Required**: Step 1 or 3: extract top 5 Google Reviews by quality (length >50 chars, 4+ stars, recency <2 years). Step 9: integrate as testimonials with "Google Bewertung" attribution. Step 10: display with Google logo for trust.

### 🟡 IMPORTANT: No Rebuild Quality Prediction Score (F83)
**Affects**: Step 7, Gate 3 decision quality
**Problem**: The scoring predicts CONVERSION LIKELIHOOD but not WEBSITE QUALITY. A lead with Score 90 (bad website, good business) may produce a THIN website if content is sparse. Operator decides at Gate 3 without knowing how good the result will be.
**Required**: Second score: Rebuild Quality Prediction (0-100) based on: content completeness (40%), image quality + quantity (25%), brand asset quality (15%), data richness – testimonials, team, prices, services detail (20%). Displayed alongside conversion score at Gate 3. Low quality prediction + high conversion score = warning: "This lead is likely to buy but the website may not be impressive enough."

### 🟡 IMPORTANT: Headline Quality Not Specified (F84)
**Affects**: Step 9
**Problem**: No specification for section headlines. "Über uns" is boring. "Seit 2008 Ihr Partner für Familienrecht" informs. "Wenn alles zusammenbricht, sind wir da" moves emotionally. Headlines are what 80% of visitors actually read.
**Required**: Add headline generation rules to polish prompt: headlines must contain a BENEFIT or EMOTIONAL HOOK, not a generic label. Industry-specific headline templates: Coaches: action/transformation focus. Lawyers: security/expertise focus. Salons: feeling/experience focus. Generate 3 headline variants per section, pick the best.

### 🟡 IMPORTANT: Competitor Best-Practices Not Used as Inspiration (F85)
**Affects**: Step 6, Step 8
**Problem**: We benchmark competitors on features (has booking: yes/no) but don't extract DESIGN INSPIRATION. One competitor has a beautiful hero, another has perfect price display, a third has great testimonial layout. We don't use any of this to improve our rebuild.
**Required**: Step 6 addition: for each competitor, AI identifies the ONE best design element: "Best element: hero section with full-bleed photo and clear CTA overlay." These best elements become design references in the rebuild plan (Step 8): "Hero inspiration: competitor A's full-bleed photo approach. Pricing inspiration: competitor C's clear table layout."

### 🔵 IMPROVEMENT: Personalized Video Tour for Preview (F86)
**Affects**: Step 12, conversion rate  
**Problem**: Preview is a website with a banner. A personalized video walkthrough would be 10x more engaging.
**Required**: Future feature: auto-generate screen recording with voiceover (TTS) walking through: old website problems → new website solutions → how to get started. Sent as link in outreach email. Would dramatically increase engagement. Evaluate: HeyGen API, Synthesia, or custom Playwright screen recording with TTS overlay.
