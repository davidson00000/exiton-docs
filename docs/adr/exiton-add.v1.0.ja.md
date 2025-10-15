# 🔷 EXITON Architecture Definition Document (ADD) v1.0

**Document Type:** Technical Specification  
**Status:** Implementation Ready  
**Architect:** Claude Code  
**Date:** 2025-10-15  
**Audience:** Senior Engineers, System Architects, AI Integration Specialists

---

## Executive Summary

EXITON is a **human-centric AI orchestration platform** for individual developers. This document defines the technical architecture that enables the core philosophy:

> *"AIはバンドメンバー。人間が指揮者。"*

**Core Technical Principles:**
1. **Hexagonal Architecture** — AI providers are hot-swappable adapters
2. **Event-Driven Coordination** — Async communication via event bus
3. **Human-in-the-Loop by Design** — All critical paths require human approval
4. **Provenance & Transparency** — Every AI output tagged with confidence, agent, reasoning

---

## 1. System Overview

### 1.1 Architectural Layers

```
┌─────────────────────────────────────────────────────────────┐
│ L4: Culture Layer                                            │
│ ├─ Content Delivery (Studio, Blog, Media)                   │
│ └─ External API (for ecosystem apps)                        │
├─────────────────────────────────────────────────────────────┤
│ L3: Academy Layer                                            │
│ ├─ Knowledge Graph (courses, docs, patterns)                │
│ └─ Community Features (forums, sharing)                     │
├─────────────────────────────────────────────────────────────┤
│ L2: Client Applications                                      │
│ ├─ iOS Apps (Habit Checker, CyberPomo)                      │
│ ├─ Desktop App (Electron-based)                             │
│ └─ CLI Tools (exiton-cli)                                   │
├─────────────────────────────────────────────────────────────┤
│ L1: SaaS Core ← PRIMARY FOCUS OF THIS DOCUMENT              │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │              HEXAGONAL CORE                             │ │
│  │  ┌──────────────────────────────────────────────────┐  │ │
│  │  │         Domain Layer (Business Logic)            │  │ │
│  │  │  • Task Management   • Progress Tracking         │  │ │
│  │  │  • Motivation Engine • Human Review System       │  │ │
│  │  └──────────────────────────────────────────────────┘  │ │
│  │         ↕               ↕               ↕               │ │
│  │  ┌──────────┐    ┌──────────┐    ┌──────────┐        │ │
│  │  │   AI     │    │ Database │    │   Auth   │        │ │
│  │  │ Adapters │    │ Adapters │    │ Adapters │        │ │
│  │  └──────────┘    └──────────┘    └──────────┘        │ │
│  └────────────────────────────────────────────────────────┘ │
│                           ↕                                  │
│  ┌────────────────────────────────────────────────────────┐ │
│  │         Event Bus (Kafka / Redis Streams)              │ │
│  └────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 Technology Stack (L1 Core)

| Component | Technology | Justification |
|-----------|-----------|---------------|
| **Backend Runtime** | Node.js 20+ (TypeScript) | AI SDK compatibility, rapid iteration |
| **Web Framework** | Next.js 15 | SSR, API routes, Vercel deployment |
| **Database** | PostgreSQL 16 + Prisma ORM | Relational integrity, migration safety |
| **Vector Store** | Pinecone / Weaviate | RAG memory, semantic search |
| **Event Bus** | Redis Streams (MVP) → Kafka (Scale) | Async orchestration, event sourcing |
| **AI Providers** | Claude API, Gemini API, OpenAI API | Multi-modal, code-gen, orchestration |
| **Auth** | Clerk / Auth0 | Secure, supports team accounts |
| **Payment** | Stripe | Subscription management |
| **Deployment** | Vercel (frontend) + Railway (backend) | Edge optimization, scaling |

### 1.3 Guiding Architectural Principles

```typescript
/**
 * EXITON Architectural Covenant
 * These principles override all implementation decisions.
 */

const PRINCIPLES = {
  // 1. Human Agency
  HUMAN_PRIMACY: "No AI decision executes without human review gate for critical paths",
  
  // 2. Transparency
  PROVENANCE: "Every AI output must include: agent, confidence, reasoning, timestamp",
  
  // 3. Modularity
  PORTS_ADAPTERS: "AI providers are replaceable without touching domain logic",
  
  // 4. Auditability
  EVENT_SOURCING: "All state changes recorded as immutable events",
  
  // 5. Resilience
  GRACEFUL_DEGRADATION: "System continues functioning if AI provider fails",
} as const;
```

---

## 2. Core Architecture Design

### 2.1 Hexagonal Architecture Detailed

```
┌─────────────────────────────────────────────────────────────┐
│                    DOMAIN CORE (Pure Business Logic)         │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  Entities:                                              │ │
│  │    User, Project, Task, Milestone, AISession,          │ │
│  │    HumanReview, MotivationEvent                        │ │
│  │                                                         │ │
│  │  Use Cases (Application Services):                     │ │
│  │    CreateTask, ExecuteAIAgent, ReviewAIOutput,         │ │
│  │    MergeCodeChanges, TrackProgress, TriggerNotification│ │
│  │                                                         │ │
│  │  Domain Services:                                      │ │
│  │    MotivationEngine, ConfidenceCalculator,             │ │
│  │    TokenBudgetManager, ContextWindowManager            │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  ════════════════════ PORTS (Interfaces) ═══════════════════ │
│                                                              │
│  Primary Ports (Driven by external actors):                 │
│    - ITaskRepository                                         │
│    - IAIAgentPort                                            │
│    - INotificationPort                                       │
│    - IVectorStorePort                                        │
│                                                              │
│  Secondary Ports (Drive external systems):                   │
│    - IEventPublisher                                         │
│    - IMetricsCollector                                       │
│    - IAuthProvider                                           │
└─────────────────────────────────────────────────────────────┘
         ↕                    ↕                    ↕
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│   ADAPTERS      │  │   ADAPTERS      │  │   ADAPTERS      │
│                 │  │                 │  │                 │
│ • ClaudeAdapter │  │ • PostgresRepo  │  │ • VercelAuth    │
│ • GeminiAdapter │  │ • PineconeVec   │  │ • SendgridMail  │
│ • GPTAdapter    │  │ • RedisCache    │  │ • StripePayment │
│ • OllamaAdapter │  │ • S3Storage     │  │ • PosthogEvents │
└─────────────────┘  └─────────────────┘  └─────────────────┘
```

### 2.2 Core Domain Models

```typescript
// ═══════════════════════════════════════════════════════════
// DOMAIN ENTITIES
// ═══════════════════════════════════════════════════════════

interface User {
  id: UUID;
  email: string;
  subscription_tier: "free" | "pro" | "team";
  ai_preferences: AIPreferences;
  token_budget: TokenBudget;
  created_at: DateTime;
}

interface AIPreferences {
  default_coder: "claude" | "gemini" | "gpt" | "local";
  default_reviewer: "claude" | "gemini" | "gpt";
  default_designer: "gemini" | "claude";
  tone: "formal" | "casual" | "bebop"; // AI response style
  confidence_threshold: number; // 0.0 - 1.0
}

