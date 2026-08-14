# Developer Onboarding Guide

**Get Up to Speed in 1 Day**  
**Updated:** November 21, 2025 - 12:15 PM EST

---

## Welcome!

This guide will get you productive on the CR AudioViz AI platform in one day.

---

## Prerequisites

### Required Knowledge
- JavaScript/TypeScript
- React/Next.js
- Git/GitHub
- REST APIs
- PostgreSQL basics

### Required Tools
- Node.js 18+
- Git
- VSCode (recommended)
- GitHub account
- Supabase CLI

---

## Day 1: Setup

### Hour 1: Environment Setup

**Clone Repositories:**
```bash
# Main platform
git clone https://github.com/CR-AudioViz-AI/craudiovizai-com.git

# Javari AI
git clone https://github.com/CR-AudioViz-AI/javari-javari-ai.git

# Admin Dashboard
git clone https://github.com/CR-AudioViz-AI/javari-admin-dashboard.git
```

**Install Dependencies:**
```bash
cd craudiovizai-com
npm install

cd ../javari-javari-ai
npm install

cd ../javari-admin-dashboard
npm install
```

**Environment Variables:**
Contact Roy for `.env.local` file with all credentials.

### Hour 2: Architecture Overview

**Read These Docs:**
1. MASTER_SUMMARY.md (30 min)
2. DEPLOYMENT_PLAYBOOK.md (30 min)

**Key Concepts:**
- Next.js 14 App Router
- Supabase backend
- TypeScript strict mode
- Vercel deployment
- Multi-AI routing

### Hour 3: Database Familiarity

**Supabase Dashboard:**
- Login: kteobfyferrukqeolofj.supabase.co
- Explore tables
- Review RLS policies
- Check functions

**Key Tables:**
- `profiles` - User data
- `credits_transactions` - Billing
- `tool_usage` - Analytics
- `ai_conversations` - Chat history

### Hour 4: First Contribution

**Starter Task:**
Pick one of these simple tasks:
1. Fix a TypeScript error
2. Add a new page
3. Update a component
4. Write a test
5. Improve documentation

**Process:**
```bash
git checkout -b feature/your-task
# Make changes
git add .
git commit -m "Description"
git push origin feature/your-task
# Create Pull Request
```

---

## Day 2-7: Deep Dive

### Key Areas to Learn

**Frontend (React/Next.js):**
- App Router structure
- Server Components
- Client Components
- Streaming responses
- Error handling

**Backend (Supabase):**
- RLS policies
- Edge Functions
- Real-time subscriptions
- Storage buckets
- Auth flows

**AI Integration:**
- OpenAI API
- Anthropic Claude
- Google Gemini
- Perplexity API
- Cost optimization

**Deployment (Vercel):**
- Preview builds
- Production deploys
- Environment variables
- Monitoring
- Analytics

---

## Code Standards

### TypeScript
```typescript
// Always use strict types
interface User {
  id: string;
  email: string;
  credits: number;
}

// No 'any' types
❌ const data: any = await fetch()
✅ const data: User = await fetch()
```

### React Components
```typescript
// Use TypeScript
export function MyComponent({ title }: { title: string }) {
  return <div>{title}</div>
}

// Not JavaScript
export function MyComponent({ title }) {
  return <div>{title}</div>
}
```

### Error Handling
```typescript
try {
  const result = await riskyOperation()
  return result
} catch (error) {
  console.error('Operation failed:', error)
  throw new Error('User-friendly message')
}
```

---

## Common Tasks

### Adding a New Tool

1. Create tool directory:
```bash
mkdir -p app/tools/my-tool
```

2. Create page:
```typescript
// app/tools/my-tool/page.tsx
export default function MyToolPage() {
  return <div>My Tool</div>
}
```

3. Add to tools registry
4. Create API route if needed
5. Add tests
6. Document

### Adding API Endpoint

```typescript
// app/api/my-endpoint/route.ts
import { NextRequest, NextResponse } from 'next/server'

export async function POST(request: NextRequest) {
  try {
    const body = await request.json()
    // Process
    return NextResponse.json({ success: true })
  } catch (error) {
    return NextResponse.json(
      { error: 'Failed' },
      { status: 500 }
    )
  }
}
```

### Database Query

```typescript
import { createClient } from '@/lib/supabase/server'

const supabase = createClient()

const { data, error } = await supabase
  .from('profiles')
  .select('*')
  .eq('id', userId)
  .single()

if (error) throw error
return data
```

---

## Deployment

### Development
```bash
npm run dev
# Opens localhost:3000
```

### Preview (Vercel)
```bash
git push origin feature-branch
# Vercel auto-deploys preview
# Check PR for preview link
```

### Production
```bash
# Never push to main directly
# Always use Pull Requests
# Roy/Cindy merge to main
# Auto-deploys to production
```

---

## Getting Help

### Resources
- Documentation: /docs folder
- API Reference: API_DOCUMENTATION.md
- Troubleshooting: TROUBLESHOOTING_GUIDE.md

### Team
- Roy (CEO): Slack @roy
- Lead Dev: Slack @[name]
- Team Channel: #developers

### Support
- Questions: #dev-questions
- Bugs: GitHub Issues
- Ideas: #feature-ideas

---

## Checklist

**Week 1:**
- [ ] Environment setup
- [ ] Read core docs
- [ ] Explore codebase
- [ ] First PR merged
- [ ] Understand deployment

**Week 2:**
- [ ] Complete assigned tasks
- [ ] Review team PRs
- [ ] Contribute to docs
- [ ] Attend team meetings

**Month 1:**
- [ ] Lead feature development
- [ ] Mentor new devs
- [ ] Improve processes
- [ ] Full autonomy

---

**Welcome to the team! Let's build something amazing! 🚀**
