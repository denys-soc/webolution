# STEP 1: FIND BUSINESSES WITH BAD WEBSITES
# Complete Implementation Spec v1.0

> **This is a SELF-CONTAINED spec.** Claude Code reads this file and builds Step 1.
> No prior knowledge needed. Everything is here: mission, architecture, database schema,
> algorithms, prompts, detection logic, error handling, testing strategy, and a complete example.

---

## 1. MISSION

**What this step does:** Find businesses whose website is so bad that we can build a dramatically better one – and they'll pay for it.

**What this step does NOT do:** It does not collect "leads" and hope some have bad websites. It FILTERS aggressively: only businesses with clearly bad websites enter the pipeline. Everything else is archived immediately.

**The core insight:** We don't sell websites. We solve a problem: "Your bad website is costing you customers." Step 1 identifies businesses with this problem.

**Input:** A campaign configuration (industry + cities + budget).
**Output:** 30-60 qualified leads per city, each with a Website Weakness Score, technical analysis, and personalized outreach arguments.

## 2. CONTEXT: THE 5 LEVELS OF "BAD"

A website isn't just "bad." It fails on specific LEVELS, each with different indicators and different sales arguments:

**Level 1 – INVISIBLE:** The website exists but Google can't find it. No SSL, no meta tags, no schema markup, not indexed. The business is invisible online. Sales argument: "Potential customers search for [industry] in [city] – and find your competitors, not you."

**Level 2 – REPULSIVE:** Visitors arrive but leave in 3 seconds. Design from 2012, not mobile-optimized, loads 8 seconds, cluttered layout, auto-play music. Sales argument: "Open your website on your phone. That's what 70% of your visitors see."

**Level 3 – PASSIVE:** Visitors stay but do nothing. No call-to-action, no booking widget, no contact form above the fold, phone number not clickable. The website is a dead end. Sales argument: "Where on your website can someone book an appointment? Nowhere."

**Level 4 – TRUSTLESS:** Visitors consider acting but nothing convinces them. No testimonials, no Google reviews shown, no real photos, no team page, no certifications visible. Sales argument: "You have 4.7 stars on Google with 34 reviews. Your website shows none of them."

**Level 5 – EMBARRASSING:** The business owner KNOWS the website is bad. They cringe when giving out the URL. Sales argument: "When you share your website link – are you proud of it?"

Every weakness we detect maps to one of these levels. The Quick Score aggregates across all levels.

## 3. TECH STACK

```
Language:        TypeScript
Runtime:         Node.js 20+
Queue:           BullMQ (Redis)
Database:        PostgreSQL via Prisma ORM
HTTP Client:     undici (Node.js built-in, fastest)
HTML Parser:     cheerio (fast, jQuery-like, no JS execution)
Browser:         Playwright (Chromium headless) – only for screenshots
AI:              Claude Haiku (claude-haiku-4-5-20251001) via Anthropic API – only for visual checks
External APIs:   Google Places API (New), Google PageSpeed Insights API
Object Storage:  Cloudflare R2 (S3-compatible) – for screenshots
Logging:         pino (structured JSON logging)
```

## 4. DATABASE SCHEMA (Prisma)

```prisma
// ============================================
// CAMPAIGNS
// ============================================

model Campaign {
  id                  String            @id @default(uuid())
  name                String            // "Coaches München März 2026"
  industryTag         String            // "coaches", "friseure", etc.
  cities              String[]          // ["München", "Augsburg"]
  status              CampaignStatus    @default(CREATED)
  mode                CampaignMode      @default(FULL)
  
  // Budget & Costs
  budgetCents         Int?              // max budget in cents, null = unlimited
  spentCents          Int               @default(0)
  
  // Progress
  totalSearchCalls    Int               @default(0)
  completedSearchCalls Int              @default(0)
  totalRawLeads       Int               @default(0)
  totalQualified      Int               @default(0)
  totalFiltered       Int               @default(0)
  
  // Timing
  startedAt           DateTime?
  completedAt         DateTime?
  estimatedDurationMin Int?
  
  // Progress log for resume-after-crash
  progressLog         Json?             // { searchPhase: {}, checkPhase: {}, enrichPhase: {} }
  
  // Relations
  leads               Lead[]
  apiCostLogs         ApiCostLog[]
  
  createdAt           DateTime          @default(now())
  updatedAt           DateTime          @updatedAt
  createdBy           String?           // operator user ID
}

enum CampaignStatus {
  CREATED
  RUNNING
  PAUSED
  COMPLETED
  FAILED
  CANCELLED
}

enum CampaignMode {
  FULL        // scrape everything
  PREVIEW     // only 5 leads as sample
  DRY_RUN     // estimate only, no API calls
}

// ============================================
// LEADS
// ============================================

model Lead {
  id                  String            @id @default(uuid())
  campaignId          String
  campaign            Campaign          @relation(fields: [campaignId], references: [id])
  
  // === IDENTITY ===
  googlePlaceId       String            @unique
  firmName            String
  address             String
  city                String
  postalCode          String?
  latitude            Float?
  longitude           Float?
  phone               String?
  industryTag         String
  
  // === GOOGLE DATA ===
  googleRating        Float?            // 1.0-5.0, null if no rating
  googleReviewCount   Int               @default(0)
  googleCategory      String?
  googlePriceLevel    Int?              // 1-4
  googlePhotosCount   Int               @default(0)
  googleOwnerPhotosCount Int            @default(0)
  googleLatestPhotoDate DateTime?
  googleBusinessDescription String?
  googleMapsUrl       String?
  googleProfileCompleteness Float?      // 0.0-1.0
  googleReviewTexts   Json?             // top 5 reviews as [{text, rating, date, authorName}]
  googleAttributes    Json?             // {wheelchairAccessible, parking, outdoorSeating, ...}
  
  // === WEBSITE URLs ===
  googleListedUrl     String?           // URL from Google listing
  actualUrl           String?           // URL after following all redirects
  websiteType         WebsiteType       @default(UNKNOWN)
  
  // === TECHNICAL SIGNALS (from HTTP Quick-Check, €0.00) ===
  hasSsl              Boolean?
  sslStatus           SslStatus?
  sslExpiryDate       DateTime?
  httpStatus          Int?
  redirectChain       String[]          @default([])
  serverType          String?           // "Apache/2.4.41", "nginx/1.18", "cloudflare"
  cmsType             String?           // "wordpress", "jimdo", "wix", "squarespace", etc.
  cmsVersion          String?           // "5.2", "6.0", etc.
  hasViewportMeta     Boolean?
  copyrightYear       Int?
  titleTag            String?
  metaDescription     String?
  hasH1               Boolean?
  primaryFont         String?           // from Google Fonts link detection
  hasContactForm      Boolean?
  hasTelLink          Boolean?          // <a href="tel:">
  hasBookingWidget    Boolean?
  bookingPlatform     String?           // "calendly", "treatwell", "booksy", etc.
  hasEcommerce        Boolean?
  ecommercePlatform   String?           // "shopify", "woocommerce", etc.
  estimatedPageCount  Int?              // from sitemap or navigation link count
  websiteLanguage     String?           // "de", "en", "tr", etc.
  isSpa               Boolean?
  isParked            Boolean?
  isUnderConstruction Boolean?
  homepageWordCount   Int?
  homepageTextHash    String?           // SHA-256 for duplicate content detection
  
  // === CONTENT SIGNALS ===
  hasBlog             Boolean?
  latestBlogDate      DateTime?
  blogStatus          BlogStatus?
  websiteLastActivity ActivityStatus?
  
  // === TRACKING SIGNALS ===
  hasGoogleAnalytics  Boolean?
  hasFacebookPixel    Boolean?
  hasOtherTracking    String[]          @default([])
  hasCookieConsent    Boolean?
  likelyDsgvoViolation Boolean?
  
  // === REACHABILITY SIGNALS ===
  emailFoundOnWebsite String?
  socialLinks         Json?             // {instagram: "url", facebook: "url", linkedin: "url"}
  instagramHandle     String?
  
  // === PLATFORM SIGNALS ===
  websitePlatformTier PlatformTier?
  baukastenPlatform   String?
  websiteBuiltBy      String?           // from footer "Webdesign: ..."
  websiteBuiltByType  BuilderType?
  
  // === DOMAIN SIGNALS ===
  domainType          DomainType?
  domainAgeYears      Int?
  
  // === CDN SIGNALS ===
  behindCdn           Boolean?
  cdnProvider         String?
  
  // === FORM QUALITY ===
  formQuality         Json?             // {hasAction, actionTarget, hasSubmit, fieldCount, hasCaptcha, estimatedQuality}
  
  // === SEO SIGNALS ===
  googleIndexedPages  Int?
  ranksForPrimaryKeyword Boolean?
  rankingPosition     Int?
  seoTransitionRisk   RiskLevel?
  
  // === LIGHTHOUSE SCORES ===
  lighthousePerformance Int?
  lighthouseSeo       Int?
  lighthouseAccessibility Int?
  
  // === AI VISION (only for gray-zone leads) ===
  screenshotDesktopUrl String?          // R2 URL
  screenshotMobileUrl  String?          // R2 URL
  aiDesignEra         String?           // "pre-2015", "2015-2019", "2020-2023", "2024+"
  aiProfessionalImpression Int?         // 1-10
  aiHasHero           Boolean?
  aiHasVisibleCta     Boolean?
  aiHasRealPhotos     Boolean?
  aiHasStockPhotos    Boolean?
  aiBiggestWeakness   String?
  aiNotableStrength   String?
  
  // === SCORES ===
  quickScore          Int?              // 0-100 (higher = worse website = better candidate)
  quickScoreBreakdown Json?             // {no_cta: 18, outdated: 12, ...}
  functionalScore     Int?              // 0-50
  visualScore         Int?              // 0-50
  scoreConfidence     Confidence?
  improvementPotential PotentialLevel?
  scoreVersion        String?           // score formula version for reproducibility
  
  // === BUSINESS SIGNALS ===
  estimatedSize       BusinessSize?
  estimatedRevenue    RevenueLevel?
  growthSignal        GrowthSignal?
  competitorDensity   Int?
  onlineActivityScore Float?            // 0.0-1.0
  noGoogleRating      Boolean           @default(false)
  
  // === GEO CLUSTERING ===
  geoClusterId        String?
  geoClusterSize      Int?
  areaPurchasingPower PurchasingPower?
  
  // === MULTI-BUSINESS ===
  possibleSameOwner   String[]          @default([]) // other lead IDs
  possibleDuplicate   String[]          @default([])
  
  // === TEMPLATE DETECTION ===
  templateStructureHash String?
  sharedTemplate      Boolean           @default(false)
  
  // === RATING MISMATCH ===
  ratingMismatch      Boolean           @default(false)
  ratingMismatchDetail Json?            // {websiteShows, googleActual, difference}
  
  // === ROBOTS.TXT ===
  robotsTxtStatus     RobotsTxtStatus?
  sitemapUrl          String?
  
  // === POSITIVE SIGNALS ===
  strengths           String[]          @default([]) // ["good_photos", "good_text", ...]
  
  // === OUTREACH PREPARATION ===
  weaknesses          String[]          @default([]) // ["no_cta", "design_pre2020", ...]
  weaknessLabels      String[]          @default([]) // ["Kein Buchungsweg", ...]
  weaknessArguments   String[]          @default([]) // ["Wo kann jemand...?", ...]
  weaknessCount       Int               @default(0)
  screenshotPriority  ScreenshotPrio?   // industry-specific
  
  // === PRIORITY ===
  priorityTag         PriorityTag?
  goldReason          String?
  reachabilityScore   Float?            // 0.0-1.0
  emotionalTiming     TimingSignal?
  spamSaturationRisk  RiskLevel?
  
  // === STATUS ===
  status              LeadStatus        @default(RAW)
  filteredReason      String?           // why filtered out
  source              String            @default("google_maps")
  
  // === TIMESTAMPS ===
  scrapedAt           DateTime          @default(now())
  quickCheckDate      DateTime?
  screenshotExpiresAt DateTime?         // for retention policy cleanup
  
  createdAt           DateTime          @default(now())
  updatedAt           DateTime          @updatedAt
  
  // === INDEXES ===
  @@index([campaignId])
  @@index([status])
  @@index([industryTag])
  @@index([city])
  @@index([quickScore])
  @@index([priorityTag])
  @@index([geoClusterId])
}

// ============================================
// SUPPORTING TABLES
// ============================================

model Blacklist {
  id                  String            @id @default(uuid())
  googlePlaceId       String?           @unique
  domain              String?           @unique
  reason              String            // "user_requested_deletion", "permanently_closed", etc.
  createdAt           DateTime          @default(now())
}

model ApiCostLog {
  id                  String            @id @default(uuid())
  campaignId          String?
  campaign            Campaign?         @relation(fields: [campaignId], references: [id])
  apiName             String            // "google_text_search", "google_place_details", "lighthouse", "claude_haiku"
  endpoint            String
  costCents           Int               // in euro cents
  responseStatus      Int?
  durationMs          Int?
  createdAt           DateTime          @default(now())
  
  @@index([campaignId])
  @@index([apiName])
  @@index([createdAt])
}

model ScoreVersion {
  id                  String            @id @default(uuid())
  version             String            @unique // "v1.0", "v1.1", etc.
  industryTag         String
  weightsJson         Json              // the complete weights configuration
  activeFrom          DateTime
  activeUntil         DateTime?
  performanceMetrics  Json?             // {precision, recall, correlation} from calibration
  createdAt           DateTime          @default(now())
  
  @@index([industryTag])
}

// ============================================
// ENUMS
// ============================================

enum LeadStatus {
  RAW                 // just scraped from Google, no checks yet
  CHECKING            // currently being checked
  QUALIFIED           // passed quality check, waiting for Gate 1
  FILTERED_OUT        // did not pass quality check
  DUPLICATE           // duplicate of existing lead
  NO_WEBSITE          // no website URL (Track B candidate)
  PARKED              // domain is parked
  UNDER_CONSTRUCTION  // website shows "coming soon"
  UNREACHABLE         // website not reachable (DNS error, timeout)
  AUTH_REQUIRED       // website requires login
  GATE1_APPROVED      // approved by operator at Gate 1
  GATE1_REJECTED      // rejected by operator at Gate 1
}

enum WebsiteType {
  OWN_WEBSITE
  BAUKASTEN_SUBDOMAIN
  AGGREGATOR_PROFILE
  SOCIAL_MEDIA_ONLY
  PARKED
  PDF_ONLY
  NONE
  UNKNOWN
}

enum SslStatus {
  VALID
  EXPIRED
  MISCONFIGURED
  MISSING
}

enum BlogStatus {
  ACTIVE              // posts within last 6 months
  STALE               // last post 6-24 months ago
  ABANDONED           // last post >24 months ago
  NONE                // no blog
}

enum ActivityStatus {
  ACTIVELY_MAINTAINED
  STALE
  ABANDONED
}

enum PlatformTier {
  FREE_BAUKASTEN      // jimdo free, wix free (subdomain, branding visible)
  PAID_BAUKASTEN      // jimdo paid, wix paid (own domain, less branding)
  CUSTOM              // wordpress self-hosted, custom development
  UNKNOWN
}

enum BuilderType {
  AGENCY
  AMATEUR             // "Webdesign: Max Mustermann" (not a company)
  UNKNOWN
}

enum DomainType {
  OWN_DE              // friseur-mueller.de
  OWN_COM             // friseur-mueller.com
  OWN_OTHER           // friseur-mueller.net, .info, etc.
  BAUKASTEN_SUBDOMAIN // friseur-mueller.jimdosite.com
  KEYWORD_STUFFED     // friseur-mueller-muenchen-haare-schneiden.de
}

enum Confidence {
  HIGH                // ≥8 of 12 checks completed
  MEDIUM              // 5-7 checks
  LOW                 // <5 checks (e.g. SPA, auth-required)
}

enum PotentialLevel {
  HIGH
  MEDIUM
  LOW
}

enum BusinessSize {
  SOLO
  SMALL               // 2-5 people
  MEDIUM              // 6-20 people
  UNKNOWN
}

enum RevenueLevel {
  LOW                 // <€100k/year estimated
  MEDIUM              // €100-500k
  HIGH                // >€500k
}

enum GrowthSignal {
  GROWING
  STABLE
  DECLINING
  NEW
  UNKNOWN
}

enum PriorityTag {
  GOLD                // score ≥70, rating ≥4.3, reviews ≥15
  SILVER              // score ≥50, rating ≥3.5, reviews ≥5
  STANDARD            // all other qualified leads
}

enum TimingSignal {
  GOOD                // positive momentum (milestone, new photos)
  NEUTRAL
  BAD                 // negative reviews spike
}

enum RiskLevel {
  LOW
  MEDIUM
  HIGH
}

enum PurchasingPower {
  HIGH
  MEDIUM
  LOW
}

enum ScreenshotPrio {
  MOBILE
  DESKTOP
}

enum RobotsTxtStatus {
  NONE                // no robots.txt
  NORMAL              // has rules but doesn't block everything
  BLOCKS_ALL          // Disallow: /
}
```

