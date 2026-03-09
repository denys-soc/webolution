**WEBOLUTION**

Operations Command Center

**Du entscheidest. Die Maschine führt aus.**

*Approval Gates • Action-Required Inbox • Content-Editing • Full Pipeline Control*

**SocialCooks × Webolution**

März 2026 -- v2.0 (Complete Rewrite)

Die neue Philosophie: Approval Gates

+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Der fundamentale Paradigmenwechsel**                                                                                                                                                                                                                                                                                                               |
|                                                                                                                                                                                                                                                                                                                                                      |
| Version 1 war ein Dashboard zum Anschauen mit optionalem Eingreifen. Version 2 ist eine Steuerzentrale zum Entscheiden. Die Pipeline läuft NICHT automatisch durch. Sie STOPPT an 5 kritischen Punkten und WARTET auf deine Entscheidung. Automation führt deine Entscheidungen aus -- sie trifft sie nicht. Du bist der Pilot, nicht der Passagier. |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

Die 5 Approval Gates

  ------------ ---------------------------- ------------------------------------------------------------------------------- -------------------------------------------------------------------------------
  **Gate**     **Pipeline stoppt nach**     **Du entscheidest**                                                             **Dann passiert**

  **GATE 1**   Step 2: Validation           Aus 200 gescrapten Leads: Welche 80 enrichen wir?                               Gewählte Leads → Enrichment. Rest → Archive

  **GATE 2**   Step 3: Enrichment           Aus 60 enriched Leads: Für welche 30 crawlen und analysieren wir die Website?   Gewählte Leads → Crawl + Review + Score. Rest → Nurture Pool

  **GATE 3**   Step 7: Scoring              Aus 25 scored Leads: Für welche 15 bauen wir eine Website?                      Gewählte Leads → Rebuild Plan + Polish + Build. Rest → Nurture oder Archive

  **GATE 4**   Step 12: Preview Deploy      Jede generierte Website einzeln prüfen: Approve / Reject / Edit?                Approved → Outreach Prep. Rejected → Rebuild mit Notizen

  **GATE 5**   Step 13: Outreach Prepared   Personalisierte Emails + Preview-URL reviewen: Senden / Edit / Skip?            Approved → Email-Sequenz startet. Edit → Texte anpassen. Skip → kein Outreach
  ------------ ---------------------------- ------------------------------------------------------------------------------- -------------------------------------------------------------------------------

+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Konfigurierbar: Von Full-Manual bis Full-Auto**                                                                                                                                                                                                                                                                                                                                                                                |
|                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| Im MVP stehen alle 5 Gates auf MANUAL -- du entscheidest alles. Später kannst du Gates schrittweise auf AUTO schalten: „Gate 1 automatisch wenn Validation Confidence \>0.8", „Gate 4 automatisch wenn QA Score \>85 und Visual QA \>80". Jedes Gate hat einen Toggle: MANUAL (Pipeline stoppt und wartet) oder AUTO (Pipeline läuft wenn Confidence-Threshold erreicht). Du kannst jederzeit zwischen Manual und Auto wechseln. |
+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

Modul 1: Action Required (Homepage)

+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Die wichtigste Seite im gesamten Backend**                                                                                                                                                                              |
|                                                                                                                                                                                                                           |
| Wenn du morgens das Backend öffnest, siehst du EINE Frage: Was braucht meine Aufmerksamkeit? Alles andere -- Statistiken, Charts, Tabellen -- ist sekundär. Diese Seite ist dein Posteingang für Pipeline-Entscheidungen. |
+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

1.1 Entscheidungs-Inbox (Kern der Seite)

  ------------------------------------ --------------- ------------------------------------------------------------------------ --------------------------------------------------
  **Entscheidungs-Typ**                **Badge**       **Beispiel-Eintrag**                                                     **Aktion**

  **Gate 1: Leads zum Enrichen**       🟢 47 Leads     „47 neue Coaches in München gescraped -- welche enrichen?"               Klick → Batch-Approve-Screen

  **Gate 2: Leads zum Analysieren**    🟡 23 Leads     „23 enriched Leads mit Health \>50 -- für welche Website analysieren?"   Klick → Batch-Approve-Screen

  **Gate 3: Leads zum Rebuilden**      🟠 12 Leads     „12 Leads mit Score \>65 -- für welche Website bauen?"                   Klick → Batch-Approve-Screen mit Analyse-Details

  **Gate 4: Websites zum Reviewen**    🟣 5 Websites   „5 generierte Websites warten auf Review"                                Klick → Website-Review mit Split-View

  **Gate 5: Outreach zum Freigeben**   🔴 3 Pakete     „3 Outreach-Pakete warten auf Freigabe"                                  Klick → Email-Preview mit Inbox-Darstellung

  **QA Failed**                        ⚠️ 2 Leads      „2 Websites haben QA nicht bestanden"                                    Klick → QA-Report mit Repair-Optionen

  **Kunden-Anfragen**                  📩 4 Tickets    „4 Änderungswünsche von Kunden"                                          Klick → Ticket-Übersicht

  **System-Alerts**                    🚨 1 Alert      „Instantly-Webhook seit 2h inaktiv"                                      Klick → System Health
  ------------------------------------ --------------- ------------------------------------------------------------------------ --------------------------------------------------

1.2 Quick-Stats (Sidebar oder Top-Bar)

-   Leads in Pipeline: \[Zahl\] (davon \[X\] warten auf dich)

-   Previews Live: \[Zahl\]

-   Outreach Aktiv: \[Zahl\] Sequenzen

-   Conversions diesen Monat: \[Zahl\] (€\[Betrag\])

-   API-Kosten heute: €\[Betrag\]

1.3 Activity Feed (kompakt, unter der Inbox)

