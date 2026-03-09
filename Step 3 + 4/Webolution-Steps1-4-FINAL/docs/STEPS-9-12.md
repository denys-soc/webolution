**WEBOLUTION**

Deep Spec: Steps 9--12

*Content Polish → Website Build → QA & Testing → Preview Deployment*

Vom Plan zur live Preview -- Execution in epischer Qualität

**SocialCooks × Webolution**

März 2026

+-------+-------------------------------------------------------------------------------------------------+
| **9** | **Content Polish & Fact-Verification**                                                          |
|       |                                                                                                 |
|       | *Texte optimieren, Fakten verifizieren, Bilder aufbereiten -- das Uncanny-Valley-Problem lösen* |
+-------+-------------------------------------------------------------------------------------------------+

+-----------------------------------------------------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------------------------------------------+
| **INPUT**                                                             | **OUTPUT**                                                                                                                                                                                                          | **QUALITY GATE**                                                                                                              |
|                                                                       |                                                                                                                                                                                                                     |                                                                                                                               |
| Rebuild Plan (Stufe 8) + Content Package (Stufe 4) + Industry Profile | Polished Content Set: • texts{} pro Section (final, publish-ready) • images{} pro Section (optimiert/ersetzt) • meta_tags{title, description, og} • schema_markup{} • fact_check_report{} • polish_confidence (0-1) | Fact-Check bestanden (0 Halluzinationen). Banned-Phrase-Check bestanden. Tone-of-Voice konsistent. Alle Bilder publish-ready. |
+-----------------------------------------------------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------------------------------------------+

9.1 Das Uncanny-Valley-Problem

+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Das größte Risiko bei AI-generiertem Content**                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
|                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| Eine Website die zu 90% gut ist aber subtile AI-Fingerprints hat, wirkt SCHLECHTER als eine komplett schlechte Website. Der Lead denkt: „Die haben mein Business nicht verstanden, das ist ein Template." Typische AI-Fingerprints: Generische Phrasen („Wir legen Wert auf Qualität"), zu perfekte Struktur ohne menschliche Ecken, Fakten die fast stimmen aber nicht ganz, übertrieben positive Sprache ohne Substanz. Das Ziel ist nicht „AI-optimiert" sondern „von einem guten Texter geschrieben". |
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

9.2 Banned Phrases & AI-Fingerprint-Erkennung

Vor der Text-Generierung wird ein striktes Regelwerk geladen. Jeder generierte Text wird gegen diese Liste geprüft:

Universelle Banned Phrases (alle Industrien)

  ------------------------------------------------------- ------------------------------------------------------------ ----------------------------------------------------------------------------------
  **Banned Phrase**                                       **Warum verboten**                                           **Bessere Alternative**

  „Wir legen Wert auf Qualität und Kundenzufriedenheit"   Generischste Phrase im Internet. Sagt null aus               Konkretes Beispiel: „97% unserer Mandanten empfehlen uns weiter" (nur wenn Fakt)

  „Ihr kompetenter Partner für..."                        AI-Standard-Einstieg. Sofort erkennbar                       Direkt zur Sache: „Spezialisiert auf Familienrecht seit 2008"

  „Mit langjähriger Erfahrung"                            Vage. Wie lange genau? Wenn wir es nicht wissen: weglassen   „Seit \[Jahr\] in \[Stadt\]" (nur wenn extrahiert)

  „Wir bieten maßgeschneiderte Lösungen"                  Bedeutungslose Floskel                                       Konkret: „Jeder Fall wird individuell geprüft"

  „Ganzheitlicher Ansatz"                                 Überstrapaziert, besonders bei Coaches                       Methode konkret benennen: „Kombination aus \[Methode A\] und \[Methode B\]"

  „Profitieren Sie von..."                                Werbe-Sprech aus den 2000ern                                 Nutzen direkt formulieren: „\[Konkreter Vorteil\]"

  „Wir stehen für..."                                     Abstract, nicht beweisbar                                    Stattdessen: Fakten zeigen die es beweisen

  „Herzlich willkommen auf unserer Website"               Verschwendeter Hero-Platz                                    Hero-Text: Was macht die Firma + für wen + CTA

  „Schauen Sie sich um"                                   Passiv, kein klarer CTA                                      „Jetzt Termin vereinbaren" oder „Unsere Leistungen ansehen"

  „Kontaktieren Sie uns gerne"                            Akzeptabel am Ende, aber nie als CTA                         „Jetzt anrufen: \[Nummer\]" oder „Termin buchen"
  ------------------------------------------------------- ------------------------------------------------------------ ----------------------------------------------------------------------------------

Industrie-spezifische Banned Phrases

+---------------+----------------------------------------+---------------------------------------------------------------------------------------------+
| **Industrie** | **Zusätzlich verboten**                | **Warum**                                                                                   |
+---------------+----------------------------------------+---------------------------------------------------------------------------------------------+
| Anwälte       | „Wir kämpfen für Ihr Recht"            | Klischee-Anwalts-Sprech. Jeder zweite Anwalt schreibt das. Kein Differenzierungsmerkmal     |
|               |                                        |                                                                                             |
|               | „Rechtliche Rundumbetreuung"           |                                                                                             |
|               |                                        |                                                                                             |
|               | „Stets an Ihrer Seite"                 |                                                                                             |
+---------------+----------------------------------------+---------------------------------------------------------------------------------------------+
| Ärzte         | „Ihr Wohlbefinden liegt uns am Herzen" | Standard-Praxis-Text. Plus HWG-Risiko bei „modernste Methoden"                              |
|               |                                        |                                                                                             |
|               | „Modernste Behandlungsmethoden"        |                                                                                             |
|               |                                        |                                                                                             |
|               | „In angenehmer Atmosphäre"             |                                                                                             |
+---------------+----------------------------------------+---------------------------------------------------------------------------------------------+
| Coaches       | „Entfalten Sie Ihr volles Potenzial"   | Coaching-Buzzwords. Jeder Coach nutzt diese. Wirkt austauschbar                             |
|               |                                        |                                                                                             |
|               | „Auf Augenhöhe"                        |                                                                                             |
|               |                                        |                                                                                             |
|               | „Transformation erleben"               |                                                                                             |
+---------------+----------------------------------------+---------------------------------------------------------------------------------------------+
| Friseure      | „Weil Sie es sich wert sind"           | Generischer Salon-Speak. Kein USP erkennbar                                                 |
|               |                                        |                                                                                             |
|               | „Für ein perfektes Styling"            |                                                                                             |
|               |                                        |                                                                                             |
|               | „Ihr neues Ich"                        |                                                                                             |
+---------------+----------------------------------------+---------------------------------------------------------------------------------------------+
| Handwerker    | „Sauber und zuverlässig"               | Standard. Jeder Handwerker behauptet das. Stattdessen: konkrete Zahlen, Projekte, Garantien |
|               |                                        |                                                                                             |
|               | „Qualität vom Fachmann"                |                                                                                             |
|               |                                        |                                                                                             |
|               | „Alles aus einer Hand"                 |                                                                                             |
+---------------+----------------------------------------+---------------------------------------------------------------------------------------------+

Automatische Banned-Phrase-Detection