---

## 5. WORKER ARCHITECTURE

```
┌──────────────────────────────────────────────────────────┐
│ Admin Backend (Next.js + tRPC)                           │
│                                                          │
│  POST /api/campaigns/create                              │
│  → validates input                                       │
│  → creates Campaign in DB (status: CREATED)              │
│  → adds job to BullMQ "campaign-queue"                   │
│  → returns campaignId to frontend                        │
│                                                          │
│  GET /api/campaigns/:id/progress (SSE)                   │
│  → streams real-time progress updates to frontend        │
└──────────────────────┬───────────────────────────────────┘
                       │ BullMQ job
                       ▼
┌──────────────────────────────────────────────────────────┐
│ Campaign Worker (Node.js process)                        │
│                                                          │
│  Listens on: "campaign-queue"                            │
│  Concurrency: 2 (max 2 campaigns simultaneously)        │
│                                                          │
│  For each campaign job:                                  │
│    1. Load Industry Profile from DB                      │
│    2. Phase A: SCRAPE (Google API calls)                 │
│    3. Phase B: CHECK (HTTP + Lighthouse + AI Vision)     │
│    4. Phase C: ENRICH (Place Details + Score + Save)     │
│    5. Emit campaign.completed event                      │
│                                                          │
│  Progress: Emits campaign.progress events via Redis PubSub│
│  Resume: Reads progressLog from Campaign record on start │
│  Errors: Logs to pino, emits campaign.error event        │
└──────────────────────────────────────────────────────────┘
```

### Queue Configuration

```typescript
// queues/campaign.queue.ts
import { Queue, Worker } from 'bullmq';
import { redis } from '../lib/redis';

export const campaignQueue = new Queue('campaign-queue', {
  connection: redis,
  defaultJobOptions: {
    attempts: 3,
    backoff: { type: 'exponential', delay: 60000 },
    removeOnComplete: { count: 100 },
    removeOnFail: { count: 50 },
  },
});

// Job data shape:
interface CampaignJobData {
  campaignId: string;
  industryTag: string;
  cities: string[];
  mode: 'FULL' | 'PREVIEW' | 'DRY_RUN';
  budgetCents: number | null;
}
```

### Rate Limiter (shared across all workers)

```typescript
// lib/rate-limiter.ts
import { RateLimiterRedis } from 'rate-limiter-flexible';
import { redis } from './redis';

// Google Places API: 600 QPM = 10 QPS
export const googleRateLimiter = new RateLimiterRedis({
  storeClient: redis,
  keyPrefix: 'rl:google',
  points: 10,         // 10 requests
  duration: 1,        // per 1 second
});

// Google PageSpeed API: 400 QPM ≈ 6 QPS
export const lighthouseRateLimiter = new RateLimiterRedis({
  storeClient: redis,
  keyPrefix: 'rl:lighthouse',
  points: 6,
  duration: 1,
});

// Anthropic Haiku: 100 RPM (Tier 1) 
export const haikuRateLimiter = new RateLimiterRedis({
  storeClient: redis,
  keyPrefix: 'rl:haiku',
  points: 100,
  duration: 60,
});
```

---

## 6. THE FLOW: PHASE A – SCRAPE

### 6.1 Pre-Scrape Intelligence

Before the operator clicks "Start", we show market intelligence:

```typescript
async function getMarketIntelligence(industryTag: string, city: string): Promise<MarketIntel> {
  // 1. One Google Text Search call to get total result count
  const response = await googleTextSearch(`${industryProfile.keywords[0]} ${city}`, {
    fieldMask: 'places.id',  // minimal fields = cheapest call
    maxResultCount: 1,       // we only need the totalCount from metadata
  });
  const estimatedTotal = response.metadata?.totalResultCount ?? 0;
  
  // 2. Check how many we already have in DB
  const alreadyScraped = await prisma.lead.count({
    where: { industryTag, city, status: { not: 'RAW' } }
  });
  const blacklisted = await prisma.blacklist.count({
    where: { /* domain matches city pattern */ }
  });
  
  // 3. Calculate
  const estimatedNew = Math.max(0, estimatedTotal - alreadyScraped - blacklisted);
  const estimatedQualified = Math.round(estimatedNew * 0.35); // ~35% qualification rate avg
  const estimatedCostCents = Math.round(estimatedNew * 2.9);  // ~€0.029 per qualified lead
  const estimatedDurationMin = Math.round(estimatedNew * 0.15); // ~9 seconds per lead avg
  
  // 4. Seasonal note from industry profile
  const currentMonth = new Date().getMonth() + 1;
  const seasonalNote = industryProfile.outreach.seasonalNotes[currentMonth] ?? null;
  
  return {
    estimatedTotal,
    alreadyScraped,
    blacklisted,
    estimatedNew,
    estimatedQualified,
    estimatedCostCents,
    estimatedDurationMin,
    seasonalNote,
    spamSaturationRisk: industryProfile.outreach.spamSaturationRisk,
  };
}
```

### 6.2 Google Text Search

```typescript
// We use Text Search (NOT Nearby Search) because:
// - Better results for our use case (searches by business type, not just location)
// - Finds businesses that describe themselves differently than their Google category
// - Works across the entire city, not just a radius from one point

const GOOGLE_TEXT_SEARCH_ENDPOINT = 'https://places.googleapis.com/v1/places:searchText';
const GOOGLE_PLACE_DETAILS_ENDPOINT = 'https://places.googleapis.com/v1/places/';

// Cost: €0.032 per Text Search call (returns up to 20 results)
// Cost: €0.017 per Place Details call

const SEARCH_FIELD_MASK = [
  'places.id',
  'places.displayName',
  'places.formattedAddress',
  'places.addressComponents',
  'places.location',                // lat/lng
  'places.rating',
  'places.userRatingCount',
  'places.websiteUri',
  'places.nationalPhoneNumber',
  'places.primaryType',
  'places.priceLevel',
  'places.photos',                  // photo references (count + metadata)
  'places.editorialSummary',        // Google's business description
  'places.googleMapsUri',
  'places.businessStatus',
  'places.currentOpeningHours',
].join(',');

async function searchGoogle(keyword: string, city: string, pageToken?: string): Promise<GoogleSearchResult> {
  await googleRateLimiter.consume('text-search', 1);
  
  const body: any = {
    textQuery: `${keyword} ${city}`,
    languageCode: 'de',
    regionCode: 'DE',
    maxResultCount: 20,
  };
  if (pageToken) body.pageToken = pageToken;
  
  const response = await fetch(GOOGLE_TEXT_SEARCH_ENDPOINT, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-Goog-Api-Key': process.env.GOOGLE_API_KEY,
      'X-Goog-FieldMask': SEARCH_FIELD_MASK,
    },
    body: JSON.stringify(body),
  });
  
  await logApiCost('google_text_search', GOOGLE_TEXT_SEARCH_ENDPOINT, 3.2); // €0.032 = 3.2 cents
  
  return response.json();
}
```

### 6.3 Adaptive Grid Algorithm

Google Text Search returns max 60 results (3 pages × 20). For large cities with >60 businesses, we need to search in QUADRANTS:

```typescript
async function scrapeCity(
  industryProfile: IndustryProfile,
  city: string,
  campaignId: string
): Promise<void> {
  const keywords = industryProfile.scraping.keywords;
  const allPlaceIds = new Set<string>(); // for dedup across quadrants
  
  // Start with the primary keyword (highest relevance)
  const primaryKeyword = keywords[0]; // e.g., "Coach"
  
  // First call: check total results
  const firstResult = await searchGoogle(primaryKeyword, city);
  const totalEstimated = firstResult.metadata?.totalResultCount ?? firstResult.places?.length ?? 0;
  
  if (totalEstimated <= 60) {
    // Small city: single search with pagination is enough
    await processSearchResults(firstResult, allPlaceIds, campaignId, industryProfile);
    
    // Paginate (up to 3 pages total)
    let pageToken = firstResult.nextPageToken;
    while (pageToken) {
      const nextPage = await searchGoogle(primaryKeyword, city, pageToken);
      await processSearchResults(nextPage, allPlaceIds, campaignId, industryProfile);
      pageToken = nextPage.nextPageToken;
    }
  } else {
    // Large city: grid search
    // Get city bounding box from first result's location
    const cityCenter = firstResult.places?.[0]?.location;
    if (!cityCenter) return;
    
    // Split into 4 quadrants, each with their own search
    const offsets = [
      { lat: +0.02, lng: +0.02 },  // NE
      { lat: +0.02, lng: -0.02 },  // NW
      { lat: -0.02, lng: +0.02 },  // SE
      { lat: -0.02, lng: -0.02 },  // SW
    ];
    
    for (const offset of offsets) {
      const quadrantCenter = {
        lat: cityCenter.latitude + offset.lat,
        lng: cityCenter.longitude + offset.lng,
      };
      
      // Use locationBias to focus search on this quadrant
      const result = await fetch(GOOGLE_TEXT_SEARCH_ENDPOINT, {
        method: 'POST',
        headers: { /* same as above */ },
        body: JSON.stringify({
          textQuery: `${primaryKeyword} ${city}`,
          languageCode: 'de',
          regionCode: 'DE',
          maxResultCount: 20,
          locationBias: {
            circle: {
              center: quadrantCenter,
              radius: 3000.0,  // 3km radius per quadrant
            }
          }
        }),
      });
      await logApiCost('google_text_search', GOOGLE_TEXT_SEARCH_ENDPOINT, 3.2);
      
      const data = await result.json();
      await processSearchResults(data, allPlaceIds, campaignId, industryProfile);
      
      // Paginate within quadrant
      let pageToken = data.nextPageToken;
      while (pageToken) {
        const nextPage = await searchGoogle(primaryKeyword, city, pageToken);
        await processSearchResults(nextPage, allPlaceIds, campaignId, industryProfile);
        pageToken = nextPage.nextPageToken;
      }
    }
  }
  
  // If primary keyword found <20 results: try additional keywords
  if (allPlaceIds.size < 20 && keywords.length > 1) {
    for (const keyword of keywords.slice(1, 4)) { // max 3 additional keywords
      const result = await searchGoogle(keyword, city);
      await processSearchResults(result, allPlaceIds, campaignId, industryProfile);
      // Don't paginate secondary keywords (cost control)
    }
  }
}
```

### 6.4 Immediate Filters (before any HTTP calls)

