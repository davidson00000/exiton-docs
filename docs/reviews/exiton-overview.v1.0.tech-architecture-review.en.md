🔮 EXITON 上位構想書 v1.0 — Technical Architecture Review
Reviewer: Claude Code (Lead Architect)
Date: 2025-10-15
Tone: AI-Bebop Protocol — Technical Resonance Mode

1. Overall Impression
🎵 Architectural Harmony Assessment: ★★★★☆ (4.2/5.0)
The vision resonates strongly. EXITON's core philosophy—"継続できる知性の場"—translates into a technically sound multi-layered architecture. The document succeeds in bridging product vision with implementation reality, a rare feat in early-stage SaaS design.
Strengths:

Philosophical Clarity: The "AI as band member" metaphor provides clear constraints for interaction design
Layered Thinking: The L1–L4 ecosystem model naturally maps to microservices architecture
Monetization Realism: Staged revenue model aligns with technical maturity phases

Gaps Requiring Attention:

Data Flow Architecture: Inter-layer communication protocols undefined
AI Orchestration Specifics: The ChatGPT → Claude → Gemini pipeline needs explicit state management
Scale Constraints: No discussion of concurrent user limits or AI quota management

The foundation is solid. What's needed now is structural steel beneath the neon glow.

2. Architecture Review
🏗️ System Design Analysis
Current Structure (Implicit)
┌─────────────────────────────────────────┐
│  L4: Culture Layer (Content Delivery)   │
├─────────────────────────────────────────┤
│  L3: Academy (Knowledge Graph)          │
├─────────────────────────────────────────┤
│  L2: Apps (Mobile Sync Nodes)           │
├─────────────────────────────────────────┤
│  L1: SaaS Core ← YOU ARE HERE           │
│  ┌──────────────────────────────────┐  │
│  │ [PM Engine] [AI Layer] [Launch]  │  │
│  └──────────────────────────────────┘  │
└─────────────────────────────────────────┘
```

#### Recommended: **Hexagonal + Event-Driven Architecture**
```
        ┌─────────────────────────────────┐
        │   External Event Bus (Kafka)     │
        │   L4, L3, L2 publish/subscribe   │
        └─────────────────────────────────┘
                      ↕
    ┌─────────────────────────────────────────┐
    │         L1 Core — Hexagonal Design       │
    │                                           │
    │  ┌───────────────────────────────────┐  │
    │  │      Domain Core (Ports)          │  │
    │  │  • Task Management                │  │
    │  │  • Progress Tracking              │  │
    │  │  • Motivation Engine              │  │
    │  └───────────────────────────────────┘  │
    │              ↕         ↕         ↕       │
    │  ┌──────────┐  ┌──────────┐  ┌───────┐ │
    │  │ AI Agent │  │ Database │  │  API  │ │
    │  │ Adapters │  │ Adapters │  │Gateway│ │
    │  └──────────┘  └──────────┘  └───────┘ │
    └─────────────────────────────────────────┘
Why Hexagonal?

Port-Adapter Pattern: Swap AI providers (Claude → GPT-4 → Gemini) without touching core logic
Testability: Mock AI responses during development
Future-Proof: Add local LLM support (Ollama/LM Studio) without refactoring

🗄️ Data Architecture Proposal
yamlCore Data Entities:

User:
  - id (UUID)
  - ai_preferences: { claude_model, gemini_model, tone }
  - subscription_tier

Project:
  - id (UUID)
  - owner_id
  - ai_context_window: [last_50_interactions]
  
Task:
  - id (UUID)
  - project_id
  - ai_generated: boolean
  - ai_agent: "claude" | "gemini" | "chatgpt"
  - confidence_score: 0.0–1.0

AISession:
  - id (UUID)
  - agent_chain: ["chatgpt", "claude", "gemini"]
  - tokens_used
  - human_intervention_points: [timestamp, reason]
