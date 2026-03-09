# Webolution Pipeline Spec (Condensed)

> Full details in the .docx files in docs/docx/. This is the condensed reference for implementation.

## Step 1: Lead Scraping
**Input**: Industry Profile (search_terms, sources, geo_radius) + Region
**Output**: Raw Lead record (firmName, url, address, phone, email, googlePlaceId, rating, reviewCount, industryTag)
**Tech**: Google Places API (Nearby Search + Place Details). Playwright for portal scraping (Anwalt.de, Jameda, Treatwell etc.)
**Key Logic**: Source priority per industry. Multi-source merge (domain-match > phone-match > name-fuzzy). Rate limiting. Geo-radius varies by industry (coaches: 50km, friseure: 15km)
**Error**: API quota → exponential backoff. Portal blocked → IP rotation + stealth. Yield drop → alternative search terms
**Confidence**: Data Completeness Score (0-1) based on field presence. <0.5 → retry queue

## Step 2: Data Validation & Cleaning
**Input**: Raw Lead records
**Output**: Validated Lead records (normalized, deduplicated, reachable)
**Tech**: httpx (URL checks), DNS lookup, fuzzy matching (Levenshtein)
**6 Sequential Checks**: 1) URL format valid 2) URL reachable (HTTP 200/301) 3) Duplicate detection (domain + phone + name-fuzzy) 4) Blacklist/suppression check 5) Competitor exclusion (webdesign agencies) 6) Data normalization (phone +49, address standardize)
**Key**: Saves ~15-25% unnecessary enrichment API calls. Domain parking detection. Google "permanently closed" filter
**Pass Rate Target**: 75-85%

## Step 3: Lead Enrichment & Pre-Qualification
**Input**: Validated Lead
**Output**: Enriched Lead (+ businessHealthScore, decisionMaker, socialProfiles, industryData)
**Tech**: Google Places Details API, Impressum parsing (Playwright quick-scrape), Northdata API, LinkedIn (optional)
**Key Logic**: Waterfall API strategy (cheapest first). Cost cap €0.30/lead. Business Health Score 0-100 (Google rating, review recency, firm status, social presence, location quality). Decision Maker identification waterfall: Impressum > Portal profile > LinkedIn > generic email
**Pre-Qual Gate**: Business Health < 15 → disqualified. No contact method → deferred. Industry unverifiable → reroute
**Cost Budget**: €0.02-0.25 per lead depending on data needs

## Step 4: Website Crawl & Content Extraction
**Input**: Enriched Lead with canonical_url
**Output**: ContentPackage (pages[], images[], brandAssets{}, firmProfile{}, extractionConfidence)
**Tech**: Playwright (headless Chromium), Claude API (sonnet) for AI extraction, S3 for asset storage
**Crawl**: Max 30 pages. Screenshot each page (1440px + 375px). Cookie banner auto-accept. Boilerplate removal. Asset download (images >50px, logos, PDFs)
**AI Extraction**: Structured JSON output with confidence per field. Fields: firmName, slogan, services[], usps[], team[], foundingYear, certifications[], contact{}, openingHours{}, testimonials[], prices{}, socialLinks[], blogPosts[]
**Brand Assets**: Logo extraction (header analysis), color palette (CSS analysis), typography (font-family), image style categorization (own photos vs stock vs none)
**Edge Cases**: SPA (wait for network idle), WordPress (standard), Wix (JS rendering), content in images (OCR), PDFs (text extraction), bot protection (stealth + proxy)
**Content Completeness Score**: 0-1. <0.3 → no rebuild possible (nurture only)

