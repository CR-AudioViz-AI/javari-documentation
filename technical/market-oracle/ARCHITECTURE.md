# MARKET ORACLE - TECHNICAL ARCHITECTURE
**For: Development Team**
**Repository:** market-oracle-app

## 🏗️ Technology Stack

### Frontend
- **Framework:** Next.js 14 (App Router)
- **Language:** TypeScript (strict mode)
- **Styling:** Tailwind CSS
- **Components:** shadcn/ui
- **Icons:** Lucide React

### Backend
- **API Routes:** Next.js API routes
- **Database:** Supabase (PostgreSQL)
- **Real-time:** Supabase subscriptions
- **Authentication:** Supabase Auth (ready, not enabled)

### AI Providers
1. **OpenAI GPT-4** - Primary analysis
2. **Anthropic Claude** - Secondary analysis  
3. **Google Gemini** - Technical analysis
4. **Perplexity** - News-based picks
5. **Javari AI** - Custom analysis

### Hosting
- **Platform:** Vercel
- **Domain:** javari-market-oracle.vercel.app
- **Environment:** Preview deployments only (cost control)

## 📁 Repository Structure
```
market-oracle-app/
├── app/
│   ├── page.tsx              # Dashboard
│   ├── hot-picks/page.tsx    # Consensus picks
│   ├── portfolio/page.tsx    # User positions
│   ├── insights/page.tsx     # AI reasoning
│   ├── backtesting/page.tsx  # Performance
│   ├── sectors/page.tsx      # Industry breakdown
│   ├── paper-trading/page.tsx# Virtual trading
│   ├── voting/page.tsx       # Community votes
│   ├── alerts/page.tsx       # Price alerts
│   ├── community/page.tsx    # Chat rooms
│   ├── learn/page.tsx        # Education
│   ├── charts/page.tsx       # Technical charts
│   ├── export/page.tsx       # Data export
│   └── watchlist/page.tsx    # Favorites
├── components/
│   ├── Navigation.tsx        # Main nav
│   ├── AIConsensusTracker.tsx
│   ├── StockCard.tsx
│   └── [20+ more components]
├── lib/
│   └── supabase.ts          # DB client
└── public/
    └── market-oracle-logo.png
```

## 💾 Database Schema

### Table: ai_stock_picks
```sql
CREATE TABLE ai_stock_picks (
  id UUID PRIMARY KEY,
  ticker TEXT NOT NULL,
  ai_name TEXT NOT NULL,
  price DECIMAL,           -- Entry price
  current_price DECIMAL,   -- Current price
  target_price DECIMAL,    -- Target price
  confidence_score INTEGER,
  reasoning TEXT,
  picked_at TIMESTAMP,
  created_at TIMESTAMP
);
```

**Current data:** 106 real picks

## 🔌 API Endpoints

### Stock Picks
- `GET /api/picks` - Get all AI picks
- `POST /api/picks` - Create new pick (admin)

### Price Updates
- `GET /api/update-prices` - Fetch current prices
- Runs every 15 minutes via cron

### Community
- `POST /api/vote` - Submit vote
- `GET /api/votes` - Get voting results

## 🚀 Deployment

### Local Development
```bash
git clone https://github.com/CR-AudioViz-AI/market-oracle-app
cd market-oracle-app
npm install
npm run dev
```

### Environment Variables
```env
NEXT_PUBLIC_SUPABASE_URL=https://kteobfyferrukqeolofj.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=[key]
```

### Production Deploy
Automatic via Vercel when pushing to main branch.

## 🧪 Testing
- Manual testing on each page
- Database queries tested in Supabase UI
- No automated tests yet (add in Phase 2)

## 📊 Performance
- Page load: < 1s
- API response: < 200ms
- Database queries: < 50ms
- Real-time updates: Every 30s

## 🔐 Security
- Supabase RLS policies
- API rate limiting (planned)
- Input validation
- SQL injection protection (Supabase handles)
