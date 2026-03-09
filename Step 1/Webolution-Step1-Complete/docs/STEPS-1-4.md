**WEBOLUTION**

Deep Spec: Steps 1--4

*Lead Scraping → Data Validation → Enrichment → Crawl & Extraction*

Vollautomatisiert. Fehlerfrei. Epische Qualität.

**SocialCooks × Webolution**

März 2026

Design-Prinzipien für Full Automation

Jeder der 17 Steps wird nach folgenden Prinzipien spezifiziert. Ohne diese Prinzipien bricht die Pipeline beim ersten Sonderfall:

+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Die 7 Prinzipien der Webolution-Pipeline**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
|                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| 1\. Confidence Scoring: Jeder Output bekommt einen Confidence-Wert (0-1). Low-Confidence-Outputs werden anders geroutet als High-Confidence. 2. Graceful Degradation: Wenn ein Sub-Step fehlschlägt, darf nicht die ganze Pipeline stoppen. Fallback-Logik für jeden Failure Mode. 3. Self-Healing: Wenn ein externer Service ausfällt (API down, Scraper blocked), automatische Retry + alternative Quelle + Alert. 4. Human-in-the-Loop Escalation: Wenn Confidence \< Threshold, geht der Lead in eine manuelle Review-Queue statt blind weiter. 5. Idempotenz: Jeder Step kann sicher wiederholt werden ohne Duplikate oder Seiteneffekte. 6. Audit Trail: Jede Entscheidung wird geloggt. Warum hat ein Lead diesen Score? Warum wurde dieses Bild ersetzt? Vollständige Nachvollziehbarkeit. 7. Feedback Loop: Jedes Ergebnis (Conversion, Ablehnung, Bounce) fließt zurück und verbessert die vorgelagerten Steps. |
+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

+-------+-------------------------------------------------------------------------+
| **1** | **Lead Scraping**                                                       |
|       |                                                                         |
|       | *Leads aus branchenspezifischen Quellen sammeln -- die Basis für alles* |
+-------+-------------------------------------------------------------------------+

+-----------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+--------------------------------------------------------------------------------------------------------+
| **INPUT**                                                                                           | **OUTPUT**                                                                                                                                                                              | **QUALITY GATE**                                                                                       |
|                                                                                                     |                                                                                                                                                                                         |                                                                                                        |
| Industry Profile Config: • search_terms\[\] • source_priority\[\] • geo_regions\[\] • geo_radius_km | Raw Lead Record: • firm_name • website_url • address • phone • email_generic • google_place_id • google_rating • google_review_count • source_tag • industry_tag • scraped_at timestamp | Mindestens firm_name + website_url vorhanden. URL-Format valide (http/https). Industry-Tag zugewiesen. |
+-----------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+--------------------------------------------------------------------------------------------------------+

1.1 Scraping-Architektur

Die Scraping-Engine läuft als verteiltes System mit einer Queue pro Industrie-Region-Kombination. Jede Queue ist unabhängig und kann separat skaliert, pausiert oder restarted werden.

Orchestrierung

-   Job-Scheduler (Celery Beat): Startet Scraping-Jobs nach Zeitplan. Jede Industrie-Region-Kombination ist ein eigener Job. Beispiel: „anwaelte-muenchen", „friseure-hamburg", „coaches-berlin"

-   Worker-Pool (Celery Workers): Verarbeitet Jobs parallel. Auto-Scaling basierend auf Queue-Länge. Max 10 concurrent Workers pro Quelle (Rate-Limit-konform)

-   Result Store (PostgreSQL): Jeder gescrapte Lead wird sofort in die DB geschrieben mit source_tag und scraped_at Timestamp

-   Dead Letter Queue: Jobs die 3x fehlschlagen werden in eine DLQ verschoben und lösen einen Alert aus

Scraping-Frequenz

  --------------- --------------------------- -----------------------------------------
  **Industrie**   **Frequenz**                **Begründung**

  Anwälte         1x pro Woche pro Stadt      Kanzleien ändern sich selten

  Ärzte           1x pro Woche pro Stadt      Praxen ändern sich selten

  Coaches         2x pro Woche pro Stadt      Höhere Fluktuation, viele Neugründungen

  Friseure        1x pro Woche pro Stadt      Stabil, aber saisonale Eröffnungen

  Handwerker      1x pro Woche pro Region     Größerer Radius, weniger dicht

  Steuerberater   1x pro 2 Wochen pro Stadt   Sehr stabil, wenig Fluktuation
  --------------- --------------------------- -----------------------------------------

1.2 Quellen-spezifische Scraping-Logik

Google Maps / Places API

Primäre Quelle für alle Industrien. Höchste Datenqualität und Abdeckung.

1.  Nearby Search: Suche nach Industry-Keyword + Geo-Center + Radius. Beispiel: „Anwalt" in 48.1351,11.5820 (München) mit 30km Radius

2.  Pagination: Google Maps gibt max 60 Ergebnisse pro Suche. Bei \>60 erwarteten Ergebnissen: Grid-basierte Suche (Stadt in Quadranten aufteilen, pro Quadrant suchen)

3.  Place Details: Für jeden Treffer Place Details API aufrufen: Name, Adresse, Website, Telefon, Öffnungszeiten, Rating, Review-Count, Fotos, Place-ID

4.  Category Verification: Google-Kategorie muss zum Industry-Tag passen. „Law firm" = ok für Anwälte, „Accounting firm" = falsche Industrie → skip

5.  Cost Budgeting: Places API kostet \~\$17 pro 1.000 Calls. Budget pro Industrie-Region festlegen und tracken

+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Failure Mode: Google API Quota/Rate Limit**                                                                                                                                                                                                                           |
|                                                                                                                                                                                                                                                                         |
| Erkennung: HTTP 429 oder OVER_QUERY_LIMIT Response. Fallback: Exponential Backoff (1s, 2s, 4s, 8s, max 60s). Nach 5 Retries: Job pausieren, nächste Quelle verwenden. Prevention: Request-Rate auf 80% des Quotas begrenzen. Quota-Usage-Dashboard mit Alert bei \>90%. |
+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

Branchenportale (Anwalt.de, Jameda, Treatwell, etc.)

Sekundäre Quellen mit branchenspezifischen Zusatzdaten. Scraping statt API (keine offenen APIs verfügbar).

-   Technologie: Playwright mit Stealth-Plugin (undetected-playwright). Rotierender Proxy-Pool (mind. 50 IPs)

-   Anti-Bot-Maßnahmen: Randomisierte Delays (2-8s zwischen Requests), Human-like Scrolling, Cookie-Akzeptierung automatisiert, Fingerprint-Randomisierung

-   Daten-Extraktion: CSS-Selektoren für strukturierte Daten. Fallback auf XPath. Fallback auf AI-Extraktion (Claude) wenn Seitenstruktur sich geändert hat

-   Incremental Scraping: Nur neue/geänderte Profile scrapen. Hash-Vergleich des Profil-Contents mit letztem Scrape

+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Failure Mode: Branchenportal ändert HTML-Struktur**                                                                                                                                                                                                                                                                                                            |
|                                                                                                                                                                                                                                                                                                                                                                  |
| Erkennung: Extraction liefert \<50% der erwarteten Felder für \>10 aufeinanderfolgende Profile. Fallback: Automatischer Switch auf AI-Extraktion (Claude analysiert Screenshot des Profils). Alert an Team: „Selektor-Update nötig für \[Portal\]". Prevention: Monatlicher Selector-Health-Check. Redundante Selektoren (CSS + XPath + AI als Triple-Fallback). |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

Gelbe Seiten / Das Örtliche

