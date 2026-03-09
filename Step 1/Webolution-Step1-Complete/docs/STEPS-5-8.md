**WEBOLUTION**

Deep Spec: Steps 5--8

*Quality Review → Competitor Benchmarking → Lead Scoring → Rebuild Planning*

Analyse, Bewertung & strategische Planung

**SocialCooks × Webolution**

März 2026

+-------+----------------------------------------------------------------------------------------+
| **5** | **Website Quality Review**                                                             |
|       |                                                                                        |
|       | *Technische + visuelle + inhaltliche Bewertung gegen kalibrierten Industrie-Benchmark* |
+-------+----------------------------------------------------------------------------------------+

+----------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------+
| **INPUT**                                                                                                                              | **OUTPUT**                                                                                                                                                                                                                    | **QUALITY GATE**                                                                                                           |
|                                                                                                                                        |                                                                                                                                                                                                                               |                                                                                                                            |
| Content Package aus Stufe 4: • pages\[\] mit Text + Screenshots • images\[\] • brand_assets{} • firm_profile{} • extraction_confidence | Quality Report: • technical_score (0-100) • visual_score (0-100) • content_score (0-100) • industry_compliance_score (0-100) • weaknesses\[\]{type, severity, description, evidence, outreach_hook} • review_confidence (0-1) | Mindestens 3 verwertbare Schwächen identifiziert. Jede Schwäche hat evidence (Screenshot/Daten). Review Confidence \> 0.6. |
+----------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------+

5.1 Das Calibration-Problem

+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Kernproblem: Ohne Ground Truth ist jeder Score willkürlich**                                                                                                                                                                                                                          |
|                                                                                                                                                                                                                                                                                         |
| Wenn die AI sagt „Diese Website scored 42/100" -- was bedeutet das? Gegen welchen Maßstab? Ohne kalibrierte Referenz-Websites ist der Score bedeutungslos. Deshalb braucht Webolution für jede Industrie ein Calibration Set: 25-30 manuell bewertete Websites die als Referenz dienen. |
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

Calibration Set pro Industrie

Für jede Industrie werden 25-30 Websites manuell bewertet und als Benchmark gespeichert:

1.  5 Excellent Websites (Score 85-100): Die besten der Branche. Modern, schnell, mobile-first, hervorragender Content, alle Branchenstandards erfüllt. Diese definieren das „Ideal"

2.  10 Good Websites (Score 60-84): Solide, professionell, aber mit Optimierungspotenzial. Typisch für Firmen die vor 2-3 Jahren relauncht haben

3.  10 Poor Websites (Score 25-59): Veraltet, langsam, nicht mobil-optimiert, dünner Content. Das sind unsere typischen Leads

4.  5 Terrible Websites (Score 0-24): Die schlimmsten Fälle. Flash-Reste, Visitenkarten-Websites, Under-Construction seit 2019

Das AI-Modell bewertet neue Websites relativ zu diesem Calibration Set. Monatlich wird das Set überprüft und ggf. erweitert.

Calibration-Prozess

1.  Initiales Set erstellen: Manuell 30 Websites pro Industrie bewerten (Team-Aufgabe, \~2h pro Industrie)

2.  AI-Scoring testen: AI bewertet dieselben 30 Websites. Abweichung von manueller Bewertung messen

3.  Prompt-Tuning: AI-Prompt anpassen bis Abweichung \<10 Punkte auf dem Calibration Set

4.  Ongoing Calibration: Jede 100. bewertete Website wird zusätzlich manuell geprüft. Drift-Detection: Wenn AI-Scores systematisch höher/niedriger werden als manuelle Checks → Re-Calibration

5.2 Technical Score (0-100)

Komplett automatisiert, keine AI nötig. Basiert auf messbaren Metriken:

+------------------------+-------------+-----------------------+-------------------------------------------------------------------+----------------+
| **Metrik**             | **Gewicht** | **Tool**              | **Scoring-Logik**                                                 | **Max Punkte** |
+------------------------+-------------+-----------------------+-------------------------------------------------------------------+----------------+
| Lighthouse Performance | 20%         | PageSpeed API         | Score direkt übernehmen (0-100)                                   | 20             |
+------------------------+-------------+-----------------------+-------------------------------------------------------------------+----------------+
| Mobile Friendliness    | 15%         | Viewport + Tap-Target | Responsive + Touch-friendly = 15. Nur responsive = 10. Keines = 0 | 15             |
+------------------------+-------------+-----------------------+-------------------------------------------------------------------+----------------+
| SSL/HTTPS              | 10%         | HTTP Check            | HTTPS + gültig + HSTS = 10. HTTPS ohne HSTS = 7. Kein SSL = 0     | 10             |
+------------------------+-------------+-----------------------+-------------------------------------------------------------------+----------------+
| Core Web Vitals        | 15%         | PageSpeed API         | Alle grün = 15. 1 orange = 10. 1 rot = 5. Alle rot = 0            | 15             |
+------------------------+-------------+-----------------------+-------------------------------------------------------------------+----------------+
| SEO Basics             | 15%         | Custom Crawler        | Title-Tag vorhanden (+3)                                          | 15             |
|                        |             |                       |                                                                   |                |
|                        |             |                       | Meta-Description (+3)                                             |                |
|                        |             |                       |                                                                   |                |
|                        |             |                       | H1-Struktur (+3)                                                  |                |
|                        |             |                       |                                                                   |                |
|                        |             |                       | Sitemap.xml (+2)                                                  |                |
|                        |             |                       |                                                                   |                |
|                        |             |                       | Schema Markup (+4)                                                |                |
+------------------------+-------------+-----------------------+-------------------------------------------------------------------+----------------+
| Barrierefreiheit       | 10%         | Lighthouse a11y       | Score \> 90 = 10. 70-90 = 7. 50-70 = 4. \<50 = 0                  | 10             |
+------------------------+-------------+-----------------------+-------------------------------------------------------------------+----------------+
| Ladezeit (absolut)     | 10%         | Playwright Timing     | \<2s = 10. 2-4s = 7. 4-6s = 4. \>6s = 0                           | 10             |
+------------------------+-------------+-----------------------+-------------------------------------------------------------------+----------------+
| Broken Links           | 5%          | Crawler               | 0 broken = 5. 1-3 = 3. 4-10 = 1. \>10 = 0                         | 5              |
+------------------------+-------------+-----------------------+-------------------------------------------------------------------+----------------+

Inversion für Opportunity: Technical Weakness Score = 100 - Technical Score. Je höher der Weakness Score, desto größer die Opportunity.

5.3 Visual Score (0-100) -- AI-basiert

Die visuelle Bewertung ist die anspruchsvollste und gleichzeitig die wirkungsvollste. Eine Website kann technisch ok sein aber trotzdem „alt" aussehen. Menschen entscheiden in 50ms ob eine Website vertrauenswürdig wirkt.

AI Vision Analysis Prompt

Claude Vision analysiert die Desktop- und Mobile-Screenshots mit einem kalibrierten Prompt:

+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Prompt-Struktur (vereinfacht)**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
|                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| Du bist ein erfahrener Webdesigner. Bewerte diese Website-Screenshots für ein \[INDUSTRIE\]-Business auf einer Skala von 0-100. Hier sind Referenz-Beispiele: \[5 Calibration-Screenshots mit Scores\]. Bewerte anhand folgender Kriterien und gib für JEDES einen Sub-Score (0-100) mit 1-Satz-Begründung: 1. Design-Modernität (2024-Standard?) 2. Farbschema & Konsistenz 3. Typografie & Lesbarkeit 4. Bildqualität & Relevanz 5. Layout & Whitespace 6. Above-the-Fold-Wirkung 7. Branchenfit Antwort als JSON. |
+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