-   Post-Generation Scan: Jeder generierte Text wird gegen die Banned-Liste geprüft (Exact Match + Fuzzy Match mit 0.85 Threshold)

-   Bei Treffer: Automatische Neuformulierung des betroffenen Absatzes mit explizitem Prompt „Ersetze die Phrase \[X\] durch eine konkrete, faktenbasierte Aussage"

-   Second-Pass-Check: Nach Neuformulierung nochmal scannen. Wenn immer noch Banned Phrases: Absatz auf das Minimum reduzieren (nur Fakten, kein Fluff)

-   Monitoring: Track welche Banned Phrases am häufigsten generiert werden → Prompt-Tuning

9.3 Tone-of-Voice-Engine

Jede Industrie hat ein detailliertes Tone-of-Voice-Profil das in den Generierungs-Prompt geladen wird:

+-------------------+---------------------------------------------+--------------------------------------------+--------------------------------------+-----------------------------------------+------------------------------------------+
| **Dimension**     | **Anwälte**                                 | **Ärzte**                                  | **Coaches**                          | **Friseure**                            | **Handwerker**                           |
+-------------------+---------------------------------------------+--------------------------------------------+--------------------------------------+-----------------------------------------+------------------------------------------+
| **Formalität**    | Sehr formell. Sie-Form. Kein Slang          | Formell aber warm. Sie-Form. Empathisch    | Locker. Du-Form erlaubt. Motivierend | Locker-freundlich. Ihr/Du je nach Brand | Direkt. Sie-Form. Pragmatisch            |
+-------------------+---------------------------------------------+--------------------------------------------+--------------------------------------+-----------------------------------------+------------------------------------------+
| **Satzlänge**     | Mittel-lang. Präzise. Juristisch klar       | Kurz-mittel. Verständlich. Kein Fachjargon | Kurz. Punchy. Aktivierend            | Kurz. Einladend. Visuell beschreibend   | Kurz. Auf den Punkt. Kein Schnickschnack |
+-------------------+---------------------------------------------+--------------------------------------------+--------------------------------------+-----------------------------------------+------------------------------------------+
| **Emotionalität** | Sachlich-vertrauensvoll. Keine Übertreibung | Empathisch-beruhigend. Patientenorientiert | Inspirierend-motivierend. Persönlich | Enthusiastisch-einladend. Lifestyle     | Nüchtern-kompetent. Zahlen \> Worte      |
+-------------------+---------------------------------------------+--------------------------------------------+--------------------------------------+-----------------------------------------+------------------------------------------+
| **Power-Wörter**  | Kompetenz, Expertise                        | Gesundheit, Wohlbefinden                   | Klarheit, Wachstum                   | Stil, Verwöhnen                         | Zuverlässig, Meisterarbeit               |
|                   |                                             |                                            |                                      |                                         |                                          |
|                   | Vertrauen, Diskretion                       | Betreuung, Vorsorge                        | Authentizität, Fokus                 | Kreativität, Trend                      | Termin, Festpreis                        |
|                   |                                             |                                            |                                      |                                         |                                          |
|                   | Durchsetzung, Recht                         | Fachwissen, Empathie                       | Ergebnis, Wandel                     | Wohlfühlen, Look                        | Erfahrung, Handwerk                      |
+-------------------+---------------------------------------------+--------------------------------------------+--------------------------------------+-----------------------------------------+------------------------------------------+
| **Tabu-Wörter**   | Billig, günstig                             | Heilung (HWG!)                             | Muss, Pflicht                        | Alt, veraltet                           | Billig                                   |
|                   |                                             |                                            |                                      |                                         |                                          |
|                   | Garantie (rechtlich heikel)                 | Garantie                                   | Problem (statt: Herausforderung)     | Billig                                  | Pfusch, Murks                            |
|                   |                                             |                                            |                                      |                                         |                                          |
|                   | Kampf, Krieg                                | Schmerz (zu negativ)                       | Angst                                | Durchschnittlich                        | Provisorisch                             |
+-------------------+---------------------------------------------+--------------------------------------------+--------------------------------------+-----------------------------------------+------------------------------------------+

9.4 Text-Optimierungs-Workflow

Für jede Section im Rebuild-Plan wird folgender Workflow durchlaufen:

1.  Content-Decision laden: ÜBERNEHMEN / OPTIMIEREN / NEU AUS DATEN / WEGLASSEN (aus Stufe 8)

2.  Tone-of-Voice-Profil laden: Industry Profile + ggf. Brand-spezifische Anpassung

3.  SEO-Keywords laden: Primary + Secondary Keywords aus SEO-Plan (Stufe 8). Natural Integration, kein Stuffing

4.  Generierung/Optimierung: Claude API Call mit: Originaltext (wenn vorhanden), extrahierte Fakten, Tone-Profile, SEO-Keywords, Banned-Phrases-Liste, Section-spezifische Anweisungen (Hero braucht CTA, About braucht Persönlichkeit, etc.)

5.  Banned-Phrase-Check: Automatischer Scan. Bei Treffer: Re-Generate der betroffenen Passage

6.  Fact-Verification: Jede Behauptung im generierten Text gegen extrahierte Daten cross-checken (Stufe 9.5)

7.  Längen-Check: Mindest- und Maximallänge pro Section. Hero: 20-50 Wörter. Service-Beschreibung: 80-200 Wörter. About: 100-300 Wörter

8.  Finaler Output: Text mit Quellen-Annotation (jeder Satz hat eine source_reference auf den extrahierten Datenpunkt)

9.5 Fact-Verification-Engine

+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Zero-Tolerance für falsche Fakten**                                                                                                                                                                                                                                                                                                        |
|                                                                                                                                                                                                                                                                                                                                              |
| Ein einziger falscher Fakt auf der Preview zerstört die gesamte Glaubwürdigkeit. Wenn der Lead sieht dass seine Telefonnummer falsch ist, sein Gründungsjahr nicht stimmt oder eine Leistung erwähnt wird die er nicht anbietet -- vertraut er nichts mehr auf der Seite. Und kauft nicht. Fact-Verification ist deshalb ein harter Blocker. |
+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

Cross-Check-Matrix

  ------------------------ ------------------------------ ---------------------------- -------------------------------------------------------------------------
  **Datenpunkt im Text**   **Cross-Check-Quelle 1**       **Cross-Check-Quelle 2**     **Bei Widerspruch**

  Firmenname               Impressum (aus Extraction)     Google Business              Impressum gewinnt (rechtlich verbindlich)

  Adresse                  Impressum                      Google Business              Bei Abweichung: Flaggen. Impressum verwenden

  Telefonnummer            Impressum                      Google Business              Impressum verwenden. Google als Fallback

  Öffnungszeiten           Website (Extraction)           Google Business              Google Business verwenden (aktueller). Flag wenn Abweichung

  Gründungsjahr            Über-uns-Text                  Handelsregister / WHOIS      HR-Daten haben Priorität. Wenn nicht verfügbar: Website-Text

  Leistungen/Services      Leistungsseiten (Extraction)   Google Business Kategorie    Nur Services verwenden die auf mind. 1 Quelle vorkommen

  Team-Mitglieder          Team-Seite (Extraction)        Impressum / Branchenportal   Nur Personen die auf Website erwähnt sind

  Preise                   Preisseite (Extraction)        Keine Cross-Check-Quelle     Nur verwenden wenn EXAKT auf Website publiziert. Nie runden oder ändern

  Zertifizierungen         Website (Extraction)           Kammer-Verzeichnis           Nur verwenden wenn verifizierbar

  Testimonials             Website (Extraction)           Google Reviews               Website-Testimonials 1:1 übernehmen. Google Reviews separat einbinden
  ------------------------ ------------------------------ ---------------------------- -------------------------------------------------------------------------