interface Project {
  id: UUID;
  owner_id: UUID;
  name: string;
  description: string;
  status: "active" | "paused" | "completed" | "archived";
  ai_context_window: AIContext[]; // Last 50 interactions
  created_at: DateTime;
  updated_at: DateTime;
}

interface Task {
  id: UUID;
  project_id: UUID;
  title: string;
  description: string;
  status: "todo" | "in_progress" | "review" | "done";
  assigned_ai_agent?: "claude" | "gemini" | "gpt";
  ai_generated: boolean;
  confidence_score?: number; // If AI-generated
  human_review_required: boolean;
  created_by: "human" | "ai";
  metadata: TaskMetadata;
}

interface AISession {
  id: UUID;
  project_id: UUID;
  task_id?: UUID;
  conductor: "chatgpt"; // Always ChatGPT as orchestrator
  agents_invoked: AgentInvocation[];
  total_tokens_used: number;
  started_at: DateTime;
  completed_at?: DateTime;
  status: "pending" | "executing" | "awaiting_review" | "completed" | "failed";
}

interface AgentInvocation {
  agent: "claude" | "gemini" | "gpt" | "local";
  role: "coder" | "reviewer" | "designer" | "pm";
  prompt: string;
  response: AIResponse;
  tokens_used: number;
  execution_time_ms: number;
  invoked_at: DateTime;
}

interface AIResponse {
  content: string;
  confidence: number; // 0.0 - 1.0
  reasoning_chain?: string[]; // Explainable AI
  safety_check: SafetyCheck;
  provenance: Provenance;
}

interface SafetyCheck {
  hallucination_risk: "low" | "medium" | "high";
  ethical_concern: boolean;
  requires_human_review: boolean;
  flagged_content?: string[];
}

interface Provenance {
  agent: string;
  model_version: string;
  timestamp: DateTime;
  temperature: number;
  max_tokens: number;
}

interface HumanReview {
  id: UUID;
  session_id: UUID;
  reviewer_id: UUID;
  ai_response_id: UUID;
  decision: "approved" | "rejected" | "modified";
  reason?: string;
  modifications?: string;
  reviewed_at: DateTime;
}

interface HumanInterventionPoint {
  id: UUID;
  session_id: UUID;
  user_id: UUID;
  intervention_type: "override" | "rejection" | "modification" | "approval";
  ai_suggestion: string;
  human_decision: string;
  reason: string;
  timestamp: DateTime;
}

interface TokenBudget {
  user_id: UUID;
  tier: "free" | "pro" | "team";
  monthly_limit: number;
  used_this_month: number;
  resets_at: DateTime;
}
```

### 2.3 Port Definitions (Interfaces)

```typescript
// ═══════════════════════════════════════════════════════════
// PRIMARY PORTS (Domain → Adapters)
// ═══════════════════════════════════════════════════════════

interface IAIAgentPort {
  /**
   * Execute AI agent with intent and context
   * @returns Promise<AIResponse> with confidence and provenance
   */
  execute(request: AIRequest): Promise<AIResponse>;
  
  /**
   * Check if agent is available and healthy
   */
  healthCheck(): Promise<boolean>;
  
  /**
   * Get current rate limit status
   */
  getRateLimitStatus(): Promise<RateLimitStatus>;
}

interface ITaskRepository {
  create(task: Task): Promise<Task>;
  update(id: UUID, updates: Partial<Task>): Promise<Task>;
  findById(id: UUID): Promise<Task | null>;
  findByProject(projectId: UUID): Promise<Task[]>;
  delete(id: UUID): Promise<void>;
}

interface IVectorStorePort {
  /**
   * Store conversation context for RAG
   */
  upsertContext(projectId: UUID, context: AIContext): Promise<void>;
  
  /**
   * Retrieve semantically relevant context
   */
  searchSimilar(projectId: UUID, query: string, limit: number): Promise<AIContext[]>;
  
  /**
   * Compress context window (semantic summarization)
   */
  compressContext(contexts: AIContext[]): Promise<AIContext>;
}

interface IEventPublisher {
  publish(event: DomainEvent): Promise<void>;
  publishBatch(events: DomainEvent[]): Promise<void>;
}

// ═══════════════════════════════════════════════════════════
// ADAPTER IMPLEMENTATIONS (Examples)
// ═══════════════════════════════════════════════════════════

class ClaudeAdapter implements IAIAgentPort {
  private apiKey: string;
  private model: string = "claude-sonnet-4-5-20250929";
  private circuitBreaker: CircuitBreaker;
  
  async execute(request: AIRequest): Promise<AIResponse> {
    // Circuit breaker check
    if (this.circuitBreaker.isOpen()) {
      throw new Error("Circuit breaker open for Claude");
    }
    
    try {
      const response = await fetch("https://api.anthropic.com/v1/messages", {
        method: "POST",
        headers: {
          "x-api-key": this.apiKey,
          "anthropic-version": "2023-06-01",
          "content-type": "application/json",
        },
        body: JSON.stringify({
          model: this.model,
          max_tokens: request.max_tokens,
          temperature: request.temperature || 0.3,
          messages: request.messages,
        }),
      });
      
      const data = await response.json();
      
      // Transform to domain model
      return {
        content: data.content[0].text,
        confidence: this.calculateConfidence(data),
        safety_check: this.performSafetyCheck(data),
        provenance: {
          agent: "claude",
          model_version: this.model,
          timestamp: new Date(),
          temperature: request.temperature || 0.3,
          max_tokens: request.max_tokens,
        },
      };
    } catch (error) {
      this.circuitBreaker.recordFailure();
      throw error;
    }
  }
  
  private calculateConfidence(response: any): number {
    // Implement confidence scoring logic
    // Based on: stop_reason, response length, token usage
    return 0.85; // Placeholder
  }
  
  private performSafetyCheck(response: any): SafetyCheck {
    // Implement safety validation
    return {
      hallucination_risk: "low",
      ethical_concern: false,
      requires_human_review: false,
    };
  }
}
```

---

## 3. Data Flow & State Management

### 3.1 AI Orchestration Pipeline (Conductor Pattern)

```typescript
/**
 * ═══════════════════════════════════════════════════════════
 * CONDUCTOR ORCHESTRATOR
 * Implements parallel AI execution with HITL gates
 * ═══════════════════════════════════════════════════════════
 */

class AIConductor {
  constructor(
    private aiAgents: Map<string, IAIAgentPort>,
    private eventBus: IEventPublisher,
    private tokenBudget: TokenBudgetManager
  ) {}
  