Sub-Scores mit Gewichtung

  --------------------- ------------- ----------------------------------------------------------- ------------------------------------------------ ----------------------------------------------------
  **Kriterium**         **Gewicht**   **Score 90-100**                                            **Score 50-70**                                  **Score 0-30**

  Design-Modernität     20%           Aktueller Trend (2023+), klares Grid, moderne UI-Patterns   Erkennbar datiert (2018-2021), aber funktional   Vor 2015, offensichtlich veraltet, Tabellenlayouts

  Farbschema            10%           Konsistent, professionell, guter Kontrast                   Etwas inkonsistent, aber akzeptabel              Clash, zu viele Farben, schlechter Kontrast

  Typografie            10%           Web-Fonts, klare Hierarchie, gute Lesbarkeit                System-Fonts, aber lesbar                        Comic Sans, zu klein, keine Hierarchie

  Bildqualität          20%           Professionelle Fotos, relevant, hochauflösend               Stock-Fotos, aber passend                        Pixelig, irrelevant, Clipart, keine Bilder

  Layout & Whitespace   15%           Großzügiger Whitespace, klare Sections                      Etwas eng, aber strukturiert                     Kein Whitespace, überladen, chaotisch

  Above-the-Fold        15%           Sofort klar: Was macht die Firma? + CTA sichtbar            Firma erkennbar, aber kein klarer CTA            Unklar was die Firma macht, kein CTA

  Branchenfit           10%           Design passt perfekt zur Branche                            Generisch, aber nicht störend                    Völlig unpassend (Friseur sieht aus wie Bank)
  --------------------- ------------- ----------------------------------------------------------- ------------------------------------------------ ----------------------------------------------------

Confidence-Sicherung für Visual Score

-   Dual-Pass: Jeder Screenshot wird ZWEIMAL von Claude bewertet (unterschiedliche Temperature-Settings). Abweichung \>15 Punkte → dritter Pass + Median

-   Cross-Validation: Desktop-Score und Mobile-Score sollten korrelieren (±20 Punkte). Größere Abweichung → Manuelles Review

-   Calibration-Check: Alle 50 Bewertungen wird ein Website aus dem Calibration Set mitbewertet. Abweichung \>12 Punkte → Prompt-Recalibration

-   Outlier-Detection: Scores die \>30 Punkte vom Technical Score abweichen werden flagged (technisch gut aber visuell schlecht ist möglich, aber \>30 Punkte Differenz ist verdächtig)

5.4 Content Score (0-100) -- Hybrid

Kombination aus automatisierten Metriken und AI-Analyse:

  --------------------------- ------------- ----------------------------------------------------- --------------------------------------------------------------------
  **Kriterium**               **Gewicht**   **Methode**                                           **Scoring**

  Textlänge (Gesamt)          10%           Wortcount aller Seiten                                \>5.000 Wörter = 10. 2-5k = 7. 500-2k = 4. \<500 = 0

  Textlänge pro Seite (Avg)   10%           Wortcount / Seitenanzahl                              \>300 Wörter/Seite = 10. 100-300 = 6. \<100 = 2

  Lesbarkeit                  10%           Flesch-Reading-Ease (DE)                              \>60 = 10. 40-60 = 6. \<40 = 3 (zu akademisch)

  Aktualität                  10%           Copyright-Jahr, letzte Blog-Posts, „Aktuelles"        2024-2025 = 10. 2022-2023 = 6. \<2022 = 2

  Service-Beschreibungen      15%           AI: Sind Leistungen klar & detailliert beschrieben?   Dedizierte Seiten mit Detail = 15. Stichpunkte = 8. Nur in Nav = 2

  Social Proof                15%           AI: Testimonials, Referenzen, Fallstudien?            ≥5 Testimonials + Fallstudien = 15. 1-4 = 8. Keine = 0

  Conversion-Elemente         15%           CTA-Count, Formular, Telefon sichtbar                 CTA + Formular + Telefon = 15. 2/3 = 10. 1/3 = 5. 0 = 0

  Unique Content              10%           AI: Original vs. Template-Text                        100% original = 10. Mix = 6. Offensichtlich Template = 2

  Branchenspezifisch          5%            Required Elements aus Industry Profile                Alle vorhanden = 5. \>50% = 3. \<50% = 0
  --------------------------- ------------- ----------------------------------------------------- --------------------------------------------------------------------

5.5 Industry Compliance Score (0-100)

Jede Industrie hat spezifische Anforderungen. Der Compliance Score misst wie gut die aktuelle Website diese erfüllt:

+---------------+--------------------------------------------+-----------------------------------+
| **Industrie** | **Kritische Anforderungen (je 20 Punkte)** | **Nice-to-Have (je 5-10 Punkte)** |
+---------------+--------------------------------------------+-----------------------------------+
| Anwälte       | Rechtsgebiet-Übersicht mit Beschreibungen  | Blog/Fachartikel (+10)            |
|               |                                            |                                   |
|               | Anwaltsprofile mit Qualifikation + Foto    | FAQ pro Rechtsgebiet (+10)        |
|               |                                            |                                   |
|               | BRAO-konforme Pflichtangaben               | Schema Attorney (+5)              |
|               |                                            |                                   |
|               | Klare Kontaktmöglichkeit                   | Mandantenbewertungen (+5)         |
|               |                                            |                                   |
|               | Impressum vollständig + korrekt            |                                   |
+---------------+--------------------------------------------+-----------------------------------+
| Ärzte         | Leistungsspektrum beschrieben              | Patienteninfo-Bereich (+10)       |
|               |                                            |                                   |
|               | Team mit Qualifikationen                   | Praxisvideo/-tour (+5)            |
|               |                                            |                                   |
|               | Sprechzeiten + Anfahrt                     | Jameda-Integration (+5)           |
|               |                                            |                                   |
|               | Online-Terminbuchung                       | Mehrsprachigkeit (+10)            |
|               |                                            |                                   |
|               | HWG-konforme Texte                         |                                   |
+---------------+--------------------------------------------+-----------------------------------+
| Coaches       | Über-mich mit Persönlichkeit               | Lead Magnet/Freebie (+10)         |
|               |                                            |                                   |
|               | Klares Angebot/Pakete mit Preisen          | Podcast/Blog (+10)                |
|               |                                            |                                   |
|               | Social Proof (Testimonials)                | Instagram-Integration (+5)        |
|               |                                            |                                   |
|               | Buchungs-CTA above-the-fold                | About-Video (+5)                  |
|               |                                            |                                   |
|               | Erklärung der Methode/Ansatz               |                                   |
+---------------+--------------------------------------------+-----------------------------------+
| Friseure      | Bildergalerie der Arbeiten                 | Vorher/Nachher-Bilder (+10)       |
|               |                                            |                                   |
|               | Preisliste (aktuell)                       | Instagram-Feed (+5)               |
|               |                                            |                                   |
|               | Online-Buchungstool                        | Produktverkauf (+5)               |
|               |                                            |                                   |
|               | Öffnungszeiten + Standort                  | Bewertungs-Widget (+10)           |
|               |                                            |                                   |
|               | Team-Vorstellung                           |                                   |
+---------------+--------------------------------------------+-----------------------------------+
| Handwerker    | Leistungsübersicht                         | Vorher/Nachher-Galerie (+10)      |
|               |                                            |                                   |
|               | Einzugsgebiet klar definiert               | Notdienst-Info (+5)               |
|               |                                            |                                   |
|               | Kontaktformular + Telefon                  | Google-Bewertungen (+5)           |
|               |                                            |                                   |
|               | Referenzen/Projekte mit Bildern            | Angebotsanfrage-Formular (+10)    |
|               |                                            |                                   |
|               | Meisterbrief/Zertifikate                   |                                   |
+---------------+--------------------------------------------+-----------------------------------+

5.6 Schwachstellen-Identifikation & Priorisierung

Jede identifizierte Schwäche ist ein potenzielles Sales-Argument. Aber nicht jede Schwäche überzeugt einen Lead. Wir priorisieren nach drei Dimensionen:

  --------------- ----------------------------------- ----------------------------------------------------------------- -------------------------------------------------- ----------------------------------------------
  **Dimension**   **Frage**                           **High**                                                          **Medium**                                         **Low**

  Impact          Wie sehr schadet es dem Geschäft?   Direkt messbar: Kein SSL, nicht mobil, keine Kontaktmöglichkeit   Indirekt: Schlechte SEO, keine Bewertungen         Kosmetisch: Datiertes Design, fehlende Icons

  Sichtbarkeit    Kann der Lead es selbst sehen?      Sofort erkennbar: Seite sieht alt aus, Handy unbenutzbar          Nach Erklärung klar: SEO-Lücken, fehlende Schema   Technisch: Core Web Vitals, Security Headers

  Lösbarkeit      Können wir es im Rebuild fixen?     Einfach: Neues Design, bessere Struktur, SEO-Basics               Aufwendig: Content-Erstellung, Fotografie          Nicht in Scope: Domain-Alter, Backlinks
  --------------- ----------------------------------- ----------------------------------------------------------------- -------------------------------------------------- ----------------------------------------------