Live-Feed der letzten 20 Aktionen. Jeder Eintrag klickbar zum Lead Detail:

-   „14:23 -- Lead „Coach Sandra K.": Preview deployed → wartet auf Review"

-   „14:19 -- Email geöffnet: Kanzlei Müller (Touch 1)"

-   „14:15 -- Payment: Salon Bella -- Professional Paket -- €1.990"

-   „14:10 -- QA FAILED: Friseur am Markt -- Broken Contact Form"

Modul 2: Gate Approval Screens

Jedes der 5 Gates hat einen eigenen Approval-Screen der für schnelle Entscheidungen optimiert ist.

2.1 Gate 1: „Welche Leads enrichen?"

Nach dem Scraping einer Kampagne siehst du alle validierten Leads in einer Tabelle:

  ----------------------- ----------------------------------------------------------- ------------------------------------------------------
  **Spalte**              **Daten**                                                   **Zweck**

  ☐ Checkbox              Zum Auswählen                                               Bulk-Approve

  Firmenname              Name + Website-Link (klickbar, öffnet in neuem Tab)         Schnell-Check: Existiert die Firma? Website live?

  Website-Thumbnail       Mini-Screenshot der Website (aus Validation, 200px breit)   Auf einen Blick: Sieht die Website alt/schlecht aus?

  Google Rating           Sterne + Anzahl Reviews                                     Business Health Quick-Check

  URL-Status              HTTP 200 = Grün, Redirect = Gelb, SSL-Error = Orange        Technischer Quick-Check

  Validation Confidence   Score als Farbbalken                                        Datenqualität

  Quelle                  Google Maps / Branchenportal / etc.                         Zuverlässigkeit der Daten
  ----------------------- ----------------------------------------------------------- ------------------------------------------------------

-   Quick-Aktionen: „Alle auswählen", „Alle mit Rating \>4.0 auswählen", „Alle mit Confidence \>0.8 auswählen"

-   Approve Selected → Gewählte Leads gehen in Enrichment

-   Reject Selected → Leads werden archiviert mit Grund „Manually rejected at Gate 1"

-   Reject ist umkehrbar: Archivierte Leads können später reaktiviert werden

2.2 Gate 2: „Welche Leads analysieren?"

