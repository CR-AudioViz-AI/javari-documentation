# Javari AI - Operations Guide

**Last Updated:** November 8, 2025  
**Version:** 1.0  
**For:** Operations Team  
**Company:** CR AudioViz AI, LLC

---

## OPERATIONS OVERVIEW

This guide provides complete operational procedures for managing, monitoring, and maintaining Javari AI in production. It covers deployment workflows, monitoring dashboards, incident response, and maintenance schedules.

---

## SYSTEM ARCHITECTURE

### Production Environment

**Hosting Infrastructure:**
- **Frontend + Backend:** Vercel (Serverless)
- **Database:** Supabase (PostgreSQL 15)
- **File Storage:** Supabase Storage
- **CDN:** Vercel Edge Network
- **DNS:** Cloudflare

**Deployment URLs:**
- **Production:** javariai.com
- **Preview:** javari-javari-77b6mtat5-roy-hendersons-projects-1d3d5e94.vercel.app
- **API:** api.craudiovizai.com/javari

**Geographic Distribution:**
- Primary: US East (Ohio) - Supabase
- CDN: Global (Vercel Edge)
- Backup: US West (planned Q1 2026)

### Technology Stack

| Component | Technology | Version | Purpose |
|-----------|-----------|---------|---------|
| Frontend | Next.js | 14.1.0 | React framework |
| Language | TypeScript | 5.x | Type safety |
| Database | PostgreSQL | 15 | Data storage |
| Auth | Supabase Auth | latest | User authentication |
| Payments | Stripe | latest | Billing |
| AI | OpenAI GPT-4 | latest | Primary AI |
| Monitoring | Vercel Analytics | latest | Performance |
| Logging | Vercel Logs | latest | Error tracking |

---

## DEPLOYMENT PROCEDURES

### Standard Deployment (Preview Mode)

**Automatic on Git Push:**
```bash
# Every push to 'main' triggers preview deployment
git push origin main

# Vercel automatically:
1. Detects changes
2. Builds application
3. Runs tests
4. Deploys to preview URL
5. Sends notification

# Preview URL format:
# javari-javari-[hash]-roy-hendersons-projects.vercel.app
```

**Manual Deployment (Production):**
```bash
# 1. Verify preview deployment success
curl https://javari-javari-latest.vercel.app/api/health

# 2. Promote to production via Vercel dashboard
vercel promote javari-javari-[deployment-id] --prod

# 3. Verify production health
curl https://javariai.com/api/health
```

**Rollback Procedure:**
```bash
# If production issues detected:
# 1. Immediate rollback to last known good deployment
vercel rollback prj_zxjzE2qvMWFWqV0AspGvago6aPV5

# 2. Verify rollback success
curl https://javariai.com/api/health

# 3. Investigate issue in preview
# 4. Fix and redeploy when ready
```

### Deployment Checklist

**Pre-Deployment:**
- [ ] All environment variables configured
- [ ] Database migrations tested in staging
- [ ] Preview deployment successful
- [ ] Manual testing completed
- [ ] Performance benchmarks met
- [ ] Security scan passed
- [ ] Changelog updated

**Post-Deployment:**
- [ ] Health check passed
- [ ] Error rate < 0.1%
- [ ] Response time < 500ms p99
- [ ] Database connections stable
- [ ] AI API connectivity verified
- [ ] Monitoring dashboards green
- [ ] Customer notifications sent (if major changes)

### Environment Variables

**Required Production Variables:**
```bash
# Supabase
NEXT_PUBLIC_SUPABASE_URL=https://kteobfyferrukqeolofj.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=[anon_key]
SUPABASE_SERVICE_ROLE_KEY=[service_role_key]

# OpenAI
OPENAI_API_KEY=[api_key]

# Credential Encryption
CREDENTIAL_ENCRYPTION_KEY=[32-byte base64 key]

# Vercel (for self-healing)
VERCEL_TOKEN=[token]
VERCEL_TEAM_ID=team_s4aMvVDt3NO6NmeH90TgpFqv

# GitHub (for autonomous commits)
GITHUB_TOKEN=[token]

# Stripe (billing)
STRIPE_SECRET_KEY=[sk_live_...]
STRIPE_PUBLISHABLE_KEY=[pk_live_...]
STRIPE_WEBHOOK_SECRET=[whsec_...]

# Optional: Additional AI providers
ANTHROPIC_API_KEY=[api_key]
GOOGLE_API_KEY=[api_key]
PERPLEXITY_API_KEY=[api_key]

# Monitoring
SENTRY_DSN=[sentry_url]
VERCEL_ANALYTICS_ID=[analytics_id]
```