Schwachstellen-Klassifikation

  ---------------------- ---------------------------------------------------- --------------------------------------------------- ------------------------------------------------------------------------
  **Klasse**             **Beschreibung**                                     **Verwendung**                                      **Beispiel**

  **A -- Sales-Gold**    High Impact + High Sichtbarkeit + Einfach lösbar     In Outreach-Email UND auf Preview-Vergleichsseite   „Ihre Website ist nicht mobiloptimiert -- 67% der Besucher sind mobil"

  **B -- Überzeugend**   High Impact + Medium Sichtbarkeit + Lösbar           Auf Analyse-PDF und Preview-Seite                   „Kein Schema Markup = unsichtbar für Google Rich Snippets"

  **C -- Ergänzend**     Medium Impact oder nur nach Erklärung verständlich   Nur auf detailliertem Analyse-PDF                   „Core Web Vitals im roten Bereich"

  **D -- Intern**        Low Impact oder nicht lösbar                         Nur interne Dokumentation, nie zum Lead             „Security Headers fehlen"
  ---------------------- ---------------------------------------------------- --------------------------------------------------- ------------------------------------------------------------------------

Outreach-Hook-Generierung

Für jede Klasse-A- und Klasse-B-Schwäche wird automatisch ein „Outreach Hook" generiert -- ein fertiger Satz für die Email:

-   Template: „\[Problem erklärt in Alltagssprache\] -- \[Konsequenz für das Geschäft\] -- \[Was wir besser gemacht haben\]"

-   Beispiel Anwalt: „Wenn potenzielle Mandanten „Familienrecht München" googlen, taucht Ihre Kanzlei erst auf Seite 3 auf. In unserer Vorschau haben wir Ihre Rechtsgebiete SEO-optimiert, damit Sie besser gefunden werden."

-   Beispiel Friseur: „Ihr Salon hat 4.6 Sterne auf Google, aber auf Ihrer Website findet man keine einzige Kundenbewertung. Wir haben Ihre Google-Reviews direkt eingebunden."

-   Industry Profile definiert typische Hooks pro Schwächen-Typ und Branche

5.7 Error Handling

  ------------------------------------------ ------------------------------------------------ ---------------------------------------------------------------------------------------------------------------------
  **Failure Mode**                           **Erkennung**                                    **Fallback**

  PageSpeed API nicht verfügbar              HTTP Error                                       Lighthouse lokal ausführen (via Playwright). Oder: Technical Score nur aus eigenen Messungen berechnen

  AI Visual Score Halluzination              Score weicht \>30 Punkte von Technical ab        Dritter AI-Pass mit expliziten Calibration-Beispielen. Bei weiterhin verdächtig: Manuelles Review

  Zu wenige Schwächen (\<3)                  Weakness Count nach Analyse                      Sensitivität erhöhen (Schwellenwerte senken). Wenn wirklich \<3: Lead hat gute Website → niedriger Score in Stufe 7

  AI-Prompt-Drift                            Calibration-Check Score \>12 Punkte Abweichung   Sofortige Re-Calibration. Letzte 20 Bewertungen manuell prüfen und ggf. korrigieren

  Screenshots fehlgeschlagen (aus Stufe 4)   Keine Screenshots vorhanden                      Visual Score = 0 (nicht bewertbar). Nur Technical + Content Score berechnen. Flag setzen
  ------------------------------------------ ------------------------------------------------ ---------------------------------------------------------------------------------------------------------------------

5.8 KPIs & Monitoring

  --------------------------------------- -------------------------------- --------------------------------------
  **Metrik**                              **Zielwert**                     **Alert wenn**

  Avg. Review-Dauer pro Lead              \<60s (technisch) + \<30s (AI)   \>120s

  Avg. Schwächen pro Lead                 5-12                             \<3 (zu lasch) oder \>20 (zu streng)

  Klasse-A-Schwächen pro Lead             2-4                              \<1 (keine Sales-Argumente)

  AI-Score-Stabilität (Dual-Pass Delta)   \<10 Punkte                      \>15 Punkte

  Calibration Drift                       \<8 Punkte Abweichung            \>12 Punkte

  Review Confidence Avg.                  \>0.75                           \<0.6

  Cost pro Review                         \<€0.05                          \>€0.12
  --------------------------------------- -------------------------------- --------------------------------------

+-------+---------------------------------------------------------------------------------+
| **6** | **Competitor Benchmarking**                                                     |
|       |                                                                                 |
|       | *Lokale Konkurrenten analysieren -- relativer Vergleich statt abstrakter Score* |
+-------+---------------------------------------------------------------------------------+

+-------------------------------------------------------------------------------------------------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------+
| **INPUT**                                                                                                               | **OUTPUT**                                                                                                                                                                                                                     | **QUALITY GATE**                                                                                                 |
|                                                                                                                         |                                                                                                                                                                                                                                |                                                                                                                  |
| Enriched Lead mit: • competitor_place_ids\[\] (aus Stufe 3) • industry_tag • city/region • Quality Report (aus Stufe 5) | Competitor Report: • competitors\[\]{name, url, scores, features\[\]} • comparison_matrix{} • relative_position (1-5: 1=best, 5=worst) • top_differentiators\[\] • competitive_pressure_score (0-100) • outreach_arguments\[\] | Mindestens 3 Konkurrenten analysiert. Relative Position bestimmt. Mind. 2 konkrete Differenzierer identifiziert. |
+-------------------------------------------------------------------------------------------------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------+

6.1 Warum Feature-Level statt Score-Level?

+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Der Unterschied zwischen „Ihr Score ist 35" und echtem Wettbewerbsdruck**                                                                                                                                                                                                                                                                                                                                                                                                          |
|                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| Score-Level: „Ihre Website scored 35/100, der Durchschnitt in Ihrer Branche ist 62." → Abstrakt, wenig emotional. Feature-Level: „Kanzlei Schmidt in Ihrer Straße hat eine Online-Terminbuchung mit 89 Google-Bewertungen prominent auf der Startseite. Potenzielle Mandanten sehen das zuerst." → Konkret, emotional, handlungsfördernd. Webolution arbeitet auf Feature-Level. Jeder Konkurrent wird nicht nur gescored, sondern seine SPEZIFISCHEN Features werden identifiziert. |
+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

6.2 Konkurrenten-Auswahl

Automatische Identifikation

1.  Google Maps „Similar Places": Automatisch von Google vorgeschlagene ähnliche Geschäfte in der Nähe. Beste Quelle für direkte lokale Konkurrenz

2.  Google-Suche nach Branche + Stadt: Top 10 organische Ergebnisse für „\[Branche\] \[Stadt\]". Wer hier rankt, ist der Wettbewerb den der Lead in Google sieht

3.  Gleiche Kategorie + Radius: Alle Google-Maps-Ergebnisse in gleicher Kategorie innerhalb des industrie-spezifischen Radius

Auswahlkriterien

-   Top 3-5 Konkurrenten auswählen, priorisiert nach: Geografische Nähe (je näher, desto relevanter als Argument), Google-Ranking (wer rankt für die gleichen Keywords?), Google-Rating (wer hat die besten Bewertungen?), Website-Qualität (wir wollen Konkurrenten mit BESSEREN Websites -- das erzeugt Druck)

-   Mindestens 1 Konkurrent muss eine objektivMessbar bessere Website haben als der Lead (sonst kein Wettbewerbsdruck-Argument)

-   Exclude: Ketten/Franchise (nicht vergleichbar), Websites die offensichtlich auch schlecht sind (kein Druck-Argument), Firmen die selbst Webolution-Kunden sind

6.3 Quick-Analyse pro Konkurrent

Kein Full-Crawl. Nur schnelle Checks die in \<30s pro Konkurrent machbar sind:

  ------------------------------- ------------------------------------------------- ------------- -------------------------------------
  **Check**                       **Methode**                                       **Dauer**     **Zweck**

  Screenshot (Desktop + Mobile)   Playwright: Homepage laden + Screenshot           5s            Visueller Vergleich

  PageSpeed Score                 PageSpeed API: Performance + Mobile               3s            Technischer Vergleich

  Mobile-Check                    Viewport-Test                                     2s            Mobil ja/nein

  SSL-Check                       HTTPS-Prüfung                                     1s            Sicherheit ja/nein

  Schema Markup                   Structured Data Check                             2s            SEO-Feature ja/nein

  Feature-Detection               AI Vision: Screenshot analysieren (siehe unten)   10s           Welche Features hat der Konkurrent?

  Google Rating + Count           Aus Enrichment-Daten oder Places API              0s (cached)   Reputationsvergleich

  Design-Epoche (AI)              AI Vision: Geschätztes Alter des Designs          5s            „Moderner als der Lead?"
  ------------------------------- ------------------------------------------------- ------------- -------------------------------------