-   Nur als Fallback wenn Google Maps + Branchenportal nicht genug Leads liefern

-   Scraping über Suchfunktion: Branche + Stadt

-   Liefert: Name, Adresse, Telefon, Website (oft unvollständig)

-   Niedrigste Datenqualität: Viele veraltete Einträge, geschlossene Geschäfte

LinkedIn (optional, nur für Entscheider-Identifikation)

-   NICHT für Lead-Scraping verwenden (LinkedIn ToS-Risiko für Mass-Scraping)

-   Nur gezielt für Decision Maker Identification in Stufe 3

-   Via LinkedIn Sales Navigator API (bezahlter Account) oder manuelle Recherche

Instagram (Coaches, Friseure)

-   Bio-Link-Check: Hat der Account eine Website verlinkt? Wenn ja, als Lead-Quelle nutzen

-   Follower-Count als Enrichment-Signal (für Stufe 3)

-   Nur öffentliche Profile, kein Scraping privater Accounts

-   Instagram Graph API für Business-Profile (wo möglich) statt Scraping

1.3 Multi-Source-Merge & Deduplizierung

Ein Friseur kann bei Google Maps, Treatwell, Instagram und Gelbe Seiten auftauchen. Diese Einträge müssen zu einem einzigen Lead zusammengeführt werden.

Matching-Algorithmus

1.  Domain-Match (höchste Priorität): Website-URL normalisieren (lowercase, www entfernen, trailing slash entfernen, http/https ignorieren). Exakter Match = selber Lead

2.  Phone-Match: Telefonnummer normalisieren (+49 Format). Exakter Match = sehr wahrscheinlich selber Lead

3.  Name+Address Match: Firmenname Fuzzy-Match (Levenshtein-Distanz \>0.85) UND Adresse in gleicher Straße = wahrscheinlich selber Lead

4.  Name-Only Match (niedrigste Priorität): Nur bei sehr spezifischen Namen. „Kanzlei Dr. Müller-Schmidt" = wahrscheinlich unique. „Friseur am Markt" = zu generisch, kein Match

Merge-Strategie

Wenn ein Lead über mehrere Quellen gefunden wird, werden die Daten zusammengeführt. Bei Konflikten gewinnt die Quelle mit höherer Priorität:

  ------------------- ------------------------------ ---------------------------- --------------------------
  **Feld**            **Prio 1: Google Maps**        **Prio 2: Branchenportal**   **Prio 3: Gelbe Seiten**

  Firmenname          ✓ (offiziellster Name)         Nur wenn Google fehlt        Nur als letzter Fallback

  Adresse             ✓ (verifiziert durch Google)   ✓ (kann aktueller sein)      Oft veraltet

  Telefon             ✓                              ✓                            ✓

  Website             ✓                              ✓                            Oft fehlend

  Email               Selten vorhanden               Manchmal im Profil           Manchmal vorhanden

  Öffnungszeiten      ✓ (höchste Aktualität)         Selten                       Selten

  Bewertungen         ✓ (Google Reviews)             ✓ (Jameda, Anwalt.de)        Keine

  Spezialisierungen   Nur Kategorie                  ✓ (detaillierteste Daten)    Nur Kategorie
  ------------------- ------------------------------ ---------------------------- --------------------------

1.4 Industrie-spezifische Scraping-Parameter

+-----------------------+------------------+------------------------+------------------+-----------------------+-------------------+
| **Parameter**         | **Anwälte**      | **Ärzte**              | **Coaches**      | **Friseure**          | **Handwerker**    |
+-----------------------+------------------+------------------------+------------------+-----------------------+-------------------+
| **search_terms**      | Anwalt           | Arzt                   | Coach            | Friseur               | Handwerker        |
|                       |                  |                        |                  |                       |                   |
|                       | Kanzlei          | Praxis                 | Coaching         | Friseursalon          | Elektriker        |
|                       |                  |                        |                  |                       |                   |
|                       | Rechtsanwalt     | Ärztin                 | Berater          | Haarsalon             | Klempner          |
|                       |                  |                        |                  |                       |                   |
|                       | Notar            | Facharzt               | Trainer          | Barbershop            | Maler             |
|                       |                  |                        |                  |                       |                   |
|                       | Fachanwalt       | Zahnarzt               | Mentor           | Coiffeur              | Dachdecker        |
+-----------------------+------------------+------------------------+------------------+-----------------------+-------------------+
| **geo_radius_km**     | 30               | 20                     | 50 (oft remote)  | 15                    | 50                |
+-----------------------+------------------+------------------------+------------------+-----------------------+-------------------+
| **min_google_rating** | 3.0              | 3.0                    | Kein Minimum     | 3.5                   | 3.0               |
+-----------------------+------------------+------------------------+------------------+-----------------------+-------------------+
| **min_reviews**       | 3                | 5                      | 0 (viele neue)   | 5                     | 3                 |
+-----------------------+------------------+------------------------+------------------+-----------------------+-------------------+
| **exclude_chains**    | Nein             | Ja (Helios, Asklepios) | Nein             | Ja (Klier, Super Cut) | Ja (HomeServe)    |
+-----------------------+------------------+------------------------+------------------+-----------------------+-------------------+
| **expected_yield**    | 50-200 pro Stadt | 100-500 pro Stadt      | 20-100 pro Stadt | 100-400 pro Stadt     | 50-300 pro Region |
+-----------------------+------------------+------------------------+------------------+-----------------------+-------------------+

1.5 Error Handling & Self-Healing

  ---------------------------- --------------------------------- ------------------------------------------------------- -----------------------------------
  **Failure Mode**             **Erkennung**                     **Automatische Reaktion**                               **Eskalation wenn**

  Google API Down              HTTP 5xx für \>5 Min              Pause Scraping, Retry in 15 Min, Fallback auf Cache     Down \>2h

  Scraper blockiert (Portal)   Captcha-Detection, 403 Response   IP rotieren, Delay erhöhen, Stealth-Settings anpassen   Blockiert \>24h trotz Rotation

  Portal Redesign              Extraction Rate \<50%             Switch auf AI-Extraktion                                AI-Extraction auch \<50%

  Yield Drop                   Leads/Job \<30% des Erwarteten    Alternative Search Terms, Radius erhöhen                Drop \>3 aufeinanderfolgende Runs

  Duplikat-Explosion           Duplikat-Rate \>25%               Matching-Threshold verschärfen                          Rate \>25% nach Anpassung

  API-Kosten-Spike             Daily Cost \>150% des Budgets     Pagination limitieren, Low-Prio-Jobs pausieren          3 Tage über Budget
  ---------------------------- --------------------------------- ------------------------------------------------------- -----------------------------------

1.6 Confidence Scoring für Raw Leads

Jeder Raw Lead bekommt einen Data Completeness Score (0-1), der angibt wie vollständig und zuverlässig die gescrapten Daten sind:

  ----------------------------- ------------- --------------------------------------------- ---------------------------
  **Feld**                      **Gewicht**   **Score wenn vorhanden**                      **Score wenn fehlend**

  firm_name                     0.15          1.0 (immer vorhanden durch Quelle)            0.0 → Lead wird verworfen

  website_url                   0.20          1.0 wenn URL-Format valide                    0.0 → Lead wird verworfen

  address                       0.10          1.0 wenn vollständig (Straße + PLZ + Stadt)   0.3 wenn nur Stadt

  phone                         0.10          1.0 wenn DE-Nummer erkannt                    0.0

  email_generic                 0.10          1.0 wenn Format valide                        0.0

  google_place_id               0.10          1.0 (ermöglicht Enrichment)                   0.0

  google_rating                 0.10          1.0                                           0.0

  google_review_count           0.05          1.0                                           0.0

  source_count (Multi-Source)   0.10          1.0 wenn ≥2 Quellen                           0.5 wenn 1 Quelle
  ----------------------------- ------------- --------------------------------------------- ---------------------------

  ----------------- ------------------- ---------------------------------------------------------------------------------------------------------------------
  **Score Range**   **Kategorie**       **Routing**

  **0.8 - 1.0**     High Confidence     Direkt weiter zu Stufe 2

  **0.5 - 0.79**    Medium Confidence   Weiter zu Stufe 2, aber mit Enrichment-Priority (mehr Daten sammeln)

  **\< 0.5**        Low Confidence      In „Nachrecherche-Queue": Automatischer Retry mit alternativen Quellen. Nach 2 Retries ohne Verbesserung: verwerfen
  ----------------- ------------------- ---------------------------------------------------------------------------------------------------------------------

