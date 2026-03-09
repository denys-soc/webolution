**WEBOLUTION**

Technical Architecture

*Full Automation -- Event-Driven -- Self-Healing -- Scale to 10.000+ Leads/Woche*

**SocialCooks × Webolution**

März 2026

Architektur-Prinzipien

+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Design für Full Automation bei maximaler Qualität**                                                                                                                                                                                                                                                                                                                                                                                                                |
|                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| Das System muss 10.000+ Leads pro Woche verarbeiten können, ohne dass ein Mensch eingreift -- außer bei Edge Cases. Gleichzeitig darf kein Lead mit fehlerhafter Website, falschem Namen oder kaputtem Formular an den Kunden gehen. Die Architektur löst diesen Widerspruch über: Event-driven Processing, Confidence-basiertes Routing, Self-Healing mit Fallback-Kaskaden, und Human-in-the-Loop Escalation nur wenn Automation-Confidence unter Threshold fällt. |
+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

Die 8 Architektur-Prinzipien

1.  Event-Driven: Jeder Step emittiert Events. Der nächste Step wird getriggert, nicht gepollt. Asynchrone Verarbeitung über Message Queues. Kein Step wartet blockierend auf einen anderen

2.  Microservice-Architektur: Jeder der 17 Pipeline-Steps ist ein eigenständiger Service. Unabhängig deploybar, skalierbar, austauschbar. Kommunikation nur über Events und APIs

3.  Idempotenz: Jeder Service kann sicher wiederholt aufgerufen werden. Duplikate werden erkannt und ignoriert. Retries sind sicher. Kein Datenkorruptions-Risiko

4.  Confidence-basiertes Routing: Jeder Output hat einen Confidence-Score. High-Confidence → vollauto. Medium → Auto mit erhöhtem QA. Low → Human-Review-Queue

5.  Industry-Profile als Config: Branchenspezifische Logik wird NICHT im Code implementiert, sondern über JSON-Profile gesteuert. Neue Industrie = neues JSON, kein Code-Change

6.  Audit Trail: Jede Entscheidung, jedes Event, jeder API-Call wird geloggt. Vollständige Nachvollziehbarkeit pro Lead von Scraping bis Conversion

7.  Cost-Awareness: Jeder API-Call wird mit Kosten getrackt. Budget-Caps pro Lead, pro Stufe, pro Industrie. Keine unkontrollierten Kosten-Explosionen

8.  Graceful Degradation: Wenn ein externer Service ausfällt (Google API, Claude API, Vercel), darf die Pipeline nicht komplett stoppen. Fallback-Kaskaden, Retry-Logik, Queue-basiertes Recovery

Systemübersicht

Service-Landschaft

Das System besteht aus 4 Schichten: Ingestion (Daten rein), Processing (Analyse + Generierung), Delivery (Outreach + Preview), und Platform (Shared Services die alle anderen nutzen).

+----------------+---------------------+-------------------------------------+---------------------------------------------------------+
| **Schicht**    | **Services**        | **Technologie**                     | **Skalierung**                                          |
+----------------+---------------------+-------------------------------------+---------------------------------------------------------+
| **INGESTION**  | Lead Scraper        | Python + Scrapy + Playwright        | Horizontal: Mehr Worker pro Queue                       |
|                |                     |                                     |                                                         |
|                | Data Validator      | Python + httpx + DNS-Libs           |                                                         |
|                |                     |                                     |                                                         |
|                | Lead Enricher       | Python + Google/Northdata APIs      |                                                         |
+----------------+---------------------+-------------------------------------+---------------------------------------------------------+
| **PROCESSING** | Website Crawler     | Python + Playwright + Claude API    | Horizontal: CPU-/GPU-bound. Auto-Scale nach Queue-Länge |
|                |                     |                                     |                                                         |
|                | Quality Reviewer    | Python + Lighthouse + Claude Vision |                                                         |
|                |                     |                                     |                                                         |
|                | Competitor Analyzer | Python + Playwright + Claude Vision |                                                         |
|                |                     |                                     |                                                         |
|                | Scoring Engine      | Python (pure computation)           |                                                         |
|                |                     |                                     |                                                         |
|                | Rebuild Planner     | Python + Claude API                 |                                                         |
|                |                     |                                     |                                                         |
|                | Content Polisher    | Python + Claude API + NanoBanana    |                                                         |
|                |                     |                                     |                                                         |
|                | Website Builder     | Node.js + Next.js + Vercel API      |                                                         |
|                |                     |                                     |                                                         |
|                | QA Tester           | Node.js + Playwright + Lighthouse   |                                                         |
+----------------+---------------------+-------------------------------------+---------------------------------------------------------+
| **DELIVERY**   | Preview Deployer    | Node.js + Vercel API                | Horizontal: IO-bound. Scale nach Throughput             |
|                |                     |                                     |                                                         |
|                | Outreach Preparer   | Python + Claude API + PDF-Gen       |                                                         |
|                |                     |                                     |                                                         |
|                | Email Sender        | Python + Instantly.ai API           |                                                         |
|                |                     |                                     |                                                         |
|                | Payment Handler     | Node.js + Stripe API + Webhooks     |                                                         |
|                |                     |                                     |                                                         |
|                | Onboarding Manager  | Node.js + DNS APIs + Playwright     |                                                         |
|                |                     |                                     |                                                         |
|                | Customer Success    | Node.js + Plausible + Stripe        |                                                         |
+----------------+---------------------+-------------------------------------+---------------------------------------------------------+
| **PLATFORM**   | Event Bus           | Redis Streams / BullMQ              | Vertikal + Managed Services                             |
|                |                     |                                     |                                                         |
|                | Database            | PostgreSQL (Primary)                |                                                         |
|                |                     |                                     |                                                         |
|                | Object Storage      | S3-kompatibel (MinIO / AWS S3)      |                                                         |
|                |                     |                                     |                                                         |
|                | Cache               | Redis                               |                                                         |
|                |                     |                                     |                                                         |
|                | Orchestrator        | Custom (Event-driven State Machine) |                                                         |
|                |                     |                                     |                                                         |
|                | Admin Dashboard     | Next.js + tRPC                      |                                                         |
|                |                     |                                     |                                                         |
|                | Kunden-Dashboard    | Next.js + tRPC                      |                                                         |
|                |                     |                                     |                                                         |
|                | Monitoring          | Grafana + Prometheus + Sentry       |                                                         |
+----------------+---------------------+-------------------------------------+---------------------------------------------------------+

Event Bus & Orchestration

Event-Driven Pipeline

Die Pipeline ist NICHT eine sequentielle Kette von API-Calls. Stattdessen: Jeder Service emittiert Events in den Event Bus (Redis Streams). Andere Services subscriben auf relevante Events und reagieren autonom.

+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Warum Event-Driven statt Request/Response?**                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
|                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| 1\. Entkopplung: Wenn der Website Crawler 30 Sekunden braucht, blockiert er nicht den Lead Enricher der parallel andere Leads bearbeitet. 2. Retry ohne Datenverlust: Wenn ein Service crashed, liegen die Events noch in der Queue und werden erneut verarbeitet. 3. Parallelisierung: Stufe 5 (Quality Review) und Stufe 6 (Competitor Benchmarking) können PARALLEL laufen -- beide subscriben auf das „crawl.completed" Event. 4. Observability: Jedes Event ist sichtbar, loggbar, replaybar. Debugging wird trivial. |
+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

