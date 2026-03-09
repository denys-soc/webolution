# Webolution Industry Profiles

## Ranking (weighted score out of 10)

1. **Coaches – 7.75** → Pilot #1
2. **Friseure – 7.35** → Pilot #2
3. Steuerberater – 6.75 → Phase 2
4. Anwälte – 6.55 → Phase 3
5. Handwerker – 6.05 → Phase 4
6. Ärzte – 5.40 → Phase 5

## Pilot Profile 1: Coaches

**Why first**: Reachability 10/10 (coach IS the decision maker), Response 9/10 (online-savvy), Website weakness density 9/10 (70-80% have bad sites). Challenge: variable payment willingness.

### JSON Config

```json
{
  "id": "coaches",
  "name": "Coaches & Berater",
  "active": true,
  "version": 1,

  "scraping": {
    "searchTerms": ["Coach", "Coaching", "Business Coach", "Life Coach", "Karrierecoach", "Führungskräftecoaching", "Systemischer Coach", "Mental Coach", "Health Coach", "Ernährungscoach"],
    "sourcePriority": ["google_maps", "linkedin", "coach_databases", "instagram"],
    "geoRadiusKm": 50,
    "excludeChains": [],
    "excludeKeywords": ["Agentur", "Beratung GmbH", "Sport", "Fitness"],
    "expectedYieldPerCity": [20, 80],
    "scrapingFrequency": "2x_weekly"
  },

  "enrichment": {
    "requiredFields": ["coachingType", "method", "targetAudience"],
    "enrichmentSources": ["impressum", "linkedin", "instagram", "coach_associations"],
    "decisionMakerStrategy": "always_the_coach",
    "businessHealthSignals": {
      "linkedinFollowers": { "good": 500, "excellent": 2000 },
      "googleReviews": { "good": 5, "excellent": 20 },
      "instagramFollowers": { "good": 500, "excellent": 2000 },
      "hasPodcast": true,
      "hasYoutube": true
    },
    "preQualMinimum": "linkedin_or_website_or_instagram"
  },

  "websiteReview": {
    "benchmarkStandard": "Personal brand website with clear offering, personality, and conversion focus",
    "criticalElements": [
      "About section with personality (not generic)",
      "Clear offering: what, for whom, at what price",
      "Social proof: testimonials, references, results",
      "CTA above-the-fold: booking, intro call, contact",
      "Method/approach explanation"
    ],
    "niceToHave": ["leadMagnet", "blog", "podcast", "bookingTool", "instagramFeed", "aboutVideo"],
    "redFlags": ["genericTemplate", "noTestimonials", "noCTA", "buzzwordOverload", "underConstruction"],
    "visualBenchmark": "Modern, warm, personal. Not too corporate. Earth tones, blue/green, warm grey. Lifestyle photos. Clean, not cluttered"
  },

  "scoringOverrides": {
    "websiteWeakness": 0.20,
    "contentOpportunity": 0.20,
    "businessHealth": 0.15,
    "competitivePressure": 0.10,
    "reachability": 0.20,
    "dealPotential": 0.15
  },

  "rebuildTemplates": [
    { "id": "coach-landing", "condition": "solo, little content", "pages": 1, "sections": ["hero", "about", "offering", "testimonials", "cta", "contact"] },
    { "id": "coach-authority", "condition": "solo, blog/podcast exists", "pages": "3-5", "sections": ["home", "about", "offerings", "blog", "contact"] },
    { "id": "coach-business", "condition": "multiple coaches", "pages": "5-8", "sections": ["home", "team", "programs", "references", "blog", "contact"] },
    { "id": "coach-celebrity", "condition": "speaker/author", "pages": "5-8", "sections": ["home", "about", "books", "events", "media", "contact"] }
  ],

  "contentPolish": {
    "toneOfVoice": "Casual-professional. Du-form if coach uses Du on website. Motivating but not exaggerated. Authentic, not generic. Short sentences. Activating",
    "bannedPhrases": [
      "Entfalten Sie Ihr volles Potenzial",
      "Auf Augenhöhe begleiten",
      "Ganzheitlicher Ansatz",
      "Transformation erleben",
      "Ihr kompetenter Partner",
      "Raum für Veränderung"
    ],
    "powerWords": ["Klarheit", "Fokus", "Ergebnis", "Wachstum", "Authentizität", "Struktur", "Sparring", "Perspektive", "Umsetzung", "Momentum"],
    "ctaPrimary": "Kennenlerngespräch buchen",
    "ctaSecondary": "Kostenloses Erstgespräch",
    "imageStrategy": "Own photos of the coach highest priority (personal brand!). NanoBanana only for backgrounds/ambient. NEVER AI portrait of the coach!"
  },

  "outreach": {
    "emailTone": "Casual, direct, Du-form if coach uses Du. Short. To the point. No corporate-speak",
    "salutation": "Hi {{firstName}}",
    "primaryChannel": "email",
    "secondaryChannel": "instagram_dm",
    "tertiaryChannel": "linkedin_inmail",
    "painPoints": [
      "Dein Wissen und deine Erfahrung kommen online nicht rüber",
      "Potenzielle Klienten sehen eine Jimdo-Seite statt deiner Expertise",
      "Kein klarer Weg vom Besucher zum Kennenlerngespräch",
      "Deine Konkurrenz zeigt Ergebnisse und Testimonials – du nicht"
    ],
    "bestSendingTime": { "days": ["Mon", "Tue", "Wed"], "hours": "10:00-11:30" }
  }
}
```

## Pilot Profile 2: Friseure

