# Webolution – Setup Guide for Claude Code

## Was du hast

6 .docx Spezifikations-Dokumente (in `docs/docx/`):
- Deep Spec Steps 1-4
- Deep Spec Steps 5-8
- Deep Spec Steps 9-12
- Deep Spec Steps 13-17
- Industry Profile Ranking
- Technical Architecture

## Was Claude Code zusätzlich braucht

Claude Code kann keine .docx lesen. Deshalb sind 4 Markdown-Referenzdokumente vorbereitet:

| Datei | Inhalt | Wann konsultieren |
|-------|--------|-------------------|
| `CLAUDE.md` | **Projekt-Instruktionen** – das wichtigste File. Claude Code liest das automatisch | Immer (automatisch) |
| `docs/SPEC.md` | Kondensierte 17-Step-Pipeline-Spec | Beim Implementieren jedes Steps |
| `docs/DATABASE.md` | Komplettes Prisma Schema (copy-paste ready) | Beim DB-Setup |
| `docs/EVENTS.md` | Event-Katalog + BullMQ Queue-Definitionen | Beim Queue/Worker-Setup |
| `docs/INDUSTRY-PROFILES.md` | Coaches + Friseure Profile als JSON | Beim Implementieren industrie-spezifischer Logik |

## Setup-Schritte

### 1. Repository anlegen

```bash
mkdir webolution
cd webolution
git init
```

### 2. Diese Dateien reinkopieren

```
webolution/
├── CLAUDE.md                     ← Projekt-Instruktionen (wird automatisch gelesen)
├── docs/
│   ├── SPEC.md                   ← Pipeline-Spec (kondensiert, Quick Reference)
│   ├── DATABASE.md               ← DB Prisma Schema (copy-paste ready)
│   ├── EVENTS.md                 ← Event-Katalog + BullMQ Queues
│   ├── INDUSTRY-PROFILES.md      ← Coaches + Friseure JSON Configs
│   ├── SETUP-GUIDE.md            ← Diese Datei
│   ├── STEPS-1-4.md              ← VOLLSTÄNDIG: Scraping, Validation, Enrichment, Crawl
│   ├── STEPS-5-8.md              ← VOLLSTÄNDIG: Review, Competitor, Scoring, Rebuild Plan
│   ├── STEPS-9-12.md             ← VOLLSTÄNDIG: Polish, Build, QA, Preview Deploy
│   ├── STEPS-13-17.md            ← VOLLSTÄNDIG: Outreach, Payment, Onboarding, Post-Launch
│   ├── INDUSTRY-RANKING-FULL.md  ← VOLLSTÄNDIG: 10-Dimensionen-Ranking, Pilot-Profile
│   ├── TECH-ARCHITECTURE.md      ← VOLLSTÄNDIG: Services, Events, DB, Infra, Costs, MVP
│   └── docx/                     ← Original .docx (Backup)
```

**Wichtig**: Die `STEPS-*.md`, `INDUSTRY-RANKING-FULL.md` und `TECH-ARCHITECTURE.md` sind 1:1 aus den .docx konvertiert (31.700 Wörter). `SPEC.md` ist eine kondensierte Quick-Reference (1.800 Wörter). Beim Implementieren eines Steps immer die VOLLSTÄNDIGE Datei referenzieren.

### 3. Claude Code starten

```bash
claude
```

### 4. Erster Befehl an Claude Code

```
Lies CLAUDE.md und docs/DATABASE.md. Dann setze das Monorepo auf:
Turborepo + pnpm workspaces, packages/db mit dem Prisma Schema aus DATABASE.md,
packages/shared mit TypeScript types, packages/queue mit BullMQ base worker.
Folge exakt die Projekt-Struktur aus CLAUDE.md.
```

## Empfohlene Reihenfolge für Claude Code Befehle

### Woche 1-2: Foundation

```
1. "Setze das Monorepo auf: Turborepo + pnpm + TypeScript. Root package.json, turbo.json, .gitignore, tsconfig.json. Struktur wie in CLAUDE.md beschrieben."

2. "Erstelle packages/db mit dem Prisma Schema aus docs/DATABASE.md. Generiere den Prisma Client. Erstelle die erste Migration."

3. "Erstelle packages/shared mit TypeScript types für Lead, ContentPackage, QualityReport, CompetitorReport, RebuildPlan, PolishedContent, Build, QaReport. Leite die Types aus dem Prisma Schema ab."

4. "Erstelle packages/shared/src/events mit allen Event-Type-Definitionen aus docs/EVENTS.md. EventType enum + Payload interfaces."

5. "Erstelle packages/queue mit BullMQ setup. Queue-Definitionen aus EVENTS.md. BaseWorker abstract class mit: idempotency check, industry profile loading, confidence-based routing, error handling (RetryableError, PermanentError, ConfidenceError), audit logging, cost tracking."

6. "Erstelle ein minimales Admin Dashboard Skeleton in apps/admin: Next.js App Router + Tailwind + shadcn/ui. Login-Page + Pipeline-Overview-Page (leer, aber Routing steht)."
```

### Woche 3-4: Erste Services

```
7. "Erstelle services/scraper: Google Places API Integration. Liest Industry Profile aus DB. Sucht nach searchTerms in geo_radius. Speichert Raw Leads in DB. Emittiert lead.scraped Event. Lies docs/SPEC.md Step 1 für Details."

8. "Erstelle services/validator: Konsumiert lead.scraped Events. 6 sequentielle Checks (URL format, reachability, duplicate, blacklist, competitor exclusion, normalization). Emittiert lead.validated oder lead.disqualified. Lies docs/SPEC.md Step 2."

9. "Erstelle services/enricher: Konsumiert lead.validated. Google Places Details API call. Impressum quick-scrape (Playwright). Business Health Score berechnung. Decision Maker extraction. Emittiert lead.enriched oder lead.disqualified. Lies docs/SPEC.md Step 3."
```

### Woche 5+: Weiter nach Build Order in CLAUDE.md

## Tipps für Claude Code

1. **Immer auf CLAUDE.md verweisen**: "Lies CLAUDE.md" als Prefix wenn Claude Code den Kontext verliert

2. **Einen Service nach dem anderen**: Nicht alles auf einmal. Ein Service fertig → testen → nächster

3. **Spec referenzieren**: "Lies docs/SPEC.md Step 4" wenn du einen spezifischen Pipeline-Step implementierst

4. **Database Schema nicht manuell editieren**: Immer Prisma Schema ändern, dann `prisma migrate dev`

5. **Test early**: Nach jedem Service: "Schreibe einen Test der einen Fake-Lead durch diesen Service schickt"

## Environment Setup

Bevor du startest, brauchst du diese Accounts/Keys:

| Service | Wofür | Priorität |
|---------|-------|-----------|
| Anthropic API Key | Claude API für AI-Schritte | SOFORT (ohne geht nichts in Step 4+) |
| Google Cloud + Places API Key | Scraping + Enrichment | SOFORT (Step 1+3) |
| Supabase/Neon Account | PostgreSQL Database | SOFORT |
| Upstash Account | Redis für Queues | SOFORT |
| Vercel Account + Domain | Preview Hosting | Ab Woche 9 |
| Cloudflare R2 | Object Storage (Screenshots, Bilder) | Ab Woche 5 |
| Instantly.ai Account | Email Outreach | Ab Woche 9 |
| Stripe Account | Payment | Ab Woche 11 |
| NeverBounce/ZeroBounce | Email Validation | Ab Woche 9 |
| Sentry Account | Error Tracking | Nice-to-have (free tier) |
| NanoBanana API | Image Generation | Ab Woche 7 |