Event-Katalog

  ---------------------- ------------------------------ ----------------------------------------- -------------------------------------------------------------------------
  **Event Name**         **Emittiert von (Step)**       **Konsumiert von**                        **Payload (Kern)**

  lead.scraped           1: Scraper                     2: Validator                              lead_id, firm_name, url, source, industry_tag

  lead.validated         2: Validator                   3: Enricher                               lead_id, validation_confidence, canonical_url

  lead.enriched          3: Enricher                    4: Crawler                                lead_id, enrichment_confidence, business_health_score, decision_maker{}

  lead.disqualified      2/3: Validator/Enricher        Orchestrator (logging)                    lead_id, reason, step

  crawl.completed        4: Crawler                     5: Reviewer + 6: Competitor (parallel!)   lead_id, content_package_id, extraction_confidence

  review.completed       5: Reviewer                    7: Scorer (wartet auch auf 6)             lead_id, quality_report_id, weakness_count

  competitor.completed   6: Competitor                  7: Scorer                                 lead_id, competitor_report_id, competitive_pressure_score

  lead.scored            7: Scorer                      Orchestrator (routing)                    lead_id, final_score, category, routing_decision

  rebuild.planned        8: Planner                     9: Polisher                               lead_id, rebuild_plan_id, rebuild_confidence

  content.polished       9: Polisher                    10: Builder                               lead_id, polished_content_id, polish_confidence

  website.built          10: Builder                    11: QA                                    lead_id, build_id, lighthouse_scores{}

  qa.passed              11: QA                         12: Deployer                              lead_id, qa_report_id, verdict

  qa.failed              11: QA                         Orchestrator (retry/escalate)             lead_id, blocker_count, auto_fixable

  preview.deployed       12: Deployer                   13: Outreach Prep                         lead_id, preview_url, deploy_timestamp

  outreach.prepared      13: Preparer                   14: Sender                                lead_id, outreach_package_id, channel

  email.sent             14: Sender                     Orchestrator (tracking)                   lead_id, email_id, touch_number, status

  email.opened           14: Sender (webhook)           Orchestrator (branching)                  lead_id, email_id, timestamp

  preview.visited        12: Analytics (webhook)        Orchestrator (score update)               lead_id, visit_count, scroll_depth, cta_clicked

  payment.completed      15: Payment (Stripe webhook)   16: Onboarding                            lead_id, customer_id, package, amount

  onboarding.completed   16: Onboarding                 17: Customer Success                      lead_id, customer_id, domain, go_live_timestamp

  lead.nurture           7: Scorer (Cool leads)         14: Sender (Nurture track)                lead_id, score, nurture_reason
  ---------------------- ------------------------------ ----------------------------------------- -------------------------------------------------------------------------

Orchestrator: State Machine pro Lead

Jeder Lead durchläuft eine State Machine. Der Orchestrator trackt den Status jedes Leads und trifft Routing-Entscheidungen basierend auf Events und Confidence-Scores.

+-----------------------------------------------------------------------+
| **Lead State Machine (vereinfacht)**                                  |
+-----------------------------------------------------------------------+
| SCRAPED → VALIDATING → VALIDATED → ENRICHING → ENRICHED               |
|                                                                       |
| → CRAWLING → CRAWLED → \[REVIEWING + BENCHMARKING\] (parallel)        |
|                                                                       |
| → SCORED → {                                                          |
|                                                                       |
| HOT/WARM: PLANNING → POLISHING → BUILDING → QA_TESTING                |
|                                                                       |
| → QA_PASSED → DEPLOYING → DEPLOYED → OUTREACH_PREPPING                |
|                                                                       |
| → OUTREACH_ACTIVE → {                                                 |
|                                                                       |
| CONVERTED → ONBOARDING → LIVE → CUSTOMER_SUCCESS                      |
|                                                                       |
| NO_RESPONSE → NURTURE_POOL                                            |
|                                                                       |
| OPTED_OUT → SUPPRESSED                                                |
|                                                                       |
| }                                                                     |
|                                                                       |
| COOL: NURTURE_POOL (nur Analyse-Report)                               |
|                                                                       |
| COLD: ARCHIVED (Re-Score in 6 Monaten)                                |
|                                                                       |
| }                                                                     |
|                                                                       |
| Error States (von jedem State erreichbar):                            |
|                                                                       |
| → RETRY_PENDING (auto-retry nach Delay)                               |
|                                                                       |
| → MANUAL_REVIEW (Confidence \< threshold)                             |
|                                                                       |
| → PERMANENTLY_FAILED (max retries exceeded)                           |
+-----------------------------------------------------------------------+

Parallel Processing (Stufe 5 + 6)

Quality Review (Stufe 5) und Competitor Benchmarking (Stufe 6) laufen parallel. Stufe 7 (Scoring) wartet auf BEIDE:

-   crawl.completed Event wird von BEIDEN Services konsumiert

-   Orchestrator trackt: review_done = false, competitor_done = false

-   Wenn review.completed eintrifft: review_done = true. Prüfe ob competitor_done auch true → wenn ja: Scoring triggern

-   Wenn competitor.completed eintrifft: competitor_done = true. Gleiche Logik

-   Timeout: Wenn einer der beiden nach 5 Minuten nicht fertig ist: Scoring mit verfügbaren Daten starten (Confidence reduzieren)

Datenbank-Architektur

PostgreSQL als Primary Database

PostgreSQL ist der Single Source of Truth für alle strukturierten Daten. JSONB-Spalten für semi-strukturierte Daten (Industry Profiles, Rebuild Plans, Quality Reports). S3 für Binärdaten (Screenshots, Bilder, PDFs).

Kern-Tabellen

