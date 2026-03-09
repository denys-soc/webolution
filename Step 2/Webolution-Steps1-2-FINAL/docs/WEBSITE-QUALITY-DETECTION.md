# WEBOLUTION – Website-Schwäche-Erkennung

> **Die fundamentalste Frage im gesamten Projekt:**
> Wie erkennen wir in 5 Sekunden und für <€0.01, dass eine Website schlecht genug ist,
> dass wir dafür eine neue bauen und verkaufen können?
>
> Dieses Dokument definiert:
> 1. Was "schlecht" konkret bedeutet (messbar, nicht subjektiv)
> 2. Wie wir das ultra-schnell und ultra-günstig erkennen
> 3. Warum ein Lead KAUFEN würde (nicht nur "Website ist schlecht" sondern "Lead HAT EIN PROBLEM")

---

## Das eigentliche Business: Wir verkaufen keine Websites. Wir lösen Probleme.

Ein Lead kauft nicht weil seine Website "schlecht" ist. Er kauft weil seine schlechte Website ihm SCHADET:

- Er verliert Kunden an die Konkurrenz (die eine bessere Website hat)
- Er bekommt keine Anfragen über seine Website (kein CTA, kein Formular, nicht auffindbar)
- Er schämt sich wenn er jemandem seine URL gibt (sieht unprofessionell aus)
- Patienten/Mandanten/Klienten suchen online und finden ihn nicht oder werden abgeschreckt
- Er zahlt einem Webdesigner Geld und die Seite ist trotzdem veraltet

**Unsere Pipeline muss nicht "schlechte Websites" finden. Sie muss "Geschäftsinhaber mit einem Online-Präsenz-Problem" finden.**

Das verändert was wir suchen:

| Falsche Frage | Richtige Frage |
|---------------|---------------|
| "Ist die Website hässlich?" | "Verliert der Inhaber Kunden wegen seiner Website?" |
| "Hat die Website einen niedrigen Lighthouse-Score?" | "Können Besucher auf dem Handy einen Termin buchen?" |
| "Ist das Design veraltet?" | "Würde ein potenzieller Kunde nach 3 Sekunden auf dieser Website bleiben oder wegklicken?" |
| "Fehlen Meta-Tags?" | "Findet jemand der 'Anwalt München Familienrecht' googelt diese Kanzlei?" |
| "Ist die Ladezeit >5 Sekunden?" | "Springt ein ungeduldiger mobiler Besucher ab bevor die Seite geladen hat?" |

---

## Die 5 Schwäche-Kategorien (was "schlecht" konkret bedeutet)

### Kategorie 1: UNSICHTBAR (Lead wird online nicht gefunden)

**Das Problem:** Potenzielle Kunden suchen bei Google, finden aber die Konkurrenz statt diesen Lead.

**Indikatoren:**
- Kein SSL (http statt https) → Google bestraft das seit 2018 im Ranking
- Kein Schema.org Markup → Google versteht nicht WAS das Business macht
- Keine Meta-Tags (title, description) → Google zeigt generischen Text in Suchergebnissen
- Keine lokale SEO-Optimierung → "Friseur München" findet diese Seite nicht
- Seite nicht in Google indexiert (site:domain.de zeigt 0 Ergebnisse)
- Kein Google Business Profil oder Profil nicht mit Website verknüpft
- Keine Sitemap.xml → Google crawlt die Seite ineffizient

**Erkennungsgeschwindigkeit:** 2-3 Sekunden (automatische HTTP-Checks + DNS-Lookup)
**Kosten:** €0.00 (eigene Infrastruktur)
**Outreach-Argument:** "Wenn potenzielle [Kunden/Mandanten/Patienten] in [Stadt] nach [Branche] suchen, finden sie Ihre Konkurrenz – nicht Sie."

### Kategorie 2: ABSCHRECKEND (Besucher kommen, aber gehen sofort wieder)

**Das Problem:** Die Website macht in den ersten 3 Sekunden einen schlechten Eindruck. Besucher schließen den Tab.