Automatischer Fact-Check-Ablauf

1.  Source-Annotation: Jeder Satz im generierten Text hat eine source_reference. Satz ohne Quelle = verdächtig

2.  Fakten-Extraktion: AI extrahiert alle konkreten Fakten aus dem generierten Text (Zahlen, Namen, Daten, Orte, Leistungen)

3.  Cross-Check: Jeder extrahierte Fakt wird gegen die Cross-Check-Matrix geprüft. Match = OK. Mismatch = Flag. Kein Match (Fakt existiert in keiner Quelle) = HALLUZINATION

4.  Halluzinations-Handling: Bei erkannter Halluzination: Satz entfernen oder durch Fakt aus Quelle ersetzen. Gesamten Absatz re-generieren. Nach 2. Halluzination im selben Text: Gesamte Section auf Minimum reduzieren (nur verifizierte Fakten)

5.  Report: Fact-Check-Report pro Lead: Wie viele Fakten geprüft, wie viele bestätigt, wie viele korrigiert, wie viele Halluzinationen entfernt

9.6 Bild-Optimierung & NanoBanana-Integration

  --------------------------------------------------------- ---------------------------------------------------------------------------------------- ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ -----------------------------------------------------------------
  **Bild-Situation**                                        **Aktion**                                                                               **NanoBanana-Prompt**                                                                                                                                                                **Kennzeichnung auf Preview**

  Gutes eigenes Foto (\>800px, gute Belichtung, relevant)   Komprimieren (WebP, 80% Quality). Resize auf Template-Dimensionen. Alt-Text generieren   Nicht nötig                                                                                                                                                                          Keine (eigenes Foto)

  Mittleres Foto (ok Auflösung, leicht dunkel/unscharf)     AI-Enhancement: Aufhellen, Schärfen, Crop optimieren. Dann komprimieren                  Nicht nötig                                                                                                                                                                          Keine (verbessertes Originalfoto)

  Schlechtes Foto (pixelig, \<400px, irrelevant)            Ersetzen durch NanoBanana-Bild                                                           „\[Industrie\]-\[Typ\]: \[Beschreibung des benötigten Bildes basierend auf Section\]" z.B. „Anwaltskanzlei-Hero: Professionelles Büro-Ambiente, warm beleuchtet, Akten und Bücher"   Dezentes Badge: „Beispielbild -- wird durch Ihre Fotos ersetzt"

  Kein Foto vorhanden für Section                           NanoBanana generiert branchenpassendes Bild                                              Prompt aus Industry Profile + Section-Kontext                                                                                                                                        Badge: „Platzhalter -- hier können Ihre Fotos stehen"

  Team-Foto fehlt                                           Initialen-Avatar generieren (Vorname + Nachname → stilvoller Kreis mit Initialen)        Nicht nötig (kein AI-Portrait!)                                                                                                                                                      „Foto folgt" oder nur Initialen

  Logo niedrige Auflösung                                   SVG-Vektorisierung versuchen (Potrace). Wenn nicht möglich: best available verwenden     Nicht für Logos (zu riskant)                                                                                                                                                         Keine
  --------------------------------------------------------- ---------------------------------------------------------------------------------------- ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ -----------------------------------------------------------------

+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **KRITISCH: Keine AI-generierten Personen-Portraits**                                                                                                                                                                                                                                                                                                     |
|                                                                                                                                                                                                                                                                                                                                                           |
| NanoBanana darf NIEMALS für Team-Fotos / Personen-Portraits verwendet werden. Ein AI-generiertes Gesicht als „Anwalt Müller" ausgeben ist: (a) rechtlich problematisch, (b) sofort erkennbar bei genauem Hinsehen, (c) zerstört Vertrauen wenn der Lead es merkt. Stattdessen: Initialen-Avatare, abstrakte Silhouetten, oder Section komplett weglassen. |
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

9.7 Meta-Tags & Schema Markup Generierung

Title-Tags (pro Seite)

-   Homepage: \[Primary Keyword\] \| \[Firmenname\] -- Max 60 Zeichen. Beispiel: „Anwalt Familienrecht München \| Kanzlei Müller"

-   Leistungsseite: \[Leistung\] in \[Stadt\] \| \[Firmenname\] -- Beispiel: „Scheidungsrecht in München \| Kanzlei Müller"

-   About: Über \[Firmenname\] \| \[Branche\] in \[Stadt\]

-   Kontakt: Kontakt \| \[Firmenname\] \[Stadt\]

Schema Markup

  --------------- ----------------------------- ---------------------------------------------------------- --------------------------------------------------
  **Industrie**   **Schema-Typ**                **Pflicht-Felder**                                         **Optional**

  Anwälte         Attorney + LegalService       name, address, telephone, priceRange, areaServed           knowsAbout (Rechtsgebiete), review

  Ärzte           Physician + MedicalBusiness   name, address, telephone, medicalSpecialty, openingHours   availableService, review, isAcceptingNewPatients

  Coaches         ProfessionalService           name, address, telephone, description                      review, hasOfferCatalog, knowsAbout

  Friseure        HairSalon + LocalBusiness     name, address, telephone, openingHours, priceRange         review, image, makesOffer

  Handwerker      HomeAndConstructionBusiness   name, address, telephone, areaServed, serviceType          review, priceRange, knowsAbout
  --------------- ----------------------------- ---------------------------------------------------------- --------------------------------------------------

9.8 Error Handling

  ------------------------------------------ ----------------------------------------------------- --------------------------------------------------------------------------------------------------------------
  **Failure Mode**                           **Erkennung**                                         **Fallback**

  AI generiert Banned Phrases trotz Prompt   Post-Generation Scan                                  Re-Generate mit explizitem „Nutze NICHT:" Prompt. Max 3 Retries, dann: Section auf Fakten-Minimum reduzieren

  Halluzination erkannt                      Fact-Check findet Behauptung ohne Quelle              Satz entfernen. Absatz re-generieren. Nach 2x: nur verifizierte Fakten, kein Fließtext

  NanoBanana API nicht verfügbar             HTTP Error / Timeout                                  Branchenstock-Bilder als Fallback (lizenzfreie Bibliothek). Oder: Section ohne Bild bauen

  NanoBanana generiert schlechtes Bild       AI-Quality-Check auf generiertes Bild (Score \<0.5)   Neuer Prompt mit mehr Kontext. Max 3 Retries. Dann: Stock-Fallback

  Text zu lang/kurz für Section              Wortcount-Check                                       Re-Generate mit expliziter Längen-Anweisung. Wenn weiterhin falsch: manuelles Trimming

  Tone-of-Voice inkonsistent                 AI-Tone-Check (zweiter Claude Pass)                   Re-Generate mit stärkerem Tone-Profil im Prompt
  ------------------------------------------ ----------------------------------------------------- --------------------------------------------------------------------------------------------------------------