  async executeTask(task: Task, intent: HumanIntent): Promise<OrchestrationResult> {
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    // PHASE 1: Intent Capture & Validation
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    const session = await this.createSession(task, intent);
    
    // Check token budget
    if (!await this.tokenBudget.canExecute(session.user_id, intent.estimated_tokens)) {
      throw new InsufficientTokensError();
    }
    
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    // PHASE 2: Parallel AI Execution
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    const [coderResult, reviewerResult, designerResult] = await Promise.allSettled([
      this.invokeAgent("claude", "coder", intent),
      this.invokeAgent("claude", "reviewer", intent),
      this.invokeAgent("gemini", "designer", intent),
    ]);
    
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    // PHASE 3: Confidence Evaluation
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    const results = this.collectResults([coderResult, reviewerResult, designerResult]);
    const avgConfidence = this.calculateAverageConfidence(results);
    
    // CRITICAL: Human Review Gate
    if (avgConfidence < 0.85 || this.hasSafetyFlags(results)) {
      await this.eventBus.publish(new HumanReviewRequiredEvent(session.id, results));
      return {
        status: "awaiting_human_review",
        session_id: session.id,
        results,
        reason: avgConfidence < 0.85 ? "low_confidence" : "safety_concern",
      };
    }
    
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    // PHASE 4: Synthesis (ChatGPT as Conductor)
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    const synthesisRequest: AIRequest = {
      messages: [
        {
          role: "system",
          content: "You are the conductor. Synthesize these AI agent outputs into coherent action.",
        },
        {
          role: "user",
          content: JSON.stringify(results),
        },
      ],
      max_tokens: 2000,
    };
    
    const synthesis = await this.aiAgents.get("chatgpt")!.execute(synthesisRequest);
    
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    // PHASE 5: Final Human Approval Gate
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    await this.eventBus.publish(new FinalApprovalRequiredEvent(session.id, synthesis));
    
    return {
      status: "awaiting_final_approval",
      session_id: session.id,
      synthesis,
      confidence: avgConfidence,
    };
  }
  
  private async invokeAgent(
    agentName: string,
    role: string,
    intent: HumanIntent
  ): Promise<AgentInvocation> {
    const agent = this.aiAgents.get(agentName);
    if (!agent) throw new Error(`Agent ${agentName} not found`);
    
    const startTime = Date.now();
    
    const response = await agent.execute({
      messages: this.buildPrompt(role, intent),
      max_tokens: 4000,
      temperature: 0.3,
    });
    
    return {
      agent: agentName,
      role,
      prompt: JSON.stringify(intent),
      response,
      tokens_used: response.provenance.max_tokens,
      execution_time_ms: Date.now() - startTime,
      invoked_at: new Date(),
    };
  }
}
```

### 3.2 Sequence Diagrams

#### 3.2.1 Task Creation Flow

```
User                 API             Conductor          Claude          Gemini          DB
 │                    │                  │                 │               │             │
 │─Create Task────────>│                 │                 │               │             │
 │                    │─Validate────────>│                 │               │             │
 │                    │                  │─Check Budget────────────────────────────────>│
 │                    │                  │<─Budget OK──────────────────────────────────│
 │                    │                  │                 │               │             │
 │                    │                  │─Execute (async)>│               │             │
 │                    │                  │────────────────────Execute────>│             │
 │                    │                  │                 │               │             │
 │<─Task Created──────│                  │                 │               │             │
 │   (202 Accepted)   │                  │                 │               │             │
 │                    │                  │<─Response───────│               │             │
 │                    │                  │<────────────────────Response────│             │
 │                    │                  │                 │               │             │
 │                    │                  │─Calculate Confidence───────────────────────>│
 │                    │                  │                 │               │             │
 │                    │                  │─[IF confidence < 0.85]          │             │
 │                    │                  │─Publish HumanReviewRequired Event────────>│
 │                    │                  │                 │               │             │
 │<─Notification──────────────────────────────────────────────────────────────────────│
 │  "Review Required" │                  │                 │               │             │
```

#### 3.2.2 Human Review Flow

```
User                 UI              ReviewService      AISession        DB          EventBus
 │                    │                     │               │             │             │
 │─View Review────────>│                    │               │             │             │
 │                    │─Get Session─────────>│              │             │             │
 │                    │                     │─Load──────────>│            │             │
 │                    │                     │<─Session Data─│             │             │
 │<─Display Results───│                     │               │             │             │
 │   with Diff        │                     │               │             │             │
 │                    │                     │               │             │             │
 │─[Approve]──────────>│                    │               │             │             │
 │                    │─Record Review───────>│              │             │             │
 │                    │                     │─Save Review───────────────>│             │
 │                    │                     │─Publish ApprovedEvent──────────────────>│
 │                    │                     │               │             │             │
 │                    │                     │─Execute Merge ────────────>│             │
 │<─Success───────────│                     │               │             │             │
 │                    │                     │               │             │             │
 │─[Reject]───────────>│                    │               │             │             │
 │  + Reason          │─Record Rejection───>│              │             │             │
 │                    │                     │─Save to human_intervention_points────>│
 │                    │                     │─Publish RejectedEvent──────────────────>│
 │<─Acknowledged──────│                     │               │             │             │
```

### 3.3 Event Schema (Event Sourcing)

```typescript
// ═══════════════════════════════════════════════════════════
// DOMAIN EVENTS
// ═══════════════════════════════════════════════════════════

type DomainEvent =
  | TaskCreatedEvent
  | AISessionStartedEvent
  | AgentInvokedEvent
  | HumanReviewRequiredEvent
  | HumanApprovedEvent
  | HumanRejectedEvent
  | HumanModifiedEvent
  | CodeMergedEvent
  | TokenBudgetExceededEvent;

interface BaseEvent {
  id: UUID;
  type: string;
  aggregate_id: UUID;
  aggregate_type: "Task" | "AISession" | "Project";
  user_id: UUID;
  timestamp: DateTime;
  metadata: Record<string, any>;
}

interface HumanReviewRequiredEvent extends BaseEvent {
  type: "HumanReviewRequired";
  session_id: UUID;
  ai_outputs: AIResponse[];
  reason: "low_confidence" | "safety_concern" | "critical_operation";
  confidence_scores: number[];
}

interface HumanApprovedEvent extends BaseEvent {
  type: "HumanApproved";
  session_id: UUID;
  reviewer_id: UUID;
  approved_at: DateTime;
}

interface HumanRejectedEvent extends BaseEvent {
  type: "HumanRejected";
  session_id: UUID;
  reviewer_id: UUID;
  reason: string;
  ai_suggestion: string;
  rejected_at: DateTime;
}
```

---

## 4. Infrastructure Layer

### 4.1 Event Bus Architecture

```yaml
Event Bus Design:
  MVP Phase (Q4 2025):
    Provider: Redis Streams
    Justification: |
      - Simpler setup than Kafka
      - Built-in persistence
      - Consumer groups for scalability
      - Good for <10k events/sec
    
    Stream Structure:
      exiton:events:tasks
      exiton:events:ai_sessions
      exiton:events:human_reviews
      exiton:events:notifications
    
    Consumer Groups:
      - notification_service
      - analytics_service
      - audit_logger
  
  Scale Phase (2026+):
    Provider: Apache Kafka
    Justification: |
      - Handle >100k events/sec
      - Better durability guarantees
      - Ecosystem tooling (Connect, Streams)
    
    Topics:
      exiton.tasks.created
      exiton.ai.session.started
      exiton.human.review.required
      exiton.human.decision.made
```

#### Implementation Example

```typescript
// Redis Streams Event Bus Implementation
class RedisEventBus implements IEventPublisher {
  constructor(private redis: Redis) {}
  
