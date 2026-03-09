# STEPS 3+4: ENRICHMENT & WEBSITE CRAWLING (PARALLEL)
# Complete Implementation Spec v1.0

> **This is a SELF-CONTAINED spec.** Claude Code reads Step 1 + Step 2 + this file and builds Steps 3+4.
> Steps 3 and 4 run IN PARALLEL after Step 2. Step 3 fills remaining data gaps.
> Step 4 crawls the full website and extracts all content. An Orchestrator merges results
> before dispatching to Steps 5+6.

---

## 1. MISSION

### Step 3: "Gap-Fill Enrichment"

**Old name:** "Lead Enrichment & Pre-Qualification"
**New name:** "Gap-Fill Enrichment"

**What this step does:** Execute the enrichment plan created by Step 2. Fill ONLY the data gaps that Steps 1+2 couldn't fill. Find competitors from our own DB. Optionally call LinkedIn, Northdata, or portal APIs — but ONLY if the plan says so.

**What this step does NOT do:** Call Google Place Details (Step 1 already did). Parse the Impressum (Step 2 already did). Calculate a Business Health Score (redundant — Step 1+2 signals are sufficient). Do a full website crawl (that's Step 4).

**Cost:** €0.00-0.07 per lead (most leads: €0.00-0.02)
**Duration:** 3-15 seconds per lead

### Step 4: "Full Website Crawl & Content Extraction"

**Old name:** "Website Crawl & Content Extraction" (unchanged — this step was always substantial)

**What this step does:** Crawl the FULL website (not just homepage), extract ALL content, download images, identify brand assets, run AI-powered content extraction into a structured firm profile, and produce a complete ContentPackage for the rebuild.

**What this step does NOT do:** Score the website (that's Step 5). Analyze competitors (that's Step 6). Plan the rebuild (that's Step 8).

**Cost:** €0.05-0.15 per lead (Playwright compute + Claude Sonnet extraction + R2 storage)
**Duration:** 30-300 seconds per lead (depending on site size and crawlStrategy)

### Parallel Architecture

```
Step 2 completes
  ↓
  ├──→ Step 3 Queue (enrichment-queue)     [3-15 sec]
  │       ↓
  │    competitors.found event ──→ Step 6 CAN start early
  │       ↓
  │    enrichment.completed event
  │
  └──→ Step 4 Queue (crawl-queue)          [30-300 sec]
          ↓
       crawl.completed event
          
Both complete? → Orchestrator MERGE → dispatch Steps 5+6
```

**Key insight:** Step 3 finishes FAST (3-15 sec). Step 4 finishes SLOW (30-300 sec). Step 6 (Competitor Benchmarking) only needs Step 3's competitor_ids — it can start BEFORE Step 4 finishes.

## 2. WHAT STEPS 3+4 RECEIVE (from Step 2)

### Step 3 receives:

```typescript
// From Step 2 (Section 13, STEP-2-FINAL.md):
interface Step2ToStep3 {
  leadId: string;
  status: 'READY_FOR_ENRICHMENT';
  firmName: string;
  actualUrl: string;
  industryTag: string;
  enrichmentPlan: EnrichmentStep[];      // which APIs, in which order
  outreachChannel: OutreachChannel;
  pipelineRoute: PipelineRoute;          // FULL, ANALYSIS_ONLY, PHONE_ONLY, FAST_TRACK, NURTURE
  pipelinePriority: number;
  
  // Already available (may be null):
  decisionMakerName: string | null;
  decisionMakerEmail: string | null;
  phoneNormalized: string | null;
  legalForm: string | null;
  step2Incomplete: boolean;
}
```

### Step 4 receives:

```typescript
// From Step 2 (Section 13, STEP-2-FINAL.md):
interface Step2ToStep4 {
  leadId: string;
  actualUrl: string;
  industryTag: string;
  crawlStrategy: CrawlStrategy;          // FAST, THROTTLED, SKIP
  priorityPages: string[];               // URLs to crawl first
  estimatedPageCount: number;
  isSpa: boolean;
  hasGoogleAnalytics: boolean;
  cmsType: string | null;                // wordpress, jimdo, wix, etc.
  pipelineRoute: PipelineRoute;
}
```

## 3. WHAT STEPS 3+4 ADD (the combined DELTA)

After Steps 3+4 + Merge, the lead has EVERYTHING from Steps 1+2 PLUS:

```
From Step 3 (NEW):
  🆕 competitorPlaceIds: string[]          // 3-5 nearby competitors
  🆕 competitorLeadIds: string[]           // if competitors are also leads in our DB
  🆕 linkedInUrl: string | null            // found via Google search
  🆕 portalProfiles: PortalProfile[]       // Treatwell, MyHammer, etc. (existence + URL)
  🆕 northdataResult: NorthdataResult      // only for GmbH/UG: firmStatus, capital, employees
  🆕 enrichmentConfidence: number          // 0-1
  🆕 enrichmentCostCents: number
  🆕 allEmailCandidates: EmailCandidate[]  // all found emails ranked by priority
  🆕 outreachChannelUpdated: boolean       // true if channel changed based on new data
  🆕 enrichmentStatus: StepStatus          // COMPLETED, FAILED, SKIPPED

From Step 4 (NEW):
  🆕 contentPackageId: string              // FK to ContentPackage
  🆕 ContentPackage (separate model):
      - pages[]: ContentPage[]             // all crawled pages with text, screenshots
      - images[]: ContentImage[]           // all downloaded images, classified
      - brandAssets: BrandAssets           // logo, colors, fonts, design direction
      - firmProfile: FirmProfile           // AI-extracted structured business data
      - navigationStructure: NavStructure  // site navigation hierarchy
      - embeds: EmbedInfo[]                // iframes: maps, videos, booking widgets
      - videos: VideoInfo[]                // YouTube, Vimeo, self-hosted
      - pdfs: PdfContent[]                // menus, price lists
      - structuredData: any[]              // Schema.org JSON-LD
      - seoAnalysis: SeoAnalysis          // per-page + site-wide SEO
      - accessibilityIssues: string[]
      - formAnalysis: FormAnalysis[]
      - responsiveDesign: ResponsiveResult
      - cookieBannerText: string | null
      - fontUsage: FontUsage
      - styleAnalysis: StyleAnalysis
  🆕 contentCompleteness: number           // 0-1 (REAL, replaces Step 2 estimate)
  🆕 crawlDurationMs: number
  🆕 crawlCostCents: number
  🆕 crawlStatus: StepStatus              // COMPLETED, FAILED, SKIPPED
  🆕 needsRescore: boolean                // true if website better/worse than Step 1 thought

From Orchestrator MERGE (NEW):
  🆕 contentBriefData: UPDATED            // Step 2 base + Step 3 refinements + Step 4 real data
  🆕 pipelineRoute: POSSIBLY CHANGED      // FULL→NURTURE if content too thin
  🆕 lifecycleTracker: UPDATED            // Steps 3+4 entries added
```

## 4. DATABASE SCHEMA

### New Models (Step 4)

```prisma
// ============================================
// CONTENT PACKAGE (Step 4 output)
// ============================================

model ContentPackage {
  id                     String          @id @default(uuid())
  leadId                 String          @unique
  lead                   Lead            @relation(fields: [leadId], references: [id], onDelete: Cascade)
  
  // AI Extraction
  firmProfile            Json?           // structured business data (services, team, etc.)
  extractionConfidence   Float           @default(0)
  extractionModel        String?         // "claude-sonnet-4-20250514"
  extractionPromptVersion String?        // "coaches-v1.2"
  
  // Site Analysis
  navigationStructure    Json?           // { mainNav[], footerNav[], depth }
  embeds                 Json?           // [{ type, src }] (maps, videos, booking)
  videos                 Json?           // [{ platform, videoId, embedUrl }]
  pdfs                   Json?           // [{ url, text, pageCount, type }]
  structuredData         Json?           // Schema.org JSON-LD blocks
  seoAnalysis            Json?           // per-page + site-wide SEO
  accessibilityIssues    String[]        @default([])
  responsiveDesign       Json?           // { isResponsive, hasHorizontalScroll, etc. }
  cookieBannerText       String?
  
  // Brand Assets
  brandAssets            Json?           // { logoUrl, colors, fonts, designDirection }
  fontUsage              Json?           // { fonts[], primaryFont, usesGoogleFonts }
  styleAnalysis          Json?           // { borderRadius, shadows, animations, etc. }
  
  // Metrics
  contentCompleteness    Float           @default(0)
  pagesAttempted         Int             @default(0)
  pagesSuccessful        Int             @default(0)
  crawlDurationMs        Int?
  crawlCostCents         Int             @default(0)
  
  // Relations
  pages                  ContentPage[]
  images                 ContentImage[]
  
  createdAt              DateTime        @default(now())
  updatedAt              DateTime        @updatedAt
}

model ContentPage {
  id                     String          @id @default(uuid())
  contentPackageId       String
  contentPackage         ContentPackage  @relation(fields: [contentPackageId], references: [id], onDelete: Cascade)
  
  url                    String
  type                   PageType
  title                  String?
  textContent            String?         // plain text, boilerplate removed
  wordCount              Int             @default(0)
  htmlCleanR2Url         String?         // clean HTML stored in R2
  screenshotDesktopUrl   String?         // R2 URL
  screenshotMobileUrl    String?         // R2 URL
  loadTimeMs             Int?
  
  // Page-level SEO
  metaDescription        String?
  h1Text                 String?
  h1Count                Int             @default(0)
  hasCanonical           Boolean         @default(false)
  hasOgTags              Boolean         @default(false)
  hasStructuredData      Boolean         @default(false)
  internalLinkCount      Int             @default(0)
  externalLinkCount      Int             @default(0)
  imagesOnPage           Int             @default(0)
  imagesWithAlt          Int             @default(0)
  
  // Form analysis (if page has a form)
  formAnalysis           Json?           // { fieldCount, hasEmail, actionType, etc. }
  
  crawledAt              DateTime        @default(now())
  
  @@index([contentPackageId])
  @@index([type])
}

model ContentImage {
  id                     String          @id @default(uuid())
  contentPackageId       String
  contentPackage         ContentPackage  @relation(fields: [contentPackageId], references: [id], onDelete: Cascade)
  
  originalUrl            String
  r2Url                  String
  alt                    String?
  width                  Int
  height                 Int
  fileSizeBytes          Int?
  type                   ImageType
  usable                 Boolean         @default(true)
  fromPageUrl            String?
  
  @@index([contentPackageId])
  @@index([type])
}

enum PageType {
  HOME
  ABOUT
  SERVICES
  TEAM
  PRICING
  GALLERY
  BLOG
  CONTACT
  LEGAL          // impressum, datenschutz, agb
  FAQ
  REFERENCES     // referenzen, projekte, portfolio
  OTHER
}

enum ImageType {
  PHOTO          // real business photo
  LOGO           // business logo
  ICON           // small icon/symbol
  STOCK          // detected stock photo
  FAVICON        // site favicon
  CERTIFICATION  // badge/seal image
  UNKNOWN
}
```

### Additions to Lead Model (Steps 3+4)

```prisma
model Lead {
  // ... all Step 1 + Step 2 fields ...
  
  // === STEP 3 FIELDS ===
  competitorPlaceIds       String[]       @default([])
  competitorLeadIds        String[]       @default([])
  linkedInUrl              String?
  portalProfiles           Json?          // [{ platform, url, exists }]
  northdataResult          Json?          // { firmStatus, capital, employees }
  enrichmentConfidence     Float?
  enrichmentCostCents      Int            @default(0)
  allEmailCandidates       Json?          // [{ email, source, validation, priority }]
  outreachChannelUpdated   Boolean        @default(false)
  enrichmentStartedAt      DateTime?
  enrichmentCompletedAt    DateTime?
  enrichmentStatus         StepStatus?
  
  // === STEP 4 FIELDS ===
  contentPackage           ContentPackage?
  contentCompleteness      Float?         // REAL value from crawl (overrides Step 2 estimate)
  crawlDurationMs          Int?
  crawlCostCents           Int            @default(0)
  crawlStartedAt           DateTime?
  crawlCompletedAt         DateTime?
  crawlStatus              StepStatus?
  needsRescore             Boolean        @default(false)
  rescoreReason            String?
  
  // === INDEXES ===
  @@index([enrichmentStatus])
  @@index([crawlStatus])
}
```

---

## 5. STEP 3: THE FLOW (Gap-Fill Enrichment)

### 5.1 Worker Setup

```typescript
import { Worker, Job } from 'bullmq';

const step3Worker = new Worker('enrichment-queue', async (job: Job) => {
  await executeEnrichment(job.data.leadId);
}, { concurrency: 10, connection: redis }); // 10 parallel (lightweight API calls)
```

### 5.2 Main Execution Logic

```typescript
async function executeEnrichment(leadId: string): Promise<void> {
  const lead = await prisma.lead.findUnique({ where: { id: leadId } });
  if (!lead || lead.status !== 'READY_FOR_ENRICHMENT') return;
  
  const industryProfile = await loadIndustryProfile(lead.industryTag);
  const startTime = Date.now();
  let totalCostCents = 0;
  
  await prisma.lead.update({
    where: { id: leadId },
    data: { enrichmentStartedAt: new Date(), enrichmentStatus: 'RUNNING' },
  });
  
  try {
    // === FAST-TRACK / PHONE-ONLY: Minimal enrichment ===
    if (lead.pipelineRoute === 'FAST_TRACK') {
      await markEnrichmentComplete(leadId, 'SKIPPED', 0, 0);
      return;
    }
    
    // === STEP 1: Find Competitors (ALWAYS, FIRST — enables Step 6 early start) ===
    const competitors = await findCompetitors(lead, industryProfile);
    
    // Emit immediately so Step 6 can start
    await prisma.lead.update({
      where: { id: leadId },
      data: {
        competitorPlaceIds: competitors.placeIds,
        competitorLeadIds: competitors.leadIds,
      },
    });
    await emitEvent('lead.competitors_found', {
      leadId,
      competitorCount: competitors.placeIds.length,
      competitorLeadIds: competitors.leadIds,
    });
    
    // === STEP 2: Execute Enrichment Plan (waterfall per gap) ===
    if (lead.pipelineRoute !== 'PHONE_ONLY') {
      const plan = lead.enrichmentPlan as EnrichmentStep[] ?? [];
      const filledFields = new Set<string>();
      const allEmails: EmailCandidate[] = collectExistingEmails(lead);
      
      for (const step of plan) {
        if (filledFields.has(step.field)) continue; // already filled opportunistically
        
        for (const source of step.sources) {
          // Skip sources the industry profile excludes
          const skipSources = industryProfile.step2?.skipSources ?? [];
          if (skipSources.includes(source.source)) continue;
          
          // Cost check
          if (totalCostCents + source.costCents > 25) { // max €0.25 per lead
            logger.info({ leadId, source: source.source }, 'Enrichment cost cap reached');
            break;
          }
          
          const result = await callWithRetry(source.source, async () => {
            switch (source.source) {
              case 'google_maps_similar': return await findCompetitorsViaGoogle(lead);
              case 'linkedin': return await findLinkedInProfile(lead);
              case 'hunter_io': return await findEmailViaHunter(lead);
              case 'northdata': return await checkNorthdata(lead);
              case 'treatwell_search': return await checkPortalProfile(lead, 'treatwell');
              case 'booksy_search': return await checkPortalProfile(lead, 'booksy');
              case 'myhammer_search': return await checkPortalProfile(lead, 'myhammer');
              default: return null;
            }
          });
          
          if (result) {
            totalCostCents += source.costCents;
            await logApiCost(source.source, source.source, source.costCents, lead.campaignId);
            
            // Apply result
            if (result.email) {
              allEmails.push(result.email);
              filledFields.add('email');
            }
            if (result.decisionMakerName && !lead.decisionMakerName) {
              filledFields.add('decisionMakerName');
            }
            if (result.portalProfile) {
              filledFields.add(step.field);
            }
            if (result.linkedInUrl) {
              filledFields.add('linkedInUrl');
            }
            if (result.northdata) {
              filledFields.add('northdata');
            }
            
            // Opportunistic: did this source fill OTHER fields too?
            if (result.bonusFields) {
              for (const bonus of result.bonusFields) {
                filledFields.add(bonus.field);
              }
            }
            
            break; // field filled, next plan step
          }
        }
      }
      
      // === STEP 3: Rank all email candidates ===
      const rankedEmails = rankEmailCandidates(allEmails, lead.actualUrl!);
      
      // === STEP 4: Update outreach channel if new data changes it ===
      const channelUpdate = checkOutreachChannelUpdate(lead, rankedEmails);
      
      // === STEP 5: Disqualification check ===
      const disqualified = checkStep3Disqualification(lead, { northdata: lead.northdataResult });
      if (disqualified) {
        await disqualifyInStep3(leadId, disqualified.reason, disqualified.detail);
        return;
      }
      
      // === SAVE ALL RESULTS (atomic write) ===
      await prisma.lead.update({
        where: { id: leadId },
        data: {
          allEmailCandidates: rankedEmails,
          decisionMakerEmail: rankedEmails[0]?.email ?? lead.decisionMakerEmail,
          linkedInUrl: filledFields.has('linkedInUrl') ? lead.linkedInUrl : undefined,
          portalProfiles: lead.portalProfiles,
          northdataResult: lead.northdataResult,
          enrichmentConfidence: calculateEnrichmentConfidence(lead, filledFields),
          enrichmentCostCents: totalCostCents,
          ...(channelUpdate ?? {}),
        },
      });
    }
    
    await markEnrichmentComplete(leadId, 'COMPLETED', totalCostCents, Date.now() - startTime);
    
  } catch (error) {
    logger.error({ leadId, error: String(error) }, 'Step 3 failed');
    await markEnrichmentComplete(leadId, 'FAILED', totalCostCents, Date.now() - startTime);
  }
}

async function markEnrichmentComplete(
  leadId: string, status: StepStatus, costCents: number, durationMs: number
): Promise<void> {
  await prisma.lead.update({
    where: { id: leadId },
    data: {
      enrichmentStatus: status,
      enrichmentCompletedAt: new Date(),
      enrichmentCostCents: costCents,
    },
  });
  await emitEvent('lead.enrichment_completed', { leadId, status, costCents, durationMs });
  await updateLifecycle(leadId, 3, 'Gap-Fill Enrichment', status === 'COMPLETED' ? 'passed' : 'failed', costCents);
  await checkBothStepsComplete(leadId);
}
```

### 5.3 Find Competitors (FREE — from our own DB)

```typescript
async function findCompetitors(lead: Lead, profile: IndustryProfile): Promise<CompetitorResult> {
  // FIRST: Try to find competitors from our OWN database (other leads in same city+industry)
  const dbCompetitors = await prisma.lead.findMany({
    where: {
      industryTag: lead.industryTag,
      city: lead.city,
      id: { not: lead.id },
      status: { notIn: ['FILTERED_OUT', 'SUPPRESSED', 'DUPLICATE'] },
      googlePlaceId: { not: null },
    },
    orderBy: [
      { googleRating: 'desc' },
      { googleReviewCount: 'desc' },
    ],
    take: 5,
    select: {
      id: true,
      googlePlaceId: true,
      firmName: true,
      actualUrl: true,
      quickScore: true,
      googleRating: true,
      googleReviewCount: true,
    },
  });
  
  if (dbCompetitors.length >= 3) {
    // Enough competitors from DB — FREE!
    return {
      placeIds: dbCompetitors.map(c => c.googlePlaceId!),
      leadIds: dbCompetitors.map(c => c.id),
      source: 'database',
      costCents: 0,
    };
  }
  
  // FALLBACK: If <3 competitors in DB, supplement with a Google Text Search
  // Only if competitor field was in the enrichment plan
  if (dbCompetitors.length < 3) {
    try {
      const searchResponse = await fetch('https://places.googleapis.com/v1/places:searchText', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'X-Goog-Api-Key': process.env.GOOGLE_API_KEY!,
          'X-Goog-FieldMask': 'places.id,places.displayName,places.rating,places.userRatingCount,places.websiteUri',
        },
        body: JSON.stringify({
          textQuery: `${profile.scraping.keywords[0]} ${lead.city}`,
          languageCode: 'de',
          regionCode: 'DE',
          maxResultCount: 10,
        }),
      });
      await logApiCost('google_text_search', 'competitor_search', 3.2, lead.campaignId);
      
      const data = await searchResponse.json();
      const apiCompetitors = (data.places ?? [])
        .filter((p: any) => p.id !== lead.googlePlaceId)
        .slice(0, 5 - dbCompetitors.length);
      
      return {
        placeIds: [
          ...dbCompetitors.map(c => c.googlePlaceId!),
          ...apiCompetitors.map((c: any) => c.id),
        ],
        leadIds: dbCompetitors.map(c => c.id),
        source: 'database+google',
        costCents: 3.2,
      };
    } catch {
      // Google search failed — use what we have from DB
      return {
        placeIds: dbCompetitors.map(c => c.googlePlaceId!),
        leadIds: dbCompetitors.map(c => c.id),
        source: 'database',
        costCents: 0,
      };
    }
  }
  
  return { placeIds: [], leadIds: [], source: 'none', costCents: 0 };
}
```

### 5.4 LinkedIn Profile Finder (MVP — via Google Search)

```typescript
async function findLinkedInProfile(lead: Lead): Promise<EnrichmentSourceResult | null> {
  const name = lead.decisionMakerName ?? lead.firmNameNormalized ?? lead.firmName;
  const query = `"${name}" "${lead.city}" site:linkedin.com/in`;
  
  const results = await googleCustomSearch(query, 1);
  
  if (results.length > 0 && results[0].url.includes('linkedin.com/in/')) {
    return {
      linkedInUrl: results[0].url,
      // Extract name from LinkedIn snippet if we didn't have it
      bonusFields: !lead.decisionMakerName && results[0].title ? [{
        field: 'decisionMakerName',
        value: results[0].title.split(' - ')[0].split(' | ')[0].trim(),
      }] : [],
    };
  }
  return null;
}
```

### 5.5 Northdata Check (only for GmbH/UG)

```typescript
async function checkNorthdata(lead: Lead): Promise<EnrichmentSourceResult | null> {
  // Only check for incorporated companies
  if (!lead.legalForm || !['GmbH', 'UG', 'AG', 'KG', 'OHG', 'GmbH & Co. KG'].includes(lead.legalForm)) {
    return null;
  }
  
  // Check if Handelsregister number is available from Impressum
  if (!lead.handelsregisterNr) return null;
  
  // Northdata API (€0.05 per lookup)
  try {
    const response = await fetch(`https://www.northdata.de/api/v1/company?name=${encodeURIComponent(lead.firmName)}&city=${encodeURIComponent(lead.city)}`, {
      headers: { 'Authorization': `Bearer ${process.env.NORTHDATA_API_KEY}` },
      signal: AbortSignal.timeout(10000),
    });
    
    if (!response.ok) return null;
    const data = await response.json();
    
    return {
      northdata: {
        firmStatus: data.status ?? 'unknown',     // 'active', 'dissolved', 'liquidation'
        registeredCapital: data.registeredCapital ?? null,
        employeeCount: data.employeeCount ?? null,
        foundingDate: data.foundingDate ?? null,
        industry: data.industry ?? null,
      },
    };
  } catch {
    return null;
  }
}
```

### 5.6 Portal Profile Check

```typescript
async function checkPortalProfile(
  lead: Lead, 
  platform: string
): Promise<EnrichmentSourceResult | null> {
  const PORTAL_SEARCH_URLS: Record<string, (name: string, city: string) => string> = {
    treatwell: (name, city) => `https://www.treatwell.de/suche/?q=${encodeURIComponent(name + ' ' + city)}`,
    booksy: (name, city) => `https://booksy.com/de-de/s/${encodeURIComponent(name + ' ' + city)}`,
    myhammer: (name, city) => `https://www.myhammer.de/suche?q=${encodeURIComponent(name + ' ' + city)}`,
  };
  
  const searchUrl = PORTAL_SEARCH_URLS[platform];
  if (!searchUrl) return null;
  
  try {
    const response = await fetch(searchUrl(lead.firmName, lead.city), {
      headers: { 'User-Agent': BROWSER_USER_AGENT },
      signal: AbortSignal.timeout(5000),
      redirect: 'follow',
    });
    
    // We don't scrape the portal — just check if it returned results
    const exists = response.ok && response.url !== searchUrl(lead.firmName, lead.city);
    
    return {
      portalProfile: {
        platform,
        url: exists ? response.url : null,
        exists,
      },
    };
  } catch {
    return null;
  }
}
```

### 5.7 Email Ranking + Outreach Channel Update

```typescript
function rankEmailCandidates(candidates: EmailCandidate[], websiteDomain: string): EmailCandidate[] {
  const domain = normalizeDomain(websiteDomain);
  
  return candidates
    .map(c => ({
      ...c,
      priority: calculateEmailPriority(c, domain),
    }))
    .sort((a, b) => b.priority - a.priority);
}