9.9 KPIs & Monitoring

  ----------------------------------------------------- ------------------------- ---------------------------------------------
  **Metrik**                                            **Zielwert**              **Alert wenn**

  Halluzinations-Rate (Fakten ohne Quelle)              \< 1% aller Sätze         \> 3%

  Banned-Phrase-Hits pro Text                           \< 0.5 nach Generierung   \> 2 (Prompt zu schwach)

  Fact-Check-Pass-Rate (erster Durchlauf)               \> 90%                    \< 75%

  Avg. Generierungs-Dauer (alle Sections eines Leads)   \< 120s                   \> 300s

  NanoBanana-Bilder pro Lead (Avg)                      2-5                       \> 8 (zu wenig eigene Bilder in Zielgruppe)

  Content Polish Confidence (Avg)                       \> 0.8                    \< 0.6

  Cost pro Content Polish                               \< €0.15                  \> €0.30
  ----------------------------------------------------- ------------------------- ---------------------------------------------

+--------+--------------------------------------------------------------------------------------------------+
| **10** | **Website Assembly & Build**                                                                     |
|        |                                                                                                  |
|        | *Template + polished Content + Code zusammenführen -- mit Performance- und Accessibility-Budget* |
+--------+--------------------------------------------------------------------------------------------------+

+-----------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------+
| **INPUT**                                                                                     | **OUTPUT**                                                                                                                                                       | **QUALITY GATE**                                                                                                            |
|                                                                                               |                                                                                                                                                                  |                                                                                                                             |
| Rebuild Plan (Stufe 8) + Polished Content Set (Stufe 9) + Brand Decision + Template Selection | Deploybare Next.js App: • Alle Seiten fertig gerendert • Bilder optimiert + eingebettet • Formulare funktionsfähig • Schema Markup integriert • build_metadata{} | Build erfolgreich (kein Error). Lighthouse Performance \> 90. Alle Links intern auflösbar. Formular sendet Test-Submission. |
+-----------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------+

10.1 Template-Engine-Architektur

Die Website wird nicht von der AI „von Grund auf" geschrieben. Stattdessen: Ein strukturiertes Template-System das mit Content befüllt wird. Das ist schneller, konsistenter und weniger fehleranfällig.

Template-Struktur

-   Base-Template: Next.js + Tailwind. Enthalt: Layout-Grundstruktur, Responsive Breakpoints, Header/Footer-Skeleton, Section-Slots, Formular-Logik, Analytics-Integration

-   Industry-Template: Erweitert Base. Enthält: Branchenspezifische Section-Reihenfolge, Standard-Farbschema als Fallback, Schema-Markup-Templates, CTA-Wording-Defaults

-   Variant-Layer: Modifiziert Industry-Template. Enthält: Hero-Variante (1 von 4), Font-Pairing (1 von 5), Bild-Layout-Stil (rund/eckig/overlay), Section-Reihenfolgen-Variation

-   Content-Layer: Füllt alles mit dem Polished Content Set. Texte in Sections, Bilder an definierten Positionen, Farben aus Brand-Decision, Logo eingesetzt

Build-Prozess

1.  Template-Resolution: Industry-Template + Variant (aus Rebuild-Plan Stufe 8) laden

2.  Brand-Application: Farbschema anwenden (CSS Custom Properties). Font-Pairing laden. Logo einsetzen

3.  Content-Injection: Für jede aktivierte Section: Polished Text einsetzen, Bilder einsetzen (bereits optimiert aus Stufe 9), CTA-Text und Link einsetzen

4.  Meta-Tag-Injection: Title, Description, OG-Tags aus Stufe 9 einsetzen. Schema Markup in \<head\> einfügen

5.  Formular-Setup: Kontaktformular mit Formspree oder eigenem Handler konfigurieren. Ziel-Email aus Enrichment-Daten

6.  Maps-Integration: Google Maps Embed mit Firmenadresse

7.  Analytics-Setup: Plausible Analytics Script einfügen (DSGVO-konform, kein Cookie-Banner nötig)

8.  Build: next build. Bei Erfolg: Deploybare App bereit. Bei Fehler: Error-Handling (siehe 10.4)

10.2 Performance Budget (Enforced at Build)

Die generierte Website MUSS besser performen als die alte. Das ist eines unserer stärksten Verkaufsargumente. Deshalb werden Performance-Constraints beim Build enforced:

  -------------------------------- ------------------------------- --------------------------------- ----------------------------------------------------------
  **Constraint**                   **Zielwert**                    **Wie enforced**                  **Bei Verletzung**

  Lighthouse Performance           \> 90                           Lighthouse CI im Build-Schritt    Build blockiert. Bilder re-optimieren, Code re-splitten

  Largest Contentful Paint (LCP)   \< 2.5s                         Lighthouse CI                     Hero-Bild zu groß? WebP + Lazy-Loading prüfen

  Cumulative Layout Shift (CLS)    \< 0.1                          Lighthouse CI                     Fehlende Bild-Dimensionen? Font-Loading-Strategie prüfen

  First Input Delay (FID)          \< 100ms                        Lighthouse CI                     Zu viel JavaScript? Code-Splitting prüfen

  Total Page Weight                \< 1.5 MB                       Build-Script misst Bundle-Größe   Bilder weiter komprimieren. Fonts subsetten

  Time to Interactive (TTI)        \< 3.5s                         Lighthouse CI                     SSR prüfen. Unnötige Client-Side JS entfernen

  Image Format                     100% WebP (mit JPEG Fallback)   Build-Script prüft                Bilder automatisch konvertieren

  Font Loading                     font-display: swap              CSS-Check im Build                Automatisch setzen
  -------------------------------- ------------------------------- --------------------------------- ----------------------------------------------------------

10.3 Accessibility-Constraints

  ---------------------------------------- ---------------- ------------------------------------------------------------- ----------------------
  **Anforderung**                          **WCAG-Level**   **Wie sichergestellt**                                        **Automatisierbar?**

  Alt-Texte auf allen Bildern              A                Aus Stufe 9 generiert. Build prüft Vollständigkeit            Ja

  Farbkontrast \> 4.5:1 (Text)             AA               Beim Brand-Application: Kontrast-Check der gewählten Farben   Ja

  Keyboard-Navigation                      A                Template-Basis hat bereits Tab-Reihenfolge + Focus-Styles     Ja (Template)

  ARIA-Labels auf interaktiven Elementen   A                Template-Basis enthält ARIA für Nav, Formular, Buttons        Ja (Template)

  Heading-Hierarchie (H1 \> H2 \> H3)      A                Content-Injection prüft Hierarchie                            Ja

  Skip-to-Content-Link                     AA               Im Base-Template integriert                                   Ja (Template)

  Formular-Labels                          A                Template-Formular hat Labels auf allen Inputs                 Ja (Template)

  Keine Auto-Play-Videos/Audio             A                Template erlaubt kein Autoplay                                Ja (Template)
  ---------------------------------------- ---------------- ------------------------------------------------------------- ----------------------

Lighthouse Accessibility Score Ziel: \> 90. Bei \< 85: Build blockiert.