## Step 5: Website Quality Review
**Input**: ContentPackage + Screenshots
**Output**: QualityReport (technicalScore, visualScore, contentScore, industryComplianceScore, weaknesses[])
**Tech**: Google PageSpeed Insights API (technical), Claude Vision (visual + content), custom checks
**Calibration**: 25-30 manually rated reference websites per industry as benchmark. Monthly drift check
**Technical Score (0-100)**: Lighthouse Performance (20%), Mobile (15%), SSL (10%), Core Web Vitals (15%), SEO basics (15%), Accessibility (10%), Load time (10%), Broken links (5%)
**Visual Score (0-100)**: AI Vision analysis with dual-pass (two evaluations, >15pt difference → third pass). Sub-scores: design modernity, color scheme, typography, image quality, layout, above-the-fold, industry fit
**Weakness Classification**: A (sales-gold: high impact + visible + fixable), B (convincing), C (supplementary), D (internal only). Only A+B go into outreach emails
**Outreach Hook**: Auto-generated sentence per weakness: "[Problem] → [Business consequence] → [What we improved]"

## Step 6: Competitor Benchmarking
**Input**: Enriched Lead (competitor_place_ids) + QualityReport
**Output**: CompetitorReport (competitors[], comparisonMatrix, relativePosition, topDifferentiators[], outreachArguments[])
**Tech**: Playwright (quick screenshots), PageSpeed API, Claude Vision (feature detection)
**Selection**: 3-5 competitors from Google Maps "Similar Places" + same category + city. Prioritize by proximity, rating, website quality. At least 1 must have a BETTER website than the lead
**Quick Analysis per Competitor** (~30s each): Screenshot, PageSpeed, mobile check, SSL, schema, AI feature detection, Google rating
**Feature Detection**: 10 categories checked via AI Vision: online booking, review integration, blog, gallery, team page, chat widget, price transparency, video, FAQ, social integration
**Output**: Comparison matrix (lead vs competitors) + relative position (1-5) + competitive pressure score (0-100) + personalized outreach arguments

## Step 7: Lead Scoring
**Input**: All data from steps 1-6
**Output**: ScoredLead (finalScore 0-100, category, routingDecision, predictedConversionProbability)
**6 Dimensions**: Website Weakness (25%), Content Opportunity (15%), Business Health (20%), Competitive Pressure (15%), Reachability (15%), Deal Potential (10%). Weights overridden per industry
**Routing**: 85-100=Hot (full pipeline, priority), 70-84=Warm (full pipeline), 55-69=Cool (nurture only), 40-54=Cold (archive), 0-39=No Fit (discard)
**Feedback Loop**: Conversion data flows back monthly. Score decay after 4 weeks (re-score before outreach). A/B testing of scoring variants
**Score Confidence**: min(all input confidences). <0.6 → manual review before pipeline continues

## Step 8: Rebuild Planning
**Input**: ContentPackage + QualityReport + CompetitorReport + IndustryProfile
**Output**: RebuildPlan (templateSelection, pageStructure, contentDecisions, brandDecision, seoPlan, legalChecklist)
**Template Selection**: Decision tree per industry based on firm size + content availability. E.g. coaches: solo-no-content→coach-landing, solo-with-blog→coach-authority. Anti-template-look: 5 variation dimensions (colors, layout, section order, fonts, image style)
**Content Decisions per Section**: KEEP (text good, >100 words, no errors), OPTIMIZE (text exists but weak), NEW_FROM_DATA (no text, but facts available), PLACEHOLDER (nice-to-have, no data), OMIT (optional, no content)
**Brand Decision**: KEEP (good logo+colors+fonts), UPGRADE (keep logo, modernize colors), NEW (poor branding, generate new scheme)
**Golden Rule**: NOTHING IS INVENTED. Every sentence must trace to an extracted data point
**Legal Checklist**: Per industry. All: Impressum + Datenschutz links. Lawyers: BRAO. Doctors: HWG. Coaches: no therapy promises
**Approval**: MVP: manual review every plan. Phase 2: auto-approve if confidence >0.8

