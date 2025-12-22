# CRAVCARDS - TRADING CARDS MASTER BLUEPRINT
## Complete API, Asset, Game, Law & Monetization Specification
### Generated: December 22, 2025 | CR AudioViz AI

---

## 🎯 EXECUTIVE SUMMARY

CRAVCards will be the **ultimate trading card collector platform** supporting:
- **Pokémon TCG** - 15,000+ cards
- **Magic: The Gathering** - 25,000+ cards
- **Yu-Gi-Oh!** - 12,000+ cards
- **Sports Cards** - MLB, NFL, NBA, NHL
- **Other TCGs** - Lorcana, Flesh and Blood, One Piece, etc.

This document provides everything needed to build features that surpass CollX, Ludex, and Collectr.

---

## 📊 SECTION 1: CARD DATA APIs

### 1.1 POKÉMON TCG

#### Pokémon TCG API
- **URL**: https://pokemontcg.io/
- **Documentation**: https://docs.pokemontcg.io/
- **Data Available**:
  - Card images (small, large, hi-res)
  - Card metadata (name, HP, types, attacks)
  - Set information
  - Rarity
  - TCGPlayer prices
  - Legality (Standard, Expanded, Unlimited)
- **Endpoints**:
  - `GET /cards` - List cards with filtering
  - `GET /cards/{id}` - Get specific card
  - `GET /sets` - List all sets
  - `GET /sets/{id}` - Get specific set
  - `GET /types` - Card types
  - `GET /subtypes` - Card subtypes
  - `GET /supertypes` - Card supertypes
  - `GET /rarities` - All rarities
- **Authentication**: API key optional (increases rate limits)
- **Rate Limits**: 
  - Without key: 1000 requests/day
  - With key: 20,000 requests/day
- **Use Case**: Full Pokémon card database, collection tracking, pricing

#### TCGdex (Pokémon Alternative)
- **URL**: https://tcgdex.dev/
- **GitHub**: https://github.com/tcgdex
- **Features**:
  - Multi-language support (EN, FR, DE, ES, IT, PT, JP, KO, ZH)
  - REST and GraphQL APIs
  - SDKs for multiple languages
- **Use Case**: International card data, multi-language support

### 1.2 MAGIC: THE GATHERING

#### Scryfall API
- **URL**: https://scryfall.com/docs/api
- **Documentation**: https://scryfall.com/docs/api
- **Data Available**:
  - Card images (all printings)
  - Oracle text (official card text)
  - Prices (USD, EUR, TIX)
  - Legality (all formats)
  - Rulings
  - Set information
  - Artist information
- **Endpoints**:
  - `GET /cards/search?q={query}` - Search cards
  - `GET /cards/named?exact={name}` - Exact name lookup
  - `GET /cards/random` - Random card
  - `GET /cards/{id}` - Get by ID
  - `GET /sets` - All sets
  - `GET /sets/{code}` - Specific set
  - `GET /rulings/{id}` - Card rulings
- **Authentication**: None required
- **Rate Limits**: 10 requests/second, respect delays
- **Use Case**: Complete MTG database, deck building, collection tracking

#### MTGJSON (Bulk Data)
- **URL**: https://mtgjson.com/
- **Documentation**: https://mtgjson.com/getting-started/
- **Features**:
  - Complete downloadable datasets
  - Daily updates
  - Multiple formats (JSON, CSV, SQL, Parquet)
  - Atomic cards (unique cards across printings)
- **Use Case**: Offline access, bulk operations, analytics

### 1.3 YU-GI-OH!

#### YGOPRODeck API
- **URL**: https://ygoprodeck.com/api-guide/
- **API Base**: https://db.ygoprodeck.com/api/v7/
- **Data Available**:
  - Card images
  - Card data (ATK, DEF, Level, Type, etc.)
  - Archetype information
  - Card sets and prices
  - Ban list status
- **Endpoints**:
  - `GET /cardinfo.php` - Get card info
  - `GET /cardinfo.php?name={name}` - By name
  - `GET /cardinfo.php?archetype={archetype}` - By archetype
  - `GET /cardsets.php` - All sets
  - `GET /cardsetsinfo.php?setcode={code}` - Set details
  - `GET /checkDBVer.php` - Database version
- **Authentication**: None required
- **Rate Limits**: 20 requests/second
- **Use Case**: Complete Yu-Gi-Oh database, deck building

### 1.4 MULTI-TCG PRICING

#### TCGPlayer API
- **URL**: https://docs.tcgplayer.com/
- **Coverage**: MTG, Pokémon, Yu-Gi-Oh, Sports, more
- **Data Available**:
  - Market prices
  - Price history
  - Product catalog
  - Inventory data
- **Authentication**: OAuth required
- **Use Case**: Real-time pricing, market data

#### JustTCG
- **URL**: https://justtcg.com/
- **Documentation**: https://justtcg.com/docs
- **Coverage**: MTG, Pokémon, Yu-Gi-Oh, Lorcana, One Piece, Flesh and Blood
- **Features**:
  - Price tracking
  - Market trends
  - Multiple marketplaces
