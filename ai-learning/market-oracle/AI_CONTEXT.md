# MARKET ORACLE - AI LEARNING CONTEXT
**For: Javari AI, JAVARI, Kairo, Pulse, Scout**

## 🎯 App Purpose
Market Oracle is an AI battle platform where 5 different AIs compete to pick the best-performing penny stocks.
It's designed to be entertaining, educational, and demonstrate CR AudioViz AI's multi-AI orchestration capabilities.

## 🏗️ Technical Architecture
- **Built with:** Next.js 14, TypeScript, Tailwind CSS
- **Database:** Supabase PostgreSQL
- **Table:** ai_stock_picks (106 current records)
- **APIs:** Next.js API routes
- **Integrations:** OpenAI, Anthropic, Google, Perplexity APIs

## 💡 Key Concepts

### Concept 1: AI Battle
Each of 5 AIs independently analyzes the penny stock market and picks their top stocks.
Users can watch the competition unfold in real-time and see which AI performs best.

### Concept 2: Consensus Picks
When multiple AIs pick the same stock, it's shown as a "Hot Pick" - indicating stronger confidence
across different AI systems.

### Concept 3: Paper Trading
Users can create virtual portfolios to practice trading without real money, learning how to
evaluate AI recommendations.

## 🔄 User Workflows

### Workflow 1: View AI Picks
```
User visits dashboard → Sees 106 picks from 5 AIs → Filters by AI/confidence → Clicks pick → Reads AI reasoning
```

### Workflow 2: Paper Trading
```
User clicks "Add to Portfolio" → Enters quantity → Tracks performance → Compares vs AI performance
```

### Workflow 3: Community Voting
```
User navigates to Voting → Predicts which AI will win → Submits vote → Sees community consensus
```

## 🤖 AI Interactions

### When users ask about Market Oracle:
- **Explain:** "Market Oracle is where 5 AIs battle to pick the best stocks. It's like fantasy sports for stock picking!"
- **Suggest:** "Check out the Hot Picks page to see which stocks multiple AIs agree on"
- **Warn:** "This is for entertainment and education - not financial advice. Always do your own research."

### Code Examples
```typescript
// Fetching AI picks
const { data: picks } = await supabase
  .from('ai_stock_picks')
  .select('*')
  .order('confidence_score', { ascending: false })

// Filtering by AI
const javariPicks = picks.filter(p => p.ai_name === 'Javari AI')
```

## 📊 Data Patterns
- **Typical usage:** Users browse picks, filter by AI, read reasoning, add to watchlist
- **Common errors:** None currently - stable platform
- **Best practices:** Always show Entry | Current | Target prices together

## 🔍 Search Keywords
Market Oracle, AI stock picks, penny stocks, stock battle, AI competition, Javari AI picks,
GPT-4 stocks, Claude stocks, Gemini stocks, Perplexity stocks, paper trading, virtual portfolio

## 📝 Learning Updates
- **Last trained:** November 8, 2025
- **Training data:** 106 real AI picks, user interaction patterns, market data
- **Platform status:** 86% complete, actively being improved