```typescript
async function processSearchResults(
  searchResult: GoogleSearchResult,
  seenPlaceIds: Set<string>,
  campaignId: string,
  industryProfile: IndustryProfile
): Promise<void> {
  for (const place of searchResult.places ?? []) {
    const placeId = place.id;
    
    // === FILTER 1: Already seen in this search (cross-quadrant dedup) ===
    if (seenPlaceIds.has(placeId)) continue;
    seenPlaceIds.add(placeId);
    
    // === FILTER 2: Permanently closed ===
    if (place.businessStatus === 'CLOSED_PERMANENTLY') continue;
    
    // === FILTER 3: Category check (3-tier soft filter) ===
    const category = place.primaryType?.toLowerCase() ?? '';
    if (industryProfile.scraping.excludedCategories.includes(category)) continue; // hard exclude
    const categoryAccepted = industryProfile.scraping.acceptedCategories.includes(category);
    // If no category or unknown category: don't exclude, but flag for verification
    
    // === FILTER 4: Chain exclusion ===
    const firmName = place.displayName?.text ?? '';
    if (industryProfile.scraping.excludeChains.some(chain => 
      firmName.toLowerCase().includes(chain.toLowerCase())
    )) continue;
    
    // === FILTER 5: Duplicate check against ENTIRE database ===
    const existingByPlaceId = await prisma.lead.findUnique({ where: { googlePlaceId: placeId } });
    if (existingByPlaceId) {
      // Already in DB: update campaign association if needed, skip scraping
      continue;
    }
    
    // Normalize domain for domain-based dedup
    const websiteUrl = place.websiteUri ?? null;
    if (websiteUrl) {
      const domain = normalizeDomain(websiteUrl);
      const existingByDomain = await prisma.lead.findFirst({ 
        where: { actualUrl: { contains: domain } }
      });
      if (existingByDomain) continue;
    }
    
    // Check blacklist
    const isBlacklisted = await prisma.blacklist.findFirst({
      where: { OR: [{ googlePlaceId: placeId }, { domain: websiteUrl ? normalizeDomain(websiteUrl) : undefined }] }
    });
    if (isBlacklisted) continue;
    
    // === FILTER 6: Website URL routing ===
    if (!websiteUrl) {
      if (industryProfile.scraping.noWebsiteAction === 'trackB') {
        // Save as Track B lead (no website, future feature)
        await prisma.lead.create({
          data: {
            googlePlaceId: placeId,
            firmName,
            address: place.formattedAddress ?? '',
            city: extractCity(place.addressComponents),
            campaignId,
            industryTag: industryProfile.tag,
            status: 'NO_WEBSITE',
            googleRating: place.rating ?? null,
            googleReviewCount: place.userRatingCount ?? 0,
            googleCategory: category || null,
            googleMapsUrl: place.googleMapsUri ?? null,
            // ... other Google fields
          }
        });
      }
      continue; // Skip to next place (no website to check)
    }
    
    // === PASSED ALL FILTERS: Create raw lead and proceed to Phase B ===
    const lead = await prisma.lead.create({
      data: {
        googlePlaceId: placeId,
        firmName,
        address: place.formattedAddress ?? '',
        city: extractCity(place.addressComponents),
        postalCode: extractPostalCode(place.addressComponents),
        latitude: place.location?.latitude ?? null,
        longitude: place.location?.longitude ?? null,
        phone: place.nationalPhoneNumber ?? null,
        campaignId,
        industryTag: industryProfile.tag,
        status: 'RAW',
        googleListedUrl: websiteUrl,
        googleRating: place.rating ?? null,
        googleReviewCount: place.userRatingCount ?? 0,
        googleCategory: category || null,
        googlePriceLevel: place.priceLevel ?? null,
        googlePhotosCount: place.photos?.length ?? 0,
        googleBusinessDescription: place.editorialSummary?.text ?? null,
        googleMapsUrl: place.googleMapsUri ?? null,
        noGoogleRating: (place.rating === null || place.rating === undefined),
        source: 'google_maps',
      }
    });
    
    // Queue for Phase B (check website)
    await checkWebsite(lead.id, industryProfile);
  }
}
```

### 6.5 Domain Normalization

```typescript
function normalizeDomain(url: string): string {
  try {
    const parsed = new URL(url.startsWith('http') ? url : `https://${url}`);
    return parsed.hostname
      .replace(/^www\./, '')     // remove www.
      .toLowerCase();            // lowercase
  } catch {
    return url.toLowerCase().replace(/^https?:\/\//, '').replace(/^www\./, '').split('/')[0];
  }
}

// Examples:
// "https://www.friseur-mueller.de/index.html?ref=google" → "friseur-mueller.de"
// "http://friseur-mueller.de" → "friseur-mueller.de"
// "friseur-mueller.jimdosite.com" → "friseur-mueller.jimdosite.com"
```

---

## 7. THE FLOW: PHASE B – CHECK WEBSITE

Phase B runs for every lead that passed Phase A filters. It has 4 sub-steps, executed IN ORDER with early-exit logic.

### 7.1 HTTP Quick-Check (€0.00, ~1 second)

```typescript
import * as cheerio from 'cheerio';

const BROWSER_USER_AGENT = 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36';

const AGGREGATOR_DOMAINS = [
  'facebook.com', 'instagram.com', 'linkedin.com', 'xing.com',
  'treatwell.de', 'treatwell.com', 'booksy.com', 'jameda.de',
  'doctolib.de', 'lieferando.de', 'yelp.de', 'yelp.com',
  'tripadvisor.de', 'tripadvisor.com', 'booking.com',
  'check24.de', 'myhammer.de', 'anwalt.de', 'gelbeseiten.de',
  'dasoertliche.de', 'golocal.de', 'kununu.de', 'kennstdueinen.de',
  'google.com/maps', 'maps.google.com',
];

const PARKING_PATTERNS = [
  'registriert bei ionos', 'registriert bei strato', 'this domain is for sale',
  'domain geparkt', 'sedoparking.com', 'parkingcrew.net',
  'webseite befindet sich im aufbau', 'diese webseite wurde gerade erstellt',
  'uniteddomains.de', 'sedo.com/parking', 'diese domain wurde registriert',
  'this domain has been registered', 'buy this domain',
];

const CONSTRUCTION_PATTERNS = [
  'coming soon', 'under construction', 'im aufbau',
  'wird gerade überarbeitet', 'in kürze verfügbar', 'demnächst',
  'relaunch', 'neue webseite', 'wir arbeiten daran',
  'lorem ipsum dolor sit amet', 'example.com',
  'wordpress starter content', 'hello world', 'just another wordpress site',
  'website in arbeit', 'seite wird aktualisiert',
];

const BAUKASTEN_PATTERNS = {
  jimdo: {
    footer: ['made with jimdo', 'jimdo gmbh', 'erstellt mit jimdo'],
    generator: ['jimdo'],
    domain: ['.jimdosite.com', '.jimdofree.de', '.jimdo.com'],
    scripts: ['jimdo.com/app'],
  },
  wix: {
    footer: ['powered by wix', 'wix.com'],
    generator: ['wix.com'],
    domain: ['.wixsite.com', '.wix.com'],
    scripts: ['static.wixstatic.com', 'parastorage.com'],
  },
  squarespace: {
    generator: ['squarespace'],
    scripts: ['squarespace.com/universal', 'squarespace-cdn.com'],
    domain: ['.squarespace.com'],
  },
  wordpress_com: {
    domain: ['.wordpress.com'],
    generator: ['wordpress.com'],
  },
  weebly: {
    footer: ['powered by weebly'],
    domain: ['.weebly.com'],
    scripts: ['weeblycloud.com'],
  },
  webflow: {
    generator: ['webflow'],
    scripts: ['webflow.com', 'assets.website-files.com'],
  },
};

const BOOKING_PATTERNS = [
  { pattern: 'calendly.com', platform: 'calendly' },
  { pattern: 'treatwell', platform: 'treatwell' },
  { pattern: 'booksy.com', platform: 'booksy' },
  { pattern: 'shore.com', platform: 'shore' },
  { pattern: 'eversports', platform: 'eversports' },
  { pattern: 'tucalendi', platform: 'tucalendi' },
  { pattern: 'simplybook', platform: 'simplybook' },
  { pattern: 'doctolib', platform: 'doctolib' },
  { pattern: 'acuity', platform: 'acuity' },
  { pattern: 'koalendar', platform: 'koalendar' },
  { pattern: 'hubspot.com/meetings', platform: 'hubspot' },
];

async function httpQuickCheck(leadId: string, url: string): Promise<HttpCheckResult> {
  const result: HttpCheckResult = {};
  
  try {
    // Follow redirects manually to track the chain
    const redirectChain: string[] = [];
    let currentUrl = url.startsWith('http') ? url : `https://${url}`;
    let finalResponse: Response | null = null;
    
    for (let i = 0; i < 10; i++) { // max 10 redirects
      const response = await fetch(currentUrl, {
        method: 'GET',
        headers: {
          'User-Agent': BROWSER_USER_AGENT,
          'Accept': 'text/html,application/xhtml+xml',
          'Accept-Language': 'de-DE,de;q=0.9,en;q=0.8',
        },
        redirect: 'manual',
        signal: AbortSignal.timeout(5000), // 5 second timeout
      });
      
      redirectChain.push(currentUrl);
      
      if (response.status >= 300 && response.status < 400) {
        const location = response.headers.get('location');
        if (!location) break;
        currentUrl = new URL(location, currentUrl).href;
        continue;
      }
      
      finalResponse = response;
      break;
    }
    
    if (!finalResponse) {
      return { status: 'REDIRECT_LOOP', redirectChain };
    }
    
    result.httpStatus = finalResponse.status;
    result.redirectChain = redirectChain;
    result.actualUrl = currentUrl;
    
    // === IMMEDIATE EXIT CHECKS ===
    
    // Check: Is final URL a social media / aggregator?
    const finalDomain = normalizeDomain(currentUrl);
    if (AGGREGATOR_DOMAINS.some(d => finalDomain.includes(d))) {
      return { ...result, websiteType: 'AGGREGATOR_PROFILE' };
    }
    
    // Check: Is it a PDF?
    const contentType = finalResponse.headers.get('content-type') ?? '';
    if (contentType.includes('application/pdf')) {
      return { ...result, websiteType: 'PDF_ONLY' };
    }
    
    // Check: Auth required
    if (finalResponse.status === 401 || finalResponse.status === 403) {
      return { ...result, status: 'AUTH_REQUIRED' };
    }
    
    // === PARSE HTTP HEADERS ===
    result.serverType = finalResponse.headers.get('server') ?? null;
    result.behindCdn = result.serverType?.toLowerCase().includes('cloudflare') || 
                       finalResponse.headers.get('x-served-by')?.includes('cache') || false;
    result.cdnProvider = result.behindCdn ? (result.serverType?.toLowerCase().includes('cloudflare') ? 'cloudflare' : 'other') : null;
    
    const lastModified = finalResponse.headers.get('last-modified');
    const poweredBy = finalResponse.headers.get('x-powered-by');
    const hasSecurityHeaders = !!(
      finalResponse.headers.get('content-security-policy') ||
      finalResponse.headers.get('x-frame-options') ||
      finalResponse.headers.get('strict-transport-security')
    );
    
    // === SSL CHECK ===
    result.hasSsl = currentUrl.startsWith('https://');
    if (result.hasSsl) {
      // SSL status: check for common issues
      // Note: Detailed SSL check (expiry, misconfiguration) requires TLS socket
      // For MVP: basic check via URL scheme. Advanced check as future improvement.
      result.sslStatus = 'VALID'; // assume valid if HTTPS works
    } else {
      result.sslStatus = 'MISSING';
    }
    
    // === PARSE HTML ===
    const html = await finalResponse.text();
    const $ = cheerio.load(html);
    const bodyText = $('body').text().replace(/\s+/g, ' ').trim();
    const bodyTextLower = bodyText.toLowerCase();
    const htmlLower = html.toLowerCase();
    
    // --- HEAD SIGNALS ---
    result.hasViewportMeta = !!($('meta[name="viewport"]').length);
    result.titleTag = $('title').text().trim() || null;
    result.metaDescription = $('meta[name="description"]').attr('content') ?? null;
    result.hasH1 = !!($('h1').length);
    
    // Generator meta (CMS detection)
    const generator = $('meta[name="generator"]').attr('content') ?? '';
    result.cmsType = detectCms(generator, htmlLower, currentUrl);
    result.cmsVersion = extractCmsVersion(generator);
    
    // --- BODY SIGNALS ---
    
    // Copyright year
    const copyrightMatch = bodyText.match(/©\s*(\d{4})/);
    result.copyrightYear = copyrightMatch ? parseInt(copyrightMatch[1]) : null;
    
    // Tel link
    result.hasTelLink = !!($('a[href^="tel:"]').length);
    
    // Contact form
    const forms = $('form');
    result.hasContactForm = forms.length > 0;
    if (result.hasContactForm) {
      const form = forms.first();
      const action = form.attr('action') ?? '';
      result.formQuality = {
        hasAction: action !== '' && action !== '#',
        actionTarget: detectFormTarget(action, htmlLower),
        hasSubmit: !!($('button[type="submit"], input[type="submit"]', form).length),
        fieldCount: $('input, textarea, select', form).length,
        hasEmailField: !!($('input[type="email"]', form).length),
        hasCaptcha: htmlLower.includes('recaptcha') || htmlLower.includes('hcaptcha'),
        estimatedQuality: 'functional', // will be refined
      };
    }
    
    // Booking widget
    result.hasBookingWidget = false;
    result.bookingPlatform = null;
    for (const bp of BOOKING_PATTERNS) {
      if (htmlLower.includes(bp.pattern)) {
        result.hasBookingWidget = true;
        result.bookingPlatform = bp.platform;
        break;
      }
    }
    
    // E-commerce detection
    result.hasEcommerce = !!(
      htmlLower.includes('add-to-cart') ||
      htmlLower.includes('warenkorb') ||
      htmlLower.includes('shopify') ||
      htmlLower.includes('woocommerce') ||
      htmlLower.includes('/shop/') ||
      $('meta[property="og:type"][content="product"]').length
    );
    
    // Email on website
    const emailMatch = bodyText.match(/[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}/);
    result.emailFoundOnWebsite = emailMatch ? emailMatch[0] : null;
    
    // Social links
    result.socialLinks = {
      instagram: $('a[href*="instagram.com"]').attr('href') ?? null,
      facebook: $('a[href*="facebook.com"]').attr('href') ?? null,
      linkedin: $('a[href*="linkedin.com"]').attr('href') ?? null,
    };
    result.instagramHandle = extractInstagramHandle(result.socialLinks.instagram);
    
    // Font detection (from Google Fonts link)
    const fontLink = $('link[href*="fonts.googleapis.com"]').attr('href') ?? '';
    const fontMatch = fontLink.match(/family=([^&:]+)/);
    result.primaryFont = fontMatch ? fontMatch[1].replace(/\+/g, ' ') : null;
    
    // Blog detection
    const blogLink = $('a[href*="/blog"], a[href*="/aktuelles"], a[href*="/news"], a[href*="/journal"], a[href*="/beiträge"]');
    result.hasBlog = blogLink.length > 0;
    if (result.hasBlog) {
      // Try to find dates in the page
      const dateMatches = bodyText.match(/\b(20\d{2})\b/g) ?? [];
      const years = dateMatches.map(Number).filter(y => y >= 2015 && y <= new Date().getFullYear());
      result.latestBlogDate = years.length > 0 ? new Date(Math.max(...years), 0, 1) : null;
    }
    
    // Tracking detection
    result.hasGoogleAnalytics = htmlLower.includes('google-analytics.com') || htmlLower.includes('gtag(') || htmlLower.includes('googletagmanager.com');
    result.hasFacebookPixel = htmlLower.includes('fbevents.js') || htmlLower.includes('facebook.com/tr');
    result.hasCookieConsent = htmlLower.includes('cookie-consent') || htmlLower.includes('cookieconsent') || 
                             htmlLower.includes('cookie-notice') || htmlLower.includes('cookie-banner') ||
                             htmlLower.includes('cc-banner') || htmlLower.includes('cookie-accept');
    result.likelyDsgvoViolation = (result.hasGoogleAnalytics || result.hasFacebookPixel) && !result.hasCookieConsent;
    
    // Language detection
    const htmlLang = $('html').attr('lang') ?? '';
    result.websiteLanguage = htmlLang.substring(0, 2).toLowerCase() || detectLanguage(bodyText);
    
    // SPA detection
    const visibleTextLength = bodyText.split(/\s+/).filter(w => w.length > 1).length;
    result.isSpa = visibleTextLength < 50 && (htmlLower.includes('"root"') || htmlLower.includes('"app"') || htmlLower.includes('__next'));
    result.homepageWordCount = visibleTextLength;
    
    // Parking detection
    result.isParked = PARKING_PATTERNS.some(p => bodyTextLower.includes(p));
    
    // Under construction detection
    result.isUnderConstruction = CONSTRUCTION_PATTERNS.some(p => bodyTextLower.includes(p));
    
    // Baukasten detection
    result.baukastenPlatform = null;
    result.websitePlatformTier = 'UNKNOWN';
    for (const [platform, patterns] of Object.entries(BAUKASTEN_PATTERNS)) {
      const isMatch = 
        patterns.footer?.some(f => bodyTextLower.includes(f)) ||
        patterns.generator?.some(g => generator.toLowerCase().includes(g)) ||
        patterns.domain?.some(d => currentUrl.toLowerCase().includes(d)) ||
        patterns.scripts?.some(s => htmlLower.includes(s));
      if (isMatch) {
        result.baukastenPlatform = platform;
        const hasOwnDomain = !patterns.domain?.some(d => currentUrl.toLowerCase().includes(d));
        result.websitePlatformTier = hasOwnDomain ? 'PAID_BAUKASTEN' : 'FREE_BAUKASTEN';
        break;
      }
    }
    if (!result.baukastenPlatform && result.cmsType === 'wordpress') {
      result.websitePlatformTier = 'CUSTOM';
    }
    
    // Footer "Webdesign by" detection
    const footerText = $('footer').text().toLowerCase();
    const webdesignMatch = footerText.match(/(?:webdesign|design|erstellt|gestaltet|powered)\s*(?:by|von|:)\s*([^|•·\n.]+)/i);
    result.websiteBuiltBy = webdesignMatch ? webdesignMatch[1].trim() : null;
    result.websiteBuiltByType = result.websiteBuiltBy 
      ? (result.websiteBuiltBy.match(/gmbh|gbr|ug|agentur|studio|media|design/i) ? 'AGENCY' : 'AMATEUR')
      : 'UNKNOWN';
    
    // Domain type
    const domain = normalizeDomain(currentUrl);
    result.domainType = detectDomainType(domain);
    
    // Page count estimate (from sitemap or navigation)
    const navLinks = new Set<string>();
    $('nav a[href], header a[href]').each((_, el) => {
      const href = $(el).attr('href') ?? '';
      if (href.startsWith('/') || href.startsWith(currentUrl)) navLinks.add(href);
    });
    result.estimatedPageCount = Math.max(navLinks.size, 1);
    
    // Check sitemap
    try {
      const sitemapResponse = await fetch(new URL('/sitemap.xml', currentUrl).href, {
        signal: AbortSignal.timeout(3000),
        headers: { 'User-Agent': BROWSER_USER_AGENT },
      });
      if (sitemapResponse.ok) {
        const sitemapText = await sitemapResponse.text();
        const urlCount = (sitemapText.match(/<loc>/g) ?? []).length;
        if (urlCount > 0) result.estimatedPageCount = urlCount;
      }
    } catch { /* sitemap not available, use nav count */ }
    
    // robots.txt check
    try {
      const robotsResponse = await fetch(new URL('/robots.txt', currentUrl).href, {
        signal: AbortSignal.timeout(3000),
        headers: { 'User-Agent': BROWSER_USER_AGENT },
      });
      if (robotsResponse.ok) {
        const robotsTxt = await robotsResponse.text();
        if (robotsTxt.includes('Disallow: /') && !robotsTxt.includes('Allow:')) {
          result.robotsTxtStatus = 'BLOCKS_ALL';
        } else {
          result.robotsTxtStatus = 'NORMAL';
          const sitemapMatch = robotsTxt.match(/Sitemap:\s*(.+)/i);
          if (sitemapMatch) result.sitemapUrl = sitemapMatch[1].trim();
        }
      } else {
        result.robotsTxtStatus = 'NONE';
      }
    } catch {
      result.robotsTxtStatus = 'NONE';
    }
    
    // Homepage text hash (for duplicate content detection)
    const first200Words = bodyText.split(/\s+/).slice(0, 200).join(' ').toLowerCase();
    result.homepageTextHash = createHash('sha256').update(first200Words).digest('hex');
    
    // Template structure hash (for white-label detection)
    const structureOnly = html.replace(/>([^<]+)</g, '><').replace(/\s+/g, '');
    result.templateStructureHash = createHash('sha256').update(structureOnly).digest('hex').substring(0, 16);
    
    // Website activity estimate
    const currentYear = new Date().getFullYear();
    if (result.copyrightYear && result.copyrightYear >= currentYear - 1) {
      result.websiteLastActivity = 'ACTIVELY_MAINTAINED';
    } else if (result.copyrightYear && result.copyrightYear >= currentYear - 3) {
      result.websiteLastActivity = 'STALE';
    } else {
      result.websiteLastActivity = 'ABANDONED';
    }
    
    // Rating mismatch detection
    const ratingOnWebsite = bodyText.match(/(\d[.,]\d)\s*(?:Sterne?|★|stars?|von\s*5|\/\s*5)/i);
    if (ratingOnWebsite) {
      const websiteRating = parseFloat(ratingOnWebsite[1].replace(',', '.'));
      // googleRating is passed separately and compared in the scoring step
      result.websiteDisplayedRating = websiteRating;
    }
    
    return result;
    
  } catch (error) {
    if (error instanceof DOMException && error.name === 'TimeoutError') {
      return { status: 'TIMEOUT' };
    }
    if (error instanceof TypeError && error.message.includes('fetch failed')) {
      return { status: 'DNS_ERROR' };
    }
    return { status: 'ERROR', error: String(error) };
  }
}
```

### 7.2 Lighthouse Quick-Audit (€0.003, ~3-5 seconds)

```typescript
const PAGESPEED_API = 'https://www.googleapis.com/pagespeedonline/v5/runPagespeed';