**Variable Management:**
```bash
# Add/update via Vercel CLI
vercel env add OPENAI_API_KEY production

# Or via Vercel dashboard:
# Project Settings → Environment Variables
```

**Security Note:** Never commit `.env` files to Git. Store master copy in 1Password vault: "CR AudioViz AI - Javari Production Secrets"

---

## MONITORING & OBSERVABILITY

### Health Checks

**API Endpoint:**
```bash
GET /api/health

Response:
{
  "status": "healthy",
  "timestamp": "2025-11-08T12:00:00Z",
  "services": {
    "database": "ok",
    "ai_providers": {
      "openai": "ok",
      "claude": "ok"
    },
    "storage": "ok"
  },
  "version": "1.0.0"
}
```

**Automated Monitoring:**
- **UptimeRobot:** Checks /api/health every 5 minutes
- **Vercel:** Checks every 30 seconds
- **Custom Cron:** Self-healing monitor every 30 minutes

**Alert Thresholds:**
| Metric | Warning | Critical | Action |
|--------|---------|----------|--------|
| Uptime | <99.5% | <99% | Page on-call |
| Response Time | >500ms | >1000ms | Investigate |
| Error Rate | >1% | >5% | Rollback |
| Database CPU | >70% | >90% | Scale up |
| AI API Errors | >5% | >10% | Switch provider |

### Performance Dashboards

**Vercel Analytics:**
- URL: https://vercel.com/CR-AudioViz-AI/javari-javari/analytics
- Metrics:
  - Real User Monitoring (RUM)
  - Core Web Vitals
  - Page load times
  - API response times
  - Error rates by endpoint

**Supabase Dashboard:**
- URL: https://supabase.com/dashboard/project/kteobfyferrukqeolofj
- Metrics:
  - Database CPU/Memory
  - Active connections
  - Query performance
  - Storage usage
  - API request volume

**Custom Analytics:**
- URL: https://craudiovizai.com/admin/javari/analytics
- Metrics:
  - AI usage by model
  - Cost per user
  - Credit consumption rate
  - Popular features
  - Conversion funnel

### Logging

**Access Logs:**
```bash
# View real-time logs
vercel logs javari-javari --follow

# Filter by severity
vercel logs javari-javari --severity error

# Search logs
vercel logs javari-javari --query "database connection"
```

**Application Logs:**
```typescript
// Production logging format
{
  "timestamp": "2025-11-08T12:00:00Z",
  "level": "error",
  "service": "javari-ai",
  "environment": "production",
  "message": "Failed to generate embedding",
  "context": {
    "user_id": "uuid",
    "project_id": "uuid",
    "error": "OpenAI API timeout"
  },
  "stack_trace": "..."
}
```

**Log Retention:**
- Vercel: 7 days (Pro plan)
- Supabase: 30 days
- Archive to S3: 1 year (future)

---

## DATABASE OPERATIONS

### Backup Strategy

**Automated Backups (Supabase):**
- **Frequency:** Daily at 3 AM UTC
- **Retention:** 
  - Daily: 7 days
  - Weekly: 4 weeks
  - Monthly: 3 months
- **Location:** Supabase managed (same region)
- **Recovery Time Objective (RTO):** <1 hour
- **Recovery Point Objective (RPO):** <24 hours

**Manual Backup Procedure:**
```bash
# Create on-demand backup
curl -X POST https://api.supabase.com/v1/projects/kteobfyferrukqeolofj/backups \
  -H "Authorization: Bearer [service_role_key]"

# Download backup
pg_dump -h db.kteobfyferrukqeolofj.supabase.co \
  -U postgres -d postgres -F c -f javari_backup_$(date +%Y%m%d).dump
```

### Database Migrations