```

**Critical Addition**: **`human_intervention_points`** table  
This records when and why humans override AI suggestions—essential for the "human-in-the-loop" philosophy and ML feedback loops.

---

## 3. AI Agent Coordination

### 🎼 Current Orchestration Model
```
ChatGPT (PM) → Claude Code (Dev) → Gemini (UI)
```

**Issue**: This is a **sequential pipeline**, which creates bottlenecks and lacks parallelization.

### ⚡ Proposed: **Conductor Pattern with Parallel Execution**
```
                   ┌─────────────┐
                   │   Human     │
                   │  (Kousuke)  │
                   └──────┬──────┘
                          │
                   ┌──────▼──────────┐
                   │  Conductor AI    │ ← ChatGPT as orchestrator
                   │  (State Machine) │
                   └──────┬───────────┘
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
   ┌────▼────┐      ┌────▼────┐      ┌────▼────┐
   │ Claude  │      │ Claude  │      │ Gemini  │
   │ Coder   │      │Reviewer │      │Designer │
   └────┬────┘      └────┬────┘      └────┬────┘
        │                 │                 │
        └─────────────────┼─────────────────┘
                          │
                   ┌──────▼──────────┐
                   │  Review Gate    │ ← Human approval
                   │  (HITL Check)   │
                   └──────┬──────────┘
                          │
                   ┌──────▼──────────┐
                   │   Deployment    │
                   └─────────────────┘
🔧 Implementation Protocol
pythonclass AIOrchestrator:
    """
    Human-in-the-Loop Coordination Engine
    Philosophy: Humans set intent, AIs execute, humans validate
    """
    
    def execute_task(self, task: Task) -> Result:
        # 1. Intent Capture (Human → System)
        intent = self.capture_human_intent(task)
        
        # 2. Parallel AI Execution
        with ThreadPoolExecutor() as executor:
            code_future = executor.submit(claude_coder, intent)
            review_future = executor.submit(claude_reviewer, intent)
            ui_future = executor.submit(gemini_designer, intent)
        
        # 3. Collect Results
        results = {
            'code': code_future.result(),
            'review': review_future.result(),
            'ui': ui_future.result()
        }
        
        # 4. CRITICAL: Human Review Gate
        if results['review'].confidence < 0.85:
            return self.request_human_review(results)
        
        # 5. Synthesis (ChatGPT as conductor)
        final = chatgpt_conductor.synthesize(results)
        
        # 6. Human Final Approval
        return self.present_for_approval(final)
Key Principles:

No AI makes final decisions without human review
Confidence thresholds trigger human-in-the-loop
All AI outputs tagged with provenance (which agent, when, confidence)


4. Risks & Mitigations
⚠️ Critical Risks
RiskSeverityMitigation StrategyAI Quota Exhaustion🔴 HighImplement token budget system per user tier; queue long tasksInconsistent AI Responses🟡 MediumVersion-lock models; use temperature=0.3 for deterministic tasksContext Window Overflow🔴 HighImplement sliding window with semantic compression (RAG)AI Hallucination in Critical Paths🔴 HighRequire human sign-off for: code deployment, user-facing content, billing changesCross-Agent Context Loss🟡 MediumShared vector database for session memory; use structured handoff protocolsVendor Lock-In🟠 MediumAbstract AI calls behind interfaces; support multiple providers per role
🛡️ Technical Safeguards
typescript// Example: AI Response Validation Layer
interface AIResponse {
  agent: "claude" | "gemini" | "chatgpt";
  content: string;
  confidence: number;
  requires_human_review: boolean;
  safety_check: {
    hallucination_risk: "low" | "medium" | "high";
    ethical_concern: boolean;
  };
}

function validateAIOutput(response: AIResponse): ValidatedResponse {
  // 1. Confidence threshold
  if (response.confidence < 0.7) {
    return { status: "HUMAN_REVIEW_REQUIRED", reason: "Low confidence" };
  }
  
  // 2. Safety checks
  if (response.safety_check.hallucination_risk === "high") {
    return { status: "REJECTED", reason: "Hallucination detected" };
  }
  
  // 3. Critical path protection
  if (isCriticalOperation(response.content)) {
    return { status: "HUMAN_APPROVAL_REQUIRED" };
  }
  
  return { status: "APPROVED", response };
}
```

---

## 5. Suggested Enhancements

### 🚀 Technical Evolution Path

#### **Phase 1: Foundational (MVP v2 — Q4 2025)**
```
✅ Implement hexagonal core architecture
✅ Add AI response validation layer
✅ Build token budget system
✅ Create human-in-the-loop approval gates
✅ Version-lock all AI models
```

#### **Phase 2: Intelligence Layer (Alpha — Q4 2025)**
```
🔹 RAG system for long-term project context
   → Use Pinecone/Weaviate for vector storage
   → Compress conversation history semantically
   
🔹 AI Agent Performance Metrics
   → Track: response time, confidence, human override rate
   → Feed back into agent selection logic
   