6.4 Feature-Detection auf Konkurrenten-Websites

AI Vision analysiert die Konkurrenten-Screenshots und identifiziert spezifische Features:

  -------------------------- -------------------------------------------------------------- ----------------------------------------------------------------
  **Feature-Kategorie**      **Was gesucht wird**                                           **Outreach-Relevanz wenn Konkurrent es hat, Lead nicht**

  Online-Buchung             Calendly-Widget, Doctolib, Treatwell, „Termin buchen"-Button   „Ihr Konkurrent bietet Online-Termine -- Sie nicht"

  Bewertungs-Integration     Google Reviews Widget, Jameda-Badge, ProvenExpert              „Kanzlei Schmidt zeigt 89 Bewertungen auf der Startseite"

  Blog/Content               Blog-Section sichtbar, News, Fachartikel                       „Die Konkurrenz zeigt Expertise durch regelmäßige Fachartikel"

  Bildergalerie              Portfolio, Projekte, Vorher/Nachher                            „Salon Bella zeigt beeindruckende Arbeiten online"

  Team-Präsentation          Team-Section mit Fotos + Beschreibungen                        „Bei der Konkurrenz sieht man, mit wem man es zu tun hat"

  Chat/Messenger             Live-Chat-Widget, WhatsApp-Button                              „Sofortige Kontaktmöglichkeit wie bei \[Konkurrent\]"

  Preistransparenz           Sichtbare Preisliste, Pakete                                   „Kunden schätzen Preistransparenz wie bei \[Konkurrent\]"

  Video-Content              Eingebettete Videos, Praxistour                                „Ein Praxisvideo schafft Vertrauen -- wie bei \[Konkurrent\]"

  FAQ-Section                Häufige Fragen beantwortet                                     „Beantwortet Fragen bevor der Kunde anruft"

  Social Media Integration   Instagram-Feed, Facebook-Widget                                „Social Media direkt auf der Website integriert"
  -------------------------- -------------------------------------------------------------- ----------------------------------------------------------------

6.5 Vergleichsmatrix & Relative Position

Aus allen Analysen wird eine Vergleichsmatrix erstellt. Beispiel für einen Anwalt in München:

  ------------------------ ------------------ ----------------------- ------------------- ------------------
                                              **Schmidt & Partner**   **Kanzlei Weber**   **RA Dr. Bauer**

  **Google Rating**        4.2 (28 Reviews)   4.6 (89)                4.4 (45)            4.8 (12)

  **Mobile**               ✗                  ✓                       ✓                   ✓

  **PageSpeed**            23                 72                      58                  85

  **SSL**                  ✗                  ✓                       ✓                   ✓

  **Online-Termin**        ✗                  ✓                       ✗                   ✓

  **Blog**                 ✗                  ✓ (24 Artikel)          ✓ (8)               ✗

  **Reviews integriert**   ✗                  ✓                       ✗                   ✓

  **Schema Markup**        ✗                  ✓                       ✗                   ✓

  **Team-Fotos**           Stock              Echt                    Echt                ✗

  **Design-Epoche**        \~2016             \~2023                  \~2020              \~2024
  ------------------------ ------------------ ----------------------- ------------------- ------------------

Relative Position berechnen

-   Position 1 (Best): Lead hat die beste Website unter allen Konkurrenten → Kein Outreach sinnvoll. Lead verwerfen

-   Position 2-3 (Mittelfeld): Lead ist nicht der Schlechteste. Outreach-Argument: „Sie könnten zur Nr. 1 aufsteigen"

-   Position 4-5 (Schlechtester): Lead hat die schlechteste Website. Stärkstes Outreach-Argument: „Ihre Konkurrenz ist online deutlich besser aufgestellt"

Competitive Pressure Score (0-100)

  ------------------------------------------------------------------ ------------- -------------------------------------------------------------------------------------
  **Faktor**                                                         **Gewicht**   **Berechnung**

  Position (1-5)                                                     30%           Position 5 = 100, Position 4 = 80, Position 3 = 50, Position 2 = 20, Position 1 = 0

  Feature-Gap (Anzahl Features die Konkurrenten haben, Lead nicht)   30%           \>5 Gaps = 100, 3-5 = 70, 1-2 = 40, 0 = 0

  Design-Gap (Avg Konkurrenten-Design-Epoche vs. Lead)               20%           \>5 Jahre Unterschied = 100, 3-5 = 70, 1-3 = 40, \<1 = 0

  Review-Gap (Avg Konkurrenten-Bewertungen vs. Lead)                 20%           Konkurrenten \>2x mehr Reviews = 100, 1.5-2x = 60, ähnlich = 20
  ------------------------------------------------------------------ ------------- -------------------------------------------------------------------------------------

6.6 Outreach-Argument-Generierung

Aus der Vergleichsmatrix werden automatisch personalisierte Sales-Argumente generiert:

Argument-Templates

1.  Konkurrenz-Überlegenheit: „\[Konkurrent\] in \[Straße/Stadtteil\] hat \[Feature\], das Sie noch nicht haben. Potenzielle \[Mandanten/Patienten/Kunden\] sehen das zuerst."

2.  Feature-Gap: „\[Anzahl\] von \[Anzahl\] \[Branche\] in \[Stadt\] bieten bereits \[Feature\] an. Ihre Website gehört zu den wenigen ohne."

3.  Design-Vergleich: „Im Vergleich zu \[Konkurrent\] wirkt Ihre Website wie aus einer anderen Generation. Erster Eindruck zählt -- besonders online."

4.  Review-Leverage: „Sie haben \[Rating\] Sterne auf Google -- das ist gut! Aber Ihre Website zeigt diese Bewertungen nicht. \[Konkurrent\] hat sie prominent auf der Startseite."

Pro Lead werden die Top 2-3 stärksten Argumente ausgewählt (basierend auf Klasse A aus Stufe 5 + Competitor Gap). Diese fließen direkt in Stufe 13 (Outreach Preparation).

6.7 Error Handling

  --------------------------------------- ------------------------------------- --------------------------------------------------------------------------------------------------------------------------------------------------------
  **Failure Mode**                        **Erkennung**                         **Fallback**

  \<3 Konkurrenten gefunden               competitor_count \< 3                 Radius erhöhen. Branche verallgemeinern. Wenn trotzdem \<3: Kein Competitor-Report, Lead trotzdem weiter (Score in Stufe 7 ohne Competitor-Komponente)

  Alle Konkurrenten auch schlecht         Avg. Competitor Score \< 40           Kein Wettbewerbsdruck-Argument möglich. Alternatives Framing: „Sie könnten der Erste sein, der online glänzt"

  Lead hat beste Website                  relative_position = 1                 Kein Outreach sinnvoll. Lead disqualifizieren für diese Pipeline. Ggf. in andere Pipeline (z.B. SEO-Optimierung statt Redesign)

  Konkurrenten-Website nicht erreichbar   HTTP Error beim Quick-Scan            Konkurrent aus Vergleich entfernen. Nächstbesten nehmen

  AI Feature-Detection unsicher           Confidence \< 0.5 auf einem Feature   Feature als „unconfirmed" markieren. Nicht in Outreach-Argumente aufnehmen
  --------------------------------------- ------------------------------------- --------------------------------------------------------------------------------------------------------------------------------------------------------

6.8 KPIs & Monitoring

  ---------------------------------------- ---------------------- ----------------------------------------------------------------------
  **Metrik**                               **Zielwert**           **Alert wenn**

  Avg. Konkurrenten pro Lead               3-5                    \<2

  Avg. Feature-Gaps pro Lead               3-6                    \<1 (Website zu gut für Zielgruppe) oder \>10 (Scoring zu aggressiv)

  Leads mit Position 4-5                   \>60%                  \<40% (zu gute Leads werden gescraped)

  Leads mit Position 1 (disqualifiziert)   \<10%                  \>20% (Scraping-Filter zu breit)

  Competitor Quick-Analyse Dauer           \<45s pro Konkurrent   \>90s

  Cost pro Competitor Report               \<€0.08                \>€0.20

  Outreach-Argumente pro Lead              2-3                    \<1
  ---------------------------------------- ---------------------- ----------------------------------------------------------------------

+-------+---------------------------------------------------------------------------------------------------------+
| **7** | **Lead Scoring**                                                                                        |
|       |                                                                                                         |
|       | *Multi-dimensionales Scoring mit Feedback Loop -- die Entscheidung ob ein Lead die Pipeline durchläuft* |
+-------+---------------------------------------------------------------------------------------------------------+