**Schema Version Control:**
```bash
# Location: database/migrations/
001_initial_schema.sql
002_add_learning_system.sql
003_add_monitoring_tables.sql
```

**Apply Migration:**
```sql
-- 1. Connect to Supabase SQL Editor
-- 2. Copy migration file contents
-- 3. Execute
-- 4. Verify tables created
SELECT table_name FROM information_schema.tables 
WHERE table_schema = 'public';
```

**Rollback Migration:**
```sql
-- Each migration file has accompanying rollback
-- Example: 002_add_learning_system_rollback.sql
DROP TABLE IF EXISTS javari_learned_patterns;
DROP TABLE IF EXISTS javari_ab_tests;
```

### Database Maintenance

**Weekly Tasks:**
```sql
-- Vacuum database (reclaim space)
VACUUM ANALYZE;

-- Update statistics
ANALYZE;

-- Check table bloat
SELECT schemaname, tablename, pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

**Monthly Tasks:**
```sql
-- Reindex tables
REINDEX DATABASE postgres;

-- Check for unused indexes
SELECT schemaname, tablename, indexname
FROM pg_indexes
WHERE schemaname = 'public'
AND indexname NOT IN (
  SELECT indexrelid::regclass::text
  FROM pg_stat_user_indexes
  WHERE idx_scan > 0
);

-- Archive old data (>1 year)
-- Move to cold storage table
INSERT INTO javari_work_log_archive
SELECT * FROM javari_work_log
WHERE timestamp < NOW() - INTERVAL '1 year';

DELETE FROM javari_work_log
WHERE timestamp < NOW() - INTERVAL '1 year';
```

---

## CRON JOBS & AUTOMATION

### Active Cron Jobs

**Health Check (Every 30 Minutes):**
```typescript
// app/api/javari/cron/health-check/route.ts
// Vercel Cron: */30 * * * *

export async function GET() {
  // 1. Fetch all active monitors
  const { data: monitors } = await supabase
    .from('javari_health_monitors')
    .select('*')
    .eq('is_active', true);
  
  // 2. Check each project's build status
  for (const monitor of monitors) {
    const deployment = await checkVercelDeployment(monitor.project_id);
    
    if (deployment.state === 'ERROR') {
      // 3. Attempt auto-fix
      const fixed = await attemptAutoFix(deployment);
      
      // 4. Alert if fix failed
      if (!fixed) {
        await sendAlert(monitor);
      }
    }
  }
}
```

**Web Crawler (Daily at 2 AM UTC):**
```typescript
// app/api/javari/cron/web-crawl/route.ts
// Vercel Cron: 0 2 * * *

export async function GET() {
  // 1. Fetch URLs to crawl
  const { data: crawls } = await supabase
    .from('javari_web_crawls')
    .select('*')
    .lte('next_crawl_at', new Date().toISOString());
  
  // 2. Crawl each URL
  for (const crawl of crawls) {
    const pages = await fetchWebsite(crawl.url);
    
    // 3. Extract text and generate embeddings
    for (const page of pages) {
      const embedding = await generateEmbedding(page.text);
      
      // 4. Store in knowledge base
      await supabase.from('javari_knowledge_base').upsert({
        source_type: 'web_crawl',
        source_url: page.url,
        content: page.text,
        embedding
      });
    }
    
    // 5. Update crawl status
    await supabase.from('javari_web_crawls').update({
      last_crawled_at: new Date(),
      next_crawl_at: calculateNextCrawl(crawl.crawl_frequency)
    }).eq('id', crawl.id);
  }
}
```

**Credit Reset (Monthly):**
```typescript
// app/api/javari/cron/credit-reset/route.ts
// Vercel Cron: 0 0 1 * * (1st of month)

export async function GET() {
  // Reset monthly allowance for all users
  await supabase
    .from('users')
    .update({
      credits_balance: supabase.raw('monthly_allowance'),
      allowance_reset_date: supabase.raw("NOW() + INTERVAL '1 month'")
    })
    .eq('plan', 'free');
}
```

### Cron Job Monitoring

**Verify Execution:**
```bash
# Check Vercel cron logs
vercel logs javari-javari --output json | jq 'select(.path | contains("/cron/"))'