1.7 KPIs & Monitoring

  --------------------------------------- --------------------------- ---------------------- -------------------
  **Metrik**                              **Zielwert**                **Alert wenn**         **Dashboard**

  Raw Leads / Woche / Industrie / Stadt   \>25 (MVP), \>200 (Scale)   \<50% des Erwarteten   Pipeline Overview

  Source Health (pro Quelle)              \>90% Success Rate          \<70%                  Source Monitor

  Duplikat-Rate                           \<10%                       \>20%                  Data Quality

  Data Completeness Score (Avg)           \>0.7                       \<0.5                  Data Quality

  Cost per Raw Lead                       \<€0.05                     \>€0.15                Cost Dashboard

  Scraping-Dauer pro Job                  \<10 Min                    \>30 Min               Performance
  --------------------------------------- --------------------------- ---------------------- -------------------

+-------+-----------------------------------------------------------------------+
| **2** | **Data Validation & Cleaning**                                        |
|       |                                                                       |
|       | *Datenqualität sichern bevor Enrichment-Ressourcen ausgegeben werden* |
+-------+-----------------------------------------------------------------------+

+----------------------------------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------+
| **INPUT**                                                | **OUTPUT**                                                                                                                                 | **QUALITY GATE**                                                                          |
|                                                          |                                                                                                                                            |                                                                                           |
| Raw Lead Records aus Stufe 1 mit Data Completeness Score | Validated Lead Records: • Alle Felder normalisiert • URL erreichbar • Duplikate entfernt • Blacklist geprüft • Validation-Confidence (0-1) | URL erreichbar (HTTP 200/301). Kein Duplikat in DB. Nicht auf Blacklist. Kein Competitor. |
+----------------------------------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------+

2.1 Warum ein eigener Step?

+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Kosten-Argument**                                                                                                                                                                                                                                                                                                                                                                                                         |
|                                                                                                                                                                                                                                                                                                                                                                                                                             |
| Stufe 3 (Enrichment) verwendet bezahlte APIs: Google Places Details (\~€0.015/Call), Northdata, LinkedIn. Stufe 4 (Crawling) kostet Compute-Zeit und Claude API Calls (\~€0.03-0.10 pro Website-Analyse). Bei 1.000 Raw Leads/Woche mit 20% Müll spart Validation \~200 unnötige Enrichment-Calls (€3-15/Woche) und \~200 unnötige Crawls (€6-20/Woche). Über Scale: Bei 10.000 Leads/Woche spart Validation €90-350/Woche. |
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

2.2 Validierungs-Pipeline (sequentiell)

Jeder Check ist ein eigener Sub-Step. Ein Lead muss ALLE Checks bestehen. Bei Failure wird der Grund geloggt und der Lead aussortiert oder in eine Retry-Queue geschoben.

Check 1: URL-Format-Validierung

-   Was: URL-String ist syntaktisch korrekt. Schema vorhanden (http/https). Domain hat gültige TLD

-   Wie: URL-Parser (Python urllib). Regex für gängige Fehler: fehlende Punkte, Leerzeichen, Sonderzeichen

-   Normalisierung: Lowercase, www. entfernen, trailing slash entfernen, http → https wenn möglich

-   Fail-Aktion: Ungültige URL → verwerfen (nicht reparabel)

Check 2: URL-Erreichbarkeit

-   Was: HTTP HEAD Request an die URL. Erwartung: 200, 301, 302

-   Wie: Async HTTP Client (httpx) mit 10s Timeout. Follow Redirects (max 5). User-Agent: Standard-Browser-String

-   Handling von Redirects: Finale URL speichern (kann anders sein als gescrapte URL)

```{=html}
<!-- -->
```
-   301/302 zu anderer Domain → Neue Domain speichern, alten Entry updaten

-   301/302 zu Subdomain → OK, Subdomain speichern

-   301/302 zu Social Media (facebook.com/firmenseite) → Kein eigenständiger Web-Auftritt, trotzdem behalten aber flaggen

```{=html}
<!-- -->
```
-   Retry-Logik: Bei Timeout oder 5xx: 3 Retries mit 5s, 15s, 30s Delay. Nach 3 Fails: „unreachable" markieren

+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Failure Modes & Handling**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
|                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| HTTP 403 Forbidden: Manche Websites blockieren Bots. Retry mit Browser-UA. Wenn weiterhin 403: Lead behalten, aber flaggen als „bot-protected" -- Stufe 4 (Crawling) muss mit vollem Browser crawlen. HTTP 5xx: Server-Problem. Retry später. Nach 3 Fails über 24h: Lead behalten, Crawling später retrien. Connection Timeout: Server antwortet nicht. Oft überlastete Billig-Hoster. Retry mit längerem Timeout (30s). SSL Error: Ungültiges/abgelaufenes Zertifikat. Lead behalten -- das IST eine Schwachstelle die wir in der Analyse nutzen können. Domain Parking/For Sale: Erkennung über bekannte Parking-Seiten-Patterns (Sedo, GoDaddy Parking, etc.). Lead verwerfen. |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

Check 3: Duplikat-Erkennung

Prüfung gegen die gesamte Lead-Datenbank, nicht nur den aktuellen Batch:

1.  Exakter Domain-Match: Normalisierte Domain gegen alle existierenden Leads prüfen. Bei Match: Merge-Logik aus Stufe 1.3 anwenden (Daten ergänzen statt neuen Lead erstellen)

2.  Fuzzy-Name-Match: Firmenname normalisieren (Lowercase, Rechtsform entfernen: GmbH, AG, e.K., UG). Levenshtein-Distanz \>0.85 UND gleiche Stadt = wahrscheinlich Duplikat

3.  Phone-Match: Normalisierte Telefonnummer gegen DB prüfen. Exakter Match = Duplikat (sofern nicht Shared-Number wie Gebäude-Zentrale)

4.  Manuelles Review bei Unsicherheit: Wenn Fuzzy-Match zwischen 0.75-0.85: In Review-Queue zur manuellen Entscheidung

Check 4: Blacklist & Suppression

-   Opt-out-Liste: Domain oder Email auf Opt-out-Liste? → Sofort verwerfen. Nie wieder kontaktieren

-   Cooling-Period: Lead wurde kontaktiert und hat nicht reagiert oder abgelehnt? → 12-Monate Cooling. Datum prüfen

-   Bounce-Liste: Email-Adresse hat in der Vergangenheit gebounced? → Email-Feld löschen, Lead behalten (Kontaktweg ändern)

-   Beschwerde-Liste: Lead hat sich bei Instantly beschwert (Spam-Report)? → Permanent verwerfen + Domain blacklisten

Check 5: Competitor & Non-Target Exclusion

