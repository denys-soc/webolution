# Webolution Database Schema (PostgreSQL + Prisma)

## Complete Prisma Schema

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// ═══════════════════════════════════════
// CORE: Lead Lifecycle
// ═══════════════════════════════════════

model Lead {
  id                String   @id @default(uuid()) @db.Uuid
  state             String   @default("scraped") // scraped, validating, validated, enriching, enriched, crawling, crawled, reviewing, scoring, scored, planning, polishing, building, qa_testing, qa_passed, deploying, deployed, outreach_prepping, outreach_active, converted, onboarding, live, customer_success, nurture_pool, archived, suppressed, manual_review, retry_pending, permanently_failed
  industryTag       String   @map("industry_tag")
  region            String?

  // Basic Info (from Scraping)
  firmName          String   @map("firm_name")
  canonicalUrl      String?  @unique @map("canonical_url")
  phone             String?
  emailGeneric      String?  @map("email_generic")
  address           Json?    // {street, zip, city, lat, lng}
  googlePlaceId     String?  @map("google_place_id")
  googleRating      Decimal? @db.Decimal(2, 1) @map("google_rating")
  googleReviews     Int?     @map("google_reviews")
  sourceTags        String[] @map("source_tags")

  // Enrichment Data (from Step 3)
  decisionMaker     Json?    @map("decision_maker") // {name, role, email, phone, linkedinUrl, confidence}
  businessHealthScore Int?   @map("business_health_score")
  socialProfiles    Json?    @map("social_profiles") // {instagram?, facebook?, linkedin?}
  industryData      Json?    @map("industry_data") // Industry-specific enrichment

  // Scoring (from Step 7)
  finalScore        Int?     @map("final_score")
  scoreComponents   Json?    @map("score_components") // {websiteWeakness, contentOpp, businessHealth, competitivePressure, reachability, dealPotential}
  scoreCategory     String?  @map("score_category") // hot, warm, cool, cold
  scoreConfidence   Decimal? @db.Decimal(3, 2) @map("score_confidence")
  scoredAt          DateTime? @map("scored_at")

  // Confidence Scores (per step)
  validationConfidence  Decimal? @db.Decimal(3, 2) @map("validation_confidence")
  enrichmentConfidence  Decimal? @db.Decimal(3, 2) @map("enrichment_confidence")
  extractionConfidence  Decimal? @db.Decimal(3, 2) @map("extraction_confidence")
  reviewConfidence      Decimal? @db.Decimal(3, 2) @map("review_confidence")
  rebuildConfidence     Decimal? @db.Decimal(3, 2) @map("rebuild_confidence")
  polishConfidence      Decimal? @db.Decimal(3, 2) @map("polish_confidence")

  // Delivery
  previewUrl        String?  @map("preview_url")
  previewDeployedAt DateTime? @map("preview_deployed_at")
  outreachStatus    String?  @map("outreach_status")
  stripeCustomerId  String?  @map("stripe_customer_id")
  customerPackage   String?  @map("customer_package")
  goLiveAt          DateTime? @map("go_live_at")

  // Meta
  createdAt         DateTime @default(now()) @map("created_at")
  updatedAt         DateTime @updatedAt @map("updated_at")

  // Relations
  contentPackage    ContentPackage?
  qualityReport     QualityReport?
  competitorReport  CompetitorReport?
  rebuildPlan       RebuildPlan?
  polishedContent   PolishedContent?
  builds            Build[]
  qaReports         QaReport[]
  outreachPackage   OutreachPackage?
  outreachEvents    OutreachEvent[]
  previewEvents     PreviewEvent[]
  customer          Customer?
  auditLogs         AuditLog[]

  @@index([state])
  @@index([industryTag])
  @@index([finalScore(sort: Desc)])
  @@index([region])
  @@map("leads")
}

// ═══════════════════════════════════════
// PIPELINE DATA (one per lead)
// ═══════════════════════════════════════

model ContentPackage {
  id                  String   @id @default(uuid()) @db.Uuid
  leadId              String   @unique @map("lead_id") @db.Uuid
  lead                Lead     @relation(fields: [leadId], references: [id])

  pages               Json     // [{url, title, text, htmlHash, screenshotS3Key}]
  pageCount           Int      @map("page_count")
  crawlDurationMs     Int      @map("crawl_duration_ms")
  firmProfile         Json     @map("firm_profile") // {name, slogan, services[], usps[], team[], foundingYear, certifications[], contact{}, openingHours{}, testimonials[], prices{}, socialLinks{}, blogPosts[], industrySpecific{}}
  brandAssets         Json     @map("brand_assets") // {logoS3Key, colors{primary, secondary, accent, text, bg}, fonts{heading, body}, imageStyle}
  images              Json     // [{s3Key, alt, width, height, type, qualityScore}]
  extractionConfidence Decimal @db.Decimal(3, 2) @map("extraction_confidence")
  contentCompleteness Decimal  @db.Decimal(3, 2) @map("content_completeness")

  createdAt           DateTime @default(now()) @map("created_at")
  @@map("content_packages")
}