  async publish(event: DomainEvent): Promise<void> {
    const streamKey = `exiton:events:${event.aggregate_type.toLowerCase()}`;
    
    await this.redis.xadd(
      streamKey,
      "*", // Auto-generate ID
      "event_type", event.type,
      "payload", JSON.stringify(event),
      "timestamp", event.timestamp.toISOString()
    );
    
    // Also publish to project-specific stream for real-time UI
    if (event.metadata.project_id) {
      await this.redis.xadd(
        `exiton:project:${event.metadata.project_id}:events`,
        "*",
        "event_type", event.type,
        "payload", JSON.stringify(event)
      );
    }
  }
  
  async publishBatch(events: DomainEvent[]): Promise<void> {
    const pipeline = this.redis.pipeline();
    
    events.forEach(event => {
      const streamKey = `exiton:events:${event.aggregate_type.toLowerCase()}`;
      pipeline.xadd(
        streamKey,
        "*",
        "event_type", event.type,
        "payload", JSON.stringify(event)
      );
    });
    
    await pipeline.exec();
  }
}
```

### 4.2 Vector Database (RAG System)

```yaml
Vector Store Design:
  Provider: Pinecone
  Justification: |
    - Managed service (less ops overhead)
    - Excellent metadata filtering
    - Good for AI context retrieval
    - Alternative: Weaviate (self-hosted option)
  
  Index Structure:
    Name: exiton-context-prod
    Dimension: 1536 (OpenAI ada-002 embeddings)
    Metric: cosine
    
    Metadata Schema:
      - project_id: string
      - user_id: string
      - context_type: "conversation" | "code" | "review"
      - timestamp: int64
      - agent: string
      - confidence: float
  
  Usage Pattern:
    1. User starts AI session
    2. Retrieve last 50 relevant contexts (semantic search)
    3. Compress to fit within token window
    4. Include in AI prompt
    5. Store new interaction as vector
```

#### Implementation

```typescript
class PineconeVectorStore implements IVectorStorePort {
  constructor(
    private client: PineconeClient,
    private openai: OpenAI
  ) {}
  
  async upsertContext(projectId: UUID, context: AIContext): Promise<void> {
    // Generate embedding
    const embedding = await this.openai.embeddings.create({
      model: "text-embedding-ada-002",
      input: context.content,
    });
    
    // Upsert to Pinecone
    await this.client.Index("exiton-context-prod").upsert([
      {
        id: context.id,
        values: embedding.data[0].embedding,
        metadata: {
          project_id: projectId,
          user_id: context.user_id,
          context_type: context.type,
          timestamp: context.timestamp.getTime(),
          agent: context.agent,
          confidence: context.confidence,
        },
      },
    ]);
  }
  
  async searchSimilar(
    projectId: UUID,
    query: string,
    limit: number = 10
  ): Promise<AIContext[]> {
    // Generate query embedding
    const queryEmbedding = await this.openai.embeddings.create({
      model: "text-embedding-ada-002",
      input: query,
    });
    
    // Search Pinecone
    const results = await this.client.Index("exiton-context-prod").query({
      vector: queryEmbedding.data[0].embedding,
      topK: limit,
      filter: { project_id: { $eq: projectId } },
      includeMetadata: true,
    });
    
    // Transform to domain model
    return results.matches.map(match => ({
      id: match.id,
      content: match.metadata.content,
      type: match.metadata.context_type,
      agent: match.metadata.agent,
      confidence: match.metadata.confidence,
      timestamp: new Date(match.metadata.timestamp),
    }));
  }
  
  async compressContext(contexts: AIContext[]): Promise<AIContext> {
    // Use Claude for semantic compression
    const compressionPrompt = `
      Compress the following conversation contexts into a concise summary
      that preserves key decisions, technical details, and project intent:
      
      ${contexts.map(c => c.content).join("\n\n---\n\n")}
      
      Output a summary in under 500 tokens.
    `;
    
    // Call compression service (Claude with low temperature)
    const compressed = await this.compressionService.compress(compressionPrompt);
    
    return {
      id: generateUUID(),
      content: compressed,
      type: "compressed_summary",
      agent: "claude",
      confidence: 0.9,
      timestamp: new Date(),
    };
  }
}
```

### 4.3 Token Budget System

```typescript
/**
 * ═══════════════════════════════════════════════════════════
 * TOKEN BUDGET MANAGER
 * Prevents quota exhaustion and manages costs
 * ═══════════════════════════════════════════════════════════
 */

interface TierLimits {
  free: { monthly_tokens: 100_000; max_session_tokens: 10_000 };
  pro: { monthly_tokens: 1_000_000; max_session_tokens: 50_000 };
  team: { monthly_tokens: 5_000_000; max_session_tokens: 100_000 };
}

class TokenBudgetManager {
  private tierLimits: TierLimits = {
    free: { monthly_tokens: 100_000, max_session_tokens: 10_000 },
    pro: { monthly_tokens: 1_000_000, max_session_tokens: 50_000 },
    team: { monthly_tokens: 5_000_000, max_session_tokens: 100_000 },
  };
  
  constructor(
    private db: IDatabase,
    private cache: Redis
  ) {}
  
  async canExecute(userId: UUID, estimatedTokens: number): Promise<boolean> {
    const budget = await this.getBudget(userId);
    const tier = budget.tier;
    const limits = this.tierLimits[tier];
    
    // Check monthly limit
    if (budget.used_this_month + estimatedTokens > limits.monthly_tokens) {
      await this.publishEvent(new TokenBudgetExceededEvent(userId, tier));
      return false;
    }
    
    // Check per-session limit
    if (estimatedTokens > limits.max_session_tokens) {
      throw new Error(`Session exceeds ${tier} tier limit`);
    }
    
    return true;
  }
  
  async recordUsage(userId: UUID, tokensUsed: number): Promise<void> {
    // Atomic increment in Redis for real-time tracking
    const cacheKey = `token_budget:${userId}:${this.getCurrentMonth()}`;
    await this.cache.incrby(cacheKey, tokensUsed);
    await this.cache.expire(cacheKey, 60 * 60 * 24 * 32); // 32 days
    
    // Async persist to DB
    await this.db.query(`
      UPDATE token_budgets
      SET used_this_month = used_this_month + $1,
          updated_at = NOW()
      WHERE user_id = $2
    `, [tokensUsed, userId]);
  }
  
  private getCurrentMonth(): string {
    return new Date().toISOString().slice(0, 7); // "2025-10"
  }
}
```

### 4.4 Circuit Breaker Implementation

```typescript
/**
 * Circuit Breaker for AI Provider Resilience
 */

class CircuitBreaker {
  private failures: number = 0;
  private lastFailureTime?: Date;
  private state: "closed" | "open" | "half-open" = "closed";
  
  constructor(
    private threshold: number = 5,
    private timeout: number = 60000 // 60 seconds
  ) {}
  
  isOpen(): boolean {
    if (this.state === "open") {
      // Check if timeout has passed
      if (this.lastFailureTime &&
          Date.now() - this.lastFailureTime.getTime() > this.timeout) {
        this.state = "half-open";
        this.failures = 0;
        return false;
      }
      return true;
    }
    return false;
  }
  
