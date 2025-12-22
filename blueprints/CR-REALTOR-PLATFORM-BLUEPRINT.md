# CR-REALTOR PLATFORM - MASTER BLUEPRINT
## Complete Asset, API, Law & Feature Specification
### Generated: December 22, 2025 | CR AudioViz AI

---

## 🎯 EXECUTIVE SUMMARY

This document provides everything needed to build a world-class real estate intelligence platform that surpasses Zillow, Redfin, and Realtor.com through:
- **Deeper data layers** (flood, disaster, environmental, lending, schools)
- **User-controlled feature toggles** (show what YOU want)
- **Educational content** (not just data - explanations)
- **Games & engagement** (property simulations, quizzes)
- **Monetization** (affiliate, premium tiers, leads)

---

## 📊 SECTION 1: FREE DATA APIs & SOURCES

### 1.1 DEMOGRAPHIC & HOUSING FUNDAMENTALS

#### US Census Bureau API
- **URL**: https://api.census.gov/data.html
- **Documentation**: https://www.census.gov/data/developers.html
- **Key Datasets**:
  - American Community Survey (ACS) - housing characteristics
  - Decennial Census - population
  - Building Permits Survey
- **Data Available**:
  - Population by ZIP/county/tract
  - Median household income
  - Home ownership rates
  - Median home values (self-reported)
  - Housing age, size, density
  - Rent vs. ownership ratios
- **Authentication**: API key required (free)
- **Rate Limits**: 500 requests/day without key, more with key
- **Use Case**: Neighborhood profiles, affordability scoring, market demand indicators

#### FHFA House Price Index API
- **URL**: https://www.fhfa.gov/DataTools/Downloads
- **Documentation**: https://www.fhfa.gov/data/hpi
- **Data Available**:
  - Quarterly price changes by metro/state/nation
  - Historical trends (1991-present)
  - Repeat-sale price indices
- **Authentication**: None required
- **Format**: CSV downloads, some APIs available
- **Use Case**: Market trend charts, "Is this area rising or declining?" features

### 1.2 RENTAL MARKET DATA

#### HUD Fair Market Rent (FMR) API
- **URL**: https://www.huduser.gov/portal/dataset/fmr-api.html
- **Documentation**: https://www.huduser.gov/portal/dataset/fmr-api.html
- **Data Available**:
  - Fair Market Rents by county/metro
  - Rent by bedroom count (0BR-4BR)
  - Small Area FMRs (ZIP level)
- **Authentication**: None required
- **Use Case**: Rent affordability tools, "Can you afford to live here?"

#### HUD Income Limits API
- **URL**: https://www.huduser.gov/portal/dataset/il.html
- **Data Available**:
  - Area Median Income (AMI)
  - Income limits by household size
- **Use Case**: Housing assistance eligibility, affordability calculations

### 1.3 FLOOD & DISASTER RISK

#### FEMA National Flood Hazard Layer (NFHL)
- **URL**: https://www.fema.gov/flood-maps/national-flood-hazard-layer
- **GIS Services**: https://hazards.fema.gov/femaportal/resources/flood_map_svc.htm
- **ArcGIS REST**: https://www.arcgis.com/home/item.html?id=d4e50201710144e8bff1909dab469250
- **Data Available**:
  - Flood zones (A, AE, X, etc.)
  - Base Flood Elevations
  - Floodway boundaries
  - Community panel information
- **Authentication**: None required
- **Use Case**: "Flood Risk Card" per property with zone, risk type, map overlay

#### OpenFEMA API
- **URL**: https://www.fema.gov/about/openfema/api
- **Documentation**: https://www.fema.gov/about/openfema/data-sets
- **Key Endpoints**:
  - `/DisasterDeclarationsSummaries` - declared disasters
  - `/FemaWebDisasterSummaries` - disaster details
  - `/HousingAssistanceOwners` - assistance data
  - `/PublicAssistanceFundedProjectsDetails`
- **Data Available**:
  - Disaster declarations by county/state
  - Disaster types (hurricane, flood, fire, etc.)
  - Declaration dates and incident periods
  - Assistance amounts
- **Authentication**: None required
- **Rate Limits**: Standard API limits apply
- **Use Case**: County disaster timeline, "Has this area been hit?" feature

### 1.4 ENVIRONMENTAL HAZARDS

#### EPA Envirofacts API
- **URL**: https://www.epa.gov/enviro/envirofacts-data-service-api
- **Documentation**: https://www.epa.gov/enviro/web-services
- **Key Tables**:
  - CERCLIS - Superfund sites
  - RCRAINFO - Hazardous waste
  - SDWIS - Drinking water
  - AIR_FACILITY - Air quality
  - TRI - Toxic Release Inventory