**Indikatoren:**
- Design sieht aus wie vor 2015 (Erkennung: Copyright-Jahr im Footer, CSS-Analyse, AI-Vision Quick-Check)
- Nicht mobil-optimiert (kein Viewport Meta-Tag, oder Lighthouse Mobile <50)
- Ladezeit >5 Sekunden (Speed-Check)
- Unklare Navigation (>7 Menüpunkte, verschachtelte Dropdowns)
- Kein Hero-Bereich / kein klarer Ersteindruck above-the-fold
- Schlechte Typografie (zu kleine Schrift <14px, Comic Sans, zu viele verschiedene Fonts)
- Überladenes Layout (zu viele Farben, zu viele Elemente, kein Whitespace)
- Stock-Fotos die offensichtlich generisch sind (Handschlag-Foto, Team-High-Five)
- Auto-Play Musik oder Video (ja, das gibt es noch bei KMU-Seiten)
- Flash-Inhalte oder "Diese Seite benötigt Flash Player"
- Pop-ups die den Content verdecken
- Cookie-Banner der 50% des Screens einnimmt

**Erkennungsgeschwindigkeit:** 10-15 Sekunden (Screenshot + Lighthouse Quick-Audit + Viewport-Check)
**Kosten:** €0.001-0.005 (1 PageSpeed API Call + 1 Screenshot)
**Outreach-Argument:** "67% Ihrer Besucher kommen vom Handy – und sehen eine Website die auf dem Smartphone nicht funktioniert."

### Kategorie 3: PASSIV (Besucher bleiben, aber tun nichts)

**Das Problem:** Die Website zeigt Informationen, aber konvertiert nicht. Kein Besucher wird zum Kunden.

**Indikatoren:**
- Kein klarer Call-to-Action (kein "Termin buchen", kein "Jetzt anfragen", kein "Kontakt")
- CTA existiert aber ist versteckt (nur auf Kontakt-Seite, nicht auf Homepage)
- Kein Kontaktformular (nur E-Mail-Adresse als mailto: Link)
- Keine Online-Buchungsmöglichkeit (bei Branchen wo Buchung erwartet wird: Friseure, Ärzte, Coaches)
- Keine Telefonnummer sichtbar (oder nur im Impressum)
- Telefonnummer nicht klickbar auf Mobile (kein tel: Link)
- Keine Öffnungszeiten auf der Startseite (Besucher weiß nicht ob gerade offen)
- Keine Preistransparenz (Besucher weiß nicht was es kostet → geht zur Konkurrenz die Preise zeigt)
- Keine Wegbeschreibung / kein eingebetteter Maps (Besucher muss Adresse manuell suchen)

**Erkennungsgeschwindigkeit:** 15-30 Sekunden (Quick-Crawl der Homepage + AI-Quick-Check auf CTA-Existenz)
**Kosten:** €0.005-0.01 (Homepage-Crawl + Mini-AI-Check)
**Outreach-Argument:** "Ihre Website hat keinen Weg für Besucher, direkt einen Termin zu vereinbaren. [Konkurrent] hat Online-Buchung – und fängt Ihre Kunden ab."

### Kategorie 4: VERTRAUENSLOS (Besucher vertrauen nicht genug um zu handeln)

**Das Problem:** Die Website sieht ok aus, aber nichts überzeugt den Besucher dass dies die richtige Wahl ist.

**Indikatoren:**
- Keine Testimonials / Kundenstimmen
- Google-Bewertungen nicht auf der Website eingebunden (obwohl 4.5★ mit 30 Reviews)
- Keine Team-Vorstellung (wer sind die Menschen hinter dem Business?)
- Keine Fotos von echten Mitarbeitern (nur Stock oder gar keine)
- Keine Zertifizierungen / Auszeichnungen sichtbar
- Kein "Über uns" / kein persönlicher Background
- Kein "Seit [Jahr]" oder anderes Erfahrungs-Signal
- Keine Referenzen / Fallbeispiele / Portfolio
- Impressum unvollständig oder fehlend (wirkt unseriös)

**Erkennungsgeschwindigkeit:** 30-60 Sekunden (Quick-Crawl Homepage + 2-3 Unterseiten + AI-Check auf Trust-Elemente)
**Kosten:** €0.01-0.03 (Crawl + AI-Quick-Analyse)
**Outreach-Argument:** "Sie haben 4.6 Sterne bei Google mit 28 Bewertungen – aber Ihre Website zeigt das nicht. Potenzielle Kunden die Ihre Website besuchen, sehen diese Bewertungen nie."