10.4 Error Handling

  ------------------------------------ ---------------------------------- ------------------------------------------------------------------------------------------------------------ ------------------------------------------------------
  **Failure Mode**                     **Erkennung**                      **Automatische Behebung**                                                                                    **Eskalation**

  Build-Error (Next.js Compile Fail)   Exit Code ≠ 0                      Error-Log analysieren. Häufigste Ursache: Ungültige Zeichen in Content. Automatisches Sanitizing + Rebuild   2\. Build-Fail: Manuelles Review

  Lighthouse Performance \< 90         Lighthouse CI Report               Bilder re-komprimieren (60% statt 80%). Fonts subsetten. Nicht-kritisches JS deferred laden. Re-Build        3\. Fail: Template-Simplification (weniger Sections)

  Lighthouse a11y \< 85                Lighthouse CI Report               Fehlende Alt-Texte ergänzen. Kontrast-Fixes (dunklere Textfarbe). ARIA ergänzen                              Selten -- Template sollte Basis sichern

  Bild fehlt (Reference Error)         Build-Log: Missing Asset           Platzhalter-Bild einsetzen. Flag für QA                                                                      Kein Blocker, aber QA prüft

  Formular sendet nicht                Test-Submission im Build-Schritt   Formspree-Config prüfen. API-Key rotieren. Fallback: mailto-Link                                             Blocker -- Formular MUSS funktionieren

  Maps lädt nicht                      Embed-Test im Build                Google Maps API Key prüfen. Fallback: Statisches Karten-Bild mit Link                                        Kein Blocker
  ------------------------------------ ---------------------------------- ------------------------------------------------------------------------------------------------------------ ------------------------------------------------------

10.5 KPIs & Monitoring

  ------------------------------------------------ -------------------- ------------------------------
  **Metrik**                                       **Zielwert**         **Alert wenn**

  Build-Erfolgsrate (1. Versuch)                   \> 85%               \< 70%

  Build-Erfolgsrate (nach Auto-Fix)                \> 98%               \< 90%

  Avg. Build-Dauer                                 \< 90s               \> 180s

  Lighthouse Performance (Avg)                     \> 92                \< 88

  Lighthouse Accessibility (Avg)                   \> 93                \< 88

  Avg. Page Weight                                 \< 1.2 MB            \> 1.8 MB

  Template-Variation (keine Duplikate pro Stadt)   100%                 \< 100%

  Cost pro Build                                   \< €0.02 (Compute)   \> €0.05
  ------------------------------------------------ -------------------- ------------------------------

+--------+----------------------------------------------------------------------------------------------+
| **11** | **QA & Testing**                                                                             |
|        |                                                                                              |
|        | *Harter Gate vor Deploy: Technik + Content + Legal + Visual -- kein Fehler darf durchkommen* |
+--------+----------------------------------------------------------------------------------------------+

+--------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------------------+
| **INPUT**                                                                                                    | **OUTPUT**                                                                                                                                                        | **QUALITY GATE**                                                                                        |
|                                                                                                              |                                                                                                                                                                   |                                                                                                         |
| Gebaute Website-App (aus Stufe 10) + Rebuild Plan (für Content-Verifikation) + Legal Checklist (aus Stufe 8) | QA Report: • test_results\[\]{category, test, pass/fail, details} • blocker_count • warning_count • overall_verdict (PASS/CONDITIONAL/FAIL) • qa_confidence (0-1) | 0 Blocker. Alle Legal-Checks bestanden. Visual-Quality-Check bestanden. Overall: PASS oder CONDITIONAL. |
+--------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------------------+

11.1 Die 5 QA-Dimensionen

+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **QA ist der letzte Schutzwall vor dem Kunden**                                                                                                                                                                                                                                       |
|                                                                                                                                                                                                                                                                                       |
| Wenn hier ein Fehler durchrutscht, sieht der Lead ihn. Ein kaputtes Formular, ein toter Link, ein falscher Name, ein Rechtschreibfehler im Hero-Text -- jeder einzelne Fehler zerstört die Illusion einer professionell erstellten Website. QA muss deshalb gnadenlos gründlich sein. |
+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

  --------------------------- -------------------------------------------------------------------------- ---------------------------------- -------------------------------------------------------------
  **Dimension**               **Was geprüft wird**                                                       **Automatisierbar?**               **Blocker-Potenzial**

  **1. Funktional**           Links, Formulare, Navigation, Maps, Telefon-Links, Scroll-Verhalten        95% automatisiert                  Hoch -- kaputtes Formular = kein Lead

  **2. Performance**          Lighthouse Scores, Core Web Vitals, Page Weight, Ladezeit                  100% automatisiert                 Mittel -- langsam aber funktional ist akzeptabel

  **3. Content-Integrität**   Fakten stimmen, kein Lorem Ipsum, keine leeren Sections, Rechtschreibung   80% automatisiert, 20% AI-Check    Hoch -- falscher Name = Deal-Killer

  **4. Legal Compliance**     Impressum-Link, Datenschutz-Link, Cookie-Banner, Branchenspezifisches      90% automatisiert                  Hoch -- fehlendes Impressum = rechtliches Risiko für Kunden

  **5. Visual Quality**       Sieht es professionell aus? Branchenpassend? Keine Darstellungsfehler?     30% automatisiert, 70% AI-Vision   Mittel-Hoch -- unprofessioneller Look = kein Kauf
  --------------------------- -------------------------------------------------------------------------- ---------------------------------- -------------------------------------------------------------

11.2 Funktionale Test-Suite

  ----------------------------------- -------------------------------------------------- ------------------------------------------ --------------------------------------------------
  **Test**                            **Methode**                                        **Pass-Kriterium**                         **Bei Fail**

  Alle internen Links funktionieren   Playwright: Alle \<a href\> prüfen                 0 broken Links (404/500)                   BLOCKER: Broken Link fixen oder entfernen

  Kontaktformular sendet              Playwright: Test-Submission + Email-Verification   Test-Email kommt an                        BLOCKER: Formspree-Config prüfen

  Telefon-Links korrekt               Regex-Check: tel: Format                           Alle tel: Links haben gültige Nummer       BLOCKER: Nummer aus Enrichment-Daten korrigieren

  Email-Links korrekt                 Regex-Check: mailto: Format                        Alle mailto: Links valide                  BLOCKER: Korrigieren

  Navigation funktioniert             Playwright: Alle Nav-Items klicken                 Jeder Nav-Link führt zur richtigen Seite   BLOCKER: Routing fixen

  Maps-Embed lädt                     Playwright: iframe Check                           Map sichtbar mit korrekter Adresse         WARNING: Statisches Fallback-Bild

  Scroll-Verhalten                    Playwright: Smooth Scroll Test                     Anchor-Links scrollen korrekt              WARNING: CSS Fix

  Responsive Breakpoints              Playwright: 375px, 768px, 1024px, 1440px           Kein Content überlappt oder versteckt      BLOCKER bei 375px/768px, WARNING bei Rest

  404-Seite existiert                 HTTP Request an /nonexistent                       Custom 404-Seite wird gezeigt              WARNING: Default 404 akzeptabel

  Favicon vorhanden                   HTTP Request an /favicon.ico                       Favicon lädt                               WARNING: Kein Blocker
  ----------------------------------- -------------------------------------------------- ------------------------------------------ --------------------------------------------------