- **Use Case**: Cross-TCG pricing comparisons

### 1.5 SPORTS CARDS

#### CardHedger (Enterprise)
- **URL**: https://www.cardhedger.com/
- **Features**:
  - 1.8M+ sports cards
  - Real-time pricing
  - PSA, BGS, SGC, CGC grades
  - 5 years price history
  - Machine learning datasets
- **Authentication**: API key required
- **Pricing**: Contact for enterprise pricing
- **Use Case**: Sports card pricing, grading data

---

## 🖼️ SECTION 2: IMAGE & ASSET STRATEGY

### 2.1 CARD IMAGES BY SOURCE

| TCG | Image Source | License | Quality |
|-----|--------------|---------|---------|
| Pokémon | pokemontcg.io | Fair use for reference | High |
| MTG | Scryfall | Wizards of the Coast policy | Very High |
| Yu-Gi-Oh | YGOPRODeck | Fair use for reference | High |
| Sports | User upload / Partner feeds | Rights-managed | Varies |

### 2.2 IMAGE RIGHTS CONSIDERATIONS

**Important Legal Notes:**
- Card images contain copyrighted artwork
- Generally allowed for:
  - Collection tracking (personal use)
  - Reference/identification
  - Educational purposes
- Restrictions:
  - Cannot sell prints of card images
  - Cannot create derivative products
  - Must follow API terms of service

### 2.3 USER UPLOAD SYSTEM

```javascript
const cardUploadSystem = {
  allowed_content: {
    own_cards: true,        // Photos of cards you own
    graded_slabs: true,     // PSA, BGS slabs
    collection_shots: true  // Collection displays
  },
  
  moderation: {
    auto_scan: true,        // Check for inappropriate content
    duplicate_detection: true,
    quality_minimum: "500x700px"
  },
  
  rewards: {
    first_upload: 50,       // XP
    verified_photo: 100,    // After moderation
    rare_card: 200          // For missing rare cards
  }
};
```

---

## 🎮 SECTION 3: GAMES & ENGAGEMENT

### 3.1 GAMES BUILT TODAY

| Game | API | Description |
|------|-----|-------------|
| **Pokemon Memory Match** | Pokémon TCG API | Match pairs of real cards |
| **Card Collector Challenge** | Scryfall API | Price guessing, rarity identification |

### 3.2 ADDITIONAL GAMES TO BUILD

| Game | Description | Data Source |
|------|-------------|-------------|
| **Pack Simulator** | Open virtual packs | Set odds data |
| **Price Predictor** | Guess if card will go up/down | Historical prices |
| **Rarity Quiz** | Identify card rarities | All APIs |
| **Set Completion Race** | Compete to complete sets | Collection data |
| **Grading Estimator** | Guess PSA grade from photo | User data |
| **Card Battle** | Turn-based using real stats | Pokémon/MTG/YGO APIs |
| **Market Trader** | Buy low, sell high simulation | Price history |
| **Collection Valuation Game** | Estimate collection value | All pricing APIs |

### 3.3 ENGAGEMENT MECHANICS

```javascript
const collectorEngagement = {
  collection_milestones: {
    first_card: { xp: 50, badge: 'first_pull' },
    complete_set: { xp: 500, badge: 'set_master' },
    collection_100: { xp: 1000, badge: 'centurion' },
    collection_1000: { xp: 5000, badge: 'hoarder' },
    rare_find: { xp: 250, badge: 'lucky_pull' }
  },
  
  daily_activities: {
    price_check: 25,
    add_card: 10,
    complete_trade: 100,
    share_collection: 50
  },
  
  competitive: {
    collection_value_leaderboard: true,
    rarest_card_showcase: true,
    set_completion_races: true,
    trading_volume_ranks: true
  }
};
```

---

## ⚖️ SECTION 4: LEGAL FRAMEWORK

### 4.1 INTELLECTUAL PROPERTY

#### Pokémon Cards
- **Copyright**: The Pokémon Company / Nintendo
- **Trademark**: Pokémon, character names
- **Policy**: Generally allows collection apps with attribution
- **Don't**: Create counterfeit products, sell card images

#### Magic: The Gathering
- **Copyright**: Wizards of the Coast / Hasbro
- **Fan Content Policy**: https://company.wizards.com/en/legal/fancontentpolicy
- **Allowed**: Non-commercial fan content with attribution
- **Don't**: Charge money, imply endorsement

#### Yu-Gi-Oh!
- **Copyright**: Konami
- **Policy**: Fair use for reference/collection tracking
- **Don't**: Distribute card scans for gameplay, counterfeits

#### Sports Cards
- **Copyright**: Card manufacturer + League + Player
- **Complexity**: Multiple rights holders
- **Best Practice**: Focus on metadata, user-uploaded photos

### 4.2 RESALE & TRADING

#### First Sale Doctrine
- You CAN resell cards you legitimately own
- You CAN track and value your collection
- You CAN facilitate trades between users

#### Platform Considerations
- If facilitating sales, consider:
  - Payment processing regulations
  - State sales tax requirements
  - Consumer protection laws
  - Dispute resolution policies

