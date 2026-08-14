# JAVARI AI - OPERATIONS RUNBOOK

**Version:** 1.0  
**Last Updated:** November 8, 2025  
**Maintained By:** CR AudioViz AI Operations Team

---

## TABLE OF CONTENTS

1. [Quick Reference](#quick-reference)
2. [System Status](#system-status)
3. [Deployment Procedures](#deployment-procedures)
4. [Database Operations](#database-operations)
5. [Monitoring & Alerts](#monitoring--alerts)
6. [Troubleshooting Guide](#troubleshooting-guide)
7. [Maintenance Tasks](#maintenance-tasks)
8. [Incident Response](#incident-response)
9. [Backup & Recovery](#backup--recovery)
10. [Performance Tuning](#performance-tuning)

---

## QUICK REFERENCE

### Emergency Contacts

| Role | Contact | Availability |
|------|---------|--------------|
| **Primary On-Call** | Roy Henderson | 24/7 |
| **Supabase Support** | support@supabase.com | Business hours |
| **Vercel Support** | vercel.com/support | 24/7 (Enterprise) |
| **OpenAI Support** | help.openai.com | 24/7 |

### Critical URLs

| Service | URL | Purpose |
|---------|-----|---------|
| **Production App** | https://javariai.com | Main application |
| **Admin Dashboard** | https://javariai.com/admin | Operations dashboard |
| **Vercel Dashboard** | https://vercel.com/cr-audioviz-ai/javari-javari | Deployments |
| **Supabase Dashboard** | https://supabase.com/dashboard/project/kteobfyferrukqeolofj | Database |
| **Status Page** | https://status.javariai.com | Public status |
| **GitHub Repo** | https://github.com/CR-AudioViz-AI/javari-javari | Source code |

### Quick Commands

```bash
# Check deployment status
curl -H "Authorization: Bearer $VERCEL_TOKEN" \
  "https://api.vercel.com/v6/deployments?projectId=$PROJECT_ID"

# Check database status
curl "$SUPABASE_URL/rest/v1/" \
  -H "apikey: $SUPABASE_ANON_KEY"

# View recent logs
vercel logs --since 1h

# Trigger health check
curl https://javariai.com/api/health

# Check AI provider status
curl https://status.openai.com/api/v2/status.json
```

### Environment Variables

**Critical (cannot function without):**
- `NEXT_PUBLIC_SUPABASE_URL`
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- `SUPABASE_SERVICE_ROLE_KEY`
- `OPENAI_API_KEY`
- `CREDENTIAL_ENCRYPTION_KEY`

**Optional (degrades gracefully):**
- `ANTHROPIC_API_KEY`
- `GOOGLE_API_KEY`
- `MISTRAL_API_KEY`
- `SENTRY_DSN`

---

## SYSTEM STATUS

### Health Check Endpoints

**Primary Health Check:**
```bash
curl https://javariai.com/api/health
```

**Response (Healthy):**
```json
{
  "status": "healthy",
  "timestamp": "2025-11-08T12:00:00.000Z",
  "checks": {
    "database": {
      "status": "healthy",
      "latency": 45
    },
    "ai_providers": {
      "status": "healthy",
      "openai": "operational",
      "anthropic": "operational"
    },
    "storage": {
      "status": "healthy",
      "latency": 120
    }
  },
  "version": "1.0.0"
}
```

**Response (Unhealthy):**
```json
{
  "status": "degraded",
  "timestamp": "2025-11-08T12:00:00.000Z",
  "checks": {
    "database": {
      "status": "healthy",
      "latency": 45
    },
    "ai_providers": {
      "status": "degraded",
      "openai": "operational",
      "anthropic": "error",
      "error": "Connection timeout"
    }
  }
}
```

### Key Metrics Dashboard

**Access:** https://javariai.com/admin/metrics

**Monitor:**
- Active users (real-time)
- API response times (p50, p95, p99)
- Error rates (by endpoint)
- AI provider usage (tokens, cost)
- Database performance (query time, connections)
- Credits consumed (per hour, per day)

### Service Dependencies

**Critical (required for operation):**
1. Supabase (database + auth)
2. OpenAI (AI responses)
3. Vercel (hosting)

**Non-critical (graceful degradation):**
1. Anthropic (fallback AI)
2. Google AI (fallback AI)
3. Sentry (error tracking)
4. Analytics services

---

## DEPLOYMENT PROCEDURES

### Pre-Deployment Checklist

Before deploying to production:

- [ ] All tests passing locally
- [ ] TypeScript compilation successful (`npm run build`)
- [ ] No console errors in development
- [ ] Environment variables verified
- [ ] Database migrations applied (if any)
- [ ] Changelog updated
- [ ] Stakeholders notified (if major change)

### Standard Deployment

**Method 1: Manual Preview Deployment (Recommended)**

```bash
# 1. Ensure you're on main branch
git checkout main
git pull origin main

# 2. Create preview deployment
vercel --token=$VERCEL_TOKEN

# 3. Get preview URL from output
# Preview URL: https://javari-javari-abc123.vercel.app

# 4. Test preview deployment
# - Run smoke tests
# - Check critical user flows
# - Verify no console errors

# 5. If tests pass, promote to production
vercel --prod --token=$VERCEL_TOKEN

# 6. Verify production
curl https://javariai.com/api/health

# 7. Monitor for 30 minutes
# - Check error rates
# - Monitor API latency
# - Review user feedback
```

**Method 2: GitHub Push (Automatic Preview Only)**

```bash
# 1. Push to feature branch
git push origin feature-branch

# 2. Preview deployment automatically created
# Check Vercel dashboard for URL

# 3. When ready, merge to main via PR
# NO automatic production deployment

# 4. Manually promote after validation
```

### Rolling Back Deployment

If issues detected after deployment:

```bash
# Option 1: Revert to previous deployment (instant)
vercel rollback --token=$VERCEL_TOKEN

# Option 2: Revert git commit and redeploy
git revert HEAD
git push origin main
vercel --prod --token=$VERCEL_TOKEN

# Option 3: Deploy specific commit
git checkout <previous-good-commit>
vercel --prod --token=$VERCEL_TOKEN
```

### Emergency Deployment

For critical hotfixes:

```bash
# 1. Create hotfix branch
git checkout -b hotfix/critical-issue

# 2. Make minimal changes
# Edit only what's necessary

# 3. Test locally
npm run build
npm run test

# 4. Deploy directly to production
git push origin hotfix/critical-issue
vercel --prod --token=$VERCEL_TOKEN

# 5. Monitor closely
# Watch error rates, user reports

# 6. Merge hotfix back to main after validation
git checkout main
git merge hotfix/critical-issue
git push origin main
```

### Post-Deployment Tasks

- [ ] Verify health check returns 200
- [ ] Check error tracking dashboard (no spike in errors)
- [ ] Monitor response times (no degradation)
- [ ] Review first 10-20 user interactions
- [ ] Update status page if customer-facing changes
- [ ] Document any issues encountered

---

## DATABASE OPERATIONS

### Accessing the Database

**Via Supabase Dashboard:**
1. Go to https://supabase.com/dashboard/project/kteobfyferrukqeolofj
2. Click "SQL Editor"
3. Write and execute queries

**Via psql (local):**
```bash
psql "$DATABASE_URL"
```

**Via API (programmatic):**
```bash
curl "$SUPABASE_URL/rest/v1/javari_conversations?select=*&limit=10" \
  -H "apikey: $SUPABASE_SERVICE_ROLE_KEY" \
  -H "Authorization: Bearer $SUPABASE_SERVICE_ROLE_KEY"
```

### Running Migrations

**New Migration:**
```bash
# 1. Create migration file
# migrations/YYYY-MM-DD-description.sql

# 2. Write migration SQL
# migrations/2025-11-08-add-user-preferences.sql

CREATE TABLE javari_user_preferences (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES auth.users(id),
  preferences JSONB NOT NULL DEFAULT '{}'::jsonb,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

ALTER TABLE javari_user_preferences ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can manage own preferences"
  ON javari_user_preferences
  FOR ALL
  USING (auth.uid() = user_id);

# 3. Test locally first
psql "$LOCAL_DATABASE_URL" < migrations/2025-11-08-add-user-preferences.sql

# 4. Apply to production
# Copy SQL to Supabase SQL Editor and execute

# 5. Verify
SELECT * FROM javari_user_preferences LIMIT 1;
```

**Rollback Migration:**
```sql
-- Create rollback script
-- migrations/2025-11-08-add-user-preferences-rollback.sql

DROP POLICY IF EXISTS "Users can manage own preferences" ON javari_user_preferences;
DROP TABLE IF EXISTS javari_user_preferences;
```

### Common Database Tasks

**Check Table Sizes:**
```sql
SELECT 
  schemaname,
  tablename,
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

**Check Active Connections:**
```sql
SELECT 
  count(*) AS connections,
  state
FROM pg_stat_activity
WHERE datname = 'postgres'
GROUP BY state;
```

**Find Slow Queries:**
```sql
SELECT 
  query,
  mean_exec_time,
  calls
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;
```

**Vacuum Full Database (monthly):**
```sql
-- Do this during low-traffic periods
VACUUM FULL ANALYZE;
```

**Update Statistics (weekly):**
```sql
ANALYZE;
```

### Data Cleanup

**Delete Old Conversation Summaries:**
```sql
-- Keep only last 90 days
DELETE FROM javari_conversation_summaries
WHERE created_at < NOW() - INTERVAL '90 days';
```

**Archive Old Health Checks:**
```sql
-- Keep only last 30 days
DELETE FROM javari_health_checks
WHERE created_at < NOW() - INTERVAL '30 days';
```

**Clean Up Failed Tasks:**
```sql
-- Remove tasks that failed > 7 days ago
DELETE FROM javari_task_queue
WHERE status = 'failed'
  AND completed_at < NOW() - INTERVAL '7 days';
```

---

## MONITORING & ALERTS

### Alert Thresholds

**Critical (immediate action required):**
- API error rate > 5%
- Database connections > 90% of pool
- AI provider completely down
- Response time p99 > 10 seconds
- Disk usage > 90%

**Warning (investigate within 1 hour):**
- API error rate > 1%
- Database connections > 70% of pool
- AI provider degraded performance
- Response time p99 > 5 seconds
- Memory usage > 80%

**Info (monitor, no immediate action):**
- API error rate > 0.1%
- Unusual traffic patterns
- Cost anomalies
- Feature usage spikes

### Setting Up Alerts

**Vercel Alerts (already configured):**
- Deployment failures
- Build timeouts
- High error rates
- Performance degradation

**Custom Alerts (setup required):**

**1. Database Alert:**
```sql
-- Create alert function
CREATE OR REPLACE FUNCTION check_database_health()
RETURNS void AS $$
DECLARE
  connection_count INTEGER;
BEGIN
  SELECT count(*) INTO connection_count
  FROM pg_stat_activity
  WHERE datname = 'postgres';
  
  IF connection_count > 180 THEN
    PERFORM pg_notify('db_alert', 
      json_build_object(
        'type', 'high_connections',
        'count', connection_count
      )::text
    );
  END IF;
END;
$$ LANGUAGE plpgsql;

-- Schedule to run every minute
-- (Use pg_cron or external scheduler)
```

**2. Cost Alert:**
```sql
-- Daily cost check
SELECT 
  DATE(created_at) as date,
  SUM(cost) as total_cost
FROM javari_messages
WHERE created_at >= CURRENT_DATE
GROUP BY DATE(created_at)
HAVING SUM(cost) > 100; -- Alert if > $100/day
```

**3. Error Rate Alert:**
```sql
-- Check error rate in last hour
SELECT 
  COUNT(*) FILTER (WHERE status >= 500) * 100.0 / COUNT(*) as error_rate
FROM api_logs
WHERE created_at > NOW() - INTERVAL '1 hour'
HAVING COUNT(*) FILTER (WHERE status >= 500) * 100.0 / COUNT(*) > 1.0;
```

### Monitoring Dashboards

**Vercel Analytics Dashboard:**
- Access: https://vercel.com/cr-audioviz-ai/javari-javari/analytics
- Shows: Traffic, performance, error rates

**Supabase Dashboard:**
- Access: https://supabase.com/dashboard/project/kteobfyferrukqeolofj
- Shows: Database metrics, API usage, storage

**Custom Admin Dashboard:**
- Access: https://javariai.com/admin
- Shows: Business metrics, user activity, costs

### Log Analysis

**View Recent Errors:**
```bash
# Via Vercel CLI
vercel logs --since 1h --filter "error"

# Or via Supabase logs
# Dashboard → Logs → Error logs
```

**Search Specific User Logs:**
```sql
SELECT 
  created_at,
  level,
  action,
  metadata
FROM javari_logs
WHERE user_id = '<user-id>'
  AND created_at > NOW() - INTERVAL '24 hours'
ORDER BY created_at DESC;
```

**Analyze API Performance:**
```sql
SELECT 
  action,
  AVG(duration) as avg_duration,
  PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY duration) as p95,
  COUNT(*) as request_count
FROM javari_logs
WHERE created_at > NOW() - INTERVAL '1 hour'
GROUP BY action
ORDER BY p95 DESC;
```

---

## TROUBLESHOOTING GUIDE

### Application Won't Start

**Symptoms:** Deployment succeeds but app returns 500 errors

**Diagnosis:**
```bash
# 1. Check build logs
vercel logs --since 30m

# 2. Look for specific errors
vercel logs --since 30m --filter "error"

# 3. Verify environment variables
vercel env pull
cat .env.local
```

**Common Causes & Fixes:**

**Missing Environment Variable:**
```bash
# Add missing variable
vercel env add VARIABLE_NAME

# Redeploy
vercel --prod
```

**Database Connection Failed:**
```bash
# Test connection manually
curl "$SUPABASE_URL/rest/v1/" \
  -H "apikey: $SUPABASE_ANON_KEY"

# If fails, check:
# 1. Supabase service status
# 2. Database connection string
# 3. Firewall rules (if applicable)
```

**TypeScript Compilation Error:**
```bash
# Build locally to see detailed error
npm run build

# Common issues:
# - Missing type definitions
# - Import errors
# - tsconfig.json misconfiguration

# Fix and redeploy
```

### Users Can't Sign In

**Symptoms:** Login button doesn't work, or returns error

**Diagnosis:**
```bash
# 1. Check auth logs in Supabase
# Dashboard → Authentication → Logs

# 2. Test auth endpoint
curl "$SUPABASE_URL/auth/v1/token?grant_type=password" \
  -H "apikey: $SUPABASE_ANON_KEY" \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password"}'
```

**Common Causes & Fixes:**

**Auth Service Down:**
- Check Supabase status page
- Wait for restoration or contact support

**Email Not Confirmed:**
```sql
-- Force confirm user (emergency only)
UPDATE auth.users
SET email_confirmed_at = NOW()
WHERE email = 'user@example.com';
```

**Rate Limiting:**
```sql
-- Check rate limit logs
SELECT * FROM auth.audit_log_entries
WHERE created_at > NOW() - INTERVAL '1 hour'
  AND error_msg LIKE '%rate limit%';
```

### Chat Not Responding

**Symptoms:** Message sent, no response received

**Diagnosis:**
```bash
# 1. Check AI provider status
curl https://status.openai.com/api/v2/status.json

# 2. Check recent chat attempts
SELECT 
  conversation_id,
  created_at,
  model,
  error
FROM javari_messages
WHERE created_at > NOW() - INTERVAL '10 minutes'
  AND role = 'assistant'
ORDER BY created_at DESC;

# 3. Check for errors in logs
vercel logs --since 10m --filter "chat"
```

**Common Causes & Fixes:**

**OpenAI API Key Invalid:**
```bash
# Test API key
curl https://api.openai.com/v1/models \
  -H "Authorization: Bearer $OPENAI_API_KEY"

# If invalid, regenerate and update:
vercel env rm OPENAI_API_KEY
vercel env add OPENAI_API_KEY
vercel --prod
```

**Rate Limit Reached:**
- Check OpenAI dashboard for usage
- Wait for limit reset
- Consider upgrading tier
- Enable fallback providers

**Network Timeout:**
```typescript
// Increase timeout in chat API route
const timeout = 60000; // 60 seconds
```

### High Database Load

**Symptoms:** Slow queries, timeout errors

**Diagnosis:**
```sql
-- Check current load
SELECT 
  pid,
  now() - pg_stat_activity.query_start AS duration,
  query,
  state
FROM pg_stat_activity
WHERE state != 'idle'
  AND query NOT LIKE '%pg_stat_activity%'
ORDER BY duration DESC;

-- Check connection pool
SELECT count(*) FROM pg_stat_activity;

-- Find slow queries
SELECT 
  query,
  mean_exec_time,
  calls
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;
```

**Common Causes & Fixes:**

**Too Many Connections:**
```bash
# Increase connection pool size
# Update DATABASE_URL with larger pool size
# Or upgrade Supabase tier
```

**Missing Indexes:**
```sql
-- Check for sequential scans (bad)
SELECT 
  schemaname,
  tablename,
  seq_scan,
  idx_scan,
  seq_scan / (idx_scan + 1) as ratio
FROM pg_stat_user_tables
WHERE seq_scan > 100
ORDER BY ratio DESC;

-- Add index where needed
CREATE INDEX idx_table_column ON table_name(column_name);
```

**Unbounded Queries:**
```typescript
// Always use LIMIT
const { data } = await supabase
  .from('javari_conversations')
  .select('*')
  .limit(100); // Add this!
```

### High AI Costs

**Symptoms:** Unexpected increase in OpenAI bills

**Diagnosis:**
```sql
-- Check token usage by user
SELECT 
  user_id,
  DATE(created_at) as date,
  SUM(tokens) as total_tokens,
  SUM(cost) as total_cost
FROM javari_messages
WHERE created_at > NOW() - INTERVAL '7 days'
GROUP BY user_id, DATE(created_at)
ORDER BY total_cost DESC;

-- Check most expensive conversations
SELECT 
  conversation_id,
  SUM(tokens) as total_tokens,
  SUM(cost) as total_cost,
  COUNT(*) as message_count
FROM javari_messages
GROUP BY conversation_id
ORDER BY total_cost DESC
LIMIT 10;
```

**Common Causes & Fixes:**

**Abuse/Bot Activity:**
```sql
-- Identify suspicious users
SELECT 
  user_id,
  COUNT(*) as message_count,
  SUM(cost) as total_cost
FROM javari_messages
WHERE created_at > NOW() - INTERVAL '24 hours'
GROUP BY user_id
HAVING COUNT(*) > 1000 -- Flag users with >1000 messages/day
ORDER BY message_count DESC;

-- Temporarily block user
UPDATE users
SET blocked = true,
  blocked_reason = 'Excessive usage - possible abuse'
WHERE id = '<user-id>';
```

**Not Using Cheaper Models:**
```typescript
// Route simple queries to cheaper models
if (isSimpleQuery(message)) {
  model = 'gpt-3.5-turbo'; // 10x cheaper
}
```

**Inefficient Prompts:**
- Review system prompts for verbosity
- Reduce context window when possible
- Cache common responses

---

## MAINTENANCE TASKS

### Daily Tasks

**Automated (no action required):**
- Database backups (Supabase automatic)
- Health checks (every 5 minutes)
- Log rotation
- SSL certificate renewal

**Manual:**
- Review error dashboard (5 minutes)
- Check cost dashboard (2 minutes)
- Scan support tickets (10 minutes)

### Weekly Tasks

**Monday Morning (30 minutes):**
- [ ] Review weekend metrics
- [ ] Check for security updates
- [ ] Review feature flags
- [ ] Plan deployments for the week

**Friday Afternoon (30 minutes):**
- [ ] Deploy non-critical updates
- [ ] Review week's performance
- [ ] Update documentation (if needed)
- [ ] Prepare next week's plan

### Monthly Tasks

**First Monday (2 hours):**
- [ ] Review monthly costs
- [ ] Analyze user growth
- [ ] Database maintenance (vacuum, analyze)
- [ ] Update dependencies
- [ ] Security audit
- [ ] Backup verification
- [ ] Review and archive old logs

**Process:**
```bash
# 1. Update dependencies
npm outdated
npm update
npm audit fix

# 2. Test locally
npm run build
npm run test

# 3. Deploy to preview
vercel

# 4. If successful, deploy to production
vercel --prod

# 5. Monitor for 1 hour
```

### Quarterly Tasks

**Security Review (4 hours):**
- Rotate encryption keys
- Review access logs
- Update credentials
- Penetration testing
- Compliance check

**Performance Optimization (4 hours):**
- Analyze slow queries
- Review cache hit rates
- Optimize large tables
- Update indexes
- Review AI provider costs

**Documentation Update (2 hours):**
- Update runbook (this document)
- Review API documentation
- Update architecture diagrams
- Document new features

---

## INCIDENT RESPONSE

### Incident Severity Levels

**P0 - Critical (15 minute response)**
- Complete service outage
- Data loss or corruption
- Security breach
- Revenue-impacting bug

**P1 - High (1 hour response)**
- Major feature unavailable
- Significant performance degradation
- Partial service outage
- High error rates

**P2 - Medium (4 hour response)**
- Minor feature issues
- Intermittent errors
- Non-critical performance issues

**P3 - Low (Next business day)**
- Cosmetic issues
- Feature requests
- Minor bugs with workarounds

### Incident Response Process

**1. Detection (0-5 minutes)**
- Alert received OR user report OR monitoring dashboard
- Assess severity
- Page on-call if P0/P1

**2. Initial Response (0-15 minutes)**
```bash
# Create incident in system
incident_id=$(date +%Y%m%d-%H%M%S)
echo "INCIDENT: $incident_id" >> /var/log/incidents.log

# Gather initial information
# - What is broken?
# - Since when?
# - How many users affected?
# - Is it getting worse?

# Post to status page if customer-facing
curl -X POST https://status.javariai.com/api/incidents \
  -d '{"title":"Investigating chat response delays","status":"investigating"}'
```

**3. Mitigation (15 minutes - 2 hours)**
- Implement quick fix or workaround
- Roll back recent deployment if needed
- Route around failing component

```bash
# Example: AI provider down, switch to fallback
vercel env add FORCE_PROVIDER=anthropic
vercel --prod

# Example: Database slow, enable read-only mode
vercel env add READ_ONLY_MODE=true
vercel --prod
```

**4. Resolution (2-24 hours)**
- Implement permanent fix
- Deploy and verify
- Monitor for recurrence

**5. Post-Mortem (24-48 hours after resolution)**
- Write incident report
- Identify root cause
- Document lessons learned
- Create action items to prevent recurrence

### Common Incidents & Playbooks

**Playbook 1: Complete Service Outage**

```bash
# 1. Verify outage
curl https://javariai.com/api/health
# Returns: timeout or 500 error

# 2. Check recent deployments
vercel deployments --since 1h

# 3. Roll back to last known good deployment
vercel rollback

# 4. If rollback doesn't fix, check dependencies
# - Supabase status
# - Vercel status
# - AI provider status

# 5. Update status page
# Post: "We're aware of the issue and investigating"

# 6. Once resolved, update status page
# Post: "Service restored. Monitoring for stability."

# 7. Post-mortem within 48 hours
```

**Playbook 2: High Error Rate**

```bash
# 1. Identify error types
vercel logs --since 1h --filter "error"

# 2. Check if error rate increasing or stable
# Compare current hour to previous hours

# 3. If specific endpoint, disable if possible
# Or implement circuit breaker

# 4. If specific user, temporarily limit
UPDATE users
SET rate_limit = 10
WHERE id = '<user-id>';

# 5. Deploy fix when identified
# 6. Monitor for return to normal
```

**Playbook 3: Database Performance Issues**

```sql
-- 1. Identify slow queries
SELECT 
  pid,
  now() - query_start AS duration,
  query
FROM pg_stat_activity
WHERE state = 'active'
ORDER BY duration DESC;

-- 2. Kill long-running queries if needed
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE pid = <pid>;

-- 3. Check for lock contention
SELECT * FROM pg_locks
WHERE NOT granted;

-- 4. Emergency: Restart database
-- (Last resort, contact Supabase support)
```

---

## BACKUP & RECOVERY

### Backup Schedule

**Automated Backups:**
- **Database:** Daily full backup (Supabase, 7-day retention)
- **Point-in-time recovery:** Up to 7 days ago (Supabase)
- **Files:** Continuous replication (Supabase Storage)

**Manual Backups:**
- **Weekly:** Database export to S3
- **Monthly:** Full system snapshot
- **Before major changes:** On-demand backup

### Creating Manual Backups

**Database Backup:**
```bash
# 1. Export full database
pg_dump "$DATABASE_URL" > backup-$(date +%Y%m%d).sql

# 2. Compress
gzip backup-$(date +%Y%m%d).sql

# 3. Upload to S3 (or other storage)
aws s3 cp backup-$(date +%Y%m%d).sql.gz \
  s3://javari-backups/database/
```

**Files Backup:**
```bash
# Supabase Storage already replicated
# For extra redundancy, sync to S3:
supabase storage download <bucket-name> \
  --output ./storage-backup/

aws s3 sync ./storage-backup/ \
  s3://javari-backups/storage/
```

**Environment Variables Backup:**
```bash
# Export environment variables
vercel env pull .env.backup

# Store securely (encrypted)
gpg --encrypt .env.backup

# Upload to secure storage
```

### Restore Procedures

**Restore Database from Backup:**

**Recent (within 7 days):**
```bash
# 1. Use Supabase point-in-time recovery
# Dashboard → Database → Backups → Restore

# 2. Choose timestamp
# 3. Confirm restoration
# 4. Wait 5-10 minutes
# 5. Verify data
```

**Older (manual backup):**
```bash
# 1. Download backup from S3
aws s3 cp s3://javari-backups/database/backup-20251101.sql.gz .

# 2. Decompress
gunzip backup-20251101.sql.gz

# 3. Restore to database
# WARNING: This will overwrite current data
psql "$DATABASE_URL" < backup-20251101.sql

# 4. Verify restoration
psql "$DATABASE_URL" -c "SELECT COUNT(*) FROM javari_conversations;"
```

**Restore Files:**
```bash
# 1. Download backup
aws s3 sync s3://javari-backups/storage/ ./storage-backup/

# 2. Upload to Supabase Storage
supabase storage upload <bucket-name> ./storage-backup/
```

### Disaster Recovery

**Complete System Loss:**

1. **Restore Infrastructure (30 minutes)**
   ```bash
   # 1. Create new Vercel project
   vercel link
   
   # 2. Create new Supabase project
   # Or restore existing from backup
   
   # 3. Set environment variables
   vercel env add SUPABASE_URL
   vercel env add OPENAI_API_KEY
   # ... all other variables
   
   # 4. Deploy application
   vercel --prod
   ```

2. **Restore Database (1 hour)**
   - Follow database restore procedures above
   - Verify data integrity

3. **Restore Files (30 minutes)**
   - Follow file restore procedures above

4. **Verify System (30 minutes)**
   - Run full test suite
   - Manual testing of critical paths
   - Check all integrations

5. **Resume Operations (15 minutes)**
   - Update DNS if needed
   - Update status page
   - Notify users

**Total Recovery Time Objective (RTO):** 2.5 hours  
**Recovery Point Objective (RPO):** 24 hours (daily backups)

---

## PERFORMANCE TUNING

### Database Query Optimization

**Identify Slow Queries:**
```sql
-- Top 10 slowest queries
SELECT 
  query,
  mean_exec_time,
  calls,
  total_exec_time
FROM pg_stat_statements
WHERE mean_exec_time > 100 -- >100ms
ORDER BY mean_exec_time DESC
LIMIT 10;
```

**Add Missing Indexes:**
```sql
-- Check for sequential scans
SELECT 
  schemaname,
  tablename,
  seq_scan,
  idx_scan
FROM pg_stat_user_tables
WHERE seq_scan > 1000
  AND idx_scan = 0;

-- Add index
CREATE INDEX CONCURRENTLY idx_table_column 
  ON table_name(column_name);
```

**Optimize Queries:**
```sql
-- Before: Slow
SELECT * FROM javari_messages
WHERE conversation_id IN (
  SELECT id FROM javari_conversations WHERE user_id = '<user-id>'
);

-- After: Fast
SELECT m.* FROM javari_messages m
INNER JOIN javari_conversations c ON m.conversation_id = c.id
WHERE c.user_id = '<user-id>';
```

### Caching Implementation

**Add Redis Cache:**
```typescript
import Redis from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

// Cache expensive queries
async function getCachedConversations(userId: string) {
  const cacheKey = `conversations:${userId}`;
  
  // Try cache first
  const cached = await redis.get(cacheKey);
  if (cached) return JSON.parse(cached);
  
  // If miss, query database
  const conversations = await supabase
    .from('javari_conversations')
    .select('*')
    .eq('user_id', userId);
  
  // Store in cache (1 hour)
  await redis.setex(cacheKey, 3600, JSON.stringify(conversations));
  
  return conversations;
}
```

**Cache Embeddings:**
```typescript
async function getCachedEmbedding(text: string) {
  const hash = crypto.createHash('sha256').update(text).digest('hex');
  const cacheKey = `embedding:${hash}`;
  
  const cached = await redis.get(cacheKey);
  if (cached) return JSON.parse(cached);
  
  const embedding = await openai.embeddings.create({ input: text });
  await redis.setex(cacheKey, 86400, JSON.stringify(embedding));
  
  return embedding;
}
```

### AI Cost Optimization

**Route to Cheaper Models:**
```typescript
function selectModel(message: string) {
  // Simple queries → GPT-3.5 (10x cheaper)
  if (message.length < 100 && !message.includes('```')) {
    return 'gpt-3.5-turbo';
  }
  
  // Complex queries → GPT-4
  return 'gpt-4-turbo-preview';
}
```

**Reduce Context Size:**
```typescript
// Don't send full conversation every time
const relevantMessages = conversation.slice(-10); // Last 10 messages only

// Add summary of older messages instead
const summary = generateSummary(conversation.slice(0, -10));
const messages = [
  { role: 'system', content: `Previous context: ${summary}` },
  ...relevantMessages
];
```

**Cache Common Responses:**
```typescript
const commonResponses = {
  'hello': 'Hello! How can I help you today?',
  'thanks': "You're welcome! Let me know if you need anything else.",
};

if (commonResponses[message.toLowerCase()]) {
  return commonResponses[message.toLowerCase()];
}
```

---

## CONCLUSION

This runbook provides operational procedures for Javari AI. Keep it updated as the system evolves.

**Key Principles:**
1. **Monitor constantly** - Catch issues before users notice
2. **Document everything** - Every incident teaches us something
3. **Automate relentlessly** - Manual tasks are error-prone
4. **Test thoroughly** - Especially before major changes
5. **Communicate clearly** - Keep stakeholders informed

**Questions or Issues?**
- Check this runbook first
- Search past incidents for similar issues
- Escalate to Roy Henderson if needed
- Document solution for next time

**Remember:** Your success is the platform's success. When you make Javari better, you make every user's experience better.

---

**Last updated:** November 8, 2025  
**Next review:** December 8, 2025  
**Maintained by:** CR AudioViz AI Operations Team

*Stay vigilant. Stay informed. Keep building.* 🚀