- **Data Available**:
  - Facility locations and types
  - Violation history
  - Permit information
  - Chemical releases
- **Query Format**: REST with table/column filtering
- **Authentication**: None required
- **Use Case**: "What's near this address?" environmental proximity scan

### 1.5 GEOLOGICAL HAZARDS

#### USGS Earthquake Catalog API
- **URL**: https://earthquake.usgs.gov/fdsnws/event/1/
- **Documentation**: https://earthquake.usgs.gov/fdsnws/event/1/
- **Parameters**:
  - starttime/endtime
  - minlatitude/maxlatitude
  - minlongitude/maxlongitude
  - minmagnitude/maxmagnitude
  - maxradiuskm (search radius)
- **Data Available**:
  - Earthquake events with magnitude, depth, location
  - Historical catalog (1900s-present)
  - Real-time updates
- **Authentication**: None required
- **Format**: GeoJSON, CSV, KML
- **Use Case**: "Quake history within X miles," seismic risk overlays

### 1.6 LENDING & FINANCING INTELLIGENCE

#### HMDA Data Browser API
- **URL**: https://ffiec.cfpb.gov/documentation/api/data-browser/
- **Documentation**: https://ffiec.cfpb.gov/documentation/
- **Data Available**:
  - Loan applications by location
  - Approval/denial rates
  - Loan types (conventional, FHA, VA)
  - Lender presence by area
  - Borrower demographics
- **Query Options**:
  - By state, county, census tract
  - By year (2018-present)
  - By loan purpose, type, action
- **Authentication**: None required
- **Use Case**: "Lending patterns" heatmaps, financing difficulty indicators

#### FRED (Federal Reserve Economic Data)
- **URL**: https://fred.stlouisfed.org/docs/api/fred/
- **Documentation**: https://fred.stlouisfed.org/docs/api/
- **Key Series**:
  - MORTGAGE30US - 30-year mortgage rate
  - MORTGAGE15US - 15-year mortgage rate
  - CSUSHPISA - Case-Shiller Home Price Index
  - UNRATE - Unemployment rate
  - CPIAUCSL - Consumer Price Index
- **Authentication**: API key required (free registration)
- **Use Case**: Mortgage calculators, market timing tools, economic context

### 1.7 WEATHER & CLIMATE

#### National Weather Service API
- **URL**: https://api.weather.gov/
- **Documentation**: https://www.weather.gov/documentation/services-web-api
- **Endpoints**:
  - `/points/{lat},{lon}` - get forecast office
  - `/gridpoints/{office}/{x},{y}/forecast` - 7-day forecast
  - `/alerts/active` - active weather alerts
- **Data Available**:
  - Forecasts (hourly, daily, extended)
  - Active alerts and warnings
  - Historical observations
- **Authentication**: None required (User-Agent header recommended)
- **Use Case**: "Storm Mode" for saved properties, weather alerts

### 1.8 EDUCATION DATA

#### NCES EDGE Open Data
- **URL**: https://data-nces.opendata.arcgis.com/
- **API Documentation**: https://data-nces.opendata.arcgis.com/pages/use-apis
- **Data Available**:
  - School locations (public, private, charter)
  - District boundaries
  - Enrollment numbers
  - School characteristics
- **Format**: ArcGIS REST services, GeoJSON
- **Use Case**: School proximity maps, "family friendliness" views

### 1.9 GEOCODING & MAPPING

#### Nominatim (OpenStreetMap)
- **URL**: https://nominatim.org/release-docs/latest/api/Overview/
- **Usage Policy**: https://operations.osmfoundation.org/policies/nominatim/
- **Endpoints**:
  - `/search` - address to coordinates
  - `/reverse` - coordinates to address
- **Rate Limits**: 1 request/second, no bulk use
- **Use Case**: Address lookup to trigger all other API calls

#### OpenStreetMap Data
- **URL**: https://www.openstreetmap.org/
- **API**: https://wiki.openstreetmap.org/wiki/API
- **Data Available**:
  - Road networks
  - Points of interest
  - Land use
  - Building footprints
- **Use Case**: Base maps, POI overlays, walkability calculations

---

## ⚖️ SECTION 2: LEGAL & REGULATORY FRAMEWORK

### 2.1 FLORIDA REAL ESTATE LAW (PRIMARY MARKET)

#### Florida Statutes - Real Property
- **Source**: https://www.leg.state.fl.us/statutes/
- **Key Chapters**:

| Chapter | Title | Relevance |
|---------|-------|-----------|
| 83 | Landlord & Tenant | Rental rights, evictions, deposits |
| 475 | Real Estate Brokers | Licensing, commission rules |
| 689 | Conveyances of Land | Property transfers |
| 695 | Record of Conveyances | Recording requirements |
| 718 | Condominiums | Condo regulations |
| 719 | Cooperatives | Co-op regulations |
| 720 | Homeowners' Associations | HOA rules |
| 723 | Mobile Home Parks | Mobile home regulations |

#### Florida Disclosure Requirements
- **Lead-based paint** (pre-1978 homes)
- **Property condition defects** (known material defects)
- **Flood zone status** (expanded in recent years)
- **HOA/COA rules and fees**
- **Coastal Construction Control Line** (coastal properties)

### 2.2 FEDERAL REAL ESTATE LAW

#### Fair Housing Act
- **Source**: https://www.hud.gov/program_offices/fair_housing_equal_opp
- **Protected Classes**: Race, color, religion, sex, national origin, disability, familial status
- **App Implications**:
  - No discriminatory filtering
  - Equal display of properties
  - Accessible design (ADA compliance)
  - Content moderation for user submissions

#### Truth in Lending Act (TILA)
- **Source**: https://www.consumerfinance.gov/
- **App Implications**:
  - APR calculations must be accurate
  - Disclosure of loan terms
  - Clear mortgage comparisons

#### RESPA (Real Estate Settlement Procedures Act)
- **App Implications**:
  - Disclosure of settlement costs
  - No kickbacks for referrals
  - Clear fee explanations

### 2.3 PROPERTY TYPE REGULATIONS

| Property Type | Key Regulations |
|---------------|-----------------|
| Residential | Fair Housing, State landlord-tenant law, Habitability |
| Commercial | Contract law, Zoning, ADA compliance, Environmental |
| Industrial | EPA regulations, OSHA, Zoning, Environmental liability |

---

## 🚀 SECTION 3: FEATURE SPECIFICATIONS

### 3.1 PROPERTY INTELLIGENCE CARD

The signature feature that makes this app "blow minds."