+--------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------+
| **INPUT**                                                                                              | **OUTPUT**                                                                                                                                                                               | **QUALITY GATE**                                                                       |
|                                                                                                        |                                                                                                                                                                                          |                                                                                        |
| Alle bisherigen Daten: • Enriched Lead • Quality Report • Competitor Report • Content Package Metadata | Scored Lead: • final_score (0-100) • score_components{} • category (Hot/Warm/Cool/Cold) • routing_decision • score_confidence (0-1) • score_timestamp • predicted_conversion_probability | Score berechnet. Routing-Entscheidung klar. Kein Score-Komponent fehlt oder ist stale. |
+--------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------+

7.1 Scoring-Philosophie

+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Was der Score wirklich beantwortet**                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
|                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| Der Score beantwortet EINE Frage: Wenn wir für diesen Lead eine Website bauen und ihm eine Email schicken -- wie wahrscheinlich ist es, dass er bezahlt? Das ist eine Kombination aus: (A) Wie schlecht ist seine aktuelle Website? (B) Wie gut wird unsere Website? (C) Wird er die Email öffnen und die Preview ansehen? (D) Hat er das Budget und die Bereitschaft? Kein einzelner Score kann das perfekt vorhersagen. Aber mit genug Datenpunkten und einem Feedback Loop wird das Modell immer besser. |
+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

7.2 Score-Komponenten (6 Dimensionen)

+--------------------------+------------------------+----------------------------------------------------------------------------+-----------------------------+
| **Dimension**            | **Gewicht (Standard)** | **Was fließt ein**                                                         | **Datenquelle**             |
+--------------------------+------------------------+----------------------------------------------------------------------------+-----------------------------+
| **Website Weakness**     | 25%                    | Technical Score invertiert (100 - Score)                                   | Quality Report (Stufe 5)    |
|                          |                        |                                                                            |                             |
|                          |                        | Visual Score invertiert                                                    |                             |
|                          |                        |                                                                            |                             |
|                          |                        | Content Score invertiert                                                   |                             |
|                          |                        |                                                                            |                             |
|                          |                        | Gewichteter Durchschnitt                                                   |                             |
+--------------------------+------------------------+----------------------------------------------------------------------------+-----------------------------+
| **Content Opportunity**  | 15%                    | Extraction Completeness Score                                              | Content Package (Stufe 4)   |
|                          |                        |                                                                            |                             |
|                          |                        | Wie gut wird unsere Website? (genug Content für überzeugenden Rebuild?)    |                             |
|                          |                        |                                                                            |                             |
|                          |                        | Bild-Verfügbarkeit                                                         |                             |
+--------------------------+------------------------+----------------------------------------------------------------------------+-----------------------------+
| **Business Health**      | 20%                    | Business Health Score aus Enrichment                                       | Enrichment (Stufe 3)        |
|                          |                        |                                                                            |                             |
|                          |                        | Google Rating + Review Count                                               |                             |
|                          |                        |                                                                            |                             |
|                          |                        | Firmenstatus + Alter                                                       |                             |
+--------------------------+------------------------+----------------------------------------------------------------------------+-----------------------------+
| **Competitive Pressure** | 15%                    | Relative Position unter Konkurrenten                                       | Competitor Report (Stufe 6) |
|                          |                        |                                                                            |                             |
|                          |                        | Feature-Gap-Count                                                          |                             |
|                          |                        |                                                                            |                             |
|                          |                        | Design-Gap                                                                 |                             |
+--------------------------+------------------------+----------------------------------------------------------------------------+-----------------------------+
| **Reachability**         | 15%                    | Entscheider identifiziert? (+Name, +pers. Email)                           | Enrichment (Stufe 3)        |
|                          |                        |                                                                            |                             |
|                          |                        | Email validiert?                                                           |                             |
|                          |                        |                                                                            |                             |
|                          |                        | Branchenspezifischer Response-Indikator (Coaches responsen mehr als Ärzte) |                             |
+--------------------------+------------------------+----------------------------------------------------------------------------+-----------------------------+
| **Deal Potential**       | 10%                    | Firmengröße (größer = höheres Paket)                                       | Enrichment (Stufe 3)        |
|                          |                        |                                                                            |                             |
|                          |                        | Standort-Qualität (Premium = höhere Bereitschaft)                          |                             |
|                          |                        |                                                                            |                             |
|                          |                        | Industrie-Avg-Deal-Size                                                    |                             |
+--------------------------+------------------------+----------------------------------------------------------------------------+-----------------------------+

7.3 Industrie-spezifische Gewichtungs-Overrides

Jede Industrie gewichtet die 6 Dimensionen unterschiedlich, weil die Conversion-Treiber anders sind:

  ---------------------- -------------- ------------------------- ----------- ----------------------------- ----------------------------- ----------------
  **Dimension**          **Standard**   **Anwälte**               **Ärzte**   **Coaches**                   **Friseure**                  **Handwerker**

  Website Weakness       25%            20% (Content wichtiger)   25%         20% (Conversion wichtiger)    30% (visuell kritisch)        25%

  Content Opportunity    15%            20% (Expertise-Content)   15%         20% (Messaging)               10% (Bilder \> Text)          15%

  Business Health        20%            20%                       20%         15% (viele Neugründungen)     25% (viele Reviews = aktiv)   20%

  Competitive Pressure   15%            15%                       15%         10% (weniger lokal)           15%                           15%

  Reachability           15%            15%                       15%         20% (persönlich erreichbar)   10%                           15%

  Deal Potential         10%            10%                       10%         15% (variable Größe)          10%                           10%
  ---------------------- -------------- ------------------------- ----------- ----------------------------- ----------------------------- ----------------

7.4 Score-Berechnung (Pseudo-Code)

+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Berechnungs-Logik**                                                                                                                                                                                                                                                                                                                                                                                                                      |
|                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| final_score = ∑(dimension_score × industry_weight) Jede dimension_score ist 0-100. Jede industry_weight ist 0-1, Summe = 1.0. score_confidence = min(alle_input_confidences) → Der Score ist nur so zuverlässig wie sein schwächstes Input. predicted_conversion_probability = lookup(final_score, industry_conversion_table) → Basierend auf historischen Daten: Welche Conversion-Rate haben Leads mit diesem Score in dieser Industrie? |
+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

7.5 Score-Routing & Decision Matrix

+------------+---------------+------------------------------------------------------+------------------------------------------------------------------------+
| **Score**  | **Kategorie** | **Pipeline-Route**                                   | **Begründung**                                                         |
+------------+---------------+------------------------------------------------------+------------------------------------------------------------------------+
| **85-100** | Hot Lead      | Full Pipeline: Rebuild + Preview + Premium-Outreach  | Höchste Conversion-Wahrscheinlichkeit. Jede Stunde Verzögerung kostet. |
|            |               |                                                      |                                                                        |
|            |               | Priorität: Sofort bearbeiten (max 24h Queue-Zeit)    |                                                                        |
|            |               |                                                      |                                                                        |
|            |               | Ggf. höheres Paket in Pricing-Page vorselektieren    |                                                                        |
+------------+---------------+------------------------------------------------------+------------------------------------------------------------------------+
| **70-84**  | Warm Lead     | Full Pipeline: Rebuild + Preview + Standard-Outreach | Gute Chancen, aber kein Rush.                                          |
|            |               |                                                      |                                                                        |
|            |               | Normaler Queue-Durchlauf (max 48h)                   |                                                                        |
|            |               |                                                      |                                                                        |
|            |               | Standard-Paket vorselektieren                        |                                                                        |
+------------+---------------+------------------------------------------------------+------------------------------------------------------------------------+
| **55-69**  | Cool Lead     | Light Pipeline: Nur Analyse-Report (kein Rebuild)    | Zu unsicher für vollen Rebuild-Investment. Aber warm halten.           |
|            |               |                                                      |                                                                        |
|            |               | Nurture-Email mit Report + generischem Angebot       |                                                                        |
|            |               |                                                      |                                                                        |
|            |               | In Re-Engagement-Pool für späteren Retry             |                                                                        |
+------------+---------------+------------------------------------------------------+------------------------------------------------------------------------+
| **40-54**  | Cold Lead     | Kein Outreach. In Datenbank behalten                 | Investment lohnt sich aktuell nicht.                                   |
|            |               |                                                      |                                                                        |
|            |               | Re-Score in 6 Monaten                                |                                                                        |
|            |               |                                                      |                                                                        |
|            |               | Bei signifikantem Score-Anstieg: Upgrade             |                                                                        |
+------------+---------------+------------------------------------------------------+------------------------------------------------------------------------+
| **0-39**   | No Fit        | Aus Pipeline entfernen                               | Kein sinnvoller Anwendungsfall.                                        |
|            |               |                                                      |                                                                        |
|            |               | Nur in anonymisierten Statistiken behalten           |                                                                        |
+------------+---------------+------------------------------------------------------+------------------------------------------------------------------------+