+----------------------------------------------------------------------------+
| **leads (Zentrale Lead-Tabelle)**                                          |
+----------------------------------------------------------------------------+
| id UUID PRIMARY KEY DEFAULT gen_random_uuid()                              |
|                                                                            |
| state VARCHAR(50) NOT NULL DEFAULT \'scraped\'                             |
|                                                                            |
| industry_tag VARCHAR(50) NOT NULL                                          |
|                                                                            |
| region VARCHAR(100)                                                        |
|                                                                            |
| firm_name VARCHAR(500) NOT NULL                                            |
|                                                                            |
| canonical_url VARCHAR(2000)                                                |
|                                                                            |
| phone VARCHAR(50)                                                          |
|                                                                            |
| email_generic VARCHAR(500)                                                 |
|                                                                            |
| address JSONB \-- {street, zip, city, lat, lng}                            |
|                                                                            |
| google_place_id VARCHAR(200)                                               |
|                                                                            |
| google_rating DECIMAL(2,1)                                                 |
|                                                                            |
| google_reviews INTEGER                                                     |
|                                                                            |
| source_tags VARCHAR(200)\[\] \-- Array: \[\'google_maps\', \'anwalt_de\'\] |
|                                                                            |
| \-- Enrichment Data                                                        |
|                                                                            |
| decision_maker JSONB \-- {name, role, email, phone, linkedin_url}          |
|                                                                            |
| business_health_score INTEGER                                              |
|                                                                            |
| social_profiles JSONB \-- {instagram, facebook, linkedin}                  |
|                                                                            |
| industry_data JSONB \-- Branchenspezifische Daten                          |
|                                                                            |
| \-- Scoring                                                                |
|                                                                            |
| final_score INTEGER                                                        |
|                                                                            |
| score_components JSONB \-- {website_weakness, content_opp, \...}           |
|                                                                            |
| score_category VARCHAR(20) \-- hot/warm/cool/cold                          |
|                                                                            |
| score_confidence DECIMAL(3,2)                                              |
|                                                                            |
| scored_at TIMESTAMP                                                        |
|                                                                            |
| \-- Confidence Scores (pro Step)                                           |
|                                                                            |
| validation_confidence DECIMAL(3,2)                                         |
|                                                                            |
| enrichment_confidence DECIMAL(3,2)                                         |
|                                                                            |
| extraction_confidence DECIMAL(3,2)                                         |
|                                                                            |
| review_confidence DECIMAL(3,2)                                             |
|                                                                            |
| rebuild_confidence DECIMAL(3,2)                                            |
|                                                                            |
| polish_confidence DECIMAL(3,2)                                             |
|                                                                            |
| \-- Tracking                                                               |
|                                                                            |
| preview_url VARCHAR(500)                                                   |
|                                                                            |
| preview_deployed_at TIMESTAMP                                              |
|                                                                            |
| outreach_status VARCHAR(50)                                                |
|                                                                            |
| stripe_customer_id VARCHAR(200)                                            |
|                                                                            |
| customer_package VARCHAR(50)                                               |
|                                                                            |
| go_live_at TIMESTAMP                                                       |
|                                                                            |
| \-- Meta                                                                   |
|                                                                            |
| created_at TIMESTAMP DEFAULT NOW()                                         |
|                                                                            |
| updated_at TIMESTAMP DEFAULT NOW()                                         |
|                                                                            |
| \-- Indexes                                                                |
|                                                                            |
| INDEX idx_leads_state ON leads(state)                                      |
|                                                                            |
| INDEX idx_leads_industry ON leads(industry_tag)                            |
|                                                                            |
| INDEX idx_leads_score ON leads(final_score DESC)                           |
|                                                                            |
| INDEX idx_leads_region ON leads(region)                                    |
|                                                                            |
| UNIQUE INDEX idx_leads_url ON leads(canonical_url)                         |
+----------------------------------------------------------------------------+

+-------------------------------------------------------------------------------+
| **content_packages (Extrahierter Website-Content)**                           |
+-------------------------------------------------------------------------------+
| id UUID PRIMARY KEY                                                           |
|                                                                               |
| lead_id UUID REFERENCES leads(id)                                             |
|                                                                               |
| \-- Crawl Results                                                             |
|                                                                               |
| pages JSONB \-- \[{url, title, text, html_hash, screenshot_s3_key}\]          |
|                                                                               |
| page_count INTEGER                                                            |
|                                                                               |
| crawl_duration_ms INTEGER                                                     |
|                                                                               |
| \-- Extracted Profile                                                         |
|                                                                               |
| firm_profile JSONB \-- {name, slogan, services\[\], usps\[\], team\[\], \...} |
|                                                                               |
| \-- Brand Assets                                                              |
|                                                                               |
| brand_assets JSONB \-- {logo_s3_key, colors{}, fonts{}, image_style}          |
|                                                                               |
| images JSONB \-- \[{s3_key, alt, width, height, type, quality_score}\]        |
|                                                                               |
| \-- Scores                                                                    |
|                                                                               |
| extraction_confidence DECIMAL(3,2)                                            |
|                                                                               |
| content_completeness DECIMAL(3,2)                                             |
|                                                                               |
| created_at TIMESTAMP DEFAULT NOW()                                            |
+-------------------------------------------------------------------------------+

+----------------------------------------------------------------------------------------+
| **quality_reports (Website-Bewertung)**                                                |
+----------------------------------------------------------------------------------------+
| id UUID PRIMARY KEY                                                                    |
|                                                                                        |
| lead_id UUID REFERENCES leads(id)                                                      |
|                                                                                        |
| technical_score INTEGER                                                                |
|                                                                                        |
| visual_score INTEGER                                                                   |
|                                                                                        |
| content_score INTEGER                                                                  |
|                                                                                        |
| industry_compliance_score INTEGER                                                      |
|                                                                                        |
| overall_score INTEGER                                                                  |
|                                                                                        |
| weaknesses JSONB \-- \[{type, severity, class, description, evidence, outreach_hook}\] |
|                                                                                        |
| review_confidence DECIMAL(3,2)                                                         |
|                                                                                        |
| created_at TIMESTAMP DEFAULT NOW()                                                     |
+----------------------------------------------------------------------------------------+

+-----------------------------------------------------------------------------------------+
| **competitor_reports**                                                                  |
+-----------------------------------------------------------------------------------------+
| id UUID PRIMARY KEY                                                                     |
|                                                                                         |
| lead_id UUID REFERENCES leads(id)                                                       |
|                                                                                         |
| competitors JSONB \-- \[{name, url, scores{}, features\[\], design_epoch}\]             |
|                                                                                         |
| comparison_matrix JSONB                                                                 |
|                                                                                         |
| relative_position INTEGER \-- 1-5                                                       |
|                                                                                         |
| competitive_pressure_score INTEGER                                                      |
|                                                                                         |
| top_differentiators JSONB \-- \[{feature, competitor_has, lead_missing, outreach_arg}\] |
|                                                                                         |
| created_at TIMESTAMP DEFAULT NOW()                                                      |
+-----------------------------------------------------------------------------------------+

+------------------------------------------------------------------------------------+
| **rebuild_plans**                                                                  |
+------------------------------------------------------------------------------------+
| id UUID PRIMARY KEY                                                                |
|                                                                                    |
| lead_id UUID REFERENCES leads(id)                                                  |
|                                                                                    |
| template_id VARCHAR(100)                                                           |
|                                                                                    |
| template_variant VARCHAR(100)                                                      |
|                                                                                    |
| page_structure JSONB \-- \[{page, sections\[{id, source, action, content_ref}\]}\] |
|                                                                                    |
| brand_decision VARCHAR(20) \-- keep/upgrade/new                                    |
|                                                                                    |
| brand_config JSONB \-- {colors{}, fonts{}, logo_strategy}                          |
|                                                                                    |
| seo_plan JSONB \-- {primary_kw\[\], secondary_kw\[\], local_seo{}}                 |
|                                                                                    |
| cta_strategy JSONB \-- {primary_text, secondary_text, placement\[\]}               |
|                                                                                    |
| legal_checklist JSONB \-- \[{requirement, status, source}\]                        |
|                                                                                    |
| image_strategy JSONB \-- {per_section\[{section, source, action}\]}                |
|                                                                                    |
| rebuild_confidence DECIMAL(3,2)                                                    |
|                                                                                    |
| approved BOOLEAN DEFAULT FALSE                                                     |
|                                                                                    |
| approved_by VARCHAR(50) \-- \'auto\' or reviewer name                              |
|                                                                                    |
| created_at TIMESTAMP DEFAULT NOW()                                                 |
+------------------------------------------------------------------------------------+

+--------------------------------------------------------------------------------------+
| **polished_content**                                                                 |
+--------------------------------------------------------------------------------------+
| id UUID PRIMARY KEY                                                                  |
|                                                                                      |
| lead_id UUID REFERENCES leads(id)                                                    |
|                                                                                      |
| rebuild_plan_id UUID REFERENCES rebuild_plans(id)                                    |
|                                                                                      |
| texts JSONB \-- {per_section: {section_id: {text, source_refs\[\], confidence}}}     |
|                                                                                      |
| images JSONB \-- {per_section: {section_id: {s3_key, source, is_placeholder}}}       |
|                                                                                      |
| meta_tags JSONB \-- {title, description, og_title, og_description, og_image}         |
|                                                                                      |
| schema_markup JSONB                                                                  |
|                                                                                      |
| fact_check_report JSONB \-- {checked\[\], confirmed\[\], corrected\[\], removed\[\]} |
|                                                                                      |
| banned_phrase_hits INTEGER                                                           |
|                                                                                      |
| polish_confidence DECIMAL(3,2)                                                       |
|                                                                                      |
| created_at TIMESTAMP DEFAULT NOW()                                                   |
+--------------------------------------------------------------------------------------+

+-----------------------------------------------------------------------+
| **builds**                                                            |
+-----------------------------------------------------------------------+
| id UUID PRIMARY KEY                                                   |
|                                                                       |
| lead_id UUID REFERENCES leads(id)                                     |
|                                                                       |
| polished_content_id UUID REFERENCES polished_content(id)              |
|                                                                       |
| lighthouse_performance INTEGER                                        |
|                                                                       |
| lighthouse_accessibility INTEGER                                      |
|                                                                       |
| lighthouse_seo INTEGER                                                |
|                                                                       |
| page_weight_kb INTEGER                                                |
|                                                                       |
| build_duration_ms INTEGER                                             |
|                                                                       |
| build_log TEXT                                                        |
|                                                                       |
| artifact_s3_key VARCHAR(500) \-- Built Next.js app                    |
|                                                                       |
| created_at TIMESTAMP DEFAULT NOW()                                    |
+-----------------------------------------------------------------------+

+-----------------------------------------------------------------------+
| **qa_reports**                                                        |
+-----------------------------------------------------------------------+
| id UUID PRIMARY KEY                                                   |
|                                                                       |
| lead_id UUID REFERENCES leads(id)                                     |
|                                                                       |
| build_id UUID REFERENCES builds(id)                                   |
|                                                                       |
| test_results JSONB \-- \[{category, test, pass, details, severity}\]  |
|                                                                       |
| blocker_count INTEGER                                                 |
|                                                                       |
| warning_count INTEGER                                                 |
|                                                                       |
| visual_qa_score INTEGER                                               |
|                                                                       |
| verdict VARCHAR(20) \-- pass/conditional/fail                         |
|                                                                       |
| auto_repairs JSONB \-- \[{repair_type, before, after, success}\]      |
|                                                                       |
| created_at TIMESTAMP DEFAULT NOW()                                    |
+-----------------------------------------------------------------------+

+-----------------------------------------------------------------------------+
| **outreach_packages**                                                       |
+-----------------------------------------------------------------------------+
| id UUID PRIMARY KEY                                                         |
|                                                                             |
| lead_id UUID REFERENCES leads(id)                                           |
|                                                                             |
| email_validated BOOLEAN                                                     |
|                                                                             |
| email_validation_result VARCHAR(20)                                         |
|                                                                             |
| channel_strategy VARCHAR(50)                                                |
|                                                                             |
| email_sequence JSONB \-- \[{touch, subject, body, personalization_vars{}}\] |
|                                                                             |
| analysis_pdf_s3_key VARCHAR(500)                                            |
|                                                                             |
| comparison_screenshots JSONB \-- {desktop_s3_key, mobile_s3_key}            |
|                                                                             |
| pricing_url VARCHAR(500)                                                    |
|                                                                             |
| created_at TIMESTAMP DEFAULT NOW()                                          |
+-----------------------------------------------------------------------------+

+--------------------------------------------------------------------------+
| **outreach_events (Event Log)**                                          |
+--------------------------------------------------------------------------+
| id UUID PRIMARY KEY                                                      |
|                                                                          |
| lead_id UUID REFERENCES leads(id)                                        |
|                                                                          |
| event_type VARCHAR(50) \-- sent/opened/clicked/replied/bounced/opted_out |
|                                                                          |
| touch_number INTEGER                                                     |
|                                                                          |
| channel VARCHAR(50) \-- email/instagram_dm/linkedin/letter               |
|                                                                          |
| details JSONB                                                            |
|                                                                          |
| timestamp TIMESTAMP DEFAULT NOW()                                        |
|                                                                          |
| INDEX idx_outreach_lead ON outreach_events(lead_id, timestamp)           |
+--------------------------------------------------------------------------+

+---------------------------------------------------------------------------------------------+
| **preview_events (Analytics)**                                                              |
+---------------------------------------------------------------------------------------------+
| id UUID PRIMARY KEY                                                                         |
|                                                                                             |
| lead_id UUID REFERENCES leads(id)                                                           |
|                                                                                             |
| event_type VARCHAR(50) \-- visited/scrolled/comparison_clicked/cta_clicked/calendly_clicked |
|                                                                                             |
| details JSONB \-- {scroll_depth, duration_seconds, visit_count}                             |
|                                                                                             |
| timestamp TIMESTAMP DEFAULT NOW()                                                           |
|                                                                                             |
| INDEX idx_preview_lead ON preview_events(lead_id, timestamp)                                |
+---------------------------------------------------------------------------------------------+

+-----------------------------------------------------------------------+
| **customers (Post-Payment)**                                          |
+-----------------------------------------------------------------------+
| id UUID PRIMARY KEY                                                   |
|                                                                       |
| lead_id UUID REFERENCES leads(id)                                     |
|                                                                       |
| stripe_customer_id VARCHAR(200) UNIQUE                                |
|                                                                       |
| package VARCHAR(50)                                                   |
|                                                                       |
| amount_paid INTEGER \-- in cents                                      |
|                                                                       |
| domain VARCHAR(500)                                                   |
|                                                                       |
| domain_provider VARCHAR(100)                                          |
|                                                                       |
| onboarding_status VARCHAR(50)                                         |
|                                                                       |
| go_live_at TIMESTAMP                                                  |
|                                                                       |
| dashboard_access BOOLEAN DEFAULT FALSE                                |
|                                                                       |
| \-- Monitoring                                                        |
|                                                                       |
| last_uptime_check TIMESTAMP                                           |
|                                                                       |
| last_lighthouse_score INTEGER                                         |
|                                                                       |
| last_form_test TIMESTAMP                                              |
|                                                                       |
| churn_risk VARCHAR(20) \-- low/medium/high                            |
|                                                                       |
| created_at TIMESTAMP DEFAULT NOW()                                    |
+-----------------------------------------------------------------------+

+-----------------------------------------------------------------------------------+
| **industry_profiles (JSON Config)**                                               |
+-----------------------------------------------------------------------------------+
| id VARCHAR(50) PRIMARY KEY \-- \'coaches\', \'friseure\', etc.                    |
|                                                                                   |
| config JSONB \-- Komplettes Industry Profile (scraping, enrichment, review, \...) |
|                                                                                   |
| active BOOLEAN DEFAULT TRUE                                                       |
|                                                                                   |
| version INTEGER DEFAULT 1                                                         |
|                                                                                   |
| updated_at TIMESTAMP DEFAULT NOW()                                                |
+-----------------------------------------------------------------------------------+

+-------------------------------------------------------------------------------+
| **suppression_list**                                                          |
+-------------------------------------------------------------------------------+
| id UUID PRIMARY KEY                                                           |
|                                                                               |
| identifier VARCHAR(500) NOT NULL \-- email, domain, or phone                  |
|                                                                               |
| identifier_type VARCHAR(20) \-- email/domain/phone                            |
|                                                                               |
| reason VARCHAR(50) \-- opted_out/bounced/spam_report/cooling/competitor       |
|                                                                               |
| expires_at TIMESTAMP \-- NULL = permanent, date = cooling period              |
|                                                                               |
| created_at TIMESTAMP DEFAULT NOW()                                            |
|                                                                               |
| UNIQUE INDEX idx_suppression ON suppression_list(identifier, identifier_type) |
+-------------------------------------------------------------------------------+

+-----------------------------------------------------------------------+
| **audit_log (Vollständiges Audit Trail)**                             |
+-----------------------------------------------------------------------+
| id BIGSERIAL PRIMARY KEY                                              |
|                                                                       |
| lead_id UUID                                                          |
|                                                                       |
| step VARCHAR(50)                                                      |
|                                                                       |
| action VARCHAR(100)                                                   |
|                                                                       |
| details JSONB                                                         |
|                                                                       |
| cost_cents INTEGER \-- API-Kosten in Cent                             |
|                                                                       |
| duration_ms INTEGER                                                   |
|                                                                       |
| confidence DECIMAL(3,2)                                               |
|                                                                       |
| timestamp TIMESTAMP DEFAULT NOW()                                     |
|                                                                       |
| INDEX idx_audit_lead ON audit_log(lead_id, timestamp)                 |
|                                                                       |
| INDEX idx_audit_step ON audit_log(step, timestamp)                    |
+-----------------------------------------------------------------------+

Infrastruktur & Deployment

Cloud-Architektur

+-------------------------------+-----------------------------------------+------------------------------------+----------------------------------+
| **Komponente**                | **Service**                             | **Spezifikation**                  | **Kosten (MVP Schätzung/Monat)** |
+-------------------------------+-----------------------------------------+------------------------------------+----------------------------------+
| **Compute: Pipeline Workers** | Hetzner Cloud / Fly.io                  | 4-8 Worker-VMs (4 vCPU, 8GB RAM)   | €80-200                          |
|                               |                                         |                                    |                                  |
|                               |                                         | Auto-Scale: 2 Base + 2-6 On-Demand |                                  |
|                               |                                         |                                    |                                  |
|                               |                                         | Docker Container pro Service       |                                  |
+-------------------------------+-----------------------------------------+------------------------------------+----------------------------------+
| **Compute: Web Apps**         | Vercel (Frontend) + Fly.io (API)        | Admin Dashboard: Vercel Pro        | €40-80                           |
|                               |                                         |                                    |                                  |
|                               |                                         | Kunden Dashboard: Vercel Pro       |                                  |
|                               |                                         |                                    |                                  |
|                               |                                         | API Server: Fly.io 2x shared-cpu   |                                  |
+-------------------------------+-----------------------------------------+------------------------------------+----------------------------------+
| **Database**                  | Supabase oder Neon (Managed PostgreSQL) | Pro Plan: 8GB RAM, 50GB Storage    | €25-50                           |
|                               |                                         |                                    |                                  |
|                               |                                         | Connection Pooling (PgBouncer)     |                                  |
|                               |                                         |                                    |                                  |
|                               |                                         | Daily Backups                      |                                  |
+-------------------------------+-----------------------------------------+------------------------------------+----------------------------------+
| **Redis**                     | Upstash Redis (Serverless)              | Event Bus (Redis Streams)          | €10-30                           |
|                               |                                         |                                    |                                  |
|                               |                                         | Cache (Session, API-Responses)     |                                  |
|                               |                                         |                                    |                                  |
|                               |                                         | Rate Limiting                      |                                  |
+-------------------------------+-----------------------------------------+------------------------------------+----------------------------------+
| **Object Storage**            | Cloudflare R2 oder Hetzner Storage      | Screenshots, Bilder, PDFs          | €5-15                            |
|                               |                                         |                                    |                                  |
|                               |                                         | Built Website Artifacts            |                                  |
|                               |                                         |                                    |                                  |
|                               |                                         | Kein Egress-Kosten (R2)            |                                  |
+-------------------------------+-----------------------------------------+------------------------------------+----------------------------------+
| **Preview Hosting**           | Vercel                                  | Wildcard-Domain \*.webolution.de   | €20-100 (scale-abhängig)         |
|                               |                                         |                                    |                                  |
|                               |                                         | Auto-SSL                           |                                  |
|                               |                                         |                                    |                                  |
|                               |                                         | Pro Plan für Bandwidth             |                                  |
+-------------------------------+-----------------------------------------+------------------------------------+----------------------------------+
| **Email Sending**             | Instantly.ai                            | Growth Plan: 5.000 Emails/Monat    | €97                              |
|                               |                                         |                                    |                                  |
|                               |                                         | 5 Sender-Accounts                  |                                  |
|                               |                                         |                                    |                                  |
|                               |                                         | Warmup inklusive                   |                                  |
+-------------------------------+-----------------------------------------+------------------------------------+----------------------------------+
| **Monitoring**                | Grafana Cloud (Free Tier) + Sentry      | Metrics + Logs + Traces            | €0-30                            |
|                               |                                         |                                    |                                  |
|                               |                                         | Error Tracking + Alerting          |                                  |
|                               |                                         |                                    |                                  |
|                               |                                         | Uptime Monitoring                  |                                  |
+-------------------------------+-----------------------------------------+------------------------------------+----------------------------------+
| **DNS**                       | Cloudflare                              | webolution.de DNS                  | €0 (Free)                        |
|                               |                                         |                                    |                                  |
|                               |                                         | Wildcard-Setup                     |                                  |
|                               |                                         |                                    |                                  |
|                               |                                         | DDoS Protection                    |                                  |
+-------------------------------+-----------------------------------------+------------------------------------+----------------------------------+
| **CI/CD**                     | GitHub Actions                          | Auto-Deploy on Push                | €0 (Free Tier)                   |
|                               |                                         |                                    |                                  |
|                               |                                         | Test-Suite                         |                                  |
|                               |                                         |                                    |                                  |
|                               |                                         | Container-Builds                   |                                  |
+-------------------------------+-----------------------------------------+------------------------------------+----------------------------------+

+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Geschätzte Gesamtkosten Infrastruktur (MVP): €280-600/Monat**                                                                                                                                        |
|                                                                                                                                                                                                        |
| Plus variable API-Kosten (Google, Claude, NanoBanana, NeverBounce): €200-500/Monat bei 500 Leads/Woche. Gesamt MVP: €500-1.100/Monat. Bei 10 Conversions à €1.500 = €15.000 Revenue → sehr profitabel. |
+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

API-Kosten-Modell pro Lead

  ------------------------------------- ------------------------------------------------ --------------------- ---------------
  **Step**                              **API-Calls**                                    **Kosten pro Lead**   **Kumuliert**

  1\. Scraping                          Google Places: 1 Nearby + 1 Detail               €0.03                 €0.03

  2\. Validation                        HTTP Checks (self-hosted), DNS Lookup            €0.00                 €0.03

  3\. Enrichment                        Google Details + ggf. Northdata + NeverBounce    €0.05-0.15            €0.08-0.18

  4\. Crawling + Extraction             Playwright (Compute) + Claude API (Extraction)   €0.08-0.15            €0.16-0.33

  5\. Quality Review                    Lighthouse API + Claude Vision (2 passes)        €0.04-0.08            €0.20-0.41

  6\. Competitor Benchmarking           3-5 Quick-Scans + Claude Vision                  €0.05-0.10            €0.25-0.51

  7\. Scoring                           Pure Computation (kein API)                      €0.00                 €0.25-0.51

  8\. Rebuild Planning                  Claude API (Plan-Generierung)                    €0.03-0.06            €0.28-0.57

  9\. Content Polish                    Claude API (Texte) + NanoBanana (Bilder)         €0.10-0.25            €0.38-0.82

  10\. Website Build                    Compute (Next.js Build)                          €0.02                 €0.40-0.84

  11\. QA & Testing                     Lighthouse + Playwright + Claude Vision          €0.04-0.08            €0.44-0.92

  12\. Preview Deploy                   Vercel API (Deploy)                              €0.01                 €0.45-0.93

  13\. Outreach Prep                    Claude API (Emails) + PDF-Gen + NeverBounce      €0.05-0.10            €0.50-1.03

  14\. Email Sending                    Instantly.ai (5 Emails)                          €0.02-0.05            €0.52-1.08

  **GESAMT pro Lead (Full Pipeline)**                                                    **€0.52-1.08**        
  ------------------------------------- ------------------------------------------------ --------------------- ---------------

+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Unit Economics**                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
|                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| Cost per Lead (Full Pipeline): \~€0.80 Avg. Conversion Rate: \~6% (Visit → Payment) Leads needed per Conversion: \~17 Full-Pipeline-Leads (nur Hot+Warm gehen durch Full Pipeline, das sind \~30% aller Leads. Also \~55 Raw Leads pro Conversion) Cost per Acquisition: 55 × €0.30 (Avg inkl. abgebrochene) = \~€16.50 Avg. Deal Size: €1.500 ROI pro Conversion: €1.500 / €16.50 = 91x Return Selbst bei pessimistischen Annahmen (3% Conversion, €2/Lead): €1.500 / €110 = 14x Return |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

Service-Architektur: Details

Worker-Architektur (Python Services)

Alle Pipeline-Services (Steps 1-14) folgen dem gleichen Worker-Pattern:

+-----------------------------------------------------------------------+
| **Worker-Pattern (Pseudo-Code)**                                      |
+-----------------------------------------------------------------------+
| class PipelineWorker:                                                 |
|                                                                       |
| def \_\_init\_\_(self, step_name, input_event, output_event):         |
|                                                                       |
| self.step = step_name                                                 |
|                                                                       |
| self.consumer = RedisStreamConsumer(input_event)                      |
|                                                                       |
| self.producer = RedisStreamProducer(output_event)                     |
|                                                                       |
| self.db = DatabaseConnection()                                        |
|                                                                       |
| self.audit = AuditLogger(step_name)                                   |
|                                                                       |
| async def run():                                                      |
|                                                                       |
| while True:                                                           |
|                                                                       |
| event = await self.consumer.read(block=5000) \# 5s timeout            |
|                                                                       |
| if event:                                                             |
|                                                                       |
| lead_id = event.payload.lead_id                                       |
|                                                                       |
| try:                                                                  |
|                                                                       |
| \# Idempotency check                                                  |
|                                                                       |
| if self.already_processed(lead_id): continue                          |
|                                                                       |
| \# Load industry profile                                              |
|                                                                       |
| profile = self.db.get_industry_profile(event.payload.industry_tag)    |
|                                                                       |
| \# Process                                                            |
|                                                                       |
| start = time.now()                                                    |
|                                                                       |
| result = await self.process(event.payload, profile)                   |
|                                                                       |
| duration = time.now() - start                                         |
|                                                                       |
| \# Confidence check + routing                                         |
|                                                                       |
| if result.confidence \< MANUAL_REVIEW_THRESHOLD:                      |
|                                                                       |
| await self.escalate_to_review(lead_id, result)                        |
|                                                                       |
| else:                                                                 |
|                                                                       |
| await self.producer.emit(result)                                      |
|                                                                       |
| \# Audit log                                                          |
|                                                                       |
| self.audit.log(lead_id, result, duration, result.cost)                |
|                                                                       |
| except RetryableError as e:                                           |
|                                                                       |
| await self.retry(event, delay=e.retry_after)                          |
|                                                                       |
| except PermanentError as e:                                           |
|                                                                       |
| await self.fail_permanently(lead_id, e)                               |
|                                                                       |
| except Exception as e:                                                |
|                                                                       |
| sentry.capture(e)                                                     |
|                                                                       |
| await self.retry(event, delay=60) \# generic retry                    |
+-----------------------------------------------------------------------+

Error-Kategorisierung

+---------------------+-------------------------------------+-----------------------------------------------------------------------------+-----------------+
| **Error-Typ**       | **Beispiele**                       | **Handling**                                                                | **Max Retries** |
+---------------------+-------------------------------------+-----------------------------------------------------------------------------+-----------------+
| **RetryableError**  | HTTP 429 (Rate Limit)               | Exponential Backoff: 10s, 30s, 90s, 270s. Event bleibt in Queue             | 4               |
|                     |                                     |                                                                             |                 |
|                     | HTTP 5xx (Server Error)             |                                                                             |                 |
|                     |                                     |                                                                             |                 |
|                     | Timeout                             |                                                                             |                 |
|                     |                                     |                                                                             |                 |
|                     | Redis Connection Lost               |                                                                             |                 |
+---------------------+-------------------------------------+-----------------------------------------------------------------------------+-----------------+
| **PermanentError**  | HTTP 404 (Website nicht erreichbar) | Lead als PERMANENTLY_FAILED markieren. Alert ans Team wenn \>5% betroffen   | 0               |
|                     |                                     |                                                                             |                 |
|                     | Invalid Data (nicht reparabel)      |                                                                             |                 |
|                     |                                     |                                                                             |                 |
|                     | API Key ungültig                    |                                                                             |                 |
+---------------------+-------------------------------------+-----------------------------------------------------------------------------+-----------------+
| **ConfidenceError** | AI-Output unter Threshold           | In MANUAL_REVIEW_QUEUE verschieben. Dashboard-Notification. Kein Auto-Retry | 0 (manuell)     |
|                     |                                     |                                                                             |                 |
|                     | Halluzination erkannt               |                                                                             |                 |
|                     |                                     |                                                                             |                 |
|                     | Unvollständige Extraction           |                                                                             |                 |
+---------------------+-------------------------------------+-----------------------------------------------------------------------------+-----------------+
| **UnexpectedError** | Uncaught Exceptions                 | Sentry-Alert. Auto-Retry 1x nach 60s. Bei 2. Fail: MANUAL_REVIEW_QUEUE      | 1               |
|                     |                                     |                                                                             |                 |
|                     | Memory-Fehler                       |                                                                             |                 |
|                     |                                     |                                                                             |                 |
|                     | Segfaults                           |                                                                             |                 |
+---------------------+-------------------------------------+-----------------------------------------------------------------------------+-----------------+

Skalierungs-Strategie

Horizontal Auto-Scaling

Jeder Service skaliert unabhängig basierend auf Queue-Länge und Processing-Rate:

  ----------------------------------- ------------------------------------------- ------------------- ------------------- ---------------------------
  **Service-Gruppe**                  **Scale-Trigger**                           **Min Instances**   **Max Instances**   **Scale-Up-Delay**

  Scraper Workers                     Queue \> 100 pending                        1                   5                   30s

  Crawler Workers                     Queue \> 20 pending (ressourcenintensiv!)   1                   3                   60s (Browser braucht RAM)

  AI Workers (Review, Polish, Plan)   Queue \> 30 pending                         1                   8                   30s

  Build Workers                       Queue \> 10 pending                         1                   4                   60s

  QA Workers                          Queue \> 15 pending                         1                   4                   30s

  Outreach Workers                    Rate-Limited durch Instantly.ai             1                   2                   N/A (fixed rate)
  ----------------------------------- ------------------------------------------- ------------------- ------------------- ---------------------------

Throughput-Kalkulation

  ------------ --------------------- ------------------------------- ------------------------------- ------------------------
  **Phase**    **Raw Leads/Woche**   **Full-Pipeline-Leads/Woche**   **Workers nötig**               **Infra-Kosten/Monat**

  MVP          100-250               30-75                           2-3 Worker VMs                  €500-800

  Growth       1.000-2.500           300-750                         4-8 Worker VMs                  €800-1.500

  Scale        5.000-10.000          1.500-3.000                     8-16 Worker VMs                 €1.500-3.000

  Enterprise   10.000+               3.000+                          16+ Worker VMs + Dedicated DB   €3.000+
  ------------ --------------------- ------------------------------- ------------------------------- ------------------------

Bottleneck-Analyse

-   Crawler (Stufe 4): Langsamster Service (\~60-90s pro Lead). Braucht vollen Browser. Max \~40 Leads/Stunde pro Worker. Bei 3.000 Leads/Woche: 3 Crawler-Worker permanent aktiv

-   Claude API (Stufe 4, 5, 8, 9): Throughput-Limit durch API Rate Limits und Token-Budget. Bei 100k Tokens/Lead im Durchschnitt: Claude API Tier-2 (80k tokens/min) reicht für \~50 Leads/Stunde

-   Vercel Deployments (Stufe 12): Vercel Pro erlaubt \~100 Deployments/Tag. Bei 750 Full-Pipeline-Leads/Woche: \~15 Deployments/Tag → kein Problem. Bei Scale (3.000/Woche): Enterprise-Plan nötig oder Alternative (Netlify, Cloudflare Pages)

-   Instantly.ai (Stufe 14): 5.000 Emails/Monat auf Growth Plan. Bei 5 Emails pro Lead + 500 Leads/Monat = 2.500 Emails/Monat → ausreichend. Scale braucht höheren Plan

Monitoring & Observability

Three Pillars: Metrics + Logs + Traces

+-------------+-------------------------------+-------------------------------------------------+---------------------------------+
| **Pillar**  | **Tool**                      | **Was wird getrackt**                           | **Alerting**                    |
+-------------+-------------------------------+-------------------------------------------------+---------------------------------+
| **Metrics** | Prometheus + Grafana          | Pipeline-Throughput pro Step (Leads/Minute)     | Queue \> 100: Scale-Up          |
|             |                               |                                                 |                                 |
|             |                               | Queue-Längen pro Service                        | Error-Rate \> 5%: Alert         |
|             |                               |                                                 |                                 |
|             |                               | Processing-Duration pro Step (P50, P95, P99)    | P95 \> 3x Normal: Alert         |
|             |                               |                                                 |                                 |
|             |                               | Error-Rate pro Step                             | Cost \> 150% Budget: Alert      |
|             |                               |                                                 |                                 |
|             |                               | API-Kosten pro Stunde/Tag/Monat                 |                                 |
|             |                               |                                                 |                                 |
|             |                               | Confidence-Score-Verteilung                     |                                 |
|             |                               |                                                 |                                 |
|             |                               | Conversion Funnel (Leads pro State)             |                                 |
+-------------+-------------------------------+-------------------------------------------------+---------------------------------+
| **Logs**    | Grafana Loki                  | Strukturierte Logs pro Service (JSON)           | Error-Pattern-Detection         |
|             |                               |                                                 |                                 |
|             |                               | Lead-ID in jedem Log-Entry                      |                                 |
|             |                               |                                                 |                                 |
|             |                               | Error Logs mit Stack Traces                     |                                 |
|             |                               |                                                 |                                 |
|             |                               | Audit Trail Queries                             |                                 |
+-------------+-------------------------------+-------------------------------------------------+---------------------------------+
| **Traces**  | OpenTelemetry + Grafana Tempo | End-to-End Trace pro Lead (Scrape → Conversion) | Latency-Anomalien               |
|             |                               |                                                 |                                 |
|             |                               | Latency pro Step sichtbar                       |                                 |
|             |                               |                                                 |                                 |
|             |                               | Cross-Service Correlation                       |                                 |
+-------------+-------------------------------+-------------------------------------------------+---------------------------------+
| **Errors**  | Sentry                        | Uncaught Exceptions                             | Neue Error-Gruppe: Sofort-Alert |
|             |                               |                                                 |                                 |
|             |                               | Error Grouping + Dedup                          |                                 |
|             |                               |                                                 |                                 |
|             |                               | Release Tracking                                |                                 |
|             |                               |                                                 |                                 |
|             |                               | Performance Monitoring                          |                                 |
+-------------+-------------------------------+-------------------------------------------------+---------------------------------+
| **Uptime**  | BetterUptime / Grafana        | Kunden-Websites: HTTP Check alle 5 Min          | Down \> 5 Min: Alert            |
|             |                               |                                                 |                                 |
|             |                               | Preview-Websites: HTTP Check alle 15 Min        |                                 |
|             |                               |                                                 |                                 |
|             |                               | Interne Services: Health-Endpoints              |                                 |
+-------------+-------------------------------+-------------------------------------------------+---------------------------------+

Admin Dashboard: Pipeline-Übersicht

Das interne Admin Dashboard zeigt die Pipeline-Gesundheit in Echtzeit:

-   Pipeline-Funnel: Wie viele Leads sind aktuell in welchem State? Visualisierung als Sankey-Diagramm. Drop-off-Raten zwischen Steps

-   Industry-Vergleich: Performance-Metriken pro Industrie nebeneinander. Welche Pipeline läuft besser? Wo sind Bottlenecks?

-   Lead-Detail-View: Drill-down in jeden einzelnen Lead. Kompletter Audit Trail: Was ist wann passiert? Jede Entscheidung nachvollziehbar

-   Queue-Monitor: Live-Ansicht aller Queues. Länge, Processing-Rate, Oldest-Message. Auto-Scale-Trigger sichtbar

-   Cost-Dashboard: API-Kosten in Echtzeit. Pro Step, pro Industrie, pro Tag. Budget-Verbrauch vs. Limit. Cost-per-Lead und Cost-per-Acquisition

-   Conversion-Tracking: Revenue Dashboard. MRR, neue Kunden, Churn, LTV. Pro Industrie. Korrelation mit Score-Ranges

-   Manual Review Queue: Liste aller Leads die auf manuelles Review warten. Sortiert nach Priorität (Score). One-Click-Approve/Reject

Security & Compliance

+--------------------------------+---------------------------+-------------------------------------------------------------------+
| **Bereich**                    | **Maßnahme**              | **Implementierung**                                               |
+--------------------------------+---------------------------+-------------------------------------------------------------------+
| **Daten-Verschlüsselung**      | At-Rest + In-Transit      | PostgreSQL: Encrypted at Rest (Managed Service)                   |
|                                |                           |                                                                   |
|                                |                           | S3: Server-Side Encryption                                        |
|                                |                           |                                                                   |
|                                |                           | Redis: TLS Connection                                             |
|                                |                           |                                                                   |
|                                |                           | All APIs: HTTPS only                                              |
+--------------------------------+---------------------------+-------------------------------------------------------------------+
| **API-Keys & Secrets**         | Kein Plaintext, kein Git  | Doppler oder Infisical für Secret Management                      |
|                                |                           |                                                                   |
|                                |                           | Environment Variables über CI/CD injected                         |
|                                |                           |                                                                   |
|                                |                           | API-Key-Rotation alle 90 Tage                                     |
+--------------------------------+---------------------------+-------------------------------------------------------------------+
| **Zugangs-Kontrolle**          | Role-Based Access Control | Admin Dashboard: OAuth2 Login (Google Workspace)                  |
|                                |                           |                                                                   |
|                                |                           | Kunden Dashboard: Magic Link + Session Tokens                     |
|                                |                           |                                                                   |
|                                |                           | API: Service-to-Service via API Keys mit Scope-Limits             |
+--------------------------------+---------------------------+-------------------------------------------------------------------+
| **DSGVO-Konformität**          | Privacy by Design         | Nur öffentliche Geschäftsdaten speichern                          |
|                                |                           |                                                                   |
|                                |                           | Opt-out = sofortige Löschung aus allen Tabellen                   |
|                                |                           |                                                                   |
|                                |                           | Daten-Retention: Nicht-konvertierte Leads nach 12 Monaten löschen |
|                                |                           |                                                                   |
|                                |                           | Kein Tracking privater Personen                                   |
+--------------------------------+---------------------------+-------------------------------------------------------------------+
| **Kunden-Domain-Zugangsdaten** | Maximale Sicherheit       | Verschlüsseltes Formular (Frontend-Encryption)                    |
|                                |                           |                                                                   |
|                                |                           | Zugangsdaten nur temporär gespeichert (TTL 72h)                   |
|                                |                           |                                                                   |
|                                |                           | Nach DNS-Switch automatisch gelöscht                              |
|                                |                           |                                                                   |
|                                |                           | Zugriff nur für Onboarding-Service (kein Mensch)                  |
+--------------------------------+---------------------------+-------------------------------------------------------------------+
| **DDoS-Schutz**                | Cloudflare                | Preview-Websites hinter Cloudflare CDN                            |
|                                |                           |                                                                   |
|                                |                           | Rate Limiting auf APIs                                            |
|                                |                           |                                                                   |
|                                |                           | Bot Protection für Pricing Pages                                  |
+--------------------------------+---------------------------+-------------------------------------------------------------------+

Deployment & CI/CD

Deployment-Strategie

  --------------------------- ---------------------------------------------------------- ----------------------------------------------------------- ----------------------------------------------
  **Komponente**              **Deployment-Methode**                                     **Rollback**                                                **Zero-Downtime?**

  Pipeline Workers (Python)   Docker Container → Fly.io/Hetzner. Blue-Green Deployment   Vorherige Container-Version sofort startbar                 Ja (Queue-basiert: neue Worker lesen weiter)

  API Server (Node.js)        Docker → Fly.io. Rolling Update                            Vorherige Version in \<1 Min                                Ja (Rolling)

  Admin Dashboard             Vercel Auto-Deploy on Git Push                             Vercel Instant Rollback                                     Ja (Vercel Atomic)

  Kunden Dashboard            Vercel Auto-Deploy                                         Vercel Instant Rollback                                     Ja

  Database Migrations         Prisma Migrate (Forward-only). Reviewed vor Merge          Nur Forward. Breaking Changes erfordern Feature Flags       Ja (additive migrations)

  Industry Profiles           JSON in DB. Änderung via Admin Dashboard                   Version-History in DB. Rollback = alte Version aktivieren   Ja (sofort wirksam)
  --------------------------- ---------------------------------------------------------- ----------------------------------------------------------- ----------------------------------------------

CI/CD Pipeline

1.  Push to main → GitHub Actions triggered

2.  Lint + Type-Check: ESLint + mypy/pyright

3.  Unit Tests: Jest (Node) + pytest (Python). Min Coverage: 70%

4.  Integration Tests: Key-Flows end-to-end mit Test-Datenbank. Ein Lead komplett durch die Pipeline

5.  Build: Docker-Images für alle Services

6.  Deploy to Staging: Automatisch. Staging-Umgebung spiegelt Production

7.  Smoke Tests on Staging: 5 Test-Leads durch die Pipeline. Alle Steps erfolgreich?

8.  Manual Approval (für Production): Ein Team-Member approved den Deploy

9.  Deploy to Production: Blue-Green Deployment. Health-Checks nach Deploy

10. Post-Deploy-Monitoring: 15 Minuten erhöhte Aufmerksamkeit. Error-Rate-Spike → Auto-Rollback

MVP Technical Plan: Was bauen wir zuerst?

+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **MVP-Philosophie: Functional \> Perfect**                                                                                                                                                                                                         |
|                                                                                                                                                                                                                                                    |
| Der MVP muss FUNKTIONIEREN, nicht perfekt skalieren. Wir brauchen: 50 Leads/Woche scrapen, 10 Previews bauen, 5 Outreach-Sequenzen starten, 1-2 Conversions erzielen. Das validiert das Business-Modell. Optimierung und Skalierung kommen danach. |
+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

MVP: Was ist automatisiert, was manuell?

  ----------------------------- -------------------------------- -------------------------------------------------------------------------------------------------
  **Step**                      **MVP-Status**                   **Begründung**

  1\. Scraping                  **AUTOMATISIERT**                Google Maps API + einfacher Scraper. Kernfunktionalität, relativ einfach

  2\. Validation                **AUTOMATISIERT**                HTTP-Checks + Duplikat-Detection. Einfach zu implementieren

  3\. Enrichment                **SEMI-AUTO**                    Google Places auto. Decision Maker: Impressum-Parse auto, LinkedIn manuell für Top-Leads

  4\. Crawling + Extraction     **AUTOMATISIERT**                Playwright + Claude API. Kernstück der Pipeline

  5\. Quality Review            **AUTOMATISIERT**                Lighthouse auto + Claude Vision auto. Calibration Set manuell erstellt

  6\. Competitor Benchmarking   **SEMI-AUTO**                    Quick-Analyse auto. Feature-Detection auto. Outreach-Argumente manuell reviewed

  7\. Scoring                   **AUTOMATISIERT**                Reine Berechnung. Feedback Loop manuell (monatlich Gewichtungen anpassen)

  8\. Rebuild Planning          **MANUELL REVIEWED**             AI generiert Plan. JEDER Plan wird vor Build manuell geprüft. Zu riskant für Full-Auto im MVP

  9\. Content Polish            **AUTOMATISIERT + SPOT-CHECK**   Texte auto. Fact-Check auto. Stichproben-Review auf jeden 5. Lead

  10\. Build                    **AUTOMATISIERT**                Template-Engine + Content-Injection. Gut testbar

  11\. QA                       **AUTOMATISIERT**                Test-Suite auto. Visual QA auto. Blocker = Pipeline stoppt

  12\. Preview Deploy           **AUTOMATISIERT**                Vercel API. Einfach und zuverlässig

  13\. Outreach Prep            **SEMI-AUTO**                    Email-Texte auto. Analyse-PDF auto. Email-Validation auto. Finaler Check manuell

  14\. Email Outreach           **AUTOMATISIERT**                Instantly.ai Sequenz. Einmal konfiguriert, läuft automatisch

  15\. Payment                  **AUTOMATISIERT**                Stripe Checkout. Standard-Integration

  16\. Onboarding               **MANUELL**                      DNS-Switch im MVP komplett manuell. Zu riskant für Auto. Formular sammelt Infos, Team führt aus

  17\. Post-Launch              **SEMI-AUTO**                    Uptime-Monitoring auto. Customer Success Emails auto. Support manuell
  ----------------------------- -------------------------------- -------------------------------------------------------------------------------------------------

MVP Development Roadmap

+-----------+-----------------------------------+-----------------------------------------------------------------------------------------+
| **Woche** | **Fokus**                         | **Deliverable**                                                                         |
+-----------+-----------------------------------+-----------------------------------------------------------------------------------------+
| **1-2**   | Foundation                        | PostgreSQL Schema deployen                                                              |
|           |                                   |                                                                                         |
|           |                                   | Redis Event Bus aufsetzen                                                               |
|           |                                   |                                                                                         |
|           |                                   | Worker-Pattern implementieren (Template)                                                |
|           |                                   |                                                                                         |
|           |                                   | Industry Profiles für Coaches + Friseure als JSON                                       |
|           |                                   |                                                                                         |
|           |                                   | Admin Dashboard Skeleton (Login + Pipeline View)                                        |
+-----------+-----------------------------------+-----------------------------------------------------------------------------------------+
| **3-4**   | Ingestion Pipeline (Steps 1-3)    | Google Maps Scraper für 2 Industrien                                                    |
|           |                                   |                                                                                         |
|           |                                   | Data Validation Service                                                                 |
|           |                                   |                                                                                         |
|           |                                   | Basic Enrichment (Google Details + Impressum-Parse)                                     |
|           |                                   |                                                                                         |
|           |                                   | 50 Test-Leads scrapen und validieren                                                    |
+-----------+-----------------------------------+-----------------------------------------------------------------------------------------+
| **5-6**   | Analysis Pipeline (Steps 4-7)     | Playwright Crawler mit Content Extraction                                               |
|           |                                   |                                                                                         |
|           |                                   | Quality Review (Lighthouse + Claude Vision)                                             |
|           |                                   |                                                                                         |
|           |                                   | Basic Competitor Analysis                                                               |
|           |                                   |                                                                                         |
|           |                                   | Scoring Engine mit Industry-Weights                                                     |
|           |                                   |                                                                                         |
|           |                                   | Calibration Sets erstellen (30 Websites pro Industrie)                                  |
+-----------+-----------------------------------+-----------------------------------------------------------------------------------------+
| **7-8**   | Generation Pipeline (Steps 8-11)  | Rebuild Planner (Claude API)                                                            |
|           |                                   |                                                                                         |
|           |                                   | Content Polisher mit Banned-Phrase-Check + Fact-Verification                            |
|           |                                   |                                                                                         |
|           |                                   | 2 Templates pro Industrie (coach-landing, coach-authority, salon-gallery, salon-barber) |
|           |                                   |                                                                                         |
|           |                                   | Website Builder (Next.js Template-Engine)                                               |
|           |                                   |                                                                                         |
|           |                                   | QA Test-Suite                                                                           |
+-----------+-----------------------------------+-----------------------------------------------------------------------------------------+
| **9-10**  | Delivery Pipeline (Steps 12-14)   | Vercel Preview Deployment mit Sales-Layer                                               |
|           |                                   |                                                                                         |
|           |                                   | Outreach Preparation (Email-Texte + PDF-Generator)                                      |
|           |                                   |                                                                                         |
|           |                                   | Instantly.ai Integration + 5-Touch-Sequenz                                              |
|           |                                   |                                                                                         |
|           |                                   | Preview Analytics (Plausible + Custom Events)                                           |
|           |                                   |                                                                                         |
|           |                                   | Email-Validierung (NeverBounce)                                                         |
+-----------+-----------------------------------+-----------------------------------------------------------------------------------------+
| **11-12** | Conversion + Polish (Steps 15-17) | Stripe Checkout + Pricing-Page                                                          |
|           |                                   |                                                                                         |
|           |                                   | Manuelles Onboarding-Formular + Prozess                                                 |
|           |                                   |                                                                                         |
|           |                                   | Kunden-Dashboard (Basis: Status + Anfragen)                                             |
|           |                                   |                                                                                         |
|           |                                   | End-to-End-Test: 10 Leads komplett durch Pipeline                                       |
|           |                                   |                                                                                         |
|           |                                   | Pilot-Stadt-Launch: Erste echte Outreach-Kampagne                                       |
+-----------+-----------------------------------+-----------------------------------------------------------------------------------------+

Nächste Schritte

1.  Repository-Setup: Monorepo mit Turborepo. /packages für Shared Code, /services für Pipeline-Workers, /apps für Dashboards

2.  Database-Schema deployen: Prisma Schema definieren, Migration auf Supabase/Neon

3.  Event Bus implementieren: Redis Streams + BullMQ. Worker-Pattern als Template. Ein Test-Service der Events produziert und konsumiert

4.  Google Maps Scraper als erster Service: Coaches + Friseure in Pilot-Stadt. 50 Leads scrapen und in DB speichern

5.  Playwright Crawler als zweiter Service: Top-10 gescrapte Leads crawlen. Content extrahieren. Screenshots speichern

6.  Claude API Integration: Extraction-Prompt für Coaches + Friseure entwickeln und testen

7.  Erstes Template bauen: coach-landing als Figma → Next.js + Tailwind. Manuell mit Test-Content befüllen

8.  End-to-End-Proof: 1 Lead manuell von Scraping bis Preview durchschleusen. Jeder Step einmal funktional

*Webolution Technical Architecture -- SocialCooks*