```
┌─────────────────────────────────────────────────────────────┐
│  PROPERTY INTELLIGENCE CARD                                  │
│  123 Main Street, Miami, FL 33101                           │
├─────────────────────────────────────────────────────────────┤
│  [Toggle Panels - User Controls What They See]              │
│                                                             │
│  ☑ Flood Risk      ☑ Disasters    ☑ Environmental          │
│  ☑ Earthquakes     ☑ Weather      ☑ Lending                 │
│  ☑ Schools         ☑ Demographics ☑ Market Trends          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  🌊 FLOOD RISK                                    [Expand]  │
│  Zone: AE (High Risk)                                       │
│  Base Flood Elevation: 9 ft                                 │
│  Insurance Required: Yes (if federally backed mortgage)     │
│                                                             │
│  🌪️ DISASTER HISTORY                              [Expand]  │
│  Last 10 years: 3 hurricanes, 2 flood events               │
│  Most recent: Hurricane Ian (2022)                          │
│                                                             │
│  🏭 ENVIRONMENTAL                                 [Expand]  │
│  EPA Sites within 5 miles: 2                               │
│  Nearest Superfund: 8.3 miles                              │
│                                                             │
│  📊 LENDING PATTERNS                              [Expand]  │
│  Avg Approval Rate: 72%                                    │
│  Common Loan Type: Conventional (65%)                      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 FEATURE MODULES TO BUILD

| Module | Data Sources | Priority |
|--------|--------------|----------|
| Property Search | Geocoding, Parcel APIs | 🔴 P0 |
| Flood Risk Report | FEMA NFHL, OpenFEMA | 🔴 P0 |
| Disaster Timeline | OpenFEMA | 🔴 P0 |
| Environmental Scan | EPA Envirofacts | 🟡 P1 |
| Market Trends | FHFA, Census | 🟡 P1 |
| Lending Intelligence | HMDA | 🟡 P1 |
| School Finder | NCES EDGE | 🟡 P1 |
| Weather Alerts | NWS API | 🟢 P2 |
| Earthquake History | USGS | 🟢 P2 |
| Mortgage Calculator | FRED + custom | 🟢 P2 |

### 3.3 GAMES & ENGAGEMENT

| Game | Description | Educational Value |
|------|-------------|-------------------|
| Flip Simulator | Buy, renovate, sell properties | ROI calculations |
| Neighborhood Quiz | Guess stats about areas | Market knowledge |
| Zoning Puzzles | Match properties to zones | Zoning education |
| Disaster Prep | Emergency planning game | Safety awareness |
| Market Predictor | Guess price trends | Market analysis |

---

## 💰 SECTION 4: MONETIZATION STRATEGY

### 4.1 SUBSCRIPTION TIERS

| Tier | Price | Features |
|------|-------|----------|
| Free | $0 | Basic search, 5 property views/day |
| Pro | $9.99/mo | Unlimited views, all data layers, alerts |
| Investor | $29.99/mo | Portfolio tracking, bulk analysis, API access |
| Agent | $49.99/mo | Lead generation, CRM integration, branding |

### 4.2 AFFILIATE PARTNERSHIPS

| Partner Type | Commission Model |
|--------------|------------------|
| Mortgage Lenders | $50-200 per qualified lead |
| Home Insurance | $20-50 per policy |
| Home Warranty | $30-100 per sale |
| Moving Services | 5-10% of service value |
| Home Improvement | 3-8% of referred sales |

### 4.3 ADVERTISING

- **Contextual Ads**: Show relevant service providers based on property data
- **Sponsored Listings**: Featured properties in search results
- **Local Services**: Contractors, inspectors, agents by area

---

## 🏆 SECTION 5: COMPETITOR DIFFERENTIATION

### What Zillow/Redfin Have (Match This)
- ✅ Property listings
- ✅ Map search
- ✅ Price estimates
- ✅ Saved searches
- ✅ Agent connections

### What They're Missing (Your Advantage)
- ❌ **Transparent risk data** (flood, disaster, environmental with sources)
- ❌ **User-controlled data layers** (toggle what you see)
- ❌ **Lending pattern intelligence** (HMDA data insights)
- ❌ **Educational content** (explain WHY data matters)
- ❌ **Gamification** (learn while exploring)
- ❌ **Historical context** (neighborhood evolution)

---

## 🔧 SECTION 6: IMPLEMENTATION CHECKLIST

### Phase 1: Core Infrastructure (Week 1-2)
- [ ] Set up API connections (Census, FEMA, EPA, USGS, NWS)
- [ ] Build geocoding service
- [ ] Create property data aggregation layer
- [ ] Design Property Intelligence Card UI

### Phase 2: Data Layers (Week 3-4)
- [ ] Flood risk module
- [ ] Disaster history module
- [ ] Environmental scan module
- [ ] Market trends module

### Phase 3: Intelligence Features (Week 5-6)
- [ ] Lending patterns module
- [ ] School finder module
- [ ] Mortgage calculator
- [ ] Alert system

### Phase 4: Engagement (Week 7-8)
- [ ] Games integration
- [ ] Achievement system
- [ ] User profiles
- [ ] Social features

### Phase 5: Monetization (Week 9-10)
- [ ] Subscription system
- [ ] Affiliate integrations
- [ ] Ad platform
- [ ] Agent tools

---

## 📎 APPENDIX: API QUICK REFERENCE

```javascript
// Example: Fetch flood zone for coordinates
const getFloodZone = async (lat, lon) => {
  const response = await fetch(
    `https://hazards.fema.gov/gis/nfhl/rest/services/public/NFHL/MapServer/28/query?` +
    `geometry=${lon},${lat}&geometryType=esriGeometryPoint&inSR=4326&` +
    `spatialRel=esriSpatialRelIntersects&outFields=*&returnGeometry=false&f=json`
  );
  return response.json();
};

// Example: Fetch disaster declarations for county
const getDisasters = async (fipsCode) => {
  const response = await fetch(
    `https://www.fema.gov/api/open/v2/DisasterDeclarationsSummaries?` +
    `$filter=fipsCountyCode eq '${fipsCode}'&$orderby=declarationDate desc`
  );
  return response.json();
};

// Example: Fetch earthquake history
const getEarthquakes = async (lat, lon, radiusKm = 100, years = 10) => {
  const endDate = new Date().toISOString().split('T')[0];
  const startDate = new Date(Date.now() - years * 365 * 24 * 60 * 60 * 1000)
    .toISOString().split('T')[0];
  
  const response = await fetch(
    `https://earthquake.usgs.gov/fdsnws/event/1/query?format=geojson&` +
    `starttime=${startDate}&endtime=${endDate}&latitude=${lat}&longitude=${lon}&` +
    `maxradiuskm=${radiusKm}&minmagnitude=2.5`
  );
  return response.json();
};
```

---

**Document Version**: 1.0
**Last Updated**: December 22, 2025
**Author**: Claude AI for CR AudioViz AI
**Status**: Ready for Implementation
