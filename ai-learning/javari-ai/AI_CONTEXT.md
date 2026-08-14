# Javari AI - AI Context Document

**Last Updated:** November 8, 2025  
**Version:** 1.0 (95% Complete)  
**Status:** DEPLOYED & OPERATIONAL  
**Repository:** [CR-AudioViz-AI/javari-javari](https://github.com/CR-AudioViz-AI/javari-javari)  
**Live URL:** javari-javari-77b6mtat5-roy-hendersons-projects-1d3d5e94.vercel.app (Preview)

---

## PURPOSE & MISSION

Javari AI is not a chatbot - it's an **autonomous AI system** that serves as the master builder and CTO of the CR AudioViz AI platform. Javari's mission is to democratize professional-grade development tools through AI while continuously learning, self-healing, and evolving without human intervention.

### Core Capabilities

**1. Autonomous Development**
- Build complete applications from natural language descriptions
- Deploy to Vercel with zero human intervention
- Manage GitHub repositories and version control
- Execute multi-step workflows while you sleep

**2. Self-Healing Operations**
- Detect build failures automatically via cron jobs (30-minute intervals)
- Diagnose root causes using build logs and error patterns
- Apply fixes and redeploy without human approval
- Track healing success rates in `javari_healing_events` table

**3. Continuous Learning**
- Ingest documentation through automated web crawls (daily cron)
- Build knowledge base with vector embeddings (pgvector)
- Pattern recognition from successful vs failed actions
- A/B testing of different approaches to same problems

**4. Multi-Agent Orchestration**
- Coordinate with specialized AI avatars (JAVARI, Kairo, Pulse, Scout)
- Parallel task execution across multiple AI instances
- Peer code review before deployment
- Collaborative problem-solving for complex challenges

**5. Transparent Intelligence**
- Every decision includes reasoning logs and confidence scores
- Complete audit trail of all actions in database
- Proactive suggestions with risk assessment
- User-controllable autonomy levels

---

## TECHNICAL ARCHITECTURE

### Technology Stack

**Frontend:**
- Next.js 14.1.0 (App Router)
- React 18+ with TypeScript
- Tailwind CSS for styling
- shadcn/ui component library
- Lucide React icons

**Backend:**
- Next.js API Routes (serverless functions)
- Supabase (PostgreSQL 15 with pgvector extension)
- Row Level Security (RLS) on all tables
- AES-256-GCM encryption for credentials

**AI Integrations:**
- OpenAI GPT-4 (primary reasoning engine)
- Anthropic Claude Sonnet 4.5 (alternative/fallback)
- Google Gemini (specialized tasks)
- Perplexity AI (research/market data)
- Intelligent routing based on task type and cost

**Deployment:**
- Vercel (preview-only mode with `autoAlias: false`)
- GitHub Actions for CI/CD
- Cron jobs for autonomous operations
- Zero-downtime deployments

### Database Schema (33+ Tables)

#### Authentication & Users
```sql
-- Core user management with credit system
users (id, email, name, plan, credits_balance, monthly_allowance)
user_preferences (user_id, default_ai_provider, temperature, theme)
```

#### Projects & Organization
```sql
-- Hierarchical project structure with inheritance
projects (id, user_id, name, repository_url, metadata)
subprojects (id, project_id, name, description)
```

#### Chat & Conversations
```sql
-- Infinite memory through parent-child linking
conversations (id, user_id, project_id, title, starred, status)
messages (id, conversation_id, role, content, ai_model, tokens_used)
conversation_continuations (child_id, parent_id, summary, context_tokens)
artifacts (id, conversation_id, type, filename, file_path, preview_data)
```

#### Credential Vault (Encrypted)
```sql
-- AES-256-GCM encrypted storage with rotation
javari_credentials (id, user_id, project_id, name, encrypted_value, 
  encryption_iv, encryption_tag, credential_type, is_inherited, allow_override)
javari_credential_snapshots (id, conversation_id, credentials_snapshot)
javari_credential_rotations (id, credential_id, old_value_hash, new_value_hash,
  rotation_type, grace_period_end)
```

#### Knowledge & Learning
```sql
-- RAG system with vector embeddings
javari_knowledge_base (id, user_id, project_id, content, content_hash,
  source_type, source_url, embedding[1536], relevance_score, last_accessed_at)
javari_learned_patterns (id, pattern_type, context, action_taken, success_rate,
  confidence_score, usage_count)
javari_ab_tests (id, test_name, variant_a, variant_b, winner, confidence_level)
```

#### Self-Healing & Monitoring
```sql
-- 24/7 monitoring with auto-fix capabilities
javari_health_monitors (id, project_id, monitor_type, check_frequency,
  alert_threshold, auto_fix_enabled, slack_webhook, email_recipients)
javari_alerts (id, monitor_id, severity, alert_message, auto_fix_attempted,
  fix_success, resolution_time_seconds)
javari_healing_events (id, error_type, error_details, healing_action_taken,
  healing_successful, time_to_heal_seconds)
```

#### Autonomous Operations
```sql
-- Background task execution without human intervention
javari_task_queue (id, user_id, project_id, task_type, priority, payload,
  status, scheduled_at, started_at, completed_at, error_message, retry_count)
javari_agents (id, agent_name, specialization, is_active, current_task_id,
  tasks_completed, success_rate, average_completion_time_seconds)
javari_agent_collaboration (id, task_id, agent_ids[], collaboration_type,
  final_decision, decision_rationale)
```

#### Cost & Financial Intelligence
```sql
-- Granular cost tracking with predictive forecasting
javari_cost_tracking (id, user_id, project_id, service, feature_used,
  input_tokens, output_tokens, cost_usd, timestamp)
javari_cost_forecasts (id, user_id, forecast_period, predicted_cost_usd,
  confidence_interval, cost_breakdown_json, recommendations)
```

#### Compliance & Security
```sql
-- Automatic PII detection and security scanning
javari_pii_detections (id, conversation_id, message_id, pii_type,
  detected_text_hash, redaction_applied, reported_to_admin)
javari_security_scans (id, project_id, scan_type, vulnerabilities_found,
  severity_levels_json, remediation_suggestions, scan_timestamp)
```

#### Audit & Work Log
```sql
-- Complete audit trail of all Javari actions
javari_work_log (id, user_id, project_id, action_type, action_description,
  reasoning_log, confidence_score, files_modified_json, outcome, timestamp)
```

### Key Database Features

**Row Level Security (RLS)**
- Every table has policies ensuring users only see their own data
- Service role key bypasses RLS for admin operations
- Team-based access control for collaborative projects

**Encryption at Rest**
- Supabase Vault for credentials (AES-256-GCM)
- pgcrypto extension for PostgreSQL-level encryption
- Separate encryption keys per user with master key in env vars

**Vector Search (pgvector)**
- 1536-dimensional embeddings for knowledge base
- Cosine similarity search for RAG (Retrieval Augmented Generation)
- Automatic re-ranking based on relevance and recency

**Triggers & Functions**
- `update_updated_at_column()` - Auto-update timestamps
- `get_credentials_with_inheritance()` - Resolve inherited credentials
- `calculate_context_tokens()` - Track conversation size
- `auto_spawn_continuation()` - Create child chats when nearing token limits

---

## AUTONOMOUS FEATURES (IMPLEMENTED)

### 1. Secure Credential Vault ✅

**What It Does:**
- Stores sensitive credentials (API keys, tokens, passwords) with military-grade encryption
- Project-level credentials automatically inherited by sub-projects
- Sub-projects can override inherited values when needed
- Zero-trust architecture - credentials never stored in plain text

**How It Works:**
```typescript
// Encryption: AES-256-GCM with unique IV per credential
const iv = crypto.randomBytes(16);
const cipher = crypto.createCipheriv('aes-256-gcm', key, iv);
const encrypted = Buffer.concat([cipher.update(value, 'utf8'), cipher.final()]);
const authTag = cipher.getAuthTag();

// Storage: Only encrypted blob, IV, and auth tag in database
await supabase.from('javari_credentials').insert({
  name: 'GITHUB_TOKEN',
  encrypted_value: encrypted.toString('base64'),
  encryption_iv: iv.toString('base64'),
  encryption_tag: authTag.toString('base64'),
  is_inherited: true,
  allow_override: true
});
```

**Credential Inheritance:**
```
Main Project: craudiovizai-website
├── GITHUB_TOKEN (inherited by ALL sub-projects)
├── VERCEL_TOKEN (inherited by ALL sub-projects)
├── SUPABASE_SERVICE_KEY (inherited by ALL sub-projects)
└── Sub-Projects:
    ├── javari-ebook-creator
    │   └── OPENAI_API_KEY (specific to this app)
    ├── javari-paycheck-calculator
    │   └── STRIPE_SECRET_KEY (specific to this app)
    └── javari-music-builder
        └── GITHUB_TOKEN (overrides parent's token)
```

**API Endpoints:**
- `POST /api/javari/credentials` - Create encrypted credential
- `GET /api/javari/credentials` - List with inheritance resolution
- `PUT /api/javari/credentials` - Update value (re-encrypts)
- `DELETE /api/javari/credentials` - Secure deletion
- `POST /api/javari/credentials/test` - Validate credential works
- `POST /api/javari/credentials/rotate` - Rotate with grace period

### 2. Chat Continuation System ✅

**What It Does:**
- Automatically spawns new child chats when approaching token limits (90% threshold)
- Maintains complete conversation history through parent-child linking
- Generates AI summaries of previous context for seamless handoff
- Never lose conversation context - infinite memory

**How It Works:**
```typescript
// Check if continuation needed
const totalTokens = conversationTokens + systemPromptTokens;
const modelLimit = 128000; // GPT-4 Turbo limit
const percentFull = (totalTokens / modelLimit) * 100;

if (percentFull >= 90) {
  // Generate summary of conversation so far
  const summary = await openai.chat.completions.create({
    model: 'gpt-4',
    messages: [
      { role: 'system', content: 'Summarize this conversation for context handoff' },
      ...conversationHistory
    ]
  });
  
  // Create child chat
  const childChat = await supabase.from('conversations').insert({
    user_id: userId,
    project_id: projectId,
    title: `${originalTitle} (Continued)`,
    parent_id: currentConversationId
  });
  
  // Link to parent
  await supabase.from('conversation_continuations').insert({
    child_id: childChat.id,
    parent_id: currentConversationId,
    summary: summary.choices[0].message.content,
    context_tokens: totalTokens
  });
  
  // Notify user
  return { needsContinuation: true, childChatUrl: `/chat/${childChat.id}` };
}
```

**User Experience:**
- Notification appears: "This chat is 90% full. Continue to new chat?"
- One-click continuation - seamless transition
- View conversation tree to navigate between parent/child chats
- Each child chat includes summary of parent context

### 3. Project Management Dashboard ✅

**What It Does:**
- CRUD operations for projects and sub-projects
- Repository linking (GitHub integration)
- Metadata storage (tech stack, dependencies, notes)
- Project hierarchy visualization
- Quick access to all project-related chats and credentials

**Key Features:**
```typescript
// Create project with metadata
const project = await supabase.from('projects').insert({
  user_id: userId,
  name: 'craudiovizai-website',
  description: 'Main company website with 60+ embedded tools',
  repository_url: 'https://github.com/CR-AudioViz-AI/craudiovizai-website',
  metadata: {
    tech_stack: ['Next.js 14', 'Tailwind CSS', 'Supabase'],
    deployment_url: 'https://craudiovizai.com',
    last_build_status: 'SUCCESS',
    dependencies_count: 47
  }
});

// Create sub-project with inheritance
const subproject = await supabase.from('subprojects').insert({
  project_id: project.id,
  name: 'javari-ebook-creator',
  description: 'PDF eBook builder with AI assistance',
  metadata: {
    inherits_credentials: true,
    custom_credentials: ['OPENAI_API_KEY']
  }
});
```

### 4. Build Health Monitoring ✅

**What It Does:**
- 24/7 monitoring of Vercel deployments via cron jobs
- Automatic detection of build failures
- Real-time alerts via Slack/Email
- Auto-fix attempts before alerting humans
- Historical tracking of all incidents

**Monitoring Flow:**
```typescript
// Cron job every 30 minutes
export async function GET() {
  // Fetch all monitored projects
  const { data: monitors } = await supabase
    .from('javari_health_monitors')
    .select('*')
    .eq('is_active', true);
  
  for (const monitor of monitors) {
    // Check Vercel deployment status
    const deployment = await fetch(
      `https://api.vercel.com/v6/deployments/${monitor.project_id}`,
      { headers: { Authorization: `Bearer ${VERCEL_TOKEN}` } }
    ).then(r => r.json());
    
    if (deployment.state === 'ERROR') {
      // Attempt auto-fix
      const fixAttempted = await attemptAutoFix(deployment);
      
      // Create alert
      await supabase.from('javari_alerts').insert({
        monitor_id: monitor.id,
        severity: 'high',
        alert_message: `Build failed for ${monitor.project_id}`,
        auto_fix_attempted: fixAttempted.attempted,
        fix_success: fixAttempted.success
      });
      
      // Send notifications if auto-fix failed
      if (!fixAttempted.success) {
        await sendSlackAlert(monitor.slack_webhook, deployment);
        await sendEmailAlert(monitor.email_recipients, deployment);
      }
    }
  }
}
```

**Auto-Fix Strategies:**
1. **TypeScript Errors:** Apply type assertions, fix imports
2. **Dependency Issues:** Clear cache, reinstall packages
3. **Environment Variables:** Verify all required vars present
4. **Build Timeouts:** Optimize bundle size, enable caching
5. **Memory Errors:** Increase Node memory limit

### 5. RAG Knowledge System ✅

**What It Does:**
- Retrieval Augmented Generation for context-aware responses
- File upload (PDF, DOCX, TXT) with text extraction
- Vector embeddings for semantic search
- Project-scoped knowledge base
- Automatic context injection into AI prompts

**Implementation:**
```typescript
// Upload file to knowledge base
export async function POST(request: Request) {
  const formData = await request.formData();
  const file = formData.get('file') as File;
  
  // Extract text based on file type
  let extractedText = '';
  if (file.type === 'application/pdf') {
    const pdfData = await pdfParse(await file.arrayBuffer());
    extractedText = pdfData.text;
  } else if (file.type.includes('word')) {
    const docData = await mammoth.extractRawText({ buffer: await file.arrayBuffer() });
    extractedText = docData.value;
  }
  
  // Generate embedding
  const embedding = await openai.embeddings.create({
    model: 'text-embedding-ada-002',
    input: extractedText.substring(0, 8000)
  });
  
  // Store in database
  await supabase.from('javari_knowledge_base').insert({
    user_id: userId,
    project_id: projectId,
    content: extractedText,
    source_type: 'file_upload',
    source_url: file.name,
    embedding: embedding.data[0].embedding
  });
}