model QualityReport {
  id                      String   @id @default(uuid()) @db.Uuid
  leadId                  String   @unique @map("lead_id") @db.Uuid
  lead                    Lead     @relation(fields: [leadId], references: [id])

  technicalScore          Int      @map("technical_score")
  visualScore             Int      @map("visual_score")
  contentScore            Int      @map("content_score")
  industryComplianceScore Int      @map("industry_compliance_score")
  overallScore            Int      @map("overall_score")
  weaknesses              Json     // [{type, severity, class (A/B/C/D), description, evidence, outreachHook}]
  reviewConfidence        Decimal  @db.Decimal(3, 2) @map("review_confidence")

  createdAt               DateTime @default(now()) @map("created_at")
  @@map("quality_reports")
}

model CompetitorReport {
  id                       String   @id @default(uuid()) @db.Uuid
  leadId                   String   @unique @map("lead_id") @db.Uuid
  lead                     Lead     @relation(fields: [leadId], references: [id])

  competitors              Json     // [{name, url, scores{}, features[], designEpoch, googleRating, googleReviews}]
  comparisonMatrix         Json     @map("comparison_matrix")
  relativePosition         Int      @map("relative_position") // 1-5
  competitivePressureScore Int      @map("competitive_pressure_score")
  topDifferentiators       Json     @map("top_differentiators") // [{feature, competitorHas, leadMissing, outreachArg}]

  createdAt                DateTime @default(now()) @map("created_at")
  @@map("competitor_reports")
}

model RebuildPlan {
  id                String   @id @default(uuid()) @db.Uuid
  leadId            String   @unique @map("lead_id") @db.Uuid
  lead              Lead     @relation(fields: [leadId], references: [id])

  templateId        String   @map("template_id")
  templateVariant   String?  @map("template_variant")
  pageStructure     Json     @map("page_structure") // [{page, sections[{id, source, action, contentRef}]}]
  brandDecision     String   @map("brand_decision") // keep, upgrade, new
  brandConfig       Json     @map("brand_config") // {colors{}, fonts{}, logoStrategy}
  seoPlan           Json     @map("seo_plan") // {primaryKw[], secondaryKw[], localSeo{}}
  ctaStrategy       Json     @map("cta_strategy") // {primaryText, secondaryText, placement[]}
  legalChecklist    Json     @map("legal_checklist") // [{requirement, status, source}]
  imageStrategy     Json     @map("image_strategy") // {perSection[{section, source, action}]}
  rebuildConfidence Decimal  @db.Decimal(3, 2) @map("rebuild_confidence")
  approved          Boolean  @default(false)
  approvedBy        String?  @map("approved_by") // 'auto' or reviewer name

  createdAt         DateTime @default(now()) @map("created_at")
  polishedContent   PolishedContent?
  @@map("rebuild_plans")
}

model PolishedContent {
  id              String   @id @default(uuid()) @db.Uuid
  leadId          String   @unique @map("lead_id") @db.Uuid
  lead            Lead     @relation(fields: [leadId], references: [id])
  rebuildPlanId   String   @unique @map("rebuild_plan_id") @db.Uuid
  rebuildPlan     RebuildPlan @relation(fields: [rebuildPlanId], references: [id])

  texts           Json     // {perSection: {sectionId: {text, sourceRefs[], confidence}}}
  images          Json     // {perSection: {sectionId: {s3Key, source, isPlaceholder}}}
  metaTags        Json     @map("meta_tags") // {title, description, ogTitle, ogDescription, ogImage}
  schemaMarkup    Json     @map("schema_markup")
  factCheckReport Json     @map("fact_check_report") // {checked[], confirmed[], corrected[], removed[]}
  bannedPhraseHits Int     @map("banned_phrase_hits")
  polishConfidence Decimal @db.Decimal(3, 2) @map("polish_confidence")

  createdAt       DateTime @default(now()) @map("created_at")
  @@map("polished_content")
}

model Build {
  id                     String   @id @default(uuid()) @db.Uuid
  leadId                 String   @map("lead_id") @db.Uuid
  lead                   Lead     @relation(fields: [leadId], references: [id])

  lighthousePerformance  Int?     @map("lighthouse_performance")
  lighthouseAccessibility Int?    @map("lighthouse_accessibility")
  lighthouseSeo          Int?     @map("lighthouse_seo")
  pageWeightKb           Int?     @map("page_weight_kb")
  buildDurationMs        Int?     @map("build_duration_ms")
  buildLog               String?  @map("build_log")
  artifactS3Key          String?  @map("artifact_s3_key")
  success                Boolean  @default(false)

  createdAt              DateTime @default(now()) @map("created_at")
  qaReports              QaReport[]
  @@map("builds")
}