function calculateEmailPriority(candidate: EmailCandidate, websiteDomain: string): number {
  let score = 0;
  
  // Personal > generic
  if (candidate.isPersonal) score += 40;
  
  // Own domain > external
  const emailDomain = candidate.email.split('@')[1];
  if (emailDomain === websiteDomain) score += 30;
  
  // Validation quality
  if (candidate.validation === 'SMTP_VALID') score += 20;
  else if (candidate.validation === 'MX_VALID') score += 10;
  else if (candidate.validation === 'CATCH_ALL') score += 5;
  
  // Source reliability
  const SOURCE_SCORES: Record<string, number> = {
    impressum: 10, website: 8, linkedin: 6, hunter: 4, guessed: 2,
  };
  score += SOURCE_SCORES[candidate.source] ?? 0;
  
  return score;
}

function collectExistingEmails(lead: Lead): EmailCandidate[] {
  const emails: EmailCandidate[] = [];
  
  if (lead.decisionMakerEmail) {
    emails.push({
      email: lead.decisionMakerEmail,
      source: lead.impressumDataSource ? 'impressum' : 'website',
      validation: lead.emailValidationResult as EmailValidation ?? 'UNVERIFIED',
      isPersonal: !lead.decisionMakerEmail.match(/^(info|kontakt|office|hello|mail)@/i),
      isOwnDomain: normalizeDomain(lead.decisionMakerEmail.split('@')[1]) === normalizeDomain(lead.actualUrl ?? ''),
      priority: 0, // will be calculated
    });
  }
  
  if (lead.emailFoundOnWebsite && lead.emailFoundOnWebsite !== lead.decisionMakerEmail) {
    emails.push({
      email: lead.emailFoundOnWebsite,
      source: 'website',
      validation: 'UNVERIFIED',
      isPersonal: !lead.emailFoundOnWebsite.match(/^(info|kontakt|office|hello|mail)@/i),
      isOwnDomain: true,
      priority: 0,
    });
  }
  
  if (lead.emailGuessed && lead.emailGuessed !== lead.decisionMakerEmail) {
    emails.push({
      email: lead.emailGuessed,
      source: 'guessed',
      validation: lead.emailGuessedValidation as EmailValidation ?? 'UNVERIFIED',
      isPersonal: true,
      isOwnDomain: true,
      priority: 0,
    });
  }
  
  return emails;
}

function checkOutreachChannelUpdate(lead: Lead, rankedEmails: EmailCandidate[]): Partial<Lead> | null {
  const bestEmail = rankedEmails[0];
  
  // Upgrade: Had no email, now we do
  if (!lead.decisionMakerEmail && bestEmail?.validation !== 'SMTP_INVALID') {
    if (['LETTER', 'PHONE', 'WHATSAPP'].includes(lead.outreachChannel!)) {
      return {
        outreachChannel: 'EMAIL' as OutreachChannel,
        outreachType: lead.pipelineRoute === 'FULL' ? 'EMAIL_PREVIEW' as OutreachType : 'EMAIL_ANALYSIS_ONLY' as OutreachType,
        outreachChannelUpdated: true,
        decisionMakerEmail: bestEmail.email,
      };
    }
  }
  
  // Downgrade: Had email, now it's invalid
  if (lead.outreachChannel === 'EMAIL' && bestEmail?.validation === 'SMTP_INVALID' && rankedEmails.length <= 1) {
    const fallback = lead.instagramHandle ? 'INSTAGRAM_DM' : 
                     lead.whatsappPossible ? 'WHATSAPP' : 
                     lead.phoneNormalized ? 'PHONE' : 'LETTER';
    return {
      outreachChannel: fallback as OutreachChannel,
      outreachChannelUpdated: true,
    };
  }
  
  return null;
}
```

### 5.8 Disqualification Check

```typescript
interface DisqualificationResult {
  reason: string;
  detail: string;
}

function checkStep3Disqualification(lead: Lead, data: any): DisqualificationResult | null {
  // Insolvent company
  if (data.northdata?.firmStatus === 'insolvent' || data.northdata?.firmStatus === 'liquidation') {
    return {
      reason: 'BUSINESS_INSOLVENT',
      detail: `Northdata zeigt Firmenstatus "${data.northdata.firmStatus}". Kein Verkauf möglich.`,
    };
  }
  
  // Dissolved company
  if (data.northdata?.firmStatus === 'dissolved') {
    return {
      reason: 'BUSINESS_DISSOLVED',
      detail: 'Firma ist laut Handelsregister aufgelöst.',
    };
  }
  
  return null;
}

async function disqualifyInStep3(leadId: string, reason: string, detail: string): Promise<void> {
  await prisma.lead.update({
    where: { id: leadId },
    data: {
      status: 'FILTERED_OUT',
      filteredReason: `step3:${reason}`,
      enrichmentStatus: 'COMPLETED',
      enrichmentCompletedAt: new Date(),
    },
  });
  
  // Cancel Step 4 if it's still running
  const crawlJobs = await crawlQueue.getJobs(['waiting', 'active']);
  const leadJob = crawlJobs.find(j => j.data.leadId === leadId);
  if (leadJob) {
    await leadJob.remove().catch(() => {});
    logger.info({ leadId }, 'Cancelled Step 4 crawl after Step 3 disqualification');
  }
  
  await emitEvent('lead.disqualified', { leadId, step: 3, reason, detail });
}
```

### 5.9 API Call Retry Logic

```typescript
const API_RETRY_CONFIG: Record<string, { maxRetries: number; backoffMs: number; multiplier: number }> = {
  google_text_search: { maxRetries: 2, backoffMs: 5000, multiplier: 2 },
  google_custom_search: { maxRetries: 2, backoffMs: 5000, multiplier: 2 },
  linkedin: { maxRetries: 1, backoffMs: 30000, multiplier: 1 },
  northdata: { maxRetries: 2, backoffMs: 10000, multiplier: 2 },
  hunter_io: { maxRetries: 2, backoffMs: 5000, multiplier: 2 },
  treatwell_search: { maxRetries: 1, backoffMs: 5000, multiplier: 1 },
  booksy_search: { maxRetries: 1, backoffMs: 5000, multiplier: 1 },
  myhammer_search: { maxRetries: 1, backoffMs: 5000, multiplier: 1 },
};