### Kategorie 5: VERALTET (funktioniert, aber wirkt nicht zeitgemäß)

**Das Problem:** Die Website ist nicht kaputt, aber sie signalisiert: "Dieses Business investiert nicht in seinen Auftritt." Kunden die Qualität erwarten werden skeptisch.

**Indikatoren:**
- Copyright im Footer zeigt altes Jahr (© 2019 oder älter)
- Letzter Blog-Post / News-Eintrag >2 Jahre alt
- Design-Ästhetik einer vergangenen Ära (Glossy Buttons, Schatten auf allem, Slider-Karussells, Parallax-Overkill)
- Baukasten-System erkennbar (Jimdo Badge im Footer, Wix Branding, WordPress.com Subdomain)
- "Under Construction" oder "Coming Soon" Sections die seit Jahren da stehen
- Broken Links die nie gefixt wurden
- Veraltete Informationen (ehemalige Mitarbeiter, alte Öffnungszeiten, abgelaufene Angebote)
- CMS-Version veraltet (erkennbar an Meta-Generator-Tag: "WordPress 4.9")
- jQuery UI Datepicker, alte Bootstrap-Version (sichtbar im Source)

**Erkennungsgeschwindigkeit:** 5-15 Sekunden (Footer-Check + Meta-Generator + AI-Vision Quick-Check)
**Kosten:** €0.001-0.005
**Outreach-Argument:** "Ihre Website wurde offenbar seit [Jahr] nicht mehr aktualisiert. In der Zwischenzeit hat sich die Erwartung Ihrer Kunden an einen Online-Auftritt grundlegend verändert."

---

## Der Quick-Quality-Check: Neue Step 1.5 (zwischen Scraping und Validation)

### Warum ein eigener Step?

Aktuell: Scrape → Validate → Enrich (€0.05-0.15) → Crawl (€0.08-0.15) → Review (€0.04-0.08). 
Bis wir wissen ob die Website schlecht ist, haben wir €0.17-0.38 pro Lead ausgegeben.
Bei 200 gescrapten Leads von denen 60% eigentlich OK-Websites haben: 120 × €0.25 = €30 verschwendet.

Besser: Scrape → **Quick-Quality-Check (€0.005)** → nur schlechte Websites in die Pipeline.
120 Leads rausgefiltert × €0.005 = €0.60 statt €30.

### Quick-Quality-Check: Was er tut

In <15 Sekunden und für <€0.01 pro Lead:

**Schritt 1: Technische Schnellprüfung (2 Sekunden, €0.00)**
- HTTP HEAD Request: Status Code, Redirect-Chain, HTTPS?
- Response Headers: Server-Type (Apache/Nginx/IIS), X-Powered-By (WordPress/Joomla/Wix)
- HTML HEAD Parse: viewport meta (mobile-ready?), generator meta (CMS + Version), title tag (vorhanden + Qualität?)
- Footer Copyright Jahr (Regex auf © YYYY)
- Result: technical_signals{}

**Schritt 2: Lighthouse Quick-Audit (5 Sekunden, €0.003)**
- PageSpeed Insights API: Nur Mobile Strategy, nur Performance + SEO Kategorien
- Extrahiert: Performance Score, LCP, CLS, Mobile-Friendly, SEO Score
- NICHT den vollständigen Lighthouse-Report – nur die Quick-Metriken
- Result: lighthouse_quick{}

**Schritt 3: Screenshot + AI-Vision-Quickcheck (8 Sekunden, €0.005)**
- Playwright: Screenshot Homepage Desktop (1440px) + Mobile (375px) – nur Homepage, keine Unterseiten
- Claude Vision (Haiku – günstigstes Modell): 
  - "Look at this website screenshot. In ONE JSON object, answer:
    design_era: (pre-2015 / 2015-2019 / 2020-2023 / 2024+)
    professional_impression: (1-10)
    has_hero_section: (true/false)
    has_visible_cta: (true/false)
    has_stock_photos: (true/false)
    has_real_photos: (true/false)
    is_mobile_usable: (true/false)
    biggest_weakness: (one sentence)"