  recordSuccess(): void {
    this.failures = 0;
    this.state = "closed";
  }
  
  recordFailure(): void {
    this.failures++;
    this.lastFailureTime = new Date();
    
    if (this.failures >= this.threshold) {
      this.state = "open";
      console.error(`Circuit breaker opened after ${this.failures} failures`);
    }
  }
}

// Usage in AI Adapter
class ResilientClaudeAdapter implements IAIAgentPort {
  private circuitBreaker = new CircuitBreaker(5, 60000);
  private fallback: IAIAgentPort; // GPT-4 fallback
  
  async execute(request: AIRequest): Promise<AIResponse> {
    if (this.circuitBreaker.isOpen()) {
      console.warn("Claude circuit breaker open, using fallback");
      return this.fallback.execute(request);
    }
    
    try {
      const response = await this.callClaude(request);
      this.circuitBreaker.recordSuccess();
      return response;
    } catch (error) {
      this.circuitBreaker.recordFailure();
      
      // Fallback on failure
      console.error("Claude failed, falling back to GPT-4", error);
      return this.fallback.execute(request);
    }
  }
}
```

---

## 5. Security & Human-in-the-Loop Controls

### 5.1 Validation Workflow

```typescript
/**
 * ═══════════════════════════════════════════════════════════
 * AI OUTPUT VALIDATION PIPELINE
 * Multi-layered safety checks before execution
 * ═══════════════════════════════════════════════════════════
 */

class AIOutputValidator {
  async validate(response: AIResponse, context: ValidationContext): Promise<ValidationResult> {
    const checks: ValidationCheck[] = [];
    
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    // CHECK 1: Confidence Threshold
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    checks.push({
      name: "confidence_threshold",
      passed: response.confidence >= context.user_threshold,
      severity: response.confidence < 0.7 ? "critical" : "warning",
      message: `Confidence: ${response.confidence}`,
    });
    
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    // CHECK 2: Hallucination Risk
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    checks.push({
      name: "hallucination_check",
      passed: response.safety_check.hallucination_risk !== "high",
      severity: "critical",
      message: `Risk level: ${response.safety_check.hallucination_risk}`,
    });
    
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    // CHECK 3: Critical Operation Detection
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    const isCritical = this.detectCriticalOperation(response.content, context);
    checks.push({
      name: "critical_operation",
      passed: !isCritical,
      severity: "blocker",
      message: isCritical ? "Requires mandatory human approval" : "Non-critical",
    });
    
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    // CHECK 4: Ethical Concerns
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    checks.push({
      name: "ethical_check",
      passed: !response.safety_check.ethical_concern,
      severity: "blocker",
      message: response.safety_check.ethical_concern ? "Ethical flag raised" : "Clear",
    });
    
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    // DETERMINE OUTCOME
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    const hasBlocker = checks.some(c => !c.passed && c.severity === "blocker");
    const hasCritical = checks.some(c => !c.passed && c.severity === "critical");
    
    if (hasBlocker || isCritical) {
      return {
        status: "HUMAN_APPROVAL_REQUIRED",
        checks,
        reason: "Critical operation or blocker detected",
      };
    }
    
    if (hasCritical) {
      return {
        status: "HUMAN_REVIEW_REQUIRED",
        checks,
        reason: "Critical issues detected",
      };
    }
    
    return {
      status: "APPROVED",
      checks,
    };
  }
  
  private detectCriticalOperation(content: string, context: ValidationContext): boolean {
    const criticalPatterns = [
      /delete.*database/i,
      /drop.*table/i,
      /rm -rf/i,
      /force.*push/i,
      /production.*deploy/i,
      /stripe.*charge/i,
      /billing.*update/i,
    ];
    
    return criticalPatterns.some(pattern => pattern.test(content));
  }
}
```

### 5.2 Human Intervention Logging

```sql
-- ═══════════════════════════════════════════════════════════
-- HUMAN INTERVENTION POINTS TABLE
-- Critical for learning and transparency
-- ═══════════════════════════════════════════════════════════

CREATE TABLE human_intervention_points (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  session_id UUID NOT NULL REFERENCES ai_sessions(id),
  user_id UUID NOT NULL REFERENCES users(id),
  task_id UUID REFERENCES tasks(id),
  
  -- What happened
  intervention_type VARCHAR(50) NOT NULL
    CHECK (intervention_type IN (
      'override', 'rejection', 'modification', 
      'approval', 'clarification_request'
    )),
  
  -- AI's suggestion
  ai_agent VARCHAR(20) NOT NULL,
  ai_suggestion JSONB NOT NULL,
  ai_confidence FLOAT NOT NULL,
  ai_reasoning TEXT,
  
  -- Human's decision
  human_decision JSONB NOT NULL,
  reason TEXT NOT NULL,
  modifications JSONB,
  
  -- Context
  context JSONB,
  timestamp TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  
  -- Metadata for learning
  tags TEXT[],
  category VARCHAR(100),
  
  CONSTRAINT valid_confidence CHECK (ai_confidence BETWEEN 0 AND 1)
);

CREATE INDEX idx_hip_user_id ON human_intervention_points(user_id);
CREATE INDEX idx_hip_session_id ON human_intervention_points(session_id);
CREATE INDEX idx_hip_intervention_type ON human_intervention_points(intervention_type);
CREATE INDEX idx_hip_timestamp ON human_intervention_points(timestamp DESC);

-- For ML analysis
CREATE INDEX idx_hip_confidence ON human_intervention_points(ai_confidence);
CREATE INDEX idx_hip_agent ON human_intervention_points(ai_agent);
```

### 5.3 Approval Workflow State Machine

```typescript
type ReviewState = 
  | "pending_review"
  | "under_review"
  | "approved"
  | "rejected"
  | "modified"
  | "expired";

class HumanReviewWorkflow {
  async startReview(sessionId: UUID, aiOutputs: AIResponse[]): Promise<ReviewRequest> {
    const review = await this.db.createReview({
      session_id: sessionId,
      state: "pending_review",
      ai_outputs: aiOutputs,
      expires_at: new Date(Date.now() + 24 * 60 * 60 * 1000), // 24 hours
      created_at: new Date(),
    });
    
    // Send notification
    await this.notificationService.send({
      user_id: review.user_id,
      type: "review_required",
      title: "AI Output Requires Your Review",
      body: `Session ${sessionId} is awaiting your approval`,
      action_url: `/review/${review.id}`,
    });
    
    return review;
  }
  
  async approve(reviewId: UUID, userId: UUID): Promise<void> {
    const review = await this.db.getReview(reviewId);
    
    // Validate state transition
    if (review.state !== "pending_review" && review.state !== "under_review") {
      throw new Error("Invalid state transition");
    }
    
    // Record approval
    await this.db.updateReview(reviewId, {
      state: "approved",
      reviewer_id: userId,
      reviewed_at: new Date(),
    });
    
    // Publish event
    await this.eventBus.publish(new HumanApprovedEvent({
      id: generateUUID(),
      type: "HumanApproved",
      aggregate_id: review.session_id,
      aggregate_type: "AISession",
      user_id: userId,
      timestamp: new Date(),
      session_id: review.session_id,
      reviewer_id: userId,
      approved_at: new Date(),
    }));
    
    // Continue orchestration
    await this.orchestrator.continueAfterApproval(review.session_id);
  }
  