// Search knowledge base
async function searchKnowledge(query: string, projectId: string) {
  // Generate query embedding
  const queryEmbedding = await openai.embeddings.create({
    model: 'text-embedding-ada-002',
    input: query
  });
  
  // Vector similarity search
  const { data } = await supabase.rpc('match_knowledge', {
    query_embedding: queryEmbedding.data[0].embedding,
    match_threshold: 0.7,
    match_count: 5,
    project_id: projectId
  });
  
  return data.map(doc => doc.content).join('\n\n');
}

// Enhance messages with RAG
async function enhanceWithRAG(messages: Message[], projectId: string) {
  const lastUserMessage = messages.findLast(m => m.role === 'user');
  const relevantKnowledge = await searchKnowledge(lastUserMessage.content, projectId);
  
  // Inject knowledge into system prompt
  const systemPrompt = {
    role: 'system',
    content: `You are Javari AI. Relevant knowledge from the project:

${relevantKnowledge}

Use this context to inform your responses.`
  };
  
  return [systemPrompt, ...messages];
}
```

### 6. Work Log & Audit Trail ✅

**What It Does:**
- Complete audit log of every action Javari takes
- Reasoning logs and confidence scores
- File modification tracking
- Outcome recording (success/failure)
- Searchable and filterable history

**Use Cases:**
- Debugging: "What did Javari do that broke this?"
- Compliance: "Show me all credential access in last 30 days"
- Learning: "What actions had highest success rate?"
- Accountability: "Prove Javari didn't touch production"

---

## AUTONOMOUS FEATURES (PLANNED - PHASE 2)

### 7. Self-Healing Build System 🔄

**Status:** Partially implemented (monitoring exists, full auto-fix pending)

**What It Will Do:**
- Detect build failures within minutes via cron
- Fetch and analyze Vercel build logs automatically
- Use GPT-4 to diagnose root cause
- Generate and apply fixes without human approval
- Redeploy and verify fix worked
- Track success rate and learn from patterns

**Planned Implementation:**
```typescript
async function autoHealBuild(deploymentId: string) {
  // 1. Fetch build logs
  const logs = await fetchVercelLogs(deploymentId);
  
  // 2. Extract error messages
  const errors = extractErrors(logs);
  
  // 3. Ask GPT-4 for diagnosis
  const diagnosis = await openai.chat.completions.create({
    model: 'gpt-4',
    messages: [
      { role: 'system', content: 'You are a debugging expert. Analyze these build errors and suggest fixes.' },
      { role: 'user', content: JSON.stringify(errors) }
    ]
  });
  
  // 4. Apply fixes using GitHub API
  const fixesApplied = await applyFixes(diagnosis.choices[0].message.content);
  
  // 5. Trigger redeploy
  await triggerVercelDeploy();
  
  // 6. Verify fix worked
  const newDeployment = await waitForDeployment();
  const success = newDeployment.state === 'READY';
  
  // 7. Log healing event
  await supabase.from('javari_healing_events').insert({
    error_type: errors[0].type,
    error_details: JSON.stringify(errors),
    healing_action_taken: fixesApplied,
    healing_successful: success,
    time_to_heal_seconds: calculateDuration()
  });
  
  return { success, fixesApplied };
}
```

### 8. Automated Web Crawling 🔄

**Status:** Schema exists, crawler pending

**What It Will Do:**
- Daily cron job crawls specified websites
- Extract text from documentation, blogs, release notes
- Automatically add to knowledge base with embeddings
- Keep Javari up-to-date with latest best practices
- No manual knowledge uploads needed

**Planned Targets:**
- Next.js documentation updates
- Vercel changelog
- Supabase release notes
- OpenAI API changes
- React best practices

### 9. Pattern Recognition & Learning 🔄

**Status:** Database schema exists, ML model pending

**What It Will Do:**
- Classify conversation types (bug fix, feature request, documentation)
- Track which approaches work best for each type
- Measure success rate of different strategies
- Automatically adapt based on what works
- Share learnings across all user interactions

**Example Patterns:**
```sql
-- Pattern: TypeScript Import Errors
INSERT INTO javari_learned_patterns (
  pattern_type: 'typescript_import_error',
  context: 'Build fails with "Cannot find module" for local file',
  action_taken: 'Add ".ts" extension to import statement',
  success_rate: 0.95,
  confidence_score: 0.89,
  usage_count: 47
);