### 4.3 GRADING COMPANY REFERENCES

| Company | Can Reference? | Notes |
|---------|----------------|-------|
| PSA | Yes with attribution | Don't imply endorsement |
| BGS/Beckett | Yes with attribution | Follow trademark guidelines |
| CGC | Yes with attribution | Relatively new to cards |
| SGC | Yes with attribution | Growing presence |

---

## 💰 SECTION 5: MONETIZATION

### 5.1 SUBSCRIPTION TIERS

| Tier | Price | Features |
|------|-------|----------|
| **Free** | $0 | 100 cards, basic tracking, ads |
| **Collector** | $4.99/mo | 1000 cards, price alerts, no ads |
| **Pro** | $9.99/mo | Unlimited cards, portfolio analytics, export |
| **Dealer** | $29.99/mo | Multi-collection, inventory management, API |

### 5.2 AFFILIATE PROGRAMS

#### TCGPlayer Affiliate
- **URL**: https://docs.tcgplayer.com/docs/tcgplayer-affiliate-program
- **Network**: Impact
- **Commission**: Variable by category
- **Integration**: Deep linking to specific cards

#### eBay Partner Network
- **URL**: https://partnernetwork.ebay.com/
- **Commission**: 1-4% depending on category
- **Integration**: Link to card listings

#### Amazon Associates
- **Commission**: 1-4% for collectibles
- **Integration**: Link to sealed products, accessories

### 5.3 PREMIUM FEATURES

```javascript
const premiumFeatures = {
  portfolio_tracking: {
    price: "Pro tier",
    features: [
      "Real-time portfolio value",
      "Gain/loss tracking",
      "Performance charts",
      "Price alerts"
    ]
  },
  
  market_intelligence: {
    price: "Pro tier",
    features: [
      "Price prediction models",
      "Market trend analysis",
      "Hot/cold indicators",
      "Arbitrage opportunities"
    ]
  },
  
  collection_insurance: {
    price: "Add-on $2.99/mo",
    features: [
      "Detailed inventory reports",
      "Photo documentation",
      "Valuation certificates",
      "Insurance claim support"
    ]
  }
};
```

---

## 🏆 SECTION 6: COMPETITOR ANALYSIS

### What CollX/Ludex Have
- ✅ Photo scanning/recognition
- ✅ Price lookup
- ✅ Collection tracking
- ✅ Portfolio value

### What They're Missing (Your Advantage)
- ❌ **Multi-TCG in one app** (not just sports OR TCG)
- ❌ **Games that educate** (not just utility)
- ❌ **Set completion tracking with gamification**
- ❌ **Historical price charts** (long-term trends)
- ❌ **Community trading features**
- ❌ **Integration with spirits/real estate** (CR ecosystem)

---

## 🔧 SECTION 7: IMPLEMENTATION

### Phase 1: Core APIs (Week 1-2)
- [ ] Integrate Pokémon TCG API
- [ ] Integrate Scryfall API
- [ ] Integrate YGOPRODeck API
- [ ] Build unified card schema

### Phase 2: Collection System (Week 3-4)
- [ ] Collection CRUD operations
- [ ] Set completion tracking
- [ ] Portfolio valuation
- [ ] Price alerts

### Phase 3: Games (Week 5-6)
- [ ] Expand game library
- [ ] Pack simulator
- [ ] Leaderboards
- [ ] Daily challenges

### Phase 4: Monetization (Week 7-8)
- [ ] Subscription system
- [ ] Affiliate integration
- [ ] Premium features
- [ ] Ad placements

---

## 📎 APPENDIX: API CODE EXAMPLES

```javascript
// Pokémon TCG - Search cards
const searchPokemonCards = async (name) => {
  const response = await fetch(
    `https://api.pokemontcg.io/v2/cards?q=name:${name}`
  );
  return response.json();
};

// Scryfall - Get MTG card
const getMTGCard = async (name) => {
  const response = await fetch(
    `https://api.scryfall.com/cards/named?exact=${encodeURIComponent(name)}`
  );
  return response.json();
};

// YGOPRODeck - Search Yu-Gi-Oh cards
const searchYugiohCards = async (name) => {
  const response = await fetch(
    `https://db.ygoprodeck.com/api/v7/cardinfo.php?fname=${encodeURIComponent(name)}`
  );
  return response.json();
};

// Get card prices from multiple sources
const getCardPrices = async (cardName, tcg) => {
  const sources = {
    pokemon: async () => {
      const data = await searchPokemonCards(cardName);
      return data.data?.[0]?.tcgplayer?.prices;
    },
    mtg: async () => {
      const data = await getMTGCard(cardName);
      return data.prices;
    },
    yugioh: async () => {
      const data = await searchYugiohCards(cardName);
      return data.data?.[0]?.card_prices;
    }
  };
  
  return sources[tcg]?.() || null;
};
```

---

**Document Version**: 1.0
**Last Updated**: December 22, 2025
**Author**: Claude AI for CR AudioViz AI
**Status**: Ready for Implementation