11.3 Content-Integritäts-Tests

  ------------------------------------------ ------------------------------------------------------ --------------------------------------------- -------------------------------------------------
  **Test**                                   **Methode**                                            **Pass-Kriterium**                            **Bei Fail**

  Firmenname korrekt                         Text-Match gegen Enrichment-Daten                      Exakter Match auf jeder Seite                 BLOCKER: Sofort korrigieren

  Telefonnummer korrekt                      Text-Match gegen Enrichment                            Nummer auf Website = Nummer in Enrichment     BLOCKER: Korrigieren

  Adresse korrekt                            Text-Match + Maps-Verify                               Adresse stimmt überein                        BLOCKER: Korrigieren

  Kein Lorem Ipsum / Platzhalter-Text        Regex-Scan: lorem, ipsum, \[PLATZHALTER\], TODO, XXX   0 Treffer                                     BLOCKER: Sofort ersetzen oder Section entfernen

  Keine leeren Sections                      DOM-Analyse: Sections mit \<50px Höhe                  Alle sichtbaren Sections haben Content        BLOCKER: Section entfernen oder füllen

  Rechtschreibung                            LanguageTool API (DE)                                  \< 3 Fehler pro 1000 Wörter                   WARNING: Auto-Korrektur, dann Re-Check

  Kein Duplicate Content zwischen Sections   AI: Textvergleich zwischen Sections                    Keine Section wiederholt Text einer anderen   WARNING: De-Duplizieren

  Bilder haben Alt-Texte                     DOM-Scan: img ohne alt                                 100% der Bilder haben alt                     WARNING: Alt-Text nachgenerieren

  Keine kaputten Bilder                      HTTP-Check aller img src                               Alle Bilder laden                             BLOCKER: Platzhalter oder Entfernen
  ------------------------------------------ ------------------------------------------------------ --------------------------------------------- -------------------------------------------------

11.4 Legal Compliance Tests

  ------------------------------------ -------------------------------------------------------------- -------------------------------------------------------------------------- --------------------------------------------------------
  **Test**                             **Methode**                                                    **Pass-Kriterium**                                                         **Bei Fail**

  Impressum-Link vorhanden             DOM: Footer enthält Link mit Text „Impressum"                  Link existiert und ist klickbar                                            BLOCKER: Link zur bestehenden Impressum-Seite einfügen

  Impressum-Link funktioniert          Playwright: Link klicken                                       Seite lädt (200 oder externe URL)                                          BLOCKER: URL korrigieren

  Datenschutz-Link vorhanden           DOM: Footer enthält Link mit Text „Datenschutz"                Link existiert und klickbar                                                BLOCKER: Link einfügen

  Cookie-Consent (wenn Tracking)       DOM: Cookie-Banner vorhanden wenn Plausible o.ä. geladen       Plausible = kein Banner nötig. Bei anderen: Banner da                      BLOCKER wenn Drittanbieter-Tracking ohne Consent

  Keine erfundenen Testimonials        Cross-Check: Testimonial-Autoren gegen Extraction              Jeder Testimonial-Autor existiert in extrahierten Daten                    BLOCKER: Testimonial entfernen

  Branchenspezifische Pflichtangaben   Checkliste aus Stufe 8 abarbeiten                              Alle Pflicht-Items als „vorhanden" oder „auf bestehender Seite verlinkt"   BLOCKER: Angabe ergänzen oder Link setzen

  Keine HWG-Verstöße (Ärzte)           AI-Scan auf Heilversprechen                                    0 HWG-verdächtige Formulierungen                                           BLOCKER: Formulierung entschärfen

  Bildrechte                           Alle Bilder: eigene ODER NanoBanana ODER explizit lizenzfrei   Keine ungeklärten Bilder                                                   BLOCKER: Bild entfernen
  ------------------------------------ -------------------------------------------------------------- -------------------------------------------------------------------------- --------------------------------------------------------

11.5 Visual Quality Check (AI-basiert)

Der schwierigste Test: Sieht die Website gut aus? Automatische Tests können technische Fehler finden, aber keine ästhetischen Probleme. Dafür brauchen wir AI Vision.

AI Visual QA Prompt

+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Visual QA Prompt (vereinfacht)**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
|                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| Du bist ein Senior Web Designer der eine Website vor dem Launch reviewed. Prüfe diese Screenshots (Desktop + Mobile) für ein \[INDUSTRIE\]-Business: 1. PROFESSIONELLER GESAMTEINDRUCK: Würde ein Geschäftsinhaber sagen „Das sieht professionell aus"? Gibt es etwas das sofort stört? 2. DARSTELLUNGSFEHLER: Überlappende Elemente? Abgeschnittener Text? Falsche Spaltenbreiten? Bilder verzerrt? 3. KONSISTENZ: Farben durchgehend gleich? Abstande konsistent? Schriftgrößen logische Hierarchie? 4. MOBILE: Ist Mobile benutzbar? Buttons groß genug? Text lesbar ohne Zoomen? 5. BRANCHENFIT: Passt der Look zu \[INDUSTRIE\]? Farbschema angemessen? Bildsprache passend? 6. CONVERSION: Ist der CTA sichtbar? Würde ein Besucher wissen was er als nächstes tun soll? Bewerte jeden Punkt 0-100 und beschreibe Probleme konkret. |
+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

Visual QA Ergebnis-Handling

  ----------------------- ------------------ -------------------------------------------------------------------------------------------------------------------------
  **Avg. Visual Score**   **Ergebnis**       **Aktion**

  **\> 80**               PASS               Weiter zu Stufe 12

  **60-80**               CONDITIONAL PASS   Weiter, aber mit Flag. Wenn AI konkrete Darstellungsfehler benennt: automatische CSS-Fixes versuchen. Re-Check nach Fix

  **40-60**               REVIEW REQUIRED    Manuelles Review. Spezifische AI-Kritik als Briefing. Häufig: Farbkontrast-Problem, Mobile-Darstellung, Bild-Layout

  **\< 40**               FAIL               Zurück zu Stufe 10 mit Anweisung: Template-Variant wechseln. Oder: komplett anderes Template wählen
  ----------------------- ------------------ -------------------------------------------------------------------------------------------------------------------------

11.6 Blocker/Warning-Logik & Auto-Repair

  --------------- ------------------------------------------------------------------------------------- ------------------------------- ----------------------------------------------------------
  **Kategorie**   **Definition**                                                                        **Max. Auto-Repair-Versuche**   **Nach Max Retries**

  **BLOCKER**     Fehler der den Lead sieht und der Vertrauen zerstört. Website darf nicht live gehen   3 automatische Fix-Versuche     Pipeline stoppt. Manuelles Review-Queue

  **WARNING**     Fehler der die Qualität mindert aber Website nutzbar lässt                            2 automatische Fix-Versuche     Weiter mit Flag. Wird in Post-Deploy-Monitoring getrackt

  **INFO**        Optimierungsvorschlag ohne kritische Auswirkung                                       0 (nur Logging)                 Dokumentiert für spätere Verbesserung
  --------------- ------------------------------------------------------------------------------------- ------------------------------- ----------------------------------------------------------

Auto-Repair-Ablauf

1.  Test-Suite läuft. Ergebnis: Liste von Blockern, Warnings, Infos