- Result: visual_quick{}

**Schritt 4: Quick-Score-Berechnung (0.1 Sekunden, €0.00)**

```
quick_score = 0  (0-100, höher = SCHLECHTERE Website = BESSERER Lead für uns)

# Technische Signale (max 25 Punkte)
if no_ssl: quick_score += 10
if no_viewport_meta: quick_score += 8
if copyright_year < 2022: quick_score += 5
if copyright_year < 2019: quick_score += 7  (additional)
if cms_version_outdated: quick_score += 5

# Lighthouse Signale (max 25 Punkte)
if lighthouse_performance < 30: quick_score += 10
elif lighthouse_performance < 50: quick_score += 6
elif lighthouse_performance < 70: quick_score += 3
if lighthouse_mobile_unfriendly: quick_score += 10
if lighthouse_seo < 50: quick_score += 5

# Visuelle Signale (max 30 Punkte)
if design_era == "pre-2015": quick_score += 15
elif design_era == "2015-2019": quick_score += 10
elif design_era == "2020-2023": quick_score += 3
if professional_impression < 4: quick_score += 10
elif professional_impression < 6: quick_score += 5
if not has_hero_section: quick_score += 5

# Conversion Signale (max 20 Punkte)
if not has_visible_cta: quick_score += 10
if not is_mobile_usable: quick_score += 10
```

**Routing basierend auf Quick-Score:**

| Quick-Score | Bedeutung | Aktion |
|-------------|-----------|--------|
| 70-100 | Website ist KLAR schlecht. Offensichtlicher Kandidat | → Sofort in Pipeline (Gate 1). Höchste Priorität |
| 50-69 | Website hat deutliche Schwächen. Wahrscheinlicher Kandidat | → In Pipeline (Gate 1). Normale Priorität |
| 30-49 | Website ist mittelmäßig. Verbesserbar aber kein dramatischer Uplift möglich | → Nur wenn Kapazität. Niedrige Priorität |
| 0-29 | Website ist ok bis gut. Kein überzeugender Vorher/Nachher möglich | → NICHT in Pipeline. Sofort archivieren. Kein Geld ausgeben |

### Was der Quick-Check NICHT tut (und warum)