7.6 Feedback Loop: Scoring-Verbesserung

Das Scoring-Modell ist nur so gut wie seine Kalibrierung. Ohne Feedback ist der Score Raten. Mit Feedback wird er immer präziser.

Daten die zurückfließen

  ------------------- -------------------------- ------------------------------------------------------------------------------------------------
  **Event**           **Wann**                   **Was fließt zurück**

  Email geöffnet      Stufe 14                   Reachability-Dimension war korrekt? Welche Score-Range öffnet am meisten?

  Preview besucht     Stufe 12                   Website Weakness + Competitive Pressure korrekt? Welche Features wurden am längsten angesehen?

  Payment             Stufe 15                   CONVERSION! Komplettes Score-Profil dieses Leads wird als „Gold Standard" gespeichert

  Opt-out/Ablehnung   Stufe 14                   Was war anders? Zu schlechter Fit? Falscher Ton? Falscher Entscheider?

  Keine Reaktion      Stufe 14 (nach 21 Tagen)   Email nicht angekommen? Thema nicht relevant? Timing falsch?
  ------------------- -------------------------- ------------------------------------------------------------------------------------------------

Monatliche Score-Optimierung

1.  Conversion-Analyse: Welche Score-Ranges konvertieren tatsächlich? Optimal: 80+ = \>8%, 65-79 = 4-7%. Wenn 70-79 genauso gut konvertiert wie 80+ → Threshold senken

2.  Dimension-Analyse: Welche Dimension korreliert am stärksten mit Conversion? Wenn Competitive Pressure der stärkste Prädiktor ist → Gewicht erhöhen

3.  False-Positive-Analyse: Leads mit Score \>80 die nicht konvertieren -- was war das Muster? Systematisch identifizieren und Scoring anpassen

4.  Industry-spezifische Recalibration: Jede Industrie separat optimieren. Coaches konvertieren bei Score 65 besser als Anwälte bei Score 80? Gewichtungen anpassen

5.  A/B-Testing: Neue Scoring-Varianten parallel laufen lassen (10% des Traffics). Wenn neue Variante besser performed → übernehmen

Score Decay & Freshness

-   Jeder Score hat einen score_timestamp. Scores älter als 4 Wochen werden als „stale" markiert

-   Vor Outreach (Stufe 13) wird geprüft: Ist der Score fresh? Wenn stale: Quick-Re-Score (Stufe 5 + 7 nochmal durchlaufen, Stufe 4 + 6 nur bei signifikanter Änderung)

-   Trigger für automatisches Re-Scoring: Website geändert (Wayback Machine Check), neue Google-Bewertungen (\>5 neue), Competitor-Änderung (neuer Konkurrent mit viel besserer Website)

7.7 Score Confidence & Unsicherheits-Handling

  ---------------------- --------------------------------------------------------- -----------------------------------------------------------------------------------------
  **Score Confidence**   **Bedeutung**                                             **Routing-Anpassung**

  **\> 0.8**             Hohe Zuverlässigkeit -- alle Inputs hochwertig            Vollautomatisches Routing. Keine Einschränkungen

  **0.6 - 0.8**          Mittlere Zuverlässigkeit -- einige Inputs unvollständig   Automatisches Routing, aber mit erhöhtem QA in Stufe 9 + 11

  **\< 0.6**             Niedrige Zuverlässigkeit -- kritische Inputs fehlen       Manuelles Review bevor Pipeline weitergeht. Welche Daten fehlen? Nachrecherche möglich?
  ---------------------- --------------------------------------------------------- -----------------------------------------------------------------------------------------

7.8 KPIs & Monitoring

  ---------------------------------------- ------------------ -----------------------------------------------------
  **Metrik**                               **Zielwert**       **Alert wenn**

  Hot Lead Rate (Score \>85)               8-15%              \<5% (zu streng) oder \>25% (zu lasch)

  Warm Lead Rate (70-84)                   15-25%             \<10% oder \>35%

  Full Pipeline Rate (Hot + Warm)          25-35%             \<15% (zu wenig Output) oder \>50% (Qualität sinkt)

  Score Confidence Avg.                    \>0.75             \<0.6

  Actual Conversion Rate vs. Predicted     ±2 Prozentpunkte   \>5 Punkte Abweichung (Modell unkalibriert)

  Time-to-Score (Stufe 1 → Stufe 7)        \<6h               \>24h

  Score Freshness (% stale bei Outreach)   \<5%               \>15%
  ---------------------------------------- ------------------ -----------------------------------------------------

+-------+------------------------------------------------------------------------------------------------+
| **8** | **Rebuild Planning**                                                                           |
|       |                                                                                                |
|       | *Der strategische Masterplan: Was übernehmen, verbessern, neu erstellen -- mit Decision Trees* |
+-------+------------------------------------------------------------------------------------------------+

+---------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------+
| **INPUT**                                                                             | **OUTPUT**                                                                                                                                                                                                                                                                                                                                                                                    | **QUALITY GATE**                                                                                                           |
|                                                                                       |                                                                                                                                                                                                                                                                                                                                                                                               |                                                                                                                            |
| Content Package + Quality Report + Competitor Report + Scored Lead + Industry Profile | Rebuild Plan: • template_selection{id, variant, justification} • page_structure\[\]{page, sections\[\]} • content_decisions\[\]{section, source, action, confidence} • image_strategy{per_section} • brand_decision{keep/upgrade/new} • seo_plan{keywords\[\], local_seo} • cta_strategy{primary, secondary, placement} • legal_checklist\[\]{requirement, status} • rebuild_confidence (0-1) | Phase 1: Manuelles Review jedes Plans. Phase 2: AI-QA-Check gegen Checkliste. Rebuild Confidence \> 0.7 für Auto-Approval. |
+---------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------+

8.1 Warum der Rebuild-Plan alles bestimmt

+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Die kritischste Entscheidung der Pipeline**                                                                                                                                                                                                                                                                                                                                                           |
|                                                                                                                                                                                                                                                                                                                                                                                                         |
| Wenn der Rebuild-Plan schlecht ist, wird die Website schlecht. Wenn die Website schlecht ist, kauft niemand. Alles downstream hängt von der Qualität dieses Plans ab. Ein erfahrener Webdesigner würde hier 2-4 Stunden Strategie investieren. Unsere AI muss dasselbe in 60 Sekunden leisten -- und genauso gute Entscheidungen treffen. Das erfordert explizite Decision Trees für jede Entscheidung. |
+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

8.2 Template-Auswahl (Decision Tree)

Jede Industrie hat 2-4 Template-Varianten. Die Auswahl basiert auf Firmengröße und Content-Verfügbarkeit:

Anwälte

  --------------------------------- ------------------- ---------------------------------------------------------
  **Bedingung**                     **Template**        **Begründung**

  1 Anwalt, ≤3 Rechtsgebiete        solo-trust          Persönlich, vertrauensvoll, kompakt

  2-5 Anwälte, Partnerschaft        firm-professional   Team-fokussiert, Rechtsgebiet-Übersichten

  \>5 Anwälte oder Großkanzlei      firm-authority      Corporate, umfassende Rechtsgebiet-Seiten, Blog-Bereich

  Nur 1 Rechtsgebiet (Spezialist)   specialist-expert   Tiefe statt Breite, Fallstudien-fokussiert
  --------------------------------- ------------------- ---------------------------------------------------------

Friseure

  -------------------------------- ----------------- ---------------------------------------------------------------
  **Bedingung**                    **Template**      **Begründung**

  Budget-Salon, wenig Bilder       salon-minimal     Clean, Preis-fokussiert, Buchungs-CTA dominant

  Mittel-Segment, gute Bilder      salon-gallery     Bildergalerie prominent, Team, Preise, Buchung

  Premium-Salon, Instagram-aktiv   salon-premium     Full-Screen-Visuals, Lifestyle-Feeling, Instagram-Integration

  Barbershop / Nische              salon-niche       Maskulin/edgy Design, Spezial-Services betont
  -------------------------------- ----------------- ---------------------------------------------------------------