# Expected: Successful execution every 30 min
# Alert if: No execution in 1 hour
```

---

## INCIDENT RESPONSE

### Severity Levels

**P0 - Critical (Page Immediately)**
- Complete system outage
- Data breach
- Payment processing failure
- Response: 15 minutes

**P1 - High (Alert Within 1 Hour)**
- Partial outage (>10% users affected)
- AI provider failures (all providers down)
- Database performance degradation
- Response: 1 hour

**P2 - Medium (Next Business Day)**
- Single feature broken
- One AI provider down (fallbacks working)
- Non-critical performance issues
- Response: 24 hours

**P3 - Low (Backlog)**
- UI bugs
- Enhancement requests
- Documentation updates
- Response: Best effort

### Incident Response Playbook

**Step 1: Detection & Triage (0-5 minutes)**
```
1. Receive alert (PagerDuty/Slack/Email)
2. Acknowledge incident
3. Assess severity
4. Create incident channel (Slack: #incident-YYYYMMDD-summary)
5. Notify stakeholders
```

**Step 2: Investigation (5-30 minutes)**
```
1. Check health dashboard
2. Review recent deployments
3. Check error logs
4. Identify root cause
5. Document findings in incident channel
```

**Step 3: Mitigation (30-60 minutes)**
```
For Deployment Issues:
- Rollback to last good deployment
- Verify fix with health check

For Database Issues:
- Scale up resources
- Kill long-running queries
- Reindex if necessary

For AI Provider Issues:
- Switch to fallback provider
- Increase timeout thresholds
- Contact provider support

For Security Issues:
- Rotate compromised credentials
- Block suspicious IPs
- Notify affected users
```

**Step 4: Recovery & Verification (60-120 minutes)**
```
1. Verify system health restored
2. Monitor for 30 minutes
3. Run smoke tests
4. Update status page
5. Notify stakeholders of resolution
```

**Step 5: Post-Incident Review (Within 48 hours)**
```
1. Schedule post-mortem meeting
2. Document timeline
3. Identify root cause
4. List action items (prevent recurrence)
5. Assign owners and deadlines
6. Publish public incident report (if customer-facing)
```

### Common Incidents & Solutions

**Incident: Build Failures**
```
Symptoms: Deployments failing, preview URLs not generated
Root Cause: TypeScript errors, missing dependencies, env var issues
Solution:
1. Check build logs in Vercel
2. Fix code errors
3. Verify all env vars present
4. Clear build cache if necessary
5. Redeploy
```

**Incident: Database Connection Errors**
```
Symptoms: 500 errors, "could not connect to database"
Root Cause: Connection pool exhausted, network issue
Solution:
1. Check active connections in Supabase dashboard
2. Kill idle connections if >90% pool used
3. Scale up connection pool if sustained high load
4. Check for connection leaks in code
```

**Incident: AI API Failures**
```
Symptoms: "AI unavailable" errors, timeouts
Root Cause: Provider outage, rate limiting, network issues
Solution:
1. Check provider status pages (status.openai.com)
2. Switch to fallback provider automatically (should happen)
3. Increase timeout from 30s → 60s temporarily
4. Contact provider support if prolonged
```

---

## COST MANAGEMENT

### Current Costs (Monthly)

**Infrastructure:**
- Vercel Pro: $20/month
- Supabase Pro: $25/month
- **Total:** $45/month

**Variable Costs (Per User):**
- OpenAI API: $0.50-2.00/month per active user
- Anthropic API: $0.20-0.80/month per active user
- Supabase Storage: $0.10/month per active user
- **Average:** $1.50/month per active user

**Projected Costs:**
| Users | Infrastructure | API Costs | Total/Month |
|-------|----------------|-----------|-------------|
| 100 | $45 | $150 | $195 |
| 1,000 | $45 | $1,500 | $1,545 |
| 10,000 | $200 | $15,000 | $15,200 |
| 100,000 | $800 | $150,000 | $150,800 |

### Cost Optimization Strategies

**1. AI Model Routing:**
```typescript
// Use cheap models for simple tasks
function selectModel(complexity: 'simple' | 'medium' | 'complex') {
  switch (complexity) {
    case 'simple':
      return 'gpt-3.5-turbo'; // $0.0005/1K tokens
    case 'medium':
      return 'gpt-4-turbo'; // $0.01/1K tokens
    case 'complex':
      return 'gpt-4'; // $0.03/1K tokens
  }
}

// Potential savings: 80% on simple tasks
```

**2. Caching:**
```typescript
// Cache frequent queries (future feature)
const cache = new Map();

async function getAIResponse(prompt: string) {
  const cacheKey = hash(prompt);
  
  if (cache.has(cacheKey)) {
    return cache.get(cacheKey); // Free!
  }
  
  const response = await openai.chat.completions.create({...});
  cache.set(cacheKey, response, { ttl: 3600 }); // 1 hour
  return response;
}

// Potential savings: 30-40% on repeat queries
```

**3. Connection Pooling:**
```typescript
// Reuse database connections
const pool = new Pool({
  max: 20, // Max connections
  idleTimeoutMillis: 30000, // Close idle after 30s
});

// Potential savings: 50% fewer database connection charges
```

### Budget Alerts

**Set Up Cost Alerts:**
```bash
# Vercel Usage Alerts
# Dashboard → Usage → Set Spending Limit
# Alert at: $200, $500, $1000/month

# Supabase Usage Alerts
# Dashboard → Usage → Set Budget
# Alert at: $100, $250, $500/month

# OpenAI Usage Alerts
# Dashboard → Usage Limits → Set Hard/Soft Limits
# Alert at: $500, $1000/month
```

---

## SECURITY OPERATIONS

### Access Control

**Production Access:**
| Role | Access Level | 2FA Required | IP Whitelist |
|------|-------------|--------------|--------------|
| Admin (Roy) | Full | Yes | No |
| Operations | Read/Write | Yes | Optional |
| Support | Read-only | Yes | No |
| Developer | Preview only | No | No |

**Credential Rotation Schedule:**
- **GitHub Tokens:** Every 90 days
- **Vercel Tokens:** Every 90 days
- **Database Passwords:** Every 180 days
- **API Keys (AI):** On breach notification
- **Encryption Keys:** Annually

### Security Monitoring

**Automated Scans:**
```bash
# Dependabot (GitHub)
# Scans: npm dependencies
# Frequency: Daily
# Action: Auto-create PR for updates

# Snyk (Code Security)
# Scans: Code vulnerabilities
# Frequency: On each commit
# Action: Block merge if critical found

# OWASP ZAP (Penetration Testing)
# Scans: API endpoints
# Frequency: Weekly
# Action: Create ticket for findings
```

**Manual Security Audit:**
```
Frequency: Quarterly
Checklist:
- [ ] Review user access logs
- [ ] Check for unused credentials
- [ ] Verify RLS policies active
- [ ] Test credential encryption
- [ ] Review API rate limits
- [ ] Check for data leaks
- [ ] Audit third-party integrations
```

### Security Incident Response

**Data Breach Protocol:**
```
1. Immediately rotate ALL credentials
2. Lock down affected accounts
3. Notify users within 72 hours (GDPR)
4. Contact legal counsel
5. File breach report with authorities
6. Conduct forensic investigation
7. Implement fixes
8. Publish post-mortem (transparency)
```

---

## MAINTENANCE WINDOWS

### Scheduled Maintenance

**Monthly (First Tuesday, 2-4 AM UTC):**
```
Tasks:
- Database maintenance (vacuum, reindex)
- OS security patches
- Dependency updates
- Performance optimization

Impact: Minimal (read-only mode for 30-60 min)
Notification: 7 days advance notice
```

**Quarterly (First Sunday, 12-6 AM UTC):**
```
Tasks:
- Major version upgrades (Next.js, PostgreSQL)
- Infrastructure changes
- Data archival (>1 year old data)
- Security audits

Impact: Moderate (full downtime for 2-4 hours)
Notification: 14 days advance notice
```

### Emergency Maintenance

**Criteria for Emergency Maintenance:**
- Critical security vulnerability
- Data corruption risk
- Cascading failures
- Regulatory compliance issue

**Procedure:**
```
1. Get approval from CEO (Roy)
2. Create incident (P0 severity)
3. Post status page update
4. Email all users (if downtime > 5 min)
5. Execute fix
6. Verify restoration
7. Post-incident review
```

---

## DISASTER RECOVERY

### Backup & Recovery

**Recovery Time Objectives (RTO):**
- Database: 1 hour
- Application: 15 minutes
- File Storage: 2 hours

**Recovery Point Objectives (RPO):**
- Database: 24 hours (daily backup)
- Application: 0 (Git is source of truth)
- File Storage: 24 hours

**Disaster Scenarios:**

**Scenario 1: Complete Vercel Outage**
```
Backup Plan: Netlify
1. Update DNS CNAME to point to Netlify
2. Deploy from GitHub to Netlify
3. Update environment variables
4. Verify functionality
5. Notify users of provider change

Estimated Recovery: 30 minutes
```

**Scenario 2: Supabase Database Corruption**
```
Recovery Plan: Restore from backup
1. Create new Supabase project
2. Restore latest backup
3. Update connection string in Vercel
4. Verify data integrity
5. Redeploy application

Estimated Recovery: 2 hours
```

**Scenario 3: Complete Region Failure (Ohio)**
```
Failover Plan: US West region
1. Activate secondary Supabase project (US West)
2. Restore latest backup to secondary
3. Update DNS to point to US West Vercel edge
4. Update env vars
5. Verify functionality

Estimated Recovery: 4 hours
```

### Disaster Recovery Testing

**Frequency:** Bi-annually (June & December)

**Test Procedure:**
```
1. Schedule test during low-traffic period
2. Create test environment (separate from prod)
3. Simulate disaster (e.g., delete database)
4. Execute recovery plan
5. Measure recovery time
6. Document lessons learned
7. Update DR plan if needed
```

---

## RUNBOOK QUICK REFERENCE

### Common Operations

**Deploy to Production:**
```bash
# 1. Verify preview
curl https://javari-javari-latest.vercel.app/api/health

# 2. Promote
vercel promote [deployment-id] --prod

# 3. Verify
curl https://javariai.com/api/health
```

**Rollback Production:**
```bash
vercel rollback prj_zxjzE2qvMWFWqV0AspGvago6aPV5
```

**Add Environment Variable:**
```bash
vercel env add VAR_NAME production
```

**Rotate Credential:**
```bash
# 1. Generate new credential at provider
# 2. Update Vercel env var
vercel env add OPENAI_API_KEY production
# 3. Redeploy
vercel --prod
```

**Database Backup:**
```bash
pg_dump -h db.kteobfyferrukqeolofj.supabase.co \
  -U postgres -d postgres -F c -f backup_$(date +%Y%m%d).dump
```

**Check System Health:**
```bash
curl https://javariai.com/api/health | jq
```

---

## CONTACT INFORMATION

### On-Call Rotation

**Primary:** Roy Henderson
- **Email:** roy@craudiovizai.com
- **Phone:** [REDACTED]
- **Hours:** 24/7 for P0, business hours for P1-P3

**Secondary:** Cindy Henderson
- **Email:** cindy@craudiovizai.com
- **Phone:** [REDACTED]
- **Hours:** Business hours

### Vendor Support

**Vercel:**
- Support: https://vercel.com/support
- Status: https://www.vercel-status.com
- Tier: Pro (24-hour response)

**Supabase:**
- Support: https://supabase.com/dashboard/support
- Status: https://status.supabase.com
- Tier: Pro (24-hour response)

**OpenAI:**
- Support: https://help.openai.com
- Status: https://status.openai.com
- Tier: Pay-as-you-go (community support)

---

## APPENDIX

### Useful SQL Queries

**Find Slow Queries:**
```sql
SELECT query, mean_exec_time, calls
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;
```

**Check Table Sizes:**
```sql
SELECT schemaname, tablename, 
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

**Active Connections:**
```sql
SELECT count(*) FROM pg_stat_activity;
```

### Useful API Endpoints

**Health Check:**
```
GET /api/health
```

**System Status:**
```
GET /api/status
```

**Metrics:**
```
GET /api/metrics
```

---

**Operations Guide v1.0 - November 2025**  
**For Internal Use Only - CR AudioViz AI, LLC**
