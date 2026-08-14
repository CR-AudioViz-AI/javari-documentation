# DEPLOYMENT PROCEDURES - COMPLETE GUIDE
**CR AudioViz AI Platform Deployment**

**Last Updated:** November 21, 2025 - 4:32 PM EST  
**For:** Developers, DevOps, System Administrators  
**Platform:** Vercel + GitHub + Supabase

---

## QUICK START

**Deploy New App in 5 Minutes:**
1. Create GitHub repository
2. Push Next.js code
3. Import to Vercel
4. Set environment variables
5. Deploy

**Deploy Main Website:**
1. Fix production issues (if any)
2. Push to main branch
3. Vercel auto-deploys
4. Verify at craudiovizai.com

---

## TABLE OF CONTENTS

1. [GitHub Setup](#github-setup)
2. [Vercel Deployment](#vercel-deployment)
3. [Environment Variables](#environment-variables)
4. [Database Setup](#database-setup)
5. [Domain Configuration](#domain-configuration)
6. [Troubleshooting](#troubleshooting)
7. [Rollback Procedures](#rollback-procedures)
8. [Monitoring](#monitoring)

---

## GITHUB SETUP

### Create New Repository

**Via GitHub API (Automated):**
```bash
curl -X POST https://api.github.com/orgs/CR-AudioViz-AI/repos \
  -H "Authorization: token [REDACTED_GITHUB_TOKEN]" \
  -d '{
    "name": "javari-new-app",
    "description": "New application",
    "private": false,
    "auto_init": true
  }'
```

**Via GitHub Website:**
1. Go to https://github.com/CR-AudioViz-AI
2. Click "New repository"
3. Name: javari-[app-name]
4. Public/Private: Public
5. Initialize with README: Yes
6. Click "Create repository"

### Initialize Local Repository

```bash
# Clone repository
git clone https://github.com/CR-AudioViz-AI/javari-new-app.git
cd javari-new-app

# Add Next.js project
npx create-next-app@latest . --typescript --tailwind --app --no-src-dir

# Commit
git add .
git commit -m "Initial Next.js setup"
git push origin main
```

### Branch Strategy

**Main Branch:**
- Production-ready code only
- Auto-deploys to production
- Protected branch (requires review)

**Development Branch:**
- Active development
- Auto-deploys to preview
- Merge to main when ready

**Feature Branches:**
- feature/[name]
- Merge to development
- Delete after merge

---

## VERCEL DEPLOYMENT

### Import Project

**Via Vercel Website:**
1. Go to https://vercel.com
2. Click "Add New..." → "Project"
3. Import Git Repository
4. Select CR-AudioViz-AI/javari-new-app
5. Configure:
   - Framework Preset: Next.js
   - Root Directory: ./
   - Build Command: npm run build
   - Output Directory: .next
6. Click "Deploy"

**Via Vercel API:**
```bash
curl -X POST https://api.vercel.com/v13/deployments \
  -H "Authorization: Bearer [REDACTED_TOKEN][REDACTED_VERCEL_TOKEN]" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "javari-new-app",
    "gitSource": {
      "type": "github",
      "repo": "CR-AudioViz-AI/javari-new-app",
      "ref": "main"
    },
    "target": "production"
  }'
```

### Deployment Settings

**vercel.json Configuration:**
```json
{
  "version": 2,
  "buildCommand": "npm run build",
  "devCommand": "npm run dev",
  "installCommand": "npm install",
  "framework": "nextjs",
  "outputDirectory": ".next",
  "regions": ["iad1"],
  "env": {
    "NEXT_PUBLIC_APP_NAME": "CR AudioViz AI"
  },
  "functions": {
    "api/**/*.ts": {
      "memory": 1024,
      "maxDuration": 10
    }
  }
}
```

### Preview vs Production

**Preview Deployments (Recommended):**
```json
{
  "github": {
    "enabled": true,
    "autoAlias": false,
    "silent": false,
    "autoJobCancelation": true
  }
}
```

**Benefits:**
- Deploy every push to preview URL
- Test before promoting to production
- Saves deployment credits
- Manual promotion to production

**Enable in Vercel:**
1. Project Settings → Git
2. Production Branch: main
3. Preview Deployments: All branches
4. Auto-Alias: OFF (saves credits)

---

## ENVIRONMENT VARIABLES

### Required Variables

**Core Platform (craudiovizai.com):**
```bash
# Database
NEXT_PUBLIC_SUPABASE_URL=https://kteobfyferrukqeolofj.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=[anon_key]
SUPABASE_SERVICE_ROLE_KEY=[service_key]

# Authentication
NEXTAUTH_SECRET=[random_string]
NEXTAUTH_URL=https://craudiovizai.com

# Payments
STRIPE_SECRET_KEY=[sk_live_...]
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=[pk_live_...]
STRIPE_WEBHOOK_SECRET=[whsec_...]
PAYPAL_CLIENT_ID=[client_id]
PAYPAL_SECRET=[secret]

# AI Providers
OPENAI_API_KEY=[sk-proj-...]
ANTHROPIC_API_KEY=[sk-ant-...]
GEMINI_API_KEY=[AIzaSy...]
PERPLEXITY_API_KEY=[pplx-...]

# Infrastructure
GITHUB_TOKEN=[[REDACTED_GITHUB_TOKEN]...]
VERCEL_TOKEN=[token]
```

**Individual Apps:**
```bash
# Minimum required
NEXT_PUBLIC_SUPABASE_URL=[url]
NEXT_PUBLIC_SUPABASE_ANON_KEY=[key]

# If uses AI
OPENAI_API_KEY=[key]
# or
ANTHROPIC_API_KEY=[key]

# If uses payments
STRIPE_SECRET_KEY=[key]
```

### Set Variables in Vercel

**Via Website:**
1. Project → Settings → Environment Variables
2. Click "Add"
3. Key: NEXT_PUBLIC_SUPABASE_URL
4. Value: [paste value]
5. Environments: ✓ Production ✓ Preview ✓ Development
6. Click "Save"

**Via API:**
```bash
curl -X POST "https://api.vercel.com/v10/projects/[project-id]/env" \
  -H "Authorization: Bearer [REDACTED_TOKEN][REDACTED_VERCEL_TOKEN]" \
  -d '{
    "key": "NEXT_PUBLIC_SUPABASE_URL",
    "value": "https://kteobfyferrukqeolofj.supabase.co",
    "type": "encrypted",
    "target": ["production", "preview", "development"]
  }'
```

**Via Vercel CLI:**
```bash
vercel env add NEXT_PUBLIC_SUPABASE_URL production
# Paste value when prompted
```

---

## DATABASE SETUP

### Run Migrations

**Supabase SQL Editor:**
1. Go to https://supabase.com/dashboard/project/kteobfyferrukqeolofj/sql
2. Click "New query"
3. Paste SQL migration
4. Click "Run"

**Example Migration:**
```sql
-- Create app-specific table
CREATE TABLE IF NOT EXISTS app_data (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  data JSONB NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Enable RLS
ALTER TABLE app_data ENABLE ROW LEVEL SECURITY;

-- Create policies
CREATE POLICY "Users can view own data"
  ON app_data FOR SELECT
  USING (auth.uid() = user_id);

CREATE POLICY "Users can insert own data"
  ON app_data FOR INSERT
  WITH CHECK (auth.uid() = user_id);
```

### Verify Tables

```sql
-- Check table exists
SELECT * FROM information_schema.tables 
WHERE table_schema = 'public' 
  AND table_name = 'app_data';

-- Check RLS enabled
SELECT tablename, rowsecurity 
FROM pg_tables 
WHERE tablename = 'app_data';

-- Check policies
SELECT * FROM pg_policies 
WHERE tablename = 'app_data';
```

---

## DOMAIN CONFIGURATION

### Custom Domain Setup

**Main Domain (craudiovizai.com):**
1. Vercel Project → Settings → Domains
2. Add Domain: craudiovizai.com
3. Configure DNS (Cloudflare):
   ```
   Type: A
   Name: @
   Content: 76.76.19.19 (Vercel)
   Proxy: Yes
   
   Type: CNAME
   Name: www
   Content: cname.vercel-dns.com
   Proxy: Yes
   ```
4. Wait for DNS propagation (5-30 minutes)
5. Verify: https://craudiovizai.com

**Subdomain (app.craudiovizai.com):**
```
Type: CNAME
Name: app
Content: cname.vercel-dns.com
Proxy: Yes
```

### SSL Certificates

**Automatic (Vercel):**
- Auto-provisioned Let's Encrypt
- Auto-renewed every 90 days
- No configuration needed

**Verify SSL:**
```bash
curl -I https://craudiovizai.com
# Look for: HTTP/2 200
```

---

## TROUBLESHOOTING

### Build Failures

**Error: TypeScript Errors**
```bash
# Locally:
npm run build

# Fix errors shown
# Commit and push
```

**Error: Missing Dependencies**
```bash
# Check package.json
# Ensure all imports have dependencies
npm install [missing-package]
```

**Error: Environment Variables Not Set**
```bash
# Verify in Vercel dashboard
# Redeploy after adding variables
```

### Runtime Errors

**503 Service Unavailable**
- Check Vercel function logs
- Verify database connectivity
- Check API route errors
- Review server-side rendering issues

**API Route Timeouts**
- Check function duration (max 10s hobby, 60s pro)
- Optimize slow queries
- Consider edge functions
- Add error handling

**Database Connection Issues**
- Verify Supabase URL correct
- Check service role key valid
- Test connection from local
- Review RLS policies

### Deployment Stuck

**Symptoms:** Deployment shows "Building..." for >10 minutes

**Solutions:**
1. Cancel deployment
2. Clear build cache (Project Settings → General → Clear Cache)
3. Trigger new deployment
4. If persists, check Vercel status page

---

## ROLLBACK PROCEDURES

### Rollback to Previous Version

**Via Vercel Dashboard:**
1. Project → Deployments
2. Find last working deployment
3. Click "..." → "Promote to Production"
4. Confirm

**Via Vercel API:**
```bash
# List deployments
curl https://api.vercel.com/v6/deployments?projectId=[id] \
  -H "Authorization: Bearer [REDACTED_TOKEN][REDACTED_VERCEL_TOKEN]"

# Promote deployment
curl -X PATCH https://api.vercel.com/v13/deployments/[deployment-id] \
  -H "Authorization: Bearer [REDACTED_TOKEN][REDACTED_VERCEL_TOKEN]" \
  -d '{"target":"production"}'
```

**Via Git:**
```bash
# Revert last commit
git revert HEAD
git push origin main

# Vercel auto-deploys reverted version
```

### Emergency Rollback

**Symptoms:** Site completely down, users affected

**Steps:**
1. Promote last known good deployment (< 1 minute)
2. Notify team
3. Investigate issue in development
4. Fix and test thoroughly
5. Deploy fix when ready

---

## MONITORING

### Vercel Analytics

**Enable:**
1. Project → Settings → Analytics
2. Enable "Web Analytics"
3. View at: Project → Analytics

**Metrics:**
- Page views
- Unique visitors
- Top pages
- Devices
- Browsers
- Countries

### Function Logs

**View Logs:**
1. Project → Deployments → [deployment]
2. Click "Functions"
3. Select function
4. View logs

**Or via API:**
```bash
curl https://api.vercel.com/v2/deployments/[id]/events \
  -H "Authorization: Bearer [REDACTED_TOKEN][REDACTED_VERCEL_TOKEN]"
```

### Error Tracking

**Sentry Integration (Optional):**
```bash
npm install @sentry/nextjs

# Initialize
npx @sentry/wizard@latest -i nextjs
```

---

## DEPLOYMENT CHECKLIST

### Pre-Deployment

- [ ] Code passes TypeScript check (`npm run build` locally)
- [ ] All tests pass (`npm test`)
- [ ] Environment variables set in Vercel
- [ ] Database migrations run
- [ ] API keys valid and not expired
- [ ] No sensitive data in code
- [ ] README.md updated
- [ ] Version bumped (package.json)

### Deployment

- [ ] Push to GitHub
- [ ] Verify preview deployment successful
- [ ] Test preview URL thoroughly
- [ ] Check Vercel function logs
- [ ] Verify database queries working
- [ ] Test authentication flow
- [ ] Check payment processing (if applicable)

### Post-Deployment

- [ ] Verify production URL loads
- [ ] Check all pages/routes working
- [ ] Test API endpoints
- [ ] Monitor error logs (first 15 minutes)
- [ ] Check bot system (if applicable)
- [ ] Notify team of deployment
- [ ] Update documentation if needed

---

## BEST PRACTICES

### Development Workflow

**1. Work in Feature Branches**
```bash
git checkout -b feature/new-feature
# Make changes
git commit -m "Add new feature"
git push origin feature/new-feature
# Create pull request
# Merge to main after review
```

**2. Test Before Deploying**
```bash
# Always test locally
npm run dev

# Test production build
npm run build
npm start

# Fix any issues before pushing
```

**3. Use Preview Deployments**
- Deploy to preview first
- Test thoroughly
- Then promote to production
- Never deploy directly to production without testing

### Security

**1. Never Commit Secrets**
```bash
# Add to .gitignore
.env.local
.env.*.local
*.key
*.pem
```

**2. Use Environment Variables**
```typescript
// ✅ Good
const apiKey = process.env.OPENAI_API_KEY;

// ❌ Bad
const apiKey = "sk-proj-abc123...";
```

**3. Rotate Keys Regularly**
- API keys: Every 90 days
- Database passwords: Every 180 days
- Tokens: Every 90 days

---

## SUMMARY

**Deployment Flow:**
1. Code → GitHub
2. GitHub → Vercel (auto-trigger)
3. Vercel → Preview URL (test)
4. Preview → Production (manual promote)

**Key Points:**
- Use preview deployments (saves credits)
- Set all environment variables
- Test thoroughly before production
- Monitor after deployment
- Have rollback plan ready

**Support:**
- Vercel Docs: https://vercel.com/docs
- GitHub Docs: https://docs.github.com
- Supabase Docs: https://supabase.com/docs

---

**END OF DEPLOYMENT PROCEDURES**

**Last Updated:** November 21, 2025 - 4:32 PM EST