-- Pattern: Vercel Environment Variables
INSERT INTO javari_learned_patterns (
  pattern_type: 'missing_env_var',
  context: 'Runtime error "process.env.X is undefined"',
  action_taken: 'Add variable to Vercel project settings via API',
  success_rate: 1.00,
  confidence_score: 0.98,
  usage_count: 23
);
```

### 10. Multi-Agent Collaboration 🔄

**Status:** Schema exists, coordination logic pending

**What It Will Do:**
- Spawn specialized Javari instances for complex tasks
- DevOps Javari handles deployments
- Frontend Javari writes UI code
- Backend Javari builds APIs
- Database Javari designs schemas
- Peer review before committing changes

**Collaboration Flow:**
```typescript
async function buildFullstackFeature(request: FeatureRequest) {
  // Spawn specialized agents
  const frontendAgent = await spawnAgent('frontend');
  const backendAgent = await spawnAgent('backend');
  const dbAgent = await spawnAgent('database');
  
  // Parallel execution
  const [uiCode, apiCode, schema] = await Promise.all([
    frontendAgent.generateUI(request),
    backendAgent.generateAPI(request),
    dbAgent.generateSchema(request)
  ]);
  
  // Peer review
  const review = await coordinateReview([
    { agent: 'frontend', code: uiCode },
    { agent: 'backend', code: apiCode },
    { agent: 'database', code: schema }
  ]);
  
  if (review.approved) {
    await commitAndDeploy({ uiCode, apiCode, schema });
  } else {
    await requestRevisions(review.feedback);
  }
}
```

---

## AI-TO-AI COMMUNICATION PROTOCOL

When other AI agents (JAVARI, Kairo, Pulse, Scout) interact with Javari:

**Request Format:**
```json
{
  "requesting_agent": "JAVARI",
  "task_type": "build_application",
  "priority": "high",
  "context": {
    "user_id": "uuid",
    "project_id": "uuid",
    "request_summary": "Build a PDF invoice generator",
    "requirements": [
      "Accept user input for company details",
      "Generate professional PDF layout",
      "Include tax calculations",
      "Export as downloadable file"
    ]
  },
  "credentials_snapshot": {
    "GITHUB_TOKEN": "inherited_from_project",
    "VERCEL_TOKEN": "inherited_from_project",
    "OPENAI_API_KEY": "project_specific"
  },
  "expected_completion": "2025-11-09T00:00:00Z"
}
```

**Response Format:**
```json
{
  "responding_agent": "Javari",
  "task_id": "uuid",
  "status": "in_progress | completed | failed",
  "reasoning_log": [
    "Analyzed requirements and determined technology stack",
    "Selected Next.js 14 with Tailwind for UI",
    "Chose react-pdf library for generation",
    "Created component structure",
    "Implemented tax calculation logic",
    "Added form validation"
  ],
  "confidence_score": 0.92,
  "deliverables": {
    "repository_url": "https://github.com/CR-AudioViz-AI/javari-invoice-generator",
    "deployment_url": "https://javari-invoice-generator-preview.vercel.app",
    "files_created": 17,
    "lines_of_code": 1243
  },
  "completion_time_seconds": 847,
  "cost_breakdown": {
    "openai_gpt4": "$0.23",
    "vercel_deployment": "$0.00",
    "total_usd": "$0.23"
  }
}
```

---

## OPERATIONAL STATUS

### What's Live (95%)

✅ **Infrastructure**
- Next.js 14 application deployed to Vercel
- Supabase database with 33+ tables operational
- Row Level Security policies active
- Environment variables configured

✅ **Core Features**
- Secure credential vault with encryption
- Chat continuation system
- Project management dashboard
- Build health monitoring (passive)
- RAG knowledge system
- Work log / audit trail

✅ **AI Integration**
- OpenAI GPT-4 chat API working
- Streaming responses implemented
- Multi-provider routing framework ready
- Token tracking and cost attribution

✅ **Authentication**
- Supabase Auth configured
- Email/password + OAuth ready
- Session management
- Protected routes

### What's Pending (5%)

🔄 **Autonomous Features**
- Full auto-fix implementation for builds
- Automated web crawling cron job
- Pattern recognition ML model
- Multi-agent coordination logic
- A/B testing automation

🔄 **Integrations**
- Google Workspace (Drive, Gmail, Calendar)
- Microsoft 365 (OneDrive, Outlook)
- Slack notifications
- GitHub webhooks for auto-documentation

🔄 **Production**
- Domain configuration (javariai.com)
- SSL certificate
- Production environment variables
- Rate limiting and abuse prevention

---

## LEARNING & ADAPTATION GUIDELINES

### For AI Agents Interacting with Javari

**1. Always Provide Context**
- Include user_id, project_id, and subproject_id when available
- Attach conversation history for better continuity
- Reference previous Javari actions if relevant

**2. Use Confidence Scores**
- Request confidence score with every Javari response
- If confidence < 0.7, consider asking for human approval
- Track confidence vs. actual success rate over time

**3. Learn from Outcomes**
- Report success/failure back to Javari after using its deliverables
- Include specific feedback on what worked or didn't
- Help Javari update its learned patterns table

**4. Respect Autonomy Levels**
- Check user's preference: `full_auto`, `approve_critical`, `manual`
- For `full_auto`: Javari proceeds without asking
- For `approve_critical`: Ask before production deploys or credential changes
- For `manual`: Ask before every significant action

**5. Collaborate, Don't Compete**
- Javari is specialized in building and deployment
- Other agents handle their domains (community, creative, analytics)
- When tasks overlap, request Javari's assistance rather than duplicating work
- Share learnings bidirectionally

### For Javari Learning System

**Success Metrics to Track:**
- Build success rate (target: >95%)
- Average time to deployment (target: <15 minutes)
- Auto-fix success rate (target: >80%)
- User satisfaction scores (target: >4.5/5)
- Cost per successful deployment (target: <$0.50)

**Patterns to Learn:**
- Which frameworks work best for which use cases
- Common error types and their reliable fixes
- User preference patterns (tech stack, UI style)
- Optimal code structure for maintainability
- Cost-effective AI model routing

**Continuous Improvement:**
- A/B test different approaches to same problem
- Measure user retention after Javari interactions
- Track which suggestions are accepted vs. rejected
- Analyze long-term project health (builds still working 6 months later?)

---

## SECURITY & COMPLIANCE

### Data Protection
- All credentials encrypted at rest (AES-256-GCM)
- Automatic PII detection in conversations
- GDPR-compliant data deletion on request
- Row Level Security prevents data leakage between users

### Audit Requirements
- Complete work log forever (never deleted)
- Credential access tracking
- Build/deploy history with timestamps
- Financial transaction records

### Incident Response
- Automatic rollback if deployment causes errors
- Immediate notification on security vulnerability
- Credential rotation enforced quarterly
- Grace period for zero-downtime updates

---

## COST OPTIMIZATION

Javari tracks costs at granular level to optimize spending:

**AI Model Costs:**
- GPT-4: $0.03/1K input + $0.06/1K output
- Claude Sonnet 4.5: $0.003/1K input + $0.015/1K output
- GPT-3.5 Turbo: $0.0005/1K input + $0.0015/1K output

**Routing Intelligence:**
- Simple queries → GPT-3.5 (90% cost savings)
- Complex reasoning → GPT-4
- Long-context tasks → Claude (larger context window)
- Real-time research → Perplexity

**Target Metrics:**
- Average cost per conversation: <$0.10
- Average cost per app built: <$2.00
- Monthly cost per active user: <$15.00

---

## VERSION HISTORY

**v1.0 (November 2025) - Current**
- Core autonomous infrastructure deployed
- 95% feature complete
- Database schema finalized
- Credential vault operational
- Chat continuation working
- Preview deployments active

**v0.9 (October 2025)**
- Initial Javari architecture designed
- Database tables created
- Basic chat interface built
- OpenAI integration completed

**v0.5 (October 2025)**
- Concept validation
- Technology stack selected
- Repository structure defined

---

## NEXT STEPS FOR AI AGENTS

When working with Javari AI:

1. **For Building Applications:**
   - Use POST /api/javari/build endpoint
   - Provide detailed requirements
   - Include credential snapshot
   - Wait for task_id and poll for completion

2. **For Monitoring Health:**
   - Query javari_health_monitors table
   - Check javari_alerts for recent issues
   - Review javari_healing_events for auto-fix history

3. **For Knowledge Queries:**
   - Search javari_knowledge_base with vector similarity
   - Filter by project_id for scoped results
   - Use RAG-enhanced prompts for better responses

4. **For Cost Tracking:**
   - Query javari_cost_tracking table
   - Group by project_id or feature_used
   - Generate forecasts using javari_cost_forecasts

5. **For Audit Trails:**
   - Query javari_work_log for complete history
   - Filter by action_type, user_id, or project_id
   - Include reasoning_log for transparency

---

**Javari AI: Your autonomous partner in building the future.** 🤖🚀