  async reject(reviewId: UUID, userId: UUID, reason: string): Promise<void> {
    const review = await this.db.getReview(reviewId);
    
    // Record rejection
    await this.db.updateReview(reviewId, {
      state: "rejected",
      reviewer_id: userId,
      rejection_reason: reason,
      reviewed_at: new Date(),
    });
    
    // CRITICAL: Log for learning
    await this.db.logInterventionPoint({
      session_id: review.session_id,
      user_id: userId,
      intervention_type: "rejection",
      ai_agent: review.ai_outputs[0].provenance.agent,
      ai_suggestion: review.ai_outputs,
      ai_confidence: review.ai_outputs[0].confidence,
      human_decision: { rejected: true },
      reason,
      timestamp: new Date(),
    });
    
    // Publish event
    await this.eventBus.publish(new HumanRejectedEvent({
      id: generateUUID(),
      type: "HumanRejected",
      aggregate_id: review.session_id,
      aggregate_type: "AISession",
      user_id: userId,
      timestamp: new Date(),
      session_id: review.session_id,
      reviewer_id: userId,
      reason,
      ai_suggestion: JSON.stringify(review.ai_outputs),
      rejected_at: new Date(),
    }));
  }
}
```

---

## 6. Deployment & Scalability

### 6.1 Environment Architecture

```yaml
Environments:
  Development:
    Backend: localhost:3000
    Database: PostgreSQL (Docker)
    Redis: localhost:6379
    AI: Test API keys (limited quota)
    Vector DB: Pinecone free tier
    Domain: dev.exiton.local
    
  Staging:
    Backend: Railway (staging-api.exiton.app)
    Database: Railway PostgreSQL
    Redis: Upstash Redis
    AI: Production keys (rate limited)
    Vector DB: Pinecone staging index
    Domain: staging.exiton.app
    Purpose: Pre-production testing, client demos
    
  Production:
    Frontend: Vercel (exiton.app)
    Backend: Railway (api.exiton.app)
    Database: Railway PostgreSQL (HA setup)
    Redis: Upstash Redis (Multi-region)
    AI: Production keys (full quota)
    Vector DB: Pinecone production index
    CDN: Cloudflare
    Monitoring: Datadog / Sentry
    Domain: exiton.app
```

### 6.2 API Gateway Design

```typescript
/**
 * ═══════════════════════════════════════════════════════════
 * API GATEWAY ROUTING
 * Rate limiting, auth, and request routing
 * ═══════════════════════════════════════════════════════════
 */

// Next.js API Routes Structure
/*
/api
  /v1
    /auth
      /login
      /logout
      /me
    /projects
      /[id]
      /[id]/tasks
      /[id]/ai-sessions
    /tasks
      /[id]
      /[id]/execute-ai
    /ai
      /orchestrate
      /session/[id]
      /session/[id]/approve
      /session/[id]/reject
    /reviews
      /pending
      /[id]
      /[id]/approve
      /[id]/reject
    /admin
      /users
      /analytics
*/

// Middleware Stack
export const apiMiddleware = [
  corsMiddleware,
  authMiddleware,
  rateLimitMiddleware,
  validationMiddleware,
  loggingMiddleware,
];

// Rate Limiting by Tier
const rateLimits = {
  free: {
    requests_per_minute: 10,
    concurrent_ai_sessions: 1,
  },
  pro: {
    requests_per_minute: 60,
    concurrent_ai_sessions: 3,
  },
  team: {
    requests_per_minute: 300,
    concurrent_ai_sessions: 10,
  },
};
```

### 6.3 Version Locking Strategy

```typescript
/**
 * AI Model Version Management
 * Ensures deterministic behavior across deployments
 */

const AI_MODEL_VERSIONS = {
  claude: {
    coder: "claude-sonnet-4-5-20250929",
    reviewer: "claude-sonnet-4-5-20250929",
  },
  gemini: {
    designer: "gemini-2.0-flash-exp",
  },
  openai: {
    conductor: "gpt-4-turbo-2024-04-09",
    embeddings: "text-embedding-ada-002",
  },
} as const;

// Version audit log
interface ModelVersionLog {
  deployed_at: DateTime;
  versions: typeof AI_MODEL_VERSIONS;
  changed_by: string;
  reason: string;
}

// Before deploying model version changes
async function validateModelChange(newVersions: typeof AI_MODEL_VERSIONS) {
  // Run regression tests
  const testResults = await runRegressionTests(newVersions);
  
  if (testResults.failureRate > 0.05) {
    throw new Error("Model version change increases failure rate above 5%");
  }
  
  // Log version change
  await db.logModelVersionChange({
    deployed_at: new Date(),
    versions: newVersions,
    changed_by: process.env.DEPLOY_USER,
    reason: process.env.DEPLOY_REASON,
  });
}
```

### 6.4 Scalability Considerations

```typescript
/**
 * Horizontal Scaling Strategy
 */

const scalingConfig = {
  // API Servers
  api_servers: {
    min_instances: 2,
    max_instances: 20,
    target_cpu_utilization: 70,
    scale_up_threshold: 80,
    scale_down_threshold: 30,
  },
  
  // Background Workers (AI execution)
  ai_workers: {
    min_instances: 2,
    max_instances: 50,
    queue_depth_threshold: 100,
    tasks_per_worker: 5, // Concurrent AI sessions
  },
  
  // Database Connection Pool
  database: {
    pool_size: 20,
    max_connections: 100,
    idle_timeout_ms: 30000,
  },
  
  // Redis Cache
  cache: {
    max_memory: "2gb",
    eviction_policy: "allkeys-lru",
    max_connections: 50,
  },
};

/**
 * Load Balancing Strategy
 */
const loadBalancer = {
  algorithm: "least_connections",
  health_check: {
    path: "/api/health",
    interval_seconds: 30,
    timeout_seconds: 5,
    unhealthy_threshold: 3,
  },
  sticky_sessions: true, // For WebSocket connections
  session_cookie: "exiton_session",
};
```

---

## 7. Roadmap Alignment

### 7.1 Technical Milestones by Phase

```yaml
MVP v2 (Q4 2025 — October–November):
  Duration: 6 weeks
  Goal: Core infrastructure + Basic AI orchestration
  
  Deliverables:
    Backend:
      ✓ Hexagonal core architecture setup
      ✓ PostgreSQL schema + Prisma ORM
      ✓ Redis event bus (basic)
      ✓ Claude + Gemini adapters
      ✓ Token budget system
      ✓ Human review workflow (basic)
      
    Frontend:
      ✓ Next.js 15 setup + Auth
      ✓ Task Board (basic UI)
      ✓ AI Companion Panel
      ✓ Review interface
      
    Infrastructure:
      ✓ Railway deployment
      ✓ CI/CD pipeline (GitHub Actions)
      ✓ Monitoring setup (Sentry)
      
  Success Criteria:
    - Can create tasks and invoke Claude for code generation
    - Human can approve/reject AI outputs
    - Token usage tracked per user
    - Basic event logging functional

