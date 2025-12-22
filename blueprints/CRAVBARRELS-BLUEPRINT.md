# CRAVBARRELS - MASTER BLUEPRINT
## Complete Asset, Image, History, Training, Law & Monetization Specification
### Generated: December 22, 2025 | CR AudioViz AI

---

## 🎯 EXECUTIVE SUMMARY

CRAVBarrels will be the **ultimate spirits collector and enthusiast platform** - combining:
- **22,000+ spirits database** with real images (not placeholders)
- **Museum-grade historical content** (Prohibition, mob history, vintage ads)
- **Training & certification prep** programs
- **Games & engagement** mechanics
- **Collector tools** (tracking, valuation, provenance)
- **Monetization** (subscriptions, advertising, affiliate, sponsorships)

This document provides everything needed to build features that make competitors like Distiller and Whiskybase look incomplete.

---

## 📊 SECTION 1: SPIRITS DATA APIs & SOURCES

### 1.1 COCKTAIL & RECIPE DATA

#### TheCocktailDB API
- **URL**: https://www.thecocktaildb.com/api.php
- **Free Tier**: Yes (with attribution)
- **Premium**: $2/month for more features
- **Data Available**:
  - Cocktail recipes (750+ drinks)
  - Ingredients list
  - Glass types
  - Categories (Ordinary Drink, Cocktail, Shot, etc.)
  - Drink images (high quality)
  - Instructions
- **Endpoints**:
  - `/search.php?s={name}` - Search by name
  - `/random.php` - Random cocktail
  - `/filter.php?i={ingredient}` - Filter by ingredient
  - `/lookup.php?i={id}` - Lookup by ID
- **Use Case**: Cocktail discovery, "What can I make?" feature, recipe cards

#### Punk API (Beer)
- **URL**: https://punkapi.com/documentation/v2
- **Data Available**:
  - BrewDog beer recipes
  - Ingredients, methods, tips
  - Food pairings
- **Use Case**: Beer knowledge expansion, brewing education

### 1.2 BREWERY & DISTILLERY DATA

#### Open Brewery DB
- **URL**: https://www.openbrewerydb.org/
- **Documentation**: https://www.openbrewerydb.org/documentation
- **Data Available**:
  - Brewery name, type (micro, regional, brewpub, etc.)
  - Address, city, state, country
  - Coordinates (latitude/longitude)
  - Website, phone
- **Endpoints**:
  - `/breweries` - List all
  - `/breweries?by_city={city}` - Filter by city
  - `/breweries?by_state={state}` - Filter by state
  - `/breweries?by_type={type}` - Filter by type
  - `/breweries/search?query={term}` - Search
- **Authentication**: None required
- **Use Case**: Distillery/brewery finder, maps, tour planning

### 1.3 LABEL & PRODUCT DATA

#### TTB COLA Registry (US Labels)
- **URL**: https://www.ttb.gov/public-records/public-cola-records
- **Access**: https://www.ttb.gov/public-records/cola-records
- **Data Available**:
  - Approved alcohol labels
  - Brand names
  - Producer/importer information
  - Approval dates
  - Label status (approved, expired, revoked, surrendered)
- **Format**: Searchable database, bulk downloads available
- **Use Case**: Brand lineage, label history, authenticity verification

#### COLA Cloud (Third-party)
- **URL**: https://colacloud.us/
- **Note**: Third-party aggregation - verify terms before production use
- **Data Available**:
  - Structured COLA data
  - Some label images (verify rights)
- **Use Case**: Supplementary label data

---

## 🖼️ SECTION 2: IMAGE ACQUISITION STRATEGY

### 2.1 THE "NO PLACEHOLDERS" PIPELINE