2.  Für jeden Blocker: Auto-Repair-Strategie anwenden (z.B. broken Link → Link entfernen, Bild fehlt → Platzhalter einsetzen, Rechtschreibfehler → LanguageTool Fix)

3.  Re-Build nach Repairs

4.  Test-Suite läuft erneut. Gleiche Blocker → nächster Repair-Versuch (max 3)

5.  Nach 3 Fails: Pipeline stoppt für diesen Lead. Manuelles Review. Alert ans Team

6.  Bei 0 Blockern: Overall Verdict = PASS (0 Warnings) oder CONDITIONAL (Warnings vorhanden). Weiter zu Stufe 12

11.7 KPIs & Monitoring

  ------------------------------------------ ----------------- ---------------------------------
  **Metrik**                                 **Zielwert**      **Alert wenn**

  First-Pass-Rate (0 Blocker beim 1. Test)   \> 75%            \< 55%

  Auto-Repair-Success-Rate                   \> 90%            \< 70%

  Manual Review Rate (Pipeline stoppt)       \< 5%             \> 15% (systematisches Problem)

  Avg. QA-Dauer pro Website                  \< 120s           \> 300s

  Avg. Blocker-Count pro Website (1. Test)   \< 2              \> 5

  Legal Compliance Pass Rate                 \> 99%            \< 95% (Template-Problem)

  Visual QA Avg. Score                       \> 75             \< 60

  Avg. Repair-Zyklen pro Website             \< 1.5            \> 3
  ------------------------------------------ ----------------- ---------------------------------

+--------+---------------------------------------------------------------------------------+
| **12** | **Preview Deployment**                                                          |
|        |                                                                                 |
|        | *Die Preview ist kein Website-Deploy -- sie ist eine geführte Sales-Experience* |
+--------+---------------------------------------------------------------------------------+

+-------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------------------------------------------------+--------------------------------------------------------------------------------------------------+
| **INPUT**                                                                                       | **OUTPUT**                                                                                                                          | **QUALITY GATE**                                                                                 |
|                                                                                                 |                                                                                                                                     |                                                                                                  |
| QA-bestandene Website-App + Quality Report (Stufe 5) + Competitor Report (Stufe 6) + Lead-Daten | Live Preview: • {firma}.webolution.de • SSL aktiv • Sales-Elemente integriert • Analytics tracking • preview_url • deploy_timestamp | URL erreichbar (200). SSL aktiv. Sales-Banner sichtbar. Analytics-Events feuern. No-Index aktiv. |
+-------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------------------------------------------------+--------------------------------------------------------------------------------------------------+

12.1 Preview ≠ Website -- Preview = Sales Pitch

+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Der Perspektivenwechsel**                                                                                                                                                                                                                                                                                                                                                                                            |
|                                                                                                                                                                                                                                                                                                                                                                                                                        |
| Die Preview ist nicht einfach „eine gehostete Website". Sie ist das wichtigste Conversion-Tool im gesamten Funnel. Der Lead sieht zum ersten Mal, wie sein Business online aussehen KÖNNTE. Dieser Moment muss perfekt inszeniert sein. Die Preview muss eine GESCHICHTE erzählen: „So sieht Ihre Website heute aus \[Problem\] → So könnte sie aussehen \[Lösung\] → Und das können Sie in 48 Stunden haben \[CTA\]." |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

12.2 Technisches Deployment

-   Platform: Vercel mit Wildcard-Domain \*.webolution.de und automatischem SSL

-   Subdomain-Generierung: Firmenname slugifiziert. „Kanzlei Dr. Müller & Partner" → kanzlei-dr-mueller-partner.webolution.de. Duplikat-Check: Wenn Slug existiert, Suffix anhängen (-2)

-   Deploy via Vercel API: Git Push → Auto-Build → Live in \< 2 Minuten