Leads die keine Zielkunden sind, müssen rausgefiltert werden:

  ----------------------------------------- -------------------------------------------------------------------------------------------------- -----------------------------------------------------
  **Kategorie**                             **Erkennungs-Logik**                                                                               **Aktion**

  Webdesign-Agenturen                       Google-Kategorie „Web design", Keywords in Name/URL: „webdesign", „agentur", „media", „creative"   Verwerfen (Competitor)

  Marketing-Agenturen                       Google-Kategorie „Marketing agency", Keywords: „marketing", „SEO", „ads"                           Verwerfen (Competitor)

  Freelancer/Einzelpersonen ohne Business   Kein Google Business Eintrag UND keine Adresse UND kein Telefon                                    Verwerfen (zu klein / nicht ernsthaft)

  Ketten/Franchise                          Name auf Chain-Liste (Filialketten pro Industrie)                                                  Verwerfen (Entscheidung nicht lokal)

  Falsche Industrie                         Google-Kategorie passt nicht zum Industry-Tag                                                      In richtige Industrie-Queue umrouten oder verwerfen
  ----------------------------------------- -------------------------------------------------------------------------------------------------- -----------------------------------------------------

Check 6: Daten-Normalisierung

-   Telefonnummer: In +49-Format konvertieren. Vorwahl validieren. Mobilnummern von Festnetz unterscheiden

-   Adresse: Straße, PLZ, Stadt extrahieren und normalisieren. PLZ-Validierung gegen offizielle PLZ-Datenbank

-   Firmenname: Rechtsform erkennen und separieren (firm_name vs. legal_form). Sonderzeichen bereinigen

-   Email: Lowercase, Format-Validierung. „info@", „kontakt@" als generisch markieren vs. persönliche Email

-   Website: Finale URL nach Redirect-Following als canonical_url speichern

2.3 Validation Confidence Score

Nach allen Checks bekommt jeder Lead einen Validation Confidence Score, der die Zuverlässigkeit der Daten widerspiegelt:

  ----------------------- ------------------------------------ --------------------------------------------------
  **Signal**              **Positiv (+)**                      **Negativ (-)**

  URL-Check               +0.2 wenn 200 ohne Redirect          -0.1 wenn nur via Redirect, -0.15 wenn SSL-Error

  Multi-Source            +0.15 wenn ≥2 Quellen matchen        0 wenn nur 1 Quelle

  Google Place ID         +0.15 wenn vorhanden                 0 wenn nicht vorhanden

  Daten-Vollständigkeit   +0.2 wenn alle Felder vorhanden      Proportional weniger bei fehlenden Feldern

  Phone validiert         +0.1 wenn DE-Nummer Format ok        0 wenn fehlend

  Address vollständig     +0.1 wenn Straße+PLZ+Stadt           -0.05 wenn nur Stadt

  Daten-Frische           +0.1 wenn Google-Update \<6 Monate   -0.1 wenn \>2 Jahre kein Update
  ----------------------- ------------------------------------ --------------------------------------------------

  --------------- -------------------------------------------------------------------------- -----------------------
  **Score**       **Routing**                                                                **Erwarteter Anteil**

  **\> 0.8**      High Confidence → Direkt weiter zu Stufe 3                                 \~60%

  **0.5 - 0.8**   Medium Confidence → Weiter, aber mit Enrichment-Fokus auf fehlende Daten   \~25%

  **\< 0.5**      Low Confidence → Nachrecherche-Queue oder verwerfen                        \~15%
  --------------- -------------------------------------------------------------------------- -----------------------

2.4 KPIs & Monitoring

  -------------------------------- ------------------- --------------------------------------------------------------------
  **Metrik**                       **Zielwert**        **Alert wenn**

  Validation Pass Rate             75-85%              \<65% (zu viel Müll aus Scraping) oder \>95% (Validation zu lasch)

  Duplikat-Rate                    5-10%               \>20% (Scraping-Quellen überlappen zu stark)

  Dead-URL-Rate                    5-8%                \>15% (Quellen liefern veraltete Daten)

  Blacklist-Hit-Rate               Steigt über Zeit    \>30% in einer Region (Markt gesättigt)

  Avg. Validation-Dauer pro Lead   \<2s                \>10s (Performance-Problem)

  Competitor-Exclusion-Rate        2-5%                \>10% (Scraping-Keywords zu breit)
  -------------------------------- ------------------- --------------------------------------------------------------------

+-------+------------------------------------------------------------------+
| **3** | **Lead Enrichment & Pre-Qualification**                          |
|       |                                                                  |
|       | *Datenanreicherung + Entscheider-Identifikation + Zombie-Filter* |
+-------+------------------------------------------------------------------+

+-------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------+
| **INPUT**                                             | **OUTPUT**                                                                                                                                                                                                           | **QUALITY GATE**                                                                          |
|                                                       |                                                                                                                                                                                                                      |                                                                                           |
| Validated Lead Record mit Validation Confidence Score | Enriched Lead: • Alle Validated-Felder + • business_health_score (0-100) • decision_maker{name, role, email} • social_profiles{} • competitor_place_ids\[\] • industry_specific_data{} • enrichment_confidence (0-1) | Business Health \> 30. Mindestens 1 Kontaktweg zum Entscheider. Industry-Tag verifiziert. |
+-------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------+

3.1 Enrichment-Quellen & API-Strategie

Enrichment ist teuer. Jeder API-Call kostet Geld. Deshalb werden Quellen in einer Wasserfall-Logik abgefragt: Günstigste zuerst, teurere nur wenn nötig.

Wasserfall-Reihenfolge

1.  Google Places Details API (€0.015/Call) -- Liefert 80% der nötigen Daten: Rating, Reviews, Öffnungszeiten, Fotos, Kategorie, Attribute. Immer aufrufen wenn place_id vorhanden

2.  Google Business Profile (kostenlos, via Scraping) -- Posts, Q&A, Updates, Fotos. Zeigt Business-Aktivität

