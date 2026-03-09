# Webolution Event Catalog

## Architecture

- **Transport**: Redis Streams via BullMQ
- **Pattern**: Each service subscribes to input events and emits output events
- **Guarantee**: At-least-once delivery. Services must be idempotent.
- **Parallelism**: Steps 5+6 run in parallel (both consume `crawl.completed`)

## Event Definitions

### Pipeline Events (Sequential Flow)

| Event | Emitter (Step) | Consumer(s) | Payload |
|-------|---------------|-------------|---------|
| `lead.scraped` | 1: Scraper | 2: Validator | `{leadId, firmName, url, source, industryTag, region}` |
| `lead.validated` | 2: Validator | 3: Enricher | `{leadId, validationConfidence, canonicalUrl}` |
| `lead.enriched` | 3: Enricher | 4: Crawler | `{leadId, enrichmentConfidence, businessHealthScore, decisionMaker{name, email}}` |
| `lead.disqualified` | 2 or 3 | Orchestrator (log) | `{leadId, reason, step}` |
| `crawl.completed` | 4: Crawler | **5: Reviewer + 6: Competitor** (parallel!) | `{leadId, contentPackageId, extractionConfidence, pageCount}` |
| `review.completed` | 5: Reviewer | 7: Scorer (waits for both 5+6) | `{leadId, qualityReportId, overallScore, weaknessCount, reviewConfidence}` |
| `competitor.completed` | 6: Competitor | 7: Scorer | `{leadId, competitorReportId, competitivePressureScore, relativePosition}` |
| `lead.scored` | 7: Scorer | Orchestrator (routing) | `{leadId, finalScore, category, routingDecision, scoreConfidence}` |
| `rebuild.planned` | 8: Planner | 9: Polisher | `{leadId, rebuildPlanId, templateId, rebuildConfidence}` |
| `content.polished` | 9: Polisher | 10: Builder | `{leadId, polishedContentId, polishConfidence, bannedPhraseHits}` |
| `website.built` | 10: Builder | 11: QA | `{leadId, buildId, lighthousePerformance, lighthouseA11y, lighthouseSeo}` |
| `qa.passed` | 11: QA | 12: Deployer | `{leadId, qaReportId, verdict}` |
| `qa.failed` | 11: QA | Orchestrator (retry/escalate) | `{leadId, qaReportId, blockerCount, autoFixable}` |
| `preview.deployed` | 12: Deployer | 13: Outreach Prep | `{leadId, previewUrl, deployTimestamp}` |
| `outreach.prepared` | 13: Preparer | 14: Sender | `{leadId, outreachPackageId, channel, emailValidated}` |
| `email.sent` | 14: Sender | Orchestrator (tracking) | `{leadId, emailId, touchNumber, status}` |
| `payment.completed` | 15: Payment (Stripe webhook) | 16: Onboarding | `{leadId, customerId, package, amount}` |
| `onboarding.completed` | 16: Onboarding | 17: Customer Success | `{leadId, customerId, domain, goLiveTimestamp}` |

### Behavioral Events (Trigger Branching)

| Event | Source | Consumer | Payload | Action Triggered |
|-------|--------|----------|---------|-----------------|
| `email.opened` | Instantly webhook | Orchestrator | `{leadId, emailId, timestamp}` | Update lead state. If touch 1: route to engagement follow-up (touch 2a) |
| `email.bounced` | Instantly webhook | Orchestrator | `{leadId, emailId, reason}` | Blacklist email. Activate secondary channel |
| `email.replied` | Instantly webhook | Orchestrator | `{leadId, emailId}` | PAUSE sequence immediately. Alert sales team |
| `email.opted_out` | Instantly webhook | Orchestrator | `{leadId}` | STOP sequence. Add to suppression list. Never contact again |
| `preview.visited` | Plausible webhook | Orchestrator | `{leadId, visitCount, scrollDepth, duration}` | Update score (+5). If 3x visited: trigger phone outreach |
| `preview.cta_clicked` | Plausible webhook | Orchestrator | `{leadId, timestamp}` | Update score (+15). If no payment in 1h: trigger abandoned cart email |
| `preview.comparison_clicked` | Plausible webhook | Orchestrator | `{leadId}` | Update score (+8). High-interest signal for follow-up |
| `preview.calendly_clicked` | Plausible webhook | Orchestrator | `{leadId}` | PAUSE automated sequence. Sales takes over |

