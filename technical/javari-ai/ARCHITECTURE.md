# Javari AI - Technical Architecture

**Last Updated:** November 8, 2025  
**Version:** 1.0  
**Repository:** [CR-AudioViz-AI/javari-javari](https://github.com/CR-AudioViz-AI/javari-javari)  
**Technical Lead:** Roy Henderson (CEO/CTO)

---

## SYSTEM OVERVIEW

Javari AI is a Next.js 14 application built with TypeScript, deployed on Vercel, using Supabase for database and authentication. The architecture is designed for autonomous operation with self-healing capabilities, continuous learning, and zero-downtime deployments.

### Technology Stack

| Layer | Technology | Version | Purpose |
|-------|-----------|---------|---------|
| **Frontend** | Next.js | 14.1.0 | React framework with App Router |
| | React | 18+ | UI library |
| | TypeScript | 5.x | Type safety |
| | Tailwind CSS | 3.x | Styling framework |
| | shadcn/ui | latest | Component library |
| **Backend** | Next.js API Routes | 14.1.0 | Serverless functions |
| | Supabase | latest | PostgreSQL + Auth |
| **Database** | PostgreSQL | 15 | Relational database |
| | pgvector | latest | Vector embeddings |
| | pgcrypto | latest | Encryption |
| **AI** | OpenAI GPT-4 | latest | Primary reasoning |
| | Claude Sonnet 4.5 | latest | Alternative/fallback |
| | Google Gemini | latest | Specialized tasks |
| | Perplexity AI | latest | Research/data |
| **Deployment** | Vercel | latest | Hosting + CI/CD |
| | GitHub | latest | Version control |

---

## DEPLOYMENT ARCHITECTURE

### Vercel Configuration

**Preview-Only Mode:**
```json
{
  "$schema": "https://openapi.vercel.sh/vercel.json",
  "github": {
    "autoAlias": false,
    "silent": true
  }
}
```

**Why Preview-Only:**
- Cost control: Production deploys are expensive
- Manual control: Roy approves production manually
- Testing: All changes tested in preview first
- Rollback: Easy to revert if issues found

**Current URLs:**
- Preview: `javari-javari-77b6mtat5-roy-hendersons-projects-1d3d5e94.vercel.app`
- Production (planned): `javariai.com`

### Environment Variables

**Required for All Environments:**
```bash
# Supabase
NEXT_PUBLIC_SUPABASE_URL=https://[project-id].supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=[anon-key]
SUPABASE_SERVICE_ROLE_KEY=[service-role-key]

# OpenAI
OPENAI_API_KEY=[openai-api-key]

# Credential Encryption
CREDENTIAL_ENCRYPTION_KEY=[32-byte-base64-string]

# Vercel (for self-healing)
VERCEL_TOKEN=[vercel-token]
VERCEL_TEAM_ID=[team-id]

# GitHub (for autonomous commits)
GITHUB_TOKEN=[github-token]

# Optional: Additional AI Providers
ANTHROPIC_API_KEY=[anthropic-api-key]
GOOGLE_API_KEY=[google-api-key]
PERPLEXITY_API_KEY=[perplexity-api-key]
```

**Note:** Actual credential values are stored in 1Password vault: "CR AudioViz AI - Javari Production Secrets"

**Generating Encryption Key:**
```bash
node -e "console.log(require('crypto').randomBytes(32).toString('base64'))"
```

### Cron Jobs (Autonomous Operations)

**Build Health Monitor:**
```json
{
  "crons": [
    {
      "path": "/api/javari/cron/health-check",
      "schedule": "*/30 * * * *"
    }
  ]
}
```
- Runs every 30 minutes
- Checks all monitored projects for build failures
- Attempts auto-fix if errors detected
- Sends alerts if auto-fix fails

**Web Crawler (Knowledge Updates):**
```json
{
  "path": "/api/javari/cron/web-crawl",
  "schedule": "0 2 * * *"
}
```
- Runs daily at 2 AM UTC
- Crawls configured documentation sites
- Extracts text and generates embeddings
- Updates knowledge base automatically

---

## DATABASE ARCHITECTURE

### Connection Details

**Supabase Project:**
- URL: `https://kteobfyferrukqeolofj.supabase.co`
- Region: US East (Ohio)
- Database: PostgreSQL 15
- Connection Pooling: PgBouncer enabled

**Extensions Enabled:**
```sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";      -- UUID generation
CREATE EXTENSION IF NOT EXISTS "pgcrypto";        -- Encryption functions
CREATE EXTENSION IF NOT EXISTS "vector";          -- Vector embeddings
CREATE EXTENSION IF NOT EXISTS "pg_stat_statements"; -- Query performance
```

### Complete Schema (33 Tables)

#### **Core System Tables (5)**

**users**
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  email TEXT UNIQUE NOT NULL,
  name TEXT,
  avatar_url TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  last_active_at TIMESTAMPTZ DEFAULT NOW(),
  plan TEXT DEFAULT 'free' CHECK (plan IN ('free', 'pro', 'enterprise')),
  credits_balance DECIMAL(10,2) DEFAULT 10.00,
  monthly_allowance DECIMAL(10,2) DEFAULT 10.00,
  allowance_reset_date TIMESTAMPTZ DEFAULT (NOW() + INTERVAL '1 month')
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_plan ON users(plan);
```

**user_preferences**
```sql
CREATE TABLE user_preferences (
  user_id UUID PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
  default_ai_provider TEXT DEFAULT 'gpt-4',
  temperature DECIMAL(3,2) DEFAULT 0.7 CHECK (temperature >= 0 AND temperature <= 2),
  max_tokens INTEGER DEFAULT 2000,
  stream_responses BOOLEAN DEFAULT TRUE,
  voice_enabled BOOLEAN DEFAULT TRUE,
  theme TEXT DEFAULT 'dark' CHECK (theme IN ('light', 'dark', 'system')),
  notifications_enabled BOOLEAN DEFAULT TRUE,
  autonomy_level TEXT DEFAULT 'approve_critical' CHECK (autonomy_level IN ('full_auto', 'approve_critical', 'manual')),
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

#### **Project Management (2)**

**projects**
```sql
CREATE TABLE projects (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  description TEXT,
  repository_url TEXT,
  metadata JSONB DEFAULT '{}',
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_projects_user_id ON projects(user_id);
```

**subprojects**
```sql
CREATE TABLE subprojects (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  description TEXT,
  metadata JSONB DEFAULT '{}',
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_subprojects_project_id ON subprojects(project_id);
```

#### **Chat System (4)**

**conversations**
```sql
CREATE TABLE conversations (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  project_id UUID REFERENCES projects(id) ON DELETE SET NULL,
  subproject_id UUID REFERENCES subprojects(id) ON DELETE SET NULL,
  title TEXT NOT NULL,
  starred BOOLEAN DEFAULT FALSE,
  status TEXT DEFAULT 'active' CHECK (status IN ('active', 'archived')),
  last_message_at TIMESTAMPTZ DEFAULT NOW(),
  total_tokens INTEGER DEFAULT 0,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_conversations_user_id ON conversations(user_id);
CREATE INDEX idx_conversations_project_id ON conversations(project_id);
CREATE INDEX idx_conversations_status ON conversations(status);
```

**messages**
```sql
CREATE TABLE messages (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  conversation_id UUID NOT NULL REFERENCES conversations(id) ON DELETE CASCADE,
  role TEXT NOT NULL CHECK (role IN ('user', 'assistant', 'system')),
  content TEXT NOT NULL,
  ai_model TEXT,
  tokens_used INTEGER,
  credits_cost DECIMAL(10,2),
  reasoning_log TEXT,
  confidence_score DECIMAL(3,2),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_messages_conversation_id ON messages(conversation_id);
CREATE INDEX idx_messages_created_at ON messages(created_at);
```

**conversation_continuations**
```sql
CREATE TABLE conversation_continuations (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  parent_id UUID NOT NULL REFERENCES conversations(id) ON DELETE CASCADE,
  child_id UUID NOT NULL REFERENCES conversations(id) ON DELETE CASCADE,
  summary TEXT,
  context_tokens INTEGER,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(child_id)
);

CREATE INDEX idx_continuations_parent_id ON conversation_continuations(parent_id);
CREATE INDEX idx_continuations_child_id ON conversation_continuations(child_id);
```

**artifacts**
```sql
CREATE TABLE artifacts (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  conversation_id UUID NOT NULL REFERENCES conversations(id) ON DELETE CASCADE,
  message_id UUID REFERENCES messages(id) ON DELETE SET NULL,
  type TEXT NOT NULL CHECK (type IN ('code', 'document', 'image', 'data')),
  filename TEXT NOT NULL,
  file_path TEXT,
  file_size BIGINT,
  language TEXT,
  preview_data JSONB,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_artifacts_conversation_id ON artifacts(conversation_id);
```

#### **Credential Vault (3)**

**javari_credentials**
```sql
CREATE TABLE javari_credentials (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  project_id UUID REFERENCES projects(id) ON DELETE CASCADE,
  subproject_id UUID REFERENCES subprojects(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  description TEXT,
  encrypted_value TEXT NOT NULL,
  encryption_iv TEXT NOT NULL,
  encryption_tag TEXT NOT NULL,
  credential_type TEXT CHECK (credential_type IN (
    'github_token', 'vercel_token', 'api_key', 'database_url',
    'webhook_secret', 'oauth_client', 'custom'
  )),
  is_inherited BOOLEAN DEFAULT FALSE,
  allow_override BOOLEAN DEFAULT TRUE,
  last_used_at TIMESTAMPTZ,
  use_count INTEGER DEFAULT 0,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  CHECK (
    (project_id IS NOT NULL AND subproject_id IS NULL) OR
    (project_id IS NULL AND subproject_id IS NOT NULL)
  ),
  UNIQUE(user_id, project_id, subproject_id, name)
);

CREATE INDEX idx_javari_credentials_user_id ON javari_credentials(user_id);
CREATE INDEX idx_javari_credentials_project_id ON javari_credentials(project_id);
```

**javari_credential_snapshots**
```sql
CREATE TABLE javari_credential_snapshots (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  conversation_id UUID NOT NULL REFERENCES conversations(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  project_id UUID REFERENCES projects(id) ON DELETE SET NULL,
  subproject_id UUID REFERENCES subprojects(id) ON DELETE SET NULL,
  credentials_snapshot JSONB NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_credential_snapshots_conversation_id ON javari_credential_snapshots(conversation_id);
```

**javari_credential_rotations**
```sql
CREATE TABLE javari_credential_rotations (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  credential_id UUID NOT NULL REFERENCES javari_credentials(id) ON DELETE CASCADE,
  old_value_hash TEXT NOT NULL,
  new_value_hash TEXT NOT NULL,
  rotation_type TEXT DEFAULT 'manual' CHECK (rotation_type IN ('manual', 'scheduled', 'emergency')),
  grace_period_end TIMESTAMPTZ,
  rotated_by UUID REFERENCES users(id) ON DELETE SET NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_credential_rotations_credential_id ON javari_credential_rotations(credential_id);
```

#### **Knowledge & RAG (2)**

**javari_knowledge_base**
```sql
CREATE TABLE javari_knowledge_base (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  project_id UUID REFERENCES projects(id) ON DELETE CASCADE,
  content TEXT NOT NULL,
  content_hash TEXT NOT NULL,
  source_type TEXT CHECK (source_type IN ('file_upload', 'web_crawl', 'api_integration', 'manual')),
  source_url TEXT,
  embedding vector(1536),
  relevance_score DECIMAL(3,2),
  last_accessed_at TIMESTAMPTZ,
  access_count INTEGER DEFAULT 0,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(content_hash, project_id)
);

CREATE INDEX idx_knowledge_user_id ON javari_knowledge_base(user_id);
CREATE INDEX idx_knowledge_project_id ON javari_knowledge_base(project_id);
CREATE INDEX idx_knowledge_embedding ON javari_knowledge_base USING ivfflat (embedding vector_cosine_ops);
```

**javari_web_crawls**
```sql
CREATE TABLE javari_web_crawls (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  url TEXT NOT NULL,
  crawl_frequency TEXT DEFAULT 'daily' CHECK (crawl_frequency IN ('hourly', 'daily', 'weekly', 'manual')),
  last_crawled_at TIMESTAMPTZ,
  next_crawl_at TIMESTAMPTZ,
  pages_indexed INTEGER DEFAULT 0,
  errors_encountered INTEGER DEFAULT 0,
  status TEXT DEFAULT 'pending' CHECK (status IN ('pending', 'in_progress', 'completed', 'failed')),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_web_crawls_next_crawl ON javari_web_crawls(next_crawl_at);
```

#### **Learning System (2)**

**javari_learned_patterns**
```sql
CREATE TABLE javari_learned_patterns (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  pattern_type TEXT NOT NULL,
  context TEXT NOT NULL,
  action_taken TEXT NOT NULL,
  success_rate DECIMAL(3,2),
  confidence_score DECIMAL(3,2),
  usage_count INTEGER DEFAULT 1,
  last_used_at TIMESTAMPTZ DEFAULT NOW(),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_learned_patterns_type ON javari_learned_patterns(pattern_type);
CREATE INDEX idx_learned_patterns_success_rate ON javari_learned_patterns(success_rate DESC);
```

**javari_ab_tests**
```sql
CREATE TABLE javari_ab_tests (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  test_name TEXT NOT NULL UNIQUE,
  variant_a TEXT NOT NULL,
  variant_b TEXT NOT NULL,
  variant_a_count INTEGER DEFAULT 0,
  variant_b_count INTEGER DEFAULT 0,
  variant_a_success INTEGER DEFAULT 0,
  variant_b_success INTEGER DEFAULT 0,
  winner TEXT CHECK (winner IN ('a', 'b', 'inconclusive')),
  confidence_level DECIMAL(3,2),
  started_at TIMESTAMPTZ DEFAULT NOW(),
  ended_at TIMESTAMPTZ
);
```

#### **Monitoring & Alerts (3)**

**javari_health_monitors**
```sql
CREATE TABLE javari_health_monitors (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  project_id UUID REFERENCES projects(id) ON DELETE CASCADE,
  monitor_type TEXT CHECK (monitor_type IN ('build', 'deployment', 'uptime', 'performance')),
  check_frequency TEXT DEFAULT '30min' CHECK (check_frequency IN ('5min', '15min', '30min', '1hour', 'daily')),
  alert_threshold INTEGER,
  auto_fix_enabled BOOLEAN DEFAULT TRUE,
  slack_webhook TEXT,
  email_recipients TEXT[],
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_health_monitors_user_id ON javari_health_monitors(user_id);
CREATE INDEX idx_health_monitors_is_active ON javari_health_monitors(is_active);
```

**javari_alerts**
```sql
CREATE TABLE javari_alerts (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  monitor_id UUID NOT NULL REFERENCES javari_health_monitors(id) ON DELETE CASCADE,
  severity TEXT CHECK (severity IN ('low', 'medium', 'high', 'critical')),
  alert_message TEXT NOT NULL,
  auto_fix_attempted BOOLEAN DEFAULT FALSE,
  fix_success BOOLEAN,
  resolution_time_seconds INTEGER,
  acknowledged_at TIMESTAMPTZ,
  acknowledged_by UUID REFERENCES users(id) ON DELETE SET NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_alerts_monitor_id ON javari_alerts(monitor_id);
CREATE INDEX idx_alerts_severity ON javari_alerts(severity);
CREATE INDEX idx_alerts_created_at ON javari_alerts(created_at DESC);
```

**javari_healing_events**
```sql
CREATE TABLE javari_healing_events (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  project_id UUID REFERENCES projects(id) ON DELETE CASCADE,
  error_type TEXT NOT NULL,
  error_details TEXT,
  healing_action_taken TEXT,
  healing_successful BOOLEAN,
  time_to_heal_seconds INTEGER,
  automated BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_healing_events_project_id ON javari_healing_events(project_id);
CREATE INDEX idx_healing_events_success ON javari_healing_events(healing_successful);
```

#### **Autonomous Operations (3)**

**javari_task_queue**
```sql
CREATE TABLE javari_task_queue (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  project_id UUID REFERENCES projects(id) ON DELETE CASCADE,
  task_type TEXT NOT NULL,
  priority INTEGER DEFAULT 5 CHECK (priority BETWEEN 1 AND 10),
  payload JSONB NOT NULL,
  status TEXT DEFAULT 'pending' CHECK (status IN ('pending', 'in_progress', 'completed', 'failed', 'cancelled')),
  scheduled_at TIMESTAMPTZ DEFAULT NOW(),
  started_at TIMESTAMPTZ,
  completed_at TIMESTAMPTZ,
  error_message TEXT,
  retry_count INTEGER DEFAULT 0,
  max_retries INTEGER DEFAULT 3,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_task_queue_status ON javari_task_queue(status);
CREATE INDEX idx_task_queue_scheduled_at ON javari_task_queue(scheduled_at);
```

**javari_agents**
```sql
CREATE TABLE javari_agents (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  agent_name TEXT NOT NULL UNIQUE,
  specialization TEXT,
  is_active BOOLEAN DEFAULT TRUE,
  current_task_id UUID REFERENCES javari_task_queue(id) ON DELETE SET NULL,
  tasks_completed INTEGER DEFAULT 0,
  success_rate DECIMAL(3,2),
  average_completion_time_seconds INTEGER,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  last_active_at TIMESTAMPTZ DEFAULT NOW()
);
```

**javari_agent_collaboration**
```sql
CREATE TABLE javari_agent_collaboration (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  task_id UUID NOT NULL REFERENCES javari_task_queue(id) ON DELETE CASCADE,
  agent_ids UUID[],
  collaboration_type TEXT CHECK (collaboration_type IN ('parallel', 'sequential', 'peer_review')),
  final_decision TEXT,
  decision_rationale TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_collaboration_task_id ON javari_agent_collaboration(task_id);
```

#### **Cost Tracking (2)**

**javari_cost_tracking**
```sql
CREATE TABLE javari_cost_tracking (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  project_id UUID REFERENCES projects(id) ON DELETE CASCADE,
  service TEXT NOT NULL,
  feature_used TEXT,
  input_tokens INTEGER,
  output_tokens INTEGER,
  cost_usd DECIMAL(10,4),
  timestamp TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_cost_tracking_user_id ON javari_cost_tracking(user_id);
CREATE INDEX idx_cost_tracking_project_id ON javari_cost_tracking(project_id);
CREATE INDEX idx_cost_tracking_timestamp ON javari_cost_tracking(timestamp DESC);
```

**javari_cost_forecasts**
```sql
CREATE TABLE javari_cost_forecasts (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  forecast_period TEXT CHECK (forecast_period IN ('daily', 'weekly', 'monthly', 'yearly')),
  predicted_cost_usd DECIMAL(10,2),
  confidence_interval DECIMAL(3,2),
  cost_breakdown_json JSONB,
  recommendations TEXT[],
  generated_at TIMESTAMPTZ DEFAULT NOW()
);
```

#### **Compliance (2)**

**javari_pii_detections**
```sql
CREATE TABLE javari_pii_detections (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  conversation_id UUID NOT NULL REFERENCES conversations(id) ON DELETE CASCADE,
  message_id UUID REFERENCES messages(id) ON DELETE CASCADE,
  pii_type TEXT CHECK (pii_type IN ('email', 'phone', 'ssn', 'credit_card', 'address')),
  detected_text_hash TEXT NOT NULL,
  redaction_applied BOOLEAN DEFAULT FALSE,
  reported_to_admin BOOLEAN DEFAULT FALSE,
  detected_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_pii_detections_conversation_id ON javari_pii_detections(conversation_id);
```

**javari_security_scans**
```sql
CREATE TABLE javari_security_scans (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  scan_type TEXT CHECK (scan_type IN ('dependency', 'code', 'secrets', 'license')),
  vulnerabilities_found INTEGER DEFAULT 0,
  severity_levels_json JSONB,
  remediation_suggestions TEXT[],
  scan_timestamp TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_security_scans_project_id ON javari_security_scans(project_id);
```

#### **Audit Trail (1)**

**javari_work_log**
```sql
CREATE TABLE javari_work_log (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  project_id UUID REFERENCES projects(id) ON DELETE CASCADE,
  action_type TEXT NOT NULL,
  action_description TEXT NOT NULL,
  reasoning_log TEXT,
  confidence_score DECIMAL(3,2),
  files_modified_json JSONB,
  outcome TEXT CHECK (outcome IN ('success', 'partial_success', 'failure')),
  timestamp TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_work_log_user_id ON javari_work_log(user_id);
CREATE INDEX idx_work_log_project_id ON javari_work_log(project_id);
CREATE INDEX idx_work_log_timestamp ON javari_work_log(timestamp DESC);
```

### Database Functions

**Credential Inheritance:**
```sql
CREATE OR REPLACE FUNCTION get_credentials_with_inheritance(
  p_project_id UUID,
  p_subproject_id UUID
)
RETURNS TABLE (
  id UUID,
  name TEXT,
  encrypted_value TEXT,
  encryption_iv TEXT,
  encryption_tag TEXT,
  source TEXT
) AS $$
BEGIN
  -- Get sub-project specific credentials
  RETURN QUERY
  SELECT 
    c.id,
    c.name,
    c.encrypted_value,
    c.encryption_iv,
    c.encryption_tag,
    'subproject'::TEXT as source
  FROM javari_credentials c
  WHERE c.subproject_id = p_subproject_id;
  
  -- Get inherited project credentials not overridden
  RETURN QUERY
  SELECT 
    c.id,
    c.name,
    c.encrypted_value,
    c.encryption_iv,
    c.encryption_tag,
    'project'::TEXT as source
  FROM javari_credentials c
  WHERE c.project_id = p_project_id
    AND c.is_inherited = TRUE
    AND c.name NOT IN (
      SELECT name 
      FROM javari_credentials 
      WHERE subproject_id = p_subproject_id
    );
END;
$$ LANGUAGE plpgsql;
```

**Vector Search:**
```sql
CREATE OR REPLACE FUNCTION match_knowledge(
  query_embedding vector(1536),
  match_threshold float,
  match_count int,
  project_id UUID
)
RETURNS TABLE (
  id UUID,
  content TEXT,
  similarity float
) AS $$
BEGIN
  RETURN QUERY
  SELECT
    javari_knowledge_base.id,
    javari_knowledge_base.content,
    1 - (javari_knowledge_base.embedding <=> query_embedding) as similarity
  FROM javari_knowledge_base
  WHERE javari_knowledge_base.project_id = match_knowledge.project_id
    AND 1 - (javari_knowledge_base.embedding <=> query_embedding) > match_threshold
  ORDER BY similarity DESC
  LIMIT match_count;
END;
$$ LANGUAGE plpgsql;
```

### Row Level Security (RLS)

**Example RLS Policies:**
```sql
-- Users can only see their own conversations
CREATE POLICY "Users can view own conversations"
  ON conversations FOR SELECT
  USING (auth.uid() = user_id);

-- Users can create conversations
CREATE POLICY "Users can create conversations"
  ON conversations FOR INSERT
  WITH CHECK (auth.uid() = user_id);

-- Users can manage own credentials
CREATE POLICY "Users can manage own credentials"
  ON javari_credentials FOR ALL
  USING (auth.uid() = user_id);

-- Service role bypasses RLS
-- (Used by Javari for autonomous operations)
```

---

## API ARCHITECTURE

### API Routes Structure

```
app/api/javari/
├── chat/
│   ├── route.ts                    # POST: Send message, stream response
│   └── continue/route.ts           # POST: Create child chat
├── credentials/
│   ├── route.ts                    # CRUD operations
│   ├── test/route.ts               # POST: Test credential validity
│   └── rotate/route.ts             # POST: Rotate credential
├── knowledge/
│   ├── upload/route.ts             # POST: Upload file to knowledge base
│   └── search/route.ts             # POST: Vector search
├── projects/
│   └── route.ts                    # CRUD for projects
├── health/
│   ├── route.ts                    # GET: Health status
│   └── monitors/route.ts           # CRUD for monitors
├── work/
│   └── log/route.ts                # GET: Audit trail
├── cron/
│   ├── health-check/route.ts      # Autonomous build monitoring
│   └── web-crawl/route.ts         # Knowledge updates
└── build/
    └── route.ts                    # POST: Build application request
```

### Authentication

**Supabase Auth:**
```typescript
import { createServerClient } from '@supabase/ssr';

export async function getUser(request: Request) {
  const supabase = createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        get(name: string) {
          return request.headers.get('cookie')?.includes(name);
        },
      },
    }
  );

  const { data: { user }, error } = await supabase.auth.getUser();
  if (error || !user) {
    throw new Error('Unauthorized');
  }
  return user;
}
```

### API Response Format

**Success:**
```json
{
  "success": true,
  "data": { ... },
  "meta": {
    "timestamp": "2025-11-08T12:00:00Z",
    "request_id": "uuid",
    "credits_used": 0.05
  }
}
```

**Error:**
```json
{
  "success": false,
  "error": {
    "code": "INVALID_INPUT",
    "message": "Project ID is required",
    "details": { ... }
  },
  "meta": {
    "timestamp": "2025-11-08T12:00:00Z",
    "request_id": "uuid"
  }
}
```

---

## ENCRYPTION IMPLEMENTATION

### AES-256-GCM

**Encryption:**
```typescript
import crypto from 'crypto';

export function encryptCredential(value: string, key: Buffer): {
  encrypted: string;
  iv: string;
  authTag: string;
} {
  const iv = crypto.randomBytes(16);
  const cipher = crypto.createCipheriv('aes-256-gcm', key, iv);
  
  let encrypted = cipher.update(value, 'utf8', 'base64');
  encrypted += cipher.final('base64');
  
  const authTag = cipher.getAuthTag();
  
  return {
    encrypted,
    iv: iv.toString('base64'),
    authTag: authTag.toString('base64')
  };
}
```

**Decryption:**
```typescript
export function decryptCredential(
  encrypted: string,
  iv: string,
  authTag: string,
  key: Buffer
): string {
  const decipher = crypto.createDecipheriv(
    'aes-256-gcm',
    key,
    Buffer.from(iv, 'base64')
  );
  
  decipher.setAuthTag(Buffer.from(authTag, 'base64'));
  
  let decrypted = decipher.update(encrypted, 'base64', 'utf8');
  decrypted += decipher.final('utf8');
  
  return decrypted;
}
```

**Key Management:**
- Master key stored in environment variable
- Never committed to Git
- Rotated annually
- Backed up in password manager (1Password)

---

## AI PROVIDER INTEGRATION

### OpenAI GPT-4

**Chat Completions:**
```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

export async function chatCompletion(messages: Message[], stream = true) {
  const response = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages,
    stream,
    temperature: 0.7,
    max_tokens: 2000
  });
  
  if (stream) {
    return new ReadableStream({
      async start(controller) {
        for await (const chunk of response) {
          const content = chunk.choices[0]?.delta?.content || '';
          controller.enqueue(content);
        }
        controller.close();
      }
    });
  }
  
  return response.choices[0].message.content;
}
```

**Embeddings:**
```typescript
export async function generateEmbedding(text: string) {
  const response = await openai.embeddings.create({
    model: 'text-embedding-ada-002',
    input: text.substring(0, 8000) // Max 8K tokens
  });
  
  return response.data[0].embedding;
}
```

### Multi-Provider Routing

```typescript
export async function routeAIRequest(
  messages: Message[],
  taskType: 'chat' | 'code' | 'research' | 'creative'
) {
  // Route based on task type and cost
  switch (taskType) {
    case 'chat':
      // Simple chat → GPT-3.5 (cheap)
      return useOpenAI('gpt-3.5-turbo', messages);
    
    case 'code':
      // Complex code → GPT-4 (high quality)
      return useOpenAI('gpt-4-turbo-preview', messages);
    
    case 'research':
      // Research → Perplexity (web access)
      return usePerplexity(messages);
    
    case 'creative':
      // Long-form content → Claude (large context)
      return useClaude('claude-3-sonnet-20240229', messages);
  }
}
```

---

## PERFORMANCE OPTIMIZATION

### Caching Strategy

**Redis/Vercel KV (Future):**
- Cache frequently accessed credentials
- Cache RAG search results (1 hour TTL)
- Cache AI responses for identical queries

**Database Indexes:**
- All foreign keys indexed
- Timestamp columns indexed for sorting
- Composite indexes on common queries

### Bundle Optimization

**Next.js Config:**
```typescript
// next.config.js
module.exports = {
  output: 'standalone',
  compress: true,
  images: {
    domains: ['kteobfyferrukqeolofj.supabase.co'],
    formats: ['image/avif', 'image/webp'],
  },
  experimental: {
    serverActions: true,
  },
};
```

---

## SECURITY BEST PRACTICES

### Input Validation
- Zod schemas for all API inputs
- SQL injection prevention via parameterized queries
- XSS prevention via React's built-in escaping

### Rate Limiting
- 100 requests per minute per user
- 1000 AI tokens per minute per free user
- Unlimited for pro/enterprise

### HTTPS Only
- All traffic encrypted in transit
- HSTS headers enabled
- Secure cookies with HttpOnly flag

---

## DEPLOYMENT CHECKLIST

**Before Production:**
- [ ] All environment variables set in Vercel
- [ ] Database migrations applied
- [ ] RLS policies verified
- [ ] SSL certificate for javariai.com
- [ ] Cron jobs enabled
- [ ] Error tracking configured (Sentry)
- [ ] Performance monitoring (Vercel Analytics)
- [ ] Backup strategy implemented
- [ ] Disaster recovery plan documented

---

**Technical Architecture v1.0 - November 2025**