async function lighthouseQuickCheck(url: string): Promise<LighthouseResult> {
  await lighthouseRateLimiter.consume('pagespeed', 1);
  
  const params = new URLSearchParams({
    url,
    key: process.env.GOOGLE_API_KEY!,
    strategy: 'mobile',                    // mobile-first
    category: 'performance',
    category: 'seo',
    category: 'accessibility',
  });
  
  try {
    const response = await fetch(`${PAGESPEED_API}?${params}`, {
      signal: AbortSignal.timeout(15000), // Lighthouse can take up to 15s
    });
    
    await logApiCost('lighthouse', PAGESPEED_API, 0.3); // ~€0.003
    
    if (!response.ok) return { error: `HTTP ${response.status}` };
    
    const data = await response.json();
    const categories = data.lighthouseResult?.categories ?? {};
    
    return {
      performance: Math.round((categories.performance?.score ?? 0) * 100),
      seo: Math.round((categories.seo?.score ?? 0) * 100),
      accessibility: Math.round((categories.accessibility?.score ?? 0) * 100),
    };
  } catch (error) {
    return { error: String(error) };
  }
}
```

### 7.3 Quick-Score Calculation (€0.00, instant)

```typescript
function calculateQuickScore(
  lead: Lead,
  httpResult: HttpCheckResult,
  lighthouseResult: LighthouseResult,
  industryProfile: IndustryProfile
): QuickScoreResult {
  const fw = industryProfile.qualityCheck.functionalWeights;
  const vw = industryProfile.qualityCheck.visualWeights;
  
  let functionalScore = 0;
  let visualScore = 0;
  const breakdown: Record<string, number> = {};
  const weaknesses: string[] = [];
  let checksCompleted = 0;
  
  // === FUNCTIONAL CHECKS ===
  
  // No CTA / No booking
  if (!httpResult.hasBookingWidget && !httpResult.hasContactForm && !httpResult.hasTelLink) {
    const points = fw.no_cta_or_booking ?? 0;
    functionalScore += points;
    breakdown.no_cta_or_booking = points;
    weaknesses.push('no_cta_or_booking');
  }
  checksCompleted++;
  
  // Not mobile optimized
  if (httpResult.hasViewportMeta === false) {
    const points = fw.not_mobile ?? 0;
    functionalScore += points;
    breakdown.not_mobile = points;
    weaknesses.push('not_mobile');
  }
  checksCompleted++;
  
  // No contact form
  if (!httpResult.hasContactForm) {
    const points = fw.no_contact_form ?? 0;
    functionalScore += points;
    breakdown.no_contact_form = points;
    weaknesses.push('no_contact_form');
  }
  checksCompleted++;
  
  // No tel link
  if (!httpResult.hasTelLink) {
    const points = fw.no_tel_link ?? 0;
    functionalScore += points;
    breakdown.no_tel_link = points;
    weaknesses.push('no_tel_link');
  }
  checksCompleted++;
  
  // Slow loading
  if (lighthouseResult.performance !== undefined && lighthouseResult.performance < 40) {
    const points = fw.slow_loading ?? 0;
    functionalScore += points;
    breakdown.slow_loading = points;
    weaknesses.push('slow_loading');
  }
  if (lighthouseResult.performance !== undefined) checksCompleted++;
  
  // No SSL
  if (!httpResult.hasSsl) {
    const points = fw.no_ssl ?? 0;
    functionalScore += points;
    breakdown.no_ssl = points;
    weaknesses.push('no_ssl');
  }
  checksCompleted++;
  
  // No SEO basics
  if (!httpResult.titleTag || httpResult.titleTag === 'Home' || !httpResult.metaDescription) {
    const points = fw.no_seo ?? 0;
    functionalScore += points;
    breakdown.no_seo = points;
    weaknesses.push('no_seo');
  }
  checksCompleted++;
  
  // Industry-specific functional checks
  if (fw.no_booking_tool && !httpResult.hasBookingWidget) {
    functionalScore += fw.no_booking_tool;
    breakdown.no_booking_tool = fw.no_booking_tool;
    weaknesses.push('no_booking_tool');
  }
  if (fw.no_price_list) {
    // Check for price indicators in the HTML
    const hasPrice = httpResult.bodyTextLower?.includes('€') || 
                     httpResult.bodyTextLower?.includes('preis') ||
                     httpResult.bodyTextLower?.includes('euro');
    if (!hasPrice) {
      functionalScore += fw.no_price_list;
      breakdown.no_price_list = fw.no_price_list;
      weaknesses.push('no_price_list');
    }
    checksCompleted++;
  }
  if (fw.opening_hours_hidden) {
    const hasHours = httpResult.bodyTextLower?.includes('öffnungszeit') ||
                     httpResult.bodyTextLower?.includes('geöffnet') ||
                     httpResult.bodyTextLower?.includes('mo-fr') ||
                     httpResult.bodyTextLower?.includes('montag');
    if (!hasHours) {
      functionalScore += fw.opening_hours_hidden;
      breakdown.opening_hours_hidden = fw.opening_hours_hidden;
      weaknesses.push('opening_hours_hidden');
    }
    checksCompleted++;
  }
  
  // === VISUAL CHECKS (from technical signals, no AI yet) ===
  
  // Design outdated (estimated from copyright year + CMS version + CSS libraries)
  const designAge = estimateDesignAge(httpResult);
  if (designAge === 'pre-2015' || designAge === '2015-2019') {
    const points = vw.design_outdated ?? 0;
    visualScore += points;
    breakdown.design_outdated = points;
    weaknesses.push('design_outdated');
  }
  checksCompleted++;
  
  // Baukasten visible
  if (httpResult.baukastenPlatform && httpResult.websitePlatformTier === 'FREE_BAUKASTEN') {
    const points = vw.baukasten_visible ?? 0;
    visualScore += points;
    breakdown.baukasten_visible = points;
    weaknesses.push('baukasten_visible');
  }
  checksCompleted++;
  
  // Copyright old
  if (httpResult.copyrightYear && (new Date().getFullYear() - httpResult.copyrightYear > 2)) {
    const points = vw.copyright_old ?? 0;
    visualScore += points;
    breakdown.copyright_old = points;
    weaknesses.push('copyright_old');
  }
  checksCompleted++;
  
  // Generic text (banned phrases check)
  const bannedPhrases = industryProfile.qualityCheck.bannedPhrases ?? [];
  const bannedMatches = bannedPhrases.filter(phrase => 
    httpResult.bodyTextLower?.includes(phrase.toLowerCase())
  );
  if (bannedMatches.length >= 3) {
    const points = vw.generic_text ?? 0;
    visualScore += points;
    breakdown.generic_text = points;
    weaknesses.push('generic_text');
  }
  checksCompleted++;
  
  // === BONUS / MALUS ===
  let bonus = 0;
  
  // Good business + bad website = gold opportunity
  if (lead.googleRating && lead.googleRating >= 4.3 && lead.googleReviewCount >= 20) bonus += 5;
  // Has ad tracking (investing in online already)
  if (httpResult.hasGoogleAnalytics || httpResult.hasFacebookPixel) bonus += 5;
  // High competition density  
  if (lead.competitorDensity && lead.competitorDensity > 20) bonus += 3;
  // Free baukasten
  if (httpResult.websitePlatformTier === 'FREE_BAUKASTEN') bonus += 3;
  // Abandoned blog
  if (httpResult.blogStatus === 'ABANDONED') bonus += 3;
  // Expired SSL
  if (httpResult.sslStatus === 'EXPIRED') bonus += 3;
  // Growing business
  if (lead.growthSignal === 'GROWING' || lead.growthSignal === 'NEW') bonus += 2;
  // Active Google profile
  if (lead.googleProfileCompleteness && lead.googleProfileCompleteness > 0.8) bonus += 2;
  
  // MALUS
  if (lead.competitorDensity && lead.competitorDensity < 5) bonus -= 3;
  if (lead.googleRating && lead.googleRating < 3.0) bonus -= 5;
  if (lead.googleReviewCount < 3) bonus -= 3;
  if (lead.emotionalTiming === 'BAD') bonus -= 5;
  if (httpResult.isSpa && checksCompleted < 5) bonus -= 3;
  
  // === CALCULATE FINAL SCORE ===
  const rawScore = functionalScore + visualScore + bonus;
  const maxScore = industryProfile.qualityCheck.maxRawScore;
  const quickScore = Math.min(100, Math.max(0, Math.round((rawScore / maxScore) * 100)));
  
  // Confidence
  let confidence: Confidence;
  if (checksCompleted >= 8) confidence = 'HIGH';
  else if (checksCompleted >= 5) confidence = 'MEDIUM';
  else confidence = 'LOW';
  
  return {
    quickScore,
    functionalScore: Math.round((functionalScore / maxScore) * 50),
    visualScore: Math.round((visualScore / maxScore) * 50),
    breakdown,
    weaknesses,
    weaknessCount: weaknesses.length,
    confidence,
    checksCompleted,
    bonus,
  };
}
```

### 7.4 Screenshot + AI Vision (ONLY for gray-zone leads, €0.005, ~15 seconds)

```typescript
import { chromium } from 'playwright';