### System Events

| Event | Source | Consumer | Purpose |
|-------|--------|----------|---------|
| `lead.retry` | Orchestrator | Original failed service | Retry a failed step with backoff |
| `lead.escalate` | Orchestrator | Admin Dashboard | Route to manual review queue |
| `lead.nurture` | Scorer (cool leads) | Sender (nurture track) | Send analysis-only email, no preview |
| `score.stale` | Cron job (daily) | Scorer | Re-score leads with scores older than 4 weeks |
| `profile.updated` | Admin Dashboard | All services (cache invalidation) | Industry profile config changed |

## BullMQ Queue Configuration

```typescript
// packages/queue/src/queues.ts

export const QUEUES = {
  // Pipeline queues (one per step)
  SCRAPE: 'pipeline:scrape',
  VALIDATE: 'pipeline:validate',
  ENRICH: 'pipeline:enrich',
  CRAWL: 'pipeline:crawl',
  REVIEW: 'pipeline:review',
  COMPETITOR: 'pipeline:competitor',
  SCORE: 'pipeline:score',
  PLAN: 'pipeline:plan',
  POLISH: 'pipeline:polish',
  BUILD: 'pipeline:build',
  QA: 'pipeline:qa',
  DEPLOY: 'pipeline:deploy',
  OUTREACH_PREP: 'pipeline:outreach-prep',
  OUTREACH_SEND: 'pipeline:outreach-send',
  PAYMENT: 'pipeline:payment',
  ONBOARD: 'pipeline:onboard',
  CUSTOMER_SUCCESS: 'pipeline:customer-success',

  // System queues
  MANUAL_REVIEW: 'system:manual-review',
  RETRY: 'system:retry',
  NURTURE: 'system:nurture',
} as const

// Default job options
export const DEFAULT_JOB_OPTIONS = {
  attempts: 3,
  backoff: { type: 'exponential', delay: 10_000 }, // 10s, 20s, 40s
  removeOnComplete: { count: 1000 },
  removeOnFail: { count: 5000 },
}
```

## Orchestrator: Parallel Step Handling

Steps 5 (Review) and 6 (Competitor) run in parallel after Step 4 (Crawl). The Orchestrator tracks completion of both before triggering Step 7 (Scoring):

```typescript
// Simplified orchestrator logic for parallel steps
async function handleCrawlCompleted(event: CrawlCompletedEvent) {
  // Dispatch to BOTH review and competitor queues
  await reviewQueue.add('review', { leadId: event.leadId })
  await competitorQueue.add('benchmark', { leadId: event.leadId })

  // Set tracking flags
  await redis.hset(`lead:${event.leadId}:parallel`, {
    reviewDone: 'false',
    competitorDone: 'false',
  })
}

async function handleReviewCompleted(event: ReviewCompletedEvent) {
  await redis.hset(`lead:${event.leadId}:parallel`, 'reviewDone', 'true')
  await checkAndTriggerScoring(event.leadId)
}

async function handleCompetitorCompleted(event: CompetitorCompletedEvent) {
  await redis.hset(`lead:${event.leadId}:parallel`, 'competitorDone', 'true')
  await checkAndTriggerScoring(event.leadId)
}

async function checkAndTriggerScoring(leadId: string) {
  const status = await redis.hgetall(`lead:${leadId}:parallel`)
  if (status.reviewDone === 'true' && status.competitorDone === 'true') {
    await scoreQueue.add('score', { leadId })
    await redis.del(`lead:${leadId}:parallel`)
  }
}
```