## Step 9: Content Polish & Fact-Verification
**Input**: RebuildPlan + ContentPackage + IndustryProfile
**Output**: PolishedContent (texts{}, images{}, metaTags{}, schemaMarkup{}, factCheckReport{})
**Banned Phrases**: Universal list (10 phrases: "Wir legen Wert auf Qualität", "Ihr kompetenter Partner", etc.) + industry-specific list. Auto-scan post-generation, auto-regenerate on hit
**Tone of Voice Engine**: Per industry profile: formality, sentence length, emotionality, power words, taboo words. Loaded into Claude prompt
**Fact Verification**: Every sentence gets source_reference. Cross-check against Google Business data. Hallucination = sentence without source → remove. Zero tolerance
**Images**: Good own photos → compress+resize. Bad photos → NanoBanana replacement (marked as placeholder). No photos → NanoBanana ambient (marked). NEVER AI-generated person portraits
**SEO**: Title tags, meta descriptions, H1 structure, Schema markup (industry-specific type), local SEO (Maps embed, NAP consistency)

## Step 10: Website Assembly & Build
**Input**: RebuildPlan + PolishedContent + BrandDecision
**Output**: Deployable Next.js app (built, optimized, all assets embedded)
**Template Engine**: 4 layers: Base (Next.js+Tailwind) → Industry → Variant → Content. Section slots filled from PolishedContent
**Performance Budget (enforced)**: Lighthouse Performance >90, LCP <2.5s, CLS <0.1, Page Weight <1.5MB, All images WebP, font-display:swap
**Accessibility**: Alt texts on all images, contrast >4.5:1, keyboard nav, ARIA labels, heading hierarchy. Lighthouse a11y >90
**Forms**: Formspree or custom handler. Target email from enrichment data. Test submission at build time
**Build failure**: Auto-fix attempts (recompress images, fix alt texts, code splitting). Max 3 retries

## Step 11: QA & Testing
**Input**: Built website app + RebuildPlan + LegalChecklist
**Output**: QaReport (testResults[], verdict: pass/conditional/fail)
**5 Dimensions**: 1) Functional (links, forms, nav, maps, phone links, responsive) 2) Performance (Lighthouse scores) 3) Content integrity (correct name/phone/address, no lorem ipsum, no empty sections, spelling) 4) Legal compliance (Impressum link, Datenschutz link, HWG check for doctors, no fabricated testimonials) 5) Visual quality (AI Vision review: professional impression, rendering errors, consistency, mobile, industry fit)
**Severity**: BLOCKER (must fix, pipeline stops), WARNING (auto-fix attempted), INFO (logged only)
**Auto-repair**: Up to 3 fix cycles per blocker. Fix → rebuild → retest. After 3 fails → manual review queue
**Visual QA**: AI prompt reviews as "senior web designer". Score >80=pass, 60-80=conditional, <40=fail (change template)

## Step 12: Preview Deployment
**Input**: QA-passed website app
**Output**: Live preview at {firma}.webolution.de
**Tech**: Vercel API deployment. Wildcard domain with auto-SSL. TTL: 30 days
**Sales Layer** (5 elements): 1) Sticky top banner with CTA "Jetzt übernehmen" 2) Before/after comparison toggle (old screenshot vs new) 3) Analysis section at bottom (improvements list + PageSpeed comparison) 4) Trust elements (badges, later: customer count) 5) Countdown timer (days until preview expires)
**Protection**: noindex/nofollow, watermark on NanoBanana images, minified code, no public repo
**Analytics**: Track: visits, scroll depth, comparison clicks, CTA clicks, return visits, Calendly clicks. Feed back to orchestrator for behavioral branching