- Keine vollständige Crawl (das kommt in Step 4 – zu teuer für einen Filter)
- Keine Content-Analyse (Texte lesen dauert zu lang)
- Keine Competitor-Analyse (irrelevant für die Filter-Entscheidung)
- Kein Enrichment (wir wollen noch kein Geld ausgeben bevor wir wissen ob sich's lohnt)
- Kein Scoring der Lead-Qualität (das kommt in Step 7 – hier geht es NUR um die Website-Qualität)

### Kostenersparnis

Annahme: 200 Leads gescraped pro Kampagne.
- Ohne Quick-Check: 200 × ~€0.25 (Enrichment + Crawl + Review) = €50 bevor wir wissen welche schlecht sind
- Mit Quick-Check: 200 × €0.005 (Quick-Check) = €1.00. Davon 80 Leads mit Score >50 → 80 × €0.25 = €20 für Pipeline
- **Ersparnis: ~€29 pro Kampagne (58%)**

Bei 10 Kampagnen/Monat: €290/Monat gespart. Das bezahlt fast die halbe Infrastruktur.

---

## Schwäche-Gewichtung pro Industrie

Nicht jede Schwäche ist für jede Industrie gleich relevant:

| Schwäche | Coaches | Friseure | Anwälte | Ärzte |
|----------|---------|----------|---------|-------|
| Nicht mobil-optimiert | KRITISCH (Klienten suchen am Handy) | KRITISCH (Besucher suchen unterwegs) | WICHTIG | KRITISCH (Patienten suchen am Handy) |
| Kein CTA / keine Buchung | KRITISCH (kein Weg zum Kennenlerngespräch) | KRITISCH (kein Weg zum Termin) | WICHTIG (Erstberatung) | KRITISCH (Termin buchen) |
| Keine Testimonials / Reviews | SEHR WICHTIG (Vertrauen ist alles bei Coaches) | WICHTIG (Reviews helfen) | KRITISCH (Mandanten brauchen Vertrauen) | WICHTIG |
| Design veraltet | KRITISCH (Coach der modern sein will mit alter Website = Widerspruch) | SEHR WICHTIG (visuelles Business) | WICHTIG (seriös aber nicht zwingend modern) | MITTEL (Inhalt wichtiger als Design) |
| Keine Preistransparenz | WICHTIG | SEHR WICHTIG (Preise sind Entscheidungskriterium Nr.1) | MITTEL (Erstberatung meist kostenlos) | GERING (Kassenleistung vs. Privat relevant) |
| Kein Team / keine Fotos | MITTEL (Solo-Coach = nur 1 Person nötig) | SEHR WICHTIG (wer schneidet mir die Haare?) | WICHTIG (welcher Anwalt betreut meinen Fall?) | KRITISCH (welcher Arzt behandelt mich?) |
| Langsame Ladezeit | WICHTIG | WICHTIG | MITTEL | WICHTIG |
| Kein Google Maps / Standort | GERING (viele Coaches remote) | KRITISCH (Besucher wollen Standort) | WICHTIG | KRITISCH (Patienten kommen vor Ort) |
| Kein SSL/HTTPS | KRITISCH (für alle gleich – Google-Ranking + Vertrauen) | KRITISCH | KRITISCH | KRITISCH |

Die Quick-Score-Formel muss diese Gewichtungen aus dem Industry Profile laden.

---

## Die 10 stärksten Verkaufsargumente (abgeleitet aus Schwächen)

Jede erkannte Schwäche wird zu einem konkreten Verkaufsargument im Outreach:

**1. "Ihre Besucher brechen ab bevor die Seite geladen hat"**
Trigger: Ladezeit >5s. Argument: "Ihre Website braucht [X] Sekunden zum Laden. 53% der mobilen Besucher verlassen eine Seite die länger als 3 Sekunden lädt (Google-Studie). Ihre neue Website lädt in unter 2 Sekunden."

**2. "70% Ihrer Besucher sehen eine unbenutzbare Version"**
Trigger: Nicht mobil-optimiert. Argument: "67-73% aller Website-Besuche kommen heute vom Smartphone. Ihre Website ist nicht für Mobilgeräte optimiert – das bedeutet [Zahl]% Ihrer potenziellen Kunden sehen eine nicht nutzbare Version."

**3. "Ihre Konkurrenz sieht professioneller aus"**
Trigger: Design-Era pre-2019 + Konkurrent mit moderner Website. Argument: "Im Vergleich zu anderen [Branche] in [Stadt] wirkt Ihr Online-Auftritt aus einer anderen Generation. Der erste Eindruck entscheidet – in 0.05 Sekunden."

**4. "Kein Besucher kann direkt anfragen"**
Trigger: Kein CTA / kein Formular / kein Booking. Argument: "Ihre Website hat keinen einfachen Weg für Besucher, Sie zu kontaktieren oder einen Termin zu buchen. Jeder Besucher der nicht SOFORT handeln kann, ist ein verlorener Kunde."

**5. "Ihre 4.8 Sterne sieht niemand"**
Trigger: Google Rating >4.0 aber nicht auf Website. Argument: "Sie haben [Rating] Sterne bei [Anzahl] Google-Bewertungen – fantastisch! Aber Ihre Website zeigt das nicht. Besucher die direkt auf Ihre Seite kommen, sehen diese Bewertungen nie."

**6. "Ihre Website sagt nichts über SIE"**
Trigger: Keine About-Section / keine Team-Fotos / keine Persönlichkeit. Argument: "Ihr Online-Auftritt zeigt nicht wer Sie sind und warum jemand ausgerechnet zu Ihnen kommen sollte. Potenzielle [Kunden] wollen wissen mit wem sie es zu tun haben."

**7. "Google findet Sie nicht"**
Trigger: Keine Meta-Tags, kein Schema, kein SSL. Argument: "Wenn jemand '[Branche] [Stadt]' googelt, hat Ihre Website kaum eine Chance auf der ersten Seite zu erscheinen. [Anzahl] technische Probleme verhindern, dass Google Ihre Seite richtig einordnet."

**8. "Ihre Preise sind ein Geheimnis"**
Trigger: Keine Preise auf der Website. Argument: "Potenzielle Kunden wollen wissen was etwas kostet bevor sie anrufen. Wenn Ihre Preise nicht online sind, gehen sie zur Konkurrenz die transparent ist."

**9. "Ihre Website wirkt wie ein Baukasten"**
Trigger: Jimdo/Wix/WordPress.com Branding erkennbar. Argument: "Besucher erkennen sofort ob eine Website professionell erstellt wurde oder ein Baukasten ist. Ein professioneller Online-Auftritt signalisiert: Dieses Business meint es ernst."

**10. "Ihre Website ist seit [Jahr] unverändert"**
Trigger: Copyright-Jahr alt, CMS-Version veraltet, letzte Änderung >2 Jahre. Argument: "Ihre Website wurde seit [Jahr] nicht aktualisiert. Online-Erwartungen verändern sich schnell – was 2019 modern war, wirkt heute veraltet."

---

## Wie der Quick-Check die gesamte Pipeline verändert

### Vorher (aktuelle Spec):
```
Scrape 200 → Validate 170 → Enrich 130 → Crawl 130 → Review 130 → Score 130
                                                                    ↓
                                                        80 haben Score >70
                                                        50 waren eigentlich OK
                                                        Geld für 50 OK-Leads verschwendet
```

### Nachher (mit Quick-Quality-Check):
```
Scrape 200 → Validate 170 → QUICK-CHECK 170 (€0.85) → 70 sind schlecht genug
                                                         ↓
                                   nur 70 → Enrich → Crawl → Review → Score
                                   100 archiviert (Website ist OK, kein Kandidat)
                                   60% weniger API-Kosten für Steps 3-7
```

### Was sich in anderen Steps ändert:

**Step 2 (Validation):** Wird um den Quick-Quality-Check erweitert. Validation prüft jetzt nicht nur "ist die URL erreichbar" sondern auch "ist die Website schlecht genug." Ergebnis: Jeder Lead hat nach Validation einen quick_quality_score.

**Step 5 (Quality Review):** Wird zum DEEP Review. Bestätigt und detailliert den Quick-Check. Findet die SPEZIFISCHEN Schwächen (nicht nur "Website ist alt" sondern "Hero fehlt, CTA versteckt, Testimonials nicht eingebunden"). Der Quick-Check sagt "schlecht", der Deep Review sagt "schlecht WEIL."

**Step 7 (Scoring):** Quick-Quality-Score fließt als Input ein. Leads mit Quick-Score >80 bekommen einen Scoring-Bonus (die Opportunity ist offensichtlich).

**Step 13 (Outreach):** Die Outreach-Argumente werden DIREKT aus den erkannten Schwächen abgeleitet. Nicht generisch "Ihre Website ist verbesserbar" sondern spezifisch "Ihre Website lädt 8 Sekunden, ist nicht mobil-optimiert, und hat keinen Weg für Online-Buchungen."

**Gate 1 (Admin Backend):** Du siehst nicht mehr nur "170 validierte Leads" sondern "170 validierte Leads, davon 70 mit Quick-Score >50 (rot markiert), 50 mit Score 30-49 (gelb), 50 mit Score <30 (grün = gute Website, kein Kandidat)." Deine Entscheidung bei Gate 1 wird viel einfacher: Die roten sind offensichtlich, die gelben prüfst du kurz, die grünen ignorierst du.

### Neue Felder im DB Schema:

```
leads.quick_quality_score    Integer     -- 0-100 (higher = worse website = better for us)
leads.quick_quality_signals  JSONB       -- {technical{}, lighthouse{}, visual{}, conversion{}}
leads.quick_quality_checked  Boolean     -- has the quick check been performed?
leads.website_weaknesses     String[]    -- ['no_ssl', 'not_mobile', 'no_cta', 'outdated_design', ...]
```

### Neues Event:
```
lead.quality_checked  -- after quick quality check
  payload: { leadId, quickScore, topWeaknesses[], worth_pursuing: boolean }
```
