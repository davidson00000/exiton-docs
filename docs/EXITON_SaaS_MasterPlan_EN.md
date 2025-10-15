# EXITON SaaS Master Plan (v1.0)

**Document Purpose:**  
This document defines the upper-level concept and dual-SaaS architecture of **EXITON**,  
a lightweight ecosystem for individual creators and small teams to move from *idea → product → launch*.  
It is designed for multi-AI collaborative review (Claude, Gemini, ChatGPT, and Google AI Studio).

---

## 1. Philosophy

EXITON aims to **empower independent developers** to sustain creative momentum  
without being overwhelmed by AI costs or complexity.

> “Humans ideate, tools facilitate, AI supports only when necessary.”

EXITON avoids over-automation and focuses on maintaining the *tactile feel of human thinking* —  
manual, mindful, and deliberate. AI is positioned as an *optional assist layer*, not as the core.

---

## 2. SaaS Layer Architecture

EXITON consists of two complementary SaaS products under the same design philosophy:

### 🧩 A. EXITON PM Tool — From Idea to Product

A local-first productivity and project management tool for personal makers.

| Module | Role | Description |
|--------|------|-------------|
| **Idea Graph** | Ideation & Structure | Human-first idea mapping. Merges Freemap and Whiteboard. Supports templates (KPT, YWT, logic tree) and simple graph visualization via Mermaid.js. |
| **Project Selector** | Validation & Prioritization | Lightweight decision assistant to evaluate which ideas should evolve into projects. |
| **PM Core** | Task Planning & Execution | Core project dashboard (based on MVP prototype `https://mvp.pm.exiton.net/#/ideas`). Enables manual planning, tagging, and progress tracking. |
| **Insight Dashboard** | Retrospective & Feedback | AI-optional analytics. Aggregates completed tasks, time usage, and categories into trend graphs (no AI inference by default). |
| **Launch Console (Lite)** | Transition to Market | Human-driven comparison UI. Displays *“MVP (left)”* vs *“Product (right)”*, encouraging self-evaluation before external launch. |

> ⚙️ Tech stack: **Next.js + Supabase + IndexedDB (cache) + D3/Mermaid.js**  
> Local-first, AI-optional, and cost-efficient.

---

### 🚀 B. EXITON Launch — From Product to Market

A lightweight matching platform connecting **individual developers and early adopters**.  
Its role is to close the loop from *build → feedback → growth*.

| Function | Description |
|-----------|-------------|
| **User Discovery** | Helps creators find the first 10 users willing to test early builds. |
| **Feedback Loop** | Structured, template-based feedback form (text + rating). |
| **Progress Showcase** | Makers can post weekly updates (“What changed this week”). |
| **Growth Insights (optional AI)** | Suggests potential audience expansion or pricing hypotheses. |

> Core value: enables small creators to validate, communicate, and grow their ideas efficiently.  

---

## 3. Relationship between PM Tool and Launch

| Flow Stage | Main Tool | Key Data Shared |
|-------------|------------|----------------|
| Ideation | PM Tool (Idea Graph) | Idea title, tag, category |
| Validation | Project Selector → Launch | Target user, value proposition |
| Execution | PM Core | Milestones, deliverables |
| Feedback | Launch | User feedback summaries |
| Review | PM Tool (Insight Dashboard) | Task history, progress trends |
| Growth | Launch | Public showcase, early adopters |

The two systems share minimal but meaningful data via Supabase API or export/import in JSON.  
They are **loosely coupled** — each is fully functional alone, yet stronger together.

---

## 4. Product Philosophy — Low-AI, High-Tactility

| Principle | Explanation |
|------------|--------------|
| **Human-first UX** | Every feature must enhance creative focus, not replace it. |
| **AI on Demand** | AI is toggle-based, opt-in, and usage is credit-based. |
| **Local-first** | Core data is stored locally (IndexedDB cache + Supabase sync). |
| **Cost Transparency** | No hidden AI API usage; every AI call must be visible and optional. |

---

## 5. Monetization Strategy

| Tier | Monthly Fee | Description |
|------|--------------|-------------|
| **Free (Low-AI)** | $0 | All modules, up to 5 projects, 6-month retention, AI off by default |
| **Pro** | ~$9 | Unlimited projects, permanent data retention, weekly AI summary, API export |
| **Team** | ~$29 | 5 members, real-time collaboration, team analytics (AI enhanced) |

AI-related costs will be credit-based (e.g., 20 AI calls = $5).  
This prevents unnecessary cloud inference and aligns value with usage.

---

## 6. Expansion Roadmap

1. **Hub Integration (2026–2027)**  
   Unified dashboard linking PM Tool + Launch, enabling community and learning features.  
   - “Build Log Feed” (public activity)  
   - “Knowledge Hub” (shared frameworks and lessons)  
   - “AI Assist Plug-ins” (user-contributed local AI recipes)

2. **Collaboration Phase (2027–2028)**  
   Lightweight team functions via Supabase Realtime and Role-based Access Control.

3. **Marketplace Phase (2028–)**  
   Developers can publish their own templates, dashboards, and extensions.

---

## 7. Review Focus for AI Experts

When reviewing this document, please evaluate from the following lenses:

1. Is the **two-SaaS structure (PM Tool + Launch)** logically and technically sound?  
2. Does the **Low-AI, Human-first** philosophy create a sustainable UX?  
3. Are the **data integration points** between PM Tool and Launch minimal yet effective?  
4. Does the **monetization structure** ensure scalability and user retention?  
5. What are potential bottlenecks for **future Hub integration**?

---

**Author:** Kousuke Nakamura (EXITON Project)  
**Reviewers:** Claude / Gemini / ChatGPT / Google AI Studio  
**Repository:** [github.com/davidson00000/exiton-docs](https://github.com/davidson00000/exiton-docs)

---

> This document is part of the EXITON initiative to create a sustainable,  
> human-centered ecosystem for independent makers in the AI era.