// === COOKIE BANNER SELECTORS ===
const COOKIE_ACCEPT_SELECTORS = [
  'button:has-text("Alle akzeptieren")',
  'button:has-text("Akzeptieren")',
  'button:has-text("Alles akzeptieren")',
  'button:has-text("Zustimmen")',
  'button:has-text("Einverstanden")',
  'button:has-text("Accept all")',
  'button:has-text("Accept")',
  'button:has-text("OK")',
  'button[id*="accept"]',
  'button[class*="accept"]',
  'a[id*="accept"]',
  '.cookie-consent button:first-child',
  '#cookie-banner button:first-child',
  '[data-testid="cookie-accept"]',
  '.cc-btn.cc-allow',
  '#CybotCookiebotDialogBodyLevelButtonLevelOptinAllowAll',
  'button.cmplz-btn.cmplz-accept',
];

// === PLAYWRIGHT CONFIG ===
const PLAYWRIGHT_CONFIG = {
  browser: 'chromium',
  headless: true,
  viewport_desktop: { width: 1440, height: 900 },
  viewport_mobile: { width: 375, height: 812 },
  userAgent: BROWSER_USER_AGENT,
  locale: 'de-DE',
  timezoneId: 'Europe/Berlin',
  timeout: 15000,    // 15 second page load timeout
  waitAfterLoad: 2000, // 2 seconds for JS to finish
};

async function takeScreenshotAndAnalyze(
  leadId: string,
  url: string
): Promise<ScreenshotResult> {
  const browser = await chromium.launch({ headless: true });
  
  try {
    // === DESKTOP SCREENSHOT ===
    const desktopContext = await browser.newContext({
      viewport: PLAYWRIGHT_CONFIG.viewport_desktop,
      userAgent: PLAYWRIGHT_CONFIG.userAgent,
      locale: PLAYWRIGHT_CONFIG.locale,
      timezoneId: PLAYWRIGHT_CONFIG.timezoneId,
    });
    const desktopPage = await desktopContext.newPage();
    
    // Block heavy resources to speed up
    await desktopPage.route('**/*.{mp4,webm,ogg,avi,mp3,wav}', route => route.abort());
    await desktopPage.route('**/*', route => {
      const url = route.request().url();
      // Block tracking scripts (not needed for screenshot)
      if (url.includes('google-analytics') || url.includes('facebook.com/tr')) {
        return route.abort();
      }
      return route.continue();
    });
    
    await desktopPage.goto(url, { 
      waitUntil: 'networkidle', 
      timeout: PLAYWRIGHT_CONFIG.timeout 
    });
    
    // Try to dismiss cookie banner
    for (const selector of COOKIE_ACCEPT_SELECTORS) {
      try {
        const button = await desktopPage.$(selector);
        if (button) {
          await button.click();
          await desktopPage.waitForTimeout(1000);
          break;
        }
      } catch { /* ignore, try next selector */ }
    }
    
    await desktopPage.waitForTimeout(PLAYWRIGHT_CONFIG.waitAfterLoad);
    
    const desktopBuffer = await desktopPage.screenshot({ 
      type: 'jpeg', 
      quality: 80,
      fullPage: false, // only viewport, not full page
    });
    await desktopContext.close();
    
    // === MOBILE SCREENSHOT ===
    const mobileContext = await browser.newContext({
      viewport: PLAYWRIGHT_CONFIG.viewport_mobile,
      userAgent: 'Mozilla/5.0 (iPhone; CPU iPhone OS 17_0 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/17.0 Mobile/15E148 Safari/604.1',
      locale: PLAYWRIGHT_CONFIG.locale,
      isMobile: true,
      hasTouch: true,
    });
    const mobilePage = await mobileContext.newPage();
    await mobilePage.goto(url, { waitUntil: 'networkidle', timeout: PLAYWRIGHT_CONFIG.timeout });
    
    // Dismiss cookie banner on mobile too
    for (const selector of COOKIE_ACCEPT_SELECTORS) {
      try {
        const button = await mobilePage.$(selector);
        if (button) { await button.click(); await mobilePage.waitForTimeout(1000); break; }
      } catch { /* ignore */ }
    }
    await mobilePage.waitForTimeout(PLAYWRIGHT_CONFIG.waitAfterLoad);
    
    const mobileBuffer = await mobilePage.screenshot({ type: 'jpeg', quality: 80, fullPage: false });
    await mobileContext.close();
    
    // === UPLOAD TO R2 ===
    const desktopUrl = await uploadToR2(`screenshots/${leadId}/desktop.jpg`, desktopBuffer);
    const mobileUrl = await uploadToR2(`screenshots/${leadId}/mobile.jpg`, mobileBuffer);
    
    // === AI VISION ANALYSIS ===
    const aiResult = await analyzeWithVision(desktopBuffer, mobileBuffer);
    
    return {
      screenshotDesktopUrl: desktopUrl,
      screenshotMobileUrl: mobileUrl,
      ...aiResult,
    };
    
  } finally {
    await browser.close();
  }
}
```

### 7.5 AI Vision Prompt (Complete, with calibration)

```typescript
async function analyzeWithVision(
  desktopScreenshot: Buffer,
  mobileScreenshot: Buffer
): Promise<AiVisionResult> {
  await haikuRateLimiter.consume('vision', 1);
  
  const response = await anthropic.messages.create({
    model: 'claude-haiku-4-5-20251001',
    max_tokens: 500,
    messages: [{
      role: 'user',
      content: [
        {
          type: 'image',
          source: { type: 'base64', media_type: 'image/jpeg', data: desktopScreenshot.toString('base64') },
        },
        {
          type: 'image',
          source: { type: 'base64', media_type: 'image/jpeg', data: mobileScreenshot.toString('base64') },
        },
        {
          type: 'text',
          text: `You are evaluating a small business website. The first image is the desktop version, the second is mobile.

SCORING GUIDE for professional_impression (1-10):
1-2: Catastrophic. Broken layout, unreadable, looks like spam or from the 1990s.
3-4: Bad. Clearly outdated (pre-2015 aesthetic), cluttered, generic stock photos, hard to navigate.
5-6: Mediocre. Functional but forgettable. Generic template look. Not embarrassing but not impressive.
7-8: Good. Looks professional and modern. Clear structure. Would trust this business.
9-10: Excellent. Outstanding design. Could be mistaken for a professional agency's work.

Respond ONLY with a JSON object, no other text:
{
  "design_era": "pre-2015" | "2015-2019" | "2020-2023" | "2024+",
  "professional_impression": 1-10,
  "has_hero_section": true/false,
  "has_visible_cta": true/false,
  "has_real_photos": true/false,
  "has_stock_photos": true/false,
  "is_mobile_usable": true/false,
  "biggest_weakness": "one sentence in German",
  "notable_strength": "one sentence in German, or null if nothing notable"
}`
        }
      ]
    }]
  });
  
  await logApiCost('claude_haiku', 'vision', 0.5); // ~€0.005
  
  const text = response.content[0].type === 'text' ? response.content[0].text : '';
  const cleaned = text.replace(/```json\n?|```/g, '').trim();
  
  try {
    return JSON.parse(cleaned);
  } catch {
    logger.warn({ text }, 'Failed to parse AI vision response');
    return { error: 'parse_failed' };
  }
}
```

---

## 8. THE FLOW: PHASE C – ENRICH, SCORE, SAVE

Phase C runs ONLY for qualified leads (quick_score ≥ 40 after Phase B).

### 8.1 Google Place Details (€0.017)

```typescript
const DETAILS_FIELD_MASK = [
  'id', 'reviews', 'currentOpeningHours', 'addressComponents',
  'businessStatus', 'photos', 'editorialSummary',
  'accessibilityOptions', 'paymentOptions', 'parkingOptions',
].join(',');

