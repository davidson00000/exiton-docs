# EXITON System Integration Spec v1.2 — “The Bridge”
**Authors:** ChatGPT (PM) × Claude Code × Gemini CLI × Kousuke Nakamura  
**Status:** Integration Ready / v1.2  
**Tone:** AI-Bebop — Structure × Emotion × Intelligence  
**Created:** 2025-10-15

---

## I. Overview
> “The architecture is the score, the interface is the melody, and the human is the conductor.”

### Purpose
EXITON全体構想（L0〜L4）を技術・体験・運用の3レイヤで統合。  
L1「SaaS Core（PM / Launch Console）」を中核に据え、AI層・UI層・人間介入層を一貫設計として整理する。

### Scope
- 対象：Next.js + Supabase + Claude / Gemini + CI/CD構成  
- 目的：モジュール間連携・AI間同期・人間レビュー制御の仕様定義

---

## II. Integration Model

### Layered Structure
```
L4  Studio / Culture
L3  Academy (Learning & Docs)
L2  Apps (iOS / Web / Tools)
L1  SaaS Core (PM / Launch / AI Layer)
L0  Brand / Vision
```

### Integration Flow
```
┌────────────────────────────┐
│  L4: Studio / Culture       │
├────────────────────────────┤
│  L3: Academy                │
├────────────────────────────┤
│  L2: Apps & Tools           │
├────────────────────────────┤
│  L1: SaaS Core              │
│     ├── PM Engine           │
│     ├── AI Orchestrator     │
│     ├── Launch Console      │
└────────────────────────────┘
```

---

## III. Data Flow & Interface Mapping

### Data Flow Summary
| Flow | Description |
|---|---|
| Task → AI Session | User action triggers AI agent processing |
| AI → Human Review | Claude/Gemini output passes review gate |
| Review → Merge | Human approval triggers Supabase update |
| Notif → Loop | Launch Console sends motivational feedback |

### Core Data Entities
```json
{
  "task": { "id": "uuid", "title": "string", "status": "enum" },
  "ai_session": { "agent": "claude|gemini", "confidence": "float", "content": "text" },
  "human_review": { "reviewer": "user_id", "decision": "approve|reject", "reason": "text" },
  "provenance": { "hash": "sha256", "source": "ai_model", "timestamp": "iso" }
}
```

---

## IV. Implementation Layer Map

### Frontend
| Layer | Framework | Notes |
|---|---|---|
| UI | Next.js (App Router) | Pages / Server Actions |
| Style | Tailwind + shadcn/ui | Unified theme system |
| State | Dexie.js + React Query | Offline caching + async sync |

### Backend
| Component | Tech | Purpose |
|---|---|---|
| Database | Supabase (Postgres) | Persistent storage |
| Event Bus | Kafka / Redis | Asynchronous AI trigger system |
| API Gateway | Node / Express | Webhook + API routing |
| CI/CD | PM2 + GitHub Actions | Deploy / process control |

---

## V. Multi-Agent Collaboration Protocol

### Agent Roles
| Agent | Role | Function |
|---|---|---|
| Claude Code | Architecture / Logic | Implements structure and algorithms |
| Gemini CLI | UX / Visual | Generates interface and design |
| ChatGPT PM | Coordinator | Task orchestration and human review |

### Collaboration Rules
1. ChatGPT assigns task context and execution order.  
2. Claude and Gemini run in parallel (async).  
3. Each AI outputs JSON with confidence + provenance tags.  
4. ChatGPT merges, validates, and submits for human approval.  
5. Human feedback re-enters the system for reinforcement.

---

## VI. Deployment Plan

### Environment Variables
```bash
NEXT_PUBLIC_SUPABASE_URL=https://xxxxx.supabase.co
SUPABASE_SERVICE_ROLE_KEY=********
STRIPE_SECRET_KEY=********
REDIS_URL=redis://localhost:6379
PM2_ENV=production
```

### CI/CD Flow
- GitHub Actions listens for tag push (`v*`)  
- Rsync to `/srv/exiton/app` on VPS  
- PM2 reloads `exiton-app` with environment updates  
- Nginx reverse proxy routes HTTPS traffic

---

## VII. Validation & Human Review Flow

| Condition | Action |
|---|---|
| confidence ≥ 0.85 | Auto-merge |
| confidence < 0.85 | Send to Human Review Queue |
| Manual Override | PM confirmation required |

### HITL Model
```yaml
human_intervention_points:
  - id: uuid
  - timestamp: iso
  - reason: string
  - reviewer: user_id
```

---

## VIII. Roadmap Alignment (Weekly)

| Week | Phase | Owner | Deliverables |
|---|---|---|---|
| 44 | Stripe Integration | Claude | API + Webhook |
| 45 | Launch Console QA | Gemini | UI Polishing |
| 46 | Multi-Agent Test | ChatGPT | Log Sync + Merge |
| 47 | Beta Prep | All | Final Review & Docs |

---

## IX. Appendix — AI Instruction Templates

### Claude
> Generate event-driven architecture within EXITON ADD v1.0 structure.

### Gemini
> Create Figma layout for Launch Console using EXITON UX v1.0 spec.

### ChatGPT
> Merge Claude & Gemini output and align with EXITON v1.1 vision.

---

## Notes
- All agents must tag output with `{ "provenance": "...", "confidence": 0.xx }`
- Human review logs stored under `human_intervention_points` in Supabase.
- Each phase aligns with corresponding Notion project board.

---

## Credits
> EXITON — A Resonance System for Human Intelligence  
> *“Structure enables freedom. Discipline births improvisation.”*  
> — Kousuke Nakamura, 2025
