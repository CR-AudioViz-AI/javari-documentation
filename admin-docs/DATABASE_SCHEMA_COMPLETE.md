# DATABASE SCHEMA - COMPLETE DOCUMENTATION
**CR AudioViz AI Supabase PostgreSQL Database**

**Last Updated:** November 21, 2025 - 4:13 PM EST  
**Database:** kteobfyferrukqeolofj.supabase.co  
**Tables:** 80+  
**Status:** Fully Operational

---

## TABLE OF CONTENTS

1. [Core System Tables](#core-system-tables)
2. [Authentication & Users](#authentication--users)
3. [Credits & Payments](#credits--payments)
4. [Javari AI Tables](#javari-ai-tables)
5. [Bot System Tables](#bot-system-tables)
6. [Applications & Tools](#applications--tools)
7. [Games Platform](#games-platform)
8. [Javariverse](#javariverse)
9. [Social Impact Modules](#social-impact-modules)
10. [Analytics & Logs](#analytics--logs)

---

## CORE SYSTEM TABLES

### `profiles`
**Purpose:** Extended user profiles beyond auth.users

```sql
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  email VARCHAR(255) UNIQUE NOT NULL,
  display_name VARCHAR(100),
  avatar_url TEXT,
  role VARCHAR(20) DEFAULT 'user', -- user, admin, owner
  credits INTEGER DEFAULT 0,
  subscription_tier VARCHAR(20), -- light, standard, advanced, premium
  subscription_status VARCHAR(20), -- active, canceled, expired
  subscription_id VARCHAR(100), -- Stripe subscription ID
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  last_login_at TIMESTAMPTZ
);

CREATE INDEX idx_profiles_email ON profiles(email);
CREATE INDEX idx_profiles_role ON profiles(role);
```

### `organizations`
**Purpose:** Company/team accounts for enterprise

```sql
CREATE TABLE organizations (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  name VARCHAR(200) NOT NULL,
  owner_id UUID REFERENCES profiles(id) ON DELETE SET NULL,
  plan VARCHAR(20) DEFAULT 'free', -- free, team, enterprise
  credits_pool INTEGER DEFAULT 0,
  max_members INTEGER DEFAULT 5,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

### `organization_members`
**Purpose:** Users belonging to organizations

```sql
CREATE TABLE organization_members (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE,
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  role VARCHAR(20) DEFAULT 'member', -- owner, admin, member
  joined_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(organization_id, user_id)
);
```

---

## AUTHENTICATION & USERS

### `auth.users` (Supabase Built-in)
**Purpose:** Core authentication

```sql
-- Supabase managed table
-- Fields: id, email, encrypted_password, email_confirmed_at, etc.
```

### `user_sessions`
**Purpose:** Track active user sessions

```sql
CREATE TABLE user_sessions (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  token VARCHAR(255) UNIQUE NOT NULL,
  ip_address INET,
  user_agent TEXT,
  expires_at TIMESTAMPTZ NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_sessions_user ON user_sessions(user_id);
CREATE INDEX idx_sessions_token ON user_sessions(token);
```

### `user_activity_log`
**Purpose:** Audit trail of user actions

```sql
CREATE TABLE user_activity_log (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  action_type VARCHAR(50) NOT NULL,
  resource_type VARCHAR(50),
  resource_id UUID,
  details JSONB,
  ip_address INET,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_activity_user ON user_activity_log(user_id, created_at DESC);
CREATE INDEX idx_activity_type ON user_activity_log(action_type);
```

---

## CREDITS & PAYMENTS

### `credits`
**Purpose:** Credit balance and transaction history

```sql
CREATE TABLE credits (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  amount INTEGER NOT NULL,
  type VARCHAR(20) NOT NULL, -- purchase, usage, refund, bonus
  description TEXT,
  balance_after INTEGER NOT NULL,
  metadata JSONB,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_credits_user ON credits(user_id, created_at DESC);
```

### `transactions`
**Purpose:** Payment transactions (Stripe/PayPal)

```sql
CREATE TABLE transactions (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  provider VARCHAR(20) NOT NULL, -- stripe, paypal
  provider_transaction_id VARCHAR(255) UNIQUE,
  type VARCHAR(20) NOT NULL, -- purchase, subscription, refund
  amount_cents INTEGER NOT NULL,
  currency VARCHAR(3) DEFAULT 'USD',
  status VARCHAR(20) NOT NULL, -- pending, completed, failed, refunded
  credits_granted INTEGER,
  metadata JSONB,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_transactions_user ON transactions(user_id);
CREATE INDEX idx_transactions_provider ON transactions(provider_transaction_id);
```

### `subscriptions`
**Purpose:** Subscription management

```sql
CREATE TABLE subscriptions (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  provider VARCHAR(20) NOT NULL, -- stripe, paypal
  provider_subscription_id VARCHAR(255) UNIQUE NOT NULL,
  tier VARCHAR(20) NOT NULL, -- light, standard, advanced, premium
  status VARCHAR(20) NOT NULL, -- active, canceled, expired, past_due
  current_period_start TIMESTAMPTZ,
  current_period_end TIMESTAMPTZ,
  cancel_at_period_end BOOLEAN DEFAULT false,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_subscriptions_user ON subscriptions(user_id);
CREATE INDEX idx_subscriptions_provider ON subscriptions(provider_subscription_id);
```

---

## JAVARI AI TABLES

### `javari_projects`
**Purpose:** Organize work into projects

```sql
CREATE TABLE javari_projects (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  name VARCHAR(200) NOT NULL,
  description TEXT,
  parent_project_id UUID REFERENCES javari_projects(id) ON DELETE CASCADE,
  status VARCHAR(20) DEFAULT 'active', -- active, archived, completed
  metadata JSONB,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_projects_user ON javari_projects(user_id);
CREATE INDEX idx_projects_parent ON javari_projects(parent_project_id);
```

### `javari_conversations`
**Purpose:** Store all chat history with Javari

```sql
CREATE TABLE javari_conversations (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  project_id UUID REFERENCES javari_projects(id) ON DELETE SET NULL,
  provider VARCHAR(20) NOT NULL, -- openai, anthropic, google, perplexity
  model VARCHAR(50) NOT NULL,
  messages JSONB NOT NULL,
  response TEXT,
  tokens_used INTEGER,
  cost_cents INTEGER,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_conversations_user ON javari_conversations(user_id, created_at DESC);
CREATE INDEX idx_conversations_project ON javari_conversations(project_id);
```

### `javari_credentials`
**Purpose:** Encrypted credential storage for automation

```sql
CREATE TABLE javari_credentials (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  service VARCHAR(50) NOT NULL, -- github, vercel, stripe, etc.
  encrypted_credentials TEXT NOT NULL,
  encryption_key_id VARCHAR(50) NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  last_used_at TIMESTAMPTZ,
  UNIQUE(user_id, service)
);

CREATE INDEX idx_credentials_user ON javari_credentials(user_id);
```

### `javari_learning`
**Purpose:** Track patterns for autonomous learning

```sql
CREATE TABLE javari_learning (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  conversation_type VARCHAR(50) NOT NULL,
  message_count INTEGER,
  avg_message_length INTEGER,
  response_quality_score DECIMAL(3,2),
  provider VARCHAR(20),
  model VARCHAR(50),
  tokens_used INTEGER,
  cost_cents INTEGER,
  patterns_detected JSONB,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_learning_user ON javari_learning(user_id);
CREATE INDEX idx_learning_type ON javari_learning(conversation_type);
```

### `javari_usage_stats`
**Purpose:** Detailed usage tracking per request

```sql
CREATE TABLE javari_usage_stats (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  provider VARCHAR(20) NOT NULL,
  model VARCHAR(50) NOT NULL,
  input_tokens INTEGER,
  output_tokens INTEGER,
  total_tokens INTEGER,
  cost_cents INTEGER,
  success BOOLEAN NOT NULL,
  error_message TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_usage_user ON javari_usage_stats(user_id, created_at DESC);
CREATE INDEX idx_usage_provider ON javari_usage_stats(provider);
```

---

## BOT SYSTEM TABLES

### `bots`
**Purpose:** Bot definitions and configuration

```sql
CREATE TABLE bots (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  name VARCHAR(50) UNIQUE NOT NULL,
  display_name VARCHAR(100) NOT NULL,
  description TEXT,
  schedule_cron VARCHAR(50) NOT NULL,
  api_endpoint VARCHAR(200) NOT NULL,
  status VARCHAR(20) DEFAULT 'active',
  last_execution_at TIMESTAMPTZ,
  next_execution_at TIMESTAMPTZ,
  total_runs INTEGER DEFAULT 0,
  successful_runs INTEGER DEFAULT 0,
  failed_runs INTEGER DEFAULT 0,
  avg_execution_time_ms INTEGER,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_bots_status ON bots(status);
```

### `bot_executions`
**Purpose:** Log every bot execution

```sql
CREATE TABLE bot_executions (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  bot_id UUID REFERENCES bots(id) ON DELETE CASCADE,
  status VARCHAR(20) NOT NULL,
  started_at TIMESTAMPTZ NOT NULL,
  completed_at TIMESTAMPTZ,
  execution_time_ms INTEGER,
  checks_performed INTEGER DEFAULT 0,
  issues_found INTEGER DEFAULT 0,
  actions_taken INTEGER DEFAULT 0,
  error_message TEXT,
  metadata JSONB,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_executions_bot ON bot_executions(bot_id, started_at DESC);
```

### `bot_findings`
**Purpose:** Issues detected by bots

```sql
CREATE TABLE bot_findings (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  bot_id UUID REFERENCES bots(id) ON DELETE CASCADE,
  execution_id UUID REFERENCES bot_executions(id) ON DELETE CASCADE,
  severity VARCHAR(20) NOT NULL,
  category VARCHAR(50) NOT NULL,
  title VARCHAR(200) NOT NULL,
  description TEXT,
  affected_component VARCHAR(100),
  detected_at TIMESTAMPTZ DEFAULT NOW(),
  resolved_at TIMESTAMPTZ,
  resolution_method VARCHAR(100),
  metadata JSONB
);

CREATE INDEX idx_findings_bot ON bot_findings(bot_id, detected_at DESC);
CREATE INDEX idx_findings_severity ON bot_findings(severity, resolved_at);
```

### `bot_actions`
**Purpose:** Actions taken by bots

```sql
CREATE TABLE bot_actions (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  bot_id UUID REFERENCES bots(id) ON DELETE CASCADE,
  finding_id UUID REFERENCES bot_findings(id) ON DELETE SET NULL,
  action_type VARCHAR(50) NOT NULL,
  action_description TEXT NOT NULL,
  success BOOLEAN NOT NULL,
  error_message TEXT,
  taken_at TIMESTAMPTZ DEFAULT NOW(),
  metadata JSONB
);

CREATE INDEX idx_actions_bot ON bot_actions(bot_id, taken_at DESC);
```

### `bot_tickets`
**Purpose:** Escalations to Javari or humans

```sql
CREATE TABLE bot_tickets (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  bot_id UUID REFERENCES bots(id) ON DELETE CASCADE,
  finding_id UUID REFERENCES bot_findings(id) ON DELETE CASCADE,
  severity VARCHAR(20) NOT NULL,
  title VARCHAR(200) NOT NULL,
  description TEXT NOT NULL,
  status VARCHAR(20) DEFAULT 'open',
  assigned_to VARCHAR(50),
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  resolved_at TIMESTAMPTZ,
  resolution_notes TEXT,
  metadata JSONB
);

CREATE INDEX idx_tickets_status ON bot_tickets(status, severity);
```

### `bot_knowledge`
**Purpose:** Self-healing knowledge base

```sql
CREATE TABLE bot_knowledge (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  category VARCHAR(50) NOT NULL,
  issue_pattern TEXT NOT NULL,
  solution TEXT NOT NULL,
  success_rate DECIMAL(5,2) DEFAULT 0.00,
  times_applied INTEGER DEFAULT 0,
  times_successful INTEGER DEFAULT 0,
  last_used_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_knowledge_category ON bot_knowledge(category);
```

---

## APPLICATIONS & TOOLS

### `app_catalog`
**Purpose:** All 60+ applications in platform

```sql
CREATE TABLE app_catalog (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  slug VARCHAR(100) UNIQUE NOT NULL,
  name VARCHAR(200) NOT NULL,
  description TEXT,
  category VARCHAR(50) NOT NULL,
  icon_url TEXT,
  credit_cost INTEGER NOT NULL,
  status VARCHAR(20) DEFAULT 'active', -- active, beta, maintenance, deprecated
  deployment_url TEXT,
  repository_url TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_apps_category ON app_catalog(category);
CREATE INDEX idx_apps_status ON app_catalog(status);
```

### `app_usage`
**Purpose:** Track application usage

```sql
CREATE TABLE app_usage (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  app_id UUID REFERENCES app_catalog(id) ON DELETE CASCADE,
  credits_used INTEGER NOT NULL,
  execution_time_ms INTEGER,
  success BOOLEAN NOT NULL,
  error_message TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_usage_user ON app_usage(user_id, created_at DESC);
CREATE INDEX idx_usage_app ON app_usage(app_id, created_at DESC);
```

### `tools_catalog`
**Purpose:** Individual tools within apps

```sql
CREATE TABLE tools_catalog (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  app_id UUID REFERENCES app_catalog(id) ON DELETE CASCADE,
  slug VARCHAR(100) NOT NULL,
  name VARCHAR(200) NOT NULL,
  description TEXT,
  credit_cost INTEGER NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(app_id, slug)
);
```

### `tools_usage`
**Purpose:** Track tool usage

```sql
CREATE TABLE tools_usage (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  tool_id UUID REFERENCES tools_catalog(id) ON DELETE CASCADE,
  credits_used INTEGER NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_tools_usage_user ON tools_usage(user_id, created_at DESC);
```

### `tools_results`
**Purpose:** Store generated content

```sql
CREATE TABLE tools_results (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  tool_id UUID REFERENCES tools_catalog(id) ON DELETE CASCADE,
  input_data JSONB,
  output_data JSONB,
  output_url TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_results_user ON tools_results(user_id, created_at DESC);
```

---

## GAMES PLATFORM

### `games_catalog`
**Purpose:** All 1,200+ games

```sql
CREATE TABLE games_catalog (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  slug VARCHAR(100) UNIQUE NOT NULL,
  name VARCHAR(200) NOT NULL,
  description TEXT,
  category VARCHAR(50) NOT NULL,
  thumbnail_url TEXT,
  is_premium BOOLEAN DEFAULT false,
  credit_cost INTEGER DEFAULT 0,
  rating DECIMAL(3,2),
  play_count INTEGER DEFAULT 0,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_games_category ON games_catalog(category);
CREATE INDEX idx_games_premium ON games_catalog(is_premium);
```

### `game_sessions`
**Purpose:** Track game plays

```sql
CREATE TABLE game_sessions (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  game_id UUID REFERENCES games_catalog(id) ON DELETE CASCADE,
  started_at TIMESTAMPTZ NOT NULL,
  ended_at TIMESTAMPTZ,
  duration_seconds INTEGER,
  score INTEGER,
  completed BOOLEAN DEFAULT false,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_sessions_user ON game_sessions(user_id, started_at DESC);
CREATE INDEX idx_sessions_game ON game_sessions(game_id);
```

### `game_leaderboards`
**Purpose:** High scores and rankings

```sql
CREATE TABLE game_leaderboards (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  game_id UUID REFERENCES games_catalog(id) ON DELETE CASCADE,
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  score INTEGER NOT NULL,
  rank INTEGER,
  achieved_at TIMESTAMPTZ NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(game_id, user_id)
);

CREATE INDEX idx_leaderboard_game ON game_leaderboards(game_id, score DESC);
```

---

## JAVARIVERSE

### `avatars`
**Purpose:** User avatars in virtual world

```sql
CREATE TABLE avatars (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  name VARCHAR(100) NOT NULL,
  appearance JSONB NOT NULL, -- 3D model config
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_avatars_user ON avatars(user_id);
```

### `virtual_spaces`
**Purpose:** Javariverse locations

```sql
CREATE TABLE virtual_spaces (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  name VARCHAR(200) NOT NULL,
  type VARCHAR(50) NOT NULL,
  capacity INTEGER DEFAULT 100,
  owner_id UUID REFERENCES profiles(id) ON DELETE SET NULL,
  is_public BOOLEAN DEFAULT true,
  coordinates JSONB,
  metadata JSONB,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### `space_visits`
**Purpose:** Track avatar presence in spaces

```sql
CREATE TABLE space_visits (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  avatar_id UUID REFERENCES avatars(id) ON DELETE CASCADE,
  space_id UUID REFERENCES virtual_spaces(id) ON DELETE CASCADE,
  entered_at TIMESTAMPTZ NOT NULL,
  exited_at TIMESTAMPTZ,
  duration_seconds INTEGER
);

CREATE INDEX idx_visits_space ON space_visits(space_id, entered_at DESC);
```

---

## SOCIAL IMPACT MODULES

### `first_responders`
**Purpose:** First Responders Haven module

```sql
CREATE TABLE first_responders (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  role VARCHAR(50) NOT NULL, -- firefighter, police, ems, dispatcher
  agency VARCHAR(200),
  verified BOOLEAN DEFAULT false,
  verification_doc_url TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### `support_groups`
**Purpose:** Peer support groups

```sql
CREATE TABLE support_groups (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  name VARCHAR(200) NOT NULL,
  module VARCHAR(50) NOT NULL,
  is_private BOOLEAN DEFAULT true,
  max_members INTEGER DEFAULT 20,
  created_by UUID REFERENCES profiles(id) ON DELETE SET NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### `group_members`
**Purpose:** Support group membership

```sql
CREATE TABLE group_members (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  group_id UUID REFERENCES support_groups(id) ON DELETE CASCADE,
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  role VARCHAR(20) DEFAULT 'member',
  joined_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(group_id, user_id)
);
```

---

## ANALYTICS & LOGS

### `page_views`
**Purpose:** Track page views

```sql
CREATE TABLE page_views (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  page_path VARCHAR(255) NOT NULL,
  referrer TEXT,
  user_agent TEXT,
  ip_address INET,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_pageviews_path ON page_views(page_path, created_at DESC);
```

### `api_logs`
**Purpose:** Track API requests

```sql
CREATE TABLE api_logs (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES profiles(id) ON DELETE SET NULL,
  endpoint VARCHAR(255) NOT NULL,
  method VARCHAR(10) NOT NULL,
  status_code INTEGER NOT NULL,
  response_time_ms INTEGER,
  ip_address INET,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_api_logs_endpoint ON api_logs(endpoint, created_at DESC);
```

### `error_logs`
**Purpose:** Application errors

```sql
CREATE TABLE error_logs (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES profiles(id) ON DELETE SET NULL,
  error_type VARCHAR(100) NOT NULL,
  error_message TEXT NOT NULL,
  stack_trace TEXT,
  context JSONB,
  ip_address INET,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_errors_type ON error_logs(error_type, created_at DESC);
```

---

## ROW LEVEL SECURITY (RLS)

### Standard Patterns

**User-scoped data:**
```sql
-- Example: profiles table
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can view own profile"
  ON profiles FOR SELECT
  USING (auth.uid() = id);

CREATE POLICY "Users can update own profile"
  ON profiles FOR UPDATE
  USING (auth.uid() = id);
```

**Admin-only data:**
```sql
-- Example: bot_executions
CREATE POLICY "Only admins can view bot executions"
  ON bot_executions FOR SELECT
  USING (
    EXISTS (
      SELECT 1 FROM profiles
      WHERE id = auth.uid() AND role IN ('admin', 'owner')
    )
  );
```

**Public read, authenticated write:**
```sql
-- Example: app_catalog
CREATE POLICY "Anyone can view apps"
  ON app_catalog FOR SELECT
  USING (status = 'active');

CREATE POLICY "Authenticated can track usage"
  ON app_usage FOR INSERT
  WITH CHECK (auth.uid() IS NOT NULL);
```

---

## FUNCTIONS & TRIGGERS

### Update Timestamps

```sql
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Apply to tables
CREATE TRIGGER update_profiles_updated_at
  BEFORE UPDATE ON profiles
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
```

### Calculate Credit Balances

```sql
CREATE OR REPLACE FUNCTION get_user_credits(user_uuid UUID)
RETURNS INTEGER AS $$
  SELECT COALESCE(SUM(amount), 0)::INTEGER
  FROM credits
  WHERE user_id = user_uuid;
$$ LANGUAGE sql STABLE;
```

---

## BACKUP & MAINTENANCE

### Automated Backups
- Supabase: Daily automatic backups
- Retention: 30 days
- Point-in-time recovery: Available

### Maintenance Windows
- Weekly: Sunday 2-4 AM EST
- Database optimization
- Index rebuilding
- Vacuuming

---

## PERFORMANCE OPTIMIZATION

### Key Indexes Created

**High-Traffic Queries:**
- User lookups: `idx_profiles_email`
- Credit balance: Optimized with function
- Bot monitoring: `idx_executions_bot`
- App usage: `idx_usage_user`, `idx_usage_app`

### Query Performance Targets
- Simple queries: <50ms
- Complex queries: <500ms
- Aggregations: <2 seconds

---

## MIGRATION HISTORY

**V1.0:** Initial schema (October 2024)
**V2.0:** Added bot system (November 2024)
**V3.0:** Added Javari tables (November 2024)
**V4.0:** Added games platform (November 2024)
**V5.0:** Current schema (November 2024)

---

## SUMMARY

**Total Tables:** 80+
**Total Indexes:** 100+
**RLS Policies:** 150+
**Functions:** 10+
**Triggers:** 20+

**Status:** Fully operational, production-ready

---

**END OF DATABASE SCHEMA DOCUMENTATION**

**Last Updated:** November 21, 2025 - 4:13 PM EST
