# TROUBLESHOOTING GUIDE - COMPLETE REFERENCE
**CR AudioViz AI Platform**

**Last Updated:** November 21, 2025 - 4:38 PM EST  
**For:** Developers, Support Team, System Administrators

---

## QUICK DIAGNOSTICS

**Site Down?**
1. Check https://craudiovizai.com status
2. Check Vercel status page
3. Check Supabase status
4. Review Vercel logs
5. Check bot system tickets

**App Not Working?**
1. Check browser console for errors
2. Verify user has credits
3. Check API endpoint status
4. Review application logs
5. Test with different account

---

## TABLE OF CONTENTS

1. [Website Issues](#website-issues)
2. [Build & Deployment](#build--deployment)
3. [Database Problems](#database-problems)
4. [Authentication Issues](#authentication-issues)
5. [Payment Processing](#payment-processing)
6. [API Errors](#api-errors)
7. [Performance Problems](#performance-problems)
8. [Bot System Issues](#bot-system-issues)

---

## WEBSITE ISSUES

### 503 Service Unavailable

**Symptoms:**
- Main website returns 503 error
- All pages inaccessible
- Users cannot login

**Common Causes:**
1. Vercel function timeout
2. Database connection failure
3. Next.js SSR error
4. API route error

**Diagnosis:**
```bash
# Check Vercel deployment status
curl -s https://api.vercel.com/v6/deployments?projectId=[id]&limit=1 \
  -H "Authorization: Bearer [token]" | jq .

# Check specific error
# Go to Vercel Dashboard → Project → Functions → View Logs
```

**Solutions:**

**If Next.js SSR Error:**
1. Identify failing page/component
2. Check server-side data fetching
3. Add error boundaries
4. Fix and redeploy

```typescript
// Bad: Failing during SSR
export default async function Page() {
  const data = await fetch('api/data'); // Might fail
  return <div>{data}</div>;
}

// Good: Error handling
export default async function Page() {
  try {
    const data = await fetch('api/data');
    return <div>{data}</div>;
  } catch (error) {
    return <div>Error loading data</div>;
  }
}
```

**If Database Connection:**
1. Verify Supabase URL in env vars
2. Check service role key valid
3. Test connection from Vercel
4. Check RLS policies

```bash
# Test from Vercel
# Add temporary API route:
// pages/api/test-db.ts
import { createClient } from '@supabase/supabase-js';
export default async function handler(req, res) {
  const supabase = createClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL,
    process.env.SUPABASE_SERVICE_ROLE_KEY
  );
  const { data, error } = await supabase.from('profiles').select('count');
  res.json({ data, error });
}
```

**If Function Timeout:**
1. Optimize slow queries
2. Add caching
3. Use edge functions
4. Increase timeout (Pro plan)

---

### Pages Loading Slowly

**Symptoms:**
- Pages take >5 seconds to load
- Users complain about performance
- High bounce rate

**Diagnosis:**
```bash
# Check Vercel Analytics
# Dashboard → Analytics → Web Vitals

# Check specific page
curl -w "@curl-format.txt" -o /dev/null -s https://craudiovizai.com/page
# curl-format.txt:
# time_total: %{time_total}\n
```

**Common Causes:**
1. Large images not optimized
2. Too many API calls
3. Slow database queries
4. No caching

**Solutions:**

**Optimize Images:**
```typescript
// Use Next.js Image component
import Image from 'next/image';

<Image 
  src="/logo.png" 
  width={200} 
  height={100}
  alt="Logo"
  priority={true} // For above-fold images
/>
```

**Add Caching:**
```typescript
// API route with caching
export const config = {
  runtime: 'edge', // Faster
};

export default async function handler(req) {
  const cached = await cache.get('key');
  if (cached) return cached;
  
  const data = await fetchData();
  await cache.set('key', data, 3600); // 1 hour
  return data;
}
```

**Optimize Database:**
```sql
-- Add indexes to slow queries
CREATE INDEX idx_profiles_email ON profiles(email);
CREATE INDEX idx_credits_user_created ON credits(user_id, created_at DESC);

-- Use EXPLAIN ANALYZE to find slow queries
EXPLAIN ANALYZE SELECT * FROM profiles WHERE email = 'test@example.com';
```

---

## BUILD & DEPLOYMENT

### TypeScript Build Errors

**Symptoms:**
- Build fails in Vercel
- Error: "Type 'X' is not assignable to type 'Y'"
- Deployment stuck

**Common Issues:**

**1. Database Type Mismatches**
```typescript
// Problem: Supabase types don't match actual schema
const { data } = await supabase
  .from('profiles')
  .select('*')
  .single();

// data.firstName doesn't exist in actual table (it's display_name)
```

**Solution:**
```bash
# Regenerate types from Supabase
npx supabase gen types typescript --project-id kteobfyferrukqeolofj > types/database.types.ts

# Or manually define
interface Profile {
  id: string;
  display_name: string; // Correct field name
  email: string;
  ...
}
```

**2. Missing Imports**
```typescript
// Problem
import { SomeType } from './types'; // File doesn't exist
```

**Solution:**
```bash
# Find missing imports locally
npm run build

# Fix all import errors
# Commit and push
```

**3. Strict Mode Issues**
```typescript
// Problem: TypeScript strict mode catches issues
const value = null;
value.toString(); // Error: Object is possibly 'null'
```

**Solution:**
```typescript
// Add null checks
if (value) {
  value.toString();
}

// Or use optional chaining
value?.toString();
```

**Verify Build Locally:**
```bash
# Always test before pushing
npm run build

# If successful
git add .
git commit -m "Fix TypeScript errors"
git push
```

---

### Deployment Stuck

**Symptoms:**
- Deployment shows "Building..." for >10 minutes
- No logs appearing
- Can't cancel

**Solutions:**

**1. Clear Build Cache**
```bash
# Vercel Dashboard
# Project → Settings → General
# Scroll to "Build & Development Settings"
# Click "Clear Cache"
# Trigger new deployment
```

**2. Check Vercel Status**
```bash
# Visit https://www.vercel-status.com
# Check for ongoing incidents
```

**3. Manual Deployment**
```bash
# Cancel stuck deployment
# Deploy via CLI
vercel --prod

# Or via API
curl -X DELETE https://api.vercel.com/v13/deployments/[deployment-id] \
  -H "Authorization: Bearer [token]"
```

---

## DATABASE PROBLEMS

### Connection Timeouts

**Symptoms:**
- "connect ETIMEDOUT" errors
- Database queries hanging
- Intermittent failures

**Diagnosis:**
```typescript
// Test connection
const supabase = createClient(url, key);
const { data, error } = await supabase
  .from('profiles')
  .select('count')
  .single();

console.log({ data, error }); // Check for errors
```

**Common Causes:**
1. Incorrect Supabase URL
2. Wrong API key
3. Network issues
4. Supabase outage

**Solutions:**

**Check Environment Variables:**
```bash
# Vercel Dashboard
# Project → Settings → Environment Variables
# Verify:
# - NEXT_PUBLIC_SUPABASE_URL is correct
# - Keys don't have extra spaces
# - Keys are for correct project
```

**Test Direct Connection:**
```bash
curl -X GET https://kteobfyferrukqeolofj.supabase.co/rest/v1/profiles?select=count \
  -H "apikey: [ANON_KEY]" \
  -H "Authorization: Bearer [SERVICE_KEY]"
```

**Implement Retry Logic:**
```typescript
async function queryWithRetry(query, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      const { data, error } = await query();
      if (error) throw error;
      return data;
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await sleep(Math.pow(2, i) * 1000); // Exponential backoff
    }
  }
}
```

---

### RLS Policy Blocking Queries

**Symptoms:**
- Query returns empty results
- User can't access their own data
- 403 errors from database

**Diagnosis:**
```sql
-- Check if RLS is enabled
SELECT tablename, rowsecurity 
FROM pg_tables 
WHERE tablename = 'your_table';

-- Check policies
SELECT * FROM pg_policies 
WHERE tablename = 'your_table';
```

**Common Issues:**

**1. Policy Using Wrong Function**
```sql
-- Bad: Using wrong auth function
CREATE POLICY "Users view own data"
  ON profiles FOR SELECT
  USING (id = current_user); -- Wrong function

-- Good: Use auth.uid()
CREATE POLICY "Users view own data"
  ON profiles FOR SELECT
  USING (id = auth.uid());
```

**2. Missing Policy for Operation**
```sql
-- User can SELECT but not INSERT
-- Need policy for INSERT

CREATE POLICY "Users can insert"
  ON profiles FOR INSERT
  WITH CHECK (auth.uid() = id);
```

**3. Service Role Not Bypassing RLS**
```typescript
// Use service role key for admin operations
const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL,
  process.env.SUPABASE_SERVICE_ROLE_KEY // Bypasses RLS
);
```

---

## AUTHENTICATION ISSUES

### Users Can't Sign In

**Symptoms:**
- Login form doesn't work
- "Invalid credentials" error
- Redirect loops

**Diagnosis:**
```typescript
// Test auth
const { data, error } = await supabase.auth.signInWithPassword({
  email: 'test@example.com',
  password: 'password'
});
console.log({ data, error });
```

**Common Causes:**
1. Wrong Supabase URL/keys
2. Email not confirmed
3. Account locked
4. Session expired

**Solutions:**

**Check Auth Config:**
```bash
# Supabase Dashboard
# Authentication → Settings
# Verify:
# - Email confirmation required?
# - Password requirements
# - Rate limiting settings
```

**Reset User Password:**
```sql
-- As admin in Supabase SQL editor
-- Send reset email
SELECT auth.send_reset_password_email('user@example.com');
```

**Check Session:**
```typescript
// Verify session exists
const { data: { session } } = await supabase.auth.getSession();
if (!session) {
  // Redirect to login
}
```

---

### Token Expired Errors

**Symptoms:**
- "JWT expired" errors
- Users logged out unexpectedly
- 401 errors

**Solutions:**

**Implement Token Refresh:**
```typescript
// Automatically refresh tokens
supabase.auth.onAuthStateChange((event, session) => {
  if (event === 'TOKEN_REFRESHED') {
    // Session refreshed automatically
    console.log('Token refreshed');
  }
  if (event === 'SIGNED_OUT') {
    // Redirect to login
  }
});
```

**Manual Refresh:**
```typescript
const { data, error } = await supabase.auth.refreshSession();
if (error) {
  // Redirect to login
}
```

---

## PAYMENT PROCESSING

### Stripe Checkout Not Working

**Symptoms:**
- Checkout button doesn't work
- Redirect to wrong URL
- Payment doesn't process

**Diagnosis:**
```bash
# Check Stripe key
curl https://api.stripe.com/v1/products \
  -u [STRIPE_SECRET_KEY]:

# Should return products list
```

**Common Issues:**

**1. Test vs Live Keys**
```typescript
// Problem: Using test key in production
const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);
// Key starts with sk_test_ but should be sk_live_
```

**Solution:**
```bash
# Verify correct key in Vercel
# Test keys: sk_test_... / pk_test_...
# Live keys: sk_live_... / pk_live_...
```

**2. Webhook Secret Wrong**
```typescript
// Webhook verification failing
const signature = req.headers['stripe-signature'];
const event = stripe.webhooks.constructEvent(
  req.body,
  signature,
  process.env.STRIPE_WEBHOOK_SECRET // Check this
);
```

**Test Webhook:**
```bash
# Use Stripe CLI
stripe listen --forward-to localhost:3000/api/webhooks/stripe

# Trigger test event
stripe trigger payment_intent.succeeded
```

---

### Credits Not Added After Payment

**Symptoms:**
- User paid but credits didn't update
- Subscription active but no credits
- Webhook not firing

**Diagnosis:**
```bash
# Check webhook logs in Stripe
# Dashboard → Developers → Webhooks
# Click webhook → View logs

# Check database
SELECT * FROM transactions 
WHERE user_id = '[user-id]' 
ORDER BY created_at DESC 
LIMIT 10;
```

**Solutions:**

**1. Webhook Not Configured**
```bash
# Add webhook in Stripe Dashboard
# URL: https://craudiovizai.com/api/webhooks/stripe
# Events: 
#   - checkout.session.completed
#   - invoice.payment_succeeded
#   - customer.subscription.updated
```

**2. Webhook Handler Error**
```typescript
// Check webhook logs
export default async function handler(req, res) {
  try {
    const event = constructEvent(req);
    
    if (event.type === 'checkout.session.completed') {
      // Add credits
      await addCredits(userId, amount);
    }
    
    res.json({ received: true });
  } catch (error) {
    console.error('Webhook error:', error);
    res.status(400).send(`Webhook Error: ${error.message}`);
  }
}
```

---

## API ERRORS

### 402 Insufficient Credits

**Symptoms:**
- User has credits but gets error
- Credit balance not updating
- Can't use any apps

**Diagnosis:**
```sql
-- Check actual balance
SELECT 
  user_id,
  SUM(amount) as balance
FROM credits
WHERE user_id = '[user-id]'
GROUP BY user_id;

-- Check last transaction
SELECT * FROM credits
WHERE user_id = '[user-id]'
ORDER BY created_at DESC
LIMIT 5;
```

**Common Causes:**
1. Balance calculation wrong
2. Transaction not recorded
3. Negative balance
4. Credit cost wrong

**Solutions:**

**Recalculate Balance:**
```typescript
async function recalculateBalance(userId) {
  const { data } = await supabase
    .from('credits')
    .select('amount')
    .eq('user_id', userId);
  
  const balance = data.reduce((sum, t) => sum + t.amount, 0);
  
  // Update profile
  await supabase
    .from('profiles')
    .update({ credits: balance })
    .eq('id', userId);
}
```

---

### 429 Rate Limit Exceeded

**Symptoms:**
- "Too Many Requests" error
- API blocked temporarily
- Users can't perform actions

**Common Causes:**
1. Bot or script hitting API
2. User retrying too quickly
3. Rate limit too strict
4. DDoS attack

**Solutions:**

**Check Rate Limit Config:**
```typescript
// Middleware: rate-limit.ts
export function rateLimit(maxRequests = 100, windowMs = 60000) {
  const requests = new Map();
  
  return (req, res, next) => {
    const key = req.ip || req.headers['x-forwarded-for'];
    const now = Date.now();
    const userRequests = requests.get(key) || [];
    
    // Remove old requests
    const recent = userRequests.filter(time => now - time < windowMs);
    
    if (recent.length >= maxRequests) {
      return res.status(429).json({
        error: 'Too many requests',
        retryAfter: Math.ceil((recent[0] + windowMs - now) / 1000)
      });
    }
    
    requests.set(key, [...recent, now]);
    next();
  };
}
```

**Implement Exponential Backoff (Client):**
```typescript
async function apiCall(url, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      const response = await fetch(url);
      
      if (response.status === 429) {
        const retryAfter = response.headers.get('Retry-After') || Math.pow(2, i);
        await sleep(retryAfter * 1000);
        continue;
      }
      
      return response;
    } catch (error) {
      if (i === retries - 1) throw error;
    }
  }
}
```

---

## PERFORMANCE PROBLEMS

### High Memory Usage

**Symptoms:**
- Functions running out of memory
- "Out of memory" errors
- Slow performance

**Diagnosis:**
```typescript
// Log memory usage
console.log({
  used: process.memoryUsage().heapUsed / 1024 / 1024,
  total: process.memoryUsage().heapTotal / 1024 / 1024
});
```

**Solutions:**

**1. Reduce Memory Footprint:**
```typescript
// Bad: Loading everything into memory
const allData = await supabase.from('large_table').select('*');

// Good: Paginate
async function* fetchPaginated(table, pageSize = 1000) {
  let offset = 0;
  while (true) {
    const { data } = await supabase
      .from(table)
      .select('*')
      .range(offset, offset + pageSize - 1);
    
    if (!data || data.length === 0) break;
    yield data;
    offset += pageSize;
  }
}
```

**2. Increase Function Memory:**
```json
// vercel.json
{
  "functions": {
    "api/heavy-task.ts": {
      "memory": 3008,
      "maxDuration": 60
    }
  }
}
```

---

## BOT SYSTEM ISSUES

### Bots Not Running

**Symptoms:**
- No recent bot executions
- Last execution timestamp old
- Issues not being detected

**Diagnosis:**
```sql
-- Check last execution
SELECT name, last_execution_at 
FROM bots 
ORDER BY last_execution_at DESC;

-- If >15 minutes ago, bots aren't running
```

**Common Causes:**
1. Vercel cron jobs paused
2. API route error
3. Environment variable missing

**Solutions:**

**Check Cron Jobs:**
```bash
# Vercel Dashboard
# Project → Settings → Cron Jobs
# Verify all bots have schedules
# Click "Run" to test manually
```

**Test Bot Endpoint:**
```bash
curl https://craudiovizai.com/api/bots/conductor
# Should return execution status
```

---

## COMMON ERROR CODES

### Quick Reference

| Code | Error | Common Cause | Solution |
|------|-------|--------------|----------|
| 400 | Bad Request | Invalid parameters | Check request format |
| 401 | Unauthorized | No/invalid token | Login again |
| 402 | Payment Required | Insufficient credits | Purchase more |
| 403 | Forbidden | No permission | Check user role |
| 404 | Not Found | Wrong URL/ID | Verify endpoint exists |
| 429 | Too Many Requests | Rate limited | Wait and retry |
| 500 | Internal Error | Server issue | Check logs |
| 503 | Service Unavailable | Site down | Check Vercel status |

---

## EMERGENCY PROCEDURES

### Site Completely Down

**1. Assess Impact**
- Check status page
- Check social media mentions
- Estimate users affected

**2. Immediate Actions**
- Roll back to last known good deployment
- Post status update
- Notify team

**3. Investigation**
- Review Vercel logs
- Check database status
- Review recent changes
- Identify root cause

**4. Communication**
- Update status page every 15 minutes
- Post on social media
- Email affected users if extended

**5. Resolution**
- Deploy fix
- Verify resolution
- Monitor for 1 hour
- Post-mortem documentation

---

## GETTING HELP

### Internal Resources
- Admin Dashboard: https://craudiovizai.com/admin
- Bot Status: https://craudiovizai.com/bots/status
- Documentation: https://github.com/CR-AudioViz-AI/javari-documentation

### External Support
- Vercel: https://vercel.com/support
- Supabase: https://supabase.com/support
- Stripe: https://support.stripe.com

---

**END OF TROUBLESHOOTING GUIDE**

**Last Updated:** November 21, 2025 - 4:38 PM EST
