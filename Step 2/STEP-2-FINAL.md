# STEP 2: LEAD INTELLIGENCE & PIPELINE ROUTING
# Complete Implementation Spec v1.0

> **This is a SELF-CONTAINED spec.** Claude Code reads Step 1 + this file and builds Step 2.
> Step 2 receives leads that the operator approved at Gate 1.
> It transforms raw qualified leads into PIPELINE-READY leads with enriched data,
> an enrichment plan, outreach routing, and economic viability assessment.

---

## 1. MISSION

**Old name:** "Data Validation & Cleaning"
**New name:** "Lead Intelligence & Pipeline Routing"

**What this step does:** Transform a Gate-1-approved lead into a pipeline-ready lead by:
1. Confirming nothing has changed since scraping (freshness)
2. Mining FREE data sources (Impressum, Email guessing, phone classification)
3. Determining HOW to reach the lead (outreach channel + type)
4. Determining WHAT the pipeline needs to do for this lead (which steps, which APIs)
5. Calculating WHETHER the lead is economically viable (expected ROI)
6. Preparing STRUCTURED DATA for all downstream steps

**What this step does NOT do:**
- Call paid APIs (that's Step 3)
- Crawl the full website (that's Step 4)
- Score the website in detail (that's Step 5)
- Build anything (that's Steps 8-10)

**Cost:** €0.00-0.005 per lead (only free HTTP requests + DB queries)
**Duration:** 10-25 seconds per lead (Phase 1) + 5 seconds for batch (Phase 2)
**Filter rate:** ~5-8% suppressed (cooling period, bounce, competitor, freshness improvement, non-profit, negative ROI)

## 2. CONTEXT: WHERE STEP 2 SITS IN THE PIPELINE

```
Step 1 (Scrape+Check+Score) 
  → Gate 1 (Operator reviews, approves/rejects)
    → STEP 2 (Intelligence + Routing)  ← YOU ARE HERE
      → Step 3 (Enrichment) + Step 4 (Crawling)  [PARALLEL]
        → Steps 5-7 (Review + Competitor + Scoring)
          → Gate 3 → Steps 8-14 (Rebuild → Outreach)
```

**Step 2 is the LAST FREE EXIT.** After Step 2, paid APIs begin (Step 3: €0.08-0.25/lead, Step 4: €0.10-0.25/lead). Every lead that Step 2 lets through costs real money. Step 2 must be AGGRESSIVELY selective.

**Step 2 is the INTELLIGENCE LAYER.** It transforms raw data into actionable intelligence: Who is the decision maker? How do we reach them? What does the pipeline need to do? Is this economically viable?

## 3. WHAT STEP 2 RECEIVES (from Step 1 + Gate 1)

A lead with status `GATE1_APPROVED` has ALL Step 1 fields populated:

```
From Step 1:
  ✅ Google data (name, address, phone, rating, reviews, photos, description, attributes)
  ✅ Website URL (actual, after redirects)
  ✅ Technical signals (40+ data points: SSL, CMS, viewport, copyright, forms, booking, etc.)
  ✅ Quick Score with breakdown (functional + visual sub-scores)
  ✅ Priority tag (GOLD/SILVER/STANDARD)
  ✅ Email from website (if found)
  ✅ Social links + Instagram handle
  ✅ Screenshots (desktop + mobile, if gray-zone lead)
  ✅ Weakness arguments (personalized outreach sentences)

From Gate 1 (operator):
  ✅ Approval decision
  ⬜ Optional: operator comments
  ⬜ Optional: manual data overrides (name, email, notes)
  ⬜ Optional: category correction
  ⬜ Optional: priority override
```

## 4. WHAT STEP 2 ADDS (the DELTA)

After Step 2, the lead has EVERYTHING from Step 1 PLUS:

```
NEW from Step 2:
  🆕 decisionMakerName          // from Impressum parse
  🆕 decisionMakerEmail         // personal email (not info@)
  🆕 decisionMakerTitle         // "Dr.", "Prof.", etc.
  🆕 decisionMakerNames         // all name variants for different channels
  🆕 legalForm                  // GmbH, e.K., Einzelunternehmen, etc.
  🆕 vatRegistered              // USt-IdNr found?
  🆕 vatId                      // the actual USt-IdNr
  🆕 qualifications             // from Impressum
  🆕 memberships                // from Impressum (dvct, ICF, IHK, etc.)
  🆕 foundingYear               // from Impressum or copyright
  🆕 handelsregisterNr          // if found in Impressum
  🆕 impressumDataSource        // true = we scraped their Impressum
  🆕 missingImpressum           // true = no Impressum found (§5 TMG violation!)
  
  🆕 phoneNormalized            // E.164 format: +4989123456789
  🆕 phoneType                  // MOBILE / LANDLINE / TOLL_FREE / UNKNOWN
  🆕 whatsappPossible           // true if mobile number
  🆕 firmNameNormalized         // clean name without legal form
  🆕 firmLegalForm              // extracted legal form (GmbH, etc.)
  🆕 addressNormalized          // street, postalCode, city separated
  
  🆕 emailValidationResult      // mx_valid / smtp_valid / smtp_invalid / unverified
  🆕 emailGuessed               // email found via name@domain guessing
  🆕 emailGuessedValidation     // validation result for guessed email
  
  🆕 dataConsistency            // google vs impressum cross-validation result
  🆕 decisionMakerConfidence    // 0-1 how sure are we about the name
  
  🆕 outreachChannel            // email / instagram_dm / phone / whatsapp / letter
  🆕 outreachType               // EMAIL_PREVIEW / EMAIL_ANALYSIS_ONLY / PHONE_COLD_CALL / etc.
  🆕 outreachLanguage           // "de" (always for now)
  🆕 personalizationLevel       // 1 (basic) / 2 (standard) / 3 (deep)
  🆕 bestContactTime            // { day, hourStart, hourEnd, reason }
  🆕 outreachWave               // 1-5 (which week to send)
  🆕 outreachStaggerDays        // days to wait (for geo-cluster diversification)
  
  🆕 enrichmentPlan             // which APIs to call in Step 3, in which order
  🆕 crawlStrategy              // fast / throttled (based on analytics presence)
  🆕 pipelinePriority           // numeric priority for queue ordering
  🆕 pipelineRoute              // FULL / ANALYSIS_ONLY / PHONE_ONLY / FAST_TRACK
  🆕 skipSteps                  // [8,9,10,11,12] if not EMAIL_PREVIEW route
  
  🆕 estimatedPipelineCost      // total € for Steps 3-14
  🆕 estimatedDealValue         // expected revenue × conversion probability
  🆕 expectedRoi                // dealValue - pipelineCost
  🆕 survivalProbability        // P(lead reaches Step 14)
  
  🆕 contentReadiness           // 0-1 how much content is available for rebuild
  🆕 contentBriefData           // structured data for Steps 8-9
  🆕 priorityPages              // URLs to crawl first in Step 4
  🆕 sitemapAnalysis            // detailed sitemap breakdown
  
  🆕 ownerRespondsToReviews     // from review analysis
  🆕 likelyClaimed              // Google profile actively managed?
  🆕 seasonalBusiness           // true/false + peak/off season months
  🆕 primaryAcquisitionChannel  // google_maps / instagram / word_of_mouth / website
  🆕 likelyActiveWebContract    // someone is maintaining this website?
  🆕 reviewSentiment            // NORMAL / CONCERNING / CRISIS
  🆕 isNonProfit                // e.V., gemeinnützig, etc.
  🆕 possibleFranchise          // detected franchise/chain pattern
  🆕 relatedDomains             // other domains linked from the website
  
  🆕 confidenceTracker          // confidence scores per step
  🆕 manualOverrides            // operator corrections (initialized empty)
  🆕 lifecycleTracker           // pipeline position tracking (initialized)
  🆕 step2CompletedAt           // timestamp
  🆕 step2Incomplete            // true if Impressum scrape or other ops failed
  🆕 step2QualityIssues         // detected issues with Step 2's own data
  
  🆕 campaignInsights           // from batch phase: avg score, top weaknesses, percentile
  🆕 geoClusterResolved         // cluster assignments updated with names
  🆕 templateVariantHint        // for template diversification
  🆕 scorePercentile            // where this lead ranks within the campaign
```

## 5. DATABASE SCHEMA EXTENSIONS (Prisma)

These fields are ADDED to the existing Lead model from Step 1:

```prisma
// ============================================
// ADDITIONS TO Lead MODEL (Step 2 fields)
// ============================================

model Lead {
  // ... all Step 1 fields remain unchanged ...

  // === DECISION MAKER (from Impressum) ===
  decisionMakerName          String?
  decisionMakerEmail         String?          // personal email, not info@
  decisionMakerTitle         String?          // "Dr.", "Prof."
  decisionMakerNames         Json?            // { official, google, firstName, lastName, title, usedInOutreach }
  decisionMakerConfidence    Float?           // 0-1
  
  // === LEGAL / BUSINESS DETAILS (from Impressum) ===
  legalForm                  String?          // "GmbH", "e.K.", "Einzelunternehmen"
  vatRegistered              Boolean?
  vatId                      String?          // "DE123456789"
  handelsregisterNr          String?          // "HRB 12345"
  impressumQualifications    String[]         @default([])
  impressumMemberships       String[]         @default([])
  foundingYear               Int?
  impressumDataSource        Boolean          @default(false)
  missingImpressum           Boolean          @default(false)
  
  // === NORMALIZED DATA ===
  phoneNormalized            String?
  phoneType                  PhoneType?
  whatsappPossible           Boolean          @default(false)
  firmNameNormalized         String?
  firmLegalForm              String?
  
  // === EMAIL VALIDATION ===
  emailValidationResult      EmailValidation?
  emailGuessed               String?
  emailGuessedValidation     EmailValidation?
  
  // === DATA QUALITY ===
  dataConsistency            DataConsistency?
  step2QualityIssues         String[]         @default([])
  
  // === OUTREACH ROUTING ===
  outreachChannel            OutreachChannel?
  outreachType               OutreachType?
  outreachLanguage           String           @default("de")
  personalizationLevel       Int?             // 1, 2, or 3
  bestContactTime            Json?            // { day, hourStart, hourEnd, reason }
  outreachWave               Int?             // 1-5
  outreachStaggerDays        Int              @default(0)
  
  // === PIPELINE ROUTING ===
  enrichmentPlan             Json?            // waterfall of API sources per data gap
  crawlStrategy              CrawlStrategy    @default(FAST)
  pipelinePriority           Float?           // higher = process first
  pipelineRoute              PipelineRoute?
  skipSteps                  Int[]            @default([])
  
  // === ECONOMICS ===
  estimatedPipelineCost      Float?           // in euros
  estimatedDealValue         Float?           // expected revenue × conversion probability
  expectedRoi                Float?           // dealValue - pipelineCost
  survivalProbability        Float?           // 0-1
  
  // === CONTENT PREPARATION ===
  contentReadiness           Float?           // 0-1
  contentBriefData           Json?            // structured data for Steps 8-9
  priorityPages              String[]         @default([])
  sitemapAnalysis            Json?
  
  // === BUSINESS INTELLIGENCE ===
  ownerRespondsToReviews     Boolean          @default(false)
  likelyClaimed              Boolean?
  seasonalBusiness           Boolean          @default(false)
  peakSeasonMonths           Int[]            @default([])
  offSeasonMonths            Int[]            @default([])
  primaryAcquisitionChannel  AcquisitionChannel?
  likelyActiveWebContract    Boolean          @default(false)
  reviewSentiment            ReviewSentiment  @default(NORMAL)
  isNonProfit                Boolean          @default(false)
  possibleFranchise          Boolean          @default(false)
  relatedDomains             String[]         @default([])
  
  // === PIPELINE TRACKING ===
  confidenceTracker          Json?            // { step1: {}, step2: {}, ... }
  manualOverrides            Json?            // operator corrections
  lifecycleTracker           Json?            // { currentStep, stepHistory[], totalCost, etc. }
  
  // === CAMPAIGN INSIGHTS (from batch phase) ===
  campaignInsights           Json?            // { avgScore, topWeaknesses, percentile }
  scorePercentile            Float?           // 0-100 where this lead ranks
  templateVariantHint        String?
  
  // === STEP 2 META ===
  step2CompletedAt           DateTime?
  step2Incomplete            Boolean          @default(false)
  
  // === INDEXES ===
  @@index([outreachChannel])
  @@index([outreachWave])
  @@index([pipelineRoute])
  @@index([pipelinePriority])
}

// ============================================
// NEW TABLES FOR STEP 2
// ============================================

model SuppressionLog {
  id                  String            @id @default(uuid())
  leadId              String
  lead                Lead              @relation(fields: [leadId], references: [id])
  reason              SuppressionReason
  detail              String            // human-readable explanation
  previousCampaignId  String?           // which campaign contacted before
  previousContactDate DateTime?
  previousOutcome     String?           // "no_reply", "bounced", "opted_out"
  createdAt           DateTime          @default(now())
  
  @@index([leadId])
}

model OutreachHistory {
  id                  String            @id @default(uuid())
  domain              String            // normalized domain
  email               String?
  lastContactedAt     DateTime
  campaignId          String
  outcome             String            // "no_reply", "opened", "replied", "bounced", "opted_out", "converted"
  sequenceCompleted   Boolean           @default(false)
  createdAt           DateTime          @default(now())
  
  @@index([domain])
  @@index([email])
  @@index([lastContactedAt])
}

model EmailDomainIntelligence {
  id                  String            @id @default(uuid())
  domain              String            @unique
  totalSent           Int               @default(0)
  bounceCount         Int               @default(0)
  spamCount           Int               @default(0)
  bounceRate          Float             @default(0)
  spamRate            Float             @default(0)
  recommendation      EmailDomainRec    @default(PROCEED)
  updatedAt           DateTime          @updatedAt
  
  @@index([recommendation])
}

// ============================================
// NEW ENUMS FOR STEP 2
// ============================================

enum PhoneType {
  MOBILE
  LANDLINE
  TOLL_FREE
  PREMIUM
  UNKNOWN
}

enum EmailValidation {
  MX_VALID              // domain has MX record
  SMTP_VALID            // SMTP says address exists
  SMTP_INVALID          // SMTP says address does NOT exist
  CATCH_ALL             // domain accepts all addresses (can't verify)
  UNVERIFIED            // couldn't determine
}

enum DataConsistency {
  HIGH                  // Google + Impressum match
  MEDIUM                // partial match or some data missing
  LOW                   // significant mismatches
}

enum OutreachChannel {
  EMAIL
  INSTAGRAM_DM
  PHONE
  WHATSAPP
  LETTER
  MANUAL_SALES
}

enum OutreachType {
  EMAIL_PREVIEW         // full pipeline: build website + send preview link
  EMAIL_ANALYSIS_ONLY   // partial pipeline: only analysis report, no build
  PHONE_COLD_CALL       // minimal pipeline: talking points + manual call
  INSTAGRAM_DM          // partial pipeline: screenshots + DM
  PHYSICAL_LETTER       // full pipeline: PDF version for print
  MANUAL_SALES          // fast-track: operator handles personally
}

enum CrawlStrategy {
  FAST                  // normal speed (30 pages in 5 min)
  THROTTLED             // slow (30 pages in 30 min, for analytics-aware sites)
  SKIP                  // don't crawl (single-page site, use Step 1 data only)
}

enum PipelineRoute {
  FULL                  // all 17 steps
  ANALYSIS_ONLY         // steps 1-7 + 13-14 (no build/deploy)
  PHONE_ONLY            // steps 1-7 + 13 (talking points, manual call)
  FAST_TRACK            // skip to step 13 (operator handles)
  NURTURE               // only analysis email, no preview
}

enum SuppressionReason {
  COOLING_PERIOD        // contacted within last 12 months
  EMAIL_BOUNCED         // email bounced previously
  SPAM_COMPLAINT        // lead complained about our email
  OPTED_OUT             // lead opted out
  COMPETITOR            // lead is a webdesign/marketing agency
  NON_PROFIT            // e.V., gemeinnützig (no budget)
  BUSINESS_CRISIS       // negative review spike
  WEBSITE_IMPROVED      // website improved since scraping
  NEGATIVE_ROI          // expected ROI is negative
  FRANCHISE             // part of a chain (central decision)
}

enum AcquisitionChannel {
  GOOGLE_MAPS           // customers find via Google Maps directly
  INSTAGRAM             // customers come from Instagram
  WORD_OF_MOUTH         // referral-based, no online marketing
  WEBSITE               // website is active customer channel
  MIXED                 // multiple channels
}

enum ReviewSentiment {
  NORMAL
  CONCERNING            // 1+ negative keywords in recent reviews
  CRISIS                // 2+ negative reviews with strong keywords
}

enum EmailDomainRec {
  PROCEED
  USE_ALTERNATIVE
  AVOID
}
```

---

## 6. WORKER ARCHITECTURE

> **Note:** Step 2 introduces new `LeadStatus` values (`STEP2_PROCESSING`, `STEP2_FAILED`, `READY_FOR_ENRICHMENT`, `SUPPRESSED`). See **Section 23** for the complete combined `LeadStatus` enum that must be in `schema.prisma`, and **Section 24** for queue name reconciliation.

```
┌──────────────────────────────────────────────────────────┐
│  Gate 1: Operator approves lead(s)                       │
│  → Event: lead.gate1_approved { leadId }                 │
│  → OR: Bulk approve → multiple events                    │
└──────────────────────┬───────────────────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────────────────┐
│  Step 2 Worker: "step2-queue"                            │
│  Concurrency: 5 (max 5 leads simultaneously)             │
│                                                          │
│  PHASE 1 (per lead, parallel):                           │
│    2.1 Suppression Checks        [BLOCKING]              │
│    2.2 Freshness Check           [BLOCKING]              │
│    2.3 Impressum Scrape          [OPTIONAL]              │
│    2.4 Data Normalization        [ALWAYS]                │
│    2.5 Email Validation+Guessing [OPTIONAL]              │
│    2.6 Intelligence Calculation  [ALWAYS]                 │
│    2.7 Enrichment Plan+Routing   [ALWAYS]                │
│                                                          │
│  After ALL leads of a campaign finish Phase 1:           │
│                                                          │
│  PHASE 2 (batch, per campaign):                          │
│    2.8 Geo-Cluster Resolution                            │
│    2.9 Template Diversification                          │
│    2.10 Outreach Wave Assignment                         │
│    2.11 Campaign Insights Calculation                    │
│    2.12 Pipeline Forecast Generation                     │
│    2.13 Cross-Campaign Dedup                             │
│                                                          │
│  Output: lead.status = READY_FOR_ENRICHMENT              │
│  Events: lead.step2_completed, campaign.step2_completed  │
└──────────────────────────────────────────────────────────┘
                       │
                       ▼ (PARALLEL dispatch)
          ┌────────────┴────────────┐
          ▼                         ▼
   Step 3 Queue              Step 4 Queue
   (Enrichment)              (Crawling)
```

### Queue Configuration

```typescript
import { Queue, Worker } from 'bullmq';

export const step2Queue = new Queue('step2-queue', {
  connection: redis,
  defaultJobOptions: {
    attempts: 3,
    backoff: { type: 'exponential', delay: 30000 },
    removeOnComplete: { count: 200 },
    removeOnFail: { count: 100 },
  },
});

// Job types:
interface Step2LeadJob {
  type: 'process_lead';
  leadId: string;
  campaignId: string;
}

interface Step2BatchJob {
  type: 'batch_campaign';
  campaignId: string;
  leadIds: string[]; // all leads that passed Phase 1
}
```

---

## 7. THE FLOW: PHASE 1 (Per-Lead Operations)

### 7.1 Suppression Checks [BLOCKING]

```typescript
async function runSuppressionChecks(lead: Lead): Promise<SuppressionResult | null> {
  
  // CHECK 1: Cooling Period (contacted in last 12 months)
  const previousOutreach = await prisma.outreachHistory.findFirst({
    where: {
      domain: normalizeDomain(lead.actualUrl),
      lastContactedAt: { gt: new Date(Date.now() - 365 * 24 * 60 * 60 * 1000) },
    },
    orderBy: { lastContactedAt: 'desc' },
  });
  
  if (previousOutreach) {
    await logSuppression(lead.id, 'COOLING_PERIOD', 
      `Kontaktiert am ${previousOutreach.lastContactedAt.toISOString().split('T')[0]} ` +
      `in Kampagne ${previousOutreach.campaignId}. Ergebnis: ${previousOutreach.outcome}. ` +
      `Cooling-Period bis ${addMonths(previousOutreach.lastContactedAt, 12).toISOString().split('T')[0]}.`
    );
    return { status: 'SUPPRESSED', reason: 'COOLING_PERIOD' };
  }
  
  // CHECK 2: Email Bounce History
  if (lead.emailFoundOnWebsite) {
    const bounced = await prisma.outreachHistory.findFirst({
      where: { email: lead.emailFoundOnWebsite, outcome: 'bounced' },
    });
    if (bounced) {
      // Don't suppress the lead entirely – just remove the email
      await prisma.lead.update({
        where: { id: lead.id },
        data: { emailFoundOnWebsite: null, emailValidationResult: 'SMTP_INVALID' },
      });
      logger.info({ leadId: lead.id, email: lead.emailFoundOnWebsite }, 'Bounced email removed');
      // Continue processing – lead might have other contact methods
    }
  }
  
  // CHECK 3: Spam Complaint / Opt-Out
  const complaint = await prisma.outreachHistory.findFirst({
    where: {
      domain: normalizeDomain(lead.actualUrl),
      outcome: { in: ['opted_out', 'spam_complaint'] },
    },
  });
  if (complaint) {
    // PERMANENT suppression – also add to blacklist
    await prisma.blacklist.upsert({
      where: { domain: normalizeDomain(lead.actualUrl) },
      update: { reason: `spam_complaint_${complaint.outcome}` },
      create: { domain: normalizeDomain(lead.actualUrl), reason: `spam_complaint_${complaint.outcome}` },
    });
    await logSuppression(lead.id, complaint.outcome === 'opted_out' ? 'OPTED_OUT' : 'SPAM_COMPLAINT',
      `Lead hat am ${complaint.createdAt.toISOString().split('T')[0]} ${complaint.outcome === 'opted_out' ? 'Opt-Out geklickt' : 'Spam-Beschwerde eingereicht'}. Permanent gesperrt.`
    );
    return { status: 'SUPPRESSED', reason: complaint.outcome === 'opted_out' ? 'OPTED_OUT' : 'SPAM_COMPLAINT' };
  }
  
  // CHECK 4: Competitor Detection (webdesign/marketing agencies)
  const COMPETITOR_KEYWORDS = [
    'webdesign', 'web design', 'webagentur', 'digitalagentur', 'digital agentur',
    'werbeagentur', 'medienagentur', 'kreativagentur', 'internetagentur',
    'seo agentur', 'online marketing', 'social media agentur',
    'webentwicklung', 'app entwicklung', 'softwareentwicklung',
  ];
  const COMPETITOR_CATEGORIES = [
    'web_designer', 'marketing_agency', 'advertising_agency',
    'software_company', 'internet_marketing_service',
  ];
  
  const nameLC = lead.firmName.toLowerCase();
  const isCompetitor = 
    COMPETITOR_KEYWORDS.some(kw => nameLC.includes(kw)) ||
    COMPETITOR_CATEGORIES.includes(lead.googleCategory?.toLowerCase() ?? '');
  
  if (isCompetitor) {
    await logSuppression(lead.id, 'COMPETITOR',
      `Firmenname "${lead.firmName}" enthält Competitor-Keyword oder Google-Kategorie ist "${lead.googleCategory}". ` +
      `Webdesign-/Marketing-Agenturen sind unsere Konkurrenten und werden nicht kontaktiert.`
    );
    return { status: 'SUPPRESSED', reason: 'COMPETITOR' };
  }
  
  // CHECK 5: Non-Profit Detection
  const NON_PROFIT_PATTERNS = [
    /\be\.?\s*V\.?\b/i,
    /gemeinnützig/i,
    /\bStiftung\b/i,
    /\bKirchengemeinde\b/i,
    /\bPfarrei\b/i,
    /\bDiakonie\b/i,
    /\bCaritas\b/i,
  ];
  const NON_PROFIT_CATEGORIES = [
    'non_profit_organization', 'church', 'community_center',
    'charitable_organization', 'religious_organization',
  ];
  
  const isNonProfit = 
    NON_PROFIT_PATTERNS.some(p => p.test(lead.firmName)) ||
    NON_PROFIT_CATEGORIES.includes(lead.googleCategory?.toLowerCase() ?? '');
  
  if (isNonProfit) {
    await logSuppression(lead.id, 'NON_PROFIT',
      `"${lead.firmName}" ist ein Verein/gemeinnützige Organisation. ` +
      `Typischerweise kein Budget für €990+ Websites.`
    );
    return { status: 'SUPPRESSED', reason: 'NON_PROFIT' };
  }
  
  // CHECK 6: Review Sentiment (business in crisis)
  const reviews = (lead.googleReviewTexts as any[]) ?? [];
  const NEGATIVE_KEYWORDS = [
    'abzocke', 'betrug', 'warnung', 'anzeige', 'polizei',
    'gesundheitsamt', 'hygiene', 'ekel', 'gefährlich', 'skandal',
    'pfusch', 'katastrophe', 'nie wieder', 'finger weg', 'vorsicht',
  ];
  
  const recentNegative = reviews.filter(r => {
    const isRecent = new Date(r.date) > new Date(Date.now() - 90 * 24 * 60 * 60 * 1000);
    const isNegative = r.rating <= 2;
    const hasKeyword = NEGATIVE_KEYWORDS.some(kw => (r.text ?? '').toLowerCase().includes(kw));
    return isRecent && isNegative && hasKeyword;
  });
  
  if (recentNegative.length >= 2) {
    await logSuppression(lead.id, 'BUSINESS_CRISIS',
      `${recentNegative.length} negative Reviews mit Alarm-Keywords in den letzten 90 Tagen. ` +
      `Business befindet sich möglicherweise in einer Krise. Re-Check in 3 Monaten.`
    );
    return { status: 'SUPPRESSED', reason: 'BUSINESS_CRISIS' };
  }
  
  // All checks passed
  return null;
}

async function logSuppression(leadId: string, reason: SuppressionReason, detail: string): Promise<void> {
  await prisma.suppressionLog.create({
    data: { leadId, reason, detail },
  });
  await prisma.lead.update({
    where: { id: leadId },
    data: { status: 'SUPPRESSED', filteredReason: reason },
  });
  await emitEvent('lead.suppressed', { leadId, reason, detail });
  logger.info({ leadId, reason }, 'Lead suppressed in Step 2');
}
```

### 7.2 Freshness Check [BLOCKING if website improved]

```typescript
async function runFreshnessCheck(lead: Lead): Promise<FreshnessResult> {
  const daysSinceScraping = (Date.now() - lead.scrapedAt.getTime()) / (1000 * 60 * 60 * 24);
  
  // Skip if scraped less than 3 days ago (too recent for changes)
  if (daysSinceScraping < 3) {
    return { status: 'fresh', changed: false, skipped: true };
  }
  
  try {
    // Light check: HTTP HEAD only
    const response = await fetch(lead.actualUrl!, {
      method: 'HEAD',
      headers: { 'User-Agent': BROWSER_USER_AGENT },
      redirect: 'follow',
      signal: AbortSignal.timeout(5000),
    });
    
    // Website DOWN since scraping?
    if (!response.ok) {
      return { 
        status: 'down', 
        changed: true, 
        updates: { websiteFreshnessConfirmed: false },
        alert: `Website returned HTTP ${response.status} (was 200 at scraping time)`,
      };
    }
    
    // Check Last-Modified header
    const lastModified = response.headers.get('last-modified');
    if (lastModified) {
      const modifiedDate = new Date(lastModified);
      if (modifiedDate > lead.scrapedAt) {
        // Website was modified AFTER scraping → needs re-check
        return await fullFreshnessRecheck(lead);
      }
    }
    
    // Check Content-Length change (>30% = significant change)
    const contentLength = parseInt(response.headers.get('content-length') ?? '0');
    if (lead.homepageWordCount && contentLength > 0) {
      // Rough heuristic: word count × 6 ≈ byte count for German HTML
      const expectedBytes = (lead.homepageWordCount ?? 0) * 6;
      if (expectedBytes > 0 && Math.abs(contentLength - expectedBytes) / expectedBytes > 0.3) {
        return await fullFreshnessRecheck(lead);
      }
    }
    
    // Check SSL status change
    const nowHttps = response.url.startsWith('https://');
    if (!lead.hasSsl && nowHttps) {
      // Lead ADDED SSL since scraping → one weakness removed
      // Don't block, but update the data
      return { 
        status: 'improved_partially', 
        changed: true,
        updates: { hasSsl: true, sslStatus: 'VALID' },
      };
    }
    
    // Check redirect changed
    if (normalizeDomain(response.url) !== normalizeDomain(lead.actualUrl!)) {
      // Domain redirect changed → might be a completely new website
      return await fullFreshnessRecheck(lead);
    }
    
    // For leads scraped >14 days ago: do a FULL re-check regardless
    if (daysSinceScraping > 14) {
      return await fullFreshnessRecheck(lead);
    }
    
    // Everything looks the same
    return { status: 'fresh', changed: false, updates: { websiteFreshnessConfirmed: true } };
    
  } catch (error) {
    // Website unreachable → flag but don't block (might be temporary)
    return {
      status: 'unreachable',
      changed: true,
      updates: { websiteFreshnessConfirmed: false },
      alert: `Website unreachable during freshness check: ${error}`,
    };
  }
}

async function fullFreshnessRecheck(lead: Lead): Promise<FreshnessResult> {
  // Full HTTP GET + reparse key signals
  try {
    const response = await fetch(lead.actualUrl!, {
      headers: { 'User-Agent': BROWSER_USER_AGENT, 'Accept-Language': 'de-DE,de;q=0.9' },
      signal: AbortSignal.timeout(5000),
    });
    
    if (!response.ok) {
      return { status: 'down', changed: true, updates: { websiteFreshnessConfirmed: false } };
    }
    
    const html = await response.text();
    const $ = cheerio.load(html);
    
    // Check for Under Construction (NEW since scraping)
    const bodyTextLC = $('body').text().toLowerCase();
    const CONSTRUCTION_PATTERNS = [
      'coming soon', 'under construction', 'im aufbau', 'wird gerade überarbeitet',
      'relaunch', 'neue webseite', 'wir arbeiten daran',
    ];
    const isNowUnderConstruction = CONSTRUCTION_PATTERNS.some(p => bodyTextLC.includes(p));
    
    if (!lead.isUnderConstruction && isNowUnderConstruction) {
      // Website went from live → under construction SINCE scraping
      // This means: someone is working on a new website RIGHT NOW
      await logSuppression(lead.id, 'WEBSITE_IMPROVED',
        `Website zeigt seit dem Scraping "Under Construction" / "Coming Soon". ` +
        `Der Inhaber hat möglicherweise bereits einen Webdesigner beauftragt. ` +
        `Re-Check in 30 Tagen (falls der Relaunch scheitert).`
      );
      return { status: 'relaunch_detected', changed: true, blocking: true };
    }
    
    // Reparse Quick Score key signals
    const newCopyrightYear = extractCopyrightYear($);
    const newHasBooking = detectBookingWidget(html);
    const newHasTelLink = !!($('a[href^="tel:"]').length);
    const newHasForm = !!($('form').length);
    
    // Recalculate quick score with new signals
    // (simplified: check if score would drop significantly)
    let improvementSignals = 0;
    if (!lead.hasBookingWidget && newHasBooking) improvementSignals++;
    if (!lead.hasTelLink && newHasTelLink) improvementSignals++;
    if (!lead.hasContactForm && newHasForm) improvementSignals++;
    if (newCopyrightYear && newCopyrightYear > (lead.copyrightYear ?? 0)) improvementSignals++;
    
    if (improvementSignals >= 2) {
      // Website has significantly improved → might not be a good candidate anymore
      // Don't auto-suppress, but recalculate score and flag
      const newEstimatedScore = lead.quickScore! - (improvementSignals * 8); // rough reduction
      if (newEstimatedScore < 35) {
        await logSuppression(lead.id, 'WEBSITE_IMPROVED',
          `Website hat sich seit dem Scraping signifikant verbessert: ` +
          `${improvementSignals} neue Features (${newHasBooking ? 'Booking, ' : ''}${newHasTelLink ? 'Tel-Link, ' : ''}${newHasForm ? 'Formular, ' : ''}). ` +
          `Geschätzter neuer Score: ${newEstimatedScore} (unter Schwelle 40).`
        );
        return { status: 'improved_beyond_threshold', changed: true, blocking: true };
      }
    }
    
    return { 
      status: 'changed_but_still_bad', 
      changed: true, 
      updates: {
        websiteFreshnessConfirmed: true,
        copyrightYear: newCopyrightYear ?? lead.copyrightYear,
        hasBookingWidget: newHasBooking,
        hasTelLink: newHasTelLink,
        hasContactForm: newHasForm,
      },
    };
    
  } catch (error) {
    return { status: 'recheck_failed', changed: false, updates: { websiteFreshnessConfirmed: false } };
  }
}
```

### 7.3 Impressum Scrape [OPTIONAL – graceful failure]

```typescript
const IMPRESSUM_LINK_PATTERNS = [
  'a[href*="impressum"]',
  'a[href*="imprint"]',
  'a[href*="legal"]',
  'a[href*="rechtlich"]',
];

const FALLBACK_LINK_PATTERNS = [
  'a[href*="kontakt"]',
  'a[href*="contact"]',
  'a[href*="about"]',
  'a[href*="ueber-uns"]',
  'a[href*="ueber"]',
  'a[href*="datenschutz"]',
  'a[href*="privacy"]',
];

const OWNER_NAME_PATTERNS = [
  /Inhaber(?:in)?:?\s*(.+?)(?:\n|<br|<\/p|<\/div|,\s*(?:Straße|Str\.|PLZ|\d{5}))/i,
  /Geschäftsführer(?:in)?:?\s*(.+?)(?:\n|<br|<\/p|<\/div|,)/i,
  /Geschäftsführung:?\s*(.+?)(?:\n|<br|<\/p|<\/div|,)/i,
  /Vertretungsberechtigt(?:er?)?:?\s*(.+?)(?:\n|<br|<\/p|<\/div)/i,
  /Verantwortlich(?:\s+(?:gemäß|nach|i\.?\s*S\.?\s*d\.?))?\s*(?:§\s*\d+[^:]*)?:?\s*(.+?)(?:\n|<br|<\/p|<\/div)/i,
  /betrieben\s+von\s+(.+?)(?:\.|,|\n|<br)/i,
  /Angaben\s+gemäß\s+§\s*5\s+TMG:?\s*\n?\s*(.+?)(?:\n|<br)/i,
];

const EMAIL_PATTERN = /([a-zA-Z0-9._%+\-]+@[a-zA-Z0-9.\-]+\.[a-zA-Z]{2,})/;

const VAT_PATTERNS = [
  /USt(?:\.?\-?)?Id(?:\.?\-?)?(?:Nr|Nummer)?(?:\.?)?:?\s*(DE\s?\d{9})/i,
  /Umsatzsteuer[\-\s]*Identifikations[\-\s]*(?:nummer)?:?\s*(DE\s?\d{9})/i,
  /VAT[\s\-]*(?:ID|Number)?:?\s*(DE\s?\d{9})/i,
];

const HANDELSREGISTER_PATTERNS = [
  /(?:Handelsregister|Registergericht|Register[\-\s]*Nr)[:\s]*(?:Amtsgericht\s+)?(\w[\w\s]*?)\s*(HRB|HRA)\s*(\d+)/i,
  /(HRB|HRA)\s*(\d+)/i,
];

const QUALIFICATION_PATTERNS = [
  /(?:Berufsbezeichnung|Qualifikation|Zertifizierung|Ausbildung):?\s*(.+?)(?:\n|<br|<\/p)/i,
  /(?:zertifiziert|akkreditiert|zugelassen)\s+(?:durch|von|bei)\s+(.+?)(?:\n|<br|<\/p|\.)/i,
];

const MEMBERSHIP_PATTERNS = [
  /(?:Mitglied(?:schaft)?|Mitglied\s+(?:bei|im|in|des|der)):?\s*(.+?)(?:\n|<br|<\/p)/i,
  /(?:Berufsverband|Kammer|Verband|Innungsmitglied):?\s*(.+?)(?:\n|<br|<\/p)/i,
];

const PLACEHOLDER_NAMES = [
  'max mustermann', 'erika mustermann', 'john doe', 'jane doe',
  'vorname nachname', 'ihr name', 'name des inhabers', 'muster',
  'xxx', 'platzhalter', 'todo', 'example',
];

async function scrapeImpressum(lead: Lead, cachedHomepageHtml?: string): Promise<ImpressumResult | null> {
  if (!lead.actualUrl) return null;
  
  // STEP 1: Find Impressum link on homepage
  // IMPORTANT: Reuse cached HTML from Freshness Check (Step 7.1) to avoid redundant fetch.
  // The caller should pass freshnessHtml when available. Only fetch if not cached.
  let homepageHtml: string;
  if (cachedHomepageHtml) {
    homepageHtml = cachedHomepageHtml;
  } else {
    try {
      const response = await fetch(lead.actualUrl, {
        headers: { 'User-Agent': BROWSER_USER_AGENT, 'Accept-Language': 'de-DE' },
        signal: AbortSignal.timeout(5000),
      });
      if (!response.ok) return null;
      homepageHtml = await response.text();
    } catch {
      return null;
    }
  }
  
  const $ = cheerio.load(homepageHtml);
  let impressumHtml: string | null = null;
  let impressumSource: string = 'not_found';
  
  // Try primary Impressum links
  for (const selector of IMPRESSUM_LINK_PATTERNS) {
    const link = $(selector).first();
    if (link.length) {
      const href = link.attr('href');
      if (href) {
        const impressumUrl = new URL(href, lead.actualUrl).href;
        impressumHtml = await fetchPage(impressumUrl);
        if (impressumHtml) {
          impressumSource = 'impressum_page';
          break;
        }
      }
    }
  }
  
  // Fallback: Check footer text directly
  if (!impressumHtml) {
    const footerText = $('footer').text();
    if (footerText.length > 100 && /inhaber|geschäftsführ|verantwortlich/i.test(footerText)) {
      impressumHtml = $('footer').html() ?? null;
      impressumSource = 'footer_inline';
    }
  }
  
  // Fallback: Try alternative pages (kontakt, about, datenschutz)
  if (!impressumHtml) {
    for (const selector of FALLBACK_LINK_PATTERNS) {
      const link = $(selector).first();
      if (link.length) {
        const href = link.attr('href');
        if (href) {
          const pageHtml = await fetchPage(new URL(href, lead.actualUrl).href);
          if (pageHtml && /inhaber|geschäftsführ|verantwortlich|§\s*5\s*TMG/i.test(pageHtml)) {
            impressumHtml = pageHtml;
            impressumSource = 'alternative_page';
            break;
          }
        }
      }
    }
  }
  
  // If NOTHING found → Impressum is MISSING (§5 TMG violation)
  if (!impressumHtml) {
    return {
      found: false,
      missingImpressum: true,
      source: 'not_found',
    };
  }
  
  // STEP 2: Parse Impressum HTML
  const cleanText = cheerio.load(impressumHtml).text().replace(/\s+/g, ' ').trim();
  const result: ImpressumResult = {
    found: true,
    source: impressumSource,
    missingImpressum: false,
  };
  
  // Extract owner name
  for (const pattern of OWNER_NAME_PATTERNS) {
    const match = cleanText.match(pattern);
    if (match) {
      let name = match[1].trim()
        .replace(/<[^>]+>/g, '')        // remove any remaining HTML
        .replace(/\s+/g, ' ')           // normalize whitespace
        .replace(/[,;].*$/, '')         // remove everything after comma
        .trim();
      
      // Validate: not a placeholder
      if (PLACEHOLDER_NAMES.some(p => name.toLowerCase().includes(p))) {
        result.qualityIssues = [...(result.qualityIssues ?? []), 'owner_name_is_placeholder'];
        continue;
      }
      
      // Validate: looks like a real name (2-4 words, starts with uppercase)
      const words = name.split(/\s+/);
      if (words.length >= 2 && words.length <= 5 && /^[A-ZÄÖÜ]/.test(words[0])) {
        result.ownerName = name;
        result.firstName = words[0].replace(/^(Dr\.|Prof\.|Dipl\.[\-\w]*)\s*/i, '');
        result.lastName = words[words.length - 1];
        result.title = name.match(/^((?:Dr|Prof|Dipl)\.[\-\w\s]*)/)?.[1] ?? null;
        break;
      }
    }
  }
  
  // Extract email (personal, not info@)
  const allEmails = cleanText.match(new RegExp(EMAIL_PATTERN.source, 'gi')) ?? [];
  const websiteDomain = normalizeDomain(lead.actualUrl!);
  
  for (const email of allEmails) {
    const emailLC = email.toLowerCase();
    const emailDomain = emailLC.split('@')[1];
    
    // Prefer emails on the same domain as the website
    if (emailDomain === websiteDomain || emailDomain.includes(websiteDomain.split('.')[0])) {
      // Skip generic addresses
      if (/^(info|kontakt|office|hello|mail|service|support|team|post)@/i.test(emailLC)) {
        if (!result.genericEmail) result.genericEmail = emailLC;
        continue;
      }
      result.personalEmail = emailLC;
      break;
    }
  }
  // If no personal email found but generic exists → use generic as fallback
  if (!result.personalEmail && result.genericEmail) {
    result.personalEmail = result.genericEmail;
    result.qualityIssues = [...(result.qualityIssues ?? []), 'only_generic_email'];
  }
  
  // Extract VAT ID
  for (const pattern of VAT_PATTERNS) {
    const match = cleanText.match(pattern);
    if (match) {
      const vatId = match[1].replace(/\s/g, '');
      if (/^DE\d{9}$/.test(vatId)) {
        result.vatId = vatId;
        result.vatRegistered = true;
      }
      break;
    }
  }
  
  // Extract Handelsregister
  for (const pattern of HANDELSREGISTER_PATTERNS) {
    const match = cleanText.match(pattern);
    if (match) {
      result.handelsregisterNr = match[0].trim();
      break;
    }
  }
  
  // Extract qualifications
  for (const pattern of QUALIFICATION_PATTERNS) {
    const match = cleanText.match(pattern);
    if (match) {
      result.qualifications = match[1].split(/[,;]/).map(q => q.trim()).filter(Boolean);
      break;
    }
  }
  
  // Extract memberships
  for (const pattern of MEMBERSHIP_PATTERNS) {
    const match = cleanText.match(pattern);
    if (match) {
      result.memberships = match[1].split(/[,;]/).map(m => m.trim()).filter(Boolean);
      break;
    }
  }
  
  // Extract legal form (from Impressum text, more reliable than from firm name)
  result.legalForm = extractLegalForm(cleanText, lead.firmName);
  
  return result;
}

async function fetchPage(url: string): Promise<string | null> {
  try {
    const response = await fetch(url, {
      headers: { 'User-Agent': BROWSER_USER_AGENT, 'Accept-Language': 'de-DE' },
      signal: AbortSignal.timeout(5000),
    });
    if (!response.ok) return null;
    const contentType = response.headers.get('content-type') ?? '';
    if (!contentType.includes('text/html')) return null;
    return await response.text();
  } catch {
    return null;
  }
}
```

### 7.4 Data Normalization [ALWAYS – no external calls]

```typescript
function normalizeLeadData(lead: Lead, impressum: ImpressumResult | null): NormalizedData {
  // Phone normalization
  const phoneResult = lead.phone ? normalizePhone(lead.phone) : null;
  
  // Firm name normalization
  const nameResult = normalizeBusinessName(lead.firmName);
  
  // Determine founding year (best source wins)
  const foundingYear = impressum?.foundingYear 
    ?? lead.copyrightYear 
    ?? null;
  
  // Legal form (Impressum > firm name inference)
  const legalForm = impressum?.legalForm 
    ?? nameResult.legalForm 
    ?? null;
  
  return {
    phoneNormalized: phoneResult?.normalized ?? null,
    phoneType: phoneResult?.type ?? 'UNKNOWN',
    whatsappPossible: phoneResult?.type === 'MOBILE',
    firmNameNormalized: nameResult.name,
    firmLegalForm: legalForm,
    foundingYear,
  };
}

function normalizePhone(raw: string): { normalized: string; type: PhoneType } {
  let digits = raw.replace(/[^\d+]/g, '');
  
  if (digits.startsWith('0049')) digits = '+49' + digits.substring(4);
  if (digits.startsWith('49') && !digits.startsWith('+')) digits = '+' + digits;
  if (digits.startsWith('0') && !digits.startsWith('+')) digits = '+49' + digits.substring(1);
  digits = digits.replace(/\+49\(?0\)?/, '+49');
  
  const national = digits.replace('+49', '');
  let type: PhoneType = 'UNKNOWN';
  if (/^1[567]/.test(national)) type = 'MOBILE';
  else if (/^800/.test(national)) type = 'TOLL_FREE';
  else if (/^900/.test(national)) type = 'PREMIUM';
  else if (/^[2-9]/.test(national)) type = 'LANDLINE';
  
  return { normalized: digits, type };
}

function normalizeBusinessName(raw: string): { name: string; legalForm: string | null } {
  const LEGAL_FORMS = [
    'GmbH & Co. KG', 'GmbH & Co. OHG', 'PartGmbB', 'PartG mbB',
    'UG (haftungsbeschränkt)', 'UG haftungsbeschränkt',
    'GmbH', 'gGmbH', 'UG', 'AG', 'KG', 'OHG', 'GbR',
    'e.K.', 'e.Kfm.', 'e.Kfr.', 'PartG', 'KGaA', 'e.V.', 'eG',
    'Inh.', 'mbH',
  ];
  
  let name = raw.trim();
  let legalForm: string | null = null;
  
  for (const form of LEGAL_FORMS) {
    const escaped = form.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
    const regex = new RegExp(`\\s*${escaped}\\s*$`, 'i');
    if (regex.test(name)) {
      legalForm = form;
      name = name.replace(regex, '').trim();
      break;
    }
  }
  
  name = name.replace(/[,.\-–—]+$/, '').trim();
  return { name, legalForm };
}

function extractLegalForm(impressumText: string, firmName: string): string | null {
  const combined = `${firmName} ${impressumText}`;
  
  if (/\bGmbH\b/i.test(combined) && !/\bUG\b/i.test(combined)) return 'GmbH';
  if (/\bUG\s*\(?haftungsbeschränkt\)?/i.test(combined)) return 'UG';
  if (/\bAG\b/.test(combined)) return 'AG';
  if (/\be\.?\s*K\.?\b/i.test(combined)) return 'e.K.';
  if (/\bOHG\b/i.test(combined)) return 'OHG';
  if (/\bKG\b/i.test(combined) && !/\bGmbH\s*&\s*Co/i.test(combined)) return 'KG';
  if (/\bGbR\b/i.test(combined)) return 'GbR';
  if (/\bPartG\b/i.test(combined)) return 'PartG';
  if (/\be\.?\s*V\.?\b/i.test(combined)) return 'e.V.';
  if (/\bInh(?:aber)?\.?\s/i.test(combined)) return 'Einzelunternehmen';
  if (/\bFreiberufler/i.test(combined)) return 'Freiberufler';
  
  return null;
}
```

### 7.5 Email Validation + Guessing [OPTIONAL]

```typescript
import { Resolver } from 'node:dns/promises';
import * as net from 'node:net';

const resolver = new Resolver();

async function validateAndGuessEmail(
  lead: Lead, 
  impressum: ImpressumResult | null
): Promise<EmailResult> {
  const result: EmailResult = {};
  
  // Determine the best email we have
  const candidateEmail = 
    impressum?.personalEmail ??           // Impressum personal email (best)
    lead.emailFoundOnWebsite ??           // Website email from Step 1
    null;
  
  // STEP 1: Validate the candidate email (MX + SMTP)
  if (candidateEmail) {
    result.primaryEmail = candidateEmail;
    result.primaryValidation = await validateEmail(candidateEmail);
    
    // Check email domain intelligence (historical bounce/spam data)
    const emailDomain = candidateEmail.split('@')[1];
    const domainIntel = await prisma.emailDomainIntelligence.findUnique({
      where: { domain: emailDomain },
    });
    if (domainIntel?.recommendation === 'AVOID') {
      result.domainAvoid = true;
      result.domainAvoidReason = `Historical bounce rate: ${(domainIntel.bounceRate * 100).toFixed(0)}%`;
    }
  }
  
  // STEP 2: If no personal email found, try GUESSING
  if (!impressum?.personalEmail && impressum?.ownerName && lead.actualUrl) {
    const domain = normalizeDomain(lead.actualUrl);
    const firstName = impressum.firstName?.toLowerCase()
      .normalize('NFD').replace(/[\u0300-\u036f]/g, '') // remove umlauts: ü→u, ö→o
      .replace(/ß/g, 'ss') ?? '';
    const lastName = impressum.lastName?.toLowerCase()
      .normalize('NFD').replace(/[\u0300-\u036f]/g, '')
      .replace(/ß/g, 'ss') ?? '';
    
    if (firstName && lastName) {
      const guesses = [
        `${firstName}@${domain}`,
        `${firstName}.${lastName}@${domain}`,
        `${firstName[0]}.${lastName}@${domain}`,
        `${lastName}@${domain}`,
        `${firstName}${lastName}@${domain}`,
        `${firstName[0]}${lastName}@${domain}`,
      ];

      // NOTE: Email guesses are validated SEQUENTIALLY (not parallel) because:
      // 1. We break on first SMTP_VALID or CATCH_ALL — no point checking further
      // 2. Multiple simultaneous SMTP connections to the same MX host look suspicious
      // 3. At 5 concurrent leads × 6 guesses = 30 SMTP connections without rate limiting
      // Use the same rate limiter pattern as haikuRateLimiter (see shared/rate-limiter.ts).
      // Consider SMTP connection pooling per MX host for efficiency.
      for (const guess of guesses) {
        const validation = await validateEmail(guess);
        if (validation === 'SMTP_VALID') {
          result.guessedEmail = guess;
          result.guessedValidation = 'SMTP_VALID';
          break;
        }
        if (validation === 'CATCH_ALL') {
          // Can't verify, but plausible → save as candidate
          result.guessedEmail = guess;
          result.guessedValidation = 'CATCH_ALL';
          break; // first guess with catch-all is most likely correct
        }
      }
    }
  }
  
  return result;
}

async function validateEmail(email: string): Promise<EmailValidation> {
  const domain = email.split('@')[1];
  
  // MX Record check
  try {
    const mxRecords = await resolver.resolveMx(domain);
    if (!mxRecords || mxRecords.length === 0) return 'UNVERIFIED';
    
    // Sort by priority (lowest = highest priority)
    mxRecords.sort((a, b) => a.priority - b.priority);
    const mxHost = mxRecords[0].exchange;
    
    // SMTP Verification (RCPT TO check)
    try {
      const smtpResult = await smtpVerify(email, mxHost);
      return smtpResult;
    } catch {
      return 'MX_VALID'; // MX exists but SMTP check failed → at least MX valid
    }
  } catch {
    return 'UNVERIFIED';
  }
}

// NOTE: Direct SMTP on port 25 is blocked by most cloud providers (AWS, GCP, Azure).
// For production, use an email verification service (NeverBounce API — key already in env vars).
// Keep SMTP as development/self-hosted fallback only.
// Control via EMAIL_VERIFICATION_MODE env var: 'smtp_direct' | 'neverbounce' | 'mx_only'
//
// When EMAIL_VERIFICATION_MODE === 'neverbounce':
//   Replace smtpVerify() call with NeverBounce single-check API
//   Cost: ~$0.008/verification (logged via logApiCost)
// When EMAIL_VERIFICATION_MODE === 'mx_only':
//   Skip SMTP entirely, return 'MX_VALID' if MX records exist
function smtpVerify(email: string, mxHost: string): Promise<EmailValidation> {
  return new Promise((resolve, reject) => {
    const socket = net.createConnection(25, mxHost);
    socket.setTimeout(10000);
    
    let step = 0;
    let response = '';
    
    socket.on('data', (data) => {
      response += data.toString();
      
      if (step === 0 && response.includes('220')) {
        socket.write(`EHLO webolution.de\r\n`);
        step = 1;
        response = '';
      } else if (step === 1 && response.includes('250')) {
        socket.write(`MAIL FROM:<check@webolution.de>\r\n`);
        step = 2;
        response = '';
      } else if (step === 2 && response.includes('250')) {
        socket.write(`RCPT TO:<${email}>\r\n`);
        step = 3;
        response = '';
      } else if (step === 3) {
        socket.write(`QUIT\r\n`);
        socket.destroy();
        
        if (response.includes('250')) {
          resolve('SMTP_VALID');
        } else if (response.includes('550') || response.includes('553') || response.includes('511')) {
          resolve('SMTP_INVALID');
        } else if (response.includes('252') || response.includes('451')) {
          resolve('CATCH_ALL'); // Server won't confirm but accepts all
        } else {
          resolve('MX_VALID');
        }
      }
    });
    
    socket.on('timeout', () => { socket.destroy(); resolve('MX_VALID'); });
    socket.on('error', () => { socket.destroy(); reject(new Error('SMTP connection failed')); });
  });
}
```

### 7.6 Intelligence Calculation [ALWAYS – no external calls]

```typescript
function calculateIntelligence(
  lead: Lead,
  impressum: ImpressumResult | null,
  emailResult: EmailResult | null,
  normalized: NormalizedData,
  industryProfile: IndustryProfile
): IntelligenceResult {
  
  // === DATA CONSISTENCY (Google vs Impressum) ===
  let dataConsistency: DataConsistency = 'MEDIUM';
  if (impressum?.found) {
    let matches = 0;
    let checks = 0;
    
    // Name match
    if (impressum.ownerName && lead.firmName) {
      checks++;
      const nameWords = impressum.ownerName.toLowerCase().split(/\s+/);
      if (nameWords.some(w => lead.firmName.toLowerCase().includes(w))) matches++;
    }
    // Phone match
    if (normalized.phoneNormalized && impressum?.phone) {
      checks++;
      if (normalizePhone(impressum.phone).normalized === normalized.phoneNormalized) matches++;
    }
    // Address match (just check if same city)
    if (impressum?.city && lead.city) {
      checks++;
      if (impressum.city.toLowerCase().includes(lead.city.toLowerCase())) matches++;
    }
    
    if (checks > 0) {
      const ratio = matches / checks;
      dataConsistency = ratio > 0.7 ? 'HIGH' : ratio > 0.3 ? 'MEDIUM' : 'LOW';
    }
  }
  
  // === DECISION MAKER CONFIDENCE ===
  let decisionMakerConfidence = 0;
  if (impressum?.ownerName) {
    decisionMakerConfidence = 0.7; // base: found in Impressum
    if (dataConsistency === 'HIGH') decisionMakerConfidence += 0.15; // matches Google
    if (!(impressum.qualityIssues ?? []).includes('owner_name_is_placeholder')) decisionMakerConfidence += 0.1;
    if (impressum.personalEmail) decisionMakerConfidence += 0.05; // email on same domain
  } else if (lead.firmName.match(/^[A-ZÄÖÜ][a-zäöüß]+\s+[A-ZÄÖÜ]/)) {
    // Firm name looks like a person's name (e.g., "Sandra Kowalski Coaching")
    decisionMakerConfidence = 0.4;
  }
  decisionMakerConfidence = Math.min(1, decisionMakerConfidence);
  
  // === DECISION MAKER NAME VARIANTS ===
  const decisionMakerNames: any = {};
  if (impressum?.ownerName) {
    decisionMakerNames.official = impressum.ownerName;
    decisionMakerNames.firstName = impressum.firstName;
    decisionMakerNames.lastName = impressum.lastName;
    decisionMakerNames.title = impressum.title;
  }
  decisionMakerNames.google = lead.firmName; // Google name as fallback
  
  // Determine name to use in outreach
  const addressForm = industryProfile.outreach.addressForm; // "Du" or "Sie"
  if (addressForm === 'Du' && decisionMakerNames.firstName) {
    decisionMakerNames.usedInOutreach = decisionMakerNames.firstName;
  } else if (decisionMakerNames.lastName) {
    decisionMakerNames.usedInOutreach = `${decisionMakerNames.title ? decisionMakerNames.title + ' ' : ''}${decisionMakerNames.lastName}`;
  } else {
    decisionMakerNames.usedInOutreach = lead.firmNameNormalized ?? lead.firmName;
  }
  
  // === OWNER RESPONDS TO REVIEWS ===
  const reviews = (lead.googleReviewTexts as any[]) ?? [];
  const ownerRespondsToReviews = reviews.some(r => r.ownerResponse);
  
  // === LIKELY CLAIMED (Google Profile actively managed) ===
  const likelyClaimed = (lead.googleProfileCompleteness ?? 0) > 0.6;
  
  // === SEASONAL BUSINESS ===
  const seasonalCategories: Record<string, { peak: number[]; off: number[] }> = {
    'ice_cream_shop': { peak: [4,5,6,7,8,9], off: [11,12,1,2] },
    'beer_garden': { peak: [4,5,6,7,8,9], off: [11,12,1,2,3] },
    'christmas_market': { peak: [11,12], off: [1,2,3,4,5,6,7,8,9,10] },
    'swimming_pool': { peak: [5,6,7,8], off: [10,11,12,1,2,3] },
  };
  const seasonData = seasonalCategories[lead.googleCategory ?? ''];
  const seasonalBusiness = !!seasonData;
  
  // === PRIMARY ACQUISITION CHANNEL ===
  let primaryAcquisitionChannel: AcquisitionChannel = 'MIXED';
  if (lead.googleReviewCount > 50 && !lead.hasBookingWidget) {
    primaryAcquisitionChannel = 'GOOGLE_MAPS';
  } else if (lead.instagramHandle && lead.googleReviewCount < 15) {
    primaryAcquisitionChannel = 'INSTAGRAM';
  } else if (lead.googleReviewCount < 10 && !lead.instagramHandle && !lead.hasBookingWidget) {
    primaryAcquisitionChannel = 'WORD_OF_MOUTH';
  } else if (lead.hasBookingWidget || lead.hasContactForm) {
    primaryAcquisitionChannel = 'WEBSITE';
  }
  
  // === ACTIVE WEB CONTRACT ===
  const likelyActiveWebContract = !!(
    lead.websiteBuiltByType === 'AGENCY' && 
    (lead.websiteLastActivity === 'ACTIVELY_MAINTAINED' || 
     (lead.copyrightYear && new Date().getFullYear() - lead.copyrightYear <= 1))
  );
  
  // === REVIEW SENTIMENT ===
  const negativeKeywords = ['abzocke', 'betrug', 'warnung', 'katastrophe', 'nie wieder', 'finger weg'];
  const recentNegativeKeywordReviews = reviews.filter(r => 
    r.rating <= 2 && negativeKeywords.some(kw => (r.text ?? '').toLowerCase().includes(kw))
  );
  const reviewSentiment: ReviewSentiment = 
    recentNegativeKeywordReviews.length >= 2 ? 'CRISIS' : 
    recentNegativeKeywordReviews.length >= 1 ? 'CONCERNING' : 'NORMAL';
  
  // === FRANCHISE DETECTION (basic, refined in Phase 2 batch) ===
  const possibleFranchise = !!(
    lead.firmName.match(/\b(Filiale|Standort|Studio\s+\d|Nr\.\s*\d)/i) ||
    (impressum?.ownerName && impressum.ownerName !== lead.firmName && 
     lead.address && impressum.address && lead.address !== impressum.address)
  );
  
  // === RELATED DOMAINS ===
  // Check homepage HTML for links to other domains owned by the lead
  // (Reuses homepage HTML from Impressum scrape)
  const relatedDomains: string[] = []; // populated from homepage link analysis
  
  // === MISSING IMPRESSUM (§5 TMG) ===
  const missingImpressum = !impressum?.found;
  
  // === CONTENT READINESS ===
  let contentReadiness = 0;
  if ((lead.homepageWordCount ?? 0) > 200) contentReadiness += 0.25;
  else contentReadiness += (lead.homepageWordCount ?? 0) / 800;
  if (lead.hasBlog && lead.blogStatus === 'ACTIVE') contentReadiness += 0.20;
  if (lead.googleBusinessDescription) contentReadiness += 0.15;
  if ((lead.googleReviewTexts as any[])?.length >= 3) contentReadiness += 0.15;
  if ((lead.googlePhotosCount ?? 0) >= 5) contentReadiness += 0.15;
  if (lead.instagramHandle) contentReadiness += 0.10;
  contentReadiness = Math.min(1, contentReadiness);
  
  // === PERSONALIZATION LEVEL ===
  let personalizationLevel = 1;
  if ((impressum?.ownerName || lead.firmNameNormalized) && (lead.weaknessArguments?.length ?? 0) >= 1) {
    personalizationLevel = 2;
  }
  if (impressum?.ownerName && lead.googleRating && (lead.weaknessArguments?.length ?? 0) >= 3 &&
      ((impressum?.qualifications?.length ?? 0) > 0 || (lead.strengths?.length ?? 0) > 0)) {
    personalizationLevel = 3;
  }
  
  // === BEST CONTACT TIME ===
  const profileSendTime = industryProfile.outreach.bestSendingTime;
  const bestContactTime = {
    day: profileSendTime.days[0],
    hourStart: profileSendTime.hours.split('-')[0],
    hourEnd: profileSendTime.hours.split('-')[1],
    reason: `Industry default: ${profileSendTime.note ?? profileSendTime.days.join(', ')}`,
  };
  
  // === CONFIDENCE TRACKER ===
  let step2Confidence = 0.5; // base
  if (impressum?.found) step2Confidence += 0.2;
  if (impressum?.ownerName) step2Confidence += 0.1;
  if (emailResult?.primaryValidation === 'SMTP_VALID' || emailResult?.primaryValidation === 'MX_VALID') step2Confidence += 0.1;
  if (dataConsistency === 'HIGH') step2Confidence += 0.1;
  step2Confidence = Math.min(1, step2Confidence);
  
  const confidenceTracker = {
    step1: { score: lead.scoreConfidence === 'HIGH' ? 0.9 : lead.scoreConfidence === 'MEDIUM' ? 0.7 : 0.4 },
    step2: { 
      score: step2Confidence,
      impressum: impressum?.found ? 0.8 : 0,
      email: emailResult?.primaryValidation === 'SMTP_VALID' ? 0.95 : 
             emailResult?.primaryValidation === 'MX_VALID' ? 0.7 : 0.3,
      overall: step2Confidence,
    },
  };
  
  return {
    dataConsistency,
    decisionMakerConfidence,
    decisionMakerNames,
    ownerRespondsToReviews,
    likelyClaimed,
    seasonalBusiness,
    peakSeasonMonths: seasonData?.peak ?? [],
    offSeasonMonths: seasonData?.off ?? [],
    primaryAcquisitionChannel,
    likelyActiveWebContract,
    reviewSentiment,
    possibleFranchise,
    relatedDomains,
    missingImpressum,
    contentReadiness,
    personalizationLevel,
    bestContactTime,
    confidenceTracker,
    foundingYear: impressum?.foundingYear ?? (lead.copyrightYear ? lead.copyrightYear : null),
    whatsappPossible: normalized.whatsappPossible,
  };
}
```

### 7.7 Enrichment Plan + Pipeline Routing [ALWAYS]

```typescript
function createEnrichmentPlanAndRouting(
  lead: Lead,
  impressum: ImpressumResult | null,
  emailResult: EmailResult | null,
  intelligence: IntelligenceResult,
  industryProfile: IndustryProfile
): RoutingResult {
  
  // === OUTREACH CHANNEL ===
  const profileChannel = industryProfile.outreach.primaryChannel as OutreachChannel;
  let outreachChannel: OutreachChannel = profileChannel;
  
  // Override based on available data
  const hasEmail = !!(emailResult?.primaryEmail || emailResult?.guessedEmail || lead.emailFoundOnWebsite);
  const hasPhone = !!lead.phone;
  const hasInstagram = !!lead.instagramHandle;
  
  if (profileChannel === 'EMAIL' && !hasEmail && !hasInstagram && hasPhone) {
    outreachChannel = intelligence.whatsappPossible ? 'WHATSAPP' : 'PHONE';
  } else if (profileChannel === 'EMAIL' && !hasEmail && hasInstagram) {
    outreachChannel = 'INSTAGRAM_DM';
  } else if (profileChannel === 'EMAIL' && !hasEmail && !hasInstagram && !hasPhone) {
    outreachChannel = 'LETTER'; // last resort: physical mail
  }
  
  // Check email domain intelligence
  if (outreachChannel === 'EMAIL' && emailResult?.domainAvoid) {
    outreachChannel = hasInstagram ? 'INSTAGRAM_DM' : hasPhone ? 'PHONE' : 'LETTER';
  }
  
  // Manual override from operator
  if (lead.manualOverrides?.outreachChannel) {
    outreachChannel = lead.manualOverrides.outreachChannel;
  }
  
  // === OUTREACH TYPE (determines pipeline route) ===
  let outreachType: OutreachType = 'EMAIL_PREVIEW'; // default: full pipeline
  let pipelineRoute: PipelineRoute = 'FULL';
  let skipSteps: number[] = [];
  
  if (outreachChannel === 'PHONE' || outreachChannel === 'WHATSAPP') {
    outreachType = 'PHONE_COLD_CALL';
    pipelineRoute = 'PHONE_ONLY';
    skipSteps = [8, 9, 10, 11, 12]; // no build/deploy needed for phone
  } else if (outreachChannel === 'INSTAGRAM_DM') {
    outreachType = 'INSTAGRAM_DM';
    pipelineRoute = 'ANALYSIS_ONLY';
    skipSteps = [8, 9, 10, 11, 12]; // no full build, just screenshots + analysis
  } else if (intelligence.contentReadiness < 0.2) {
    outreachType = 'EMAIL_ANALYSIS_ONLY';
    pipelineRoute = 'NURTURE';
    skipSteps = [8, 9, 10, 11, 12]; // not enough content for a rebuild
  } else if (outreachChannel === 'LETTER') {
    outreachType = 'PHYSICAL_LETTER';
    pipelineRoute = 'FULL'; // build website + generate PDF version
  }
  
  // Manual override
  if (lead.manualOverrides?.pipelineRoute) {
    pipelineRoute = lead.manualOverrides.pipelineRoute;
  }
  
  // === ENRICHMENT PLAN (waterfall per data gap) ===
  const enrichmentPlan: EnrichmentStep[] = [];
  
  // Gap: Decision maker email
  const bestEmail = emailResult?.guessedEmail ?? emailResult?.primaryEmail ?? lead.emailFoundOnWebsite;
  const bestEmailValidation = emailResult?.guessedValidation ?? emailResult?.primaryValidation ?? 'UNVERIFIED';
  
  if (!bestEmail || bestEmailValidation === 'SMTP_INVALID') {
    // Need to find an email in Step 3
    const emailSources = industryProfile.step2?.enrichmentSources ?? [
      { source: 'linkedin', priority: 1, costCents: 2 },
      { source: 'hunter_io', priority: 2, costCents: 1 },
    ];
    enrichmentPlan.push({
      field: 'email',
      currentValue: bestEmail,
      currentQuality: bestEmailValidation,
      sources: emailSources.filter(s => 
        !(industryProfile.step2?.skipSources ?? []).includes(s.source)
      ),
    });
  }
  
  // Gap: Decision maker name
  if (!impressum?.ownerName && intelligence.decisionMakerConfidence < 0.5) {
    enrichmentPlan.push({
      field: 'decisionMakerName',
      currentValue: null,
      currentQuality: 'MISSING',
      sources: [
        { source: 'full_website_crawl', priority: 1, costCents: 0, note: 'Step 4 will find team/about page' },
        { source: 'linkedin', priority: 2, costCents: 2 },
      ],
    });
  }
  
  // Gap: Competitor websites (needed for Step 6)
  enrichmentPlan.push({
    field: 'competitors',
    currentValue: null,
    currentQuality: 'MISSING',
    sources: [
      { source: 'google_maps_similar', priority: 1, costCents: 2, note: 'Google Places "similar" API' },
    ],
  });
  
  // Industry-specific gaps
  if (industryProfile.tag === 'friseure' && !lead.hasBookingWidget) {
    enrichmentPlan.push({
      field: 'treatwellProfile',
      currentValue: null,
      currentQuality: 'MISSING',
      sources: [
        { source: 'treatwell_search', priority: 1, costCents: 0 },
        { source: 'booksy_search', priority: 2, costCents: 0 },
      ],
    });
  }
  
  // === CRAWL STRATEGY ===
  let crawlStrategy: CrawlStrategy = 'FAST';
  if (lead.hasGoogleAnalytics) crawlStrategy = 'THROTTLED';
  // Only SKIP crawling for routes that don't need website content.
  // Even 2-page sites need content extraction for FULL/ANALYSIS_ONLY routes.
  if (pipelineRoute === 'PHONE_ONLY' || pipelineRoute === 'NURTURE') {
    crawlStrategy = 'SKIP';
  }
  
  // === PIPELINE PRIORITY ===
  const pipelinePriority = 
    (lead.quickScore ?? 0) * 0.30 +
    (lead.reachabilityScore ?? 0) * 100 * 0.25 +
    ((lead.googleRating ?? 0) / 5) * 100 * 0.15 +
    (intelligence.contentReadiness * 100) * 0.15 +
    (intelligence.personalizationLevel / 3) * 100 * 0.10 +
    (lead.priorityTag === 'GOLD' ? 5 : 0);
  
  // === ECONOMICS ===
  const conversionRate = 
    intelligence.personalizationLevel === 3 ? 0.06 :
    intelligence.personalizationLevel === 2 ? 0.04 :
    0.01;
  
  // All monetary values in euro-cents until final conversion at return
  const avgDealValue =                           // expected deal value in euro-cents
    lead.estimatedRevenue === 'HIGH' ? 249_000 :  // €2,490.00 (Professional package)
    lead.estimatedRevenue === 'MEDIUM' ? 149_000 : // €1,490.00 (Standard package)
    99_000;                                         // €990.00 (Starter package)

  const estimatedPipelineCost =                  // pipeline cost in euro-cents
    pipelineRoute === 'FULL' ? 350 :           // €3.50 (Enrichment + Crawl + Build + Deploy)
    pipelineRoute === 'ANALYSIS_ONLY' ? 120 :  // €1.20 (Enrichment + Crawl)
    pipelineRoute === 'PHONE_ONLY' ? 80 :      // €0.80 (Enrichment + Crawl basics)
    pipelineRoute === 'NURTURE' ? 30 :         // €0.30 (minimal)
    350;
  
  const estimatedDealValue = Math.round(avgDealValue * conversionRate);
  const expectedRoi = estimatedDealValue - estimatedPipelineCost;
  
  // SURVIVAL PROBABILITY
  const survivalProbability = 
    (hasEmail ? 0.95 : hasPhone ? 0.70 : 0.40) *   // P(reachable)
    (lead.isSpa ? 0.75 : 0.90) *                     // P(crawl success)
    ((lead.quickScore ?? 0) > 55 ? 0.90 : (lead.quickScore ?? 0) > 45 ? 0.70 : 0.50) * // P(score pass)
    (intelligence.contentReadiness > 0.5 ? 0.85 : 0.60) * // P(build success)
    0.95;                                              // P(outreach success)
  
  // ROI CHECK (last cheap exit)
  if (expectedRoi < 0 && survivalProbability < 0.15) {
    // This lead will probably LOSE money
    return {
      suppress: true,
      suppressionReason: 'NEGATIVE_ROI',
      suppressionDetail: `Expected ROI: €${(expectedRoi / 100).toFixed(2)} (Deal: €${(estimatedDealValue / 100).toFixed(2)}, ` +
        `Pipeline: €${(estimatedPipelineCost / 100).toFixed(2)}). Survival: ${(survivalProbability * 100).toFixed(0)}%. ` +
        `Wirtschaftlich nicht tragfähig.`,
    };
  }
  
  // === PRIORITY PAGES (for Step 4 Crawling) ===
  const priorityPages: string[] = [];
  // These would be extracted from the homepage navigation in the Impressum step
  // (the homepage HTML is already loaded there)
  // For now: default priority pages
  const defaultPriorityPaths = [
    '/impressum', '/imprint', '/about', '/ueber-uns', '/ueber',
    '/leistungen', '/services', '/angebot',
    '/team', '/ueber-mich',
    '/preise', '/prices', '/pricing',
    '/kontakt', '/contact',
    '/galerie', '/gallery', '/portfolio',
    '/referenzen', '/references',
    '/blog', '/aktuelles', '/news',
  ];
  
  // === CONTENT BRIEF DATA (structured for Steps 8-9) ===
  const contentBriefData = {
    businessName: { value: lead.firmNameNormalized ?? lead.firmName, source: 'google', confidence: 0.95 },
    ownerName: { 
      value: impressum?.ownerName ?? null, 
      source: impressum?.found ? 'impressum' : 'missing', 
      confidence: intelligence.decisionMakerConfidence 
    },
    industry: { value: industryProfile.name, source: 'profile', confidence: 1.0 },
    city: { value: lead.city, source: 'google', confidence: 0.95 },
    googleRating: { value: lead.googleRating, source: 'google', confidence: 0.99 },
    reviewCount: { value: lead.googleReviewCount, source: 'google', confidence: 0.99 },
    testimonialCandidates: { 
      value: (lead.googleReviewTexts as any[])?.filter(r => r.rating >= 4 && r.text?.length > 50) ?? [],
      source: 'google', 
      confidence: 0.85 
    },
    qualifications: { 
      value: impressum?.qualifications ?? [], 
      source: impressum?.found ? 'impressum' : 'missing', 
      confidence: impressum?.qualifications?.length ? 0.8 : 0 
    },
    memberships: { 
      value: impressum?.memberships ?? [], 
      source: impressum?.found ? 'impressum' : 'missing', 
      confidence: impressum?.memberships?.length ? 0.8 : 0 
    },
    foundingYear: { 
      value: intelligence.foundingYear ?? null, 
      source: impressum?.foundingYear ? 'impressum' : lead.copyrightYear ? 'website' : 'missing',
      confidence: impressum?.foundingYear ? 0.9 : lead.copyrightYear ? 0.5 : 0 
    },
    // NOTE: contentPolish is from the FULL industry profile (not just the step2 section).
    // The IndustryProfile type must include contentPolish: { toneOfVoice, bannedPhrases, ctaPrimary }.
    // See INDUSTRY-PROFILES.md and the Step 9 (Content Polish) spec for the complete definition.
    toneOfVoice: { value: industryProfile.contentPolish.toneOfVoice, source: 'profile', confidence: 1.0 },
    bannedPhrases: { value: industryProfile.contentPolish.bannedPhrases, source: 'profile', confidence: 1.0 },
    ctaPrimary: { value: industryProfile.contentPolish.ctaPrimary, source: 'profile', confidence: 1.0 },
    existingSlogan: { value: lead.titleTag !== 'Home' ? lead.titleTag : null, source: 'website', confidence: 0.6 },
    strengths: { value: lead.strengths ?? [], source: 'step1', confidence: 0.7 },
    contentReadiness: intelligence.contentReadiness,
    dataCompleteness: Object.values(intelligence.confidenceTracker.step2 ?? {})
      .filter(v => typeof v === 'number' && v > 0.5).length / 4,
  };
  
  return {
    outreachChannel,
    outreachType,
    pipelineRoute,
    skipSteps,
    enrichmentPlan,
    crawlStrategy,
    pipelinePriority,
    estimatedPipelineCost: estimatedPipelineCost / 100, // convert cents to euros
    estimatedDealValue: estimatedDealValue / 100,
    expectedRoi: expectedRoi / 100,
    survivalProbability,
    contentBriefData,
    priorityPages,
  };
}
```

---

## 8. THE FLOW: PHASE 2 (Batch Operations)

Phase 2 runs ONCE per campaign, AFTER all leads have completed Phase 1.

```typescript
async function runBatchOperations(campaignId: string, leadIds: string[]): Promise<void> {
  const leads = await prisma.lead.findMany({
    where: { id: { in: leadIds }, status: 'READY_FOR_ENRICHMENT' },
  });
  
  if (leads.length === 0) return;
  
  // === BATCH WRITE STRATEGY ===
  // IMPORTANT: All updates in Phase 2 (sections 2.8–2.11) should use prisma.$transaction()
  // or Promise.all() with chunking (10 at a time) to avoid N+1 individual updates.
  // The code below shows individual updates for clarity, but the implementation MUST
  // collect all updates and execute them in batched transactions:
  //   const updates: { where: { id: string }, data: any }[] = [];
  //   // ... collect updates ...
  //   await prisma.$transaction(updates.map(u => prisma.lead.update(u)));
  // This reduces DB round-trips from O(n) to O(n/batch_size).

  // === 2.8 GEO-CLUSTER RESOLUTION ===
  // (Step 1 assigned geoClusterIds, now we ENRICH them with names and outreach strategy)
  const clusters = groupBy(leads.filter(l => l.geoClusterId), 'geoClusterId');
  for (const [clusterId, clusterLeads] of Object.entries(clusters)) {
    if (clusterLeads.length >= 2) {
      // Name the cluster by location
      const district = clusterLeads[0].address?.match(/\d{5}\s+(\S+)/)?.[1] ?? 'Unbekannt';
      for (const lead of clusterLeads) {
        await prisma.lead.update({
          where: { id: lead.id },
          data: {
            campaignInsights: {
              ...(lead.campaignInsights as any ?? {}),
              geoClusterName: `${clusterLeads.length} ${lead.industryTag} in ${district}`,
              geoClusterLeadNames: clusterLeads.map(l => l.firmName),
            },
          },
        });
      }
    }
  }
  
  // === 2.9 TEMPLATE DIVERSIFICATION ===
  const templateGroups = groupBy(leads.filter(l => l.templateStructureHash), 'templateStructureHash');
  for (const [hash, groupLeads] of Object.entries(templateGroups)) {
    if (groupLeads.length >= 3) {
      const variants = ['A', 'B', 'C', 'D'];
      for (let i = 0; i < groupLeads.length; i++) {
        await prisma.lead.update({
          where: { id: groupLeads[i].id },
          data: { templateVariantHint: variants[i % variants.length] },
        });
      }
    }
  }
  
  // === 2.10 OUTREACH WAVE ASSIGNMENT ===
  const sortedLeads = [...leads].sort((a, b) => (b.pipelinePriority ?? 0) - (a.pipelinePriority ?? 0));
  const LEADS_PER_WAVE = Math.max(15, Math.ceil(leads.length / 5));
  
  for (let i = 0; i < sortedLeads.length; i++) {
    const wave = Math.floor(i / LEADS_PER_WAVE) + 1;
    
    // Within a wave: stagger geo-cluster members across different days
    let staggerDays = 0;
    if (sortedLeads[i].geoClusterId) {
      const clusterMembers = sortedLeads.filter(l => 
        l.geoClusterId === sortedLeads[i].geoClusterId && sortedLeads.indexOf(l) < i
      );
      staggerDays = clusterMembers.length * 2; // 2 days between cluster members
    }
    
    await prisma.lead.update({
      where: { id: sortedLeads[i].id },
      data: { outreachWave: wave, outreachStaggerDays: staggerDays },
    });
  }
  
  // === 2.11 CAMPAIGN INSIGHTS ===
  const scores = leads.map(l => l.quickScore ?? 0);
  const avgScore = scores.reduce((a, b) => a + b, 0) / scores.length;
  
  const allWeaknesses = leads.flatMap(l => l.weaknesses ?? []);
  const weaknessCounts = countBy(allWeaknesses);
  const topWeaknesses = Object.entries(weaknessCounts)
    .sort(([, a], [, b]) => b - a)
    .slice(0, 5)
    .map(([weakness, count]) => ({ weakness, count, percentage: Math.round(count / leads.length * 100) }));
  
  for (const lead of leads) {
    const percentile = Math.round(
      scores.filter(s => s > (lead.quickScore ?? 0)).length / scores.length * 100
    );
    
    await prisma.lead.update({
      where: { id: lead.id },
      data: {
        scorePercentile: percentile,
        campaignInsights: {
          ...(lead.campaignInsights as any ?? {}),
          avgCampaignScore: Math.round(avgScore),
          topCampaignWeaknesses: topWeaknesses,
          totalLeadsInCampaign: leads.length,
        },
      },
    });
  }
  
  // === 2.12 PIPELINE FORECAST ===
  const totalEstimatedCost = leads.reduce((sum, l) => sum + (l.estimatedPipelineCost ?? 3.5), 0);
  const estimatedConversions = leads.reduce((sum, l) => sum + (l.survivalProbability ?? 0.3) * 0.06, 0);
  
  const forecast = {
    totalLeads: leads.length,
    estimatedCostEuros: Math.round(totalEstimatedCost * 100) / 100,
    estimatedConversions: Math.round(estimatedConversions * 10) / 10,
    estimatedRevenueEuros: Math.round(estimatedConversions * 1500),
    estimatedTimelineDays: Math.ceil(leads.length / 10) * 3 + 14, // rough: 10 leads/day build + 2 weeks buffer
    waves: Math.ceil(leads.length / LEADS_PER_WAVE),
    routeBreakdown: countBy(leads.map(l => l.pipelineRoute)),
  };
  
  await emitEvent('campaign.step2_completed', {
    campaignId,
    totalProcessed: leads.length,
    forecast,
  });
  
  // === 2.13 CROSS-CAMPAIGN DEDUP (across all campaigns) ===
  // Check if any approved lead has the same phone/address as leads in OTHER campaigns
  for (const lead of leads) {
    if (lead.phoneNormalized) {
      const crossMatches = await prisma.lead.findMany({
        where: {
          phoneNormalized: lead.phoneNormalized,
          id: { not: lead.id },
          campaignId: { not: campaignId },
          status: { in: ['QUALIFIED', 'GATE1_APPROVED', 'READY_FOR_ENRICHMENT'] },
        },
        select: { id: true, firmName: true, campaignId: true },
      });
      
      if (crossMatches.length > 0) {
        await prisma.lead.update({
          where: { id: lead.id },
          data: {
            possibleSameOwner: crossMatches.map(m => m.id),
            campaignInsights: {
              ...(lead.campaignInsights as any ?? {}),
              crossCampaignMatches: crossMatches.map(m => ({
                leadId: m.id,
                firmName: m.firmName,
                campaignId: m.campaignId,
              })),
            },
          },
        });
      }
    }
  }
  
  logger.info({ campaignId, leadsProcessed: leads.length, forecast }, 'Step 2 batch operations completed');
}
```

---

## 9. MAIN ORCHESTRATOR: Putting Phase 1 + Phase 2 Together

```typescript
// workers/step2.worker.ts

import { Worker, Job } from 'bullmq';

const step2Worker = new Worker('step2-queue', async (job: Job) => {
  if (job.data.type === 'process_lead') {
    await processLeadPhase1(job.data.leadId, job.data.campaignId);
  } else if (job.data.type === 'batch_campaign') {
    await runBatchOperations(job.data.campaignId, job.data.leadIds);
  }
}, { concurrency: 5, connection: redis });

// Triggered when operator approves lead(s) at Gate 1:
async function onGate1Approved(event: { leadId: string; campaignId: string }): Promise<void> {
  await step2Queue.add('process_lead', {
    type: 'process_lead',
    leadId: event.leadId,
    campaignId: event.campaignId,
  }, {
    priority: await getLeadPriority(event.leadId), // Gold=1, Silver=2, Standard=3
  });
}

// Track Phase 1 completion for batch trigger:
async function onLeadStep2Phase1Done(event: { leadId: string; campaignId: string }): Promise<void> {
  const key = `step2:phase1:${event.campaignId}`;
  const count = await redis.incr(key);
  
  // How many leads in this campaign are approved?
  const totalApproved = await prisma.lead.count({
    where: { campaignId: event.campaignId, status: { in: ['GATE1_APPROVED', 'READY_FOR_ENRICHMENT', 'SUPPRESSED'] } },
  });
  
  if (count >= totalApproved) {
    // All leads have finished Phase 1 → trigger Phase 2 batch
    const readyLeadIds = await prisma.lead.findMany({
      where: { campaignId: event.campaignId, status: 'READY_FOR_ENRICHMENT' },
      select: { id: true },
    }).then(leads => leads.map(l => l.id));
    
    await step2Queue.add('batch_campaign', {
      type: 'batch_campaign',
      campaignId: event.campaignId,
      leadIds: readyLeadIds,
    });
    
    await redis.del(key);
  }
}

// The actual Phase 1 per-lead processor:
async function processLeadPhase1(leadId: string, campaignId: string): Promise<void> {
  const lead = await prisma.lead.findUnique({ where: { id: leadId } });
  if (!lead || lead.status !== 'GATE1_APPROVED') return;
  
  const industryProfile = await loadIndustryProfile(lead.industryTag);
  const updates: Record<string, any> = {};
  
  // Update status to processing
  await prisma.lead.update({ where: { id: leadId }, data: { status: 'STEP2_PROCESSING' } });
  
  try {
    // ── 2.1 SUPPRESSION CHECKS [BLOCKING] ──
    const suppressed = await runSuppressionChecks(lead);
    if (suppressed) {
      await emitEvent('lead.step2_suppressed', { leadId, reason: suppressed.reason });
      return; // Lead already marked as SUPPRESSED in the check function
    }
    
    // ── 2.2 FRESHNESS CHECK [BLOCKING if improved] ──
    const freshness = await safeRun(() => runFreshnessCheck(lead));
    if (freshness?.blocking) return; // Lead suppressed by freshness check
    Object.assign(updates, freshness?.updates ?? {});
    
    // ── 2.3 IMPRESSUM SCRAPE [OPTIONAL] ──
    const impressum = await safeRun(() => scrapeImpressum(lead));
    if (impressum?.found) {
      updates.decisionMakerName = impressum.ownerName ?? null;
      updates.decisionMakerEmail = impressum.personalEmail ?? null;
      updates.decisionMakerTitle = impressum.title ?? null;
      updates.legalForm = impressum.legalForm ?? null;
      updates.vatRegistered = impressum.vatRegistered ?? null;
      updates.vatId = impressum.vatId ?? null;
      updates.handelsregisterNr = impressum.handelsregisterNr ?? null;
      updates.impressumQualifications = impressum.qualifications ?? [];
      updates.impressumMemberships = impressum.memberships ?? [];
      updates.impressumDataSource = true;
      updates.missingImpressum = false;
    } else {
      updates.impressumDataSource = false;
      updates.missingImpressum = impressum?.missingImpressum ?? true;
    }
    
    // ── 2.4 DATA NORMALIZATION [ALWAYS] ──
    const normalized = normalizeLeadData(lead, impressum);
    Object.assign(updates, normalized);
    
    // ── 2.5 EMAIL VALIDATION + GUESSING [OPTIONAL] ──
    const emailResult = await safeRun(() => validateAndGuessEmail(lead, impressum));
    if (emailResult) {
      updates.decisionMakerEmail = updates.decisionMakerEmail ?? emailResult.guessedEmail ?? emailResult.primaryEmail;
      updates.emailValidationResult = emailResult.primaryValidation;
      updates.emailGuessed = emailResult.guessedEmail;
      updates.emailGuessedValidation = emailResult.guessedValidation;
    }
    
    // ── 2.6 INTELLIGENCE [ALWAYS] ──
    const intelligence = calculateIntelligence(lead, impressum, emailResult, normalized, industryProfile);
    Object.assign(updates, {
      dataConsistency: intelligence.dataConsistency,
      decisionMakerConfidence: intelligence.decisionMakerConfidence,
      decisionMakerNames: intelligence.decisionMakerNames,
      ownerRespondsToReviews: intelligence.ownerRespondsToReviews,
      likelyClaimed: intelligence.likelyClaimed,
      seasonalBusiness: intelligence.seasonalBusiness,
      peakSeasonMonths: intelligence.peakSeasonMonths,
      offSeasonMonths: intelligence.offSeasonMonths,
      primaryAcquisitionChannel: intelligence.primaryAcquisitionChannel,
      likelyActiveWebContract: intelligence.likelyActiveWebContract,
      reviewSentiment: intelligence.reviewSentiment,
      possibleFranchise: intelligence.possibleFranchise,
      missingImpressum: intelligence.missingImpressum,
      contentReadiness: intelligence.contentReadiness,
      personalizationLevel: intelligence.personalizationLevel,
      bestContactTime: intelligence.bestContactTime,
      confidenceTracker: intelligence.confidenceTracker,
    });
    
    // ── 2.7 ENRICHMENT PLAN + ROUTING [ALWAYS] ──
    const routing = createEnrichmentPlanAndRouting(lead, impressum, emailResult, intelligence, industryProfile);
    
    // Check for negative ROI suppression
    if (routing.suppress) {
      await logSuppression(leadId, 'NEGATIVE_ROI', routing.suppressionDetail!);
      return;
    }
    
    Object.assign(updates, {
      outreachChannel: routing.outreachChannel,
      outreachType: routing.outreachType,
      pipelineRoute: routing.pipelineRoute,
      skipSteps: routing.skipSteps,
      enrichmentPlan: routing.enrichmentPlan,
      crawlStrategy: routing.crawlStrategy,
      pipelinePriority: routing.pipelinePriority,
      estimatedPipelineCost: routing.estimatedPipelineCost,
      estimatedDealValue: routing.estimatedDealValue,
      expectedRoi: routing.expectedRoi,
      survivalProbability: routing.survivalProbability,
      contentBriefData: routing.contentBriefData,
      priorityPages: routing.priorityPages,
      outreachLanguage: 'de',
    });
    
    // Initialize lifecycle tracker
    updates.lifecycleTracker = {
      currentStep: 2,
      currentStepName: 'Lead Intelligence & Pipeline Routing',
      stepHistory: [
        { step: 1, stepName: 'Find Businesses With Bad Websites', exitedAt: lead.scrapedAt, outcome: 'qualified' },
        { step: 'gate1', stepName: 'Operator Review', exitedAt: new Date(), outcome: 'approved' },
      ],
      totalCostCents: 0,
    };
    
    // Initialize manual overrides (empty)
    updates.manualOverrides = lead.manualOverrides ?? {};
    
    // Quality issues
    updates.step2QualityIssues = impressum?.qualityIssues ?? [];
    
    // ── SINGLE ATOMIC DB WRITE ──
    await prisma.lead.update({
      where: { id: leadId },
      data: {
        ...updates,
        status: 'READY_FOR_ENRICHMENT',
        step2CompletedAt: new Date(),
        step2Incomplete: !impressum?.found,
      },
    });
    
    // ── EMIT EVENTS ──
    await emitEvent('lead.step2_completed', {
      leadId,
      campaignId,
      outreachChannel: routing.outreachChannel,
      pipelineRoute: routing.pipelineRoute,
      decisionMakerFound: !!impressum?.ownerName,
      emailFound: !!(emailResult?.primaryEmail || emailResult?.guessedEmail),
      expectedRoi: routing.expectedRoi,
      pipelinePriority: routing.pipelinePriority,
    });
    
    // Track Phase 1 completion for batch trigger
    await onLeadStep2Phase1Done({ leadId, campaignId });
    
    // ── DISPATCH TO STEP 3 + STEP 4 (PARALLEL) ──
    // After batch operations complete, leads are dispatched
    // (see batch operations handler)
    
  } catch (error) {
    logger.error({ leadId, error: String(error) }, 'Step 2 Phase 1 failed');
    
    // Graceful degradation: push to Step 3 anyway with incomplete data
    await prisma.lead.update({
      where: { id: leadId },
      data: {
        status: 'READY_FOR_ENRICHMENT',
        step2Incomplete: true,
        step2CompletedAt: new Date(),
        step2QualityIssues: ['step2_crashed'],
      },
    });
    
    await onLeadStep2Phase1Done({ leadId, campaignId });
  }
}

async function safeRun<T>(fn: () => Promise<T>): Promise<T | null> {
  try {
    return await fn();
  } catch (error) {
    logger.warn({ error: String(error) }, 'Step 2 sub-operation failed, continuing');
    return null;
  }
}
```

### After Batch Operations: Dispatch to Steps 3 + 4

```typescript
// Called after Phase 2 batch operations complete:
async function dispatchToNextSteps(campaignId: string): Promise<void> {
  const readyLeads = await prisma.lead.findMany({
    where: { campaignId, status: 'READY_FOR_ENRICHMENT' },
    orderBy: { pipelinePriority: 'desc' },
  });
  
  for (const lead of readyLeads) {
    // Dispatch to Step 3 (Enrichment) AND Step 4 (Crawling) in PARALLEL
    await enrichmentQueue.add('enrich', { leadId: lead.id }, {
      priority: Math.round(1000 - (lead.pipelinePriority ?? 500)),
    });
    
    // Only dispatch to Step 4 if crawling is needed for this pipeline route
    if (lead.crawlStrategy !== 'SKIP') {
      await crawlQueue.add('crawl', { 
        leadId: lead.id, 
        strategy: lead.crawlStrategy,
        priorityPages: lead.priorityPages,
      }, {
        priority: Math.round(1000 - (lead.pipelinePriority ?? 500)),
      });
    }
  }
  
  logger.info({ campaignId, dispatched: readyLeads.length }, 'Leads dispatched to Step 3 + Step 4');
}
```

---

## 10. COMPLETE EXAMPLE WALKTHROUGH

One lead from GATE1_APPROVED through all Step 2 operations:

```
INPUT: Lead "Coaching Praxis Sandra K." (from Step 1, approved at Gate 1)
════════════════════════════════════════════════════════════════════════

  status: GATE1_APPROVED
  firmName: "Coaching Praxis Sandra K."
  actualUrl: "https://www.coaching-sandra.jimdofree.de"
  city: "München"
  quickScore: 63, priorityTag: SILVER
  emailFoundOnWebsite: "info@coaching-sandra.de"
  phone: "089 12345678"
  googleRating: 4.5, googleReviewCount: 23
  googleReviewTexts: [{text: "Sandra ist eine tolle Coach...", rating: 5, author: "Maria", ownerResponse: null}, ...]
  hasGoogleAnalytics: false
  copyrightYear: 2019
  websitePlatformTier: FREE_BAUKASTEN
  estimatedPageCount: 4
  scrapedAt: 2026-03-05 (4 days ago)

PHASE 1:
════════

── 2.1 SUPPRESSION CHECKS ──
  ✅ Cooling Period: Kein Eintrag in OutreachHistory für domain "coaching-sandra.jimdofree.de"
  ✅ Bounce History: Email "info@coaching-sandra.de" hat keinen Bounce-Eintrag
  ✅ Spam/Opt-Out: Kein Eintrag
  ✅ Competitor: Firmenname "Coaching Praxis Sandra K." enthält keine Competitor-Keywords
  ✅ Non-Profit: Keine Non-Profit-Patterns erkannt
  ✅ Review Sentiment: Keine negativen Keywords in Reviews → NORMAL
  → Alle Suppression Checks bestanden

── 2.2 FRESHNESS CHECK ──
  daysSinceScraping: 4 → >3 Tage → Light Check
  HTTP HEAD https://www.coaching-sandra.jimdofree.de → 200 OK
  Last-Modified: nicht vorhanden (Jimdo gibt keinen Last-Modified Header)
  Content-Length: ~18.000 bytes (Veränderung <10% → keine signifikante Änderung)
  SSL: immer noch HTTPS ✅
  Redirect: keine Änderung ✅
  → websiteFreshnessConfirmed: true

── 2.3 IMPRESSUM SCRAPE ──
  Homepage laden: GET https://www.coaching-sandra.jimdofree.de → 200 OK
  Suche nach Impressum-Link: 
    a[href*="impressum"] → GEFUNDEN: "/impressum"
  
  GET https://www.coaching-sandra.jimdofree.de/impressum → 200 OK
  Parse Impressum-Text:
    "Coaching Praxis Sandra K.
     Inhaberin: Sandra Kowalski
     Leopoldstr. 42
     80802 München
     
     Telefon: 089 12345678
     E-Mail: sandra@coaching-sandra.de
     
     Systemischer Coach (IHK-Zertifizierung)
     Mitglied im dvct (Deutscher Verband für Coaching und Training)"
  
  Extrahiert:
    ownerName: "Sandra Kowalski" ✅ (Pattern: "Inhaberin: (.*)")
    firstName: "Sandra"
    lastName: "Kowalski"
    title: null
    personalEmail: "sandra@coaching-sandra.de" ✅ (persönlich, nicht info@!)
    genericEmail: null
    legalForm: "Einzelunternehmen" (aus "Inhaberin")
    vatId: null → vatRegistered: false (wahrscheinlich Kleinunternehmer)
    handelsregisterNr: null
    qualifications: ["Systemischer Coach (IHK-Zertifizierung)"]
    memberships: ["dvct (Deutscher Verband für Coaching und Training)"]
    
  Quality Check:
    ✅ ownerName "Sandra Kowalski" ist kein Platzhalter
    ✅ Email-Domain "coaching-sandra.de" passt zur Website-Domain
    → qualityIssues: [] (keine Probleme)

── 2.4 DATA NORMALIZATION ──
  Phone: "089 12345678" → "+4989123456789", type: LANDLINE, whatsapp: false
  FirmName: "Coaching Praxis Sandra K." → name: "Coaching Praxis Sandra K.", legalForm: null
  FoundingYear: null (Impressum hat keines), Fallback: copyrightYear 2019

── 2.5 EMAIL VALIDATION ──
  Primary: "sandra@coaching-sandra.de" (from Impressum, BETTER than "info@" from Step 1)
  MX Check: coaching-sandra.de → MX record found (Jimdo mail server) ✅
  SMTP Verify: RCPT TO → 250 OK → SMTP_VALID ✅
  
  info@coaching-sandra.de (from Step 1): Demoted, sandra@ is better
  
  Email Guessing: NOT NEEDED (we already found personal email from Impressum)
  
  Email Domain Intelligence: No history for coaching-sandra.de → PROCEED

── 2.6 INTELLIGENCE ──
  dataConsistency: HIGH
    Name: "Kowalski" appears in both Impressum and firmName → match ✅
    Phone: Impressum "089 12345678" = Google "089 12345678" → match ✅
    Address: Impressum "Leopoldstr. 42, 80802 München" = Google address → match ✅
    
  decisionMakerConfidence: 0.95
    0.7 (base: from Impressum) + 0.15 (HIGH consistency) + 0.1 (not placeholder) + 0.05 (email domain match)
    → capped at 0.95
    
  decisionMakerNames:
    official: "Sandra Kowalski"
    firstName: "Sandra"
    lastName: "Kowalski"  
    title: null
    google: "Coaching Praxis Sandra K."
    usedInOutreach: "Sandra" (Industry Profile says Du-Form for coaches)
    
  ownerRespondsToReviews: false (no ownerResponse in any review)
  likelyClaimed: true (profileCompleteness 0.70 > 0.6)
  seasonalBusiness: false
  primaryAcquisitionChannel: WORD_OF_MOUTH (23 reviews, no booking widget, no Instagram)
  likelyActiveWebContract: false (FREE_BAUKASTEN, not AGENCY)
  reviewSentiment: NORMAL
  possibleFranchise: false
  missingImpressum: false (found and parsed)
  
  contentReadiness: 0.52
    homepageWordCount 180 → +0.225 (180/800)
    hasBlog: false → +0
    googleDescription: "Systemisches Coaching..." → +0.15
    googleReviews ≥3: true → +0.15
    googlePhotos 4: <5 → +0
    instagram: null → +0
    
  personalizationLevel: 3
    ✅ ownerName: "Sandra Kowalski"
    ✅ googleRating: 4.5
    ✅ weaknessArguments.length: 8 (≥3)
    ✅ qualifications: ["Systemischer Coach (IHK)"]
    
  bestContactTime: { day: "Mon", hourStart: "10:00", hourEnd: "11:30", reason: "Industry default: Coaches Mo-Mi" }

── 2.7 ENRICHMENT PLAN + ROUTING ──
  outreachChannel: EMAIL (industry default + email found ✅)
  outreachType: EMAIL_PREVIEW (full pipeline)
  pipelineRoute: FULL
  skipSteps: [] (all steps)
  
  enrichmentPlan:
    - Field: competitors → Source: google_maps_similar (€0.02)
    - (email: NOT needed, sandra@ already found + SMTP valid)
    - (decisionMakerName: NOT needed, Sandra Kowalski already found)
  
  crawlStrategy: FAST (no Google Analytics detected)
  
  pipelinePriority: 67.3
    quickScore 63 × 0.30 = 18.9
    reachabilityScore ~0.75 × 100 × 0.25 = 18.75
    googleRating 4.5/5 × 100 × 0.15 = 13.5
    contentReadiness 0.52 × 100 × 0.15 = 7.8
    personalizationLevel 3/3 × 100 × 0.10 = 10
    GOLD bonus: 0 (SILVER)
    = 68.95 → rounded 69.0

  estimatedPipelineCost: €3.50 (FULL route)
  estimatedDealValue: €1.990 × 6% = €119.40
  expectedRoi: €119.40 - €3.50 = €115.90 ✅ (positive)
  survivalProbability: 0.95 × 0.90 × 0.90 × 0.85 × 0.95 = 0.62 (62%)
  
  contentBriefData: {
    businessName: { value: "Coaching Praxis Sandra K.", source: "google", confidence: 0.95 },
    ownerName: { value: "Sandra Kowalski", source: "impressum", confidence: 0.95 },
    qualifications: { value: ["Systemischer Coach (IHK)"], source: "impressum", confidence: 0.8 },
    memberships: { value: ["dvct"], source: "impressum", confidence: 0.8 },
    testimonialCandidates: { value: [3 Google reviews with rating ≥4], source: "google", confidence: 0.85 },
    ctaPrimary: { value: "Kennenlerngespräch buchen", source: "profile", confidence: 1.0 },
    ...
  }

PHASE 2 (Batch, after all campaign leads finish Phase 1):
═════════════════════════════════════════════════════════

  This campaign has 29 leads. Sandra K. is one of them.
  
  ── 2.8 Geo-Cluster: Sandra is NOT in a cluster (no other coaches within 500m)
  ── 2.9 Template Diversification: No shared template hash → no variant needed
  ── 2.10 Outreach Wave: pipelinePriority 69.0 → ranked 5th of 29 → Wave 1 (top 15)
  ── 2.11 Campaign Insights: avgScore 58, Sandra at 63 → scorePercentile: 35 (top 35% worst)
  ── 2.12 Pipeline Forecast: 29 leads, ~€72 total cost, ~1.7 expected conversions
  ── 2.13 Cross-Campaign Dedup: No matches in other campaigns

OUTPUT: Lead in DB after Step 2
═══════════════════════════════

  status: READY_FOR_ENRICHMENT
  
  NEW DATA:
    decisionMakerName: "Sandra Kowalski"
    decisionMakerEmail: "sandra@coaching-sandra.de" (upgraded from info@!)
    legalForm: "Einzelunternehmen"
    impressumQualifications: ["Systemischer Coach (IHK-Zertifizierung)"]
    impressumMemberships: ["dvct"]
    phoneNormalized: "+4989123456789"
    phoneType: LANDLINE
    emailValidationResult: SMTP_VALID
    dataConsistency: HIGH
    decisionMakerConfidence: 0.95
    outreachChannel: EMAIL
    outreachType: EMAIL_PREVIEW
    pipelineRoute: FULL
    enrichmentPlan: [{field: "competitors", sources: [...]}]
    crawlStrategy: FAST
    pipelinePriority: 69.0
    estimatedPipelineCost: 3.50
    expectedRoi: 115.90
    personalizationLevel: 3
    outreachWave: 1
    contentReadiness: 0.52
    scorePercentile: 35
    step2CompletedAt: 2026-03-09T14:32:00Z
    step2Incomplete: false

  → Dispatched to Step 3 (Enrichment) + Step 4 (Crawling) in PARALLEL
  → Event: lead.step2_completed emitted
```

---

## 11. ERROR HANDLING

| Error | Handling | Lead Status |
|-------|----------|-------------|
| Suppression DB query fails | Log + continue (treat as not suppressed) | Continues |
| Freshness HTTP timeout | Skip freshness, set websiteFreshnessConfirmed=false | Continues |
| Impressum page 404 | Try fallback pages → if all fail: missingImpressum=true | Continues |
| Impressum HTML parse crash | Log + continue without Impressum data | Continues (step2Incomplete=true) |
| Email MX lookup fails | Set emailValidation=UNVERIFIED | Continues |
| SMTP verification hangs | 10s timeout → set emailValidation=MX_VALID | Continues |
| Intelligence calculation error | Log + use defaults | Continues |
| Enrichment plan creation fails | Log + use default plan (all sources) | Continues |
| DB write fails | 3 retries with backoff. If still fails: buffer in Redis | STEP2_FAILED → retry |
| Entire Phase 1 crashes | Graceful degradation: push lead as-is with step2Incomplete=true | READY_FOR_ENRICHMENT |
| Phase 2 batch fails | Log. Leads still have Phase 1 data → dispatch without batch insights | Dispatched anyway |

**Critical principle:** Step 2 NEVER blocks a lead permanently. If everything fails, the lead goes to Step 3 with `step2Incomplete: true` and Step 3 does the extra work itself.

---

## 12. EVENTS

```typescript
// Step 2 events
'lead.step2_started'         { leadId, campaignId }
'lead.step2_completed'       { leadId, campaignId, outreachChannel, pipelineRoute, 
                               decisionMakerFound, emailFound, expectedRoi, pipelinePriority }
'lead.step2_suppressed'      { leadId, campaignId, reason, detail }
'lead.step2_failed'          { leadId, campaignId, error, willRetry }

// Campaign-level events
'campaign.step2_completed'   { campaignId, totalProcessed, suppressed, failed, forecast }
'campaign.step2_dispatched'  { campaignId, dispatchedToStep3, dispatchedToStep4 }

// Consumed events (triggers)
'lead.gate1_approved'        → starts Phase 1 for this lead
'lead.gate1_bulk_approved'   → starts Phase 1 for all leads in batch
```

---

## 13. HANDOFF: STEP 2 → STEP 3 + STEP 4

### What Step 3 (Enrichment) Receives

```typescript
interface Step2ToStep3 {
  leadId: string;
  status: 'READY_FOR_ENRICHMENT';
  
  // GUARANTEED:
  firmName: string;
  actualUrl: string;
  industryTag: string;
  enrichmentPlan: EnrichmentStep[]; // which APIs to call, in which order
  outreachChannel: OutreachChannel;
  
  // BEST-EFFORT (may be null):
  decisionMakerName: string | null;    // if found from Impressum
  decisionMakerEmail: string | null;   // personal email, validated
  phoneNormalized: string | null;
  legalForm: string | null;
  
  // CONTEXT:
  step2Incomplete: boolean;           // if true: Step 3 needs to do more work
  pipelineRoute: PipelineRoute;       // determines which enrichment depth
  pipelinePriority: number;           // queue ordering
}
```

**Step 3 knows:** If `enrichmentPlan` is empty → all data already found in Step 2. Minimal enrichment needed. If `enrichmentPlan` has entries → follow the waterfall: try sources in order, stop when field is filled.

**Step 3 does NOT repeat:** Impressum scrape (Step 2 already did it). Email validation (Step 2 already did it). Phone normalization (Step 2 already did it).

### What Step 4 (Crawling) Receives

```typescript
interface Step2ToStep4 {
  leadId: string;
  actualUrl: string;
  industryTag: string;
  crawlStrategy: CrawlStrategy;        // FAST, THROTTLED, or SKIP
  priorityPages: string[];             // crawl these first
  estimatedPageCount: number;
  isSpa: boolean;                      // if true: use Playwright for everything
  hasGoogleAnalytics: boolean;         // if true: throttle crawl speed
}
```

**Step 4 knows:** If `crawlStrategy === 'SKIP'` → don't crawl, use Step 1 homepage data only. If `crawlStrategy === 'THROTTLED'` → spread crawl over 30 minutes. If `priorityPages` is not empty → crawl these URLs FIRST.

### Steps 3 + 4 run IN PARALLEL

```
Step 2 dispatches simultaneously:
  → Step 3 Queue: { leadId, enrichmentPlan, ... }
  → Step 4 Queue: { leadId, crawlStrategy, priorityPages, ... }

Synchronization point: Step 5 + Step 6
  → Both wait until BOTH Step 3 AND Step 4 are complete
  → (Handled by the Orchestrator from EVENTS.md)
```

---

## 14. METRICS

```
// Per campaign
step2_total_processed: integer
step2_suppressed: integer               // total suppressed
step2_suppressed_by_reason: Record<SuppressionReason, integer>
step2_freshness_removed: integer        // websites that improved
step2_passed: integer                   // leads dispatched to Step 3+4
step2_avg_duration_ms: integer
step2_impressum_success_rate: float     // how often Impressum parse worked
step2_email_found_rate: float           // how often personal email was found
step2_name_found_rate: float            // how often decision maker name was found
step2_avg_pipeline_priority: float
step2_avg_expected_roi: float
step2_route_breakdown: Record<PipelineRoute, integer> // FULL: 25, ANALYSIS_ONLY: 3, PHONE: 1

// Over time (dashboard)
suppression_rate_trend: float[]         // grows as more leads are on cooling
impressum_parse_improvement: float[]    // should improve as patterns get better
avg_enrichment_plan_cost: float[]       // should decrease as Step 2 finds more data
personalization_level_distribution: Record<1|2|3, float>  // % at each level
```

### Key Alerts

```
impressum_success_rate < 50%  → Parser needs new patterns
suppression_rate > 15%        → Market saturation or campaign overlap
freshness_removal_rate > 10%  → Pipeline too slow (leads go stale)
avg_expected_roi < €10        → Lead quality declining, adjust Step 1 thresholds
step2_avg_duration > 30s      → Performance issue
```

---

## 15. TESTING STRATEGY

### Unit Tests

```typescript
describe('normalizePhone', () => {
  it('handles standard German format', () => {
    expect(normalizePhone('089 12345678')).toEqual({ normalized: '+4989123456789', type: 'LANDLINE' });
  });
  it('handles mobile', () => {
    expect(normalizePhone('0171 1234567')).toEqual({ normalized: '+491711234567', type: 'MOBILE' });
  });
  it('handles +49(0) prefix', () => {
    expect(normalizePhone('+49(0)89/1234567')).toEqual({ normalized: '+49891234567', type: 'LANDLINE' });
  });
});

describe('normalizeBusinessName', () => {
  it('extracts GmbH', () => {
    expect(normalizeBusinessName('Firma XY GmbH')).toEqual({ name: 'Firma XY', legalForm: 'GmbH' });
  });
  it('extracts UG haftungsbeschränkt', () => {
    expect(normalizeBusinessName('Startup UG (haftungsbeschränkt)')).toEqual({ name: 'Startup', legalForm: 'UG (haftungsbeschränkt)' });
  });
  it('handles no legal form', () => {
    expect(normalizeBusinessName('Coaching Sandra')).toEqual({ name: 'Coaching Sandra', legalForm: null });
  });
});

describe('scrapeImpressum', () => {
  it('extracts owner name from standard format', () => { /* mock HTML */ });
  it('extracts email from Impressum', () => { /* ... */ });
  it('detects placeholder names', () => { /* ... */ });
  it('handles missing Impressum gracefully', () => { /* returns missingImpressum: true */ });
  it('falls back to footer when no Impressum link', () => { /* ... */ });
});

describe('suppressionChecks', () => {
  it('detects cooling period', () => { /* mock OutreachHistory */ });
  it('detects competitor keywords', () => { /* ... */ });
  it('detects non-profit (e.V.)', () => { /* ... */ });
  it('detects review crisis', () => { /* ... */ });
  it('passes clean lead', () => { /* returns null */ });
});

describe('calculateIntelligence', () => {
  it('calculates content readiness correctly', () => { /* ... */ });
  it('determines personalization level 3 with full data', () => { /* ... */ });
  it('determines personalization level 1 with minimal data', () => { /* ... */ });
  it('detects seasonal business', () => { /* ... */ });
});
```

### Integration Tests

```typescript
describe('Step 2 Full Pipeline', () => {
  it('processes a clean lead end-to-end', async () => {
    // Setup: Create lead with GATE1_APPROVED status
    // Run: processLeadPhase1()
    // Assert: status = READY_FOR_ENRICHMENT, all fields populated
  });
  
  it('suppresses a lead with cooling period', async () => {
    // Setup: Create lead + OutreachHistory record within 12 months
    // Run: processLeadPhase1()
    // Assert: status = SUPPRESSED, suppressionLog created
  });
  
  it('handles Impressum parse failure gracefully', async () => {
    // Setup: Create lead with URL that has no Impressum
    // Run: processLeadPhase1()
    // Assert: status = READY_FOR_ENRICHMENT (not blocked), step2Incomplete = true
  });
});
```

---

## 16. INDUSTRY PROFILE EXTENSIONS FOR STEP 2

Each industry profile includes a `step2` section:

```json
{
  "step2": {
    "impressumPriority": "high",
    "enrichmentSources": [
      { "source": "impressum", "priority": 1, "costCents": 0 },
      { "source": "email_guessing", "priority": 2, "costCents": 0 },
      { "source": "linkedin", "priority": 3, "costCents": 2 }
    ],
    "skipSources": ["treatwell", "myhammer"],
    "outreachPersonalization": {
      "useFirstName": true,
      "needsDecisionMakerName": true
    },
    "impressumFields": {
      "required": ["ownerName"],
      "valuable": ["email", "qualifications", "vatId"],
      "optional": ["memberships", "foundingYear", "handelsregister"]
    },
    "investmentPerField": {
      "decisionMakerName": { "maxCostCents": 0, "reason": "Coach: Du-Form, only first name needed" },
      "email": { "maxCostCents": 2, "reason": "Email is primary outreach channel" }
    },
    "suppressionOverrides": {
      "allowNonProfit": false,
      "coolingPeriodMonths": 12,
      "minReviewRating": 3.0
    },
    "verificationSources": []
  }
}
```

For Friseure (different configuration):

```json
{
  "step2": {
    "impressumPriority": "medium",
    "enrichmentSources": [
      { "source": "impressum", "priority": 1, "costCents": 0 },
      { "source": "treatwell_search", "priority": 2, "costCents": 0 },
      { "source": "booksy_search", "priority": 3, "costCents": 0 },
      { "source": "instagram_profile", "priority": 4, "costCents": 0 }
    ],
    "skipSources": ["linkedin", "northdata"],
    "outreachPersonalization": {
      "useFirstName": false,
      "needsDecisionMakerName": false
    },
    "impressumFields": {
      "required": [],
      "valuable": ["ownerName", "email"],
      "optional": ["vatId"]
    },
    "investmentPerField": {
      "decisionMakerName": { "maxCostCents": 0, "reason": "Friseur: 'Hallo [Salonname]' reicht" },
      "email": { "maxCostCents": 0, "reason": "Instagram DM as fallback" }
    },
    "suppressionOverrides": {
      "allowNonProfit": false,
      "coolingPeriodMonths": 12,
      "minReviewRating": 3.0
    },
    "verificationSources": []
  }
}
```

---

## 17. DOWNSTREAM IMPACT MAP

Every Step 2 decision has consequences. This map documents the causal chain:

```
STEP 2 DECISION                        → DOWNSTREAM IMPACT
════════════════                          ════════════════════

outreachType = EMAIL_PREVIEW            → Steps 3-14 all active (full pipeline, ~€3.50)
outreachType = PHONE_COLD_CALL          → Steps 8-12 SKIPPED (no build/deploy, ~€0.80)
outreachType = EMAIL_ANALYSIS_ONLY      → Steps 8-12 SKIPPED (analysis + report only, ~€1.20)
outreachType = INSTAGRAM_DM             → Steps 8-12 SKIPPED (screenshots + DM, ~€1.20)

enrichmentPlan.skipLinkedIn = true      → Step 3 does NOT call LinkedIn API (€0.02 saved)
enrichmentPlan = []                     → Step 3 does minimal work (only competitor search)

crawlStrategy = FAST                    → Step 4 crawls normally (30 pages / 5 min)
crawlStrategy = THROTTLED               → Step 4 crawls slowly (30 pages / 30 min)
crawlStrategy = SKIP                    → Step 4 NOT dispatched (only for PHONE_ONLY/NURTURE routes)

personalizationLevel = 3                → Step 13 generates deep-personalized email
personalizationLevel = 1                → Step 13 generates basic email

contentReadiness < 0.2                  → Step 7 routes to NURTURE (no preview built)
contentReadiness > 0.7                  → Step 8 can use PREMIUM template

outreachWave = 3                        → Step 14 sends in week 3 (not week 1)
outreachStaggerDays = 4                 → Step 14 waits 4 extra days (cluster diversification)

templateVariantHint = "B"               → Step 8 selects template variant B
bestContactTime.day = "Monday"          → Step 14 sends on Monday 10:00-11:30

missingImpressum = true                 → Step 13 adds legal argument to outreach
likelyActiveWebContract = true          → Step 13 adjusts tone (acknowledge existing agency)
primaryAcquisitionChannel = INSTAGRAM   → Step 13 uses Instagram-specific argument
reviewSentiment = CONCERNING            → Step 13 uses cautious tone
```

---

## 18. GATE 1 UX REQUIREMENTS (for Admin Backend)

Step 2 depends on Gate 1 providing clean data. The Admin Backend Gate 1 screen should support:

```
┌────────────────────────────────────────────────────────────────┐
│  GATE 1: LEAD REVIEW                                          │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  Lead: Coaching Praxis Sandra K.  │  Score: 63  │  🥈 SILVER  │
│  München-Schwabing  │  4.5★ (23 Reviews)                     │
│                                                                │
│  [Screenshot Thumbnail]  │  Top Schwächen:                    │
│                          │  • Kein Buchungsweg                │
│                          │  • Design veraltet (2019)          │
│                          │  • Baukasten erkennbar             │
│                                                                │
│  Actions:                                                      │
│  [✅ Approve]  [❌ Reject ▼]  [🔍 Quick Intel Only]           │
│                                                                │
│  Optional:                                                     │
│  Kommentar: [                                          ]      │
│  Kategorie korrigieren: [Coaches & Berater        ▼]          │
│  Manuelle Daten:                                              │
│    Name:  [                    ]                              │
│    Email: [                    ]                              │
│    Notiz: [                    ]                              │
│  Priority: ○ Normal  ○ Urgent                                 │
│                                                                │
│  Reject Reason (if rejecting):                                │
│  ○ Website doch OK                                            │
│  ○ Falsche Branche                                            │
│  ○ Schließt bald                                              │
│  ○ Kenne Inhaber persönlich                                   │
│  ○ Sonstige: [                    ]                           │
│                                                                │
│  Bulk: [Select All ≥60]  [Select All GOLD]                    │
│        [Approve Selected (15)]  [Reject Selected (3)]         │
└────────────────────────────────────────────────────────────────┘
```

**How Gate 1 data maps to `manualOverrides`:** When the operator fills in optional fields (Name, Email, Note, Priority override, Category correction), the Admin Backend saves these as the `manualOverrides` JSON field on the lead record:

```typescript
// Gate 1 approval handler saves manual data:
await prisma.lead.update({
  where: { id: leadId },
  data: {
    status: 'GATE1_APPROVED',
    manualOverrides: {
      name: operatorName ?? undefined,         // manual decision maker name
      email: operatorEmail ?? undefined,        // manual email override
      note: operatorNote ?? undefined,          // operator note for downstream steps
      priorityOverride: operatorPriority ?? undefined,  // 'URGENT' etc.
      categoryCorrection: operatorCategory ?? undefined, // corrected industry tag
      outreachChannel: undefined,               // can be set later
      pipelineRoute: undefined,                 // can be set later
    },
  },
});
```

Step 2 reads `lead.manualOverrides` to apply operator corrections (see routing logic in Section 7.7).

---

## 19. MISSING ELEMENTS (Audit Additions)

### 19.1 Pipeline-Backlog-Warnung (P20)

Step 2 checks the current pipeline backlog before dispatching:

```typescript
async function checkPipelineBacklog(): Promise<BacklogWarning | null> {
  const pendingEnrichment = await prisma.lead.count({ where: { status: 'READY_FOR_ENRICHMENT' } });
  const pendingCrawl = await prisma.lead.count({ where: { status: 'ENRICHED' } }); // waiting for crawl
  const pendingBuild = await prisma.lead.count({ where: { status: 'SCORED' } }); // waiting for build
  
  const bottleneck = Math.max(pendingEnrichment, pendingCrawl, pendingBuild);
  const bottleneckName = pendingBuild >= pendingCrawl ? 'Build (Steps 8-10)' : 
                         pendingCrawl >= pendingEnrichment ? 'Crawling (Step 4)' : 'Enrichment (Step 3)';
  
  if (bottleneck > 50) {
    const estimatedDays = Math.ceil(bottleneck / 10); // ~10 leads/day through bottleneck
    return {
      warning: `Pipeline-Backlog: ${bottleneck} Leads warten auf ${bottleneckName}. ` +
               `Geschätzte Bearbeitungszeit für neue Leads: ${estimatedDays} Tage. ` +
               `Empfehlung: Batch-Größe reduzieren oder ${bottleneckName}-Kapazität erhöhen.`,
      bottleneck: bottleneckName,
      queueDepth: bottleneck,
      estimatedDelayDays: estimatedDays,
    };
  }
  return null;
}
```

Backlog warning is included in the `campaign.step2_completed` event and shown in the Admin Dashboard.

### 19.2 Cross-Pipeline Learnings / Historical Conversion Data (P41)

Step 2 checks historical conversion rates for similar leads:

```typescript
async function getHistoricalConversionInsight(lead: Lead): Promise<ConversionInsight | null> {
  // Need at least 20 historical data points for statistical relevance
  const similar = await prisma.lead.findMany({
    where: {
      industryTag: lead.industryTag,
      quickScore: { gte: (lead.quickScore ?? 0) - 10, lte: (lead.quickScore ?? 0) + 10 },
      websitePlatformTier: lead.websitePlatformTier,
      status: { in: ['CONVERTED', 'NOT_CONVERTED', 'GATE1_REJECTED'] },
      createdAt: { gt: new Date(Date.now() - 180 * 24 * 60 * 60 * 1000) }, // last 6 months
    },
    select: { status: true },
  });
  
  if (similar.length < 20) return null; // not enough data
  
  const converted = similar.filter(l => l.status === 'CONVERTED').length;
  const conversionRate = converted / similar.length;
  
  if (conversionRate === 0) {
    return {
      warning: `Leads mit diesem Profil (${lead.industryTag}, Score ${lead.quickScore}, ` +
               `${lead.websitePlatformTier}) haben in den letzten 6 Monaten NIE konvertiert ` +
               `(0/${similar.length} ähnliche Leads). Fortfahren auf eigenes Risiko.`,
      historicalRate: 0,
      sampleSize: similar.length,
    };
  }
  
  return {
    historicalRate: conversionRate,
    sampleSize: similar.length,
  };
}
```

Result is stored in `leads.campaignInsights.historicalConversionRate` and used to refine `expectedRoi`.

### 19.3 Image/Photo Needs Estimation (P43)

```typescript
function estimateImageNeeds(lead: Lead, industryProfile: IndustryProfile): ImageNeedsEstimate {
  // How many images does a typical website need for this industry?
  const templateImageCount = industryProfile.tag === 'friseure' ? 12 :
                              industryProfile.tag === 'fotografen' ? 20 :
                              industryProfile.tag === 'coaches' ? 5 : 8;
  
  // How many do we HAVE?
  const ownPhotosAvailable = (lead.googlePhotosCount ?? 0) + 
                              (lead.aiHasRealPhotos ? 3 : 0); // rough: 3 extractable from website
  const instagramPhotos = lead.instagramHandle ? 6 : 0; // assume 6 usable if Instagram exists
  const totalAvailable = ownPhotosAvailable + instagramPhotos;
  
  const deficit = Math.max(0, templateImageCount - totalAvailable);
  const estimatedNanoBananaCost = deficit * 0.08; // ~€0.08 per AI-generated image
  
  return {
    templateNeedsImages: templateImageCount,
    ownPhotosAvailable,
    instagramPhotosEstimate: instagramPhotos,
    deficit,
    estimatedImageCostEuros: estimatedNanoBananaCost,
    strategy: deficit === 0 ? 'OWN_PHOTOS_SUFFICIENT' :
              deficit <= 3 ? 'MINOR_AI_SUPPLEMENT' :
              'SIGNIFICANT_AI_GENERATION',
  };
}
```

Stored in `leads.contentBriefData.imageNeeds`.

### 19.4 Gate 1 Undo Mechanism (P46)

```typescript
// Undo window: Lead can be un-approved as long as it hasn't entered Step 3 yet.
async function undoGate1Approval(leadId: string, operatorId: string): Promise<boolean> {
  const lead = await prisma.lead.findUnique({ where: { id: leadId } });
  
  // Can only undo if lead is still in Step 2 territory
  const undoableStatuses: LeadStatus[] = ['GATE1_APPROVED', 'STEP2_PROCESSING', 'READY_FOR_ENRICHMENT'];
  if (!lead || !undoableStatuses.includes(lead.status as LeadStatus)) {
    return false; // too late, lead already in Step 3+
  }
  
  await prisma.lead.update({
    where: { id: leadId },
    data: { status: 'QUALIFIED', filteredReason: null }, // back to Gate 1 queue
  });
  
  logger.info({ leadId, operatorId }, 'Gate 1 approval undone');
  return true;
}
```

Admin Backend shows "Undo" button next to each approved lead until it enters Step 3.

### 19.5 Datenschutzerklärung Parse (P62)

During the Impressum scrape, if we find a Datenschutz link, we optionally parse it:

```typescript
async function parseDatenschutz(homepageHtml: string, baseUrl: string): Promise<DatenschutzInfo | null> {
  const $ = cheerio.load(homepageHtml);
  const dsLink = $('a[href*="datenschutz"], a[href*="privacy"]').first().attr('href');
  if (!dsLink) return null;
  
  const dsHtml = await fetchPage(new URL(dsLink, baseUrl).href);
  if (!dsHtml) return null;
  
  const dsText = cheerio.load(dsHtml).text().toLowerCase();
  
  return {
    hostingProvider: dsText.match(/(?:hosting|gehostet|webhosting)\s*(?:bei|durch|von|:)\s*([^.;,\n]+)/i)?.[1]?.trim() ?? null,
    usesGoogleAnalytics: dsText.includes('google analytics'),
    usesGoogleMaps: dsText.includes('google maps'),
    usesFacebookPixel: dsText.includes('facebook pixel') || dsText.includes('meta pixel'),
    hasNewsletterService: dsText.includes('mailchimp') || dsText.includes('newsletter') || dsText.includes('rapidmail'),
    generatedBy: dsText.includes('erecht24') ? 'erecht24' : 
                 dsText.includes('it-recht-kanzlei') ? 'it-recht-kanzlei' : 
                 dsText.includes('datenschutz-generator') ? 'datenschutz-generator' : null,
  };
}
```

Result stored in `leads.contentBriefData.datenschutzInfo`. Useful for Step 8 (Rebuild) to know which tools the lead uses.

### 19.6 Landing Page Detection (P66)

```typescript
// Check if lead might have a separate Google Ads landing page
function detectSeparateLandingPage(lead: Lead): boolean {
  // Signal: has Google Ads tracking BUT website quality is low
  // → the "good" landing page might be somewhere else
  return !!(
    lead.hasGoogleAnalytics && 
    // gtag with AW- prefix means Google Ads conversion tracking
    // (this was detected in Step 1 HTML parse)
    lead.hasOtherTracking?.some(t => t.includes('AW-'))
  );
}
```

If detected: `leads.contentBriefData.mayHaveSeparateLandingPage = true`. Step 4 (Crawling) can search for it.

### 19.7 DSGVO Race Condition Protection (P93)

In the main orchestrator (Section 9), add a SECOND suppression check before the final DB write:

```typescript
// In processLeadPhase1(), BEFORE the final atomic write:

// ── SECOND SUPPRESSION CHECK (race condition protection) ──
const suppressedLate = await prisma.blacklist.findFirst({
  where: {
    OR: [
      { googlePlaceId: lead.googlePlaceId },
      { domain: normalizeDomain(lead.actualUrl ?? '') },
    ],
  },
});
if (suppressedLate) {
  // Someone added this lead to the blacklist WHILE we were processing
  await prisma.lead.update({
    where: { id: leadId },
    data: { status: 'SUPPRESSED', filteredReason: 'blacklisted_during_processing' },
  });
  logger.warn({ leadId }, 'Lead blacklisted during Step 2 processing (race condition caught)');
  return;
}

// Also re-check outreach history (opt-out might have come in during processing)
const lateOptOut = await prisma.outreachHistory.findFirst({
  where: {
    domain: normalizeDomain(lead.actualUrl ?? ''),
    outcome: { in: ['opted_out', 'spam_complaint'] },
    createdAt: { gt: lead.updatedAt }, // created AFTER we started processing
  },
});
if (lateOptOut) {
  await logSuppression(leadId, 'OPTED_OUT', 'Opt-out received during Step 2 processing');
  return;
}

// NOW safe to write
```

### 19.8 Lead Re-Run / Merge Mode (P110, P116)

```typescript
async function processLeadPhase1(leadId: string, campaignId: string): Promise<void> {
  const lead = await prisma.lead.findUnique({ where: { id: leadId } });
  if (!lead) return;
  
  // CHECK: Has this lead been through Step 2 before? (re-run scenario)
  if (lead.step2CompletedAt) {
    const daysSinceLastStep2 = (Date.now() - lead.step2CompletedAt.getTime()) / (1000 * 60 * 60 * 24);
    
    if (daysSinceLastStep2 < 7) {
      // Very recent → skip Step 2, use existing data
      logger.info({ leadId }, 'Skipping Step 2: completed less than 7 days ago');
      await prisma.lead.update({
        where: { id: leadId },
        data: { status: 'READY_FOR_ENRICHMENT' },
      });
      await onLeadStep2Phase1Done({ leadId, campaignId });
      return;
    }
    
    if (daysSinceLastStep2 < 90) {
      // Semi-recent → keep Impressum data (name, email, legal form), redo freshness + routing
      logger.info({ leadId }, 'Step 2 merge mode: keeping Impressum data, refreshing routing');
      // Skip sub-step 2.3 (Impressum), run everything else
      // The existing decisionMakerName, legalForm, etc. are preserved
      await runStep2MergeMode(lead, campaignId);
      return;
    }
    
    // Old data (>90 days) → full re-run
    logger.info({ leadId }, 'Step 2 full re-run: data older than 90 days');
  }
  
  // First-time or full re-run → standard processing
  await runStep2Full(lead, campaignId);
}

// Admin Backend: Batch re-run feature
async function batchReRunStep2(filter: { industryTag?: string; step2Before?: Date }): Promise<number> {
  const leads = await prisma.lead.findMany({
    where: {
      step2CompletedAt: filter.step2Before ? { lt: filter.step2Before } : null,
      industryTag: filter.industryTag,
      status: { in: ['READY_FOR_ENRICHMENT', 'ENRICHED', 'QUALIFIED'] },
    },
    select: { id: true, campaignId: true },
  });
  
  for (const lead of leads) {
    await step2Queue.add('process_lead', {
      type: 'process_lead',
      leadId: lead.id,
      campaignId: lead.campaignId,
    });
  }
  
  return leads.length;
}
```

### 19.9 State Machine / Valid Transitions (P112)

```typescript
const VALID_TRANSITIONS: Record<string, string[]> = {
  'GATE1_APPROVED': ['STEP2_PROCESSING', 'QUALIFIED'],           // QUALIFIED = undo
  'STEP2_PROCESSING': ['READY_FOR_ENRICHMENT', 'SUPPRESSED', 'FILTERED_OUT', 'STEP2_FAILED'],
  'STEP2_FAILED': ['STEP2_PROCESSING'],                          // retry
  'READY_FOR_ENRICHMENT': ['ENRICHING', 'QUALIFIED'],            // QUALIFIED = undo
  'SUPPRESSED': [],                                               // terminal (except manual override)
};

function validateStatusTransition(currentStatus: string, newStatus: string, leadId: string): boolean {
  const allowed = VALID_TRANSITIONS[currentStatus];
  if (!allowed) {
    logger.error({ leadId, currentStatus, newStatus }, 'Unknown current status');
    return false;
  }
  if (!allowed.includes(newStatus)) {
    logger.error({ leadId, currentStatus, newStatus, allowed }, 'Invalid status transition');
    return false;
  }
  return true;
}

// Use in every status update:
async function safeStatusUpdate(leadId: string, newStatus: string, data?: any): Promise<boolean> {
  const lead = await prisma.lead.findUnique({ where: { id: leadId }, select: { status: true } });
  if (!lead) return false;
  
  if (!validateStatusTransition(lead.status, newStatus, leadId)) return false;
  
  await prisma.lead.update({
    where: { id: leadId },
    data: { status: newStatus as LeadStatus, ...data },
  });
  return true;
}
```

### 19.10 Auto-Approve Rules for Inactive Operators (P96)

```typescript
// Configurable in Admin Settings:
interface AutoApproveConfig {
  enabled: boolean;
  rules: Array<{
    condition: {
      minQuickScore?: number;
      minGoogleRating?: number;
      minReviewCount?: number;
      priorityTag?: PriorityTag;
    };
    action: 'approve' | 'reject';
  }>;
  inactivityThresholdHours: number; // auto-approve after N hours without operator action
}

// Default config (disabled):
const DEFAULT_AUTO_APPROVE: AutoApproveConfig = {
  enabled: false,
  rules: [
    { condition: { minQuickScore: 75, minGoogleRating: 4.0, minReviewCount: 10 }, action: 'approve' },
    { condition: { minQuickScore: 75, priorityTag: 'GOLD' }, action: 'approve' },
  ],
  inactivityThresholdHours: 48, // if operator hasn't touched Gate 1 in 48h
};

// Cron job: check for stale Gate 1 queues
async function checkStaleGate1Queues(): Promise<void> {
  const config = await getAutoApproveConfig();
  if (!config.enabled) return;
  
  const staleLeads = await prisma.lead.findMany({
    where: {
      status: 'QUALIFIED',
      quickCheckDate: { lt: new Date(Date.now() - config.inactivityThresholdHours * 60 * 60 * 1000) },
    },
  });
  
  if (staleLeads.length === 0) return;
  
  // Alert operator first
  await emitEvent('system.gate1_stale', {
    leadCount: staleLeads.length,
    oldestLeadDate: staleLeads[0].quickCheckDate,
    message: `${staleLeads.length} Leads warten seit ${config.inactivityThresholdHours}h+ auf Gate 1 Review.`,
  });
  
  // Apply auto-approve rules
  for (const lead of staleLeads) {
    for (const rule of config.rules) {
      const matches = 
        (!rule.condition.minQuickScore || (lead.quickScore ?? 0) >= rule.condition.minQuickScore) &&
        (!rule.condition.minGoogleRating || (lead.googleRating ?? 0) >= rule.condition.minGoogleRating) &&
        (!rule.condition.minReviewCount || (lead.googleReviewCount ?? 0) >= rule.condition.minReviewCount) &&
        (!rule.condition.priorityTag || lead.priorityTag === rule.condition.priorityTag);
      
      if (matches) {
        await prisma.lead.update({
          where: { id: lead.id },
          data: { status: rule.action === 'approve' ? 'GATE1_APPROVED' : 'GATE1_REJECTED' },
        });
        if (rule.action === 'approve') {
          await step2Queue.add('process_lead', { type: 'process_lead', leadId: lead.id, campaignId: lead.campaignId });
        }
        break;
      }
    }
  }
}
```

### 19.11 Competitor Quick-Check in Step 2 (P44)

Step 1 already has `ranksForPrimaryKeyword` and `rankingPosition` from the Google Search ranking check. Step 2 uses this to assess competitive pressure MORE specifically:

```typescript
function assessCompetitivePressure(lead: Lead): CompetitivePressureAssessment {
  // From Step 1: do we know the lead's ranking position?
  const ranks = lead.ranksForPrimaryKeyword;
  const position = lead.rankingPosition;
  const density = lead.competitorDensity ?? 0;
  
  let pressure: 'HIGH' | 'MEDIUM' | 'LOW' = 'MEDIUM';
  let outreachAngle: string;
  
  if (density > 20 && !ranks) {
    pressure = 'HIGH';
    outreachAngle = `Bei ${density} ${lead.industryTag} in Ihrem Viertel entscheidet der Online-Auftritt wer die Neukunden bekommt. Sie erscheinen aktuell nicht in den Google-Ergebnissen.`;
  } else if (density > 20 && ranks && position && position > 10) {
    pressure = 'HIGH';
    outreachAngle = `Sie erscheinen auf Position ${position} bei Google – hinter ${position - 1} Wettbewerbern. Die Top 3 bekommen 70% der Klicks.`;
  } else if (density < 5) {
    pressure = 'LOW';
    outreachAngle = `Sie sind einer von nur ${density} ${lead.industryTag} in Ihrem Viertel. Mit einer professionellen Website könnten Sie den lokalen Markt dominieren.`;
  } else {
    outreachAngle = `Ihre Wettbewerber in ${lead.city} investieren in ihren Online-Auftritt. Zeit mitzuziehen.`;
  }
  
  return { pressure, outreachAngle, density, ranking: position };
}
```

Result stored in `leads.campaignInsights.competitivePressure` and `leads.weaknessArguments` gets the outreachAngle appended.

### 19.12 Google Profile Deactivated Detection (P56)

```typescript
function detectInactiveGoogleProfile(lead: Lead): boolean {
  // A deactivated/unclaimed profile typically has:
  // - No owner photos (only user photos or none)
  // - No business description
  // - Low profile completeness
  // - No responses to reviews
  return (
    (lead.googleProfileCompleteness ?? 0) < 0.3 &&
    (lead.googleOwnerPhotosCount ?? 0) === 0 &&
    !lead.googleBusinessDescription &&
    !lead.ownerRespondsToReviews
  );
}
```

If detected: `leads.likelyClaimed = false` and a note in `contentBriefData`: "Google-Profil ist wahrscheinlich nicht aktiv verwaltet. Upsell-Opportunity: Google Business Optimierung."

### 19.13 DSGVO Section for Step 2

Step 2 processes additional personal data beyond Step 1:

| Data from Step 2 | Personal? | Legal Basis | Note |
|---|---|---|---|
| Decision maker name (from Impressum) | YES | Art. 6(1)(f) + TMG §5 (Impressum is public BY LAW) | Required to be public |
| Personal email (from Impressum) | YES | Art. 6(1)(f) + TMG §5 | Part of mandatory Impressum |
| Email guessing (firstname@domain) | GRAY AREA | Art. 6(1)(f) | Generated, not scraped |
| VAT ID | NO (business data) | N/A | Tax identifier |
| Legal form | NO (business data) | N/A | Public registry data |

**Key principle:** All data Step 2 collects comes from LEGALLY MANDATED public sources (Impressum per TMG §5). The Impressum EXISTS for the purpose of enabling contact. Our processing is covered by legitimate interest (Art. 6(1)(f) DSGVO).

**Step 2 specific deletion requirement:** When a lead is deleted (DSGVO Art. 17), Step 2 data must also be cleared: `decisionMakerName`, `decisionMakerEmail`, `vatId`, `impressumQualifications`, `impressumMemberships`, `emailGuessed`. The `deleteLeadCompletely()` function from Step 1 already handles this (cascade delete on the Lead record).

### 19.14 Helper Functions Referenced But Not Defined

```typescript
function extractCity(addressComponents: any[]): string {
  // Google Places addressComponents format
  const locality = addressComponents?.find((c: any) => c.types?.includes('locality'));
  return locality?.longText ?? locality?.shortText ?? '';
}

function extractPostalCode(addressComponents: any[]): string {
  const postal = addressComponents?.find((c: any) => c.types?.includes('postal_code'));
  return postal?.longText ?? postal?.shortText ?? '';
}

function groupBy<T>(items: T[], key: keyof T): Record<string, T[]> {
  return items.reduce((acc, item) => {
    const k = String(item[key] ?? 'unknown');
    (acc[k] = acc[k] ?? []).push(item);
    return acc;
  }, {} as Record<string, T[]>);
}

function countBy<T>(items: T[]): Record<string, number> {
  return items.reduce((acc, item) => {
    const k = String(item);
    acc[k] = (acc[k] ?? 0) + 1;
    return acc;
  }, {} as Record<string, number>);
}

function addMonths(date: Date, months: number): Date {
  const result = new Date(date);
  result.setMonth(result.getMonth() + months);
  return result;
}

function extractCopyrightYear($: cheerio.CheerioAPI): number | null {
  const text = $('body').text();
  const match = text.match(/©\s*(\d{4})/);
  return match ? parseInt(match[1]) : null;
}

function detectBookingWidget(html: string): boolean {
  const htmlLC = html.toLowerCase();
  return ['calendly', 'treatwell', 'booksy', 'shore.com', 'eversports', 
          'simplybook', 'tucalendi', 'doctolib', 'acuity'].some(p => htmlLC.includes(p));
}
```

---

## 20. TYPESCRIPT INTERFACES (all types referenced in this spec)

Every interface used in Step 2 code, formally defined:

```typescript
// ============================================
// STEP 2 INPUT/OUTPUT INTERFACES
// ============================================

/** Result from Sub-Step 2.1 Suppression */
interface SuppressionResult {
  status: 'SUPPRESSED';
  reason: SuppressionReason;
}

/** Result from Sub-Step 2.2 Freshness */
interface FreshnessResult {
  status: 'fresh' | 'down' | 'changed_but_still_bad' | 'improved_partially' | 
          'improved_beyond_threshold' | 'relaunch_detected' | 'unreachable' | 'recheck_failed';
  changed: boolean;
  blocking?: boolean;            // true = lead should be removed from pipeline
  skipped?: boolean;             // true = freshness check was skipped (too recent)
  updates?: Partial<Lead>;       // fields to update on the lead record
  alert?: string;                // human-readable alert message
}

/** Result from Sub-Step 2.3 Impressum Scrape */
interface ImpressumResult {
  found: boolean;
  source: 'impressum_page' | 'footer_inline' | 'alternative_page' | 'not_found';
  missingImpressum: boolean;     // true = no Impressum at all (§5 TMG violation)
  
  // Extracted data (all nullable)
  ownerName?: string | null;
  firstName?: string | null;
  lastName?: string | null;
  title?: string | null;         // "Dr.", "Prof."
  personalEmail?: string | null; // personal, not info@
  genericEmail?: string | null;  // info@, kontakt@, etc.
  phone?: string | null;
  address?: string | null;
  city?: string | null;
  legalForm?: string | null;
  vatId?: string | null;
  vatRegistered?: boolean | null;
  handelsregisterNr?: string | null;
  qualifications?: string[];
  memberships?: string[];
  foundingYear?: number | null;
  
  // Quality
  qualityIssues?: string[];      // e.g. ['owner_name_is_placeholder', 'email_domain_mismatch']
}

/** Result from Sub-Step 2.4 Normalization */
interface NormalizedData {
  phoneNormalized: string | null;
  phoneType: PhoneType;
  whatsappPossible: boolean;
  firmNameNormalized: string;
  firmLegalForm: string | null;
  foundingYear: number | null;
}

/** Result from Sub-Step 2.5 Email Validation */
interface EmailResult {
  primaryEmail?: string | null;
  primaryValidation?: EmailValidation;
  guessedEmail?: string | null;
  guessedValidation?: EmailValidation;
  domainAvoid?: boolean;
  domainAvoidReason?: string;
}

/** Result from Sub-Step 2.6 Intelligence */
interface IntelligenceResult {
  dataConsistency: DataConsistency;
  decisionMakerConfidence: number;
  decisionMakerNames: DecisionMakerNames;
  ownerRespondsToReviews: boolean;
  likelyClaimed: boolean;
  seasonalBusiness: boolean;
  peakSeasonMonths: number[];
  offSeasonMonths: number[];
  primaryAcquisitionChannel: AcquisitionChannel;
  likelyActiveWebContract: boolean;
  reviewSentiment: ReviewSentiment;
  possibleFranchise: boolean;
  relatedDomains: string[];
  missingImpressum: boolean;
  contentReadiness: number;
  personalizationLevel: number;
  bestContactTime: ContactWindow;
  confidenceTracker: ConfidenceTracker;
  foundingYear: number | null;
  whatsappPossible: boolean;          // from normalizePhone: phoneType === 'MOBILE'
}

interface DecisionMakerNames {
  official?: string | null;      // full name from Impressum
  google?: string | null;        // name from Google listing
  firstName?: string | null;
  lastName?: string | null;
  title?: string | null;
  usedInOutreach: string;        // the name we'll actually use
}

interface ContactWindow {
  day: string;                   // "Monday", "Tuesday", etc.
  hourStart: string;             // "10:00"
  hourEnd: string;               // "11:30"
  confidence: 'HIGH' | 'MEDIUM' | 'LOW';
  reason: string;
}

interface ConfidenceTracker {
  step1: { score: number };
  step2: {
    score: number;
    impressum: number;
    email: number;
    overall: number;
  };
  step3?: { score: number } | null;
  step4?: { score: number } | null;
  step5?: { score: number } | null;
  overall?: number;
}

/** Result from Sub-Step 2.7 Routing */
interface RoutingResult {
  // If suppress is true, lead should be stopped
  suppress?: boolean;
  suppressionReason?: SuppressionReason;
  suppressionDetail?: string;
  
  // Outreach routing
  outreachChannel: OutreachChannel;
  outreachType: OutreachType;
  pipelineRoute: PipelineRoute;
  skipSteps: number[];
  
  // Enrichment
  enrichmentPlan: EnrichmentStep[];
  crawlStrategy: CrawlStrategy;
  
  // Priority
  pipelinePriority: number;
  
  // Economics
  estimatedPipelineCost: number;  // in euros
  estimatedDealValue: number;     // in euros
  expectedRoi: number;            // in euros
  survivalProbability: number;    // 0-1
  
  // Content
  contentBriefData: ContentBriefData;
  priorityPages: string[];
}

interface EnrichmentStep {
  field: string;                  // which data field to fill
  currentValue: any | null;       // what we have now
  currentQuality: string;         // quality of current value
  sources: EnrichmentSource[];    // ordered list of sources to try
}

interface EnrichmentSource {
  source: string;                 // "impressum", "linkedin", "hunter_io", etc.
  priority: number;               // 1 = try first
  costCents: number;              // cost per lookup
  note?: string;                  // implementation hint
}

interface ContentBriefData {
  businessName: ContentBriefField<string>;
  ownerName: ContentBriefField<string>;
  industry: ContentBriefField<string>;
  city: ContentBriefField<string>;
  googleRating: ContentBriefField<number>;
  reviewCount: ContentBriefField<number>;
  testimonialCandidates: ContentBriefField<TestimonialCandidate[]>;
  qualifications: ContentBriefField<string[]>;
  memberships: ContentBriefField<string[]>;
  foundingYear: ContentBriefField<number>;
  toneOfVoice: ContentBriefField<string>;
  bannedPhrases: ContentBriefField<string[]>;
  ctaPrimary: ContentBriefField<string>;
  existingSlogan: ContentBriefField<string>;
  strengths: ContentBriefField<string[]>;
  contentReadiness: number;
  dataCompleteness: number;
  imageNeeds?: ImageNeedsEstimate;
  datenschutzInfo?: DatenschutzInfo;
  mayHaveSeparateLandingPage?: boolean;
}

interface ContentBriefField<T> {
  value: T | null;
  source: 'google' | 'impressum' | 'website' | 'profile' | 'step1' | 'inferred' | 'missing';
  confidence: number;             // 0-1
}

interface TestimonialCandidate {
  text: string;
  author: string;
  rating: number;
}

interface ImageNeedsEstimate {
  templateNeedsImages: number;
  ownPhotosAvailable: number;
  instagramPhotosEstimate: number;
  deficit: number;
  estimatedImageCostEuros: number;
  strategy: 'OWN_PHOTOS_SUFFICIENT' | 'MINOR_AI_SUPPLEMENT' | 'SIGNIFICANT_AI_GENERATION';
}

interface DatenschutzInfo {
  hostingProvider: string | null;
  usesGoogleAnalytics: boolean;
  usesGoogleMaps: boolean;
  usesFacebookPixel: boolean;
  hasNewsletterService: boolean;
  generatedBy: string | null;
}

/** Pipeline backlog check */
interface BacklogWarning {
  warning: string;
  bottleneck: string;
  queueDepth: number;
  estimatedDelayDays: number;
}

/** Historical conversion insight */
interface ConversionInsight {
  warning?: string;
  historicalRate: number;
  sampleSize: number;
}

/** Competitive pressure assessment */
interface CompetitivePressureAssessment {
  pressure: 'HIGH' | 'MEDIUM' | 'LOW';
  outreachAngle: string;
  density: number;
  ranking: number | null;
}

// ============================================
// HTTP CHECK RESULT (shared with Step 1)
// ============================================

/** 
 * HttpCheckResult is defined in Step 1 (STEP-1-FINAL.md).
 * Step 2 reuses it for the Freshness Check.
 * Key fields Step 2 cares about:
 */
interface HttpCheckResult {
  httpStatus?: number;
  actualUrl?: string;
  redirectChain?: string[];
  websiteType?: string;
  hasSsl?: boolean;
  sslStatus?: string;
  hasViewportMeta?: boolean;
  copyrightYear?: number | null;
  hasTelLink?: boolean;
  hasContactForm?: boolean;
  hasBookingWidget?: boolean;
  bookingPlatform?: string | null;
  hasEcommerce?: boolean;
  hasGoogleAnalytics?: boolean;
  hasFacebookPixel?: boolean;
  hasCookieConsent?: boolean;
  baukastenPlatform?: string | null;
  websitePlatformTier?: string;
  websiteBuiltBy?: string | null;
  websiteBuiltByType?: string;
  isSpa?: boolean;
  isParked?: boolean;
  isUnderConstruction?: boolean;
  homepageWordCount?: number;
  websiteLanguage?: string;
  bodyTextLower?: string;
  status?: string;               // 'TIMEOUT', 'DNS_ERROR', 'ERROR', 'AUTH_REQUIRED', 'REDIRECT_LOOP'
  error?: string;
}
```

---

## 21. CROSS-FILE DEPENDENCIES

Step 2 imports functions and constants from Step 1. This section documents every dependency.

### From Step 1 (`services/step1/` or `packages/shared/`)

```typescript
// These are defined in STEP-1-FINAL.md and MUST be importable by Step 2:

// === FUNCTIONS ===
import { normalizeDomain } from '@webolution/shared/url';
// Normalizes "https://www.example.de/page?q=1" → "example.de"

import { detectBookingWidget } from '@webolution/shared/detection';
// Checks HTML for known booking platform patterns

import { extractCopyrightYear } from '@webolution/shared/detection';
// Extracts year from "© 2019" in HTML body text

// === CONSTANTS ===
import { BROWSER_USER_AGENT } from '@webolution/shared/constants';
// "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36..."

import { AGGREGATOR_DOMAINS } from '@webolution/shared/constants';
// ["facebook.com", "instagram.com", "treatwell.de", ...]

import { PARKING_PATTERNS } from '@webolution/shared/constants';
// ["registriert bei ionos", "this domain is for sale", ...]

import { CONSTRUCTION_PATTERNS } from '@webolution/shared/constants';
// ["coming soon", "under construction", "im aufbau", ...]
```

### Recommended Package Structure

```
packages/
  shared/
    src/
      constants.ts          ← BROWSER_USER_AGENT, AGGREGATOR_DOMAINS, PARKING_PATTERNS, etc.
      url.ts                ← normalizeDomain()
      detection.ts          ← detectBookingWidget(), extractCopyrightYear(), etc.
      phone.ts              ← normalizePhone() (used by Step 2)
      business-name.ts      ← normalizeBusinessName() (used by Step 2)
      events.ts             ← emitEvent(), event type definitions
      industry-profile.ts   ← loadIndustryProfile()
      types/
        lead.ts             ← Lead type (generated from Prisma, re-exported)
        step1.ts            ← HttpCheckResult, QuickScoreResult, etc.
        step2.ts            ← All interfaces from Section 20 above
        events.ts           ← Event payload types
        industry-profile.ts ← IndustryProfile type
  
  db/
    prisma/
      schema.prisma         ← Combined schema from Step 1 + Step 2
    src/
      client.ts             ← Prisma client export
  
  queue/
    src/
      queues.ts             ← Queue definitions (step1-queue, step2-queue, etc.)
      redis.ts              ← Redis connection
      rate-limiter.ts       ← Rate limiters (Google, Lighthouse, Haiku)

services/
  step1/
    src/
      worker.ts             ← Step 1 campaign worker
      phases/
        phase-a.ts           ← Google search + immediate filters
        phase-b.ts           ← HTTP check + Lighthouse + Score + Screenshots
        phase-c.ts           ← Place Details + Business signals + Save
  
  step2/
    src/
      worker.ts             ← Step 2 worker (Phase 1 + Phase 2 orchestration)
      phase1/
        suppression.ts       ← Sub-Step 2.1
        freshness.ts         ← Sub-Step 2.2
        impressum.ts         ← Sub-Step 2.3
        normalization.ts     ← Sub-Step 2.4
        email.ts             ← Sub-Step 2.5
        intelligence.ts      ← Sub-Step 2.6
        routing.ts           ← Sub-Step 2.7
      phase2/
        batch.ts             ← All Phase 2 batch operations
      utils/
        impressum-patterns.ts ← Regex patterns for Impressum parsing
        legal-forms.ts       ← Legal form detection
```

---

## 22. SHARED INFRASTRUCTURE FUNCTIONS

These functions are used by BOTH Step 1 and Step 2 (and will be used by later steps). They belong in `packages/shared/` or `packages/queue/`.

```typescript
// ============================================
// packages/shared/src/events.ts
// ============================================

import { redis } from '@webolution/queue/redis';

export async function emitEvent(eventName: string, payload: Record<string, any>): Promise<void> {
  const event = {
    name: eventName,
    payload,
    timestamp: new Date().toISOString(),
    id: crypto.randomUUID(),
  };
  
  // Publish to Redis PubSub for real-time listeners (SSE to frontend)
  await redis.publish('pipeline:events', JSON.stringify(event));
  
  // Also store in event log for audit trail
  await redis.xadd('events:log', '*', 
    'name', eventName, 
    'payload', JSON.stringify(payload),
    'timestamp', event.timestamp,
  );
  
  logger.info({ event: eventName, ...payload }, 'Event emitted');
}

// ============================================
// packages/shared/src/industry-profile.ts
// ============================================

import { prisma } from '@webolution/db';

// Cache profiles in memory (invalidated on profile.updated event)
const profileCache = new Map<string, { profile: IndustryProfile; loadedAt: number }>();
const CACHE_TTL_MS = 5 * 60 * 1000; // 5 minutes

export async function loadIndustryProfile(industryTag: string): Promise<IndustryProfile> {
  const cached = profileCache.get(industryTag);
  if (cached && Date.now() - cached.loadedAt < CACHE_TTL_MS) {
    return cached.profile;
  }
  
  // Load from DB (or from JSON file for MVP)
  const profile = await prisma.industryProfile.findUnique({
    where: { tag: industryTag },
  });
  
  if (!profile) {
    throw new Error(`Industry profile not found: ${industryTag}`);
  }
  
  const parsed = profile.config as IndustryProfile;
  profileCache.set(industryTag, { profile: parsed, loadedAt: Date.now() });
  return parsed;
}

// ============================================
// packages/shared/src/r2.ts
// ============================================

import { S3Client, PutObjectCommand, DeleteObjectCommand } from '@aws-sdk/client-s3';

const r2 = new S3Client({
  region: 'auto',
  endpoint: process.env.R2_ENDPOINT!,
  credentials: {
    accessKeyId: process.env.R2_ACCESS_KEY_ID!,
    secretAccessKey: process.env.R2_SECRET_ACCESS_KEY!,
  },
});

const R2_BUCKET = process.env.R2_BUCKET ?? 'webolution';
const R2_PUBLIC_URL = process.env.R2_PUBLIC_URL ?? 'https://assets.webolution.de';

export async function uploadToR2(key: string, data: Buffer, contentType = 'image/jpeg'): Promise<string> {
  await r2.send(new PutObjectCommand({
    Bucket: R2_BUCKET,
    Key: key,
    Body: data,
    ContentType: contentType,
  }));
  return `${R2_PUBLIC_URL}/${key}`;
}

export async function deleteFromR2(url: string): Promise<void> {
  const key = url.replace(`${R2_PUBLIC_URL}/`, '');
  await r2.send(new DeleteObjectCommand({ Bucket: R2_BUCKET, Key: key }));
}

// ============================================
// packages/shared/src/cost-tracking.ts
// ============================================

import { prisma } from '@webolution/db';

export async function logApiCost(
  apiName: string, 
  endpoint: string, 
  costCents: number,
  campaignId?: string,
  durationMs?: number,
  responseStatus?: number,
): Promise<void> {
  await prisma.apiCostLog.create({
    data: {
      apiName,
      endpoint,
      costCents: Math.round(costCents * 10) / 10, // round to 0.1 cent
      campaignId: campaignId ?? null,
      durationMs: durationMs ?? null,
      responseStatus: responseStatus ?? null,
    },
  });
  
  // Update campaign spent counter
  if (campaignId) {
    await prisma.campaign.update({
      where: { id: campaignId },
      data: { spentCents: { increment: Math.round(costCents) } },
    });
  }
}

// ============================================
// packages/queue/src/redis.ts
// ============================================

import Redis from 'ioredis';

export const redis = new Redis(process.env.REDIS_URL ?? 'redis://localhost:6379', {
  maxRetriesPerRequest: null, // required for BullMQ
  enableReadyCheck: false,
});

// ============================================
// packages/queue/src/queues.ts
// ============================================

import { Queue } from 'bullmq';
import { redis } from './redis';

const defaultJobOptions = {
  attempts: 3,
  backoff: { type: 'exponential' as const, delay: 30000 },
  removeOnComplete: { count: 200 },
  removeOnFail: { count: 100 },
};

export const campaignQueue = new Queue('campaign-queue', { connection: redis, defaultJobOptions });
export const step2Queue = new Queue('step2-queue', { connection: redis, defaultJobOptions });
export const enrichmentQueue = new Queue('enrichment-queue', { connection: redis, defaultJobOptions });
export const crawlQueue = new Queue('crawl-queue', { connection: redis, defaultJobOptions });
export const reviewQueue = new Queue('review-queue', { connection: redis, defaultJobOptions });
export const competitorQueue = new Queue('competitor-queue', { connection: redis, defaultJobOptions });
export const scoreQueue = new Queue('score-queue', { connection: redis, defaultJobOptions });
// ... steps 8-17 queues follow the same pattern

// ============================================
// packages/shared/src/logger.ts
// ============================================

import pino from 'pino';

export const logger = pino({
  level: process.env.LOG_LEVEL ?? 'info',
  transport: process.env.NODE_ENV === 'development' 
    ? { target: 'pino-pretty', options: { colorize: true } } 
    : undefined,
  base: { service: process.env.SERVICE_NAME ?? 'webolution' },
});

// Usage in any service:
// import { logger } from '@webolution/shared/logger';
// logger.info({ leadId, quickScore }, 'Lead qualified');
// logger.error({ leadId, error: err.message }, 'Step failed');
// logger.warn({ leadId }, 'Impressum parse failed, continuing');
```

### IndustryProfile Type Definition

```typescript
// packages/shared/src/types/industry-profile.ts

export interface IndustryProfile {
  tag: string;                    // "coaches", "friseure"
  name: string;                   // "Coaches & Berater"
  active: boolean;
  version: number;
  
  scraping: {
    keywords: string[];
    acceptedCategories: string[];
    excludedCategories: string[];
    excludeChains: string[];
    geoRadiusKm: number;
    noWebsiteAction: 'skip' | 'trackB';
    additionalSources: string[];
  };
  
  qualityCheck: {
    functionalWeights: Record<string, number>;
    visualWeights: Record<string, number>;
    maxRawScore: number;
    qualifyThreshold: number;
    priorityThreshold: number;
    goldThreshold: number;
    bannedPhrases: string[];
  };
  
  scoring: {
    goldCriteria: {
      minQuickScore: number;
      minGoogleRating: number;
      minReviewCount: number;
    };
    reachabilityWeights: Record<string, number>;
  };
  
  outreach: {
    primaryChannel: string;
    addressForm: 'Du' | 'Sie';
    spamSaturationRisk: string;
    seasonalNotes: Record<string, string>;
    weaknessToArgument: Record<string, string>;
    ctaAction: string;
    ctaVerb: string;
    bestSendingTime: {
      days: string[];
      hours: string;
      note?: string;
    };
  };
  
  contentPolish: {
    toneOfVoice: string;
    bannedPhrases: string[];
    powerWords: string[];
    ctaPrimary: string;
    ctaSecondary: string;
    imageStrategy: string;
  };
  
  display: {
    screenshotPriority: 'mobile' | 'desktop';
  };
  
  step2: {
    impressumPriority: 'high' | 'medium' | 'low';
    enrichmentSources: Array<{ source: string; priority: number; costCents: number }>;
    skipSources: string[];
    outreachPersonalization: {
      useFirstName: boolean;
      needsDecisionMakerName: boolean;
    };
    impressumFields: {
      required: string[];
      valuable: string[];
      optional: string[];
    };
    investmentPerField: Record<string, { maxCostCents: number; reason: string }>;
    suppressionOverrides: {
      allowNonProfit: boolean;
      coolingPeriodMonths: number;
      minReviewRating: number;
    };
    verificationSources: Array<{ source: string; url: string; field: string }>;
  };
  
  // Profiles for later steps (defined in their respective specs):
  rebuildTemplates?: any[];
  // ... etc
}
```

### DB Model for Industry Profiles

```prisma
// Add to schema.prisma:

model IndustryProfile {
  id          String    @id @default(uuid())
  tag         String    @unique   // "coaches", "friseure"
  name        String               // "Coaches & Berater"
  active      Boolean   @default(true)
  version     Int       @default(1)
  config      Json                 // the full IndustryProfile JSON
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt
  
  @@index([tag])
  @@index([active])
}
```

---

## 23. PRISMA SCHEMA: COMPLETE COMBINED LeadStatus ENUM

**CRITICAL:** The `LeadStatus` enum in Step 1 (STEP-1-FINAL.md) must be EXTENDED with Step 2 statuses. The combined enum that must be in `schema.prisma`:

```prisma
// REPLACES the LeadStatus enum from Step 1.
// Includes ALL statuses from Step 1 + Step 2 + later steps.

enum LeadStatus {
  // Step 1 statuses
  RAW                   // just scraped from Google, no checks yet
  CHECKING              // currently being checked (Step 1 Phase B)
  QUALIFIED             // passed Step 1 quality check, waiting for Gate 1
  FILTERED_OUT          // did not pass quality check
  DUPLICATE             // duplicate of existing lead
  NO_WEBSITE            // no website URL (Track B candidate)
  PARKED                // domain is parked
  UNDER_CONSTRUCTION    // website shows "coming soon"
  UNREACHABLE           // website not reachable (DNS error, timeout)
  AUTH_REQUIRED         // website requires login
  
  // Gate 1 statuses
  GATE1_APPROVED        // approved by operator at Gate 1
  GATE1_REJECTED        // rejected by operator at Gate 1
  
  // Step 2 statuses
  STEP2_PROCESSING      // currently being processed by Step 2
  STEP2_FAILED          // Step 2 crashed, waiting for retry
  READY_FOR_ENRICHMENT  // passed Step 2, ready for Steps 3+4
  SUPPRESSED            // suppressed by Step 2 (cooling, bounce, competitor, etc.)
  
  // Step 3+ statuses (placeholders for later steps)
  ENRICHING             // Step 3 is running
  ENRICHED              // Step 3 complete
  CRAWLING              // Step 4 is running
  CRAWLED               // Step 4 complete
  REVIEWING             // Steps 5-6 running
  REVIEWED              // Steps 5-6 complete
  SCORING               // Step 7 running
  SCORED                // Step 7 complete
  GATE3_APPROVED        // approved at Gate 3
  GATE3_REJECTED        // rejected at Gate 3
  PLANNING              // Step 8 running
  POLISHING             // Step 9 running
  BUILDING              // Step 10 running
  QA_TESTING            // Step 11 running
  QA_PASSED             // Step 11 passed
  QA_FAILED             // Step 11 failed
  GATE4_APPROVED        // approved at Gate 4
  DEPLOYING             // Step 12 running
  DEPLOYED              // Step 12 complete (preview live)
  OUTREACH_PREP         // Step 13 running
  GATE5_APPROVED        // approved at Gate 5
  SENDING               // Step 14 sending emails
  SENT                  // outreach complete
  CONVERTED             // lead became a paying customer
  NOT_CONVERTED         // outreach completed, no conversion
}
```

**When building the Prisma schema:** Use THIS combined enum, not the one from Step 1 alone. Step 1's version is a SUBSET.

Also add the `SuppressionLog` relation to the Lead model:

```prisma
model Lead {
  // ... all fields from Step 1 + Step 2 ...
  
  suppressionLogs      SuppressionLog[]
}
```

---

## 24. QUEUE NAME RECONCILIATION

The original `EVENTS.md` uses queue names like `pipeline:validate`. The new Step 1 and Step 2 use different names. This is the AUTHORITATIVE mapping:

```typescript
// OLD names (from EVENTS.md) → NEW names (from STEP-1-FINAL + STEP-2-FINAL)
// =========================================================================

// Step 1: Campaign-based processing (not per-lead queue)
'pipeline:scrape'        → 'campaign-queue'     // Step 1 processes entire campaigns

// Step 2: Per-lead + batch processing  
'pipeline:validate'      → 'step2-queue'        // Step 2 (was "validate", now "intelligence")

// Steps 3+4: Dispatched IN PARALLEL by Step 2
'pipeline:enrich'        → 'enrichment-queue'   // Step 3
'pipeline:crawl'         → 'crawl-queue'        // Step 4

// Steps 5+: Keep original names from EVENTS.md for now
'pipeline:review'        → 'review-queue'       // Step 5
'pipeline:competitor'    → 'competitor-queue'    // Step 6
'pipeline:score'         → 'score-queue'        // Step 7
// ... etc.
```

The queue definitions in Step 2 Section 22 (`packages/queue/src/queues.ts`) are AUTHORITATIVE and should be used when building.