-   Preview-TTL: 30 Tage aktiv. Danach automatisch deaktiviert (301 → Landing-Page „Ihre Vorschau ist abgelaufen" mit CTA)

-   No-Index: robots meta tag noindex, nofollow. Preview darf nicht bei Google auftauchen

12.3 Sales-Experience-Layer

Auf die fertige Website wird ein Sales-Layer gelegt. Dieser besteht aus 5 Elementen:

Element 1: Sticky-Top-Banner

-   Immer sichtbar, auch beim Scrollen. Nicht wegklickbar (nur minimierbar)

-   Text: „Dies ist eine kostenlose Website-Vorschau für \[Firmenname\]. Gefällt Ihnen was Sie sehen?"

-   CTA-Button: „Jetzt übernehmen" → Link zur personalisierten Pricing-Page (Stufe 15)

-   Sekundärer Link: „Fragen? 15-Min-Gespräch buchen" → Calendly

Element 2: Vorher/Nachher-Vergleich

-   Zugriff über Button: „Ihre aktuelle Website vergleichen"

-   Split-Screen-Ansicht: Links = Screenshot der alten Website (aus Stufe 4). Rechts = neue Preview

-   Desktop + Mobile Toggle: „Auf dem Handy vergleichen" zeigt Mobile-Screenshots nebeneinander

-   Annotationen: Rote Markierungen auf dem alten Screenshot bei den Top-3-Schwachstellen

-   Technisch: Overlay-Modal das über die Preview gelegt wird. Nicht navigierbar (ist nur ein Bild des alten Zustands)

Element 3: Analyse-Section (am Ende der Preview)

-   Nach dem regularen Website-Content kommt eine zusätzliche Section: „Was wir für \[Firmenname\] verbessert haben"

-   3-5 konkrete Verbesserungen mit Vorher/Nachher: z.B. „Ihre Website hatte kein SSL → Unsere Version ist komplett verschlüsselt", „Kein Schema Markup → Vollständiges \[Industrie\]-Schema für bessere Google-Darstellung"

-   PageSpeed-Vergleich: „Ihre Website: 23/100 → Unsere Version: 95/100"

-   Competitor-Teaser: „Ihre Konkurrenz \[Konkurrent-Name\] hat bereits \[Feature\]. Mit dieser Website schließen Sie die Lücke."

Element 4: Trust-Elemente

-   Ab Phase 2 (wenn echte Kunden vorhanden): „Bereits X Unternehmen in \[Stadt/Region\] nutzen Webolution"

-   Ab Phase 1 alternativ: „Technologie-Badges": Next.js, SSL, DSGVO-konform, Google-optimiert

-   Geld-zurück-Garantie (wenn im Pricing enthalten): „30 Tage Geld-zurück wenn Sie nicht zufrieden sind"

Element 5: Scarcity & Countdown

-   Countdown-Timer: „Diese Vorschau ist noch \[X\] Tage verfügbar"

-   TTL-Berechnung: 30 Tage ab Deploy. Countdown wird live berechnet (nicht gefaked)

-   Psychologie: Kein aggressives „NUR NOCH HEUTE". Sondern sachlich: „Wir können Vorschauen nicht unbegrenzt hosten. Diese ist bis \[Datum\] verfügbar."

-   Nach Ablauf: 301-Redirect auf eine Landing-Page: „Ihre Vorschau ist abgelaufen. Kontaktieren Sie uns für eine neue."

12.4 Preview-Schutz

  ------------------------- ------------------------------------------------------------- ---------------------------------------- --------------------------------------------
  **Maßnahme**              **Zweck**                                                     **Aufwand**                              **Effektivität**

  No-Index, No-Follow       Nicht von Google indexiert                                    Minimal (Meta-Tag)                       100% (Google respektiert das)

  Watermark auf Bildern     Dezentes „VORSCHAU -- webolution.de" auf NanoBanana-Bildern   Automatisiert beim Build                 Mittel (hindert Copy, nicht Screenshot)

  Right-Click Disabled      Erschwert Copy-Paste von Text und Bildern                     JS-Snippet                               Niedrig (leicht umgehbar)

  Minified + Bundled Code   Quellcode nicht einfach lesbar                                Standard bei Next.js Build               Mittel (für Laien effektiv)

  Kein Public Git-Repo      Kein Zugang zum Source Code                                   Standard (Vercel deploys sind private)   Hoch

  Analytics-Tracking        Wir sehen wer die Preview wie nutzt                           Plausible Integration                    Indirekt -- kein Schutz, aber Intelligence
  ------------------------- ------------------------------------------------------------- ---------------------------------------- --------------------------------------------

+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Pragmatische Philosophie zum Kopierschutz**                                                                                                                                                                                                                                                                                                                                                         |
|                                                                                                                                                                                                                                                                                                                                                                                                       |
| Vollständiger Kopierschutz ist unmöglich und den Versuch nicht wert. Ein technisch versierter Mensch kann jede Website kopieren. Aber: Unsere Zielgruppe IST NICHT technisch versiert. Ein Friseur oder Anwalt wird keine Website reverse-engineeren. Der echte Wert liegt im Managed Service: Domain-Switch, Email-Routing, Formular-Setup, laufendes Hosting, Support. Das kann man nicht kopieren. |
+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

12.5 Preview-Analytics & Behavior-Tracking

Jede Interaktion mit der Preview wird getrackt und steuert die Outreach-Logik in Stufe 14:

  ----------------------------------- ----------------------------------- ---------------------------------------------- -------------------------------------------------------------
  **Event**                           **Tracking-Methode**                **Bedeutung**                                  **Auswirkung auf Outreach**

  Preview geöffnet                    Page Load Event                     Lead hat die Email gelesen und ist neugierig   Email 2: Engagement-Follow-up statt Reminder

  Scroll-Tiefe \> 75%                 Scroll-Tracking                     Lead hat die meiste Website gesehen            Hohe Kaufwahrscheinlichkeit → CTA-fokussiertes Follow-up

  Vergleich angeklickt                Click Event auf Vergleichs-Button   Lead vergleicht aktiv alt vs. neu              Sehr hohes Interesse → Persönliches Angebot

  CTA geklickt („Jetzt übernehmen")   Click Event auf CTA                 Kaufabsicht! Aber noch nicht konvertiert       Abandoned-Cart-Logik: Email mit Pricing-Summary + Calendly

  Return Visit (2. oder 3. Besuch)    Cookie/Fingerprint                  Lead kommt wieder -- erwägt Kauf               Nach 3. Visit ohne Kauf: Telefonangebot

  Verweildauer \> 3 Minuten           Time-on-Page                        Intensives Studieren der Preview               Qualifizierter Lead -- ggf. direkter Telefonkontakt

  Analyse-Section erreicht            Scroll Event                        Lead liest die Schwachstellen-Analyse          Follow-up fokussiert auf die spezifischen Schwachstellen

  Calendly geklickt                   Click Event                         Lead will persönlich reden                     Kein weiteres automatisches Follow-up. Sales-Team übernimmt
  ----------------------------------- ----------------------------------- ---------------------------------------------- -------------------------------------------------------------

Behavior-basiertes Lead-Scoring-Update

Preview-Engagement-Daten fließen zurück in den Lead-Score (Stufe 7 Feedback Loop):

-   Preview geöffnet + Scroll \> 50% → Score +5

-   Vergleich angeklickt → Score +8

-   CTA geklickt → Score +15

-   Return Visit → Score +10 pro Visit (max +30)

-   Keine Öffnung nach 7 Tagen → Score -5 (Lead hat kein Interesse oder Email nicht gesehen)

12.6 Error Handling

  ------------------------------------ ------------------------------- ------------------------------------------------------------------------------------------
  **Failure Mode**                     **Erkennung**                   **Fallback**

  Vercel Deploy fehlgeschlagen         Vercel API Error Response       Retry (max 3). Bei persistentem Fail: Alternative Deployment auf Netlify. Alert ans Team

  SSL-Zertifikat nicht provisioniert   HTTPS-Check 5 Min nach Deploy   Vercel SSL-Retry. Fallback: Cloudflare SSL Proxy. Max 30 Min warten

  Subdomain-Kollision                  Slug existiert bereits          Suffix anhängen: -2, -3. Bei \>3: Firmenort als Suffix (kanzlei-mueller-muenchen)

  Preview lädt langsam (\>5s)          Lighthouse-Check nach Deploy    CDN-Cache prüfen. Bilder nochmals optimieren. Vercel Edge-Region prüfen

  Analytics feuert nicht               Test-Event nach Deploy          Plausible-Script prüfen. Fallback: Eigenes Minimal-Tracking

  Sales-Banner nicht sichtbar          DOM-Check nach Deploy           CSS-Injection prüfen. Z-Index erhöhen
  ------------------------------------ ------------------------------- ------------------------------------------------------------------------------------------

12.7 KPIs & Monitoring

  -------------------------------------------- ------------------------------ -----------------------------
  **Metrik**                                   **Zielwert**                   **Alert wenn**

  Deploy-Erfolgsrate                           \> 99%                         \< 95%

  Time-to-Live (Push → URL erreichbar)         \< 3 Minuten                   \> 10 Minuten

  SSL-Provisioning-Rate                        \> 99%                         \< 97%

  Preview-Uptime (30 Tage)                     \> 99.5%                       \< 98%

  Sales-Banner-Sichtbarkeit                    100%                           \< 100%

  Analytics-Event-Rate (% Visits mit Events)   \> 95%                         \< 80%

  Avg. aktive Previews gleichzeitig            Tracking (Kosten-Management)   \> 500 (Vercel-Plan prüfen)

  Cost pro Preview (30 Tage Hosting)           \< €0.50                       \> €2.00
  -------------------------------------------- ------------------------------ -----------------------------

Nächste Iteration: Steps 13-17

Dieses Dokument deckt Steps 9-12 ab. Die finale Iteration spezifiziert:

-   Step 13: Outreach Preparation -- Personalisierte Assets, Email-Validierung, Multi-Channel-Strategie, Analyse-PDF-Generierung

-   Step 14: Email Outreach -- Behavioral Branching, Hyper-Personalisierung, Multi-Channel-Fallback, Compliance

-   Step 15: Conversion & Payment -- Trust-Building, Objection Handling, Pricing-Psychologie, Sales-Gespräch-Integration

-   Step 16: Managed Onboarding & Go-Live -- DNS-Automation, Email-Routing-Schutz, Rollback-Plan, End-to-End-Test

-   Step 17: Post-Launch Monitoring & Customer Success -- Uptime, Upsell-Trigger, Retention, Referral, Re-Engagement

*Webolution Deep Spec -- Steps 9-12 -- SocialCooks*