async function enrichWithPlaceDetails(lead: Lead): Promise<void> {
  await googleRateLimiter.consume('place-details', 1);
  
  const response = await fetch(
    `${GOOGLE_PLACE_DETAILS_ENDPOINT}${lead.googlePlaceId}`,
    {
      headers: {
        'X-Goog-Api-Key': process.env.GOOGLE_API_KEY!,
        'X-Goog-FieldMask': DETAILS_FIELD_MASK,
      },
    }
  );
  await logApiCost('google_place_details', GOOGLE_PLACE_DETAILS_ENDPOINT, 1.7); // €0.017
  
  const data = await response.json();
  
  // Extract top 5 reviews by quality (length > 50 chars, recent, high rating)
  const reviews = (data.reviews ?? [])
    .filter((r: any) => r.text?.text?.length > 50)
    .sort((a: any, b: any) => new Date(b.publishTime).getTime() - new Date(a.publishTime).getTime())
    .slice(0, 5)
    .map((r: any) => ({
      text: r.text.text,
      rating: r.rating,
      date: r.publishTime,
      authorName: r.authorAttribution?.displayName ?? 'Anonym',
    }));
  
  // Google attributes (structured data)
  const attributes: any = {};
  if (data.accessibilityOptions) {
    attributes.wheelchairAccessible = data.accessibilityOptions.wheelchairAccessibleEntrance ?? null;
  }
  if (data.parkingOptions) {
    attributes.parking = data.parkingOptions.paidParkingLot || data.parkingOptions.freeParkingLot || false;
  }
  
  // Owner vs user photos count
  const ownerPhotos = (data.photos ?? []).filter((p: any) => 
    p.authorAttributions?.some((a: any) => a.displayName === lead.firmName)
  );
  
  await prisma.lead.update({
    where: { id: lead.id },
    data: {
      googleReviewTexts: reviews,
      googleAttributes: attributes,
      googleOwnerPhotosCount: ownerPhotos.length,
      googleLatestPhotoDate: data.photos?.[0]?.publishTime ? new Date(data.photos[0].publishTime) : null,
    }
  });
}
```

### 8.2 Business Signals Calculation

```typescript
function calculateBusinessSignals(lead: Lead): BusinessSignals {
  // Estimated size
  let estimatedSize: BusinessSize = 'UNKNOWN';
  if (lead.googleReviewCount < 15) estimatedSize = 'SOLO';
  else if (lead.googleReviewCount < 50) estimatedSize = 'SMALL';
  else estimatedSize = 'MEDIUM';
  
  // Estimated revenue (industry-dependent heuristic)
  let estimatedRevenue: RevenueLevel = 'MEDIUM';
  if (lead.googlePriceLevel && lead.googlePriceLevel >= 3) estimatedRevenue = 'HIGH';
  else if (lead.googlePriceLevel && lead.googlePriceLevel <= 1) estimatedRevenue = 'LOW';
  
  // Growth signal (from review dates)
  let growthSignal: GrowthSignal = 'UNKNOWN';
  const reviews = lead.googleReviewTexts as any[] ?? [];
  if (reviews.length >= 3) {
    const recentReviews = reviews.filter(r => {
      const date = new Date(r.date);
      return date > new Date(Date.now() - 90 * 24 * 60 * 60 * 1000); // last 90 days
    });
    if (recentReviews.length >= 3) growthSignal = 'GROWING';
    else if (recentReviews.length >= 1) growthSignal = 'STABLE';
    else growthSignal = reviews.length > 0 ? 'DECLINING' : 'UNKNOWN';
  }
  if (lead.googleReviewCount > 0 && lead.googleReviewCount <= 5) growthSignal = 'NEW';
  
  // Emotional timing
  let emotionalTiming: TimingSignal = 'NEUTRAL';
  if (reviews.length >= 3) {
    const recentNegative = reviews.filter(r => r.rating <= 2 && 
      new Date(r.date) > new Date(Date.now() - 14 * 24 * 60 * 60 * 1000)
    );
    if (recentNegative.length >= 3) emotionalTiming = 'BAD'; // review shitstorm
    
    const roundNumbers = [50, 100, 200, 500];
    if (roundNumbers.some(n => lead.googleReviewCount >= n && lead.googleReviewCount <= n + 3)) {
      emotionalTiming = 'GOOD'; // just hit a milestone
    }
  }
  
  // Online activity score (0-1)
  let activityScore = 0;
  if (lead.googlePhotosCount > 10) activityScore += 0.25;
  else if (lead.googlePhotosCount > 3) activityScore += 0.15;
  if (lead.hasBlog && lead.blogStatus === 'ACTIVE') activityScore += 0.25;
  if (lead.socialLinks && Object.values(lead.socialLinks as any).some(Boolean)) activityScore += 0.2;
  if (lead.googleBusinessDescription) activityScore += 0.15;
  if (lead.googleOwnerPhotosCount > 0) activityScore += 0.15;
  
  // Google profile completeness
  let completeness = 0;
  if (lead.googleRating) completeness += 0.15;
  if (lead.googleReviewCount > 0) completeness += 0.15;
  if (lead.googleListedUrl) completeness += 0.15;
  if (lead.phone) completeness += 0.10;
  if (lead.googlePhotosCount > 0) completeness += 0.10;
  if (lead.googleBusinessDescription) completeness += 0.10;
  if (lead.googleCategory) completeness += 0.10;
  if (lead.googleAttributes && Object.keys(lead.googleAttributes as any).length > 0) completeness += 0.05;
  
  // Improvement potential
  let improvementPotential: PotentialLevel = 'MEDIUM';
  const contentSignals = [
    lead.homepageWordCount && lead.homepageWordCount > 200,
    lead.hasBlog,
    lead.googleBusinessDescription,
    lead.googlePhotosCount > 5,
    lead.instagramHandle,
  ].filter(Boolean).length;
  if (contentSignals >= 3) improvementPotential = 'HIGH';
  else if (contentSignals <= 1) improvementPotential = 'LOW';
  
  // Reachability score (industry-specific weights from profile)
  // calculated in the priority assignment step
  
  // Strengths detection
  const strengths: string[] = [];
  if (lead.googlePhotosCount > 10) strengths.push('good_google_photos');
  if (lead.homepageWordCount && lead.homepageWordCount > 500) strengths.push('good_text_volume');
  if (lead.instagramHandle) strengths.push('has_instagram');
  if (lead.hasBlog && lead.blogStatus === 'ACTIVE') strengths.push('active_blog');
  if (lead.googleRating && lead.googleRating >= 4.5 && lead.googleReviewCount >= 20) strengths.push('excellent_reviews');
  
  return {
    estimatedSize, estimatedRevenue, growthSignal, emotionalTiming,
    onlineActivityScore: activityScore,
    googleProfileCompleteness: completeness,
    improvementPotential, strengths,
  };
}
```

### 8.3 Priority Tag & Outreach Prep

```typescript
function assignPriorityAndGenerateArguments(
  lead: Lead,
  scoreResult: QuickScoreResult,
  businessSignals: BusinessSignals,
  industryProfile: IndustryProfile
): PriorityResult {
  // Priority tag
  const gc = industryProfile.scoring.goldCriteria;
  let priorityTag: PriorityTag = 'STANDARD';
  let goldReason: string | null = null;
  
  if (
    scoreResult.quickScore >= gc.minQuickScore &&
    lead.googleRating && lead.googleRating >= gc.minGoogleRating &&
    lead.googleReviewCount >= gc.minReviewCount
  ) {
    priorityTag = 'GOLD';
    goldReason = `Score ${scoreResult.quickScore} + Rating ${lead.googleRating}★ + ${lead.googleReviewCount} Reviews`;
  } else if (
    scoreResult.quickScore >= 50 &&
    lead.googleRating && lead.googleRating >= 3.5 &&
    lead.googleReviewCount >= 5
  ) {
    priorityTag = 'SILVER';
  }
  
  // Reachability score
  const rw = industryProfile.scoring.reachabilityWeights;
  let reachabilityScore = 0;
  if (lead.emailFoundOnWebsite) reachabilityScore += rw.email ?? 0;
  if (lead.hasContactForm) reachabilityScore += rw.contactForm ?? 0;
  if (lead.socialLinks && (lead.socialLinks as any).linkedin) reachabilityScore += rw.linkedin ?? 0;
  if (lead.phone) reachabilityScore += rw.phone ?? 0;
  if (lead.instagramHandle) reachabilityScore += rw.instagram ?? 0;
  
  // Spam saturation risk
  const spamRisk = industryProfile.outreach.spamSaturationRisk as RiskLevel;
  
  // Screenshot priority
  const screenshotPrio = industryProfile.display?.screenshotPriority === 'desktop' ? 'DESKTOP' : 'MOBILE';
  
  // Generate outreach arguments from weaknesses
  const weaknessLabels: string[] = [];
  const weaknessArguments: string[] = [];
  
  const argTemplates = industryProfile.outreach.weaknessToArgument;
  for (const weakness of scoreResult.weaknesses) {
    // Label (human-readable short form)
    const labelMap: Record<string, string> = {
      no_cta_or_booking: 'Kein Buchungsweg',
      not_mobile: 'Nicht mobiloptimiert',
      no_contact_form: 'Kein Kontaktformular',
      no_tel_link: 'Telefon nicht klickbar',
      slow_loading: 'Ladezeit zu hoch',
      no_ssl: 'Kein HTTPS',
      no_seo: 'Kein SEO',
      no_booking_tool: 'Kein Buchungstool',
      no_price_list: 'Keine Preise online',
      opening_hours_hidden: 'Öffnungszeiten versteckt',
      design_outdated: 'Design veraltet',
      baukasten_visible: 'Baukasten erkennbar',
      copyright_old: 'Copyright veraltet',
      generic_text: 'Generischer Text',
      no_real_photos: 'Keine echten Fotos',
      no_social_proof: 'Kein Social Proof',
      no_about: 'Kein Über-uns',
      no_own_photos: 'Keine eigenen Fotos',
      no_team: 'Kein Team vorgestellt',
      reviews_not_shown: 'Bewertungen nicht eingebunden',
    };
    weaknessLabels.push(labelMap[weakness] ?? weakness);
    
    // Argument (full outreach sentence, personalized)
    const template = argTemplates[weakness];
    if (template) {
      const argument = template
        .replace('{{firm_name}}', lead.firmName)
        .replace('{{city}}', lead.city)
        .replace('{{google_rating}}', String(lead.googleRating ?? 'N/A'))
        .replace('{{review_count}}', String(lead.googleReviewCount))
        .replace('{{copyright_year}}', String(lead.copyrightYear ?? '?'))
        .replace('{{cta_action}}', industryProfile.outreach.ctaAction ?? 'Termin')
        .replace('{{cta_verb}}', industryProfile.outreach.ctaVerb ?? 'buchen');
      weaknessArguments.push(argument);
    }
  }
  
  // Rating mismatch check
  let ratingMismatch = false;
  let ratingMismatchDetail = null;
  if (lead.websiteDisplayedRating && lead.googleRating) {
    const diff = Math.abs(lead.websiteDisplayedRating - lead.googleRating);
    if (diff > 0.5) {
      ratingMismatch = true;
      ratingMismatchDetail = {
        websiteShows: lead.websiteDisplayedRating,
        googleActual: lead.googleRating,
        difference: diff,
      };
    }
  }
  
  return {
    priorityTag,
    goldReason,
    reachabilityScore,
    spamSaturationRisk: spamRisk,
    screenshotPriority: screenshotPrio,
    weaknessLabels,
    weaknessArguments,
    ratingMismatch,
    ratingMismatchDetail,
  };
}
```

### 8.4 Geo-Clustering

```typescript
function assignGeoClusters(leads: Lead[]): void {
  // Simple clustering: leads within 500m of each other in same industry
  const CLUSTER_RADIUS_KM = 0.5;
  let clusterCounter = 0;
  const assigned = new Set<string>();
  
  for (const lead of leads) {
    if (assigned.has(lead.id) || !lead.latitude || !lead.longitude) continue;
    
    const cluster: Lead[] = [lead];
    for (const other of leads) {
      if (other.id === lead.id || assigned.has(other.id) || !other.latitude || !other.longitude) continue;
      
      const distance = haversineKm(lead.latitude, lead.longitude, other.latitude, other.longitude);
      if (distance <= CLUSTER_RADIUS_KM) {
        cluster.push(other);
      }
    }
    
    if (cluster.length >= 2) {
      const clusterId = `cluster_${++clusterCounter}`;
      for (const c of cluster) {
        c.geoClusterId = clusterId;
        c.geoClusterSize = cluster.length;
        assigned.add(c.id);
      }
    }
  }
}

function haversineKm(lat1: number, lon1: number, lat2: number, lon2: number): number {
  const R = 6371;
  const dLat = (lat2 - lat1) * Math.PI / 180;
  const dLon = (lon2 - lon1) * Math.PI / 180;
  const a = Math.sin(dLat/2) ** 2 + Math.cos(lat1 * Math.PI/180) * Math.cos(lat2 * Math.PI/180) * Math.sin(dLon/2) ** 2;
  return R * 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
}
```

### 8.5 Save and Emit Events

```typescript
async function saveQualifiedLead(
  lead: Lead,
  httpResult: HttpCheckResult,
  lighthouseResult: LighthouseResult,
  scoreResult: QuickScoreResult,
  businessSignals: BusinessSignals,
  priorityResult: PriorityResult,
  screenshotResult?: ScreenshotResult
): Promise<void> {
  // Set screenshot retention
  let screenshotExpiresAt: Date | null = null;
  if (screenshotResult) {
    screenshotExpiresAt = new Date(Date.now() + 180 * 24 * 60 * 60 * 1000); // 6 months
  }
  
  await prisma.lead.update({
    where: { id: lead.id },
    data: {
      status: 'QUALIFIED',
      actualUrl: httpResult.actualUrl,
      websiteType: httpResult.websiteType ?? 'OWN_WEBSITE',
      
      // Technical signals
      hasSsl: httpResult.hasSsl,
      sslStatus: httpResult.sslStatus,
      httpStatus: httpResult.httpStatus,
      redirectChain: httpResult.redirectChain,
      serverType: httpResult.serverType,
      cmsType: httpResult.cmsType,
      cmsVersion: httpResult.cmsVersion,
      hasViewportMeta: httpResult.hasViewportMeta,
      copyrightYear: httpResult.copyrightYear,
      titleTag: httpResult.titleTag,
      metaDescription: httpResult.metaDescription,
      hasH1: httpResult.hasH1,
      primaryFont: httpResult.primaryFont,
      hasContactForm: httpResult.hasContactForm,
      hasTelLink: httpResult.hasTelLink,
      hasBookingWidget: httpResult.hasBookingWidget,
      bookingPlatform: httpResult.bookingPlatform,
      hasEcommerce: httpResult.hasEcommerce,
      estimatedPageCount: httpResult.estimatedPageCount,
      websiteLanguage: httpResult.websiteLanguage,
      isSpa: httpResult.isSpa,
      isParked: httpResult.isParked,
      isUnderConstruction: httpResult.isUnderConstruction,
      homepageWordCount: httpResult.homepageWordCount,
      homepageTextHash: httpResult.homepageTextHash,
      templateStructureHash: httpResult.templateStructureHash,
      
      // Content signals
      hasBlog: httpResult.hasBlog,
      latestBlogDate: httpResult.latestBlogDate,
      blogStatus: httpResult.blogStatus,
      websiteLastActivity: httpResult.websiteLastActivity,
      
      // Tracking
      hasGoogleAnalytics: httpResult.hasGoogleAnalytics,
      hasFacebookPixel: httpResult.hasFacebookPixel,
      hasCookieConsent: httpResult.hasCookieConsent,
      likelyDsgvoViolation: httpResult.likelyDsgvoViolation,
      
      // Reachability
      emailFoundOnWebsite: httpResult.emailFoundOnWebsite,
      socialLinks: httpResult.socialLinks,
      instagramHandle: httpResult.instagramHandle,
      
      // Platform
      websitePlatformTier: httpResult.websitePlatformTier,
      baukastenPlatform: httpResult.baukastenPlatform,
      websiteBuiltBy: httpResult.websiteBuiltBy,
      websiteBuiltByType: httpResult.websiteBuiltByType,
      domainType: httpResult.domainType,
      behindCdn: httpResult.behindCdn,
      cdnProvider: httpResult.cdnProvider,
      formQuality: httpResult.formQuality,
      robotsTxtStatus: httpResult.robotsTxtStatus,
      sitemapUrl: httpResult.sitemapUrl,
      
      // Lighthouse
      lighthousePerformance: lighthouseResult.performance,
      lighthouseSeo: lighthouseResult.seo,
      lighthouseAccessibility: lighthouseResult.accessibility,
      
      // AI Vision (if available)
      ...(screenshotResult ? {
        screenshotDesktopUrl: screenshotResult.screenshotDesktopUrl,
        screenshotMobileUrl: screenshotResult.screenshotMobileUrl,
        aiDesignEra: screenshotResult.design_era,
        aiProfessionalImpression: screenshotResult.professional_impression,
        aiHasHero: screenshotResult.has_hero_section,
        aiHasVisibleCta: screenshotResult.has_visible_cta,
        aiHasRealPhotos: screenshotResult.has_real_photos,
        aiHasStockPhotos: screenshotResult.has_stock_photos,
        aiBiggestWeakness: screenshotResult.biggest_weakness,
        aiNotableStrength: screenshotResult.notable_strength,
        screenshotExpiresAt,
      } : {}),
      
      // Scores
      quickScore: scoreResult.quickScore,
      quickScoreBreakdown: scoreResult.breakdown,
      functionalScore: scoreResult.functionalScore,
      visualScore: scoreResult.visualScore,
      scoreConfidence: scoreResult.confidence,
      scoreVersion: 'v1.0',
      
      // Business signals
      ...businessSignals,
      
      // Priority & Outreach
      ...priorityResult,
      weaknesses: scoreResult.weaknesses,
      weaknessCount: scoreResult.weaknessCount,
      
      quickCheckDate: new Date(),
    }
  });
  
  // Emit events
  await emitEvent('lead.qualified', {
    leadId: lead.id,
    firmName: lead.firmName,
    city: lead.city,
    quickScore: scoreResult.quickScore,
    priorityTag: priorityResult.priorityTag,
    topWeaknesses: scoreResult.weaknesses.slice(0, 3),
  });
  
  // Gold leads get instant notification
  if (priorityResult.priorityTag === 'GOLD') {
    await emitEvent('lead.gold_detected', {
      leadId: lead.id,
      firmName: lead.firmName,
      quickScore: scoreResult.quickScore,
      googleRating: lead.googleRating,
      googleReviewCount: lead.googleReviewCount,
      goldReason: priorityResult.goldReason,
    });
  }
}
```

---

## 9. COMPLETE EXAMPLE WALKTHROUGH

A single lead from Google API response to qualified lead in DB:

```
INPUT (from Google Text Search "Coach München"):
{
  id: "ChIJ_abc123",
  displayName: { text: "Coaching Praxis Sandra K." },
  formattedAddress: "Leopoldstr. 42, 80802 München",
  location: { latitude: 48.1574, longitude: 11.5849 },
  rating: 4.5,
  userRatingCount: 23,
  websiteUri: "https://www.coaching-sandra.jimdofree.de",
  nationalPhoneNumber: "+49 89 12345678",
  primaryType: "life_coach",
  priceLevel: null,
  photos: [{ name: "places/ChIJ_abc123/photos/photo1" }, ...], // 4 photos
  editorialSummary: { text: "Systemisches Coaching für Führungskräfte in München" },
  googleMapsUri: "https://maps.google.com/?cid=12345",
}