```
┌─────────────────────────────────────────────────────────────┐
│  IMAGE ACQUISITION PRIORITY                                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  TIER A: Official/Licensed Sources                          │
│  └── Partner feeds, API images with redistribution rights   │
│                                                             │
│  TIER B: Open Knowledge Graphs                              │
│  └── Wikidata + Wikimedia Commons (CC0/CC-BY)              │
│                                                             │
│  TIER C: Open License Search                                │
│  └── Openverse API (CC-licensed images)                     │
│                                                             │
│  TIER D: User-Generated Content                             │
│  └── "Missing Image Quest" uploads with rights declaration  │
│                                                             │
│  TIER E: Fallback (Last Resort Only)                        │
│  └── Category-level beautiful generic images                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 OPEN IMAGE SOURCES

#### Wikidata + Wikimedia Commons
- **Wikidata SPARQL**: https://www.wikidata.org/wiki/Wikidata:SPARQL_query_service
- **Commons API**: https://commons.wikimedia.org/wiki/Commons:API
- **MediaWiki Imageinfo**: https://www.mediawiki.org/wiki/API:Imageinfo
- **Strategy**:
  1. Query Wikidata for spirit entities with P18 (image) property
  2. Fetch image file URL from Wikimedia Commons
  3. Store with license attribution metadata
- **Example SPARQL**:
```sparql
SELECT ?item ?itemLabel ?image WHERE {
  ?item wdt:P31 wd:Q156 .  # Instance of alcoholic beverage
  ?item wdt:P18 ?image .    # Has image
  SERVICE wikibase:label { bd:serviceParam wikibase:language "en". }
}
LIMIT 100
```

#### Openverse API
- **URL**: https://api.openverse.org/v1/
- **Documentation**: https://api.openverse.org/v1/
- **Data Available**:
  - CC-licensed images from multiple sources
  - License metadata
  - Attribution information
- **Endpoints**:
  - `/images/?q={query}` - Search images
  - `/images/{id}` - Get image details
- **Filters**:
  - `license` - cc0, by, by-sa, by-nc, etc.
  - `license_type` - commercial, modification
- **Use Case**: Find CC-licensed bottle/brand images

### 2.3 USER-GENERATED CONTENT SYSTEM

#### "Missing Image Quest" Mechanics
```javascript
// Quest system for missing images
const imageQuest = {
  trigger: "spirit_missing_image",
  reward: {
    xp: 50,
    badges: ["Contributor", "Photographer"],
    premium_days: 1
  },
  requirements: {
    image_quality: "min_800px",
    rights_declaration: true,
    moderation_approval: true
  },
  gamification: {
    first_upload_bonus: 100,
    weekly_leader_bonus: 500,
    quality_bonus: 25
  }
};
```

### 2.4 IMAGE RIGHTS SCHEMA

```typescript
interface ImageAsset {
  id: string;
  spirit_id: string;
  url: string;
  source: 'wikimedia' | 'openverse' | 'user' | 'partner' | 'fallback';
  license: {
    type: 'cc0' | 'cc-by' | 'cc-by-sa' | 'cc-by-nc' | 'proprietary' | 'user-granted';
    attribution?: string;
    attribution_url?: string;
  };
  metadata: {
    uploaded_by?: string;
    upload_date: Date;
    moderation_status: 'pending' | 'approved' | 'rejected';
    quality_score?: number;
  };
}
```

---

## 📚 SECTION 3: HISTORICAL CONTENT SOURCES

### 3.1 LIBRARY OF CONGRESS APIs

#### Main API Portal
- **URL**: https://www.loc.gov/apis/
- **Documentation**: https://libraryofcongress.github.io/data-exploration/

#### Chronicling America (Historic Newspapers)
- **URL**: https://www.loc.gov/apis/additional-apis/chronicling-america-api/
- **Data Available**:
  - 19M+ digitized newspaper pages
  - OCR text searchable
  - High-resolution scans
  - 1789-1963 coverage
- **Search Queries for Alcohol Content**:
  - "prohibition" + "raid"
  - "bootlegger" OR "bootlegging"
  - "speakeasy"
  - "rum runner" OR "rum-running"
  - "Al Capone" + "alcohol"
  - "temperance"
  - "moonshine"
  - "whiskey" + "distillery"
- **Use Case**: Prohibition stories, mob coverage, vintage ads, "Today in History"

### 3.2 MUSEUM OPEN ACCESS APIs

#### Smithsonian Open Access
- **URL**: https://www.si.edu/openaccess
- **API**: https://www.si.edu/openaccess/devtools
- **Data Available**:
  - 3M+ images and records
  - Distilling equipment
  - Vintage advertisements
  - Bottles and decanters
  - Cultural artifacts
- **Authentication**: API key via api.data.gov
- **License**: CC0 for Open Access items

#### The Metropolitan Museum of Art
- **URL**: https://metmuseum.github.io/
- **API**: https://collectionapi.metmuseum.org/public/collection/v1/
- **Data Available**:
  - 470,000+ artworks
  - Glassware and decanters
  - Drinking vessels
  - Bar-related decorative arts
- **Endpoints**:
  - `/search?q={term}` - Search collection
  - `/objects/{id}` - Get object details
- **License**: CC0 for public domain works

#### New York Public Library Digital Collections
- **URL**: https://digitalcollections.nypl.org/
- **API**: https://api.repo.nypl.org/
- **Data Available**:
  - 900,000+ digitized items
  - Vintage menus (huge collection!)
  - Bar and restaurant photographs
  - Advertising ephemera
  - Prohibition-era materials
- **Authentication**: Token required
- **Use Case**: Vintage menu galleries, bar history

#### Digital Public Library of America (DPLA)
- **URL**: https://dp.la/
- **API**: https://pro.dp.la/developers/api-basics
- **Data Available**:
  - Aggregated cultural heritage
  - Cross-institutional search
  - Images, documents, artifacts
- **Authentication**: API key required
- **Use Case**: Broad historical content discovery

### 3.3 CONTENT CATEGORIES TO BUILD

| Category | Sources | Content Type |
|----------|---------|--------------|
| Prohibition Era | LOC, Chronicling America | News articles, photos, documents |
| Mob & Bootlegging | LOC, NYPL | Stories, court records, photos |
| Distillery History | Smithsonian, Met | Equipment, bottles, evolution |
| Vintage Advertising | LOC, NYPL, DPLA | Posters, print ads, signs |
| Bar Culture | NYPL, Met | Menus, glassware, interiors |
| Cocktail Evolution | All sources | Recipe history, trends |

---

## 🎓 SECTION 4: TRAINING & CERTIFICATION SYSTEM

### 4.1 LEARNING PATH STRUCTURE

```
┌─────────────────────────────────────────────────────────────┐
│  SPIRITS ACADEMY - LEARNING PATHS                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  🥉 LEVEL 1: EXPLORER (Free)                                │
│  ├── Introduction to Spirits Categories                     │
│  ├── Basic Tasting Terminology                              │
│  ├── Reading a Label 101                                    │
│  └── Quiz: Spirits Basics                                   │
│                                                             │
│  🥈 LEVEL 2: ENTHUSIAST ($9.99 or Subscription)             │
│  ├── Production Methods Deep Dive                           │
│  │   ├── Distillation Explained                            │
│  │   ├── Aging & Maturation                                │
│  │   └── Regional Terroir                                  │
│  ├── Tasting Like a Pro                                    │
│  │   ├── Nosing Techniques                                 │
│  │   ├── Palate Development                                │
│  │   └── Tasting Notes Writing                             │
│  └── Quiz: Intermediate Knowledge                           │
│                                                             │
│  🥇 LEVEL 3: CONNOISSEUR ($29.99 or Premium Sub)           │
│  ├── Whiskey Mastery                                       │
│  ├── Rum & Cachaça Deep Dive                               │
│  ├── Tequila & Mezcal Expert                               │
│  ├── Gin & Botanical Spirits                               │
│  ├── Brandy & Cognac                                       │
│  └── Certification Exam Prep                                │
│                                                             │
│  👑 LEVEL 4: MASTER (Premium Only)                          │
│  ├── Blind Tasting Mastery                                 │
│  ├── Collection Curation                                   │
│  ├── Investment & Valuation                                │
│  └── Master Certification                                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 4.2 CERTIFICATION PREP (IMPORTANT: Legal Positioning)