async function callWithRetry<T>(
  source: string,
  fn: () => Promise<T>
): Promise<T | null> {
  const config = API_RETRY_CONFIG[source] ?? { maxRetries: 1, backoffMs: 5000, multiplier: 2 };
  
  for (let attempt = 0; attempt <= config.maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      if (attempt === config.maxRetries) {
        logger.warn({ source, error: String(error), attempts: attempt + 1 }, 'API call failed after retries');
        return null;
      }
      const delay = config.backoffMs * Math.pow(config.multiplier, attempt);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
  return null;
}
```

### 5.10 Google Custom Search with Budget Tracking

```typescript
let dailySearchCount = 0;
let dailySearchResetDate = new Date().toDateString();
const DAILY_FREE_LIMIT = 100;

async function googleCustomSearch(query: string, maxResults: number = 3): Promise<SearchResult[]> {
  // Reset counter at day boundary
  const today = new Date().toDateString();
  if (today !== dailySearchResetDate) {
    dailySearchCount = 0;
    dailySearchResetDate = today;
  }
  
  dailySearchCount++;
  const isPaid = dailySearchCount > DAILY_FREE_LIMIT;
  const costCents = isPaid ? 0.5 : 0;
  
  if (isPaid) {
    logger.info({ query, dailyCount: dailySearchCount }, 'Google Custom Search: paid tier');
  }
  
  try {
    const params = new URLSearchParams({
      key: process.env.GOOGLE_API_KEY!,
      cx: process.env.GOOGLE_CSE_ID!,
      q: query,
      num: String(Math.min(maxResults, 10)),
    });
    
    const response = await fetch(`https://www.googleapis.com/customsearch/v1?${params}`, {
      signal: AbortSignal.timeout(10000),
    });
    
    if (!response.ok) return [];
    const data = await response.json();
    
    await logApiCost('google_custom_search', query, costCents);
    
    return (data.items ?? []).map((item: any) => ({
      url: item.link,
      title: item.title,
      snippet: item.snippet,
    }));
  } catch {
    return [];
  }
}
```

---

## 6. STEP 4: THE FLOW (Full Website Crawl & Content Extraction)

### 6.1 Worker Setup

```typescript
const step4Worker = new Worker('crawl-queue', async (job: Job) => {
  await executeCrawl(job.data.leadId, job.data as Step2ToStep4);
}, { concurrency: 3, connection: redis }); // 3 parallel (Playwright is RAM-heavy)
```

### 6.2 Shared Playwright Browser Management

```typescript
import { chromium, Browser, BrowserContext, Page } from 'playwright';

let sharedBrowser: Browser | null = null;
let crawlsSinceRecycle = 0;
const RECYCLE_AFTER = 20; // recycle browser every 20 crawls to prevent memory leaks

async function getOrRecycleBrowser(): Promise<Browser> {
  crawlsSinceRecycle++;
  if (crawlsSinceRecycle >= RECYCLE_AFTER || !sharedBrowser?.isConnected()) {
    if (sharedBrowser) await sharedBrowser.close().catch(() => {});
    sharedBrowser = await chromium.launch({
      headless: true,
      args: ['--disable-dev-shm-usage', '--no-sandbox', '--disable-gpu'],
    });
    crawlsSinceRecycle = 0;
    logger.info('Playwright browser recycled');
  }
  return sharedBrowser;
}
```

### 6.3 CMS-Specific Crawl Strategies

```typescript
interface CmsStrategy {
  sitemapPaths: string[];
  imagePrefix?: string;
  skipPaths: string[];
  needsPlaywright: boolean;
  boilerplateHeavy?: boolean;
  cssPath?: (html: string) => string | null;
}

function getCmsStrategy(cmsType: string | null): CmsStrategy {
  switch (cmsType?.toLowerCase()) {
    case 'wordpress':
      return {
        sitemapPaths: ['/wp-sitemap.xml', '/sitemap.xml', '/sitemap_index.xml'],
        imagePrefix: '/wp-content/uploads/',
        skipPaths: ['/wp-admin', '/wp-login', '/wp-json', '/feed', '/wp-content/plugins', '?replytocom='],
        needsPlaywright: false,
        cssPath: (html) => html.match(/\/wp-content\/themes\/[\w-]+\/style\.css/)?.[0] ?? null,
      };
    case 'jimdo':
      return {
        sitemapPaths: ['/sitemap.xml'],
        imagePrefix: 'jimcdn.com',
        skipPaths: [],
        needsPlaywright: true,
      };
    case 'wix':
      return {
        sitemapPaths: ['/sitemap.xml'],
        imagePrefix: 'wixstatic.com',
        skipPaths: ['/_api/', '/_partials/', '/corvid-devmode/'],
        needsPlaywright: true,
        boilerplateHeavy: true,
      };
    case 'squarespace':
      return {
        sitemapPaths: ['/sitemap.xml'],
        skipPaths: ['/api/', '/config/'],
        needsPlaywright: false,
      };
    case 'webflow':
      return {
        sitemapPaths: ['/sitemap.xml'],
        skipPaths: [],
        needsPlaywright: false,
      };
    default:
      return {
        sitemapPaths: ['/sitemap.xml'],
        skipPaths: [],
        needsPlaywright: false,
      };
  }
}
```

### 6.4 Main Crawl Execution

```typescript
async function executeCrawl(leadId: string, config: Step2ToStep4): Promise<void> {
  const lead = await prisma.lead.findUnique({ where: { id: leadId } });
  if (!lead) return;
  
  // Check: Was lead disqualified by Step 3 while we were in queue?
  if (lead.status === 'FILTERED_OUT') {
    logger.info({ leadId }, 'Skip crawl: lead was disqualified by Step 3');
    await prisma.lead.update({ where: { id: leadId }, data: { crawlStatus: 'SKIPPED' } });
    await checkBothStepsComplete(leadId);
    return;
  }
  
  // SKIP strategy: no crawl needed
  if (config.crawlStrategy === 'SKIP') {
    logger.info({ leadId }, 'Skip crawl: crawlStrategy=SKIP (single-page site)');
    await prisma.lead.update({
      where: { id: leadId },
      data: {
        crawlStatus: 'SKIPPED',
        crawlCompletedAt: new Date(),
        contentCompleteness: lead.contentReadiness, // keep Step 2 estimate
      },
    });
    await emitEvent('lead.crawl_completed', { leadId, status: 'SKIPPED', pages: 0 });
    await checkBothStepsComplete(leadId);
    return;
  }
  
  const industryProfile = await loadIndustryProfile(config.industryTag);
  const cmsStrategy = getCmsStrategy(config.cmsType);
  const startTime = Date.now();
  let totalCostCents = 0;
  
  await prisma.lead.update({
    where: { id: leadId },
    data: { crawlStartedAt: new Date(), crawlStatus: 'RUNNING' },
  });
  
  // Determine max pages based on pipeline route
  let maxPages = 30;
  if (lead.pipelineRoute === 'ANALYSIS_ONLY' || lead.pipelineRoute === 'PHONE_ONLY') {
    maxPages = 8; // light crawl for non-FULL routes
  }
  
  // Determine throttle based on strategy
  const throttleMs = config.crawlStrategy === 'THROTTLED' ? 60000 : 2000;
  
  try {
    const browser = await getOrRecycleBrowser();
    
    // === PHASE 1: Discover all URLs ===
    const urls = await discoverUrls(config.actualUrl, config.priorityPages, cmsStrategy, maxPages);
    
    await emitEvent('lead.crawl_started', {
      leadId,
      strategy: config.crawlStrategy,
      estimatedPages: urls.length,
    });
    
    // === PHASE 2: Crawl pages ===
    const contentPackage = await prisma.contentPackage.create({
      data: { leadId },
    });
    
    const pages: ContentPageData[] = [];
    const allImages: ImageCandidate[] = [];
    let earlyExit = false;
    
    for (let i = 0; i < urls.length; i++) {
      const url = urls[i];
      
      // Check: Lead still valid? (might be disqualified mid-crawl)
      if (i > 0 && i % 5 === 0) {
        const current = await prisma.lead.findUnique({ where: { id: leadId }, select: { status: true } });
        if (current?.status === 'FILTERED_OUT') {
          logger.info({ leadId, page: i }, 'Crawl aborted: lead disqualified mid-crawl');
          break;
        }
      }
      
      const pageResult = await crawlSinglePage(browser, url, {
        throttleMs: i > 0 ? throttleMs : 0, // no throttle for first page
        cmsStrategy,
        isSpa: config.isSpa,
        downloadImages: lead.pipelineRoute === 'FULL', // only download images for full rebuild
      });
      
      if (pageResult) {
        pages.push(pageResult);
        if (pageResult.images) allImages.push(...pageResult.images);
        
        // Progress event
        await emitEvent('lead.crawl_progress', {
          leadId,
          pagesCrawled: i + 1,
          totalEstimated: urls.length,
          currentUrl: url,
          wordCount: pageResult.wordCount,
        });
      }
      
      // EARLY EXIT: after priority pages, check if we have enough
      if (i === config.priorityPages.length - 1 && config.crawlStrategy !== 'FAST') {
        const currentCompleteness = calculateContentCompleteness(pages, industryProfile);
        if (currentCompleteness > 0.7) {
          logger.info({ leadId, completeness: currentCompleteness, pagesCount: pages.length },
            'Early exit: sufficient content from priority pages');
          earlyExit = true;
          break;
        }
      }
    }
    
    // === PHASE 3: Download and classify images ===
    const downloadedImages: ContentImageData[] = [];
    if (lead.pipelineRoute === 'FULL') {
      for (const img of allImages) {
        if (!shouldDownloadImage(img)) continue;
        
        try {
          const response = await fetch(img.src, {
            signal: AbortSignal.timeout(10000),
            headers: { 'User-Agent': BROWSER_USER_AGENT },
          });
          if (!response.ok) continue;
          
          const buffer = Buffer.from(await response.arrayBuffer());
          if (buffer.length > 5 * 1024 * 1024) continue; // skip >5MB
          if (buffer.length < 500) continue; // skip tiny files
          
          const r2Key = `images/${leadId}/${Date.now()}_${img.filename}`;
          const r2Url = await uploadToR2(r2Key, buffer, img.contentType);
          
          downloadedImages.push({
            originalUrl: img.src,
            r2Url,
            alt: img.alt ?? '',
            width: img.width,
            height: img.height,
            fileSizeBytes: buffer.length,
            type: classifyImage(img),
            usable: img.width >= 200 && img.height >= 200,
            fromPageUrl: img.fromPage,
          });
          
          totalCostCents += 0.1; // ~€0.001 per image stored
        } catch { /* skip failed image */ }
      }
      
      // Extract favicon
      const faviconUrl = await extractFavicon(pages[0]?.html ?? '', config.actualUrl);
      if (faviconUrl) {
        downloadedImages.push({
          originalUrl: config.actualUrl + '/favicon.ico',
          r2Url: faviconUrl,
          alt: 'Favicon',
          width: 32, height: 32,
          type: 'FAVICON',
          usable: false,
          fileSizeBytes: 0,
        });
      }
    }
    
    // === PHASE 4: Extract brand assets ===
    const brandAssets = extractBrandAssets(pages, allImages, downloadedImages);
    
    // === PHASE 5: Extract navigation structure ===
    const navigationStructure = extractNavigation(pages[0]?.html ?? '', config.actualUrl);
    
    // === PHASE 6: Catalogue embeds, videos, structured data ===
    const embeds = pages.flatMap(p => p.iframes ?? []);
    const videos = pages.flatMap(p => p.videos ?? []);
    const structuredData = pages.flatMap(p => p.structuredData ?? []);
    const pdfs = []; // PDFs collected during crawl
    
    // === PHASE 7: SEO Analysis (site-wide) ===
    const seoAnalysis = calculateSeoAnalysis(pages);
    
    // === PHASE 8: Responsive design test ===
    const responsiveDesign = await testResponsiveDesign(browser, config.actualUrl);
    
    // === PHASE 9: AI Content Extraction ===
    let firmProfile = null;
    let extractionConfidence = 0;
    
    const relevantPages = pages.filter(p =>
      ['HOME', 'ABOUT', 'SERVICES', 'TEAM', 'PRICING', 'CONTACT', 'FAQ', 'REFERENCES'].includes(p.type)
    );
    
    if (relevantPages.length > 0) {
      // Quick-scan with Haiku to categorize pages (already done via URL patterns)
      // Deep extraction with Sonnet
      const extractionResult = await extractFirmProfile(
        relevantPages, 
        industryProfile, 
        lead.contentBriefData as any,
        lead
      );
      firmProfile = extractionResult.profile;
      extractionConfidence = extractionResult.confidence;
      totalCostCents += extractionResult.costCents;
    }
    
    // === PHASE 10: Content completeness calculation ===
    const contentCompleteness = calculateContentCompleteness(pages, industryProfile, firmProfile);
    
    // === PHASE 11: Consistency check with Step 1 ===
    const consistencyResult = checkConsistencyWithStep1(lead, pages, contentCompleteness);
    
    // === PHASE 12: Route validation ===
    const routeUpdate = checkRouteStillValid(lead, contentCompleteness);
    
    // === SAVE EVERYTHING ===
    
    // Save pages to DB
    for (const page of pages) {
      // Store clean HTML in R2 (not DB — too large)
      let htmlCleanR2Url: string | null = null;
      if (page.htmlClean) {
        htmlCleanR2Url = await uploadToR2(
          `html/${leadId}/${encodeURIComponent(new URL(page.url).pathname)}.html`,
          Buffer.from(page.htmlClean),
          'text/html'
        );
      }
      
      await prisma.contentPage.create({
        data: {
          contentPackageId: contentPackage.id,
          url: page.url,
          type: page.type as PageType,
          title: page.title,
          textContent: page.textContent?.substring(0, 50000), // limit for DB
          wordCount: page.wordCount,
          htmlCleanR2Url,
          screenshotDesktopUrl: page.screenshotDesktopUrl,
          screenshotMobileUrl: page.screenshotMobileUrl,
          loadTimeMs: page.loadTimeMs,
          metaDescription: page.metaDescription,
          h1Text: page.h1Text,
          h1Count: page.h1Count,
          hasCanonical: page.hasCanonical,
          hasOgTags: page.hasOgTags,
          hasStructuredData: (page.structuredData ?? []).length > 0,
          internalLinkCount: page.internalLinkCount,
          externalLinkCount: page.externalLinkCount,
          imagesOnPage: page.imagesOnPage,
          imagesWithAlt: page.imagesWithAlt,
          formAnalysis: page.formAnalysis,
        },
      });
    }
    
    // Save images to DB
    for (const img of downloadedImages) {
      await prisma.contentImage.create({
        data: {
          contentPackageId: contentPackage.id,
          originalUrl: img.originalUrl,
          r2Url: img.r2Url,
          alt: img.alt,
          width: img.width,
          height: img.height,
          fileSizeBytes: img.fileSizeBytes,
          type: img.type as ImageType,
          usable: img.usable,
          fromPageUrl: img.fromPageUrl,
        },
      });
    }
    
    // Update ContentPackage
    await prisma.contentPackage.update({
      where: { id: contentPackage.id },
      data: {
        firmProfile,
        extractionConfidence,
        extractionModel: 'claude-sonnet-4-20250514',
        extractionPromptVersion: `${config.industryTag}-v1.0`,
        navigationStructure,
        embeds,
        videos,
        pdfs,
        structuredData,
        seoAnalysis,
        accessibilityIssues: pages.flatMap(p => p.accessibilityIssues ?? []),
        responsiveDesign,
        cookieBannerText: pages[0]?.cookieBannerText ?? null,
        brandAssets,
        fontUsage: brandAssets.fontUsage,
        styleAnalysis: brandAssets.styleAnalysis,
        contentCompleteness,
        pagesAttempted: urls.length,
        pagesSuccessful: pages.length,
        crawlDurationMs: Date.now() - startTime,
        crawlCostCents: totalCostCents,
      },
    });
    
    // Update Lead
    await prisma.lead.update({
      where: { id: leadId },
      data: {
        contentCompleteness,
        crawlDurationMs: Date.now() - startTime,
        crawlCostCents: totalCostCents,
        crawlStatus: 'COMPLETED',
        crawlCompletedAt: new Date(),
        needsRescore: consistencyResult.needsRescore,
        rescoreReason: consistencyResult.issues.length > 0 ? consistencyResult.issues.join('; ') : null,
        ...(routeUpdate ? {
          pipelineRoute: routeUpdate.newRoute,
          skipSteps: routeUpdate.newSkipSteps ?? lead.skipSteps,
        } : {}),
      },
    });
    
    await emitEvent('lead.crawl_completed', {
      leadId,
      status: 'COMPLETED',
      pages: pages.length,
      contentCompleteness,
      costCents: totalCostCents,
      durationMs: Date.now() - startTime,
      earlyExit,
    });
    
    await updateLifecycle(leadId, 4, 'Website Crawl & Extraction', 'passed', totalCostCents);
    await checkBothStepsComplete(leadId);
    
  } catch (error) {
    logger.error({ leadId, error: String(error) }, 'Step 4 crawl failed');
    
    await prisma.lead.update({
      where: { id: leadId },
      data: {
        crawlStatus: 'FAILED',
        crawlCompletedAt: new Date(),
        crawlDurationMs: Date.now() - startTime,
        contentCompleteness: 0,
      },
    });
    
    await emitEvent('lead.crawl_completed', { leadId, status: 'FAILED', error: String(error) });
    await checkBothStepsComplete(leadId);
  }
}
```

### 6.5 URL Discovery

```typescript
async function discoverUrls(
  baseUrl: string,
  priorityPages: string[],
  cmsStrategy: CmsStrategy,
  maxPages: number
): Promise<string[]> {
  const discovered = new Set<string>();
  const baseHostname = new URL(baseUrl).hostname;
  
  // 1. Start with the homepage
  discovered.add(baseUrl);
  
  // 2. Add priority pages
  for (const path of priorityPages) {
    try {
      const fullUrl = new URL(path, baseUrl).href;
      if (new URL(fullUrl).hostname === baseHostname) discovered.add(fullUrl);
    } catch { /* invalid URL */ }
  }
  
  // 3. Try sitemap
  for (const sitemapPath of cmsStrategy.sitemapPaths) {
    try {
      const sitemapUrl = new URL(sitemapPath, baseUrl).href;
      const response = await fetch(sitemapUrl, {
        signal: AbortSignal.timeout(5000),
        headers: { 'User-Agent': BROWSER_USER_AGENT },
      });
      if (response.ok) {
        const xml = await response.text();
        const urls = xml.match(/<loc>(.*?)<\/loc>/g)?.map(m => m.replace(/<\/?loc>/g, '')) ?? [];
        for (const url of urls) {
          if (new URL(url).hostname === baseHostname) discovered.add(url);
        }
        break; // found a working sitemap
      }
    } catch { /* try next sitemap path */ }
  }
  
  // 4. Crawl homepage to discover navigation links
  try {
    const response = await fetch(baseUrl, {
      signal: AbortSignal.timeout(5000),
      headers: { 'User-Agent': BROWSER_USER_AGENT },
    });
    if (response.ok) {
      const html = await response.text();
      const $ = cheerio.load(html);
      $('nav a[href], header a[href], footer a[href], [role="navigation"] a[href]').each((_, el) => {
        const href = $(el).attr('href');
        if (href) {
          try {
            const fullUrl = new URL(href, baseUrl).href;
            if (new URL(fullUrl).hostname === baseHostname) discovered.add(fullUrl);
          } catch { /* invalid URL */ }
        }
      });
    }
  } catch { /* homepage fetch failed, use sitemap/priority only */ }
  
  // 5. Add mandatory pages (impressum, datenschutz)
  const MANDATORY_PATHS = ['/impressum', '/imprint', '/datenschutz', '/privacy'];
  for (const path of MANDATORY_PATHS) {
    discovered.add(new URL(path, baseUrl).href);
  }
  
  // 6. Filter out skip paths
  const filtered = [...discovered].filter(url => {
    const path = new URL(url).pathname.toLowerCase();
    return !cmsStrategy.skipPaths.some(skip => path.includes(skip.toLowerCase()));
  });
  
  // 7. Prioritize and limit
  return prioritizeUrls(filtered, priorityPages, maxPages);
}

function prioritizeUrls(urls: string[], priorityPages: string[], maxPages: number): string[] {
  const scored = urls.map(url => ({
    url,
    score: calculateUrlPriority(url, priorityPages),
  }));
  
  return scored
    .sort((a, b) => b.score - a.score)
    .slice(0, maxPages)
    .map(s => s.url);
}

function calculateUrlPriority(url: string, priorityPages: string[]): number {
  const path = new URL(url).pathname.toLowerCase();
  
  // Priority pages from Step 2 get highest score
  if (priorityPages.some(pp => url.includes(pp) || path.includes(pp))) return 200;
  
  // Mandatory legal pages
  if (path.match(/impressum|imprint/)) return 150;
  if (path.match(/datenschutz|privacy/)) return 145;
  
  // Core content pages
  if (path === '/' || path === '/index' || path === '/index.html') return 100;
  if (path.match(/ueber|about|wir/)) return 90;
  if (path.match(/leistung|service|angebot/)) return 85;
  if (path.match(/team|mitarbeiter/)) return 80;
  if (path.match(/preis|tarif|pricing|kosten/)) return 75;
  if (path.match(/kontakt|contact/)) return 70;
  if (path.match(/referenz|projekt|portfolio/)) return 65;
  if (path.match(/galerie|gallery|fotos|bilder/)) return 60;
  if (path.match(/faq|haeufig/)) return 40;
  if (path.match(/blog|news|aktuell|journal/)) return 30;
  if (path.match(/agb|terms/)) return 20;
  
  // Deep URLs = likely blog posts (lower priority)
  if (path.split('/').filter(Boolean).length > 2) return 10;
  
  return 15;
}
```

### 6.6 Single Page Crawl

```typescript
import { dismissCookieBanner, COOKIE_ACCEPT_SELECTORS } from '@webolution/shared/cookie-banner';
import { Readability } from '@mozilla/readability';
import { JSDOM } from 'jsdom';

async function crawlSinglePage(
  browser: Browser,
  url: string,
  config: { throttleMs: number; cmsStrategy: CmsStrategy; isSpa: boolean; downloadImages: boolean }
): Promise<ContentPageData | null> {
  // Throttle between pages
  if (config.throttleMs > 0) {
    await new Promise(resolve => setTimeout(resolve, config.throttleMs));
  }
  
  const context = await browser.newContext({
    viewport: { width: 1440, height: 900 },
    userAgent: BROWSER_USER_AGENT,
    locale: 'de-DE',
    timezoneId: 'Europe/Berlin',
  });
  
  // Set total timeout for this page
  const pageTimeout = setTimeout(async () => {
    await context.close().catch(() => {});
  }, 60000); // 60s max per page
  
  try {
    const page = await context.newPage();
    
    // Block heavy resources to speed up
    await page.route('**/*.{mp4,webm,ogg,avi,mp3,wav}', route => route.abort());
    
    // Navigate
    const startTime = Date.now();
    const response = await page.goto(url, {
      waitUntil: config.isSpa || config.cmsStrategy.needsPlaywright ? 'networkidle' : 'domcontentloaded',
      timeout: 30000,
    });
    
    if (!response || !response.ok()) return null;
    const loadTimeMs = Date.now() - startTime;
    
    // Dismiss cookie banner
    await dismissCookieBanner(page);
    
    // Wait for SPA content if needed
    if (config.isSpa) {
      await page.waitForFunction(() => document.body.innerText.length > 200, { timeout: 10000 }).catch(() => {});
    }
    
    // Scroll to trigger lazy-loaded content
    await scrollToLoadContent(page);
    
    // Ensure German version
    await ensureGermanVersion(page);
    
    // Get HTML and text
    const html = await page.content();
    const $ = cheerio.load(html);
    
    // Boilerplate removal
    const { text: cleanText, html: cleanHtml } = extractMainContent(html, url);
    
    // Detect page type
    const pageType = detectPageType(url, cleanText, $);
    
    // Check for login wall
    if (isLoginWall(html)) {
      return { url, type: 'OTHER', skipped: true, reason: 'login_wall' };
    }
    
    // Check for duplicate content
    // (handled in the main loop)
    
    // Take screenshots (full page)
    const screenshotDesktopUrl = await takeAndUploadScreenshot(page, `screenshots/${url.replace(/[^a-z0-9]/gi, '_')}_desktop.jpg`, false);
    
    // Mobile screenshot
    await page.setViewportSize({ width: 375, height: 812 });
    await page.waitForTimeout(500);
    const screenshotMobileUrl = await takeAndUploadScreenshot(page, `screenshots/${url.replace(/[^a-z0-9]/gi, '_')}_mobile.jpg`, false);
    
    // Collect images from this page
    const images: ImageCandidate[] = [];
    if (config.downloadImages) {
      $('img[src]').each((_, el) => {
        const src = $(el).attr('src');
        const absUrl = src ? new URL(src, url).href : null;
        if (absUrl) {
          images.push({
            src: absUrl,
            alt: $(el).attr('alt') ?? '',
            width: parseInt($(el).attr('width') ?? '0') || 0,
            height: parseInt($(el).attr('height') ?? '0') || 0,
            filename: absUrl.split('/').pop()?.split('?')[0] ?? 'unknown',
            contentType: 'image/jpeg', // will be determined on download
            fromPage: url,
            position: $(el).closest('header').length ? 'header' : 
                      $(el).closest('footer').length ? 'footer' : 'body',
          });
        }
      });
    }
    
    // Detect iframes
    const iframes = await catalogueIframes($);
    
    // Detect videos
    const videos = detectVideos(html, iframes);
    
    // Extract structured data
    const structuredData = extractStructuredData(html);
    
    // Page-level SEO
    const metaDescription = $('meta[name="description"]').attr('content') ?? null;
    const h1Elements = $('h1');
    const hasCanonical = !!$('link[rel="canonical"]').length;
    const hasOgTags = !!$('meta[property^="og:"]').length;
    const internalLinks = $('a[href]').filter((_, el) => {
      try { return new URL($(el).attr('href')!, url).hostname === new URL(url).hostname; } catch { return false; }
    }).length;
    const externalLinks = $('a[href^="http"]').filter((_, el) => {
      try { return new URL($(el).attr('href')!).hostname !== new URL(url).hostname; } catch { return false; }
    }).length;
    const imagesOnPage = $('img').length;
    const imagesWithAlt = $('img[alt]:not([alt=""])').length;
    
    // Cookie banner text (only from first page)
    const cookieBannerText = await extractCookieBannerText($);
    
    // Form analysis
    let formAnalysis = null;
    if ($('form').length > 0) {
      formAnalysis = analyzeForm($('form').first(), url, html);
    }
    
    // Accessibility quick-check
    const accessibilityIssues = checkAccessibility($, html);
    
    return {
      url,
      type: pageType,
      title: $('title').text().trim() || null,
      textContent: cleanText,
      htmlClean: cleanHtml,
      html,
      wordCount: cleanText.split(/\s+/).filter(w => w.length > 1).length,
      screenshotDesktopUrl,
      screenshotMobileUrl,
      loadTimeMs,
      metaDescription,
      h1Text: h1Elements.first().text().trim() || null,
      h1Count: h1Elements.length,
      hasCanonical,
      hasOgTags,
      internalLinkCount: internalLinks,
      externalLinkCount: externalLinks,
      imagesOnPage,
      imagesWithAlt,
      images,
      iframes,
      videos,
      structuredData,
      cookieBannerText,
      formAnalysis,
      accessibilityIssues,
    };
  } catch (error) {
    logger.warn({ url, error: String(error) }, 'Page crawl failed');
    return null;
  } finally {
    clearTimeout(pageTimeout);
    await context.close().catch(() => {});
  }
}

async function takeAndUploadScreenshot(page: Page, key: string, fullPage: boolean = true): Promise<string> {
  const buffer = await page.screenshot({
    type: 'png',
    fullPage,
    timeout: 10000,
  });
  return await uploadToR2(key, buffer, 'image/png');
}
```

### 6.7 Boilerplate Removal

```typescript
function extractMainContent(html: string, url: string): { text: string; html: string } {
  // Try Mozilla Readability first
  try {
    const dom = new JSDOM(html, { url });
    const reader = new Readability(dom.window.document);
    const article = reader.parse();
    
    if (article && article.textContent.trim().length > 100) {
      return { text: article.textContent.trim(), html: article.content };
    }
  } catch { /* Readability failed, use fallback */ }
  
  // Fallback: manual cleanup
  const $ = cheerio.load(html);
  $('nav, header, footer, aside, script, style, noscript, iframe').remove();
  $('[class*="nav"], [class*="menu"], [class*="footer"], [class*="sidebar"], [class*="cookie"], [class*="banner"]').remove();
  $('[id*="nav"], [id*="menu"], [id*="footer"], [id*="sidebar"]').remove();
  
  const main = $('main, [role="main"], article, .content, #content').first();
  const target = main.length ? main : $('body');
  
  return {
    text: target.text().replace(/\s+/g, ' ').trim(),
    html: target.html() ?? '',
  };
}
```

### 6.8 Page Type Detection

```typescript
function detectPageType(url: string, text: string, $: cheerio.CheerioAPI): PageType {
  const path = new URL(url).pathname.toLowerCase();
  const textLC = text.toLowerCase();
  
  if (path === '/' || path === '/index' || path === '/index.html') return 'HOME';
  if (path.match(/impressum|imprint|datenschutz|privacy|agb|terms/)) return 'LEGAL';
  if (path.match(/ueber|about|wir\b|über/)) return 'ABOUT';
  if (path.match(/leistung|service|angebot|was-wir/)) return 'SERVICES';
  if (path.match(/team|mitarbeiter|unser-team/)) return 'TEAM';
  if (path.match(/preis|tarif|pricing|kosten/)) return 'PRICING';
  if (path.match(/galerie|gallery|portfolio|fotos|bilder|arbeiten/)) return 'GALLERY';
  if (path.match(/blog|news|aktuell|journal|beitr/)) return 'BLOG';
  if (path.match(/kontakt|contact|anfahrt|standort/)) return 'CONTACT';
  if (path.match(/faq|haeufig|fragen/)) return 'FAQ';
  if (path.match(/referenz|projekt|kunden|partner/)) return 'REFERENCES';
  
  // Content-based fallback
  if (textLC.includes('über uns') || textLC.includes('über mich') || textLC.includes('mein werdegang')) return 'ABOUT';
  if (textLC.includes('unsere leistungen') || textLC.includes('meine angebote')) return 'SERVICES';
  if ($('form').length > 0 && textLC.includes('kontakt')) return 'CONTACT';
  
  return 'OTHER';
}
```

### 6.9 AI Content Extraction

```typescript
const INDUSTRY_EXTRACTION_PROMPTS: Record<string, string> = {
  coaches: `You are extracting structured data from a COACHING/CONSULTING website in Germany.

EXTRACT these fields from the website content. For EACH field provide:
- value: the extracted data (null if not found)
- confidence: 0.0-1.0 (1.0 = explicitly stated, 0.5 = inferred, 0.0 = not found)
- source_page: which URL contained this info

FIELDS:
- firm_name: Official business name
- slogan: Tagline or motto (from H1 or hero section)
- description: Business description (2-5 sentences)
- coaching_type: Business/Life/Health/Career/Leadership/Systemic/etc.
- methodology: Methods used (NLP, systemic, cognitive behavioral, etc.)
- target_audience: Who they serve (executives, entrepreneurs, teams, etc.)
- services: Array of { name, description } — each service offered
- usps: Array of unique selling points
- team: Array of { name, role, bio, photo_url } for each person
- founding_year: When the business was established
- certifications: Professional qualifications (IHK, ICF, DBVC, etc.)
- memberships: Professional associations (dvct, ICF, etc.)
- testimonials: Array of { text, author, role } from the website (NOT Google reviews)
- prices: Object with service names as keys and price info as values
- opening_hours: Object with days as keys and hours as values
- contact: { phone, email, address }
- booking_method: How clients book (Calendly, form, phone, etc.)
- lead_magnet: Free offers (eBook, checklist, free session, etc.)
- social_links: { instagram, facebook, linkedin, youtube, podcast }

CRITICAL RULES:
1. Extract ONLY what is EXPLICITLY present in the text. NEVER invent data.
2. Every field MUST have a source_page URL. No source = null value.
3. Testimonials from the WEBSITE only. Do NOT use Google reviews.
4. Prices must be extracted EXACTLY as shown. Mark all prices as "needs_confirmation": true.
5. If a field appears on multiple pages, use the MOST DETAILED version.`,

  friseure: `You are extracting structured data from a HAIR SALON / BARBERSHOP website in Germany.

EXTRACT these fields:
- firm_name, slogan, description (same as above)
- salon_type: Traditional salon / Premium salon / Barbershop / Budget salon
- services: Array of { category, name, price } — structured price list
  Categories: Damen, Herren, Kinder, Coloration, Styling, Extensions, Bart, Sonstige
- stylists: Array of { name, specialization, photo_url }
- product_brands: Hair product brands used or sold
- gallery_images: URLs of work photos (before/after, styling results)
- booking_system: Which platform (Treatwell, Booksy, Shore, own?)
- opening_hours, contact, social_links (same as above)
- testimonials (from website only, NOT Google)

INDUSTRY NOTES:
- Prices are often in tables or lists: "Damen Waschen+Schneiden+Föhnen ab €45"
- Barbershops have distinct services: Bart trimmen, Rasur, Bartpflege
- Look for "Unsere Marken" or product logos for brand detection
- Instagram is often MORE relevant than the website for salon portfolios`,
};

// Generic fallback for industries without a specific prompt
const GENERIC_EXTRACTION_PROMPT = `You are extracting structured data from a German small business website.

EXTRACT: firm_name, slogan, description, services[], usps[], team[], 
founding_year, certifications[], testimonials[], prices, opening_hours, 
contact{phone, email, address}, social_links, booking_method.

Rules: Extract ONLY explicit data. NEVER invent. source_page required for every field.`;

async function extractFirmProfile(
  pages: ContentPageData[],
  industryProfile: IndustryProfile,
  contentBrief: any,
  lead: Lead
): Promise<{ profile: any; confidence: number; costCents: number }> {
  
  // Build page context
  const relevantPages = pages
    .filter(p => p.textContent && p.wordCount > 30)
    .sort((a, b) => (PAGE_TYPE_ORDER[a.type] ?? 10) - (PAGE_TYPE_ORDER[b.type] ?? 10));
  
  const context = relevantPages.map(p =>
    `\n=== PAGE: ${p.type} (${p.url}) ===\n${p.textContent}`
  ).join('\n');
  
  // Trim to token budget (~40K tokens ≈ ~30K words)
  const trimmedContext = context.length > 120000 ? context.substring(0, 120000) + '\n[TRUNCATED]' : context;
  
  const prompt = INDUSTRY_EXTRACTION_PROMPTS[industryProfile.tag] ?? GENERIC_EXTRACTION_PROMPT;
  
  const response = await anthropic.messages.create({
    model: 'claude-sonnet-4-20250514',
    max_tokens: 4000,
    temperature: 0,
    messages: [{
      role: 'user',
      content: `${prompt}

WEBSITE CONTENT:
${trimmedContext}

GOOGLE DATA (for cross-reference only, do NOT copy verbatim):
- Business Name: ${lead.firmName}
- Rating: ${lead.googleRating}/5 (${lead.googleReviewCount} reviews)
- Category: ${lead.googleCategory}
- Description: ${lead.googleBusinessDescription ?? 'none'}

Respond ONLY with a JSON object. No markdown, no backticks, no preamble.`,
    }],
  });
  
  const text = response.content[0]?.type === 'text' ? response.content[0].text : '';
  const costCents = Math.round((response.usage?.input_tokens ?? 0) * 0.0003 + (response.usage?.output_tokens ?? 0) * 0.0015);
  
  try {
    const cleaned = text.replace(/```json\n?|```/g, '').trim();
    const profile = JSON.parse(cleaned);
    
    // Validate extraction
    const validated = validateExtraction(profile, lead);
    
    return {
      profile: validated.profile,
      confidence: validated.confidence,
      costCents,
    };
  } catch (error) {
    logger.warn({ leadId: lead.id, error: String(error) }, 'AI extraction parse failed');
    return { profile: null, confidence: 0, costCents };
  }
}

function validateExtraction(extracted: any, lead: Lead): { profile: any; confidence: number } {
  if (!extracted) return { profile: null, confidence: 0 };
  
  const issues: string[] = [];
  
  // Check: firm_name plausibility
  if (extracted.firm_name && !fuzzyMatch(extracted.firm_name, lead.firmName, 0.4)) {
    issues.push('firm_name_mismatch');
  }
  
  // Check: team size plausibility
  if (extracted.team?.length > 20 && lead.estimatedSize === 'SOLO') {
    issues.push('implausible_team_size');
    extracted.team = extracted.team.slice(0, 3);
  }
  
  // Check: founding year plausibility
  if (extracted.founding_year && (extracted.founding_year < 1900 || extracted.founding_year > new Date().getFullYear())) {
    issues.push('implausible_founding_year');
    extracted.founding_year = null;
  }
  
  // Check: source_page exists for key fields
  for (const field of ['services', 'team', 'testimonials']) {
    if (extracted[field]?.length > 0) {
      const hasSource = extracted[field].some((item: any) => item.source_page);
      if (!hasSource) issues.push(`no_source_for_${field}`);
    }
  }
  
  // Mark all prices as needs_confirmation
  if (extracted.prices) {
    for (const key of Object.keys(extracted.prices)) {
      if (typeof extracted.prices[key] === 'object') {
        extracted.prices[key].needs_confirmation = true;
      }
    }
  }
  
  // Calculate confidence
  const fieldCount = Object.keys(extracted).filter(k => extracted[k] !== null && extracted[k] !== undefined).length;
  const totalFields = 15; // expected field count
  const baseConfidence = fieldCount / totalFields;
  const penalty = issues.length * 0.1;
  
  return {
    profile: extracted,
    confidence: Math.max(0, Math.min(1, baseConfidence - penalty)),
  };
}

const PAGE_TYPE_ORDER: Record<string, number> = {
  HOME: 1, ABOUT: 2, SERVICES: 3, TEAM: 4, PRICING: 5,
  GALLERY: 6, REFERENCES: 7, FAQ: 8, BLOG: 9, CONTACT: 10, LEGAL: 11, OTHER: 12,
};

function fuzzyMatch(a: string, b: string, threshold: number): boolean {
  const aLC = a.toLowerCase().trim();
  const bLC = b.toLowerCase().trim();
  if (aLC === bLC) return true;
  if (aLC.includes(bLC) || bLC.includes(aLC)) return true;
  // Simple word overlap check
  const aWords = new Set(aLC.split(/\s+/));
  const bWords = new Set(bLC.split(/\s+/));
  const overlap = [...aWords].filter(w => bWords.has(w)).length;
  return overlap / Math.max(aWords.size, bWords.size) >= threshold;
}
```

---

## 7. ORCHESTRATOR: SYNCHRONIZATION + MERGE

After BOTH Step 3 and Step 4 complete (or skip), the Orchestrator runs a MERGE step before dispatching to Steps 5+6.

### 7.1 Completion Check

```typescript
async function checkBothStepsComplete(leadId: string): Promise<void> {
  const lead = await prisma.lead.findUnique({
    where: { id: leadId },
    select: {
      id: true,
      enrichmentStatus: true,
      crawlStatus: true,
      crawlStrategy: true,
      status: true,
      campaignId: true,
    },
  });
  if (!lead) return;
  
  // Was lead disqualified?
  if (lead.status === 'FILTERED_OUT') return;
  
  const enrichDone = ['COMPLETED', 'FAILED', 'SKIPPED'].includes(lead.enrichmentStatus ?? '');
  const crawlDone = ['COMPLETED', 'FAILED', 'SKIPPED'].includes(lead.crawlStatus ?? '') ||
                    lead.crawlStrategy === 'SKIP';
  
  if (enrichDone && crawlDone) {
    logger.info({ leadId }, 'Both Step 3+4 complete → running merge');
    await runPostCrawlMerge(leadId);
  }
}
```

### 7.2 Post-Crawl Merge

```typescript
async function runPostCrawlMerge(leadId: string): Promise<void> {
  const lead = await prisma.lead.findUnique({ where: { id: leadId } });
  if (!lead) return;
  
  const contentPackage = await prisma.contentPackage.findUnique({
    where: { leadId },
    include: { pages: true, images: true },
  });
  
  // === MERGE 1: Update Content Brief with REAL data ===
  const brief = (lead.contentBriefData as any) ?? {};
  
  if (contentPackage?.firmProfile) {
    const fp = contentPackage.firmProfile as any;
    
    // Owner name: Website team page > Impressum > Google
    if (fp.team?.[0]?.name) {
      brief.ownerName = { value: fp.team[0].name, source: 'website', confidence: 0.95 };
    }
    
    // Qualifications: union of Impressum + website
    if (fp.certifications?.length) {
      const existing = brief.qualifications?.value ?? [];
      brief.qualifications = {
        value: [...new Set([...existing, ...fp.certifications.map((c: any) => c.value ?? c)])],
        source: 'merged',
        confidence: 0.85,
      };
    }
    
    // Services: website is authoritative
    if (fp.services?.length) {
      brief.services = { value: fp.services, source: 'website', confidence: 0.9 };
    }
    
    // Testimonials: deduplicate website + Google
    if (fp.testimonials?.length) {
      const googleTestimonials = brief.testimonialCandidates?.value ?? [];
      brief.testimonialCandidates = {
        value: deduplicateTestimonials(fp.testimonials, googleTestimonials),
        source: 'merged',
        confidence: 0.85,
      };
    }
    
    // Founding year: website > Impressum > copyright
    if (fp.founding_year) {
      brief.foundingYear = { value: fp.founding_year, source: 'website', confidence: 0.9 };
    }
  }
  
  // Update content readiness with REAL data
  brief.contentReadiness = contentPackage?.contentCompleteness ?? brief.contentReadiness ?? 0;
  
  // Image needs: now we know exactly how many usable photos exist
  if (contentPackage) {
    const usablePhotos = await prisma.contentImage.count({
      where: { contentPackageId: contentPackage.id, usable: true, type: 'PHOTO' },
    });
    brief.imageNeeds = {
      ownPhotosAvailable: usablePhotos + (lead.googlePhotosCount ?? 0),
      // ... recalculate based on real data
    };
  }
  
  // === MERGE 2: Deduplicate testimonials ===
  // (handled above)
  
  // === MERGE 3: Check if pipeline route should change ===
  let routeUpdate: any = null;
  if (lead.pipelineRoute === 'FULL' && (contentPackage?.contentCompleteness ?? 0) < 0.2) {
    routeUpdate = {
      pipelineRoute: 'NURTURE',
      skipSteps: [8, 9, 10, 11, 12],
    };
    logger.info({ leadId, completeness: contentPackage?.contentCompleteness },
      'Route changed FULL→NURTURE: content too thin after crawl');
  }
  
  // === MERGE 4: Final status update ===
  const newStatus = (lead.enrichmentStatus === 'FAILED' && lead.crawlStatus === 'FAILED')
    ? 'FILTERED_OUT'  // both failed → can't continue
    : 'ENRICHED_AND_CRAWLED';
  
  await prisma.lead.update({
    where: { id: leadId },
    data: {
      status: newStatus as any,
      contentBriefData: brief,
      ...(routeUpdate ?? {}),
    },
  });
  
  // === DISPATCH to Steps 5+6 ===
  if (newStatus !== 'FILTERED_OUT') {
    // Step 5: Quality Review (needs content from Step 4)
    await reviewQueue.add('review', { leadId }, {
      priority: Math.round(1000 - (lead.pipelinePriority ?? 500)),
    });
    
    // Step 6: Competitor Benchmarking
    // NOTE: Step 6 may have already started via the early competitors.found event.
    // If it hasn't started yet (no competitor_ids), start it now.
    if ((lead.competitorPlaceIds ?? []).length > 0) {
      await competitorQueue.add('benchmark', { leadId }, {
        priority: Math.round(1000 - (lead.pipelinePriority ?? 500)),
      });
    }
    
    await emitEvent('lead.merge_completed', {
      leadId,
      status: newStatus,
      contentCompleteness: contentPackage?.contentCompleteness ?? 0,
      routeChanged: !!routeUpdate,
      newRoute: routeUpdate?.pipelineRoute ?? lead.pipelineRoute,
    });
  }
  
  await updateLifecycle(leadId, 'merge', 'Post-Crawl Merge', newStatus === 'FILTERED_OUT' ? 'failed' : 'passed', 0);
}

function deduplicateTestimonials(websiteTestimonials: any[], googleReviews: any[]): any[] {
  const all: any[] = [];
  const seenTexts = new Set<string>();
  
  for (const t of websiteTestimonials) {
    const key = (t.text ?? '').toLowerCase().substring(0, 50);
    if (key.length > 10 && !seenTexts.has(key)) {
      seenTexts.add(key);
      all.push({ ...t, source: 'website' });
    }
  }
  
  for (const r of googleReviews) {
    const key = (r.text ?? '').toLowerCase().substring(0, 50);
    if (key.length > 10 && !seenTexts.has(key)) {
      seenTexts.add(key);
      all.push({ ...r, source: 'google' });
    }
  }
  
  return all;
}
```

---

## 8. HELPER FUNCTIONS (used by Step 4)

### 8.1 Image Handling

```typescript
function shouldDownloadImage(img: ImageCandidate): boolean {
  if (img.width > 0 && img.width < 50) return false;
  if (img.height > 0 && img.height < 50) return false;
  
  const SKIP_DOMAINS = ['google-analytics.com', 'facebook.com', 'doubleclick.net', 'pixel.', 'googletagmanager.com', 'adservice.google'];
  if (SKIP_DOMAINS.some(d => img.src.includes(d))) return false;
  
  const STOCK_DOMAINS = ['shutterstock', 'istock', 'gettyimages'];
  if (STOCK_DOMAINS.some(d => img.src.includes(d))) return false;
  
  if (img.src.endsWith('.svg') && img.width < 100 && img.height < 100) return false;
  if (img.src.startsWith('data:') && img.src.length < 5000) return false;
  
  return true;
}

function classifyImage(img: ImageCandidate): ImageType {
  const filename = (img.filename ?? '').toLowerCase();
  const alt = (img.alt ?? '').toLowerCase();
  
  if (filename.match(/logo/i) || img.position === 'header') return 'LOGO';
  if (filename.match(/zertifi|certificate|badge|siegel|award|iso|tüv|ihk/i) || 
      alt.match(/zertifi|siegel|award/i)) return 'CERTIFICATION';
  if (img.width < 100 && img.height < 100) return 'ICON';
  if (filename.match(/stock|shutterstock|istock|adobe/i)) return 'STOCK';
  if (img.width >= 400 && img.height >= 300) return 'PHOTO';
  return 'UNKNOWN';
}

async function extractFavicon(html: string, baseUrl: string): Promise<string | null> {
  const $ = cheerio.load(html);
  const sources = [
    $('link[rel="apple-touch-icon"]').attr('href'),
    $('link[rel="icon"][type="image/png"]').attr('href'),
    $('link[rel="icon"][type="image/svg+xml"]').attr('href'),
    $('link[rel="shortcut icon"]').attr('href'),
    $('link[rel="icon"]').attr('href'),
  ].filter(Boolean);
  
  for (const src of sources) {
    try {
      const fullUrl = new URL(src!, baseUrl).href;
      const response = await fetch(fullUrl, { signal: AbortSignal.timeout(3000) });
      if (response.ok) {
        const buffer = Buffer.from(await response.arrayBuffer());
        if (buffer.length > 100) {
          return await uploadToR2(`favicons/${Date.now()}.png`, buffer, 'image/png');
        }
      }
    } catch { /* skip */ }
  }
  return null;
}
```

### 8.2 Brand Asset Extraction

```typescript
function extractBrandAssets(
  pages: ContentPageData[],
  imagesCandidates: ImageCandidate[],
  downloadedImages: ContentImageData[]
): any {
  const homepageHtml = pages.find(p => p.type === 'HOME')?.html ?? '';
  const allCss = ''; // extracted during crawl (see 8.3)
  
  // Logo: find from downloaded images
  const logos = downloadedImages.filter(img => img.type === 'LOGO');
  const bestLogo = logos.sort((a, b) => (b.width * b.height) - (a.width * a.height))[0];
  
  // Colors
  const colors = extractColors(allCss, homepageHtml);
  
  // Fonts
  const fontUsage = extractFontUsage(homepageHtml, allCss);
  
  // Style analysis
  const styleAnalysis = {
    borderRadius: detectBorderRadius(allCss),
    shadows: allCss.includes('box-shadow') ? 'has_shadows' : 'flat',
    hasAnimations: allCss.includes('animation') || allCss.includes('@keyframes'),
    layoutType: allCss.includes('display: grid') || allCss.includes('display:grid') ? 'css_grid' :
                allCss.includes('display: flex') || allCss.includes('display:flex') ? 'flexbox' : 'traditional',
    isMinimalist: !allCss.includes('gradient') && !allCss.includes('box-shadow') && !allCss.includes('text-shadow'),
  };
  
  return {
    logoUrl: bestLogo?.r2Url ?? null,
    logoType: bestLogo ? (bestLogo.r2Url.endsWith('.svg') ? 'svg' : 'png') : null,
    faviconUrl: downloadedImages.find(img => img.type === 'FAVICON')?.r2Url ?? null,
    primaryColor: colors.primaryColor,
    secondaryColor: colors.secondaryColor,
    accentColor: colors.accentColor,
    primaryFont: fontUsage.primaryFont,
    secondaryFont: fontUsage.secondaryFont,
    fontUsage,
    styleAnalysis,
  };
}

function extractColors(css: string, html: string): { primaryColor: string | null; secondaryColor: string | null; accentColor: string | null } {
  const allColors: string[] = [];
  
  // CSS custom properties
  const customProps = css.matchAll(/--[\w-]+:\s*(#[0-9a-fA-F]{3,8})/g);
  for (const match of customProps) allColors.push(match[1]);
  
  // Background colors
  const bgColors = css.matchAll(/background(?:-color)?:\s*(#[0-9a-fA-F]{3,8})/g);
  for (const match of bgColors) allColors.push(match[1]);
  
  // Text colors
  const textColors = css.matchAll(/(?:^|[{;\s])color:\s*(#[0-9a-fA-F]{3,8})/gm);
  for (const match of textColors) allColors.push(match[1]);
  
  // Inline styles in HTML
  const inlineColors = html.matchAll(/style="[^"]*(?:color|background):\s*(#[0-9a-fA-F]{3,8})/gi);
  for (const match of inlineColors) allColors.push(match[1]);
  
  // Count and sort
  const counts: Record<string, number> = {};
  for (const c of allColors) {
    const normalized = c.toLowerCase();
    counts[normalized] = (counts[normalized] ?? 0) + 1;
  }
  
  const isNeutral = (c: string) =>
    ['#fff', '#ffffff', '#000', '#000000', '#333', '#333333', '#666', '#666666',
     '#999', '#999999', '#ccc', '#cccccc', '#eee', '#eeeeee', '#f5f5f5', '#fafafa',
     '#f8f8f8', '#f0f0f0', '#e0e0e0', '#ddd', '#dddddd'].includes(c);
  
  const brandColors = Object.entries(counts)
    .filter(([c]) => !isNeutral(c))
    .sort(([, a], [, b]) => b - a);
  
  return {
    primaryColor: brandColors[0]?.[0] ?? null,
    secondaryColor: brandColors[1]?.[0] ?? null,
    accentColor: brandColors[2]?.[0] ?? null,
  };
}

function extractFontUsage(html: string, css: string): any {
  const fonts: any[] = [];
  
  const googleFontMatch = html.matchAll(/fonts\.googleapis\.com\/css2?\?family=([^&"]+)/g);
  for (const match of googleFontMatch) {
    const families = match[1].split('|').map(f => decodeURIComponent(f.split(':')[0].replace(/\+/g, ' ')));
    families.forEach(f => fonts.push({ name: f, source: 'google_fonts', license: 'free' }));
  }
  
  if (html.includes('use.typekit.net') || html.includes('adobe.com/fonts')) {
    fonts.push({ name: 'Adobe Fonts', source: 'adobe_fonts', license: 'paid' });
  }
  
  const fontFaces = css.matchAll(/@font-face\s*{[^}]*font-family:\s*['"]?([^'";\n}]+)/g);
  for (const match of fontFaces) {
    if (!fonts.some(f => f.name === match[1])) {
      fonts.push({ name: match[1], source: 'self_hosted', license: 'unknown' });
    }
  }
  
  return {
    fonts,
    primaryFont: fonts[0]?.name ?? null,
    secondaryFont: fonts[1]?.name ?? null,
    usesGoogleFonts: fonts.some(f => f.source === 'google_fonts'),
    usesAdobeFonts: fonts.some(f => f.source === 'adobe_fonts'),
  };
}

function detectBorderRadius(css: string): string {
  const matches = css.matchAll(/border-radius:\s*(\d+)/g);
  const values: number[] = [];
  for (const m of matches) values.push(parseInt(m[1]));
  if (values.length === 0) return 'sharp';
  const avg = values.reduce((a, b) => a + b, 0) / values.length;
  return avg > 20 ? 'pill' : avg > 5 ? 'rounded' : 'sharp';
}
```

### 8.3 SEO Analysis

```typescript
function calculateSeoAnalysis(pages: ContentPageData[]): any {
  const analysis: any = {
    pages: [],
    hasSitemap: false, // set during URL discovery
    duplicateTitles: [],
    duplicateDescriptions: [],
    missingTitles: [],
    missingDescriptions: [],
    thinContentPages: [],
    brokenInternalLinks: [],
    seoScore: 0,
    topIssues: [],
  };
  
  const titleMap = new Map<string, string[]>();
  const descMap = new Map<string, string[]>();
  
  for (const page of pages) {
    // Track duplicates
    if (page.title) {
      const existing = titleMap.get(page.title) ?? [];
      existing.push(page.url);
      titleMap.set(page.title, existing);
    } else {
      analysis.missingTitles.push(page.url);
    }
    
    if (page.metaDescription) {
      const existing = descMap.get(page.metaDescription) ?? [];
      existing.push(page.url);
      descMap.set(page.metaDescription, existing);
    } else {
      analysis.missingDescriptions.push(page.url);
    }
    
    if (page.wordCount < 100) {
      analysis.thinContentPages.push(page.url);
    }
  }
  
  // Find duplicates (same title/description on multiple pages)
  for (const [title, urls] of titleMap) {
    if (urls.length > 1) analysis.duplicateTitles.push({ title, urls });
  }
  for (const [desc, urls] of descMap) {
    if (urls.length > 1) analysis.duplicateDescriptions.push({ desc, urls });
  }
  
  // Calculate SEO score
  let score = 100;
  if (analysis.missingTitles.length > 0) score -= analysis.missingTitles.length * 5;
  if (analysis.missingDescriptions.length > 0) score -= analysis.missingDescriptions.length * 3;
  if (analysis.duplicateTitles.length > 0) score -= analysis.duplicateTitles.length * 4;
  if (analysis.thinContentPages.length > 0) score -= analysis.thinContentPages.length * 3;
  if (!analysis.hasSitemap) score -= 5;
  analysis.seoScore = Math.max(0, score);
  
  // Top issues
  if (analysis.missingTitles.length > 0) analysis.topIssues.push(`${analysis.missingTitles.length} Seiten ohne Title-Tag`);
  if (analysis.missingDescriptions.length > 0) analysis.topIssues.push(`${analysis.missingDescriptions.length} Seiten ohne Meta-Description`);
  if (analysis.duplicateTitles.length > 0) analysis.topIssues.push(`${analysis.duplicateTitles.length} doppelte Seitentitel`);
  if (analysis.thinContentPages.length > 0) analysis.topIssues.push(`${analysis.thinContentPages.length} Seiten mit wenig Content (<100 Wörter)`);
  
  return analysis;
}
```

### 8.4 Content Completeness (Industry-Specific)

```typescript
function calculateContentCompleteness(
  pages: ContentPageData[],
  industryProfile: IndustryProfile,
  firmProfile?: any
): number {
  const weights = industryProfile.contentCompleteness ?? {
    coreIdentity: 0.25,
    contactInfo: 0.15,
    visualAssets: 0.15,
    socialProof: 0.15,
    industrySpecific: 0.10,
    brandAssets: 0.10,
    contentVolume: 0.10,
  };
  
  let score = 0;
  
  // Core Identity (Name, Description, Services)
  const hasName = !!firmProfile?.firm_name;
  const hasDescription = !!(firmProfile?.description && firmProfile.description.length > 50);
  const hasServices = (firmProfile?.services?.length ?? 0) >= 1;
  const coreCount = [hasName, hasDescription, hasServices].filter(Boolean).length;
  score += (coreCount / 3) * weights.coreIdentity;
  
  // Contact Info
  const hasPhone = !!firmProfile?.contact?.phone;
  const hasEmail = !!firmProfile?.contact?.email;
  const hasAddress = !!firmProfile?.contact?.address;
  const contactCount = [hasPhone, hasEmail, hasAddress].filter(Boolean).length;
  score += (contactCount / 3) * weights.contactInfo;
  
  // Visual Assets
  const photoCount = pages.reduce((sum, p) => sum + (p.images?.filter(i => i.width >= 400)?.length ?? 0), 0);
  score += Math.min(1, photoCount / 5) * weights.visualAssets;
  
  // Social Proof
  const testimonialCount = (firmProfile?.testimonials?.length ?? 0);
  score += Math.min(1, testimonialCount / 3) * weights.socialProof;
  
  // Industry-specific required fields
  const requiredFields = industryProfile.enrichment?.requiredFields ?? [];
  if (requiredFields.length > 0 && firmProfile) {
    const filled = requiredFields.filter(f => firmProfile[f] !== null && firmProfile[f] !== undefined).length;
    score += (filled / requiredFields.length) * weights.industrySpecific;
  }
  
  // Brand Assets
  const hasLogo = pages.some(p => p.images?.some(i => classifyImage(i) === 'LOGO'));
  score += (hasLogo ? 1 : 0) * weights.brandAssets;
  
  // Content Volume
  const totalWords = pages.reduce((sum, p) => sum + p.wordCount, 0);
  score += Math.min(1, totalWords / 1000) * (weights.contentVolume ?? 0.10);
  
  return Math.min(1, Math.max(0, score));
}
```

### 8.5 Responsive Design Test

```typescript
async function testResponsiveDesign(browser: Browser, url: string): Promise<any> {
  const desktopCtx = await browser.newContext({ viewport: { width: 1440, height: 900 } });
  const mobilCtx = await browser.newContext({
    viewport: { width: 375, height: 812 }, isMobile: true, hasTouch: true,
    userAgent: 'Mozilla/5.0 (iPhone; CPU iPhone OS 17_0 like Mac OS X)',
  });
  
  try {
    const desktopPage = await desktopCtx.newPage();
    await desktopPage.goto(url, { waitUntil: 'domcontentloaded', timeout: 15000 });
    const desktopLinks = await desktopPage.$$eval('nav a', links => links.length);
    
    const mobilePage = await mobilCtx.newPage();
    await mobilePage.goto(url, { waitUntil: 'domcontentloaded', timeout: 15000 });
    
    const hasHorizontalScroll = await mobilePage.evaluate(() =>
      document.documentElement.scrollWidth > document.documentElement.clientWidth
    );
    const mobileHasHamburger = await mobilePage.$('.hamburger, [class*="burger"], [class*="toggle"], button[aria-label*="menu"], button[aria-label*="Menu"]');
    const smallTextCount = await mobilePage.evaluate(() => {
      return Array.from(document.querySelectorAll('p, span, li, a'))
        .filter(el => parseFloat(getComputedStyle(el).fontSize) < 14 && (el.textContent?.trim().length ?? 0) > 10)
        .length;
    });
    
    return {
      isResponsive: !hasHorizontalScroll && mobileHasHamburger !== null,
      hasHorizontalScroll,
      hasHamburgerMenu: mobileHasHamburger !== null,
      smallTextCount,
      desktopNavLinks: desktopLinks,
    };
  } finally {
    await desktopCtx.close().catch(() => {});
    await mobilCtx.close().catch(() => {});
  }
}
```

### 8.6 Additional Detection Functions

```typescript
function catalogueIframes($: cheerio.CheerioAPI): any[] {
  return $('iframe[src]').map((_, el) => {
    const src = $(el).attr('src') ?? '';
    return {
      src,
      type: src.includes('google.com/maps') || src.includes('maps.google') ? 'google_maps' :
            src.includes('youtube.com') || src.includes('youtu.be') ? 'youtube_video' :
            src.includes('calendly.com') ? 'booking_calendly' :
            src.includes('instagram.com') ? 'instagram_feed' :
            src.includes('facebook.com') ? 'facebook_widget' :
            src.includes('vimeo.com') ? 'vimeo_video' :
            src.includes('typeform.com') ? 'typeform' : 'unknown',
    };
  }).get();
}

function detectVideos(html: string, iframes: any[]): any[] {
  const videos: any[] = [];
  
  for (const iframe of iframes) {
    if (iframe.type === 'youtube_video') {
      const videoId = iframe.src.match(/(?:embed\/|v=)([a-zA-Z0-9_-]{11})/)?.[1];
      if (videoId) videos.push({ platform: 'youtube', videoId, embedUrl: iframe.src,
        thumbnailUrl: `https://img.youtube.com/vi/${videoId}/maxresdefault.jpg` });
    }
    if (iframe.type === 'vimeo_video') {
      const videoId = iframe.src.match(/player\.vimeo\.com\/video\/(\d+)/)?.[1];
      if (videoId) videos.push({ platform: 'vimeo', videoId, embedUrl: iframe.src });
    }
  }
  
  const $ = cheerio.load(html);
  $('video source[src], video[src]').each((_, el) => {
    const src = $(el).attr('src');
    if (src) videos.push({ platform: 'self_hosted', videoUrl: src });
  });
  
  return videos;
}

function extractStructuredData(html: string): any[] {
  const $ = cheerio.load(html);
  const blocks: any[] = [];
  $('script[type="application/ld+json"]').each((_, el) => {
    try { blocks.push(JSON.parse($(el).text())); } catch { /* invalid JSON-LD */ }
  });
  return blocks;
}

function extractCookieBannerText($: cheerio.CheerioAPI): string | null {
  const selectors = ['.cookie-banner', '.cookie-consent', '#cookie-notice', '.cc-banner',
    '#CybotCookiebotDialog', '.cookieconsent', '[class*="cookie-"]', '[id*="cookie"]'];
  for (const sel of selectors) {
    const el = $(sel);
    if (el.length) {
      const text = el.text().trim();
      if (text.length > 50) return text.substring(0, 2000);
    }
  }
  return null;
}

function analyzeForm(form: cheerio.Cheerio, pageUrl: string, html: string): any {
  const $ = cheerio.load(html);
  const fields = form.find('input, textarea, select');
  const action = form.attr('action') ?? '';
  
  return {
    fieldCount: fields.length,
    requiredFieldCount: fields.filter('[required]').length,
    hasNameField: fields.is('[name*="name" i], [placeholder*="name" i], [placeholder*="Name"]'),
    hasEmailField: fields.is('[type="email"], [name*="email" i]'),
    hasPhoneField: fields.is('[type="tel"], [name*="phone" i], [name*="telefon" i]'),
    hasMessageField: fields.is('textarea'),
    hasFileUpload: fields.is('[type="file"]'),
    hasCaptcha: html.includes('recaptcha') || html.includes('hcaptcha'),
    actionType: !action || action === '#' ? 'BROKEN' :
                action.includes('mailto:') ? 'MAILTO' :
                action.includes('formspree.io') ? 'FORMSPREE' :
                action.includes('netlify') ? 'NETLIFY' :
                action.includes('admin-ajax') ? 'WORDPRESS' :
                action.includes('contact-form-7') ? 'CF7' : 'SELF_HOSTED',
    method: form.attr('method')?.toUpperCase() ?? 'GET',
  };
}

function checkAccessibility($: cheerio.CheerioAPI, html: string): string[] {
  const issues: string[] = [];
  const imgsNoAlt = $('img:not([alt]), img[alt=""]').length;
  if (imgsNoAlt > 3) issues.push(`${imgsNoAlt} Bilder ohne Alt-Text`);
  if ($('h1').length === 0) issues.push('Kein H1-Tag');
  if ($('h1').length > 1) issues.push(`${$('h1').length} H1-Tags (sollte 1 sein)`);
  if (!$('html').attr('lang')) issues.push('Kein lang-Attribut auf <html>');
  return issues;
}

function isLoginWall(html: string): boolean {
  const $ = cheerio.load(html);
  return !!($('form input[type="password"]').length && $('form').length === 1);
}

async function scrollToLoadContent(page: Page): Promise<void> {
  const scrollHeight = await page.evaluate(() => document.body.scrollHeight);
  const viewportHeight = await page.evaluate(() => window.innerHeight);
  let pos = 0;
  while (pos < scrollHeight) {
    pos += viewportHeight;
    await page.evaluate((p) => window.scrollTo(0, p), pos);
    await page.waitForTimeout(400);
  }
  await page.evaluate(() => window.scrollTo(0, 0));
  await page.waitForTimeout(300);
}

async function ensureGermanVersion(page: Page): Promise<void> {
  const lang = await page.getAttribute('html', 'lang');
  if (lang?.startsWith('de')) return;
  const deLink = await page.$('a[href*="/de/"], a[href*="/de-"], a:has-text("Deutsch"), a:has-text("DE")');
  if (deLink) {
    await deLink.click().catch(() => {});
    await page.waitForLoadState('networkidle', { timeout: 5000 }).catch(() => {});
  }
}

function checkConsistencyWithStep1(lead: Lead, pages: ContentPageData[], contentCompleteness: number): { issues: string[]; needsRescore: boolean } {
  const issues: string[] = [];
  if (pages.length > (lead.estimatedPageCount ?? 0) * 3) {
    issues.push(`Tatsächlich ${pages.length} Seiten, geschätzt waren ${lead.estimatedPageCount}`);
  }
  if (contentCompleteness > 0.8 && (lead.quickScore ?? 0) > 70) {
    issues.push('Content besser als erwartet – Score möglicherweise zu hoch');
  }
  if (contentCompleteness < 0.15 && (lead.quickScore ?? 0) < 50) {
    issues.push('Content extrem dünn – möglicherweise Link-Only-Website');
  }
  return { issues, needsRescore: issues.length > 0 };
}

function checkRouteStillValid(lead: Lead, contentCompleteness: number): any | null {
  if (lead.pipelineRoute === 'FULL' && contentCompleteness < 0.2) {
    return { newRoute: 'NURTURE', newSkipSteps: [8, 9, 10, 11, 12] };
  }
  return null;
}
```

---

## 9. COMPLETE EXAMPLE WALKTHROUGH

Lead "Coaching Praxis Sandra K." through Steps 3+4 in parallel:

```
INPUT: Lead approved at Gate 1, processed by Step 2
════════════════════════════════════════════════════

  status: READY_FOR_ENRICHMENT
  pipelineRoute: FULL
  enrichmentPlan: [{ field: "competitors", sources: [{ source: "database", cost: 0 }] }]
  crawlStrategy: FAST
  priorityPages: ["/ueber-mich", "/leistungen", "/kontakt", "/impressum"]
  estimatedPageCount: 4
  isSpa: false
  cmsType: "jimdo"
  decisionMakerName: "Sandra Kowalski" (from Step 2 Impressum)
  decisionMakerEmail: "sandra@coaching-sandra.de" (SMTP_VALID)

STEP 3 (Enrichment) — runs PARALLEL with Step 4:
═════════════════════════════════════════════════

  Duration: 4 seconds | Cost: €0.00

  1. Find Competitors (DB query):
     → 3 coaches found in München: "BusinessCoach Peters", "Karriere-Coaching Huber", "NLP Coach Weber"
     → All are leads in our DB → competitorLeadIds = ["uuid-a", "uuid-b", "uuid-c"]
     → Event: lead.competitors_found emitted (Step 6 CAN start now)
     
  2. Enrichment Plan: Only "competitors" → already done. No LinkedIn needed (name found).
     No Northdata needed (Einzelunternehmen, not GmbH).
     
  3. Email Ranking: 
     → #1: sandra@coaching-sandra.de (impressum, SMTP_VALID, personal, own domain) = priority 100
     → #2: info@coaching-sandra.de (website, UNVERIFIED, generic, own domain) = priority 38
     
  4. No disqualification triggers.
  
  Output: enrichmentStatus = COMPLETED, enrichmentCostCents = 0

STEP 4 (Crawling) — runs PARALLEL with Step 3:
═══════════════════════════════════════════════

  Duration: 45 seconds | Cost: €0.067

  Phase 1 — URL Discovery:
    Sitemap: /sitemap.xml → not found (Jimdo Free doesn't have one)
    Homepage links: /ueber-mich, /leistungen, /kontakt (3 nav links)
    Priority pages: /ueber-mich, /leistungen, /kontakt, /impressum
    Mandatory: /impressum, /datenschutz
    Total URLs: 5 (home + 4 subpages)
    CMS strategy: jimdo → needsPlaywright: true

  Phase 2 — Page Crawls:
    /                  → HOME, 180 words, screenshot ✅, 1 form (broken action="#")
    /ueber-mich        → ABOUT, 320 words, screenshot ✅, 1 photo (Sandra)
    /leistungen        → SERVICES, 410 words, screenshot ✅, lists 3 coaching services
    /kontakt           → CONTACT, 80 words, screenshot ✅, address + phone + map embed
    /impressum         → LEGAL, 120 words, screenshot ✅
    /datenschutz       → 404 Not Found → skipped

  Phase 3 — Images:
    Downloaded: 4 images (1 portrait, 2 stock-looking, 1 icon)
    Classified: 1 PHOTO (Sandra portrait), 2 STOCK, 1 ICON
    Logo: Found in header (Jimdo logo widget) → LOGO
    Favicon: Found → FAVICON

  Phase 4 — Brand Assets:
    Logo: Jimdo-generated text logo (low quality)
    Colors: #2c5282 (primary blue), #e2e8f0 (light gray)
    Fonts: Jimdo system font (no Google Fonts)
    Style: rounded borders, no shadows, minimalist

  Phase 5 — Navigation:
    mainNav: [Über mich, Leistungen, Kontakt]
    footerNav: [Impressum, Datenschutz]
    depth: 1 (flat)

  Phase 6 — Embeds:
    1 iframe: Google Maps on /kontakt → type: google_maps
    No videos. No booking widget.

  Phase 7 — SEO Analysis:
    Missing titles: 0 (all pages have titles)
    Missing descriptions: 3/5 pages (only home has description)
    H1 on all pages: ✅
    Thin content: /kontakt (80 words), /impressum (120 words)
    No structured data (no JSON-LD)
    seoScore: 62

  Phase 8 — Responsive Test:
    isResponsive: true (Jimdo adds viewport meta)
    hasHorizontalScroll: false
    hasHamburgerMenu: true
    smallTextCount: 0

  Phase 9 — AI Extraction (Sonnet):
    Input: 5 pages combined (~1,110 words) → ~1,500 tokens
    Cost: €0.003 (very small site)
    
    firmProfile extracted:
      firm_name: "Coaching Praxis Sandra K."
      slogan: "Systemisches Coaching für Führungskräfte"
      description: "Sandra Kowalski bietet systemisches Coaching..."
      coaching_type: "Business + Leadership"
      methodology: "Systemisch"
      target_audience: "Führungskräfte, Manager"
      services: [
        { name: "Einzelcoaching", description: "Individuelles Coaching-Programm..." },
        { name: "Team-Coaching", description: "Für Teams die..." },
        { name: "Karriere-Beratung", description: "Berufliche Neuorientierung..." }
      ]
      team: [{ name: "Sandra Kowalski", role: "Inhaberin & Coach", bio: "Seit 2015..." }]
      certifications: ["Systemischer Coach (IHK)"]
      testimonials: [] (none on website)
      prices: null (not listed on website)
      opening_hours: null (not listed)
      contact: { phone: "089 12345678", email: "sandra@coaching-sandra.de", address: "Leopoldstr. 42..." }
      booking_method: null (no booking widget)
      lead_magnet: null
      social_links: { linkedin: "linkedin.com/in/sandrak" }
    
    extractionConfidence: 0.72

  Phase 10 — Content Completeness:
    coreIdentity: 3/3 (name ✅, description ✅, services ✅) → 0.25
    contactInfo: 3/3 (phone ✅, email ✅, address ✅) → 0.15
    visualAssets: 1 usable photo → 0.03 (needs more)
    socialProof: 0 testimonials on website → 0.00 (Google reviews exist but not on site)
    industrySpecific: certifications ✅ → 0.07
    brandAssets: logo (low quality) → 0.05
    contentVolume: 1,110 words → 0.10 × min(1, 1110/1000) = 0.10
    TOTAL: 0.65

  Phase 11 — Consistency Check:
    5 pages crawled vs 4 estimated → OK
    contentCompleteness 0.65 vs Step 2 estimate 0.52 → slightly better, no rescore needed

  Phase 12 — Route Validation:
    Route FULL + completeness 0.65 > 0.2 → route stays FULL

  Output: crawlStatus = COMPLETED, contentCompleteness = 0.65

ORCHESTRATOR MERGE (after both complete):
═════════════════════════════════════════

  Content Brief Update:
    ownerName: "Sandra Kowalski" (confirmed by website team section)
    services: [3 services from AI extraction] (NEW — Step 2 didn't have this)
    testimonials: [3 Google reviews] (website had none, Google reviews kept)
    contentReadiness: 0.65 (upgraded from Step 2's 0.52)
    
  Route: FULL (unchanged)
  needsRescore: false

  → Dispatch to Step 5 (Quality Review) + Step 6 (Competitor Benchmarking)
  → Status: ENRICHED_AND_CRAWLED

TOTAL COST Steps 3+4: €0.067
TOTAL DURATION: 45 seconds (Step 4 was the bottleneck)
```

---

## 10. ERROR HANDLING

| Error | Step | Handling | Lead Impact |
|-------|------|----------|-------------|
| Enrichment plan API timeout | 3 | Retry per API_RETRY_CONFIG | Skip failed source, continue |
| Google Custom Search daily limit | 3 | Defer to next day or skip | Continue without competitors |
| LinkedIn rate limit (429) | 3 | 30s backoff, 1 retry | Continue without LinkedIn |
| Northdata shows insolvent | 3 | DISQUALIFY lead | FILTERED_OUT + cancel Step 4 |
| Website unreachable during crawl | 4 | 3 retries with 10s backoff | crawlStatus=FAILED |
| Playwright crash | 4 | Recycle browser, retry | 1 retry, then FAILED |
| Cloudflare blocks crawl | 4 | Stealth → delay → Google Cache | Partial data or FAILED |
| Page takes >60s to load | 4 | Skip page, continue with rest | Partial ContentPackage |
| Claude extraction hallucinates | 4 | Validation catches, retry once | Lower confidence |
| Claude API 500/timeout | 4 | 3 retries with backoff | firmProfile=null, confidence=0 |
| R2 upload fails | 4 | 3 retries | Continue without screenshots |
| Both Step 3+4 fail | Merge | Set status FILTERED_OUT | Lead exits pipeline |
| Content too thin (<0.2) | Merge | Route change FULL→NURTURE | Skip Steps 8-12 |
| Step 3 disqualifies mid-crawl | 3→4 | Cancel Step 4 job in queue | Lead exits pipeline |

---

## 11. EVENTS

```typescript
// Step 3 events
'lead.enrichment_started'       { leadId, planStepCount }
'lead.competitors_found'        { leadId, competitorCount, competitorLeadIds }
'lead.enrichment_completed'     { leadId, status, costCents, durationMs }
'lead.disqualified'             { leadId, step: 3, reason, detail }

// Step 4 events
'lead.crawl_started'            { leadId, strategy, estimatedPages }
'lead.crawl_progress'           { leadId, pagesCrawled, totalEstimated, currentUrl }
'lead.crawl_completed'          { leadId, status, pages, contentCompleteness, costCents, durationMs }

// Merge events
'lead.merge_completed'          { leadId, status, contentCompleteness, routeChanged, newRoute }

// Combined status (consumed by Steps 5+6)
// Steps 5+6 listen for lead.merge_completed to know they can start
```

---

## 12. HANDOFF: STEPS 3+4 → STEPS 5+6

### What Step 5 (Quality Review) receives:

All Lead fields from Steps 1-4, PLUS the ContentPackage with:
- Full-page screenshots (desktop + mobile) for every crawled page
- SEO analysis (per-page + site-wide)
- Accessibility issues
- Responsive design test results
- Form analysis
- Content completeness score
- firmProfile (AI-extracted)

Step 5 does NOT need to re-crawl or re-screenshot.

### What Step 6 (Competitor Benchmarking) receives:

- competitorPlaceIds + competitorLeadIds (from Step 3)
- Lead's own quickScore, screenshots, and website data
- Competitor leads' data from our DB (if competitorLeadIds available)

Step 6 may have ALREADY STARTED (via early competitors.found event from Step 3). If so, the merge_completed event tells it: "Step 4 data is also available now if you need it."

---

## 13. METRICS

```
// Step 3
enrichment_avg_duration_ms: integer
enrichment_avg_cost_cents: float
enrichment_competitor_from_db_rate: float    // % of times competitors found in DB (should be >80%)
enrichment_linkedin_found_rate: float
enrichment_disqualification_rate: float
enrichment_channel_update_rate: float        // % of leads where outreach channel changed

// Step 4
crawl_avg_duration_ms: integer
crawl_avg_cost_cents: float
crawl_avg_pages_per_site: float
crawl_early_exit_rate: float                 // % of crawls that exited early (enough content)
crawl_blocked_rate: float                    // % of websites that blocked Playwright
crawl_content_completeness_avg: float
crawl_extraction_confidence_avg: float
crawl_route_change_rate: float               // % of leads where route changed after crawl

// Combined
steps34_avg_total_cost: float
steps34_avg_total_duration_ms: integer
steps34_both_failed_rate: float              // should be <2%
```

---

## 14. TESTING STRATEGY

```typescript
describe('Step 3: Gap-Fill Enrichment', () => {
  it('finds competitors from DB when available', async () => { /* mock DB with 5 leads */ });
  it('falls back to Google search when <3 in DB', async () => { /* mock empty DB + Google response */ });
  it('skips enrichment for FAST_TRACK route', async () => { /* assert SKIPPED status */ });
  it('ranks emails correctly (personal own-domain first)', () => { /* unit test rankEmailCandidates */ });
  it('disqualifies insolvent companies', async () => { /* mock Northdata response */ });
  it('cancels Step 4 on disqualification', async () => { /* verify job removal from queue */ });
  it('updates outreach channel when email found', async () => { /* mock LinkedIn finding email */ });
  it('respects cost cap', async () => { /* mock expensive plan, verify cap at €0.25 */ });
});

describe('Step 4: Website Crawl', () => {
  it('respects crawlStrategy SKIP', async () => { /* verify no crawl, SKIPPED status */ });
  it('respects crawlStrategy THROTTLED', async () => { /* verify delays between pages */ });
  it('detects page types correctly', () => { /* unit test detectPageType */ });
  it('extracts structured data (JSON-LD)', () => { /* unit test extractStructuredData */ });
  it('classifies images correctly', () => { /* unit test classifyImage */ });
  it('removes boilerplate text', () => { /* unit test extractMainContent */ });
  it('handles Jimdo sites (needs Playwright)', async () => { /* integration test with Jimdo HTML */ });
  it('handles Wix sites (heavy boilerplate)', async () => { /* integration test */ });
  it('does early exit when content sufficient', async () => { /* verify fewer pages crawled */ });
  it('changes route when content too thin', async () => { /* verify FULL→NURTURE */ });
  it('validates AI extraction (no hallucinations)', () => { /* unit test validateExtraction */ });
});

describe('Orchestrator Merge', () => {
  it('merges content brief correctly', async () => { /* verify website data overrides Step 2 estimates */ });
  it('deduplicates testimonials', () => { /* unit test deduplicateTestimonials */ });
  it('dispatches to Steps 5+6 after merge', async () => { /* verify queue additions */ });
  it('handles both-failed scenario', async () => { /* verify FILTERED_OUT status */ });
});
```

---

## 15. INDUSTRY PROFILE EXTENSIONS FOR STEPS 3+4

```json
{
  "step3": {
    "competitorSearchKeyword": "Coach",
    "portalChecks": [],
    "northdataRequired": false,
    "linkedInPriority": "medium"
  },
  "step4": {
    "maxPages": 30,
    "extractionPromptKey": "coaches",
    "priorityPagePatterns": ["/ueber", "/about", "/leistung", "/service", "/kontakt", "/contact"],
    "requiredExtractionFields": ["coaching_type", "methodology", "target_audience"],
    "niceToHaveFields": ["lead_magnet", "booking_method", "prices"]
  },
  "contentCompleteness": {
    "coreIdentity": 0.25,
    "contactInfo": 0.15,
    "visualAssets": 0.10,
    "socialProof": 0.20,
    "industrySpecific": 0.15,
    "brandAssets": 0.05,
    "contentVolume": 0.10
  }
}
```

For Friseure:
```json
{
  "step3": {
    "competitorSearchKeyword": "Friseur",
    "portalChecks": ["treatwell", "booksy"],
    "northdataRequired": false,
    "linkedInPriority": "low"
  },
  "step4": {
    "maxPages": 20,
    "extractionPromptKey": "friseure",
    "priorityPagePatterns": ["/preise", "/galerie", "/team", "/kontakt"],
    "requiredExtractionFields": ["services", "stylists", "salon_type"],
    "niceToHaveFields": ["product_brands", "booking_system"]
  },
  "contentCompleteness": {
    "coreIdentity": 0.20,
    "contactInfo": 0.15,
    "visualAssets": 0.30,
    "socialProof": 0.10,
    "industrySpecific": 0.10,
    "brandAssets": 0.05,
    "contentVolume": 0.10
  }
}
```

---

## 16. ADDED LeadStatus FOR STEPS 3+4

Add to the combined LeadStatus enum (STEP-2-FINAL.md Section 23):

```prisma
enum LeadStatus {
  // ... Steps 1-2 statuses ...
  
  // Steps 3+4 statuses (NEW)
  ENRICHED_AND_CRAWLED    // both Step 3 + Step 4 complete, merge done, ready for Steps 5+6
}
```

---

## 17. CHECKPOINT: WHAT A LEAD HAS AFTER STEPS 3+4

This is the COMPLETE data snapshot that Steps 5-10 work with:

```
From Step 1: Google data, technical signals, Quick Score, screenshots, weakness arguments
From Step 2: Decision maker, legal form, routing, enrichment plan, economics, content brief
From Step 3: Competitors, portal profiles, LinkedIn URL, refined emails, enrichment confidence
From Step 4: ContentPackage (pages, images, brand assets, firm profile, SEO, accessibility,
             responsive test, embeds, videos, navigation, form analysis, content completeness)
From Merge: Updated content brief, confirmed route, lifecycle tracker

Total invested per lead: €0.03 (Step 1) + €0.00 (Step 2) + €0.02 (Step 3) + €0.07 (Step 4) = ~€0.12
Total duration: ~60 seconds (Step 4 dominates)
```

---

## 18. TYPESCRIPT INTERFACES (all types referenced in Steps 3+4)

```typescript
// ============================================
// STEP 3 INTERFACES
// ============================================

interface CompetitorResult {
  placeIds: string[];
  leadIds: string[];            // if competitors are also leads in our DB
  source: 'database' | 'database+google' | 'google' | 'none';
  costCents: number;
}

interface EnrichmentSourceResult {
  // Each source MAY return one or more of these:
  email?: EmailCandidate;
  decisionMakerName?: string;
  linkedInUrl?: string;
  portalProfile?: PortalProfile;
  northdata?: NorthdataResult;
  bonusFields?: Array<{ field: string; value: any }>; // opportunistic extra data
}

interface PortalProfile {
  platform: string;             // "treatwell", "booksy", "myhammer"
  url: string | null;
  exists: boolean;
}

interface NorthdataResult {
  firmStatus: 'active' | 'dissolved' | 'liquidation' | 'insolvent' | 'unknown';
  registeredCapital: number | null;
  employeeCount: number | null;
  foundingDate: string | null;
  industry: string | null;
}

interface EmailCandidate {
  email: string;
  source: 'impressum' | 'website' | 'linkedin' | 'hunter' | 'guessed' | 'portal';
  validation: EmailValidation;
  isPersonal: boolean;
  isOwnDomain: boolean;
  priority: number;             // calculated by rankEmailCandidates
}

interface SearchResult {
  url: string;
  title: string;
  snippet: string;
}

// ============================================
// STEP 4 INTERFACES
// ============================================

interface CrawlConfig {
  crawlStrategy: CrawlStrategy;
  priorityPages: string[];
  maxPages: number;
  throttleMs: number;
  isSpa: boolean;
  cmsStrategy: CmsStrategy;
  downloadImages: boolean;
  pipelineRoute: PipelineRoute;
}

interface ContentPageData {
  url: string;
  type: PageType;
  title: string | null;
  textContent: string | null;   // boilerplate-removed plain text
  htmlClean: string | null;     // boilerplate-removed HTML
  html: string;                 // full original HTML
  wordCount: number;
  screenshotDesktopUrl: string | null;
  screenshotMobileUrl: string | null;
  loadTimeMs: number;
  metaDescription: string | null;
  h1Text: string | null;
  h1Count: number;
  hasCanonical: boolean;
  hasOgTags: boolean;
  internalLinkCount: number;
  externalLinkCount: number;
  imagesOnPage: number;
  imagesWithAlt: number;
  images: ImageCandidate[];
  iframes: IframeInfo[];
  videos: VideoInfo[];
  structuredData: any[];
  cookieBannerText: string | null;
  formAnalysis: FormAnalysisResult | null;
  accessibilityIssues: string[];
  skipped?: boolean;
  reason?: string;              // why skipped (e.g. 'login_wall')
}

interface ImageCandidate {
  src: string;
  alt: string;
  width: number;
  height: number;
  filename: string;
  contentType: string;
  fromPage: string;
  position: 'header' | 'footer' | 'body';
}

interface ContentImageData {
  originalUrl: string;
  r2Url: string;
  alt: string;
  width: number;
  height: number;
  fileSizeBytes: number;
  type: string;                 // ImageType enum value
  usable: boolean;
  fromPageUrl: string | null;
}

interface IframeInfo {
  src: string;
  type: 'google_maps' | 'youtube_video' | 'vimeo_video' | 'booking_calendly' |
        'instagram_feed' | 'facebook_widget' | 'typeform' | 'unknown';
}

interface VideoInfo {
  platform: 'youtube' | 'vimeo' | 'self_hosted';
  videoId?: string;
  embedUrl?: string;
  videoUrl?: string;
  thumbnailUrl?: string;
}

interface FormAnalysisResult {
  fieldCount: number;
  requiredFieldCount: number;
  hasNameField: boolean;
  hasEmailField: boolean;
  hasPhoneField: boolean;
  hasMessageField: boolean;
  hasFileUpload: boolean;
  hasCaptcha: boolean;
  actionType: 'BROKEN' | 'MAILTO' | 'FORMSPREE' | 'NETLIFY' | 'WORDPRESS' | 'CF7' | 'SELF_HOSTED' | 'EXTERNAL';
  method: string;
}

interface PdfContent {
  url: string;
  text: string;
  pageCount: number;
  type: 'menu' | 'pricelist' | 'flyer' | 'brochure' | 'other';
}

interface NavStructure {
  mainNav: Array<{ label: string; url: string; children?: Array<{ label: string; url: string }> }>;
  footerNav: Array<{ label: string; url: string }>;
  totalLinks: number;
  depth: number;
}

interface ResponsiveResult {
  isResponsive: boolean;
  hasHorizontalScroll: boolean;
  hasHamburgerMenu: boolean;
  smallTextCount: number;
  desktopNavLinks: number;
}

interface StyleAnalysis {
  borderRadius: 'sharp' | 'rounded' | 'pill';
  shadows: 'has_shadows' | 'flat';
  hasAnimations: boolean;
  layoutType: 'css_grid' | 'flexbox' | 'traditional';
  isMinimalist: boolean;
}

interface FreshnessResult {
  status: string;
  changed: boolean;
  blocking?: boolean;
  updates?: Record<string, any>;
}

interface DisqualificationResult {
  reason: string;
  detail: string;
}
```

---

## 19. MISSING FUNCTIONS (Audit Additions)

### 19.1 Not-a-Website Detection (P75)

Step 4 checks at the START of crawling whether the URL is actually a website:

```typescript
const NOT_A_WEBSITE_DOMAINS = [
  'facebook.com', 'instagram.com', 'twitter.com', 'x.com',
  'linkedin.com', 'xing.com', 'tiktok.com', 'pinterest.com',
  'linktr.ee', 'linkin.bio', 'lnk.bio', 'bio.link',
  'bit.ly', 'tinyurl.com',
  'youtube.com', 'youtu.be', 'vimeo.com',
  'google.com', 'maps.google.com',
  'yelp.de', 'yelp.com', 'tripadvisor.de', 'tripadvisor.com',
  'jameda.de', 'doctolib.de',
];

function isNotARealWebsite(url: string): boolean {
  const domain = normalizeDomain(url);
  return NOT_A_WEBSITE_DOMAINS.some(d => domain.includes(d));
}

function isLinkOnlyWebsite(pages: ContentPageData[]): boolean {
  const totalWords = pages.reduce((sum, p) => sum + p.wordCount, 0);
  if (totalWords >= 100) return false;
  
  const allLinks = pages.reduce((sum, p) => {
    const $ = cheerio.load(p.html);
    return sum + $('a[href^="http"]').length;
  }, 0);
  
  const internalLinks = pages.reduce((sum, p) => {
    const $ = cheerio.load(p.html);
    const domain = normalizeDomain(p.url);
    return sum + $(`a`).filter((_, el) => {
      const href = $(el).attr('href') ?? '';
      try { return new URL(href, p.url).hostname.includes(domain); } catch { return false; }
    }).length;
  }, 0);
  
  const externalLinks = allLinks - internalLinks;
  return totalWords < 100 && externalLinks > allLinks * 0.5;
}

// Used at the START of executeCrawl():
// if (isNotARealWebsite(config.actualUrl)) {
//   await prisma.lead.update({ where: { id: leadId }, data: { 
//     crawlStatus: 'FAILED', 
//     rescoreReason: 'URL is not a real website (social media / link aggregator)',
//     pipelineRoute: 'NURTURE',
//   }});
//   return;
// }
//
// After crawling, if isLinkOnlyWebsite(pages):
//   Same handling — route to NURTURE
```

### 19.2 Crawl Fallback Chain for Blocked Websites (P23)

```typescript
async function crawlWithFallbacks(
  browser: Browser, 
  url: string, 
  config: CrawlConfig
): Promise<ContentPageData | null> {
  // Attempt 1: Normal crawl
  let result = await crawlSinglePage(browser, url, config);
  if (result && !result.skipped) return result;
  
  // Attempt 2: Stealth mode (different user agent, slower)
  logger.info({ url }, 'Normal crawl failed, trying stealth mode');
  const stealthConfig = { ...config, throttleMs: 5000 };
  // Use a different user agent
  const stealthCtx = await browser.newContext({
    viewport: { width: 1440, height: 900 },
    userAgent: 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/17.0 Safari/605.1.15',
    locale: 'de-DE',
  });
  try {
    const page = await stealthCtx.newPage();
    await page.waitForTimeout(3000); // extra delay
    await page.goto(url, { waitUntil: 'networkidle', timeout: 30000 });
    if (await isBlockedPage(page)) {
      // Still blocked → try Google Cache
    } else {
      // Stealth worked!
      result = await extractPageData(page, url, config);
      return result;
    }
  } finally {
    await stealthCtx.close().catch(() => {});
  }
  
  // Attempt 3: Google Cache
  logger.info({ url }, 'Stealth failed, trying Google Cache');
  const cacheUrl = `https://webcache.googleusercontent.com/search?q=cache:${encodeURIComponent(url)}`;
  try {
    const cacheCtx = await browser.newContext({ viewport: { width: 1440, height: 900 } });
    const cachePage = await cacheCtx.newPage();
    await cachePage.goto(cacheUrl, { waitUntil: 'domcontentloaded', timeout: 15000 });
    const html = await cachePage.content();
    if (html.length > 1000 && !html.includes('404') && !html.includes('not found')) {
      // Cache hit! Extract what we can (screenshots will be of the cache page, less ideal)
      result = await extractPageData(cachePage, url, { ...config, isCache: true });
      result.screenshotDesktopUrl = null; // cache screenshots aren't useful
      result.screenshotMobileUrl = null;
      await cacheCtx.close();
      return result;
    }
    await cacheCtx.close();
  } catch { /* cache also failed */ }
  
  // All attempts failed
  logger.warn({ url }, 'All crawl attempts failed (normal + stealth + cache)');
  return null;
}

async function isBlockedPage(page: Page): Promise<boolean> {
  const text = await page.textContent('body') ?? '';
  const textLC = text.toLowerCase();
  return (
    textLC.includes('checking your browser') ||
    textLC.includes('please wait') ||
    textLC.includes('attention required') ||
    textLC.includes('ddos protection') ||
    text.length < 200
  );
}
```

### 19.3 PDF Extraction (P24)

```typescript
import pdfParse from 'pdf-parse';

async function extractPdfContent(pdfUrl: string): Promise<PdfContent | null> {
  try {
    const response = await fetch(pdfUrl, {
      signal: AbortSignal.timeout(15000),
      headers: { 'User-Agent': BROWSER_USER_AGENT },
    });
    if (!response.ok) return null;
    
    const contentType = response.headers.get('content-type') ?? '';
    if (!contentType.includes('pdf')) return null;
    
    const buffer = Buffer.from(await response.arrayBuffer());
    if (buffer.length > 10 * 1024 * 1024) return null; // skip >10MB
    
    const data = await pdfParse(buffer);
    
    return {
      url: pdfUrl,
      text: data.text.substring(0, 50000), // limit text length
      pageCount: data.numpages,
      type: detectPdfType(pdfUrl, data.text),
    };
  } catch (error) {
    logger.warn({ pdfUrl, error: String(error) }, 'PDF extraction failed');
    return null;
  }
}

function detectPdfType(url: string, text: string): PdfContent['type'] {
  const urlLC = url.toLowerCase();
  const textLC = text.toLowerCase();
  
  if (urlLC.includes('speisekarte') || urlLC.includes('menu') || 
      textLC.includes('vorspeisen') || textLC.includes('hauptgericht')) return 'menu';
  if (urlLC.includes('preis') || urlLC.includes('price') || 
      (textLC.match(/€/g)?.length ?? 0) > 5) return 'pricelist';
  if (urlLC.includes('flyer') || urlLC.includes('brosch')) return 'flyer';
  if (urlLC.includes('katalog') || urlLC.includes('catalog')) return 'brochure';
  return 'other';
}

// Called during page crawl when PDF links are found:
// const pdfLinks = $('a[href$=".pdf"]').map((_, el) => $(el).attr('href')).get();
// for (const pdfHref of pdfLinks.slice(0, 5)) { // max 5 PDFs
//   const pdfUrl = new URL(pdfHref, pageUrl).href;
//   const pdf = await extractPdfContent(pdfUrl);
//   if (pdf) pdfs.push(pdf);
// }
```

### 19.4 Duplicate Page Detection (P60)

```typescript
function isDuplicatePage(newPage: ContentPageData, existingPages: ContentPageData[]): boolean {
  for (const existing of existingPages) {
    // Same title AND similar word count → likely duplicate
    if (existing.title && newPage.title && 
        existing.title === newPage.title && 
        Math.abs(existing.wordCount - newPage.wordCount) < 30) {
      return true;
    }
    
    // Text content overlap >85% → duplicate
    if (existing.textContent && newPage.textContent &&
        existing.textContent.length > 100 && newPage.textContent.length > 100) {
      const similarity = calculateTextOverlap(existing.textContent, newPage.textContent);
      if (similarity > 0.85) return true;
    }
  }
  return false;
}

function calculateTextOverlap(textA: string, textB: string): number {
  // Simple word-level Jaccard similarity
  const wordsA = new Set(textA.toLowerCase().split(/\s+/).filter(w => w.length > 3));
  const wordsB = new Set(textB.toLowerCase().split(/\s+/).filter(w => w.length > 3));
  
  if (wordsA.size === 0 || wordsB.size === 0) return 0;
  
  const intersection = [...wordsA].filter(w => wordsB.has(w)).length;
  const union = new Set([...wordsA, ...wordsB]).size;
  
  return intersection / union;
}

// Used in the main crawl loop:
// if (isDuplicatePage(pageResult, pages)) {
//   logger.info({ url, duplicateOf: '...' }, 'Skipping duplicate page');
//   continue;
// }
```

### 19.5 Opening Hours Mismatch Detection (P47)

```typescript
function detectOpeningHoursMismatch(
  websiteText: string, 
  googleHours: Record<string, string> | null
): { mismatch: boolean; detail: string | null } {
  if (!googleHours) return { mismatch: false, detail: null };
  
  // Try to find opening hours in website text
  const DAYS_DE = ['Montag', 'Dienstag', 'Mittwoch', 'Donnerstag', 'Freitag', 'Samstag', 'Sonntag'];
  const DAYS_SHORT = ['Mo', 'Di', 'Mi', 'Do', 'Fr', 'Sa', 'So'];
  
  const hoursPattern = /(?:Mo(?:ntag)?|Di(?:enstag)?|Mi(?:ttwoch)?|Do(?:nnerstag)?|Fr(?:eitag)?|Sa(?:mstag)?|So(?:nntag)?)\s*[:\-–]\s*(\d{1,2}[:.]\d{2})\s*[-–bis]+\s*(\d{1,2}[:.]\d{2})/gi;
  
  const websiteHours: string[] = [];
  let match;
  while ((match = hoursPattern.exec(websiteText)) !== null) {
    websiteHours.push(match[0]);
  }
  
  if (websiteHours.length === 0) return { mismatch: false, detail: null };
  
  // Simple heuristic: if website mentions different hours than Google, flag it
  // (Full comparison would require parsing both formats — for MVP, just flag that hours exist in both)
  return {
    mismatch: true,
    detail: `Website zeigt Öffnungszeiten (${websiteHours.length} Einträge). Abgleich mit Google empfohlen.`,
  };
}

// Result stored in firmProfile.openingHoursMismatch
// Used by Step 5 as a quality issue and by Step 13 as outreach argument
```

### 19.6 Navigation Extraction (referenced but not implemented)

```typescript
function extractNavigation(homepageHtml: string, baseUrl: string): NavStructure {
  const $ = cheerio.load(homepageHtml);
  const hostname = new URL(baseUrl).hostname;
  
  const mainNav: NavStructure['mainNav'] = [];
  const footerNav: NavStructure['footerNav'] = [];
  
  // Main navigation (from <nav> or <header>)
  const navContainer = $('nav').first().length ? $('nav').first() : $('header');
  navContainer.find('> ul > li, > a, > div > ul > li, > div > a').each((_, el) => {
    const link = $(el).is('a') ? $(el) : $(el).find('> a').first();
    const href = link.attr('href');
    const label = link.text().trim();
    
    if (!href || !label || label.length > 50) return;
    
    try {
      const fullUrl = new URL(href, baseUrl);
      if (fullUrl.hostname !== hostname) return; // skip external links
      
      const item: NavStructure['mainNav'][0] = { label, url: fullUrl.pathname };
      
      // Check for children (dropdown menu)
      const children = $(el).find('ul li a');
      if (children.length > 0) {
        item.children = children.map((_, child) => ({
          label: $(child).text().trim(),
          url: new URL($(child).attr('href') ?? '', baseUrl).pathname,
        })).get().filter(c => c.label.length > 0 && c.label.length < 50);
      }
      
      mainNav.push(item);
    } catch { /* invalid URL */ }
  });
  
  // Footer navigation
  $('footer a[href]').each((_, el) => {
    const href = $(el).attr('href');
    const label = $(el).text().trim();
    if (!href || !label || label.length > 50) return;
    try {
      const fullUrl = new URL(href, baseUrl);
      if (fullUrl.hostname === hostname) {
        footerNav.push({ label, url: fullUrl.pathname });
      }
    } catch { /* skip */ }
  });
  
  const maxDepth = mainNav.some(item => item.children && item.children.length > 0) ? 2 : 1;
  
  return {
    mainNav,
    footerNav,
    totalLinks: mainNav.length + footerNav.length + mainNav.reduce((sum, item) => sum + (item.children?.length ?? 0), 0),
    depth: maxDepth,
  };
}
```