model QaReport {
  id           String   @id @default(uuid()) @db.Uuid
  leadId       String   @map("lead_id") @db.Uuid
  lead         Lead     @relation(fields: [leadId], references: [id])
  buildId      String   @map("build_id") @db.Uuid
  build        Build    @relation(fields: [buildId], references: [id])

  testResults  Json     @map("test_results") // [{category, test, pass, details, severity}]
  blockerCount Int      @map("blocker_count")
  warningCount Int      @map("warning_count")
  visualQaScore Int?    @map("visual_qa_score")
  verdict      String   // pass, conditional, fail
  autoRepairs  Json?    @map("auto_repairs") // [{repairType, before, after, success}]

  createdAt    DateTime @default(now()) @map("created_at")
  @@map("qa_reports")
}

// ═══════════════════════════════════════
// OUTREACH & TRACKING
// ═══════════════════════════════════════

model OutreachPackage {
  id                    String   @id @default(uuid()) @db.Uuid
  leadId                String   @unique @map("lead_id") @db.Uuid
  lead                  Lead     @relation(fields: [leadId], references: [id])

  emailValidated        Boolean  @map("email_validated")
  emailValidationResult String?  @map("email_validation_result")
  channelStrategy       String   @map("channel_strategy")
  emailSequence         Json     @map("email_sequence") // [{touch, subject, body, personalizationVars{}}]
  analysisPdfS3Key      String?  @map("analysis_pdf_s3_key")
  comparisonScreenshots Json?    @map("comparison_screenshots") // {desktopS3Key, mobileS3Key}
  pricingUrl            String?  @map("pricing_url")

  createdAt             DateTime @default(now()) @map("created_at")
  @@map("outreach_packages")
}

model OutreachEvent {
  id          String   @id @default(uuid()) @db.Uuid
  leadId      String   @map("lead_id") @db.Uuid
  lead        Lead     @relation(fields: [leadId], references: [id])

  eventType   String   @map("event_type") // sent, opened, clicked, replied, bounced, opted_out
  touchNumber Int      @map("touch_number")
  channel     String   // email, instagram_dm, linkedin, letter
  details     Json?

  timestamp   DateTime @default(now())
  @@index([leadId, timestamp])
  @@map("outreach_events")
}

model PreviewEvent {
  id        String   @id @default(uuid()) @db.Uuid
  leadId    String   @map("lead_id") @db.Uuid
  lead      Lead     @relation(fields: [leadId], references: [id])

  eventType String   @map("event_type") // visited, scrolled, comparison_clicked, cta_clicked, calendly_clicked
  details   Json?    // {scrollDepth, durationSeconds, visitCount}

  timestamp DateTime @default(now())
  @@index([leadId, timestamp])
  @@map("preview_events")
}

// ═══════════════════════════════════════
// POST-PAYMENT
// ═══════════════════════════════════════

model Customer {
  id                 String   @id @default(uuid()) @db.Uuid
  leadId             String   @unique @map("lead_id") @db.Uuid
  lead               Lead     @relation(fields: [leadId], references: [id])

  stripeCustomerId   String   @unique @map("stripe_customer_id")
  package            String
  amountPaid         Int      @map("amount_paid") // cents
  domain             String?
  domainProvider     String?  @map("domain_provider")
  onboardingStatus   String   @default("pending") @map("onboarding_status")
  goLiveAt           DateTime? @map("go_live_at")
  dashboardAccess    Boolean  @default(false) @map("dashboard_access")

  // Monitoring
  lastUptimeCheck    DateTime? @map("last_uptime_check")
  lastLighthouseScore Int?    @map("last_lighthouse_score")
  lastFormTest       DateTime? @map("last_form_test")
  churnRisk          String   @default("low") @map("churn_risk") // low, medium, high

  createdAt          DateTime @default(now()) @map("created_at")
  @@map("customers")
}

// ═══════════════════════════════════════
// CONFIG & SYSTEM
// ═══════════════════════════════════════

model IndustryProfile {
  id        String   @id // 'coaches', 'friseure', etc.
  config    Json     // Complete industry profile config
  active    Boolean  @default(true)
  version   Int      @default(1)
  updatedAt DateTime @updatedAt @map("updated_at")

  @@map("industry_profiles")
}

model SuppressionEntry {
  id             String   @id @default(uuid()) @db.Uuid
  identifier     String   // email, domain, or phone
  identifierType String   @map("identifier_type") // email, domain, phone
  reason         String   // opted_out, bounced, spam_report, cooling, competitor
  expiresAt      DateTime? @map("expires_at") // null = permanent

  createdAt      DateTime @default(now()) @map("created_at")
  @@unique([identifier, identifierType])
  @@map("suppression_list")
}

model AuditLog {
  id         BigInt   @id @default(autoincrement())
  leadId     String?  @map("lead_id") @db.Uuid
  lead       Lead?    @relation(fields: [leadId], references: [id])

  step       String   // service name
  action     String   // what happened
  details    Json?    // additional context
  costCents  Int?     @map("cost_cents") // API cost in cents
  durationMs Int?     @map("duration_ms")
  confidence Decimal? @db.Decimal(3, 2)

  timestamp  DateTime @default(now())
  @@index([leadId, timestamp])
  @@index([step, timestamp])
  @@map("audit_log")
}
```