**DO NOT claim to replace official certifications.**

Instead, position as:
- "Study Companion for WSET"
- "Preparation Tools for Sommelier Exams"
- "Knowledge Checks for Cicerone Prep"

#### Official Programs to Reference (NOT Replace):
| Program | Focus | Our Position |
|---------|-------|--------------|
| WSET Spirits | Wine & Spirit Education Trust | Study companion, vocabulary builder |
| Certified Sommelier | Court of Master Sommeliers | Wine knowledge supplement |
| Cicerone | Beer certification | Beer module companion |
| Certified Specialist of Spirits | Society of Wine Educators | Prep quizzes, flashcards |

### 4.3 GAMIFIED LEARNING FEATURES

```javascript
const learningMechanics = {
  xp_system: {
    lesson_complete: 100,
    quiz_perfect: 250,
    streak_bonus: 50, // per day
    achievement_unlock: 500
  },
  
  badges: [
    { id: 'first_lesson', name: 'First Sip', icon: '🥃' },
    { id: 'whiskey_101', name: 'Whiskey Novice', icon: '🥃' },
    { id: 'tasting_10', name: 'Developing Palate', icon: '👅' },
    { id: 'history_buff', name: 'History Buff', icon: '📚' },
    { id: 'quiz_master', name: 'Quiz Master', icon: '🧠' },
    { id: 'perfect_week', name: 'Perfect Week', icon: '🔥' }
  ],
  
  leaderboards: {
    weekly: true,
    monthly: true,
    all_time: true,
    by_category: true
  }
};
```

