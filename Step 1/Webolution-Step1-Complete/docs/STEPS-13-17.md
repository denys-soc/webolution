**WEBOLUTION**

Deep Spec: Steps 13--17

*Outreach → Conversion → Payment → Onboarding → Customer Success*

Vom Lead zum zahlenden Kunden -- und darüber hinaus

**SocialCooks × Webolution**

März 2026

+--------+-------------------------------------------------------------------------------------+
| **13** | **Outreach Preparation**                                                            |
|        |                                                                                     |
|        | *Personalisierte Assets generieren + Email-Validierung + Kanal-Strategie festlegen* |
+--------+-------------------------------------------------------------------------------------+

+----------------------------------------------------------------------------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------------------------+
| **INPUT**                                                                                          | **OUTPUT**                                                                                                                                                                                                               | **QUALITY GATE**                                                                                            |
|                                                                                                    |                                                                                                                                                                                                                          |                                                                                                             |
| Scored Lead + Quality Report + Competitor Report + Preview URL + Industry Profile + Decision Maker | Outreach Package: • email_sequence\[5\]{subject, body, personalization_vars} • analysis_pdf (1-Seiter) • comparison_screenshots{desktop, mobile} • validated_email_address • channel_strategy • personalized_pricing_url | Email-Adresse validiert (not bounced). Alle Assets generiert. Personalisierung vollständig (0 Platzhalter). |
+----------------------------------------------------------------------------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------------------------+

13.1 Email-Validierung (vor dem Versand)

Jede unzustellbare Email schadet unserer Sender-Reputation. Bei \>5% Bounce-Rate stufen ESPs wie Gmail uns als Spam ein. Deshalb: Zero-Tolerance für invalid Emails.

  ----------------------------- -------------------------------------------- --------------------------------------- ------------------------------------------------------------------------------
  **Check**                     **Tool/Methode**                             **Ergebnis**                            **Aktion**

  MX-Record vorhanden?          DNS MX Lookup                                MX existiert / nicht                    Kein MX = Email kann nicht zugestellt werden → Sekundärkanal

  Mailbox existiert?            SMTP Verification (NeverBounce/ZeroBounce)   Valid / Invalid / Catch-All / Unknown   Invalid → nächste Email aus Enrichment. Catch-All → riskant, trotzdem senden

  Disposable/Temporary?         Disposable-Email-DB                          Ja / Nein                               Disposable → Sekundärkanal verwenden

  Spam-Trap?                    Blacklist-DB-Check                           Ja / Nein                               Spam-Trap → NICHT senden. Lead behalten, andere Kontaktmethode

  Role-based? (info@, admin@)   Prefix-Analyse                               Role / Personal                         Role = OK, aber niedrigere Open-Rate erwartet
  ----------------------------- -------------------------------------------- --------------------------------------- ------------------------------------------------------------------------------

Fallback-Strategie bei invalider Email

1.  Fallback 1: Alternative Email aus Enrichment verwenden (z.B. info@ wenn persönliche invalid)

2.  Fallback 2: Kontaktformular der bestehenden Website als Kanal (automatische Submission via Playwright)

3.  Fallback 3: LinkedIn InMail (nur bei Coaches, Anwälte mit LinkedIn-Profil)

4.  Fallback 4: Instagram DM (nur bei Friseure, Coaches mit aktivem Instagram)

5.  Fallback 5: Physischer Brief (Premium-Kanal für High-Score-Leads ohne digitalen Kontaktweg)

13.2 Hyper-Personalisierung der Emails

+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Der Unterschied zwischen Cold Email und Hyper-Personalized Email**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
|                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| Cold Email: „Wir haben Ihre Website gesehen und würden gerne ein Redesign anbieten." → 3% Open Rate, 0.5% Reply. Hyper-Personalized: „Herr Dr. Müller, mir ist aufgefallen, dass Ihre Spezialisierung auf Familienrecht auf der Startseite Ihrer Kanzlei nicht vorkommt -- dabei suchen 73% der Mandanten in München genau danach. Kanzlei Schmidt in der Maximilianstraße zeigt ihre Rechtsgebiete prominent. Wir haben eine Vorschau gebaut, wie das für Sie aussehen könnte." → 45% Open Rate, 8% Reply. Der Unterschied: Spezifische Details die zeigen „Wir haben uns WIRKLICH mit deinem Business beschäftigt." |
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

Personalisierungs-Variablen

  ---------------------------- ------------------------------------ --------------------------------------------------------------------- --------------------------------
  **Variable**                 **Quelle**                           **Beispiel**                                                          **Pflicht?**

  {{decision_maker_name}}      Stufe 3: Enrichment                  Dr. Thomas Müller                                                     Ja (Fallback: Firmennname)

  {{decision_maker_title}}     Stufe 3: Enrichment                  RA / Dr. / Prof.                                                      Nein (Nice-to-have für Anrede)

  {{firm_name}}                Stufe 4: Extraction                  Kanzlei Müller & Partner                                              Ja

  {{city}}                     Stufe 3: Enrichment                  München                                                               Ja

  {{top_weakness}}             Stufe 5: Quality Report (Klasse A)   Nicht mobiloptimiert                                                  Ja

  {{weakness_detail}}          Stufe 5: Outreach Hook               67% Ihrer Besucher sind mobil und sehen eine nicht nutzbare Website   Ja

  {{competitor_name}}          Stufe 6: Competitor Report           Kanzlei Schmidt                                                       Nein (stark wenn verfügbar)

  {{competitor_advantage}}     Stufe 6: Feature Gap                 Online-Terminbuchung + 89 Google Reviews auf der Startseite           Nein

  {{preview_url}}              Stufe 12: Deploy                     kanzlei-mueller.webolution.de                                         Ja

  {{pricing_url}}              Generiert                            webolution.de/go/lead-abc123                                          Ja

  {{industry_specific_stat}}   Industry Profile                     73% der Mandanten suchen online nach Anwälten                         Nein (stark wenn verfügbar)

  {{google_rating}}            Stufe 3: Enrichment                  4.2 Sterne (28 Bewertungen)                                           Nein
  ---------------------------- ------------------------------------ --------------------------------------------------------------------- --------------------------------

13.3 Analyse-PDF (1-Seiter)

Jeder Lead bekommt ein personalisiertes PDF als Email-Attachment. Das PDF dient als „Leave-Behind" das der Lead speichern und intern weiterleiten kann.

PDF-Struktur

1.  Header: Webolution-Logo + „Website-Analyse für \[Firmenname\]" + Datum

2.  Section 1: Aktueller Website-Score: Kreisdiagramm mit Gesamtscore (z.B. 35/100). Subscores: Technik, Design, Content, SEO. Vergleich zum Branchendurchschnitt (Balkendiagramm)