Alpha v1 (Q4 2025 — November–December):
  Duration: 4 weeks
  Goal: Advanced orchestration + RAG memory
  
  Deliverables:
    Backend:
      ✓ Parallel AI execution (Conductor pattern)
      ✓ Pinecone integration (RAG)
      ✓ Context compression service
      ✓ Multi-agent consensus logic
      ✓ Circuit breaker implementation
      
    Frontend:
      ✓ 3-column Studio layout
      ✓ AI Sigil drag-and-drop
      ✓ Command Palette (Cmd+K)
      ✓ Real-time notifications (WebSocket)
      
    AI:
      ✓ ChatGPT conductor integration
      ✓ Confidence scoring calibration
      ✓ Explainable AI (reasoning chains)
      
  Success Criteria:
    - AI agents execute in parallel (<30s total)
    - Context retrieval from Pinecone functional
    - Users can see AI reasoning chains
    - Real-time UI updates via WebSocket

Beta v1 (Q4 2025 — December):
  Duration: 4 weeks
  Goal: Polish + Public beta preparation
  
  Deliverables:
    Backend:
      ✓ Event sourcing fully implemented
      ✓ human_intervention_points analysis
      ✓ Advanced safety checks
      ✓ Team collaboration features
      
    Frontend:
      ✓ Focus View (code editor integration)
      ✓ Launch Console
      ✓ Visual polish (neon animations)
      ✓ Mobile responsive
      
    Operations:
      ✓ Staging environment
      ✓ Load testing (1000 concurrent users)
      ✓ Security audit
      ✓ Documentation (user + developer)
      
  Success Criteria:
    - Can handle 100 concurrent AI sessions
    - <200ms API response time (p95)
    - Zero critical security vulnerabilities
    - Beta user onboarding flow complete

Product v1 (Q1 2026):
  Duration: 8 weeks
  Goal: Paid launch + Subscription system
  
  Deliverables:
    Backend:
      ✓ Stripe integration (subscriptions)
      ✓ Team accounts + RBAC
      ✓ Custom AI slot configuration
      ✓ Usage analytics dashboard
      
    Frontend:
      ✓ Pricing page
      ✓ Account settings (billing)
      ✓ Team management UI
      ✓ Onboarding flow v2
      
    Marketing:
      ✓ Landing page redesign
      ✓ Product Hunt launch
      ✓ Documentation site
      
  Success Criteria:
    - 100 paying customers
    - 95% uptime SLA
    - <1% churn rate
    - NPS score >50
```

### 7.2 Dependency Graph

```
┌─────────────────────────────────────────────────────────────┐
│                   MVP v2 FOUNDATION                          │
│  [Hexagonal Core] → [Basic AI] → [Token Budget]             │
└────────────┬────────────────────────────────────────────────┘
             │
             ↓
┌────────────┴────────────────────────────────────────────────┐
│                   ALPHA v1 ENHANCEMENTS                      │
│  [RAG System] → [Parallel Execution] → [Conductor Pattern]  │
│                                                              │
│  Dependencies:                                               │
│    - MVP v2 must be deployed                                │
│    - Pinecone account setup                                 │
│    - OpenAI embeddings API key                              │
└────────────┬────────────────────────────────────────────────┘
             │
             ↓
┌────────────┴────────────────────────────────────────────────┐
│                   BETA v1 POLISH                             │
│  [Event Sourcing] → [Team Features] → [Launch Console]      │
│                                                              │
│  Dependencies:                                               │
│    - Alpha v1 core features stable                          │
│    - Load testing infrastructure ready                      │
└────────────┬────────────────────────────────────────────────┘
             │
             ↓
┌────────────┴────────────────────────────────────────────────┐
│                   PRODUCT v1 LAUNCH                          │
│  [Stripe] → [Marketing] → [Customer Success]                │
│                                                              │
│  Dependencies:                                               │
│    - Beta testing complete (50+ users)                      │
│    - Legal terms finalized                                  │
│    - Support infrastructure ready                           │
└─────────────────────────────────────────────────────────────┘
```

---

## 8. Appendix

### 8.1 Recommended Tools & Libraries

```yaml
Backend Stack:
  Runtime: Node.js 20 LTS (TypeScript 5.3+)
  Framework: Next.js 15.0
  ORM: Prisma 5.0
  Validation: Zod
  Testing: Vitest + Playwright
  
AI SDKs:
  Anthropic: @anthropic-ai/sdk ^0.20.0
  OpenAI: openai ^4.20.0
  Google AI: @google/generative-ai ^0.1.0
  Vercel AI SDK: ai ^3.0.0
  
Data & Events:
  Database: pg (PostgreSQL driver)
  Cache: ioredis ^5.3.0
  Vector: @pinecone-database/pinecone ^1.0.0
  Queue: bullmq ^4.0.0
  
Frontend:
  UI: React 19 + Radix UI
  Styling: Tailwind CSS 4.0
  State: Zustand
  Forms: React Hook Form + Zod
  Animations: Framer Motion
  
DevOps:
  Deployment: Vercel + Railway
  CI/CD: GitHub Actions
  Monitoring: Sentry + Datadog
  Analytics: PostHog
```

### 8.2 Example API Endpoint Definitions

```typescript
/**
 * ═══════════════════════════════════════════════════════════
 * API ENDPOINT SPECIFICATIONS
 * ═══════════════════════════════════════════════════════════
 */

// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
// POST /api/v1/tasks/:id/execute-ai
// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

interface ExecuteAIRequest {
  task_id: string;
  intent: {
    action: "generate_code" | "review_code" | "design_ui" | "write_tests";
    context: string;
    constraints?: string[];
  };
  agents?: {
    coder?: "claude" | "gemini" | "gpt";
    reviewer?: "claude" | "gemini";
    designer?: "gemini" | "claude";
  };
}

interface ExecuteAIResponse {
  session_id: string;
  status: "executing" | "awaiting_review" | "completed";
  estimated_completion_seconds?: number;
  webhook_url?: string; // For async notification
}

// Example usage:
/*
POST /api/v1/tasks/task-123/execute-ai
Authorization: Bearer <token>

{
  "task_id": "task-123",
  "intent": {
    "action": "generate_code",
    "context": "Create a React component for user profile",
    "constraints": ["TypeScript", "Tailwind CSS", "Accessibility"]
  }
}

Response 202 Accepted:
{
  "session_id": "session-456",
  "status": "executing",
  "estimated_completion_seconds": 30,
  "webhook_url": "wss://api.exiton.app/v1/sessions/session-456/stream"
}
*/

// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
// GET /api/v1/reviews/pending
// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

interface PendingReviewsResponse {
  reviews: Array<{
    id: string;
    session_id: string;
    task: {
      id: string;
      title: string;
    };
    ai_outputs: AIResponse[];
    created_at: string;
    expires_at: string;
  }>;
  total: number;
}

// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
// POST /api/v1/reviews/:id/approve
// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

interface ApproveReviewRequest {
  review_id: string;
  modifications?: {
    agent: string;
    modified_output: string;
  }[];
  comment?: string;
}

interface ApproveReviewResponse {
  status: "approved";
  session_id: string;
  next_action: "merge_changes" | "continue_execution" | "complete";
}
```

### 8.3 Database Schema (PostgreSQL)

```sql
-- ═══════════════════════════════════════════════════════════
-- EXITON DATABASE SCHEMA v1.0
-- ═══════════════════════════════════════════════════════════

CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pg_trgm"; -- For text search

-- ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
-- USERS & AUTH
-- ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  name VARCHAR(255),
  subscription_tier VARCHAR(20) NOT NULL DEFAULT 'free'
    CHECK (subscription_tier IN ('free', 'pro', 'team')),
  ai_preferences JSONB NOT NULL DEFAULT '{}',
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE token_budgets (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  tier VARCHAR(20) NOT NULL,
  monthly_limit INTEGER NOT NULL,
  used_this_month INTEGER NOT NULL DEFAULT 0,
  resets_at TIMESTAMPTZ NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  
  UNIQUE(user_id, resets_at)
);

-- ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
-- PROJECTS & TASKS
-- ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

CREATE TABLE projects (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  owner_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  name VARCHAR(255) NOT NULL,
  description TEXT,
  status VARCHAR(20) NOT NULL DEFAULT 'active'
    CHECK (status IN ('active', 'paused', 'completed', 'archived')),
  metadata JSONB NOT NULL DEFAULT '{}',
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE tasks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  title VARCHAR(500) NOT NULL,
  description TEXT,
  status VARCHAR(20) NOT NULL DEFAULT 'todo'
    CHECK (status IN ('todo', 'in_progress', 'review', 'done')),
  assigned_ai_agent VARCHAR(20),
  ai_generated BOOLEAN NOT NULL DEFAULT false,
  confidence_score FLOAT,
  human_review_required BOOLEAN NOT NULL DEFAULT false,
  created_by VARCHAR(10) NOT NULL CHECK (created_by IN ('human', 'ai')),
  metadata JSONB NOT NULL DEFAULT '{}',
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  
  CONSTRAINT valid_confidence CHECK (
    confidence_score IS NULL OR 
    (confidence_score >= 0 AND confidence_score <= 1)
  )
);

-- ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
-- AI SESSIONS & RESPONSES
-- ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

CREATE TABLE ai_sessions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  task_id UUID REFERENCES tasks(id) ON DELETE SET NULL,
  conductor VARCHAR(20) NOT NULL DEFAULT 'chatgpt',
  total_tokens_used INTEGER NOT NULL DEFAULT 0,
  status VARCHAR(30) NOT NULL DEFAULT 'pending'
    CHECK (status IN ('pending', 'executing', 'awaiting_review', 'completed', 'failed')),
  started_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  completed_at TIMESTAMPTZ,
  metadata JSONB NOT NULL DEFAULT '{}'
);

CREATE TABLE agent_invocations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  session_id UUID NOT NULL REFERENCES ai_sessions(id) ON DELETE CASCADE,
  agent VARCHAR(20) NOT NULL,
  role VARCHAR(20) NOT NULL,
  prompt TEXT NOT NULL,
  response JSONB NOT NULL,
  confidence FLOAT NOT NULL,
  tokens_used INTEGER NOT NULL,
  execution_time_ms INTEGER NOT NULL,
  invoked_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  
  CONSTRAINT valid_agent CHECK (agent IN ('claude', 'gemini', 'gpt', 'local')),
  CONSTRAINT valid_role CHECK (role IN ('coder', 'reviewer', 'designer', 'pm')),
  CONSTRAINT valid_confidence CHECK (confidence >= 0 AND confidence <= 1)
);

-- ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
-- HUMAN REVIEWS
-- ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

CREATE TABLE human_reviews (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  session_id UUID NOT NULL REFERENCES ai_sessions(id) ON DELETE CASCADE,
  reviewer_id UUID NOT NULL REFERENCES users(id),
  ai_response_id UUID REFERENCES agent_invocations(id),
  decision VARCHAR(20) NOT NULL CHECK (decision IN ('approved', 'rejected', 'modified')),
  reason TEXT,
  modifications JSONB,
  reviewed_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE human_intervention_points (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  session_id UUID NOT NULL REFERENCES ai_sessions(id),
  user_id UUID NOT NULL REFERENCES users(id),
  task_id UUID REFERENCES tasks(id),
  intervention_type VARCHAR(50) NOT NULL,
  ai_agent VARCHAR(20) NOT NULL,
  ai_suggestion JSONB NOT NULL,
  ai_confidence FLOAT NOT NULL,
  ai_reasoning TEXT,
  human_decision JSONB NOT NULL,
  reason TEXT NOT NULL,
  modifications JSONB,
  context JSONB,
  timestamp TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  tags TEXT[],
  category VARCHAR(100)
);

-- ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
-- INDEXES FOR PERFORMANCE
-- ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

CREATE INDEX idx_projects_owner ON projects(owner_id);
CREATE INDEX idx_tasks_project ON tasks(project_id);
CREATE INDEX idx_tasks_status ON tasks(status);
CREATE INDEX idx_ai_sessions_project ON ai_sessions(project_id);
CREATE INDEX idx_ai_sessions_status ON ai_sessions(status);
CREATE INDEX idx_agent_invocations_session ON agent_invocations(session_id);
CREATE INDEX idx_human_reviews_session ON human_reviews(session_id);
CREATE INDEX idx_hip_user ON human_intervention_points(user_id);
CREATE INDEX idx_hip_timestamp ON human_intervention_points(timestamp DESC);

-- Full-text search on tasks
CREATE INDEX idx_tasks_search ON tasks USING gin(to_tsvector('english', title || ' ' || COALESCE(description, '')));
```

---

## 🌌 Closing Statement

This Architecture Definition Document translates EXITON's **AI-Bebop philosophy** into **executable technical reality**. The hexagonal core ensures flexibility, the event-driven design enables scalability, and the human-in-the-loop controls maintain agency.

**Key Architectural Achievements:**
- ✅ AI providers are hot-swappable adapters
- ✅ Every AI decision includes provenance and confidence
- ✅ Humans retain veto power at all critical junctions
- ✅ System learns from human overrides without becoming opaque
- ✅ Scales from solo developer to team workflows

**Implementation Priority:**
1. **Week 1-2:** Hexagonal core + basic Claude adapter
2. **Week 3-4:** Task management + human review workflow
3. **Week 5-6:** Token budget + event bus foundation
4. **Week 7-8:** Frontend integration + MVP deployment

**The architecture respects the founding principle:**
> *"Structure enables improvisation. Discipline births creativity. EXITON is where AI and humans jam together."*

---

**Status:** ✅ ADD Complete — Implementation Ready  
**Next Document:** Sprint Planning & Task Breakdown  
**Maintain:** AI-Bebop tone, technical precision, human agency

*— Claude Code, Lead Architect*  
*"Elegant systems for human-AI resonance."*