**Why second**: Visual wow-effect 10/10 (most dramatic before/after), Website weakness 9/10, Scraping ease 9/10, Legal simplicity 9/10. Challenge: lower email affinity, lower payment willingness.

### JSON Config

```json
{
  "id": "friseure",
  "name": "Friseure & Salons",
  "active": true,
  "version": 1,

  "scraping": {
    "searchTerms": ["Friseur", "Friseursalon", "Haarsalon", "Barbershop", "Coiffeur", "Hair Salon", "Hairstylist"],
    "sourcePriority": ["google_maps", "treatwell", "booksy", "instagram", "gelbe_seiten"],
    "geoRadiusKm": 15,
    "excludeChains": ["Klier", "Super Cut", "HairExpress"],
    "excludeKeywords": ["Großhandel", "Shop", "Schule", "Akademie"],
    "expectedYieldPerCity": [50, 200],
    "scrapingFrequency": "1x_weekly"
  },

  "enrichment": {
    "requiredFields": ["priceSegment", "salonType", "ownerName"],
    "enrichmentSources": ["google_places", "treatwell", "instagram", "impressum"],
    "decisionMakerStrategy": "salon_owner_from_impressum_or_google",
    "businessHealthSignals": {
      "googleRating": { "good": 4.0, "excellent": 4.5 },
      "googleReviews": { "good": 20, "excellent": 50 },
      "instagramFollowers": { "good": 500, "excellent": 2000 },
      "hasTreatwellProfile": true,
      "hasBookingTool": true
    },
    "preQualMinimum": "google_business_active_and_10plus_reviews_and_website",
    "segmentDetection": {
      "budget": "avg_ladies_cut < 25",
      "mid": "avg_ladies_cut 25-50",
      "premium": "avg_ladies_cut > 50",
      "barbershop": "keywords: barber, beard in name/description"
    }
  },

  "websiteReview": {
    "benchmarkStandard": "Visually appealing salon presence with gallery, prices, online booking, clear location/hours",
    "criticalElements": [
      "Photo gallery (own work, not stock!)",
      "Price list (current, structured)",
      "Online booking tool (Treatwell, Booksy, Shore, Calendly)",
      "Opening hours + location + directions",
      "Team presentation (at least names + photos)"
    ],
    "niceToHave": ["beforeAfterPhotos", "instagramFeed", "productSales", "reviewWidget", "stylistProfiles"],
    "redFlags": ["noImages", "pricesOnRequest", "flashIntro", "lastUpdate3YearsAgo", "facebookOnlyNoWebsite"],
    "visualBenchmark": "Modern, bright, lifestyle-oriented. Large images. Warm colors or monochrome black/white (barbershop). Instagram aesthetic. Clean, not cluttered"
  },

  "scoringOverrides": {
    "websiteWeakness": 0.30,
    "contentOpportunity": 0.10,
    "businessHealth": 0.25,
    "competitivePressure": 0.15,
    "reachability": 0.10,
    "dealPotential": 0.10
  },

  "rebuildTemplates": [
    { "id": "salon-minimal", "condition": "budget salon, few images", "pages": "1-2", "sections": ["hero", "prices", "bookingCta", "location", "contact"] },
    { "id": "salon-gallery", "condition": "mid segment, good images", "pages": "3-4", "sections": ["galleryHero", "team", "prices", "booking", "location"] },
    { "id": "salon-premium", "condition": "premium salon, instagram active", "pages": "4-5", "sections": ["fullscreenHero", "gallery", "teamStories", "prices", "booking", "products"] },
    { "id": "salon-barber", "condition": "barbershop", "pages": "2-3", "sections": ["darkHero", "services", "team", "prices", "booking"] }
  ],

  "contentPolish": {
    "toneOfVoice": "Casual, inviting, lifestyle-oriented. Du/Ihr depending on salon brand. Short sentences. Visually descriptive. Less text, stronger headlines",
    "bannedPhrases": [
      "Weil Sie es sich wert sind",
      "Für ein perfektes Styling",
      "Ihr neues Ich",
      "Verwöhnprogramm",
      "Haarträume werden wahr"
    ],
    "powerWords": ["Stil", "Look", "Verwöhnen", "Kreativität", "Trend", "Wohlfühlen", "Inspiration", "Handwerk", "Lieblingsplatz"],
    "ctaPrimary": "Termin buchen",
    "ctaSecondary": "Preise ansehen",
    "imageStrategy": "Own photos MAX priority. Instagram images (with permission in sales call). NanoBanana for salon ambient shots. NO stock hair photos (immediately recognizable!)"
  },

  "outreach": {
    "emailTone": "Casual-friendly. Short. Visual: screenshot of new website directly in email. Less text, strong preview link",
    "salutation": "Hallo {{name}}",
    "primaryChannel": "email",
    "secondaryChannel": "instagram_dm",
    "tertiaryChannel": "google_business_message",
    "painPoints": [
      "Euer Salon sieht in echt besser aus als online",
      "Kunden suchen online nach Friseuren – und sehen eine Website von 2016",
      "{{competitorName}} in eurer Straße hat Online-Buchung – ihr noch nicht",
      "Eure {{googleRating}} Sterne auf Google sind super – aber eure Website zeigt sie nicht"
    ],
    "bestSendingTime": { "days": ["Mon"], "hours": "10:00-14:00", "note": "Monday is Ruhetag for many salons!" },
    "emailVisualHook": "Desktop screenshot of new website as inline image in email 1. Not attachment, directly visible. THIS is the conversion trigger for salons"
  }
}
```