---

## 🎮 SECTION 5: GAMES & ENGAGEMENT

### 5.1 GAME LIBRARY

| Game | Description | API/Data Source |
|------|-------------|-----------------|
| **Cocktail Mixologist** | Match ingredients to cocktails | TheCocktailDB |
| **Spirits History Trivia** | Prohibition, distilling history | Built-in DB |
| **Blind Tasting Challenge** | Identify spirits from descriptions | Custom content |
| **Label Detective** | Match labels to brands/eras | TTB COLA + LOC |
| **Distillery Tour** | Virtual exploration game | OpenBreweryDB + custom |
| **Prohibition Detective** | Solve crimes with newspaper clues | Chronicling America |
| **Bottle Collection Builder** | Build valuable collections | Collection system |
| **Price Is Right (Spirits)** | Guess bottle values | Price tracking data |

### 5.2 ALREADY BUILT GAMES

From today's development session:
1. **Cocktail Mixologist** - TheCocktailDB integration ✅
2. **Spirits History Trivia** - Built-in question bank ✅

### 5.3 ENGAGEMENT MECHANICS

```javascript
const engagementSystem = {
  daily_challenges: {
    tasting_note: { xp: 50, description: "Write a tasting note" },
    photo_upload: { xp: 75, description: "Upload a bottle photo" },
    quiz_complete: { xp: 100, description: "Complete daily quiz" },
    collection_update: { xp: 25, description: "Update collection" }
  },
  
  streaks: {
    day_3: { bonus: 50, badge: 'getting_started' },
    day_7: { bonus: 150, badge: 'weekly_warrior' },
    day_30: { bonus: 500, badge: 'monthly_master' },
    day_100: { bonus: 2000, badge: 'centurion' }
  },
  
  social: {
    share_collection: true,
    compare_bottles: true,
    clubs_and_groups: true,
    tasting_events: true
  }
};
```

---

## ⚖️ SECTION 6: LEGAL & REGULATORY COMPLIANCE

### 6.1 FEDERAL ALCOHOL REGULATIONS

#### TTB (Alcohol and Tobacco Tax and Trade Bureau)
- **URL**: https://www.ttb.gov/
- **Key Regulations**:
  - Labeling requirements (27 CFR Part 5)
  - Advertising standards (27 CFR Part 5.65)
  - Health warning requirements
  - Geographic designations (Bourbon, Scotch, Tequila, etc.)