Zeigt enriched Leads mit Business-Health-Daten:

  ------------------------- --------------------------------------------------- ---------------------------
  **Spalte**                **Daten**                                           **Zweck**

  ☐ Checkbox                Bulk-Select                                         

  Firmenname + URL          Link öffnet Website in neuem Tab                    

  Entscheider               Name + Email (oder „info@" wenn kein Name)          Wen erreichen wir?

  Business Health           Score als Farbbalken (0-100)                        Lohnt sich der Lead?

  Google Rating + Reviews   Sterne + Anzahl                                     Aktivitäts-Signal

  Social Media              Icons: Instagram ✓/✗, LinkedIn ✓/✗, Facebook ✓/✗    Online-Affinität

  Industrie-Daten           Branchenspezifisch: Coaching-Art, Salon-Typ, etc.   Passt der Lead?

  Enrichment-Kosten         Was hat Enrichment gekostet (€)                     Budget-Awareness
  ------------------------- --------------------------------------------------- ---------------------------

-   Smart-Filter: „Nur Leads mit persönlicher Email", „Nur Health \>60", „Nur mit Instagram"

-   Approve → Leads gehen in Website-Crawl + Analyse (Steps 4-7)

-   Nurture → Leads gehen in Nurture-Pool (bekommen nur Analyse-Report, keine Website)

2.3 Gate 3: „Für welche Leads bauen wir?"

Die wichtigste Entscheidung. Zeigt scored Leads mit allen Analyse-Daten:

  -------------------------- ------------------------------------------ ----------------------------------------
  **Spalte**                 **Daten**                                  **Zweck**

  ☐ Checkbox                 Bulk-Select                                

  Firmenname                 Link zum Full Detail View                  

  Score                      Gesamtscore + Kategorie-Badge (Hot/Warm)   Hauptentscheidungs-Kriterium

  Website-Score (alt)        Quality-Review-Score als Farbbalken        Wie schlecht ist die aktuelle Website?

  Competitive Position       Position 1-5 Badge                         Wie steht der Lead vs. Konkurrenz?

  Content-Verfügbarkeit      Extraction-Completeness als Farbbalken     Genug Content für guten Rebuild?

  Top-3 Schwächen            Klasse-A Schwachstellen als Tags           Kernargumente für Outreach

  Alt-Website Thumbnail      Screenshot klickbar (Full-Preview)         Visueller Quick-Check

  Vorgeschlagenes Template   Template-Name + Variante                   Was würde gebaut werden
  -------------------------- ------------------------------------------ ----------------------------------------

-   Expand-Row: Klick auf einen Lead klappt eine Detail-Zeile aus mit: Competitor-Vergleichsmatrix, Outreach-Argumente, Brand Assets, Content-Übersicht

-   Approve für Rebuild → Lead geht in Steps 8-12 (Plan → Polish → Build → QA → Deploy)

-   Approve für Nurture → Lead bekommt Analyse-Report per Email, keine Website

-   Skip/Archive → Lead wird archiviert

-   VIP-Flag: Markiere einen Lead als VIP → wird in allen Queues priorisiert, höchste Qualität

2.4 Gate 4: „Ist diese Website gut genug?"

Pro Website ein Full-Screen-Review. Nacheinander abarbeiten (nicht als Liste, sondern einer nach dem anderen):

1.  Split-View: Links alte Website (Screenshot/iframe), rechts neue Website (iframe). Toggle Desktop/Mobile/Tablet

2.  Verbesserungs-Summary: „3 Blocker gelöst, PageSpeed 23→95, Mobile-optimiert, Schema Markup hinzugefügt"

3.  Polished Content Review: Alle Texte pro Section aufklappbar. Source-References sichtbar. Banned-Phrase-Hits markiert. Fact-Check-Status. Editierbar per Inline-Editor

4.  Bilder-Review: Alle verwendeten Bilder mit Label (Eigen/NanoBanana/Platzhalter). Platzhalter gelb markiert. Klick zum Austauschen

5.  QA-Report: Alle Test-Ergebnisse. Blocker/Warning aufgelistet mit Status (fixed/open). Lighthouse-Scores als Gauges

6.  Rebuild-Plan Übersicht: Template, Sections, Brand-Entscheidung, SEO-Keywords

7.  Aktionen: APPROVE (Website geht zu Outreach Prep) / REJECT + Notiz (zurück zum Build/Planner) / EDIT CONTENT (Inline-Editor für Texte) / CHANGE TEMPLATE (anderes Template, Rebuild) / EDIT IMAGES (Bilder tauschen/entfernen)

2.5 Gate 5: „Sollen diese Emails raus?"

Pro Lead: Alle 5 Outreach-Emails als Email-Inbox-Preview:

1.  Email-Preview wie echte Inbox: Absendername, Subject Line, Body mit aufgelösten Variablen, Attachments (Analyse-PDF Thumbnail). Genau so wie der Lead die Email bei Gmail/Outlook sehen würde

2.  Preview-Link testen: Klick auf Preview-URL öffnet die Preview GENAU wie der Lead sie sehen würde (mit Sales-Banner, Vergleich, Countdown)

3.  Analyse-PDF: PDF inline anzeigen (kein Download nötig). Score-Diagramm, Schwächen, Competitor-Vergleich, Preview-Screenshot

4.  Per-Email-Aktionen: Approve / Edit (Inline-Text-Editor für Subject + Body) / Skip (diese Email nicht senden, nächste in Sequenz überspringen)

5.  Gesamt-Aktionen: APPROVE ALL 5 + START SEQUENCE / APPROVE mit Edits / HOLD (nicht jetzt, später) / SKIP (kein Outreach für diesen Lead)

6.  Kanal-Änderung: „Statt Email: Instagram DM senden" oder „Statt Email: Physischer Brief" (nur Text ändert sich, Inhalt bleibt)

Modul 3: Kampagnen-Management

3.1 Neue Kampagne starten

-   Industrie: Dropdown (Coaches, Friseure, etc.)

-   Region: Stadt oder PLZ-Bereich. Autocomplete mit deutschen Städten

-   Quellen: Checkboxes (Google Maps, Branchenportal, LinkedIn, Instagram). Default aus Industry Profile

-   Anzahl: Slider 10-500. „Wie viele Leads sollen gescraped werden?"

-   Pipeline-Tiefe: Radio-Buttons: „Nur Scrapen + Validieren" / „Bis Enrichment + Scoring" / „Bis Preview (ohne Outreach)" / „Full Pipeline"

-   Gate-Modus: Toggle pro Gate: MANUAL (Pipeline stoppt, du entscheidest) / AUTO (Pipeline läuft wenn Threshold erreicht). Default: Alle MANUAL

-   Scheduling: „Jetzt starten" oder „Planung: Jeden Montag 8:00 Uhr" (wiederkehrend)

-   Start-Button: Kampagne startet. Redirect zum Kampagnen-Monitor

3.2 Kampagnen-Monitor

-   Live-Fortschritt: Progressbar mit Zahlen („134/200 gescraped, 98 validated, 12 disqualified, 86 warten auf Gate 1")

-   State-Breakdown: Kreisdiagramm: Wie viele Leads in welchem State

-   Pause/Resume/Cancel: Kampagne jederzeit pausieren oder abbrechen

-   Gate-Trigger: Wenn genug Leads für Gate 1 da sind: Button „Gate 1 öffnen -- 86 Leads reviewen"

3.3 Kampagnen-Vergleich

  --------------------------- ----------------------------- ------------------------------ -------------------------------------
  **Metrik**                  **Kampagne A (Coaches Mü)**   **Kampagne B (Friseure HH)**   **Interpretation**

  Leads gescraped             200                           180                            

  Validation Pass Rate        82%                           74%                            Ähnlich

  Gate 1 Approved             120 (60%)                     90 (50%)                       Coaches haben bessere Datenqualität

  Avg. Business Health        62                            71                             Friseur-Salons sind „gesünder"

  Gate 3 Approved (Rebuild)   18                            12                             Mehr Rebuilds bei Coaches

  Previews Built              15                            10                             

  Outreach Sent               12                            8                              

  Opened                      7 (58%)                       3 (38%)                        Coaches reagieren besser auf Email

  Converted                   2                             1                              

  Revenue                     €3.480                        €990                           Coaches kaufen teurere Pakete

  Cost per Lead               €0.72                         €0.68                          

  Cost per Acquisition        €86                           €122                           Coaches profitabler
  --------------------------- ----------------------------- ------------------------------ -------------------------------------

Modul 4: Manuelles Lead-Management

4.1 Lead manuell anlegen

Button „+ Neuer Lead" im Header. Formular:

-   Pflicht: Firmenname + Website-URL + Industrie (Dropdown)

-   Optional: Telefon, Email, Adresse, Google Place ID, Notizen

-   Option: „Sofort validieren + enrichen" (Checkbox, default: an)

-   Option: „Als VIP markieren" (Checkbox, default: aus)

-   Submit → Lead wird erstellt und je nach Optionen sofort in die Pipeline geschoben

4.2 One-Off Website Transform

+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Das interne Power-Tool**                                                                                                                                                                                   |
|                                                                                                                                                                                                              |
| Manchmal willst du für eine bestimmte Firma einfach eine neue Website sehen -- ohne Scraping, ohne Scoring, ohne Outreach. Einfach: URL rein → Website raus. Für Demos, für persönliche Kontakte, für Tests. |
+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

-   Eingabe: Website-URL + Industrie + (optional) Firmenname + (optional) Template-Präferenz

-   Klick „Transform" → System führt Steps 4-12 aus: Crawl → Extract → Review → Plan → Polish → Build → QA → Deploy

-   Skipped: Scraping (Step 1-2), Enrichment (Step 3), Scoring (Step 7), Outreach (Steps 13-14), Payment (Step 15)

-   Live-Fortschritt: Progressbar zeigt aktuellen Step. Geschätzte Dauer: 3-5 Minuten

-   Ergebnis: Preview-URL + Side-by-Side-Vergleich direkt im Backend

-   Aktion danach: „In Pipeline übernehmen" (Lead wird erstellt mit fertigem Preview) ODER „Nur Preview" (Lead wird als Demo markiert, kein Outreach)

4.3 Lead-Priorisierung (VIP)

-   VIP-Flag auf jedem Lead setzbar (Toggle im Lead Detail oder bei Gate-Approval)

-   VIP-Leads werden in ALLEN Queues nach vorne geschoben (BullMQ Job-Priorität)

-   VIP-Leads überspringen Auto-Approve NICHT -- sie bekommen IMMER manuelles Review

-   VIP-Badge sichtbar in allen Listen und dem Detail View

-   Anwendung: Persönliche Kontakte, Empfehlungen, strategisch wichtige Leads

Modul 5: Content-Editing & Template-Overrides

5.1 Inline-Text-Editor

In Gate 4 (Website Review) und im Lead Detail View: Texte direkt bearbeiten.

-   Pro Section: Aufklappbar. Links der aktuelle Text, rechts ein Editor (Textarea oder Rich-Text mit Basis-Formatting)

-   Source-References sichtbar: Unter jedem Absatz: „Quelle: Über-uns-Seite, Absatz 2". Hilft beim Verständnis woher der Text stammt

-   Banned-Phrase-Highlight: Wenn du versehentlich eine Banned Phrase tippst, wird sie rot markiert mit Tooltip „Verbotene Phrase: \[Alternative\]"

-   Speichern: „Änderung übernehmen" → Text wird im PolishedContent-Record aktualisiert. Website wird automatisch neu gebaut (wenn nur Text geändert: Rebuild in \<30s)

-   History: Jede Textänderung wird versioniert. Undo möglich

5.2 Bild-Management pro Lead

-   Bildergalerie: Alle Bilder die für diesen Lead verwendet werden. Label: Eigen / NanoBanana / Platzhalter

-   Bild austauschen: Klick auf Bild → „Neu generieren" (NanoBanana mit angepasstem Prompt) ODER „Bild hochladen" (eigenes Bild)

-   Bild entfernen: Section ohne Bild anzeigen (wenn optional) oder mit generischem Platzhalter ersetzen

-   Prompt-Editor: Für NanoBanana-Bilder: Den Generierungs-Prompt sehen und anpassen. „Regenerate" Button

5.3 Template-Override pro Lead

-   Im Lead Detail View: „Template ändern" Dropdown mit allen verfügbaren Templates der Industrie

-   Template-Variante ändern: Hero-Stil, Font-Pairing, Layout-Variante

-   Farb-Override: Eigenes Farbschema für diesen Lead setzen (Color Picker). Überschreibt Brand-Detection

-   Section-Override: Sections manuell hinzufügen/entfernen/umordnen (Drag-and-Drop)

-   Custom CSS: Für Power-User: Kleines CSS-Feld für individuelle Anpassungen (z.B. spezielles Font, Padding)

-   Rebuild: Nach jeder Änderung: „Rebuild" Button → Website wird mit neuen Einstellungen neu gebaut

5.4 Email-Text-Editor

-   In Gate 5: Alle 5 Emails inline editierbar

-   Subject Line: Einzeilig, mit Character-Counter (max 60 Zeichen für beste Open Rate)

-   Body: Rich-Text-Editor. Variablen als blaue Chips angezeigt ({{firmenname}} = klickbar, zeigt aufgelösten Wert)

-   Preview-Toggle: „Raw" (mit Variablen) vs. „Preview" (mit aufgelösten Werten)

-   Inbox-Preview: So sieht die Email in Gmail aus (Absender, Subject, erste Zeile Preview-Text, Body)

-   Speichern pro Email: Jede Email einzeln speicherbar. Änderungen werden im OutreachPackage-Record gespeichert

Modul 6: Lead Explorer (erweitert)

6.1 Global Search

Suchfeld im Header -- durchsucht ALLES:

-   Firmenname, URL, Domain, Email, Entscheider-Name, Telefonnummer, Kunden-Domain

-   Instant-Search mit Debounce (300ms). Ergebnisse als Dropdown unter dem Suchfeld

-   Ergebnis-Typen: Lead (mit State-Badge), Kunde (mit Paket-Badge), Kampagne

-   Klick auf Ergebnis → direkt zum Detail View

-   Keyboard-Shortcut: Cmd+K (wie Spotlight/Command Palette)

6.2 Lead-Tabelle

Wie in v1 definiert, plus:

-   VIP-Badge Spalte (Stern-Icon, filterbar)

-   Approval Gate Spalte: Zeigt an welchem Gate der Lead wartet („Gate 3: Waiting" als Badge)

-   Kampagne: Aus welcher Kampagne kommt der Lead

-   Thumbnail der alten Website (Mini-Screenshot, hover für größere Ansicht)

-   Preview-Thumbnail (wenn vorhanden): Mini-Screenshot der neuen Website

-   Inline-Quick-Actions: Hover über eine Zeile zeigt: ▶ Approve, 👁 View, ↻ Retry, 🗑 Archive als Icon-Buttons

6.3 Vordefinierte Views (erweitert)

  ------------------- ------------------------------------------------------ ------------------------------------------
  **Tab**             **Filter**                                             **Zweck**

  📥 Inbox: Gate 1    state = validated, approval_gate = 1                   Leads die auf Enrichment-Approval warten

  📥 Inbox: Gate 2    state = enriched, approval_gate = 2                    Leads die auf Analyse-Approval warten

  📥 Inbox: Gate 3    state = scored, approval_gate = 3                      Leads die auf Rebuild-Approval warten

  📥 Inbox: Gate 4    state = qa_passed, approval_gate = 4                   Websites die auf Review warten

  📥 Inbox: Gate 5    state = outreach_prepped, approval_gate = 5            Outreach die auf Freigabe wartet

  ⭐ VIP              is_vip = true                                          Alle priorisierten Leads

  🔥 Hot Leads        score_category = hot                                   Top-Leads

  🛠 In Bearbeitung    state IN (planning, polishing, building, qa_testing)   Leads die gerade gebaut werden

  🚀 Previews Live    preview_url IS NOT NULL                                Alle live Previews

  📧 Outreach Aktiv   state = outreach_active                                Laufende Email-Sequenzen

  💰 Converted        state IN (converted, onboarding, live)                 Zahlende Kunden
  ------------------- ------------------------------------------------------ ------------------------------------------

Modul 7: Lead Detail View (erweitert)

Wie in v1, plus folgende kritische Erweiterungen:

7.1 Header-Erweiterungen

-   VIP-Toggle: Stern-Icon neben dem Firmennamen. Ein Klick = VIP an/aus

-   Aktuelles Gate: Badge zeigt „Wartet auf Gate 3" (wenn zutreffend) mit „Jetzt reviewen" Button

-   Notiz-Button: „+ Notiz" → öffnet Notiz-Modal. Notizen erscheinen im Audit Trail

-   Quick-Nav: Pfeile links/rechts zum nächsten/vorherigen Lead in der aktuellen Liste (z.B. Gate-Queue)

7.2 Neue Tabs

Tab: Kampagne

-   Aus welcher Kampagne kommt der Lead (Link zur Kampagne)

-   Kampagnen-Kontext: Wie viele andere Leads aus dieser Kampagne, Avg. Score, Conversion Rate

Tab: Kommunikation (NEU)

-   Alle Interaktionen chronologisch: Outreach-Emails (gesendet/geöffnet/geklickt), Kunden-Replies, Support-Tickets, Telefon-Notizen, Interne Notizen

-   Inline-Antwort: „Email schreiben" Button → öffnet Compose-Feld direkt im Tab

-   Call-Log: „Anruf loggen" Button → Datum, Dauer, Outcome (Interested/Not interested/Follow-up/Converted), Notizen

-   Calendly-Integration: Wenn Lead ein Gespräch gebucht hat: Termin-Details, Meeting-Link, Status (Upcoming/Completed/No-Show)

Tab: Overrides (NEU)

-   Alle manuellen Änderungen die für diesen Lead gemacht wurden

-   Template-Override: Welches Template wird statt des Auto-Vorschlags verwendet

-   Content-Edits: Welche Texte wurden manuell geändert (Diff-View: Original vs. Edit)

-   Bild-Änderungen: Welche Bilder wurden manuell getauscht

-   Email-Edits: Welche Outreach-Emails wurden manuell angepasst

-   Scoring-Override: „Diesen Lead trotz Score 55 in Full Pipeline schieben" (mit Begründung)

Modul 8: Notifications & Alerts

8.1 In-App Notification Center

-   Glocken-Icon im Header mit Badge-Counter (ungelesene Notifications)

-   Dropdown mit den letzten 20 Notifications. Klick → zur relevanten Seite

-   Kategorien: Gate-Approval („47 Leads warten auf Gate 1"), QA-Alert („Website failed QA"), System („Webhook inaktiv"), Revenue („Neue Conversion!"), Customer („Änderungswunsch eingegangen")

8.2 Slack-Integration

-   Webhook zu einem Slack-Channel (konfigurierbar)

-   Was wird gepostet: Neue Conversions (💰), QA-Failures (⚠️), System-Alerts (🚨), Gate-Queues über Threshold („50+ Leads warten auf Gate 1")

-   Konfigurierbar: Welche Event-Typen nach Slack gehen

8.3 Email-Alerts (optional)

-   Tägliche Zusammenfassung per Email: „Gestern: 45 Leads gescraped, 3 Conversions, €4.470 Revenue, 2 System-Alerts"

-   Kritische Alerts sofort per Email: System down, Bounce-Rate \>5%, Payment failed

8.4 Webhook-Health-Monitor

  ------------------------------- -------------------------------- ------------------- ------------------------------
  **Webhook**                     **Erwartet**                     **Letztes Event**   **Status**

  Instantly.ai (Opens/Clicks)     Alle 1-2h wenn Sequenzen aktiv   Timestamp           Grün/Gelb (\>4h)/Rot (\>12h)

  Stripe (Payments)               Bei Checkout                     Timestamp           Grün/Rot

  Plausible (Preview Analytics)   Wenn Previews besucht werden     Timestamp           Grün/Gelb/Rot

  Vercel (Deploy Status)          Bei Deployments                  Timestamp           Grün/Gelb/Rot
  ------------------------------- -------------------------------- ------------------- ------------------------------

Modul 9: Scheduling & Automation Rules

9.1 Kampagnen-Scheduling

-   Wiederkehrende Kampagnen: „Jeden Montag 8:00: 100 Coaches in München scrapen"

-   Cron-ähnliche Konfiguration: Tag(e), Uhrzeit, Industrie, Region, Anzahl

-   Aktive Schedules als Liste mit On/Off Toggle, nächster Lauf, letzter Lauf

9.2 Outreach-Scheduling

-   Sendezeiten pro Industrie konfigurieren: „Coaches: Mo-Mi 10:00-11:30", „Friseure: Mo 10:00-14:00"

-   Outreach-Pause: „Keine Emails zwischen 23.12. und 02.01." (Feiertage)

-   Rate-Limit: Max X neue Sequenzen pro Tag (schützt Sender-Reputation)

9.3 Auto-Approval Rules (für spätere Skalierung)

  ---------- --------------------------------------------------------------------- ---------------------
  **Gate**   **Auto-Approve Bedingung**                                            **Default**

  Gate 1     Validation Confidence \> \[Threshold\] UND Google Rating \> \[Min\]   OFF (alles manuell)

  Gate 2     Business Health \> \[Threshold\] UND Entscheider identifiziert        OFF

  Gate 3     Final Score \> \[Threshold\] UND Content Completeness \> \[Min\]      OFF

  Gate 4     QA Verdict = pass UND Visual QA \> \[Min\] UND 0 Blocker              OFF

  Gate 5     Email Validated UND alle Variablen aufgelöst UND kein Banned Phrase   OFF
  ---------- --------------------------------------------------------------------- ---------------------

Jede Rule hat einen Toggle (ON/OFF) und konfigurierbare Thresholds. Wenn ON: Pipeline läuft automatisch durch wenn Bedingung erfüllt. Wenn Bedingung NICHT erfüllt: Lead wartet trotzdem auf manuelles Approval.

Modul 10: Kunden-Management & Support

10.1 Kunden-Übersicht

-   Alle zahlenden Kunden: Firma, Paket, Domain, Go-Live-Datum, MRR, Churn-Risk, letzter Kontakt

-   Onboarding-Status: Pending / DNS-Switch / Verification / Live mit Progress-Badge

-   Überfällige Onboardings rot markiert (\>48h bei API-Provider, \>5d bei manuell)

10.2 Kunden-Detail

-   Kontaktdaten + Stripe Customer Portal Link

-   Website-Status: Live + Uptime (letzte 30 Tage) + aktueller Lighthouse-Score

-   Formular-Submissions: Alle eingegangenen Kontaktanfragen (aus Formspree)

-   Besucher-Statistik: Chart aus Plausible

-   Änderungswünsche: Alle eingegangenen Tickets mit Status (Open/In Progress/Done)

-   Communication Log: Alle Emails, Anrufe, Notizen chronologisch

10.3 Support-Ticket-System

-   Änderungswünsche die über das Kunden-Dashboard eingehen landen hier als Tickets

-   Pro Ticket: Kunde, Inhalt, Priorität (Auto: Normal. Manuell: Urgent), Status, Zugewiesen an

-   Quick-Actions: „Text ändern" (Inline-Edit am Live-Website-Content), „Bild tauschen", „Seite hinzufügen", „Kunde kontaktieren"

-   SLA-Tracking: Tickets offen \>24h werden gelb, \>48h rot

10.4 Billing-Aktionen

-   Custom-Rabatt geben: Prozent oder Festbetrag auf nächste Rechnung

-   Refund auslösen: Teilrefund oder Vollrefund über Stripe. Grund dokumentieren

-   Zahlungserinnerung senden: Bei failed Payment -- Email mit Update-Link

-   Paket-Upgrade/Downgrade: Kunde von Starter auf Professional upgraden (Stripe Subscription ändern)

-   Hosting verlängern: Manuell Hosting-Zeitraum verlängern (z.B. als Goodwill)

10.5 DNS/Onboarding-Management

-   Dedizierte Ansicht: Alle Domains im Umzug als Kanban (Formular ausfüllen → DNS-Switch → Verification → Live)

-   Pro Domain: Provider erkannt, MX-Snapshot gespeichert, A/CNAME-Änderung dokumentiert, SSL-Status, Email-Test-Ergebnis

-   Rollback-Button: Ein Klick = DNS-Records auf Pre-Switch-Werte zurücksetzen

-   Anleitung generieren: „DNS-Anleitung für Strato generieren" → PDF mit Screenshots, bereit zum Senden an Kunden

Modul 11: Configuration Center (erweitert)

Wie in v1, plus folgende Ergänzungen:

11.1 Calibration Set Manager (NEU)

-   Pro Industrie: 25-30 Referenz-Websites mit manuellem Score (Excellent/Good/Poor/Terrible)

-   UI: Website hinzufügen (URL eingeben), Screenshot wird automatisch erstellt, manuellen Score vergeben (0-100 + Kategorie)

-   AI-Vergleich: „AI-Score vs. manueller Score" Tabelle. Abweichung \>12 Punkte = gelb markiert

-   Drift-Monitor: Chart über Zeit: Wie stark weicht der AI-Score vom manuellen Score ab? Trend steigend = Re-Calibration nötig

-   Re-Calibration starten: Button der den AI-Prompt anpasst basierend auf den größten Abweichungen

11.2 A/B-Test-Manager (NEU)

-   Test erstellen: Was testen? (Email-Subject, Scoring-Weights, Template-Variante, Outreach-Kanal). Variante A vs. B definieren. Traffic-Split (z.B. 50/50 oder 90/10)

-   Test monitoren: Live-Ergebnisse: Open Rate, Click Rate, Conversion Rate pro Variante. Statistische Signifikanz anzeigen (braucht n\>30 pro Variante)

-   Test abschließen: Gewinner deklarieren. Gewinner-Variante wird zum neuen Default. Verlierer-Variante archiviert

-   Test-History: Alle vergangenen Tests mit Ergebnissen. Lernings dokumentieren

11.3 Template-Vorschau-Tool (NEU)

-   Template auswählen + Variante (Hero-Stil, Fonts, Farben)

-   Dummy-Content oder echten Lead-Content wählen

-   Live-Preview: So sieht das Template mit diesem Content aus. Desktop + Mobile Toggle

-   Template-Performance: Conversion Rate pro Template. Avg. Visual QA Score. Nutzungs-Häufigkeit

11.4 Asset Library (NEU)

-   Zentrale Bibliothek aller NanoBanana-generierten Bilder, extrahierten Logos, Farb-Paletten

-   Filterbar nach: Industrie, Bildtyp (Hero, Ambient, Portrait, Logo), Qualität, Datum

-   Wiederverwenden: Bei einem neuen Lead ein bestehendes Asset zuweisen statt neu generieren

-   Upload: Eigene Bilder hochladen und taggen (für Kunden die eigene Fotos nachliefern)

11.5 Competitor-Datenbank (NEU)

-   Alle analysierten Konkurrenten über alle Leads hinweg als zentrale Datenbank

-   Filterbar nach Stadt, Industrie, Features, Website-Score

-   Vermeidet Doppel-Analysen: Wenn Konkurrent X schon für Lead A analysiert wurde, Ergebnisse für Lead B wiederverwenden

-   Branchen-Insights: „Die Top-Features bei Coaches in München: 80% haben Calendly, 60% haben Blog, 30% haben Podcast"

Modul 12: Reporting & Analytics

12.1 Pipeline Dashboard (Live)

Wie in v1: Funnel, Live-Zahlen, Industry-Split. Jetzt mit Gate-Indikator: An jedem Gate-Punkt im Funnel wird angezeigt wie viele Leads auf Approval warten.

12.2 Periodische Reports

-   Wochen-Report: Leads gescraped, validated, enriched, scored, rebuilt, deployed, outreached, converted. Revenue. Kosten. Profit. Pro Industrie

-   Monats-Report: Gleiches + Vergleich zum Vormonat (Trend-Pfeile). Conversion-Funnel-Analyse. Kampagnen-Vergleich. Template-Performance

-   Export: PDF-Download oder Email-Versand (konfigurierbar: jeden Montag morgens den Wochen-Report)

12.3 Revenue Analytics

-   MRR + MRR-Wachstum über Zeit (Liniendiagramm)

-   Revenue pro Industrie (Balkendiagramm)

-   Avg. Deal Size Trend

-   LTV vs. CAC pro Industrie

-   Churn-Rate über Zeit

-   Revenue-Forecast: Basierend auf aktueller Pipeline („Wenn 30% der Hot Leads konvertieren: €X erwarteter Revenue")

12.4 Kosten-Report

-   API-Kosten pro Step über Zeit

-   Cost-per-Lead und Cost-per-Acquisition Trend

-   Kosten-Breakdown pro Industrie

-   Budget vs. Actual (Monatlich)

-   Kosten-Anomalien: Tage an denen Kosten \>150% des Durchschnitts

Modul 13: System Health & Team

13.1 System Health

Wie in v1 (Service-Status, Queue-Monitor, Error-Dashboard). Jetzt ergänzt um Webhook-Health-Monitor (Modul 8.4).

13.2 Team & Rollen (NEU)

  ----------------- ----------------------------------------------------------------------------------------------------------------- ----------------------------------
  **Rolle**         **Zugriff**                                                                                                       **Typischer Nutzer**

  **Admin**         Alles. Alle Module, alle Aktionen, Configuration, Team-Management                                                 Du (Gründer/Operator)

  **QA Reviewer**   Gate 4 + Gate 5 + QA Center + Lead Detail (read-only für andere Tabs). Kein Zugriff auf Config, Revenue, System   VA für Website-Reviews

  **Sales**         Lead Explorer + Lead Detail + Outreach Monitor + Revenue. Keine Pipeline Controls, keine Config                   Sales-Mitarbeiter für Telefonate

  **Support**       Kunden-Management + Support-Tickets + Lead Detail (read-only). Kein Zugriff auf Pipeline, Config, Revenue         Kunden-Support

  **Viewer**        Alles read-only. Keine Aktionen möglich                                                                           Investor-Demo, Stakeholder
  ----------------- ----------------------------------------------------------------------------------------------------------------- ----------------------------------

-   Team-Verwaltung: Nutzer einladen per Email, Rolle zuweisen, deaktivieren

-   Auth: Google OAuth (Google Workspace) oder Magic Link

-   Audit: Wer hat wann was getan (alle Aktionen im Admin werden geloggt mit User-ID)

Technische Spezifikation

URL-Struktur (erweitert)

  ---------------------------- -------------------------------------------
  **Route**                    **Modul**

  /                            Action Required (Homepage)

  /leads                       Lead Explorer

  /leads/new                   Lead manuell anlegen

  /leads/transform             One-Off Website Transform

  /leads/\[id\]                Lead Detail View

  /leads/\[id\]/edit-content   Content-Editing

  /leads/\[id\]/preview        Website Preview (Full-Screen)

  /gates/1                     Gate 1: Enrichment Approval

  /gates/2                     Gate 2: Analysis Approval

  /gates/3                     Gate 3: Rebuild Approval

  /gates/4                     Gate 4: Website Review

  /gates/5                     Gate 5: Outreach Approval

  /campaigns                   Kampagnen-Übersicht

  /campaigns/new               Neue Kampagne

  /campaigns/\[id\]            Kampagnen-Monitor

  /campaigns/compare           Kampagnen-Vergleich

  /outreach                    Outreach Monitor

  /customers                   Kunden-Übersicht

  /customers/\[id\]            Kunden-Detail

  /customers/onboarding        DNS/Onboarding Kanban

  /tickets                     Support-Tickets

  /revenue                     Revenue Dashboard

  /config                      Configuration Center

  /config/profiles/\[id\]      Industry Profile Editor

  /config/calibration          Calibration Set Manager

  /config/ab-tests             A/B-Test-Manager

  /config/templates            Template-Vorschau

  /config/assets               Asset Library

  /config/competitors          Competitor-Datenbank

  /config/suppression          Suppression List

  /config/scheduling           Scheduling & Automation Rules

  /reports                     Reporting & Analytics

  /system                      System Health

  /system/webhooks             Webhook-Health-Monitor

  /settings/team               Team & Rollen
  ---------------------------- -------------------------------------------

Datenbank-Erweiterungen

-   leads.is_vip (Boolean, default false) -- VIP-Flag

-   leads.approval_gate (Integer, nullable) -- An welchem Gate wartet der Lead (1-5, null = nicht wartend)

-   leads.manual_override (JSONB) -- Template-Override, Content-Edits, Scoring-Override, etc.

-   leads.campaign_id (UUID, FK) -- Referenz zur Kampagne

-   admin_notes (id, lead_id, user_id, text, created_at) -- Notizen von Team-Mitgliedern

-   campaigns (id, industry, region, lead_count, pipeline_depth, gate_config, schedule, status, stats, created_at)

-   qa_reviews (id, lead_id, user_id, step_reviewed, verdict, notes, created_at) -- Manuelle QA-Stichproben

-   support_tickets (id, customer_id, subject, body, priority, status, assigned_to, created_at, resolved_at)

-   call_logs (id, lead_id, user_id, date, duration_minutes, outcome, notes, calendly_event_id, created_at)

-   ab_tests (id, name, type, variant_a, variant_b, traffic_split, status, results, created_at)

-   calibration_sets (id, industry_tag, url, manual_score, manual_category, ai_score, drift, screenshot_s3_key, created_at)

-   team_members (id, email, name, role, active, last_login, created_at)

-   notification_preferences (user_id, slack_enabled, email_daily_summary, email_critical_alerts)

-   automation_rules (id, gate, condition, threshold, enabled, created_at) -- Auto-Approve-Konfiguration

-   schedules (id, type, config, cron_expression, enabled, last_run, next_run, created_at)

Build-Reihenfolge

+------------+-----------------------------------------------------+----------------------------------------------------+
| **Sprint** | **Module**                                          | **Abhängigkeiten**                                 |
+------------+-----------------------------------------------------+----------------------------------------------------+
| **1**      | Action Required Homepage (Modul 1)                  | Braucht: DB Schema + Pipeline Services laufen      |
|            |                                                     |                                                    |
|            | Gate Approval Screens -- Gate 1+2+3 (Modul 2)       |                                                    |
|            |                                                     |                                                    |
|            | Lead Explorer + Global Search (Modul 6)             |                                                    |
|            |                                                     |                                                    |
|            | Lead Detail View Basics (Modul 7)                   |                                                    |
+------------+-----------------------------------------------------+----------------------------------------------------+
| **2**      | Gate 4: Website Review mit Split-View (Modul 2)     | Braucht: Build + Deploy + Outreach Services laufen |
|            |                                                     |                                                    |
|            | Gate 5: Outreach Review mit Inbox-Preview (Modul 2) |                                                    |
|            |                                                     |                                                    |
|            | Content-Editing + Template-Override (Modul 5)       |                                                    |
|            |                                                     |                                                    |
|            | Manual Lead + One-Off Transform (Modul 4)           |                                                    |
+------------+-----------------------------------------------------+----------------------------------------------------+
| **3**      | Kampagnen-Management + Vergleich (Modul 3)          | Braucht: Scraper + Kampagnen-Modell                |
|            |                                                     |                                                    |
|            | Scheduling & Automation Rules (Modul 9)             |                                                    |
|            |                                                     |                                                    |
|            | Notifications + Slack (Modul 8)                     |                                                    |
+------------+-----------------------------------------------------+----------------------------------------------------+
| **4**      | Kunden-Management + Support-Tickets (Modul 10)      | Braucht: Payment + Onboarding Services             |
|            |                                                     |                                                    |
|            | DNS/Onboarding Kanban (Modul 10.5)                  |                                                    |
|            |                                                     |                                                    |
|            | Billing-Aktionen (Modul 10.4)                       |                                                    |
+------------+-----------------------------------------------------+----------------------------------------------------+
| **5**      | Configuration Center Enhanced (Modul 11)            | Braucht: Stabile Pipeline + Daten                  |
|            |                                                     |                                                    |
|            | Calibration Set Manager                             |                                                    |
|            |                                                     |                                                    |
|            | A/B-Test-Manager                                    |                                                    |
|            |                                                     |                                                    |
|            | Template-Vorschau                                   |                                                    |
|            |                                                     |                                                    |
|            | Asset Library                                       |                                                    |
|            |                                                     |                                                    |
|            | Competitor-DB                                       |                                                    |
+------------+-----------------------------------------------------+----------------------------------------------------+
| **6**      | Reporting & Analytics (Modul 12)                    | Braucht: 1-2 Monate Live-Daten                     |
|            |                                                     |                                                    |
|            | System Health + Webhook Monitor (Modul 13)          |                                                    |
|            |                                                     |                                                    |
|            | Team & Rollen (Modul 13.2)                          |                                                    |
+------------+-----------------------------------------------------+----------------------------------------------------+

*Webolution Operations Command Center v2.0 -- SocialCooks*