3.  Branchenportal-Profil (Scraping, „kostenlos") -- Spezialisierungen, Zusatzdaten, Portal-Bewertungen. Nur aufrufen wenn Industry Profile eine Portal-Quelle definiert

4.  Website-Impressum (Scraping, „kostenlos") -- Quick-Scrape NUR des Impressums für: vollständiger Name, Rechtsform, Handelsregister-Nummer, Email, Geschäftsführer-Name. Noch kein Full-Crawl (das ist Stufe 4)

5.  Handelsregister / Northdata (€0.05-0.20/Abfrage) -- Firmenstatus, Gründungsjahr, Größe, Branche. Nur aufrufen wenn Zahlungsfähigkeits-Signal nötig (Firmengröße unklar)

6.  LinkedIn (Sales Navigator, €0.10-0.50/Lookup) -- Nur für Entscheider-Identifikation wenn Impressum keinen Namen liefert. Teuerste Quelle, sparsam einsetzen

+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Cost Cap pro Lead**                                                                                                                                                                                                                                                                                                                           |
|                                                                                                                                                                                                                                                                                                                                                 |
| Enrichment-Budget pro Lead: max €0.30. Bei High-Confidence-Leads aus Stufe 2 (viele Daten schon da): Enrichment kostet \~€0.02-0.05 (nur Google Details). Bei Low-Confidence-Leads (wenig Daten): Enrichment kann bis €0.25 kosten (alle Quellen). Wenn Budget-Cap erreicht: Lead mit vorhandenen Daten weiter-routen, auch wenn unvollständig. |
+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

3.2 Business Health Score (0-100)

Der Business Health Score beantwortet: Ist dieses Unternehmen aktiv, erfolgreich und zahlungsfähig genug um €990+ für eine Website auszugeben?

  --------------------------- ------------- ----------------------------------------------------------------------------- -----------------------
  **Signal**                  **Gewicht**   **Scoring-Logik**                                                             **Datenquelle**

  Google Rating               20%           4.5+ = 100, 4.0-4.4 = 80, 3.5-3.9 = 60, 3.0-3.4 = 40, \<3.0 = 10              Google Places

  Review Count                15%           \>50 = 100, 20-50 = 80, 10-19 = 60, 5-9 = 40, \<5 = 20                        Google Places

  Review Recency              15%           Letzte Review \<1 Monat = 100, \<3M = 80, \<6M = 60, \<12M = 40, \>12M = 10   Google Places

  Google Business Aktivität   10%           Posts \<1M = 100, \<3M = 60, keine Posts = 20                                 Google Business

  Firmenstatus                15%           Aktiv = 100, Unbekannt = 50, Aufgelöst/Insolvent = 0 (DQ)                     Handelsregister

  Firmenalter                 10%           \>10J = 100, 5-10J = 80, 2-5J = 60, \<2J = 40                                 HR / WHOIS

  Social Media Präsenz        10%           Aktiv auf ≥2 Plattformen = 100, 1 = 60, keine = 30                            Instagram/FB/LinkedIn

  Standort-Qualität           5%            Innenstadt/Premium = 100, Vorstadt = 70, Gewerbegebiet = 50                   Adresse + PLZ-Analyse
  --------------------------- ------------- ----------------------------------------------------------------------------- -----------------------

  ------------ ------------------ -----------------------------------------------------------------------------
  **Score**    **Kategorie**      **Aktion**

  **70-100**   Healthy Business   Weiter -- lohnender Kunde mit hoher Wahrscheinlichkeit

  **40-69**    Moderate           Weiter -- aber Deal-Potential in Stufe 7 (Scoring) niedriger gewichten

  **15-39**    Struggling         Nur weiter wenn Website-Schwäche extrem hoch (Score \>85). Sonst: verwerfen

  **0-14**     Dead / Zombie      Disqualifiziert. Nicht kontaktieren
  ------------ ------------------ -----------------------------------------------------------------------------

3.3 Decision Maker Identification

Der kritischste Sub-Step im gesamten Enrichment. An die falsche Person schreiben = 0% Conversion. An den Entscheider persönlich = 3-5x höhere Open Rate.

Identifikations-Wasserfall

1.  Impressum-Parse (Confidence: 0.9): Geschäftsführer, Inhaber, verantwortlich nach §5 TMG. Liefert fast immer den Namen. Email manchmal vorhanden

2.  Google Business Owner-Name (Confidence: 0.7): Manchmal im Google Business Profil sichtbar. Verifizieren gegen Impressum

3.  Branchenportal-Profil (Confidence: 0.8): Anwalt.de zeigt Anwaltsnamen, Jameda zeigt Arztnamen. Oft inkl. Foto und Spezialisierung

4.  LinkedIn-Suche (Confidence: 0.6): Suche nach Firmenname + Rolle. Problem: Matching-Unsicherheit (ist das der richtige Müller?). Nur nutzen wenn Schritte 1-3 keinen Namen liefern

5.  Fallback: Generische Adresse (Confidence: 0.3): info@, kontakt@, kanzlei@, praxis@ -- funktioniert, aber deutlich niedrigere Response Rate

Entscheider-Profil erstellen

  ----------------------- ---------------------------- ----------------------------- ---------------------------------
  **Feld**                **Quelle**                   **Pflicht?**                  **Verwendung**

  Name (Vor + Nachname)   Impressum / Branchenportal   Ja (für persönliche Anrede)   Email-Personalisierung

  Rolle / Titel           Impressum / LinkedIn         Nein                          Tone-Anpassung (Dr., Prof., RA)

  Persönliche Email       Impressum / LinkedIn         Nein (Nice-to-have)           Primärer Kontaktweg

  Generische Email        Impressum / Website          Ja (Fallback)                 Sekundärer Kontaktweg

  Telefon direkt          Impressum                    Nein                          Für Follow-up (Stufe 14)

  LinkedIn-Profil-URL     LinkedIn-Suche               Nein                          Multi-Channel-Outreach

  Foto                    Branchenportal / Website     Nein                          Rebuild-Planung (Stufe 8)
  ----------------------- ---------------------------- ----------------------------- ---------------------------------

+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Failure Mode: Kein Entscheider identifizierbar**                                                                                                                                                                                                                                                                                                                                                                                                              |
|                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| Situation: Impressum enthält nur Firmennamen ohne Person (bei GmbHs legal möglich), kein Branchenportal-Profil, LinkedIn liefert nichts. Fallback-Strategie: Generische Email verwenden (info@) ABER Email-Betreff so formulieren, dass Weiterleitung wahrscheinlich ist: „Für den Geschäftsführer: Ihre Website \[Firmenname\]". Conversion Impact: \~50% niedrigere Open Rate vs. persönliche Ansprache. Lead behalten, aber Erwartungen im Scoring anpassen. |
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

3.4 Industrie-spezifisches Enrichment

Jede Industrie braucht zusätzliche Datenpunkte die nur für diese Branche relevant sind:

+---------------+---------------------------------------+-----------------------+---------------------------------------------------------------------------------+
| **Industrie** | **Zusätzliche Daten**                 | **Quelle**            | **Warum relevant**                                                              |
+---------------+---------------------------------------+-----------------------+---------------------------------------------------------------------------------+
| Anwälte       | Rechtsgebiete                         | Anwalt.de             | Bestimmt Template-Wahl (Solo vs. Kanzlei) und Content-Strategie                 |
|               |                                       |                       |                                                                                 |
|               | Fachanwaltstitel                      | BRAK-Verzeichnis      |                                                                                 |
|               |                                       |                       |                                                                                 |
|               | Kanzleigröße (Anzahl Anwälte)         | Website-Impressum     |                                                                                 |
|               |                                       |                       |                                                                                 |
|               | DAV/BRAK-Mitgliedschaft               |                       |                                                                                 |
+---------------+---------------------------------------+-----------------------+---------------------------------------------------------------------------------+
| Ärzte         | Fachrichtung(en)                      | Jameda                | Unterscheidet Einzelpraxis, Gemeinschaftspraxis, MVZ -- völlig andere Templates |
|               |                                       |                       |                                                                                 |
|               | Kassenzulassung (GKV/Privat)          | KV-Verzeichnis        |                                                                                 |
|               |                                       |                       |                                                                                 |
|               | Praxisgröße (Einzel/Gemeinschaft/MVZ) | Doctolib              |                                                                                 |
|               |                                       |                       |                                                                                 |
|               | Jameda-Score                          |                       |                                                                                 |
+---------------+---------------------------------------+-----------------------+---------------------------------------------------------------------------------+
| Coaches       | Coaching-Art (Business/Life/Health)   | Website               | Coach mit Podcast + 5k Followern aber schlechter Website = Top-Lead             |
|               |                                       |                       |                                                                                 |
|               | Ausbildungen/Zertifikate              | LinkedIn              |                                                                                 |
|               |                                       |                       |                                                                                 |
|               | Podcast/YouTube vorhanden?            | YouTube/Spotify-Suche |                                                                                 |
|               |                                       |                       |                                                                                 |
|               | Lead Magnet/Funnel existiert?         |                       |                                                                                 |
+---------------+---------------------------------------+-----------------------+---------------------------------------------------------------------------------+
| Friseure      | Preissegment (Budget/Mittel/Premium)  | Treatwell             | Premium-Salon mit 2k Followern aber Website von 2016 = idealer Lead             |
|               |                                       |                       |                                                                                 |
|               | Instagram-Follower                    | Instagram             |                                                                                 |
|               |                                       |                       |                                                                                 |
|               | Booking-Tool vorhanden?               | Website               |                                                                                 |
|               |                                       |                       |                                                                                 |
|               | Produktmarken                         |                       |                                                                                 |
+---------------+---------------------------------------+-----------------------+---------------------------------------------------------------------------------+
| Handwerker    | Gewerke / Spezialisierung             | MyHammer              | Meisterbetrieb mit Spezialisierung = höhere Zahlungsbereitschaft                |
|               |                                       |                       |                                                                                 |
|               | Meisterbetrieb ja/nein                | HWK-Verzeichnis       |                                                                                 |
|               |                                       |                       |                                                                                 |
|               | Einzugsgebiet                         | Website               |                                                                                 |
|               |                                       |                       |                                                                                 |
|               | Notdienst-Verfügbarkeit               |                       |                                                                                 |
+---------------+---------------------------------------+-----------------------+---------------------------------------------------------------------------------+

3.5 Pre-Qualification Decision Tree

Am Ende von Stufe 3 wird jeder Lead durch einen Decision Tree geroutet:

  ------------------------------------------------------------------- -------------- -------------------------------------------------------------------
  **Bedingung**                                                       **Ergebnis**   **Nächster Schritt**

  Business Health \< 15                                               DISQUALIFIED   Verwerfen. Log: „Zombie/Dead Business"

  Business Health 15-39 UND keine extreme Website-Schwäche            DISQUALIFIED   Verwerfen. Log: „Business zu schwach für Investment"

  Kein Kontaktweg (weder Email noch Telefon)                          DEFERRED       In Nachrecherche-Queue. Retry in 1 Woche mit alternativen Quellen

  Industrie-Tag nicht verifizierbar                                   REROUTE        In „Unkategorisiert"-Queue für manuelles Review

  Business Health ≥ 40 + mind. 1 Kontaktweg + Industrie verifiziert   QUALIFIED      Weiter zu Stufe 4: Website Crawl & Extraction
  ------------------------------------------------------------------- -------------- -------------------------------------------------------------------

3.6 Enrichment Confidence Score

  --------------------------------------------------------- ---------------------------
  **Signal**                                                **Score-Beitrag**

  Google Place ID vorhanden + Details abgerufen             +0.20

  Entscheider-Name identifiziert (aus Impressum)            +0.20

  Persönliche Email des Entscheiders gefunden               +0.15

  Business Health Score \> 70                               +0.15

  Industrie-spezifische Daten vollständig                   +0.15

  Multi-Source-Verifikation (≥2 Quellen bestätigen Daten)   +0.10

  Social Media Profile gefunden                             +0.05
  --------------------------------------------------------- ---------------------------

Routing: High Confidence (\>0.75) → Full Automation weiter. Medium (0.50-0.75) → Automation mit erhöhtem QA in späteren Steps. Low (\<0.50) → Manuelles Review bevor Pipeline weitergeht.

3.7 Error Handling & Fallbacks

  ------------------------------ ------------------------------------------ -------------------------------------------------------------------------------------- ---------------------------------------
  **Failure Mode**               **Erkennung**                              **Fallback**                                                                           **Eskalation**

  Google Places API Fehler       HTTP Error oder leere Response             Cache verwenden wenn \<7 Tage alt. Sonst: Lead queuen für Retry                        API down \>4h

  Impressum nicht parsebar       AI-Extraction liefert \<3 Felder           Manuellen Impressum-Link speichern für später. Generische Email verwenden              Wenn \>30% der Leads betroffen

  Branchenportal blockiert       Captcha / 403                              IP rotieren. Wenn weiterhin blockiert: Quelle überspringen, ohne Portal-Daten weiter   Portal dauerhaft nicht erreichbar

  Northdata/HR nicht verfügbar   API Error                                  Business Health ohne Firmendaten berechnen (nur Google-Signale)                        Nicht kritisch

  LinkedIn Rate Limit            429 Response                               Pause 24h. Fallback: Ohne Entscheider-Name weiter (generische Email)                   Wenn \>50% der Leads ohne Entscheider

  Daten-Widerspruch              Google sagt „Closed", Website ist online   Google-Status hat Priorität. Lead als „possibly_closed" flaggen                        Manuelles Review
  ------------------------------ ------------------------------------------ -------------------------------------------------------------------------------------- ---------------------------------------

3.8 KPIs & Monitoring

  ---------------------------------------- ------------------- --------------------------------------------
  **Metrik**                               **Zielwert**        **Alert wenn**

  Qualification Rate (Qualified / Total)   \>60%               \<45% oder \>85% (zu streng oder zu lasch)

  Entscheider-Identification-Rate          \>70% mit Name      \<50%

  Persönliche-Email-Rate                   \>40%               \<20%

  Avg. Enrichment-Kosten pro Lead          \<€0.15             \>€0.30

  Avg. Enrichment-Dauer pro Lead           \<30s               \>120s

  Business Health Avg. (qualified Leads)   \>55                \<40

  Pre-Qual Disqualification Rate           15-35%              \<10% oder \>50%
  ---------------------------------------- ------------------- --------------------------------------------

+-------+-------------------------------------------------------------------------------------------+
| **4** | **Website Crawl & Content Extraction**                                                    |
|       |                                                                                           |
|       | *Systematische Ernte aller Website-Inhalte + Brand Assets -- Rohmaterial für den Rebuild* |
+-------+-------------------------------------------------------------------------------------------+

+---------------------------------------------------------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **INPUT**                                                                 | **OUTPUT**                                                                                                                                                                                                                | **QUALITY GATE**                                                                                                                                               |
|                                                                           |                                                                                                                                                                                                                           |                                                                                                                                                                |
| Enriched Lead mit: • canonical_url • industry_tag • enrichment_confidence | Content Package: • pages\[\]{url, text, html, screenshot} • images\[\]{url, alt, dimensions, type} • brand_assets{logo, colors, fonts} • structured_data{} • firm_profile{} (AI-extrahiert) • extraction_confidence (0-1) | Mindestens 60% der Required Fields (lt. Industry Profile) extrahiert. Mindestens 1 Screenshot erfolgreich. firm_profile enthält Name + mind. 3 weitere Felder. |
+---------------------------------------------------------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------+

4.1 Crawling-Engine: Architektur

Das Crawling ist die ressourcenintensivste Operation in der Pipeline. Jede Website erfordert einen vollen Headless-Browser, JavaScript-Rendering und multiple Page-Loads. Effizienz und Fehlertoleranz sind hier kritisch.

Technologie-Stack

-   Browser: Playwright (Chromium) im Headless-Mode. Vorteile gegenüber Puppeteer: Multi-Browser-Support, bessere Auto-Wait-Logik, stabilere API

-   Parallelisierung: Max 5 Browser-Instanzen parallel (RAM-limitiert: \~500MB pro Instanz). Queue-basiert über Celery

-   Proxy: Residentieller Proxy-Pool für Bot-geschützte Websites. Standard: Direkt-Zugriff, Proxy nur bei 403/Captcha

-   Storage: S3-kompatibler Object Storage für Screenshots und heruntergeladene Assets. PostgreSQL für strukturierte Daten

Crawl-Ablauf pro Website

1.  Homepage laden (mit 15s Timeout, JS-Rendering abwarten). Cookie-Banner automatisch akzeptieren (bekannte Patterns: Borlabs, Cookiebot, OneTrust, Usercentrics). Screenshot (Desktop 1440px + Mobile 375px)

2.  Sitemap-Check: /sitemap.xml und /robots.txt laden. Wenn Sitemap vorhanden: Alle URLs extrahieren. Wenn nicht: Link-Discovery aus Navigation

3.  Navigation-Analyse: Alle internen Links aus Header-Nav + Footer-Nav extrahieren. Priorisieren: Leistungen, Über uns, Team, Kontakt, Referenzen, Blog, Preise, Impressum, Datenschutz

4.  Subseiten crawlen: Priorisierte Seiten zuerst, max 30 Seiten pro Website. Pro Seite: HTML speichern, Text extrahieren (Boilerplate-Removal via Readability-Algorithmus), Screenshot, interne Links sammeln

5.  Asset-Download: Alle Bilder \>50px herunterladen und katalogisieren. Logos erkennen (Header-Position, Dateiname-Heuristik). PDFs herunterladen (Speisekarten, Preislisten, Flyer)

6.  Impressum + Datenschutz: Immer crawlen (auch wenn bereits in Stufe 3 gescraped). Vollständiger Text für Legal-Check in Stufe 11

4.2 Content Extraction (AI-Schicht)

Nach dem technischen Crawl wird der extrahierte Content durch Claude API analysiert und in ein strukturiertes Firmenprofil überführt.

AI-Extraction Prompt-Strategie

-   Ein zentraler Extraction-Prompt pro Industry-Profile. Der Prompt enthält: Alle erwarteten Felder mit Beschreibung, Branchenspezifische Hinweise (z.B. „Bei Anwälten: Rechtsgebiete sind oft in der Navigation als Untermenüpunkte"), Strenge Anweisung: Nur extrahieren was explizit vorhanden ist. Nichts erfinden. Bei Unsicherheit: Feld leer lassen + confidence_note

-   Input an Claude: Extrahierter Text aller gecrawlten Seiten (max 50k Tokens, priorisiert nach Seiten-Relevanz). Google Business Daten als Cross-Reference. Enrichment-Daten als Kontext

-   Output: Strukturiertes JSON mit allen Feldern + Confidence pro Feld (0-1) + source_page pro Feld (welche URL hat die Info geliefert)

Firmenprofil-Schema

  ---------------------------------- ----------------- --------------- --------------------------------------------------------------------
  **Feld**                           **Typ**           **Required?**   **Confidence-Logik**

  firm_name                          String            Ja              1.0 wenn aus Impressum, 0.8 wenn aus Title-Tag

  slogan                             String            Nein            0.9 wenn aus H1 auf Homepage

  description                        String (lang)     Ja              Proportional zur Länge des gefundenen Über-uns-Texts

  services\[\]                       Array\<String\>   Ja              1.0 wenn dedizierte Leistungsseite, 0.6 wenn nur aus Navigation

  usps\[\]                           Array\<String\>   Nein            0.8 wenn explizit formuliert, 0.4 wenn aus Kontext abgeleitet

  team\[\]{name, role, bio, photo}   Array\<Object\>   Nein            1.0 wenn Team-Seite, 0.5 wenn nur aus Impressum

  founding_year                      Number            Nein            1.0 wenn explizit erwähnt, 0.6 wenn aus „seit 20 Jahren" berechnet

  certifications\[\]                 Array\<String\>   Nein            1.0 wenn mit Badge/Logo, 0.7 wenn nur Text-Erwähnung

  contact{phone, email, address}     Object            Ja              1.0 wenn aus Impressum (rechtlich verbindlich)

  opening_hours{}                    Object            Nein            1.0 wenn strukturiert auf Website, 0.9 wenn aus Google Business

  testimonials\[\]{text, author}     Array\<Object\>   Nein            1.0 wenn auf Website, 0.7 wenn aus Google Reviews

  prices{}                           Object            Nein            1.0 wenn Preisseite, 0.0 wenn „auf Anfrage"

  social_links{}                     Object            Nein            1.0 wenn im Footer/Header verlinkt

  blog_posts\[\]{title, date, url}   Array\<Object\>   Nein            1.0 (entweder vorhanden oder nicht)

  industry_specific{}                Object            Teilweise       Abhängig vom Industry Profile
  ---------------------------------- ----------------- --------------- --------------------------------------------------------------------

4.3 Brand Asset Extraction

Brand Assets bestimmen ob wir das bestehende Branding übernehmen oder upgraden. Schlechtes Branding = kompletter Neu-Aufbau. Gutes Branding mit schlechter Website = nur Website-Upgrade mit bestehendem Branding.

Logo-Extraction

1.  Primär: \<img\> oder \<svg\> im \<header\>-Bereich. Dateinamen-Heuristik: logo, brand, header-logo. Alt-Text-Heuristik: Firmenname im Alt-Text

2.  Sekundär: Favicon (oft komprimierte Logo-Version). OG:Image (manchmal Logo enthalten)

3.  Qualitätsbewertung: Dateiformat (SVG = perfekt, PNG \>500px = gut, JPEG = akzeptabel, GIF = schlecht, \<200px = zu klein). Transparenter Hintergrund vorhanden?

4.  Speicherung: Alle gefundenen Logo-Varianten herunterladen. Beste Qualität als primary_logo markieren

Farbpalette-Extraktion

-   CSS-Analyse: Alle definierten Farben aus Stylesheets extrahieren. Häufigkeitsanalyse: Welche Farben werden am meisten verwendet?

-   Kategorisierung: Primary Color (Buttons, Links, Akzente), Secondary Color (Hintergründe, Sections), Text Color, Background Color

-   Qualitätsbewertung: Kontrast-Ratio (WCAG AA = \>4.5:1), Konsistenz (über alle Seiten gleich?), Branchenfit (blau/dunkel für Anwälte, bunt für Friseure)

Typografie-Extraktion

-   CSS Font-Family-Analyse: Welche Schriftarten werden geladen? Web-Font (Google Fonts, Adobe Fonts) vs. System-Font (Arial, Times)?

-   Heading-Font vs. Body-Font unterscheiden

-   Qualitätsbewertung: Web-Font = professioneller, System-Font = Budget-Indikator. Comic Sans / Papyrus / Impact = sofortiger Design-Red-Flag

Bildsprache-Analyse

-   Alle heruntergeladenen Bilder kategorisieren (AI-basiert): Eigene Fotos (Team, Geschäft, Projekte) vs. Stock-Fotos (Watermark-Check, Reverse Image Search) vs. Clipart/Icons vs. Keine relevanten Bilder

-   Qualitätsbewertung: Auflösung, Belichtung, Komposition, Relevanz zum Business

-   Quantität: Wie viele verwendbare Bilder gibt es? \>10 gute eigene Fotos = Goldmine für Rebuild. 0 eigene Fotos = NanoBanana-Heavy

4.4 Edge Cases & schwierige Websites

Viele KMU-Websites sind technisch herausfordernd. Die Crawling-Engine muss mit ALLEN folgenden Szenarien umgehen können:

  ---------------------------------------------- ---------------- ------------------------------------------------------------------------------------------------------ -------------------------------------------------------------------------
  **Szenario**                                   **Häufigkeit**   **Lösung**                                                                                             **Degradation**

  JavaScript-Heavy (React/Vue SPA)               5-10%            Playwright wartet auf Network-Idle + DOM-Stable. Scroll-Simulation für Lazy-Loaded Content             Wenn nach 30s immer noch leer: Screenshot-Only-Analyse via Vision

  WordPress mit Page Builder (Elementor, Divi)   30-40%           Standard-Crawling funktioniert. Boilerplate-Removal muss Builder-Markup ignorieren                     Selten problematisch

  Baukasten (Jimdo, Wix, Squarespace)            15-20%           JavaScript-Rendering essentiell (Wix = React-basiert). Spezielle Wait-Conditions für Wix/Squarespace   Wenn Rendering fehlschlägt: Wappalyzer-Detection + vereinfachte Analyse

  Content in Bildern statt Text                  10-15%           OCR (Tesseract) auf Screenshots. AI-Vision-Analyse für Hero-Bilder mit Text-Overlay                    OCR-Confidence \<0.5: Feld leer lassen

  PDF-only Content (Preislisten, Flyer)          5-10%            PDFs herunterladen + Text-Extraction (PyMuPDF). AI-Analyse des PDF-Contents                            PDF passwortgeschützt: Skip

  Sehr alte Websites (Frames, Flash)             \<5%             Frames: Alle Frame-URLs einzeln crawlen. Flash: Nur Screenshot + Meta-Tags                             Minimale Extraction, aber hochster Opportunity Score

  Multi-Language                                 5-10%            Deutsche Version priorisieren (hreflang-Tag oder URL-Pattern /de/). Nur DE-Content extrahieren         Wenn keine DE-Version: Lead disqualifizieren (DACH-Markt)

  Under Construction / Coming Soon               2-5%             Erkennung via häufige Patterns. Lead in Warteschlange: Re-Crawl in 4 Wochen                            Nach 3 Re-Crawls immer noch UC: verwerfen

  Bot-Protection (Cloudflare, etc.)              10-15%           Playwright-Stealth + Proxy. Cookie-Solving. Human-Behavior-Simulation                                  Wenn nach 3 Versuchen blockiert: Manuelle Queue
  ---------------------------------------------- ---------------- ------------------------------------------------------------------------------------------------------ -------------------------------------------------------------------------

4.5 Extraction Confidence & Content Completeness

Der Extraction Confidence Score bestimmt ob genug Content für einen überzeugenden Rebuild vorhanden ist:

Content Completeness Score

  -------------------------------------------------- ------------- ---------------------------------------------------------
  **Feld-Kategorie**                                 **Gewicht**   **Score-Logik**

  Core Identity (Name, Description, Services)        0.30          Alle 3 vorhanden = 1.0, 2/3 = 0.7, 1/3 = 0.3, 0/3 = 0.0

  Contact Info (Phone, Email, Address, Hours)        0.20          Alle vorhanden = 1.0, proportional weniger

  Visual Assets (Logo, \>3 verwendbare Bilder)       0.15          Logo + Bilder = 1.0, nur Logo = 0.6, nichts = 0.0

  Social Proof (Testimonials, Reviews, Referenzen)   0.15          Mind. 3 = 1.0, 1-2 = 0.6, 0 = 0.0

  Industry-Specific Required Fields                  0.10          Aus Industry Profile: % der required fields extrahiert

  Brand Assets (Logo-Qualität, Farben, Fonts)        0.10          Vollständig = 1.0, teilweise = 0.5, nichts = 0.0
  -------------------------------------------------- ------------- ---------------------------------------------------------

  --------------- ----------------------- -------------------------------------------------------------------------------------------------------------------------
  **Score**       **Bedeutung**           **Routing**

  **\> 0.7**      Reichhaltiger Content   Full Rebuild möglich. Premium-Template. Alle Sections befüllbar

  **0.5 - 0.7**   Ausreichend Content     Rebuild möglich, aber dünne Sections. NanoBanana für fehlende Bilder. Manche Optional-Sections weglassen

  **0.3 - 0.5**   Minimaler Content       Nur Basis-Template möglich. Viele Platzhalter. Lead bekommt niedrigeren Score in Stufe 7 (weniger überzeugende Preview)

  **\< 0.3**      Unzureichend            Kein Rebuild sinnvoll. Lead in „Nurture Only"-Pool: Bekommt nur Analyse-Report, keine Preview
  --------------- ----------------------- -------------------------------------------------------------------------------------------------------------------------

4.6 Error Handling & Performance

  ----------------------------------- ---------------------------------------- ------------------------------------------------------------------------------------------------------- ------------------
  **Failure Mode**                    **Erkennung**                            **Fallback**                                                                                            **Max Retries**

  Website komplett down               Connection Error / DNS Failure           In Retry-Queue: Neuer Versuch in 24h, 48h, 7d                                                           3

  Crawl dauert \>5 Min                Timeout-Monitor                          Crawl abbrechen, nur Homepage-Daten verwenden                                                           1 (dann Partial)

  Claude API Error                    HTTP 5xx / Timeout                       Retry mit Exponential Backoff. Nach 3 Fails: Queue für später                                           3

  Claude Extraction halluziniert      Confidence \<0.3 auf extrahiertem Feld   Feld verwerfen. Wenn \>50% der Felder low-confidence: komplett neu extrahieren mit angepasstem Prompt   1

  Zu viele Seiten (\>100 Subseiten)   Seitencount-Monitor                      Nach 30 Seiten stoppen. Priorisierung nach Nav-Position                                                 N/A

  Assets zu groß (\>500MB Bilder)     Download-Size-Monitor                    Nur Bilder \>100px und \<5MB herunterladen. Rest ignorieren                                             N/A

  Anti-Bot-Schutz                     Captcha-Detection / 403                  Proxy + Stealth + Delay. Wenn weiterhin blockiert: Screenshot-Only + Google-Cache                       3
  ----------------------------------- ---------------------------------------- ------------------------------------------------------------------------------------------------------- ------------------

4.7 KPIs & Monitoring

  ----------------------------------- ------------------- ------------------------------------------------
  **Metrik**                          **Zielwert**        **Alert wenn**

  Avg. Crawl-Dauer pro Website        \<90s               \>180s

  Avg. Pages Crawled pro Website      8-15                \<3 (zu wenig Content) oder \>30 (ineffizient)

  Content Completeness Score (Avg)    \>0.6               \<0.4

  Extraction Confidence (Avg)         \>0.7               \<0.5

  Screenshot Success Rate             \>98%               \<90%

  AI-Extraction Halluzination Rate    \<5%                \>15%

  Avg. Cost pro Crawl + Extraction    \<€0.10             \>€0.25

  Nurture-Only-Rate (Content \<0.3)   \<15%               \>30% (Zielgruppe hat zu wenig Content)
  ----------------------------------- ------------------- ------------------------------------------------

+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Audit Trail Reminder**                                                                                                                                                                                                                                                                                                                                         |
|                                                                                                                                                                                                                                                                                                                                                                  |
| Jede Crawling-Session wird vollständig geloggt: Welche URLs wurden besucht, welche Daten extrahiert, welche Bilder heruntergeladen, welche Fehler aufgetreten, welcher Claude-Prompt verwendet, welcher Output generiert. Dies ist essenziell für Debugging, Qualitätsverbesserung und um bei Kundenfragen nachvollziehen zu können welche Daten von wo stammen. |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

Nächste Iteration: Steps 5-8

Dieses Dokument deckt Steps 1-4 ab. Die nächste Iteration spezifiziert:

-   Step 5: Website Quality Review -- AI-basierte Bewertung mit Calibration Sets, Schwachstellen-Priorisierung, visuelle Analyse

-   Step 6: Competitor Benchmarking -- Feature-Level-Vergleich, relative Positionierung, Sales-Argument-Generierung

-   Step 7: Lead Scoring -- Multi-dimensionales Scoring mit Feedback Loop, Score Decay, Industry-Kalibrierung

-   Step 8: Rebuild Planning -- Decision Trees für Content-Übernahme, Template-Wahl, Brand-Entscheidungen, Legal Checklist

Jeder Step wird in der gleichen Tiefe spezifiziert: Sub-Operations, Error Handling, Confidence Scoring, Degradation Paths, Industry-spezifische Logik, KPIs.

*Webolution Deep Spec -- Steps 1-4 -- SocialCooks*