#### FTC Advertising Guidelines
- Truth in advertising
- Disclosure requirements
- Endorsement guidelines

### 6.2 STATE ALCOHOL LAWS

**Critical: Alcohol law varies by state!**

| Consideration | Implementation |
|---------------|----------------|
| Age Verification | Gate all alcohol content (21+ in US) |
| Shipping Restrictions | Display state-specific rules |
| Dry Counties | Geo-fence content appropriately |
| Three-Tier System | No direct sales without license |

### 6.3 APP COMPLIANCE REQUIREMENTS

```javascript
const complianceRequirements = {
  age_gate: {
    required: true,
    method: 'date_of_birth',
    minimum_age: 21,
    remember_choice: true, // with cookie consent
    bypass_prevention: true
  },
  
  disclaimers: {
    health_warning: "Please drink responsibly. If you or someone you know has a drinking problem, call 1-800-662-4357.",
    legal_disclaimer: "Content is for educational purposes. Check local laws before purchasing.",
    affiliate_disclosure: "Some links may be affiliate links."
  },
  
  content_restrictions: {
    no_targeting_minors: true,
    no_drunk_promotion: true,
    responsible_messaging: true
  }
};
```

### 6.4 COPYRIGHT & CONTENT RIGHTS

| Content Type | Rights Consideration |
|--------------|----------------------|
| Bottle Images | Trademark visible - fair use for reference |
| Historical Images | Verify public domain status |
| User Uploads | Get explicit rights grant |
| API Data | Follow API terms of service |
| Brand Names | Can mention, can't imply endorsement |

---

## 💰 SECTION 7: MONETIZATION STRATEGY

### 7.1 SUBSCRIPTION TIERS

| Tier | Price | Features |
|------|-------|----------|
| **Free** | $0 | Basic collection (50 bottles), limited history, ads |
| **Collector** | $4.99/mo | Unlimited collection, all history, no ads |
| **Enthusiast** | $9.99/mo | + Training courses, premium games |
| **Connoisseur** | $19.99/mo | + Valuation tools, price alerts, API access |
| **Professional** | $49.99/mo | + White-label features, bulk tools |

### 7.2 ADVERTISING OPPORTUNITIES

#### Contextual Advertising
```javascript
const adPlacements = {
  spirit_profile: {
    type: "sponsored_similar",
    description: "Similar bottles from partner distilleries",
    revenue_model: "CPC or CPM"
  },
  
  distillery_page: {
    type: "featured_listing",
    description: "Premium distillery profile",
    revenue_model: "Monthly fee"
  },
  
  cocktail_recipe: {
    type: "ingredient_sponsor",
    description: "Featured brand for recipe ingredient",
    revenue_model: "Per impression"
  },
  
  history_timeline: {
    type: "era_sponsor",
    description: "Brand sponsors a decade/era",
    revenue_model: "Annual sponsorship"
  },
  
  learning_module: {
    type: "educational_sponsor",
    description: "Brand sponsors a course",
    revenue_model: "Per completion"
  }
};
```

### 7.3 AFFILIATE PROGRAMS

| Partner Type | Commission | Notes |
|--------------|------------|-------|
| Online Retailers | 3-8% | Drizly (closed), ReserveBar, etc. |
| Glassware | 5-10% | Riedel, Glencairn affiliate programs |
| Books | 4-8% | Amazon Associates |
| Tours & Experiences | 10-15% | Distillery tours, tasting events |
| Merchandise | 8-12% | Branded items, accessories |

### 7.4 BRAND PARTNERSHIPS

#### Sponsorship Packages
```javascript
const sponsorshipPackages = {
  bronze: {
    price: "$1,000/month",
    features: [
      "Logo on category page",
      "1 featured article/month",
      "Basic analytics"
    ]
  },
  
  silver: {
    price: "$5,000/month",
    features: [
      "Homepage feature rotation",
      "4 featured articles/month",
      "Sponsored game challenge",
      "Email inclusion",
      "Detailed analytics"
    ]
  },
  
  gold: {
    price: "$15,000/month",
    features: [
      "Exclusive category sponsor",
      "Custom landing page",
      "Sponsored learning module",
      "Monthly featured distillery",
      "Full analytics dashboard",
      "Direct API access"
    ]
  }
};
```