Coaches

  --------------------------------------- ----------------- ----------------------------------------------------
  **Bedingung**                           **Template**      **Begründung**

  Solo-Coach, kein Content                coach-landing     Single-Page, Conversion-optimiert, CTA above fold

  Solo-Coach, Blog/Podcast vorhanden      coach-authority   Personal Brand + Content-Hub + Funnel-Elemente

  Coaching-Unternehmen, mehrere Coaches   coach-business    Team-Seite, Programm-Übersicht, Corporate-Elemente

  Speaker/Autor (bekannt)                 coach-celebrity   Event-Kalender, Buch-Promotion, Media-Kit
  --------------------------------------- ----------------- ----------------------------------------------------

Template-Variation (Anti-Template-Look)

Kritisch: Wenn alle Friseure das gleiche Template bekommen, sieht es wie ein Template aus. Dagegen:

1.  Farbschema-Variation: Jede Website bekommt die extrahierten Brand-Farben des Leads (aus Stufe 4). Bei schlechten Brand-Farben: Generierung einer neuen, branchenpassenden Palette basierend auf Industrie + Standort

2.  Layout-Variation: Jedes Template hat 3-4 Hero-Varianten (Bild links, Bild rechts, Full-Screen, Split-Screen). Zufällige oder content-basierte Auswahl

3.  Section-Reihenfolge: Optional-Sections werden in unterschiedlicher Reihenfolge platziert. Testimonials mal nach Services, mal vor dem Team

4.  Font-Pairing-Variation: 5 Font-Pairings pro Template. Auswahl basierend auf extrahierter Typografie oder zufällig

5.  Bild-Layout-Variation: Runde vs. eckige Bilder, Grid vs. Masonry, Overlay vs. clean

Ziel: Keine zwei Websites im selben Stadtgebiet sollen auf den ersten Blick als „gleiche Vorlage" erkennbar sein.

8.3 Seitenstruktur-Entscheidung

Die Seitenstruktur wird bestimmt durch: Required Sections (aus Industry Profile) + Optional Sections (aktiviert wenn Content vorhanden) + Content Completeness (wie voll können wir jede Section befüllen?).

Section-Activation Decision Tree

  ----------------------------- ---------------------------------------------------------- --------------------------------------------------- --------------------------------------------------------------------------------------------------------------
  **Section**                   **Aktiviert wenn**                                         **Content-Minimum**                                 **Bei unzureichendem Content**

  **Hero**                      Immer (Pflicht)                                            Firmenname + Slogan/Beschreibung + CTA              Firmenname + generische Branchenbeschreibung + CTA

  **Services/Leistungen**       Immer (Pflicht)                                            Mind. 3 Services mit je \>30 Wörtern Beschreibung   Services aus Navigation-Labels + kurze AI-generierte Beschreibung (nur aus Fakten)

  **About/Über uns**            Immer (Pflicht)                                            Gründungsjahr + Team/Inhaber + Firmenphilosophie    Minimal-Version: Name + Gründungsjahr + 2 Sätze aus Enrichment-Daten

  **Team**                      Wenn Team-Daten vorhanden                                  Mind. 1 Person mit Name + Rolle                     Section weglassen (nicht mit Platzhaltern füllen)

  **Testimonials**              Wenn \>2 Testimonials oder Google Reviews \>4.0 mit \>10   Mind. 3 Testimonials/Reviews                        Google-Reviews einbinden (mit Sterne-Widget). Wenn auch das fehlt: weglassen

  **Galerie/Portfolio**         Wenn \>5 gute eigene Bilder                                5 relevante, hochauflösende Bilder                  NanoBanana-Bilder MIT Platzhalter-Markierung. Oder: Section weglassen

  **Blog**                      Wenn \>3 Blog-Posts extrahiert                             Mind. 3 Posts mit Titel + Datum                     Section weglassen (leerer Blog wirkt schlimmer als keiner)

  **Preise**                    Wenn Preise auf Website veröffentlicht                     Mind. 3 Leistungen mit Preisen                      Section weglassen (Preise nie erfinden!)

  **FAQ**                       Wenn FAQ auf Website ODER Industry Profile empfiehlt       Mind. 4 Frage-Antwort-Paare                         AI generiert 4-6 branchentypische FAQs (nur allgemeine Branchenfragen, keine firmaspezifischen Behauptungen)

  **Kontakt**                   Immer (Pflicht)                                            Adresse + Telefon + Formular + Maps                 Minimal: Telefon + Email + Formular

  **Impressum + Datenschutz**   Immer (Pflicht)                                            Link zur bestehenden Seite                          Link zur bestehenden Seite (nie selbst generieren)
  ----------------------------- ---------------------------------------------------------- --------------------------------------------------- --------------------------------------------------------------------------------------------------------------

8.4 Content-Entscheidungen pro Section (Decision Tree)

Für jede aktivierte Section wird entschieden was mit dem Content passiert:

  ------------------- ------------------------------------------------------------------------------------------------------------------------------ ---------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Entscheidung**    **Bedingung**                                                                                                                  **Aktion**

  **ÜBERNEHMEN**      Text ist gut geschrieben, factisch korrekt, \>100 Wörter, Flesch \>50, keine Rechtschreibfehler                                Text 1:1 übernehmen. Nur minimale Formatierungsanpassungen

  **OPTIMIEREN**      Text vorhanden aber: zu kurz (\<100 Wörter), schlechte Lesbarkeit (Flesch \<40), Rechtschreibfehler, passiver Stil, kein SEO   Text beibehalten aber: Grammatik fixen, aktiver formulieren, SEO-Keywords einbauen, länger machen (NUR mit vorhandenen Fakten)

  **NEU AUS DATEN**   Kein Text vorhanden, aber Fakten aus Extraction oder Enrichment verfügbar                                                      Neuen Text aus verfügbaren Fakten zusammenstellen. STRENGE Regel: Jeder Satz muss auf einen extrahierten Datenpunkt zurückführbar sein. Kein Satz ohne Quelle

  **PLATZHALTER**     Weder Text noch Fakten vorhanden. Section ist „Nice-to-Have"                                                                   Section mit Platzhalter-Text: „Hier könnte Ihr \[Branchenspezifisch\] stehen. Wir helfen Ihnen gerne beim Texten." -- ODER Section weglassen

  **WEGLASSEN**       Kein Content, keine Fakten, Section ist Optional                                                                               Section nicht einbauen. Weniger ist mehr -- eine leere Section schadet mehr als keine
  ------------------- ------------------------------------------------------------------------------------------------------------------------------ ---------------------------------------------------------------------------------------------------------------------------------------------------------------

+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Die goldene Regel: NICHTS ERFINDEN**                                                                                                                                                                                                                                                                                                                                                                                                           |
|                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| Wenn die Extraction sagt „Dr. Müller hat sich 2008 niedergelassen" → wir dürfen schreiben „Seit 2008 versorgt Dr. Müller Patienten in München." Wenn die Extraction NICHT sagt wann er sich niedergelassen hat → wir dürfen NICHT schreiben „Seit vielen Jahren" oder „Mit langjähriger Erfahrung". Stattdessen: Weglassen oder „Dr. Müller praktiziert in München." Jeder halluzinierte Fakt zerstört die Glaubwürdigkeit der gesamten Preview. |
+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

8.5 Brand-Entscheidung

  --------------------------------------------------------------------------------------------------- ------------------------------------------- --------------------------------------------------------------------------------------------------------------------------------------------------------
  **Bedingung**                                                                                       **Entscheidung**                            **Umsetzung**

  Logo: SVG/PNG \>500px + Professionell Farben: Konsistent + Branchenfit Fonts: Web-Fonts             **KEEP: Bestehendes Branding übernehmen**   Logo, Farben und Fonts 1:1 übernehmen. Nur Website-Layout modernisieren. Lead erkennt seine Marke sofort wieder

  Logo: OK, aber Farben datiert oder inkonsistent ODER: Fonts = System-Fonts                          **UPGRADE: Branding auffrischen**           Logo behalten. Farbpalette modernisieren (bestehende Primärfarbe behalten, Sekundär/Akzent neu). Web-Fonts statt System-Fonts

  Logo: Niedrige Auflösung / unprofessionell Farben: Keine Konsistenz Gesamtbild: Nicht markenfähig   **NEW: Neues Branding vorschlagen**         Neues Farbschema basierend auf Branche. Moderne Font-Pairing. Logo: Bestmögliche Version verwenden + Hinweis „Logo-Redesign optional buchbar" (Upsell)
  --------------------------------------------------------------------------------------------------- ------------------------------------------- --------------------------------------------------------------------------------------------------------------------------------------------------------