3.  Section 2: Top 3 Schwachstellen: Je 1 Zeile mit Icon + Schwachstelle + Impact-Erklärung. Nur Klasse-A und Klasse-B Schwachstellen. In Alltagssprache, nicht technisch

4.  Section 3: Competitor-Vergleich (Mini): Tabelle: \[Firmenname\] vs. Bester Konkurrent. 4-5 Zeilen mit Check/Cross-Icons

5.  Section 4: Vorschau-Screenshot: Desktop-Screenshot der neuen Website mit Bildunterschrift „So könnte Ihre neue Website aussehen"

6.  Footer: Preview-URL + QR-Code zum Preview-Link + „Fragen? \[Calendly-Link\]" + Webolution-Kontaktdaten

Das PDF wird automatisch generiert (HTML-to-PDF via Puppeteer). Template pro Industrie mit angepasster Farbgebung und Sprache.

13.4 Vergleichs-Screenshots

-   Alt vs. Neu Side-by-Side: Desktop (1440px) und Mobile (375px) Screenshots

-   Alte Website: Screenshot aus Stufe 4 (Crawling)

-   Neue Website: Screenshot der Preview (aus Stufe 12, ohne Sales-Banner)

-   Annotation: Rote Pfeile auf dem alten Screenshot bei offensichtlichen Problemen (optional, manuell in Phase 1, automatisiert ab Phase 2)

-   Format: Komprimiertes JPEG für Email-Attachment. Max 200KB pro Bild

13.5 Multi-Channel-Strategie

  --------------- --------------------------------- ----------------------------------------- ----------------------------------------------------------
  **Industrie**   **Primär**                        **Sekundär (nach 2 Nicht-Öffnungen)**     **Tertiär (High-Score-Leads ohne Response)**

  Anwälte         Email (formell, an Entscheider)   LinkedIn InMail (wenn Profil vorhanden)   Physischer Brief mit gedrucktem Analyse-PDF

  Ärzte           Email (an Praxis-Email)           Kontaktformular der Praxis-Website        Fax (ja, wirklich -- viele Praxen nutzen noch Fax)

  Steuerberater   Email (formell)                   LinkedIn InMail                           Physischer Brief

  Coaches         Email (persönlich, Du-Form)       Instagram DM (wenn \>500 Follower)        LinkedIn InMail

  Friseure        Email                             Instagram DM (wenn Account vorhanden)     Google Business Nachricht

  Handwerker      Email                             Kontaktformular                           Physischer Brief (höchste Response-Rate bei Handwerkern)
  --------------- --------------------------------- ----------------------------------------- ----------------------------------------------------------

13.6 KPIs & Monitoring

  --------------------------------------------------- ----------------- ---------------------------------------------------
  **Metrik**                                          **Zielwert**      **Alert wenn**

  Email Validation Rate (valid)                       \> 85%            \< 70%

  Personalisierungs-Vollständigkeit (0 Platzhalter)   100%              \< 100% ({{variable}} in finaler Email = Blocker)

  PDF-Generierungsdauer                               \< 15s            \> 30s

  Asset-Bundle-Größe (Email + Attachments)            \< 500KB          \> 1MB (Spam-Filter-Risiko)

  Outreach-Paket-Generierung (gesamt)                 \< 60s pro Lead   \> 120s

  Cost pro Outreach-Paket                             \< €0.10          \> €0.25
  --------------------------------------------------- ----------------- ---------------------------------------------------

+--------+----------------------------------------------------------------------------------------+
| **14** | **Email Outreach**                                                                     |
|        |                                                                                        |
|        | *Behavioral-Branching-Sequenz versenden + Multi-Channel-Fallback + Engagement tracken* |
+--------+----------------------------------------------------------------------------------------+

+-------------------------------------------------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------+--------------------------------------------------------------------------------------------------+
| **INPUT**                                                         | **OUTPUT**                                                                                                                                                          | **QUALITY GATE**                                                                                 |
|                                                                   |                                                                                                                                                                     |                                                                                                  |
| Outreach Package (Stufe 13) + Preview-Analytics (Stufe 12 Events) | Outreach Log: • emails_sent\[\]{timestamp, subject, status} • engagement_events\[\] • channel_switches\[\] • final_status (Converted/Nurture/Opted-Out/No-Response) | Alle Emails delivered (nicht bounced). Opt-outs sofort respektiert. Compliance-Checks bestanden. |
+-------------------------------------------------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------+--------------------------------------------------------------------------------------------------+

14.1 Sender-Infrastruktur

-   Tool: Instantly.ai -- Warmup, Sequenzen, Domain-Rotation, Analytics

-   Absender-Domains: 3-5 eigene Domains rotierend (z.B. webolution-partner.de, socialcooks-web.de, website-vorschau.de). Nie die Hauptdomain verwenden

-   Warmup: Jede neue Domain 2 Wochen warmupen bevor Outreach startet. 20 Emails/Tag → 50 → 100 (schrittweise hochfahren)