---

## 🏆 SECTION 8: COMPETITOR DIFFERENTIATION

### What Distiller/Whiskybase Have (Match This)
- ✅ Large spirits database
- ✅ Tasting notes & reviews
- ✅ Collection tracking
- ✅ Price information
- ✅ User ratings

### What They're Missing (Your Advantage)
- ❌ **Museum-grade historical content** (Prohibition, mob stories, vintage ads)
- ❌ **Structured learning paths** with gamification
- ❌ **No-placeholder image strategy** with legal rights tracking
- ❌ **Games that educate** (not just trivia)
- ❌ **"Missing Image Quest"** community content building
- ❌ **Integrated cocktail intelligence** (what to make with your collection)
- ❌ **Brand partnership platform** (contextual, not spammy)

---

## 🔧 SECTION 9: IMPLEMENTATION CHECKLIST

### Phase 1: Core Data (Week 1-2)
- [ ] Integrate TheCocktailDB for cocktails
- [ ] Integrate OpenBreweryDB for distilleries
- [ ] Build image acquisition pipeline (Wikidata → Openverse → User)
- [ ] Set up rights metadata storage

### Phase 2: Historical Content (Week 3-4)
- [ ] Connect Library of Congress APIs
- [ ] Build Chronicling America search
- [ ] Integrate Smithsonian/Met/NYPL
- [ ] Create "Today in History" feature

### Phase 3: Training System (Week 5-6)
- [ ] Build learning path structure
- [ ] Create Level 1 free content
- [ ] Implement XP and badge system
- [ ] Add quiz engine

### Phase 4: Games Expansion (Week 7-8)
- [ ] Add remaining games from list
- [ ] Implement leaderboards
- [ ] Add daily challenges
- [ ] Create sponsored challenge system

### Phase 5: Monetization (Week 9-10)
- [ ] Implement subscription tiers
- [ ] Set up ad placements
- [ ] Build affiliate link system
- [ ] Create sponsorship dashboard

---

## 📎 APPENDIX: API CODE EXAMPLES

```javascript
// TheCocktailDB - Get random cocktail
const getRandomCocktail = async () => {
  const response = await fetch('https://www.thecocktaildb.com/api/json/v1/1/random.php');
  const data = await response.json();
  return data.drinks[0];
};

// Open Brewery DB - Search breweries
const searchBreweries = async (query) => {
  const response = await fetch(`https://api.openbrewerydb.org/breweries/search?query=${query}`);
  return response.json();
};

// Chronicling America - Search newspapers
const searchHistoricNews = async (term, startYear, endYear) => {
  const response = await fetch(
    `https://chroniclingamerica.loc.gov/search/pages/results/?` +
    `andtext=${encodeURIComponent(term)}&date1=${startYear}&date2=${endYear}&format=json`
  );
  return response.json();
};

// Openverse - Search CC images
const searchOpenImages = async (query, license = 'cc0,by') => {
  const response = await fetch(
    `https://api.openverse.org/v1/images/?q=${encodeURIComponent(query)}&license=${license}`
  );
  return response.json();
};

// Wikidata - Get spirits with images
const getSpiritsWithImages = async () => {
  const query = `
    SELECT ?item ?itemLabel ?image WHERE {
      ?item wdt:P31/wdt:P279* wd:Q156 .
      ?item wdt:P18 ?image .
      SERVICE wikibase:label { bd:serviceParam wikibase:language "en". }
    }
    LIMIT 100
  `;
  
  const response = await fetch(
    `https://query.wikidata.org/sparql?query=${encodeURIComponent(query)}&format=json`
  );
  return response.json();
};
```

---

**Document Version**: 1.0
**Last Updated**: December 22, 2025
**Author**: Claude AI for CR AudioViz AI
**Status**: Ready for Implementation