Wichtig: Die Brand-Entscheidung „KEEP" ist der sicherste Weg. Wenn der Lead sein Logo und seine Farben auf der neuen Website wiedererkennt, steigt das Vertrauen. Ein komplett neues Branding kann abschreckend wirken („Das ist nicht mein Business"). Daher: Im Zweifel KEEP \> UPGRADE \> NEW.

8.6 SEO-Plan

Keyword-Strategie

1.  Primary Keywords: \[Branche\] + \[Stadt\] → z.B. „Anwalt Familienrecht München". 2-3 Primary Keywords basierend auf Hauptleistungen

2.  Secondary Keywords: Spezifischere Varianten + Stadtteil → z.B. „Scheidungsanwalt Schwabing". 5-8 Secondary Keywords

3.  Long-Tail Keywords: Frage-basiert → z.B. „Was kostet ein Anwalt für Familienrecht in München". Für FAQ-Section und Blog-Ideen

4.  Keyword-Validierung: Suchvolumen prüfen (Google Keyword Planner oder Ubersuggest). Nur Keywords mit \>50 monatlichen Suchen verwenden

On-Page SEO Plan

-   Title-Tag: Primary Keyword + Firmenname. Max 60 Zeichen. Beispiel: „Anwalt Familienrecht München \| Kanzlei Müller"

-   Meta-Description: Unique pro Seite. Leistung + USP + CTA. Max 155 Zeichen

-   H1-Struktur: Eine H1 pro Seite mit Primary Keyword. H2s für Subsections mit Secondary Keywords

-   Schema Markup: Industry-spezifisch automatisch generiert (Attorney, Physician, LocalBusiness, etc.)

-   Local SEO: Google Maps Embed, NAP-Konsistenz (Name, Address, Phone identisch zu Google Business)

-   Sitemap: Automatisch generiert bei Build

-   Canonical-Tags: Automatisch gesetzt

-   Image Alt-Texte: Beschreibend mit Keyword wo natürlich passend

8.7 Legal Requirements Checklist

Pro Industrie wird eine Checkliste erstellt die im QA-Step (Stufe 11) geprüft wird:

  ------------------------------------------------ ---------- ------------- ----------- ------------- -------------------
  **Anforderung**                                  **Alle**   **Anwälte**   **Ärzte**   **Coaches**   **Handwerker**

  Impressum-Link im Footer                         ✓          ✓             ✓           ✓             ✓

  Datenschutz-Link im Footer                       ✓          ✓             ✓           ✓             ✓

  Cookie-Consent (wenn Tracking)                   ✓          ✓             ✓           ✓             ✓

  BRAO-Pflichtangaben                              --         ✓             --          --            --

  HWG-konforme Texte (keine Heilversprechen)       --         --            ✓           --            --

  Meisterpflicht-Angabe                            --         --            --          --            ✓ (wo zutreffend)

  Keine Therapie-Versprechen                       --         --            --          ✓             --

  Keine erfundenen Testimonials                    ✓          ✓             ✓           ✓             ✓

  Preise inkl. MwSt. ausgezeichnet (wenn Preise)   ✓          ✓             ✓           ✓             ✓

  Bildrechte geklärt (keine Fremdbilder)           ✓          ✓             ✓           ✓             ✓
  ------------------------------------------------ ---------- ------------- ----------- ------------- -------------------

8.8 Rebuild Confidence & Approval Gate

  -------------------------------------------------- --------------------------------------------------------- ----------------------------------------------------------
  **Signal**                                         **Score-Beitrag**                                         **Auswirkung bei Low**

  Template-Match Confidence                          +0.15 wenn eindeutiger Match                              Manuelles Review: Passt das Template?

  Content Coverage (% Sections mit echtem Content)   +0.25 wenn \>80% Sections gefüllt                         \<50% → Rebuild könnte dünn wirken

  Bild-Verfügbarkeit                                 +0.15 wenn \>5 gute eigene Bilder                         NanoBanana-Heavy → Platzhalter-Risiko

  Brand-Entscheidung Klarheit                        +0.10 wenn eindeutig KEEP oder UPGRADE                    „NEW" hat höheres Risiko, Lead erkennt sich nicht wieder

  SEO-Keywords mit Suchvolumen                       +0.10 wenn \>3 Keywords mit \>50 Searches                 Ohne Suchvolumen-Daten: SEO-Argument schwächer

  Legal Checklist komplett                           +0.10 wenn alle Pflicht-Items checkable                   Fehlende Legal-Daten → Compliance-Risiko

  Kein Content erfunden                              +0.15 (Negativ-Check: −0.30 wenn Halluzination erkannt)   Größtes Risiko -- zerstört Glaubwürdigkeit
  -------------------------------------------------- --------------------------------------------------------- ----------------------------------------------------------

  ----------------- ------------------------ -----------------------------------------------------
  **Confidence**    **Approval**             **Routing**

  **\> 0.80**       Auto-Approved            Direkt weiter zu Stufe 9 (Content Polish)

  **0.60 - 0.80**   Conditional              Weiter, aber erhöhtes QA in Stufe 11. Flag setzen

  **\< 0.60**       Manual Review Required   In manueller Review-Queue. Nicht automatisch weiter
  ----------------- ------------------------ -----------------------------------------------------

8.9 Error Handling

  --------------------------------- ----------------------------------------------------------------------------- -------------------------------------------------------------------------------------------------------------------------
  **Failure Mode**                  **Erkennung**                                                                 **Fallback**

  Kein passendes Template           Template-Match-Score \< 0.5                                                   Default-Template der Industrie verwenden. Flag für manuelles Review

  Zu wenig Content für Rebuild      Content Coverage \< 40%                                                       Downgrade zu „Light Rebuild": Nur Homepage + Kontakt. Oder: Lead in Nurture-Pool (nur Analyse-Report)

  Brand Assets nicht extrahierbar   Kein Logo oder Farben gefunden                                                Branchenstandard-Farbschema verwenden. Logo-Bereich leer lassen mit Platzhalter

  Legal Requirements unklar         Branchenspezifische Angaben nicht in Extraction                               Konservativ: Alle Pflichtangaben als „zu prüfen" markieren. Impressum/Datenschutz immer zur bestehenden Seite verlinken

  AI-Plan halluziniert Content      Fact-Check gegen Extraction: Text enthält Info die nicht aus Quellen stammt   Gesamten Plan verwerfen und neu generieren mit strengerem Prompt. Nach 2. Halluzination: Manuelles Review
  --------------------------------- ----------------------------------------------------------------------------- -------------------------------------------------------------------------------------------------------------------------

8.10 KPIs & Monitoring

  --------------------------------------------------------------------- ---------------------------------------------------- --------------------------------------------------
  **Metrik**                                                            **Zielwert**                                         **Alert wenn**

  Plan-Generierungsdauer                                                \<90s                                                \>180s

  Auto-Approval-Rate (Confidence \>0.8)                                 \>60% (Phase 2)                                      \<40%

  Manual Review Queue Größe                                             \<20 Leads                                           \>50 (Bottleneck)

  Avg. Content Coverage                                                 \>70%                                                \<50%

  Halluzinations-Rate                                                   \<2%                                                 \>5%

  Template-Variation Score (keine zwei identischen in gleicher Stadt)   100%                                                 \<100% (Duplikat-Template in gleicher Stadt)

  Rebuild → Conversion Correlation                                      Rebuild Confidence \>0.8 = 2x Conversion vs. \<0.6   Keine Korrelation (Plan-Qualität nicht relevant)
  --------------------------------------------------------------------- ---------------------------------------------------- --------------------------------------------------

Nächste Iteration: Steps 9-12

Dieses Dokument deckt Steps 5-8 ab. Die nächste Iteration spezifiziert:

-   Step 9: Content Polish & Fact-Verification -- Banned Phrases, Tone-of-Voice-Guides, Fact-Check-Workflows, das Uncanny-Valley-Problem

-   Step 10: Website Assembly & Build -- Template-Engine, Performance-Budget-Enforcement, Accessibility-Constraints

-   Step 11: QA & Testing -- Automatisierte Test-Suite, Visual Quality AI-Check, Legal Compliance Validation

-   Step 12: Preview Deployment -- Preview als geführte Sales-Experience, Storytelling, emotionale Überzeugung

*Webolution Deep Spec -- Steps 5-8 -- SocialCooks*