🔹 Fallback Hierarchy
   Primary: Claude Sonnet 4.5
   Secondary: GPT-4
   Tertiary: Local LLM (Ollama)
```

#### **Phase 3: Advanced Coordination (Beta — Q4 2025)**
```
🔹 Multi-Agent Consensus Protocol
   → If Claude and Gemini disagree on UI design, 
     present both to human with diff view
   
🔹 Explainable AI Layer
   → Every AI decision comes with reasoning chain
   → Users can inspect "why AI suggested this task"
   
🔹 Learning Loop
   → Track which AI suggestions humans accept/reject
   → Fine-tune agent selection over time
🎨 Architectural Patterns to Adopt

CQRS (Command Query Responsibility Segregation)

Separate AI write operations (task creation) from reads (progress queries)
Enables async AI processing without blocking UI


Event Sourcing for AI Interactions

typescript   Event: AITaskCreated
   Event: HumanApproved
   Event: AIReviewRequested
   Event: HumanOverrode { reason: "AI missed edge case" }

Full audit trail for "how this project evolved"
Enables replay for debugging AI decisions


Circuit Breaker for AI Providers

typescript   if (claude_failure_rate > 0.5) {
     circuit_breaker.open();
     fallback_to(gemini);
   }

6. Next Technical Milestones
📋 Immediate Action Items (Next 2 Weeks)
markdown🎯 Architecture Definition
  □ Finalize hexagonal core boundaries
  □ Define port interfaces for AI adapters
  □ Document data flow diagrams

🎯 AI Integration Spec
  □ Write API contracts for each agent role
  □ Define confidence threshold policies
  □ Create human-review workflow states

🎯 Infrastructure Setup
  □ Choose vector DB for RAG (recommend: Pinecone)
  □ Set up event bus (Kafka/Redis Streams)
  □ Implement token budget service
```

### 🗺️ Technical Roadmap Visualization
```
2025-Q4 (MVP v2)              2025-Q4 (Alpha)         2025-Q4 (Beta)
════════════════              ═══════════════         ══════════════
│                             │                       │
├─ Core Architecture          ├─ RAG Integration      ├─ Multi-Agent
│  • Hexagonal design         │  • Vector storage     │   Consensus
│  • AI validation layer      │  • Context compress   │  • Explain AI
│  • HITL gates               │                       │  • Learning loop
│                             │                       │
├─ Basic Orchestration        ├─ Smart Routing        ├─ Adaptive
│  • Sequential pipeline      │  • Parallel exec      │   Selection
│  • Single human gate        │  • Confidence-based   │  • Auto-improve
│                             │                       │
└─ Manual AI config           └─ Quota management     └─ Self-healing
   • Fixed models             │  • Budget per tier    │   • Circuit break
                              │                       │   • Fallbacks
```

### 🔬 Research Spikes Recommended

1. **AI Context Compression** (1 week)
   - Experiment: Can we compress 100k tokens → 10k while preserving task intent?
   - Tool: LangChain + Claude with summarization

2. **Human Override Patterns** (2 weeks)
   - Build dataset: When do humans reject AI suggestions?
   - Use for: Improving confidence thresholds

3. **Local LLM Viability** (1 week)
   - Test: Can Llama 3 70B replace Claude for simple tasks?
   - Goal: Cost reduction for high-volume users

---

## 🌌 Closing Reflection

> **"The best architecture is one you can hear."**  
> Like a jazz ensemble, EXITON's AI agents must improvise together while following the human conductor's intent. This review proposes the **sheet music**—the protocols, patterns, and safeguards—that let that improvisation happen safely.

### Core Architectural Philosophy
```
Human Intent (🧠)
     ↓
  AI Execution (⚡) — in parallel, with confidence scores
     ↓
  Human Validation (✋) — veto power always present
     ↓
  System Learning (🔄) — patterns feed back into agent selection
What makes this EXITON, not just another AI tool?

Every AI decision has a confidence score and provenance
Humans can inspect the reasoning chain behind any suggestion
The system learns from human overrides without becoming opaque
No AI lock-in: swap providers like changing instruments in a band

The architecture proposed here respects the AI-Bebop philosophy: structure that enables improvisation, rules that create freedom, technology that amplifies humanity.

Status: ✅ Review Complete — Ready for Technical Refinement Phase
Next Step: Architecture Definition Document (ADD) with detailed API specs
Vibe Check: 🟣 Neon resonance maintained, intellect sharpened, structure reinforced
— Claude Code, Lead Architect
"From chaos to clarity, through elegant systems."