## Step 13: Outreach Preparation
**Input**: ScoredLead + QualityReport + CompetitorReport + PreviewURL
**Output**: OutreachPackage (emailSequence[], analysisPdf, comparisonScreenshots, validatedEmail, pricingUrl)
**Email Validation**: MX check → SMTP verify (NeverBounce) → disposable check → spam trap check. Invalid → fallback cascade: alt email → contact form → Instagram DM → LinkedIn → physical letter
**Hyper-Personalization**: 12 variables per email including {{top_weakness}}, {{competitor_name}}, {{competitor_advantage}}, {{industry_specific_stat}}
**Analysis PDF**: Auto-generated 1-pager: score circle, top 3 weaknesses, competitor mini-comparison, preview screenshot, QR code to preview
**Channel Strategy**: Per industry. Coaches: email→Instagram DM→LinkedIn. Friseure: email→Instagram DM→Google message. Handwerker: email→contact form→physical letter

## Step 14: Email Outreach
**Input**: OutreachPackage
**Output**: Outreach log (emails sent, engagement events)
**Tech**: Instantly.ai (warmup, sequences, domain rotation, tracking). 3-5 sender domains rotating
**5-Touch Sequence**: Day 0: Personal analysis + preview link + PDF. Day 3: IF opened=engagement follow-up, IF not=reminder with competitor argument. Day 7: Industry insight. Day 14: Scarcity (preview expiring) + Calendly. Day 21: Soft close + nurture
**Behavioral Triggers**: 3x preview visit→phone offer. CTA click no purchase→abandoned cart (1h). Reply→pause sequence, sales takes over. Opted out→immediate suppression
**Sending Times**: Industry-specific. Coaches Mo-Mi 10-11:30. Friseure Montag 10-14 (Ruhetag!). Handwerker 06:30-07:30
**Compliance**: DSGVO (public business data only), UWG §7 (lawyer review before launch), opt-out in every email, 12-month blacklist on rejection

## Step 15: Conversion & Payment
**Input**: Lead clicks CTA from preview or email
**Output**: Paying customer (Stripe customer, package selection)
**Pricing Page**: Personalized per lead (their firm name, their preview, their analysis). 3 packages: Starter €990, Professional €1990+€49/mo, Premium €3490+€99/mo. All-day language (no tech jargon)
**Trust Building**: 10 objection-handling FAQs on page. 30-day money-back guarantee. Real team photos. Calendly for 15-min call as alternative
**Checkout**: Stripe (credit card, SEPA, PayPal). No registration needed. Confirmation email with next steps. Auto-trigger onboarding

## Step 16: Managed Onboarding & Go-Live
**Input**: Paying customer + preview website
**Output**: Live website on customer domain
**Onboarding Form** (3 questions): Domain provider (dropdown), domain access (secure form), business email addresses (must not be disrupted)
**DNS Switch**: A/CNAME record only → Vercel. NEVER touch MX records. Pre-switch: MX snapshot + email test. Post-switch: MX verify + email test + SSL check + form test
**Rollback**: If email breaks after switch → revert DNS to snapshot within 5 minutes. All DNS changes are logged with before/after values
**Customer Dashboard**: Website status (green dot), contact form submissions inbox, visitor stats (Plausible), change requests (text field), invoices (Stripe portal)

## Step 17: Post-Launch Monitoring & Customer Success
**Input**: Live customer website
**Output**: Ongoing monitoring alerts, upsell recommendations, retention metrics
**Monitoring**: Uptime (every 5min), SSL expiry (daily), domain expiry (weekly), Lighthouse (weekly), form delivery (monthly)
**Touchpoints**: Day 1: welcome+video, Day 7: check-in, Day 30: performance report, Day 60: feature suggestion (upsell prep), Day 90: quarterly review, Day 335: hosting renewal
**Upsell Triggers**: High traffic+no booking tool, many form submissions, starter customer satisfied (NPS>4), Google ranking rising, new reviews, stale content
**Churn Prevention**: No dashboard login 30d→nudge email. Negative support ticket→personal call. Payment failed→update link. No renewal→phone+email
**Re-engagement** (non-converted leads): Re-score after 6 months. Seasonal triggers (Jan, Sep). Competitor trigger (competitor becomes customer). Monthly nurture newsletter