-   Absender-Name: Echter Vorname + Nachname (kein „Webolution Team"). Idealerweise eine reale Person bei SocialCooks

-   Signatur: Name, Rolle, Telefon (für Rückrufe), Website-Link. Kein Logo (erhöht Spam-Score)

-   Sending-Limit: Max 50 neue Outreach-Emails pro Domain pro Tag. Bei 5 Domains = 250 neue Leads/Tag max

-   Sending-Time: Branchenspezifisch optimiert (siehe unten)

  --------------- ------------------------------------- ---------------------------------------------------------------
  **Industrie**   **Beste Sendezeit**                   **Begründung**

  Anwälte         Di-Do, 08:00-09:30                    Früh morgens vor dem ersten Mandantentermin

  Ärzte           Di-Do, 12:00-13:00                    In der Mittagspause zwischen Sprechstunden

  Steuerberater   Mo-Mi, 08:30-10:00                    Morgens bevor der Mandantenstrom beginnt

  Coaches         Mo-Mi, 10:00-11:30                    Nach dem Morgenritual, vor dem ersten Client

  Friseure        Mo (Ruhetag!), 10:00-14:00            Montag ist Ruhetag für viele Salons -- einziger Tag für Admin

  Handwerker      Mo-Di, 06:30-07:30 oder 18:00-19:00   Vor der Baustelle oder nach Feierabend
  --------------- ------------------------------------- ---------------------------------------------------------------

14.2 5-Touch-Sequenz mit Behavioral Branching

+-----------+---------+----------------------------------+------------------------------------------------------------------------+-------------------------------------------------------------------+
| **Touch** | **Tag** | **Bedingung**                    | **Betreff**                                                            | **Kern-Inhalt**                                                   |
+-----------+---------+----------------------------------+------------------------------------------------------------------------+-------------------------------------------------------------------+
| **1**     | 0       | Lead qualifiziert + Preview live | „\[Firmenname\] -- so könnte Ihre neue Website aussehen"               | Persönliche Anrede (Name + Titel)                                 |
|           |         |                                  |                                                                        |                                                                   |
|           |         |                                  |                                                                        | 1 spezifische Schwachstelle mit Detail                            |
|           |         |                                  |                                                                        |                                                                   |
|           |         |                                  |                                                                        | Preview-Link prominent                                            |
|           |         |                                  |                                                                        |                                                                   |
|           |         |                                  |                                                                        | Analyse-PDF als Attachment                                        |
|           |         |                                  |                                                                        |                                                                   |
|           |         |                                  |                                                                        | P.S.: Calendly-Link für Gespräch                                  |
+-----------+---------+----------------------------------+------------------------------------------------------------------------+-------------------------------------------------------------------+
| **2a**    | 3       | Preview geöffnet                 | „Kurze Frage zu Ihrer Website-Vorschau"                                | „Uns ist aufgefallen, dass Sie sich die Vorschau angesehen haben" |
|           |         |                                  |                                                                        |                                                                   |
|           |         |                                  |                                                                        | 1 Konkurrenz-Vergleich als Argument                               |
|           |         |                                  |                                                                        |                                                                   |
|           |         |                                  |                                                                        | CTA: Direkt-Link zur Pricing-Page                                 |
|           |         |                                  |                                                                        |                                                                   |
|           |         |                                  |                                                                        | Calendly als Alternative                                          |
+-----------+---------+----------------------------------+------------------------------------------------------------------------+-------------------------------------------------------------------+
| **2b**    | 3       | Preview NICHT geöffnet           | Neuer Betreff: „\[Konkurrent\] in \[Stadt\] hat diese Woche relauncht" | Competitor-basiertes Argument                                     |
|           |         |                                  |                                                                        |                                                                   |
|           |         |                                  |                                                                        | Preview-Link nochmal (andere Formulierung)                        |
|           |         |                                  |                                                                        |                                                                   |
|           |         |                                  |                                                                        | Nur 2-3 Sätze -- kürzer als Email 1                               |
+-----------+---------+----------------------------------+------------------------------------------------------------------------+-------------------------------------------------------------------+
| **3**     | 7       | Immer (außer Opt-out)            | „Was die Top-\[Branche\] in \[Stadt\] online richtig machen"           | Branchen-Insight (Mini-Report)                                    |
|           |         |                                  |                                                                        |                                                                   |
|           |         |                                  |                                                                        | 2-3 Trends in der Branche                                         |
|           |         |                                  |                                                                        |                                                                   |
|           |         |                                  |                                                                        | Positionierung: „Wir helfen \[Branche\] dabei"                    |
|           |         |                                  |                                                                        |                                                                   |
|           |         |                                  |                                                                        | Preview-Link als Beweis                                           |
+-----------+---------+----------------------------------+------------------------------------------------------------------------+-------------------------------------------------------------------+
| **4**     | 14      | Preview besucht + kein Kauf      | „Ihre Website-Vorschau läuft bald ab"                                  | Scarcity: Countdown erwähnen                                      |
|           |         |                                  |                                                                        |                                                                   |
|           |         |                                  |                                                                        | Vereinfachtes Pricing direkt in der Email                         |
|           |         |                                  |                                                                        |                                                                   |
|           |         |                                  |                                                                        | Calendly für 15-Min-Gespräch                                      |
|           |         |                                  |                                                                        |                                                                   |
|           |         |                                  |                                                                        | Social Proof (wenn verfügbar)                                     |
+-----------+---------+----------------------------------+------------------------------------------------------------------------+-------------------------------------------------------------------+
| **5**     | 21      | Kein Kauf                        | „Kein Interesse? Kein Problem"                                         | Soft-Close: „Wir verstehen, der Zeitpunkt passt vielleicht nicht" |
|           |         |                                  |                                                                        |                                                                   |
|           |         |                                  |                                                                        | Kostenloser Analyse-PDF nochmal als Goodwill                      |
|           |         |                                  |                                                                        |                                                                   |
|           |         |                                  |                                                                        | Frage: „Gibt es etwas, das Sie davon abhält?" (Response-Trigger)  |
|           |         |                                  |                                                                        |                                                                   |
|           |         |                                  |                                                                        | Opt-out-Option klar sichtbar                                      |
+-----------+---------+----------------------------------+------------------------------------------------------------------------+-------------------------------------------------------------------+

14.3 Advanced Behavioral Triggers

  ------------------------------------------ ------------------------------------------ --------------------------------------------------------------------------------------------------------------------------------------------------
  **Trigger**                                **Erkennung**                              **Automatische Aktion**

  Lead besucht Preview 3x ohne Kauf          Analytics Event (Return Visit Count)       Email außerhalb der Sequenz: „Ich sehe, dass Sie sich die Vorschau mehrfach angesehen haben -- darf ich Sie kurz anrufen?" + Telefonnummer-Bitte

  Lead klickt CTA aber kauft nicht           Analytics: CTA-Click + kein Stripe Event   Abandoned-Cart-Email (1h nach Click): „Noch Fragen? Hier ein Kurzgespräch buchen" + Calendly

  Lead klickt Calendly-Link                  Analytics: Calendly Click Event            Sequenz pausieren. Sales-Team bekommt Alert. Kein weiteres Auto-Follow-up

  Lead replied auf Email (jede Antwort)      Instantly: Reply Detection                 Sequenz sofort pausieren. Alert an Sales-Team. Persönliche Antwort nötig

  Lead opted out                             Instantly: Unsubscribe Event               Sequenz sofort stoppen. Lead auf Suppression-Liste. Nie wieder kontaktieren

  Email bounced                              Instantly: Bounce Event                    Email-Adresse blacklisten. Sekundärkanal aus Stufe 13 aktivieren

  Lead öffnet aber klickt nicht (3 Emails)   Instantly: Open ohne Click                 Betreff-Optimierung: Kürzerer, direkterer Betreff. Oder: Kanal-Switch
  ------------------------------------------ ------------------------------------------ --------------------------------------------------------------------------------------------------------------------------------------------------

14.4 Compliance & Rechtliches

+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **DSGVO & UWG -- Nicht verhandelbar**                                                                                                                                                                                                                                                                      |
|                                                                                                                                                                                                                                                                                                            |
| B2B-Kaltakquise per Email ist in Deutschland eine Grauzone. UWG §7 erlaubt es nur unter engen Voraussetzungen (mutmaßliche Einwilligung + direkter Bezug zum Geschäftszweck). Wir bewegen uns am Rand des Legalen. Rechtliche Prüfung VOR Launch ist PFLICHT. Ein Abmahnrisiko bei Massenversand ist real. |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

-   DSGVO: Nur öffentlich zugängliche Geschäftskontaktdaten verwenden (Impressum). Keine privaten Emails. Keine Daten aus Social-Media-Profilen ohne rechtliche Basis

-   UWG §7: Mutmaßliche Einwilligung argumentieren: Wir kontaktieren Geschäftskunden mit einem Angebot das direkt ihrem Geschäftszweck dient (Website-Verbesserung für ihr Business). Kein B2C-Spam

-   Opt-out: Jede Email enthält einen 1-Click-Abmeldelink. Sofortige Suppression bei Abmeldung. Keine weitere Email, kein Sekundärkanal

-   Blacklist: Firmen die ablehnen werden für 12 Monate nicht mehr kontaktiert. Nach 12 Monaten: Nur Re-Engagement wenn Website sich signifikant verschlechtert hat

-   Beschwerde-Handling: Bei Spam-Report sofort permanent blacklisten. Domain des Beschwerdeführers auf Never-Contact-Liste

-   Dokumentation: Jeder Versand wird geloggt. Rechtsgrundlage pro Lead dokumentiert (welche öffentliche Quelle, welcher Geschäftszweck-Bezug)

-   Anwalt-Review: Vor Launch: Muster-Emails von einem auf UWG/DSGVO spezialisierten Anwalt prüfen lassen

14.5 KPIs & Monitoring

  ------------------------------------ --------------- -----------------------------------------------
  **Metrik**                           **Zielwert**    **Alert wenn**

  Email Delivery Rate                  \> 97%          \< 93% (Sender-Reputation-Problem)

  Open Rate (Email 1)                  \> 45%          \< 30%

  Click Rate (Preview-Link, Email 1)   \> 15%          \< 8%

  Preview Visit Rate (aus Email)       \> 25%          \< 15%

  Reply Rate (gesamt)                  \> 5%           \< 2%

  Opt-out Rate                         \< 2%           \> 5% (Content/Targeting-Problem)

  Bounce Rate                          \< 2%           \> 5% (Validation-Problem in Stufe 13)

  Spam-Report Rate                     \< 0.1%         \> 0.3% (SOFORT stoppen und reviewen)

  Channel-Switch-Rate                  \< 20%          \> 40% (Email-Qualität der Leads zu schlecht)
  ------------------------------------ --------------- -----------------------------------------------

+--------+---------------------------------------------------------------------------+
| **15** | **Conversion & Payment**                                                  |
|        |                                                                           |
|        | *Trust aufbauen + Objections beseitigen + Payment so einfach wie möglich* |
+--------+---------------------------------------------------------------------------+

+-------------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------+
| **INPUT**                                                               | **OUTPUT**                                                                                          | **QUALITY GATE**                                                                          |
|                                                                         |                                                                                                     |                                                                                           |
| Lead klickt CTA (aus Preview oder Email) → Personalisierte Pricing-Page | Zahlender Kunde: • stripe_customer_id • selected_package • payment_confirmed • onboarding_triggered | Payment erfolgreich. Bestätigungs-Email versendet. Onboarding-Flow automatisch gestartet. |
+-------------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------+

15.1 Das Trust-Problem

+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Die größte Hürde: Warum sollte jemand von einem Unbekannten kaufen?**                                                                                                                                                                                                                                                                                                                                                                                       |
|                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| Ein Geschäftsinhaber erhält eine Cold Email von einer Firma die er nie gehört hat. Er soll €990-3.490 ausgeben. Für eine Website die er nicht bestellt hat. Von Leuten die er nicht kennt. Das ist ein ENORMER Trust-Gap. Die Preview allein reicht nicht. Die Pricing-Page muss massiv Vertrauen aufbauen. Jedes Element muss eine Einwand-Antwort sein. Der Lead muss denken: „Die wissen was sie tun, das Risiko ist gering, der Wert ist offensichtlich." |
+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

15.2 Personalisierte Pricing-Page (webolution.de/go/{lead-id})

Die Pricing-Page ist personalisiert pro Lead. Sie zeigt SEINEN Firmennamen, SEINE Preview, SEINE Analyse-Daten. Das ist kein generischer Checkout.

Page-Struktur (top to bottom)

1.  Hero: „Ihre neue Website für \[Firmenname\]" + Screenshot der Preview (Desktop) + „Jetzt übernehmen" CTA

2.  Social Proof Section: Ab Phase 2: „Bereits X Unternehmen vertrauen Webolution" + 2-3 Testimonials (echte Kunden). Phase 1: „Technologie-Trust-Badges" (Next.js, SSL, DSGVO, Google-optimiert)

3.  Mini-Analyse: „Ihre aktuelle Website im Vergleich" -- 3 Zeilen mit altem Score vs. neuem Score (aus Quality Report)

4.  Pakete (3 Optionen): Klar, visuell, mit Recommended-Badge auf dem mittleren Paket. Alltagssprache, kein Tech-Jargon

5.  Objection-Handling FAQ: 8-10 häufige Fragen direkt beantwortet (siehe 15.3)

6.  Garantie-Section: „30 Tage Geld-zurück-Garantie. Wenn Sie nicht zufrieden sind, erstatten wir den vollen Betrag."

7.  Sales-Gespräch CTA: „Lieber erst reden? 15-Minuten-Gespräch buchen" + Calendly-Widget

8.  Footer: Webolution-Impressum, Kontaktdaten, Telefon für direkte Rückfragen

15.3 Objection-Handling-Matrix

Jede Frage die ein Lead haben könnte, muss auf der Pricing-Page beantwortet werden BEVOR er sie stellt:

  ------------------------------------------------------------- -------------------------------------------------------------------------------------------------------------------------------------------------------------- -----------------------------------------
  **Einwand**                                                   **Antwort auf der Pricing-Page**                                                                                                                               **Vertrauens-Element**

  „Wer seid ihr überhaupt?"                                     About-Section: SocialCooks-Story, Gründer-Fotos, Standort, Erfahrung. „Wir sind ein Team aus Webdesignern und Entwicklern in \[Stadt\]"                        Echte Fotos, echte Namen, echte Adresse

  „Was passiert mit meiner alten Website?"                      „Ihre alte Website bleibt online bis die neue fertig und von Ihnen freigegeben ist. Es gibt keinen einzigen Tag Ausfall."                                      Kein Risiko

  „Funktioniert meine Email noch?"                              „Ja, Ihre Emails laufen unverändert weiter. Wir ändern nur, wohin Ihre Domain zeigt -- nicht Ihre Email-Einstellungen."                                        Technische Garantie

  „Kann ich später Änderungen machen?"                          „Ja, per Email oder über Ihr persönliches Dashboard. Änderungswünsche werden innerhalb von 48h umgesetzt."                                                     Flexibilität

  „Brauche ich technisches Wissen?"                             „Nein. Wir richten alles für Sie ein: Domain, Hosting, Formulare, Analytics. Sie müssen nichts tun."                                                           Managed Service

  „Wie lange dauert es?"                                        „In der Regel ist Ihre neue Website innerhalb von 48 Stunden live. Bei komplexen Domain-Setups max. 5 Werktage."                                               Schnelligkeit

  „Was wenn mir die Website nicht gefällt?"                     „30 Tage Geld-zurück-Garantie. Wenn Sie nicht zufrieden sind, erstatten wir den vollen Betrag. Ohne Diskussion."                                               Risikolos

  „Ist das nicht zu günstig für eine professionelle Website?"   „Wir nutzen moderne Technologie um den Prozess zu automatisieren. Sie bekommen Agentur-Qualität zum Bruchteil des Preises."                                    Wert-Proposition

  „Was passiert nach den 3/12 Monaten Hosting?"                 „Sie können Ihr Hosting zum gleichen Monatspreis verlängern. Oder die Website jederzeit zu einem anderen Hoster migrieren -- sie gehört Ihnen."                Keine Lock-in-Angst

  „Ist der Content von einer KI?"                               „Wir verwenden KI-Werkzeuge, aber jeder Text basiert auf echten Informationen von Ihrer bestehenden Website und öffentlichen Quellen. Nichts wird erfunden."   Transparenz
  ------------------------------------------------------------- -------------------------------------------------------------------------------------------------------------------------------------------------------------- -----------------------------------------

15.4 Checkout-Flow

1.  Lead wählt Paket: Klick auf „Paket wählen". Paket wird visuell hervorgehoben

2.  Stripe Checkout: Overlay oder neue Seite. Zahlungsmethoden: Kreditkarte, SEPA-Lastschrift, PayPal. KEINE Registrierung nötig. Nur Name + Email + Zahlungsdaten

3.  Bei Subscriptions (Professional/Premium): Stripe zeigt einmalige Zahlung + monatliche Folgekosten transparent an

4.  Payment Success: Redirect zu „Danke!-Seite" mit klaren nächsten Schritten. Confetti-Animation (ja, wirklich -- der Moment soll sich gut anfühlen)

5.  Bestätigungs-Email (sofort): „Vielen Dank, \[Name\]! Ihre neue Website wird jetzt eingerichtet." + Nächste Schritte (1-2-3) + Onboarding-Link + Support-Kontakt

6.  Onboarding auto-triggered: Stripe Webhook → Onboarding-Flow startet (Stufe 16)

Alternative: Sales-Gespräch (für zögerliche Leads)

-   Calendly-Widget auf der Pricing-Page: „15-Minuten-Gespräch buchen"

-   Buchbare Slots: Mo-Fr 9-17 Uhr. 15-Min-Slots

-   Vor dem Call: CRM zeigt dem Sales-Mitarbeiter: Lead-Score, Quality Report, Competitor Report, Preview-URL, bisherige Email-Interaktionen

-   Im Call: Kein Hard-Sell. Fragen beantworten, Preview gemeinsam durchgehen, Einwände klären

-   Nach dem Call: Personalisierter Payment-Link per Email mit ggf. angepasstem Angebot

-   Tracking: Call-Outcome loggen: Converted, Follow-up nötig, Nicht interessiert, Zu teuer

15.5 KPIs & Monitoring

  ---------------------------------------------- ----------------- ---------------------------------------------------------------
  **Metrik**                                     **Zielwert**      **Alert wenn**

  Pricing-Page-Besuche (aus Email + Preview)     Tracking          Niedrig = Outreach-Problem (Stufe 14)

  Conversion Rate (Pricing-Visit → Payment)      \> 6%             \< 3%

  Avg. Deal Size                                 €1.500-1.800      \< €1.000 (nur Starter-Pakete) oder \> €2.500 (unglaubwürdig)

  Calendly-Booking Rate (von Pricing-Visitors)   \> 8%             \< 3%

  Calendly → Conversion Rate                     \> 30%            \< 15%

  Checkout Abandonment Rate                      \< 40%            \> 60% (Checkout-Problem)

  Payment Failure Rate                           \< 3%             \> 8% (Zahlungsmethoden-Problem)

  Geld-zurück-Inanspruchnahme                    \< 5%             \> 15% (Qualitätsproblem)
  ---------------------------------------------- ----------------- ---------------------------------------------------------------

+--------+------------------------------------------------------------------+
| **16** | **Managed Onboarding & Go-Live**                                 |
|        |                                                                  |
|        | *Domain-Switch + Email-Schutz + End-to-End-Test + Rollback-Plan* |
+--------+------------------------------------------------------------------+

+----------------------------------------------------------------------------------------+--------------------------------------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------+
| **INPUT**                                                                              | **OUTPUT**                                                                                                                           | **QUALITY GATE**                                                                                                |
|                                                                                        |                                                                                                                                      |                                                                                                                 |
| Zahlender Kunde: • Paket-Auswahl • Preview-Website • Kunden-Email • Stripe-Customer-ID | Live Website: • \[kundendomain.de\] zeigt neue Website • SSL aktiv • Email funktioniert • Formulare liefern • Dashboard-Zugang aktiv | Domain live + SSL aktiv. Email-Zustellung verifiziert. Formular-Test bestanden. Kunde hat Bestätigung erhalten. |
+----------------------------------------------------------------------------------------+--------------------------------------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------+

16.1 Das kritischste Moment der gesamten Pipeline

+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Ein einziger Fehler hier = Refund + negative Bewertung + verbrannte Bridge**                                                                                                                                                                                                                                                                                  |
|                                                                                                                                                                                                                                                                                                                                                                 |
| Wenn nach dem Domain-Switch die Email des Kunden nicht mehr funktioniert, sein Kontaktformular keine Anfragen mehr liefert oder seine Website einen Tag offline ist -- haben wir einen feindlichen Kunden. DNS-Änderungen sind fragil. MX-Records dürfen NICHT angefasst werden. Das Onboarding muss fehlerfrei sein. Deshalb: Rollback-Plan für jeden Schritt. |
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

16.2 Onboarding-Flow

Schritt 1: Willkommens-Email (sofort nach Payment)

-   Inhalt: „Vielen Dank! Ihre neue Website wird jetzt eingerichtet. Hier sind die nächsten 3 Schritte:"

-   Link zum Onboarding-Formular (maximal simpel)

-   Erwartungsmanagement: „In der Regel ist Ihre Website innerhalb von 48h live"

-   Support-Kontakt für Fragen (Email + Telefon)

Schritt 2: Onboarding-Formular (3 Fragen)

  ----------------------------------------------------------------- ---------------------------------------------------------------------------------------- ----------------------------------- -------------------------------------------------------------------
  **Frage**                                                         **UI-Element**                                                                           **Warum nötig**                     **Fallback wenn nicht beantwortet**

  „Bei welchem Anbieter ist Ihre Domain registriert?"               Dropdown: Strato, IONOS, united-domains, GoDaddy, Hetzner, All-Inkl, Andere (Freitext)   Bestimmt DNS-Änderungsmethode       WHOIS-Lookup als Alternative

  „Können Sie uns den Zugang zu Ihrem Domain-Provider mitteilen?"   Sicheres Formular (verschlüsselt) ODER Button „Ich brauche Hilfe dabei"                  Für DNS-Änderungen                  „Hilfe"-Button: Step-by-Step-Anleitung mit Screenshots generieren

  „Welche Email-Adressen nutzen Sie geschäftlich?"                  Textfeld für 1-5 Email-Adressen                                                          Diese dürfen NICHT gestört werden   MX-Record-Analyse als Fallback -- vorhandene MX niemals ändern
  ----------------------------------------------------------------- ---------------------------------------------------------------------------------------- ----------------------------------- -------------------------------------------------------------------

Schritt 3: Pre-Switch-Checks

1.  MX-Record-Snapshot: Aktuelle MX-Records der Domain speichern. Dies ist der Referenzwert. Nach dem Switch müssen diese IDENTISCH sein

2.  A-Record-Snapshot: Aktuelle A/CNAME-Records speichern. Für Rollback

3.  Email-Test: Test-Email an eine der geschäftlichen Adressen senden. Bestätigung dass sie ankommt → Baseline für Post-Switch-Verifikation

4.  Website-Screenshot: Aktuelle Website als Screenshot speichern. Für Rollback-Verifikation

Schritt 4: DNS-Switch

  ------------------ ---------------------------------------------------------- ---------------------- ---------------------
  **Provider**       **Methode**                                                **Automatisierbar?**   **Dauer**

  Strato             API (begrenzt) oder generierte Anleitung mit Screenshots   Teilweise              15-60 Min

  IONOS              IONOS API (Cloud DNS)                                      Ja                     5-15 Min

  united-domains     Generierte Anleitung + Support-Link                        Nein                   30-120 Min

  GoDaddy            GoDaddy API                                                Ja                     5-15 Min

  Hetzner            Hetzner DNS API                                            Ja                     5-15 Min

  All-Inkl           Generierte Anleitung                                       Nein                   30-120 Min

  Andere/Unbekannt   Generische Anleitung + persönlicher Support                Nein                   60-240 Min
  ------------------ ---------------------------------------------------------- ---------------------- ---------------------

-   DNS-Änderung: NUR A-Record und/oder CNAME auf Vercel zeigen. MX-Records NIEMALS ANFASSEN

-   TTL: Wenn möglich, TTL vor dem Switch auf 300s senken (5 Minuten). Beschleunigt die Propagation

-   Propagation: DNS-Änderungen brauchen 5 Minuten bis 48 Stunden. Typisch: 15-60 Minuten. Monitoring über DNS-Checker

Schritt 5: Post-Switch-Verifikation

1.  DNS-Propagation-Check: Alle 5 Minuten prüfen ob Domain auf neue IP zeigt. Von mehreren Standorten (whatsmydns.net API)

2.  Website-Live-Check: HTTP Request an kundendomain.de → Neue Website wird gezeigt

3.  SSL-Check: HTTPS funktioniert. Vercel provisioniert automatisch Let\'s Encrypt Zertifikat. Kann 5-30 Minuten dauern

4.  MX-Record-Verifikation: MX-Records sind IDENTISCH zum Pre-Switch-Snapshot. Wenn nicht: SOFORT ROLLBACK

5.  Email-Test: Test-Email an die geschäftliche Adresse. Kommt sie an? Wenn nein: SOFORT ROLLBACK

6.  Formular-Test: Test-Submission über das Kontaktformular. Kommt die Email beim Kunden an?

7.  Telefon-Link-Test: tel: Link korrekt?

8.  Maps-Test: Google Maps zeigt richtige Adresse?

16.3 Rollback-Plan

+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Rollback muss innerhalb von 5 Minuten möglich sein**                                                                                                                                                                                   |
|                                                                                                                                                                                                                                          |
| Wenn nach dem DNS-Switch etwas nicht funktioniert (Email tot, Website kaputt, SSL-Problem), muss SOFORT zurückgerollt werden. Jede Minute Downtime ist inakzeptabel. Der Rollback setzt die DNS-Records auf die Pre-Switch-Werte zurück. |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

  --------------------------------------- --------------------------- ----------------------------------------------------------------- ----------------------------------------
  **Problem**                             **Erkennung**               **Rollback-Aktion**                                               **Max. Downtime**

  Email funktioniert nicht nach Switch    Email-Test schlägt fehl     DNS-Records auf Snapshot zurücksetzen. MX-Records verifizieren    \< 5 Min (wenn TTL kurz)

  Website zeigt Fehler                    HTTP 5xx oder leere Seite   CNAME/A-Record auf alten Wert. Vercel-Deployment prüfen           \< 5 Min

  SSL nicht provisioniert nach 30 Min     HTTPS-Check                 Cloudflare als SSL-Proxy davorschalten. Oder: warten (max 4h)     Website erreichbar via HTTP

  Formular sendet nicht                   Test-Submission-Failure     Formspree-Config prüfen. Kein DNS-Rollback nötig                  0 (Website läuft, nur Formular defekt)

  Kunde meldet: „Meine Website ist weg"   Kunden-Support-Ticket       DNS-Propagation prüfen (lokaler DNS-Cache). Wenn real: Rollback   Variabel
  --------------------------------------- --------------------------- ----------------------------------------------------------------- ----------------------------------------

16.4 Kunden-Dashboard

Maximal einfach. Die Zielgruppe ist nicht technisch. Jedes Feature muss selbsterklärend sein.

  -------------------- --------------------------------------------------------------------------------- ----------------------------------------------
  **Feature**          **Was der Kunde sieht**                                                           **Technisch dahinter**

  Website-Status       Großer grüner Punkt + „Ihre Website ist live"                                     Uptime-Monitor (HTTP-Check alle 5 Min)

  Anfragen             Liste: Datum \| Name \| Nachricht \| Status (Neu/Gelesen)                         Formspree Submissions + Inbox-API

  Besucher-Statistik   Einfaches Liniendiagramm: „Besucher pro Woche" + „Top-Seiten"                     Plausible Analytics API

  Änderungswünsche     Textfeld + „Absenden"-Button. „Was möchten Sie ändern?"                           Ticket-System (Email an Support + Ticket-DB)

  Rechnungen & Abo     Stripe Customer Portal: Zahlungshistorie, Abo verwalten, Zahlungsmethode ändern   Stripe Customer Portal Embed

  Support              Telefonnummer + Email + „Wir antworten innerhalb von 24h"                         Standard-Support-Kanal
  -------------------- --------------------------------------------------------------------------------- ----------------------------------------------

16.5 KPIs & Monitoring

  ----------------------------------------------- --------------------------------------------- -------------------------------------
  **Metrik**                                      **Zielwert**                                  **Alert wenn**

  Onboarding-Formular-Completion-Rate             \> 85%                                        \< 60% (Formular zu kompliziert)

  Time-to-Go-Live (Payment → Website live)        \< 48h (API-Provider) / \< 5 Tage (manuell)   \> 7 Tage

  DNS-Switch-Erfolgsrate (1. Versuch)             \> 80%                                        \< 60%

  Email-Funktioniert-nach-Switch-Rate             \> 99%                                        \< 95% (MX-Problem!)

  Formular-Funktioniert-Rate                      \> 99%                                        \< 95%

  Rollback-Rate                                   \< 5%                                         \> 15% (systematisches DNS-Problem)

  Kunden-Zufriedenheit (Post-Onboarding Survey)   \> 4.5/5                                      \< 4.0

  Support-Tickets pro Onboarding                  \< 2                                          \> 5 (Prozess zu komplex)
  ----------------------------------------------- --------------------------------------------- -------------------------------------

+--------+-------------------------------------------------------------------+
| **17** | **Post-Launch Monitoring & Customer Success**                     |
|        |                                                                   |
|        | *Fortlaufende Qualitätssicherung + Upsell + Retention + Referral* |
+--------+-------------------------------------------------------------------+

+---------------------------------------------------------------------+-------------------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------+
| **INPUT**                                                           | **OUTPUT**                                                                                                        | **QUALITY GATE**                                                            |
|                                                                     |                                                                                                                   |                                                                             |
| Live Kunden-Website + Stripe Subscription + Kunden-Dashboard-Zugang | Fortlaufend: • Uptime-Alerts • Performance-Reports • Upsell-Empfehlungen • Retention-Metriken • Referral-Tracking | Kein einzelner Gate. Fortlaufende Qualitätssicherung. Alerts bei Anomalien. |
+---------------------------------------------------------------------+-------------------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------+

17.1 Automatisches Monitoring

  ------------------------------ ---------------- ------------------------------------ ---------------------------
  **Was**                        **Frequenz**     **Tool**                             **Alert wenn**

  Uptime (Website erreichbar?)   Alle 5 Minuten   Uptime-Monitor (BetterUptime o.ä.)   Down \> 5 Minuten

  SSL-Zertifikat                 Täglich          SSL-Expiry-Check                     \< 30 Tage bis Ablauf

  Domain-Ablauf                  Wöchentlich      WHOIS-Check                          \< 60 Tage bis Ablauf

  Performance (Lighthouse)       Wöchentlich      Lighthouse CI                        Score \< 80 (Regression)

  Formular-Zustellung            Monatlich        Test-Submission + Email-Check        Test-Email kommt nicht an

  Broken Links                   Monatlich        Crawler                              \> 0 broken Links

  SSL/Mixed Content              Monatlich        HTTPS-Scan                           Mixed Content Warnings
  ------------------------------ ---------------- ------------------------------------ ---------------------------

17.2 Customer Success Touchpoints

  --------------- ------------------------------------------------------ ------------------------------------ --------------------------------------------------------------------------------- -----------------------------------
  **Zeitpunkt**   **Touchpoint**                                         **Kanal**                            **Ziel**                                                                          **Automatisiert?**

  Tag 1           Willkommen + Dashboard-Erklärung                       Email + kurzes Erklärvideo           Erste Schritte im Dashboard                                                       Ja

  Tag 7           Check-in: „Wie läuft es?"                              Email                                Frühes Feedback, Probleme erkennen                                                Ja

  Tag 30          Performance-Report Monat 1                             Email mit PDF-Report                 Wert demonstrieren: Besucher, Anfragen, Google-Trend                              Ja

  Tag 60          Feature-Vorschlag (wenn Daten zeigen: Feature fehlt)   Email                                Upsell-Vorbereitung: „Ihre Besucher suchen nach X -- möchten Sie Y hinzufügen?"   Ja (Trigger-basiert)

  Tag 90          Quartals-Review                                        Email mit Report + Telefon-Angebot   Retention + Upsell. „Ab Professional bekommen Sie monatliche SEO-Updates"         Ja (Report) + Optional persönlich

  Tag 335         Hosting-Verlängerung                                   Email                                30 Tage vor Ablauf: „Ihr Hosting läuft ab -- jetzt verlängern"                    Ja

  Laufend         Support-Anfragen beantworten                           Email / Dashboard                    Kundenzufriedenheit                                                               Nein (persönlich)
  --------------- ------------------------------------------------------ ------------------------------------ --------------------------------------------------------------------------------- -----------------------------------

17.3 Upsell-Engine

Automatische Erkennung von Upsell-Opportunities basierend auf Kunden-Daten:

  ---------------------------------- ----------------------------------------------------- ------------------------------------------------------------------------------------------------------ -------------------------------
  **Trigger**                        **Erkennung**                                         **Angebot**                                                                                            **Timing**

  Hoher Traffic, kein Buchungstool   Plausible: \>200 Visits/Monat + kein Booking-Widget   „Sie bekommen viele Besucher -- mit einem Online-Buchungstool könnten Sie X% mehr Anfragen bekommen"   Nach 60 Tagen

  Viele Kontaktanfragen              Formspree: \>20 Submissions/Monat                     „Sie erhalten viele Anfragen -- möchten Sie ein automatisches Antwortsystem?"                          Nach 30 Tagen

  Starter-Kunde zufrieden            NPS \> 4 + keine Support-Issues                       „Upgrade auf Professional: Fortlaufende SEO + Content-Updates"                                         Nach 90 Tagen

  Google-Ranking steigt              Google Search Console (wenn integriert)               „Ihre Website klettert bei Google! Mit SEO-Betreuung könnten Sie auf Seite 1"                          Bei messbarem Ranking-Anstieg

  Neue Google-Bewertungen            Google Places API: Review-Count Check                 „Sie haben neue Bewertungen! Sollen wir die auf Ihrer Website einbinden?"                              Bei \>5 neuen Reviews

  Content wird alt                   Blog-Posts \> 6 Monate ohne Update                    „Ihr letzter Blogpost ist von \[Monat\]. Frischer Content hilft beim Google-Ranking"                   Nach 6 Monaten
  ---------------------------------- ----------------------------------------------------- ------------------------------------------------------------------------------------------------------ -------------------------------

17.4 Retention & Churn-Prevention

  -------------------------------------------------------- -------------------------------- ----------------------------------------------------------------------------------------------
  **Churn-Signal**                                         **Erkennung**                    **Präventive Aktion**

  Keine Dashboard-Logins seit 30 Tagen                     Login-Tracking                   Email: „Haben Sie gewusst, dass Sie X neue Anfragen haben?"

  Support-Ticket unzufrieden / negativ                     Sentiment-Analyse auf Tickets    Persönlicher Anruf vom Customer Success Manager

  Payment fehlgeschlagen (Kreditkarte abgelaufen)          Stripe Webhook: payment_failed   Automatische Email: „Ihre Zahlung konnte nicht verarbeitet werden" + Stripe Update-Link

  Hosting-Verlängerung nicht gebucht (7 Tage vor Ablauf)   Stripe: Keine Renewal            Email + Telefon: „Ihr Hosting läuft in 7 Tagen ab. Sollen wir verlängern?"

  Kunde fragt nach Daten-Export / Migration                Support-Ticket-Keywords          Persönliches Gespräch: Was läuft falsch? Kann man es lösen? Migrationsunterstützung anbieten
  -------------------------------------------------------- -------------------------------- ----------------------------------------------------------------------------------------------

17.5 Referral-Programm

-   Mechanik: „Empfehlen Sie Webolution weiter -- €200 Gutschrift für jede erfolgreiche Empfehlung"

-   Tracking: Einzigartiger Referral-Link pro Kunde. Cookie-basiertes Tracking (30 Tage Attribution)

-   Auszahlung: €200 Gutschrift auf nächste Rechnung (oder PayPal-Auszahlung)

-   Kommunikation: Referral-CTA im Dashboard + in der 90-Tage-Email + als Dankeskärtchen nach Go-Live

-   Upsell-Variante: „Empfehlen Sie einen Kollegen und Sie bekommen BEIDE 15% Rabatt" (funktioniert besonders gut in Branchenverbänden)

17.6 Re-Engagement für nicht-konvertierte Leads

Leads die in Stufe 14 nicht konvertiert haben, werden nicht vergessen:

  ----------------------------------------- -------------------------------------------------------- ----------------------------------------------------------------------------------------------------------------
  **Trigger**                               **Timing**                                               **Aktion**

  Re-Score (Website immer noch schlecht?)   6 Monate nach letztem Kontakt                            Quick-Re-Analyse (Stufe 5 Lite). Wenn Score gestiegen: Nicht kontaktieren. Wenn gleich/schlechter: Re-Approach

  Saisonaler Trigger                        Januar + September                                       „Neues Jahr, neue Website" / „Herbst-Relaunch für die Saison"

  Competitor-Trigger                        Wenn ein Konkurrent des Leads Webolution-Kunde wird      „Wir haben gerade \[Branche\] in \[Stadt\] geholfen -- möchten Sie nachziehen?"

  Business-Growth-Trigger                   Google Reviews: \>10 neue Reviews seit letztem Kontakt   „Ihr Geschäft wächst -- Ihre Website hinterher. Wir haben eine aktualisierte Vorschau."

  Nurture-Newsletter                        Monatlich                                                Branchenspezifischer Newsletter mit Online-Präsenz-Tipps. Kein Sales-Push, nur Mehrwert. Soft CTA am Ende
  ----------------------------------------- -------------------------------------------------------- ----------------------------------------------------------------------------------------------------------------

17.7 KPIs & Monitoring

  ----------------------------------------------------- ----------------- ----------------------------
  **Metrik**                                            **Zielwert**      **Alert wenn**

  Uptime (alle Kunden-Websites)                         \> 99.9%          \< 99.5%

  30-Day CSAT (Post-Onboarding)                         \> 4.5 / 5        \< 4.0

  Monthly Churn Rate                                    \< 3%             \> 5%

  Upsell-Conversion-Rate                                \> 10%            \< 5%

  Avg. Customer Lifetime Value (LTV)                    \> €2.500         \< €1.500

  Referral-Rate (% Kunden die empfehlen)                \> 15%            \< 5%

  Re-Engagement-Conversion (nicht-konvertierte Leads)   \> 3%             \< 1%

  Support-Ticket-Resolution-Time                        \< 24h            \> 48h

  NPS (Net Promoter Score)                              \> 50             \< 30
  ----------------------------------------------------- ----------------- ----------------------------

Webolution Deep Spec: Komplett

Mit diesem Dokument ist die 17-Step-Pipeline vollständig und in epischer Tiefe spezifiziert. Alle 4 Dokumente zusammen bilden das komplette Product Blueprint:

  ----------------- ------------------------------------------------------------ ------------------------------------
  **Dokument**      **Steps**                                                    **Fokus**

  Deep Spec 1-4     Scraping → Validation → Enrichment → Crawl                   Daten sammeln und aufbereiten

  Deep Spec 5-8     Quality Review → Competitor → Scoring → Rebuild Plan         Analysieren und strategisch planen

  Deep Spec 9-12    Content Polish → Build → QA → Preview Deploy                 Bauen und qualitätssichern

  Deep Spec 13-17   Outreach Prep → Email → Payment → Onboarding → Post-Launch   Verkaufen und langfristig betreuen
  ----------------- ------------------------------------------------------------ ------------------------------------

Nächste Schritte

1.  Industry Profiles: Die ersten 2 Pilot-Industrien (Coaches + Friseure) komplett als JSON-Config ausarbeiten

2.  MVP-Scope: Welche der 17 Stufen sind ab Tag 1 automatisiert, welche manuell? Realistische Phase-1-Planung

3.  Tech-Architektur: Service-Design, Datenbank-Schema, API-Verträge zwischen Services

4.  Template-Design: Erste 2 Industry-Templates designen (Figma → Code)

5.  Rechtliche Prüfung: UWG-Konformität für B2B-Outreach mit spezialisiertem Anwalt klären

6.  Pilot-Stadt + Industrien festlegen und ersten Testlauf planen

7.  Cost-Modell: Detaillierte Kalkulation: Was kostet ein Lead von Scraping bis Conversion?

8.  Team: Mindest-Team für MVP definieren: Backend, Frontend, Growth, Sales/Support

*Webolution Deep Spec -- Steps 13-17 (Final) -- SocialCooks*