PHASE A (Immediate Filters):
✅ Has website URL → proceed
✅ Not permanently closed → proceed
✅ Category "life_coach" is in acceptedCategories → proceed  
✅ Not a chain → proceed
✅ Not in DB yet (new googlePlaceId) → proceed
✅ Not on blacklist → proceed
→ Created as RAW lead in DB

PHASE B (HTTP Quick-Check):
  HTTP GET https://www.coaching-sandra.jimdofree.de
  → Redirect: https://www.coaching-sandra.jimdofree.de → same (no redirect)
  
  Headers: server: nginx, no security headers
  
  HTML Parse Results:
    hasSsl: true (HTTPS)
    sslStatus: VALID
    hasViewportMeta: true (Jimdo adds this by default)
    titleTag: "Coaching Praxis Sandra K."
    metaDescription: null (!)
    hasH1: true
    cmsType: "jimdo"
    cmsVersion: null
    copyrightYear: 2019
    hasTelLink: false (!)
    hasContactForm: false (!)
    hasBookingWidget: false (!)
    hasEcommerce: false
    websiteLanguage: "de"
    isSpa: false
    isParked: false
    isUnderConstruction: false
    homepageWordCount: 180
    baukastenPlatform: "jimdo"
    websitePlatformTier: FREE_BAUKASTEN (.jimdofree.de domain)
    primaryFont: null
    hasBlog: false
    hasGoogleAnalytics: false
    hasFacebookPixel: false
    hasCookieConsent: false
    emailFoundOnWebsite: "sandra@coaching-sandra.de"
    socialLinks: { instagram: null, facebook: null, linkedin: "https://linkedin.com/in/sandrak" }
    websiteBuiltBy: null
    domainType: BAUKASTEN_SUBDOMAIN

  Lighthouse Quick-Audit:
    performance: 42
    seo: 58
    accessibility: 71

  Quick-Score Calculation (Industry: Coaches):
    FUNCTIONAL:
      no_cta_or_booking: 18   (no form, no tel, no booking widget)
      no_ssl: 0               (has SSL)
      not_mobile: 0           (has viewport)
      no_contact_form: 6      (no form)
      no_tel_link: 4          (no tel: link)
      slow_loading: 6         (performance 42 < 50)
      no_seo: 2               (no meta description)
    Functional total: 36

    VISUAL:
      design_outdated: 12     (copyright 2019, Jimdo)
      baukasten_visible: 8    (FREE_BAUKASTEN)
      copyright_old: 3        (2019, 7 years old)
      generic_text: 0         (not enough banned phrases matched)
      no_real_photos: 0       (can't determine without AI Vision)
    Visual total: 23

    BONUS: +5 (rating 4.5 + 23 reviews), +3 (free baukasten), +2 (LinkedIn present → online-affin)
    MALUS: 0
    
    Raw: 36 + 23 + 10 = 69
    Normalized: (69 / 109) × 100 = 63
    
    quick_score = 63 → QUALIFIED (≥40) + Score ≥60 → no screenshot needed!
    confidence = HIGH (9 checks completed)

PHASE C (Enrich):
  Google Place Details → 5 review texts extracted
  
  Business Signals:
    estimatedSize: SMALL (23 reviews)
    estimatedRevenue: MEDIUM (no price level data)
    growthSignal: STABLE (reviews spread over time)
    emotionalTiming: NEUTRAL
    onlineActivityScore: 0.45 (4 photos + LinkedIn + description)
    googleProfileCompleteness: 0.70
    improvementPotential: MEDIUM (180 words, 4 Google photos, LinkedIn, but no Instagram, no blog)
  
  Priority: SILVER (score 63, rating 4.5, 23 reviews – misses GOLD because score < 70)
  
  Reachability: 0.75 (email found 0.6, LinkedIn 0.15)
  
  Weakness Arguments:
    1. "Wo auf Ihrer Website kann jemand ein Kennenlerngespräch buchen? Genau – nirgends."
    2. "Sie helfen Menschen sich weiterzuentwickeln – Ihre Website hat sich seit 2019 nicht weiterentwickelt."
    3. "Ihre Klienten zahlen €150+/Stunde für Ihre Expertise. Ihre Website sagt: Baukasten."
  
  Strengths: ["excellent_reviews"]

OUTPUT (Lead in DB):
{
  id: "uuid-123",
  status: "QUALIFIED",
  firmName: "Coaching Praxis Sandra K.",
  city: "München",
  quickScore: 63,
  priorityTag: "SILVER",
  weaknesses: ["no_cta_or_booking", "no_contact_form", "no_tel_link", "slow_loading", 
               "no_seo", "design_outdated", "baukasten_visible", "copyright_old"],
  weaknessCount: 8,
  weaknessLabels: ["Kein Buchungsweg", "Kein Kontaktformular", "Telefon nicht klickbar",
                    "Ladezeit zu hoch", "Kein SEO", "Design veraltet", "Baukasten erkennbar", 
                    "Copyright veraltet"],
  weaknessArguments: ["Wo auf Ihrer Website kann jemand ein Kennenlerngespräch buchen?...", ...],
  strengths: ["excellent_reviews"],
  googleRating: 4.5,
  googleReviewCount: 23,
  emailFoundOnWebsite: "sandra@coaching-sandra.de",
  // ... all other fields populated
}

→ Lead appears in Gate 1 for operator review
→ event: lead.qualified emitted
```

---

## 10. CAMPAIGN MANAGEMENT UX

### Campaign Creation Screen

```
┌────────────────────────────────────────────────────────────────┐
│  NEUE KAMPAGNE ERSTELLEN                                       │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  Industrie:     [Coaches & Berater        ▼]                  │
│  Städte:        [München] [Augsburg] [+ hinzufügen]           │
│  Modus:         ● Full  ○ Preview (5 Leads)  ○ Dry Run       │
│                                                                │
│  ── MARKT-INTELLIGENCE ─────────────────────────────────────  │
│                                                                │
│  München, Coaches:                                             │
│    Geschätzt auf Google:         ~280                          │
│    Bereits gescraped:            180                           │
│    Verbleibend:                  ~100                          │
│    Erwartet qualifiziert:        ~35 (35%)                    │
│    Saisonaler Hinweis:  ✅ Januar – Gute Zeit                 │
│    Spam-Risiko:         🟢 Niedrig                            │
│                                                                │
│  Augsburg, Coaches:                                            │
│    Geschätzt auf Google:         ~45                           │
│    Bereits gescraped:            0                             │
│    Verbleibend:                  ~45                           │
│    Erwartet qualifiziert:        ~16 (35%)                    │
│                                                                │
│  ── KOSTEN-VORSCHAU ────────────────────────────────────────  │
│                                                                │
│    Google API:         ~€2.80                                  │
│    Lighthouse:         ~€0.35                                  │
│    AI Vision:          ~€0.15                                  │
│    Place Details:      ~€0.85                                  │
│    ───────────────────────────                                 │
│    TOTAL geschätzt:    ~€4.15                                  │
│    Budget-Limit:       [€ 15.00       ]                       │
│                                                                │
│  ── ZEITSCHÄTZUNG ──────────────────────────────────────────  │
│    Geschätzte Dauer:   ~20 Minuten                            │
│                                                                │
│              [Kampagne starten]      [Abbrechen]              │
└────────────────────────────────────────────────────────────────┘
```

### Live Progress Screen

```
┌────────────────────────────────────────────────────────────────┐
│  KAMPAGNE #47: Coaches München + Augsburg                      │
│  Status: ⏳ LÄUFT    ████████████████░░░░░░░░ 68%              │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  Phase A  Scraping         ✅ 4/4 Search-Calls (82 Results)   │
│  Phase B  Quality-Check    ⏳ 48/65 geprüft                   │
│  Phase C  Enrichment       ⏳ 18/24 enriched                  │
│                                                                │
│  Live: 24 qualifiziert | 26 gefiltert | 17 ausstehend         │
│                                                                │
│  ── LIVE-FEED ──────────────────────────────────────────────  │
│                                                                │
│  14:23  🥇 Coaching Sandra K. – Score 78 (GOLD!)              │
│         4.5★ (23 Rev) | Jimdo 2019, kein CTA                  │
│  14:23  ✗  Fahrcoach München – Falsche Kategorie              │
│  14:22  🥈 BusinessCoach Peters – Score 55 (SILVER)            │
│         4.1★ (8 Rev) | Wix, nicht mobil                       │
│  14:22  ✗  Life-Balance Studio – Score 28 → OK genug          │
│  14:21  🥈 Karriere-Coaching Huber – Score 61 (SILVER)         │
│         4.8★ (41 Rev) | WordPress 2020, kein Booking          │
│                                                                │
│  Kosten bisher: €2.87 / €15.00                                │
│  Dauer: 14 Min                                                 │
│                                                                │
│              [Pausieren]   [Abbrechen (Leads behalten)]       │
└────────────────────────────────────────────────────────────────┘
```

### Campaign Summary Screen

```
┌────────────────────────────────────────────────────────────────┐
│  KAMPAGNE #47 ABGESCHLOSSEN                                   │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  Gescraped:      82                                            │
│  Qualifiziert:   29 (35%)                                     │
│  Gefiltert:      53 (65%)                                     │
│    Website OK:   22 | Duplikat: 8 | Falsche Kategorie: 5     │
│    Keine Website: 4 | Parked: 2 | Sonstige: 12               │
│                                                                │
│  Kosten: €4.23              Dauer: 22 Min                     │
│  Kosten/qualifiziertem Lead: €0.146                           │
│                                                                │
│  Prioritäts-Verteilung:                                       │
│    🥇 GOLD:      3 Leads   (Score ≥70 + gutes Business)      │
│    🥈 SILVER:   14 Leads   (Score ≥50)                        │
│    ⬜ STANDARD: 12 Leads   (Score 40-49)                      │
│                                                                │
│  Top-Schwächen:                                               │
│    1. Kein CTA/Buchungsweg      24/29 (83%)                   │
│    2. Design veraltet           19/29 (66%)                   │
│    3. Nicht mobiloptimiert      15/29 (52%)                   │
│                                                                │
│  → 29 Leads warten in Gate 1  [Zu Gate 1 →]                  │
└────────────────────────────────────────────────────────────────┘
```

---

## 11. ERROR HANDLING

| Error | Handling | Retry | Lead Status |
|-------|----------|-------|-------------|
| Google API 429 (Rate Limit) | Pause 60s | 3× exp. backoff | - |
| Google API 403 (Key invalid) | STOP campaign, alert | 0 | - |
| Google API 0 results | Alert "No results" | 0 | - |
| Website timeout >5s | Mark lead | 1× after 10s | UNREACHABLE |
| Website DNS error | Mark lead | 0 | UNREACHABLE |
| Website redirect loop >10 | Mark lead | 0 | UNREACHABLE |
| Website Cloudflare challenge | Wait 5s, retry | 1× | UNREACHABLE if still blocked |
| Website 401/403 | Mark lead | 0 | AUTH_REQUIRED |
| Website returns PDF | Mark lead | 0 | QUALIFIED (score 85+) |
| Lighthouse API error | Skip LH, reduce confidence | 1× | Continue without LH |
| Haiku Vision timeout | Skip AI, reduce confidence | 1× | Continue without AI |
| DB connection error | Buffer in memory, retry | 5× exp. backoff | - |
| R2 upload error | Retry upload | 3× | Continue without screenshot |
| Campaign crash | Auto-resume from progressLog | Automatic | - |
| Budget exceeded | Pause campaign, alert | 0 | - |

### Cost Circuit Breaker

```typescript
async function checkCostLimits(campaignId: string): Promise<boolean> {
  const campaign = await prisma.campaign.findUnique({ where: { id: campaignId } });
  if (!campaign) return false;
  
  // Campaign budget
  if (campaign.budgetCents && campaign.spentCents >= campaign.budgetCents) {
    await emitEvent('campaign.budget_exceeded', { campaignId, spent: campaign.spentCents, budget: campaign.budgetCents });
    return false; // STOP
  }
  
  // Daily global limit (€50 = 5000 cents)
  const today = new Date(); today.setHours(0,0,0,0);
  const dailySpent = await prisma.apiCostLog.aggregate({
    _sum: { costCents: true },
    where: { createdAt: { gte: today } },
  });
  if ((dailySpent._sum.costCents ?? 0) > 5000) {
    await emitEvent('system.daily_budget_exceeded', { spent: dailySpent._sum.costCents });
    return false; // STOP ALL CAMPAIGNS
  }
  
  // Anomaly: qualification rate too high
  if (campaign.totalRawLeads > 20) {
    const qualRate = campaign.totalQualified / campaign.totalRawLeads;
    if (qualRate > 0.8) {
      await emitEvent('system.anomaly_detected', { 
        campaignId, type: 'high_qualification_rate', rate: qualRate 
      });
      // Don't stop, just alert
    }
  }
  
  return true; // CONTINUE
}
```

---

## 12. EVENTS EMITTED BY STEP 1

```typescript
// Campaign lifecycle
'campaign.started'           { campaignId, industry, cities[], estimatedLeads, estimatedCost }
'campaign.progress'          { campaignId, phase, progressPct, qualifiedCount, filteredCount, costCents }
'campaign.completed'         { campaignId, totalScraped, totalQualified, totalFiltered, totalCostCents, durationMin }
'campaign.paused'            { campaignId, reason }
'campaign.resumed'           { campaignId }
'campaign.error'             { campaignId, error, phase, canResume: boolean }
'campaign.budget_exceeded'   { campaignId, spentCents, budgetCents }

// Lead lifecycle  
'lead.scraped'               { leadId, campaignId, firmName }
'lead.qualified'             { leadId, firmName, city, quickScore, priorityTag, topWeaknesses[] }
'lead.filtered'              { leadId, reason, quickScore }
'lead.gold_detected'         { leadId, firmName, quickScore, googleRating, reviewCount, goldReason }

// System
'system.daily_budget_exceeded' { spentCents }
'system.anomaly_detected'      { campaignId, type, details }
```

---

## 13. HANDOFF: STEP 1 → GATE 1 → STEP 2

### Gate 1 Query

```sql
-- Gate 1 shows all qualified leads for a campaign, sorted by priority
SELECT * FROM leads
WHERE campaign_id = $1
  AND status = 'QUALIFIED'
ORDER BY 
  CASE priority_tag 
    WHEN 'GOLD' THEN 1 
    WHEN 'SILVER' THEN 2 
    ELSE 3 
  END,
  quick_score DESC;
```

### Gate 1 Operator Actions

**Approve:** `UPDATE leads SET status = 'GATE1_APPROVED' WHERE id = $1`
→ Emit `lead.gate1_approved` → Step 2 (Enrichment) picks up the lead

**Reject:** `UPDATE leads SET status = 'GATE1_REJECTED' WHERE id = $1`
→ Lead is archived. NOT blacklisted (could be re-evaluated later).

**Bulk approve:** All leads with score ≥ X → GATE1_APPROVED
**Bulk reject:** All leads with score < X → GATE1_REJECTED

### What Step 2 Receives

Step 2 (Enrichment) receives a lead with status GATE1_APPROVED. It already has:
- Google data (name, address, phone, rating, reviews, photos, description)
- Website URL (actual, after redirects)
- Technical signals (SSL, CMS, viewport, copyright year, etc.)
- Quick Score with breakdown
- Email found on website (if any)
- Social links + Instagram handle

Step 2 does NOT need to re-check: URL reachability (done in Step 1), duplicates (done in Step 1), or basic website signals (done in Step 1).

Step 2 ADDS: Detailed enrichment from additional sources (LinkedIn, portal profiles), decision-maker identification, email verification, full business profile.

---

## 14. TESTING STRATEGY

### Unit Tests

```typescript
describe('normalizeDomain', () => {
  it('removes www and protocol', () => {
    expect(normalizeDomain('https://www.example.de/page')).toBe('example.de');
  });
  it('handles baukasten subdomains', () => {
    expect(normalizeDomain('https://salon.jimdosite.com')).toBe('salon.jimdosite.com');
  });
});

describe('calculateQuickScore', () => {
  it('scores a terrible website high', () => {
    const result = calculateQuickScore(mockLead, terribleWebsite, badLighthouse, coachProfile);
    expect(result.quickScore).toBeGreaterThanOrEqual(60);
  });
  it('scores a good website low', () => {
    const result = calculateQuickScore(mockLead, goodWebsite, goodLighthouse, coachProfile);
    expect(result.quickScore).toBeLessThan(30);
  });
});

describe('parkingDetection', () => {
  it('detects IONOS parking page', () => { /* ... */ });
  it('detects Strato parking page', () => { /* ... */ });
  it('does not false-positive on real websites', () => { /* ... */ });
});

describe('baukastenDetection', () => {
  it('detects Jimdo free', () => { /* ... */ });
  it('detects Wix', () => { /* ... */ });
  it('distinguishes free from paid tier', () => { /* ... */ });
});
```

### Integration Tests

```typescript
describe('Phase B: HTTP Quick-Check', () => {
  // Use a set of 10 REAL URLs (test websites we control) to verify:
  it('correctly identifies a Jimdo website', async () => { /* ... */ });
  it('correctly follows redirects', async () => { /* ... */ });
  it('handles timeout gracefully', async () => { /* ... */ });
  it('detects parking pages', async () => { /* ... */ });
});
```

### E2E Calibration Test

```typescript
// Run monthly: 30 known websites through the entire pipeline
// Compare Quick Scores with manually assigned scores
// Alert if correlation drops below 0.6
describe('Calibration', () => {
  const calibrationSet = loadCalibrationSet('coaches'); // 30 manually rated websites
  
  for (const site of calibrationSet) {
    it(`scores ${site.url} within ±15 of manual rating`, async () => {
      const result = await runPipeline(site.url, coachProfile);
      expect(Math.abs(result.quickScore - site.manualScore)).toBeLessThanOrEqual(15);
    });
  }
});
```

---

## 15. LOGGING

```typescript
import pino from 'pino';

const logger = pino({
  level: process.env.LOG_LEVEL ?? 'info',
  transport: process.env.NODE_ENV === 'development' 
    ? { target: 'pino-pretty' } 
    : undefined,
});

// What we log:
// INFO:  campaign started/completed, lead qualified/filtered, phase transitions
// WARN:  API rate limits hit, lighthouse failures, AI parse errors, anomalies
// ERROR: campaign crashes, DB connection failures, unhandled exceptions
// DEBUG: individual HTTP check results, score calculations, API responses

// Every API call:
logger.info({ api: 'google_text_search', costCents: 3.2, durationMs: 450, resultCount: 20 }, 'API call');

// Every lead status change:
logger.info({ leadId, status: 'QUALIFIED', quickScore: 63, priority: 'SILVER' }, 'Lead qualified');

// Every error:
logger.error({ leadId, error: err.message, phase: 'http_check', url }, 'Website check failed');
```

---

## 16. DSGVO & DATA PROTECTION

### Data Categories

| Data | Personal? | Legal Basis | Retention |
|------|-----------|-------------|-----------|
| Business name (firm) | No | N/A | Unlimited |
| Business name = person name | Yes | Art. 6(1)(f) legitimate interest | 12 months if not converted |
| Business address | No | N/A | Unlimited |
| Business phone | Gray area | Art. 6(1)(f) | 12 months |
| Email (info@) | No | N/A | Unlimited |
| Email (personal) | Yes | Art. 6(1)(f) | 12 months |
| Website screenshots | May show persons | Art. 6(1)(f) | Per retention policy |
| Google reviews (public) | No | N/A | Unlimited |

### Deletion Process

```typescript
async function deleteLeadCompletely(leadId: string): Promise<void> {
  const lead = await prisma.lead.findUnique({ where: { id: leadId } });
  if (!lead) throw new Error('Lead not found');
  
  // 1. Delete screenshots from R2
  if (lead.screenshotDesktopUrl) await deleteFromR2(lead.screenshotDesktopUrl);
  if (lead.screenshotMobileUrl) await deleteFromR2(lead.screenshotMobileUrl);
  
  // 2. Delete from all pipeline tables (cascade)
  // TODO: Add cascade deletes for Steps 2-17 tables when they exist
  
  // 3. Delete lead record
  await prisma.lead.delete({ where: { id: leadId } });
  
  // 4. Add to blacklist (only non-personal identifiers)
  await prisma.blacklist.create({
    data: {
      googlePlaceId: lead.googlePlaceId,
      domain: lead.actualUrl ? normalizeDomain(lead.actualUrl) : null,
      reason: 'user_requested_deletion',
    }
  });
  
  logger.info({ leadId, googlePlaceId: lead.googlePlaceId }, 'Lead deleted and blacklisted');
}
```

### Screenshot Retention Cleanup (daily cron)

```typescript
async function cleanupExpiredScreenshots(): Promise<void> {
  const expiredLeads = await prisma.lead.findMany({
    where: {
      screenshotExpiresAt: { lt: new Date() },
      screenshotDesktopUrl: { not: null },
    },
    select: { id: true, screenshotDesktopUrl: true, screenshotMobileUrl: true },
  });
  
  for (const lead of expiredLeads) {
    if (lead.screenshotDesktopUrl) await deleteFromR2(lead.screenshotDesktopUrl);
    if (lead.screenshotMobileUrl) await deleteFromR2(lead.screenshotMobileUrl);
    await prisma.lead.update({
      where: { id: lead.id },
      data: { screenshotDesktopUrl: null, screenshotMobileUrl: null, screenshotExpiresAt: null },
    });
  }
  
  logger.info({ count: expiredLeads.length }, 'Expired screenshots cleaned up');
}
```

### Screenshot Retention Rules

| Lead Status | Retention |
|-------------|-----------|
| Became customer | PERMANENT |
| In active pipeline | 6 months |
| Qualified but not converted | 90 days |
| Filtered (had screenshot) | 30 days |

---

## 17. CALIBRATION MODE (first 8 weeks)

During the learning phase, Step 1 operates in calibration mode:

1. ALL leads are saved (even score < 25) with status FILTERED_OUT + reason
2. Per campaign: 20 random leads are flagged for manual review
3. Operator rates each: "Is this website bad enough? (Yes/No) + severity 1-10"
4. System compares: Quick Score vs. manual rating
5. Weekly report: correlation, false-positive rate, false-negative rate
6. If correlation < 0.6: adjust weights, redeploy score formula

After learning phase: calibration on monthly 50-lead sample.

```typescript
async function flagForCalibration(campaignId: string): Promise<void> {
  const allLeads = await prisma.lead.findMany({
    where: { campaignId },
    select: { id: true },
  });
  
  // Select 20 random leads
  const shuffled = allLeads.sort(() => Math.random() - 0.5);
  const sample = shuffled.slice(0, 20);
  
  for (const lead of sample) {
    await prisma.lead.update({
      where: { id: lead.id },
      data: { needsCalibrationReview: true }, // add this field to schema
    });
  }
}
```

---

## 18. INDUSTRY PROFILE SCHEMA (complete, for Step 1)

This is the FULL JSON structure that Step 1 reads. One profile per industry. Stored in DB or as JSON file.

```json
{
  "tag": "coaches",
  "name": "Coaches & Berater",
  
  "scraping": {
    "keywords": ["Coach", "Coaching", "Business Coach", "Life Coach",
                 "Karrierecoach", "Führungskräftecoaching", "Systemischer Coach",
                 "Berater", "Beratung persönliche Entwicklung", "Supervision",
                 "Mentoring", "NLP Trainer", "Mental Coach", "Elterncoaching"],
    "acceptedCategories": ["life_coach", "business_consultant", "career_counselor", 
                           "consultant", "corporate_office"],
    "excludedCategories": ["driving_school", "gym", "fitness_trainer", 
                           "physical_therapist", "school", "university"],
    "excludeChains": [],
    "geoRadiusKm": 50,
    "noWebsiteAction": "skip",
    "additionalSources": []
  },

  "qualityCheck": {
    "functionalWeights": {
      "no_cta_or_booking": 18,
      "not_mobile": 12,
      "no_contact_form": 6,
      "no_tel_link": 4,
      "slow_loading": 6,
      "no_ssl": 7,
      "no_seo": 2
    },
    "visualWeights": {
      "no_real_photos": 15,
      "generic_text": 12,
      "design_outdated": 12,
      "no_social_proof": 10,
      "baukasten_visible": 8,
      "no_about": 4,
      "copyright_old": 3
    },
    "maxRawScore": 109,
    "qualifyThreshold": 40,
    "priorityThreshold": 60,
    "goldThreshold": 70,
    "bannedPhrases": ["ganzheitlich", "Potenzialentfaltung", "auf Augenhöhe",
                       "Raum für Veränderung", "kompetenter Partner",
                       "individuelle Begleitung", "nachhaltige Veränderung",
                       "ganzheitliche Betrachtung", "Ihr Weg zu mehr"]
  },

  "scoring": {
    "goldCriteria": { "minQuickScore": 70, "minGoogleRating": 4.3, "minReviewCount": 15 },
    "reachabilityWeights": { "email": 0.60, "contactForm": 0.20, "linkedin": 0.15, "phone": 0.05 }
  },

  "outreach": {
    "primaryChannel": "email",
    "addressForm": "Sie",
    "spamSaturationRisk": "low",
    "seasonalNotes": {
      "1": "✅ Gute Zeit (Neujahrsvorsätze)",
      "3": "✅ Gute Zeit (Frühlingsneustart)",
      "9": "✅ Gute Zeit (Back-to-Business)",
      "12": "⚠️ Mittelmäßig (Jahresende)"
    },
    "weaknessToArgument": {
      "no_cta_or_booking": "Wo auf Ihrer Website kann jemand ein {{cta_action}} {{cta_verb}}? Genau – nirgends. Jeder Interessent der nicht sofort handeln kann, ist verloren.",
      "no_real_photos": "Klienten wollen wissen mit WEM sie arbeiten. Ihre Website zeigt kein einziges Foto von Ihnen.",
      "generic_text": "Lesen Sie Ihren Website-Text – und dann den von 5 anderen Coaches. Austauschbar? Ihre Klienten suchen IHRE Stimme, nicht eine austauschbare Vorlage.",
      "design_outdated": "Sie helfen Menschen sich weiterzuentwickeln – aber Ihre Website hat sich seit {{copyright_year}} nicht weiterentwickelt. Das ist ein Widerspruch den jeder Besucher spürt.",
      "not_mobile": "Öffnen Sie Ihre Website jetzt auf Ihrem Handy. So sehen 70% Ihrer potenziellen Klienten Sie. Würden Sie sich selbst buchen?",
      "no_social_proof": "Sie haben {{google_rating}} Sterne bei {{review_count}} Google-Bewertungen. Fantastisch. Aber Ihre Website zeigt das nicht. Neue Klienten sehen: null Beweis.",
      "baukasten_visible": "Ihre Klienten zahlen €150+/Stunde für Ihre Expertise. Ihre Website sagt: Baukasten für €0/Monat. Passt das zu Ihrem Anspruch?",
      "no_ssl": "Wenn Besucher Ihre Website öffnen, zeigt der Browser 'Nicht sicher.' Das ist das Erste was sie sehen – noch vor Ihrem Namen.",
      "no_contact_form": "Nicht jeder greift zum Telefon. Viele Klienten bevorzugen ein kurzes Kontaktformular. Ihre Website hat keins.",
      "slow_loading": "Ihre Website braucht über 5 Sekunden zum Laden. Nach 3 Sekunden ist die Hälfte der Besucher weg.",
      "no_seo": "Wenn jemand 'Coach {{city}}' googelt, zeigt Google bei Ihnen generischen Text. Bei der Konkurrenz: einen einladenden, spezifischen Eintrag.",
      "copyright_old": "© {{copyright_year}}. Ihre Website signalisiert: hier passiert seit Jahren nichts Neues."
    },
    "ctaAction": "Kennenlerngespräch",
    "ctaVerb": "buchen"
  },

  "display": {
    "screenshotPriority": "desktop"
  }
}
```
