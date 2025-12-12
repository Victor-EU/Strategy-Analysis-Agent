# Strategy Analysis Agent — Technical Architecture

## Executive Summary

A multi-agent system built on Google ADK (Agent Development Kit) that analyzes companies using classical strategy economics frameworks. The system orchestrates 14 specialized agents through an 8-phase execution pipeline (Phase 0-6, with Phase 0.5 for Deep Research), beginning with intelligent context gathering, followed by autonomous deep web research, parallel analysis, synthesis, report generation, and presentation output.

**Tech Stack:**
- Framework: Google ADK (Python)
- LLM: Gemini 3 Pro (`gemini-3-pro-preview`) — 1M context window
- Deep Research: Gemini Deep Research Agent (`deep-research-pro-preview-12-2025`) — Autonomous web research
- Image Generation: Nano Banana Pro (`gemini-3-pro-image-preview`)
- PDF Processing: Docling (extraction) + PyMuPDF (generation)
- Database: SQLite (session caching + logging)
- Frontend: React/Next.js with light blue design system
- Language: Python 3.11+

**Key Design Principles:**
1. **Context Quality Bounds Analysis Quality** — Phase 0 (Context Gathering) systematically extracts insider knowledge that cannot be found in public sources
2. **Deep Research Before Analysis** — Phase 0.5 conducts autonomous, parallel web research with cited sources, pre-populating a research cache that all analysis agents consume
3. **Hybrid Research Strategy** — Predictive deep research upfront + quick search fallback for cache misses

---

## Part 1: System Architecture Overview

```
┌───────────────────────────────────────────────────────────────────────────────────────┐
│                            STRATEGY ANALYSIS AGENT                                     │
├───────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                        │
│  ┌─────────────────────────────────────────────────────────────────────────────────┐  │
│  │                          ORCHESTRATION LAYER                                     │  │
│  │  ┌───────────────────────────────────────────────────────────────────────────┐  │  │
│  │  │               Strategy Orchestrator (Root Agent)                           │  │  │
│  │  │   - Manages 8-phase execution (Phase 0, 0.5, 1-6)                          │  │  │
│  │  │   - Gathers user context before analysis (Phase 0)                         │  │  │
│  │  │   - Triggers deep research with predictive task selection (Phase 0.5)      │  │  │
│  │  │   - Enforces dependency chain                                              │  │  │
│  │  │   - Runs coherence checks                                                  │  │  │
│  │  │   - Generates executive summary and full report                            │  │  │
│  │  │   - Triggers presentation generation                                       │  │  │
│  │  └───────────────────────────────────────────────────────────────────────────┘  │  │
│  └─────────────────────────────────────────────────────────────────────────────────┘  │
│                                        │                                              │
│  ┌─────────────────────────────────────┼───────────────────────────────────────────┐  │
│  │                           AGENT LAYER                                           │  │
│  │                                                                                  │  │
│  │  Phase 0 (Sequential) — CONTEXT GATHERING ★ CRITICAL                            │  │
│  │  ┌────────────────────────────────────────────────────────────────────────┐    │  │
│  │  │                    Context Gathering Agent                              │    │  │
│  │  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────┐     │    │  │
│  │  │  │ Foundation      │→ │ Document        │→ │ Guided Interview    │     │    │  │
│  │  │  │ Setup           │  │ Processor       │  │ Agent               │     │    │  │
│  │  │  │ (basic info)    │  │ (if docs)       │  │ (conversational)    │     │    │  │
│  │  │  └─────────────────┘  └─────────────────┘  └─────────────────────┘     │    │  │
│  │  │                              ↓                                          │    │  │
│  │  │                    Writes: user_context:{domain}                        │    │  │
│  │  └────────────────────────────────────────────────────────────────────────┘    │  │
│  │                                                                                  │  │
│  │  Phase 0.5 (Parallel) — DEEP RESEARCH ★ AUTONOMOUS WEB RESEARCH                 │  │
│  │  ┌────────────────────────────────────────────────────────────────────────┐    │  │
│  │  │                    Deep Research Agent                                  │    │  │
│  │  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────┐     │    │  │
│  │  │  │ Research Task   │  │ Parallel Deep   │  │ Research Cache      │     │    │  │
│  │  │  │ Planner         │→ │ Research Exec   │→ │ Builder             │     │    │  │
│  │  │  │ (from Phase 0)  │  │ (Interactions)  │  │ (with citations)    │     │    │  │
│  │  │  └─────────────────┘  └─────────────────┘  └─────────────────────┘     │    │  │
│  │  │                              ↓                                          │    │  │
│  │  │                    Writes: research_cache:{domain}                      │    │  │
│  │  └────────────────────────────────────────────────────────────────────────┘    │  │
│  │                                                                                  │  │
│  │  Phase 1 (Parallel) — EXTERNAL CONTEXT                                          │  │
│  │  ┌──────────────┐  ┌──────────────────────────────┐                             │  │
│  │  │ Macro Economy│  │     Market Structure         │                             │  │
│  │  │    Agent     │  │  ┌──────────┐ ┌──────────┐   │                             │  │
│  │  │  (uses cache)│  │  │Complem-  │ │Substit-  │   │                             │  │
│  │  │              │  │  │ents      │ │utes      │   │                             │  │
│  │  └──────────────┘  │  │(uses     │ │(uses     │   │                             │  │
│  │                    │  │ cache)   │ │ cache)   │   │                             │  │
│  │                    │  └──────────┘ └──────────┘   │                             │  │
│  │                    └──────────────────────────────┘                             │  │
│  │                                                                                  │  │
│  │  Phase 2 (Parallel) — FIRM ANALYSIS                                             │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                           │  │
│  │  │ Comparative  │  │ Value Chain  │  │    JTBD      │                           │  │
│  │  │  Advantage   │  │    Agent     │  │    Agent     │                           │  │
│  │  │ (uses cache) │  │ (uses cache) │  │ (uses cache) │                           │  │
│  │  └──────────────┘  └──────────────┘  └──────────────┘                           │  │
│  │                                                                                  │  │
│  │  Phase 3 (Sequential) — STRATEGY                                                │  │
│  │  ┌──────────────┐                                                               │  │
│  │  │ Competitive  │                                                               │  │
│  │  │  Strategy    │                                                               │  │
│  │  │ (uses cache) │                                                               │  │
│  │  └──────────────┘                                                               │  │
│  │                                                                                  │  │
│  │  Phase 4 (Sequential) — SYNTHESIS                                               │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                           │  │
│  │  │    SWOT      │  │  Coherence   │  │  Narrative   │                           │  │
│  │  │  Synthesis   │  │   Checker    │  │  Synthesis   │                           │  │
│  │  └──────────────┘  └──────────────┘  └──────────────┘                           │  │
│  │                                                                                  │  │
│  │  Phase 5 (Sequential) — REPORT GENERATION                                       │  │
│  │  ┌────────────────────────────────────────────────────────────────────────┐    │  │
│  │  │  ┌─────────────────────────┐    ┌─────────────────────────────────┐    │    │  │
│  │  │  │  Executive Summary      │ →  │      Full Report Generator      │    │    │  │
│  │  │  │     Generator           │    │   (Comprehensive Analysis Doc)  │    │    │  │
│  │  │  └─────────────────────────┘    └─────────────────────────────────┘    │    │  │
│  │  └────────────────────────────────────────────────────────────────────────┘    │  │
│  │                                                                                  │  │
│  │  Phase 6 (Sequential) — PRESENTATION                                            │  │
│  │  ┌────────────────────────────────────────────────────────────────────────┐    │  │
│  │  │  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐     │    │  │
│  │  │  │ Slide Structure │ →  │ Visual Generator│ →  │  PDF Assembler  │     │    │  │
│  │  │  │ (Gemini 3 Pro)  │    │(Nano Banana Pro)│    │   (PyMuPDF)     │     │    │  │
│  │  │  └─────────────────┘    └─────────────────┘    └─────────────────┘     │    │  │
│  │  └────────────────────────────────────────────────────────────────────────┘    │  │
│  └─────────────────────────────────────────────────────────────────────────────────┘  │
│                                        │                                              │
│  ┌─────────────────────────────────────┼───────────────────────────────────────────┐  │
│  │                          SHARED SERVICES                                        │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │  │
│  │  │ Market Intel │  │  Document    │  │  Research    │  │     PDF      │         │  │
│  │  │    Tool      │  │  Processor   │  │   Cache      │  │  Generator   │         │  │
│  │  │ (Fallback)   │  │  (Docling)   │  │  Manager     │  │  (PyMuPDF)   │         │  │
│  │  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘         │  │
│  └─────────────────────────────────────────────────────────────────────────────────┘  │
│                                        │                                              │
│  ┌─────────────────────────────────────┼───────────────────────────────────────────┐  │
│  │                        INFRASTRUCTURE LAYER                                     │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                           │  │
│  │  │   Session    │  │   Logging    │  │   Shared     │                           │  │
│  │  │   Service    │  │   Service    │  │   Memory     │                           │  │
│  │  │  (SQLite)    │  │  (SQLite)    │  │   (State)    │                           │  │
│  │  └──────────────┘  └──────────────┘  └──────────────┘                           │  │
│  └─────────────────────────────────────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────────────────────────────────────┘
```

---

## Part 2: Model Configuration

| Purpose | Model | Model ID | Context | Notes |
|---------|-------|----------|---------|-------|
| Analysis Agents | Gemini 3 Pro | `gemini-3-pro-preview` | 1M tokens | Extended thinking enabled |
| **Deep Research** | **Deep Research Agent** | `deep-research-pro-preview-12-2025` | N/A | **Autonomous multi-step web research via Interactions API** |
| Report Generation | Gemini 3 Pro | `gemini-3-pro-preview` | 1M tokens | Executive summary + full report |
| Presentation Structure | Gemini 3 Pro | `gemini-3-pro-preview` | 1M tokens | Slide planning |
| Visual Generation | Nano Banana Pro | `gemini-3-pro-image-preview` | N/A | 2K/4K infographics |

**Analysis Model Settings:** temperature=0.7, max_output_tokens=8192, thinking_level=HIGH

**Deep Research Settings:** background=true, stream=true, thinking_summaries=auto, max_research_time=60min, typical_completion=20min

**Image Generation Settings:** response_modalities=[IMAGE, TEXT], image_size=2K, aspect_ratio=16:9

---

## Part 3: Agent Specifications

### 3.0 Context Gathering Agent (Phase 0) — CRITICAL

**Purpose:** Extract insider knowledge that cannot be found in public sources, establishing the foundation for high-confidence analysis

**Type:** `SequentialAgent` with three sub-components

**Why This Phase Exists:**

| Agent | What It Needs | Why Public Data Isn't Enough |
|-------|--------------|------------------------------|
| Complements | Partner relationship health, dependency risks | Public announcements are PR; internal view knows which partners are critical |
| Substitutes | Real churn reasons, switching destinations | Churn data is proprietary; reviews show complaints, not switching behavior |
| Comparative Advantage | Leadership's view of the moat, internal capabilities | Self-perception differs from market perception; need both |
| Value Chain | Actual margin by activity, make-vs-buy decisions | Segment financials don't show activity-level economics |
| JTBD | Customer research, hiring/firing triggers | Internal research is gold; public reviews are biased sample |
| Competitive Strategy | Strategic intent, target customer, deliberate positioning | Public positioning is marketing; internal strategy may differ |

**Architecture:**
```
Context Gathering Agent
├── Foundation Setup Agent
│   └── Captures: company, user role, strategic question, time horizon
├── Document Processor Agent (conditional)
│   └── Extracts: insights from uploaded strategy docs, research, financials
└── Guided Interview Agent
    └── Conducts: adaptive Q&A to fill knowledge gaps
    └── Adapts: questions based on user role and documents provided
```

---

#### 3.0.1 Foundation Setup Agent

**Purpose:** Capture basic analysis parameters before deeper context gathering

**Output Fields:**
| Field | Type | Description |
|-------|------|-------------|
| company_name | string | Company being analyzed |
| company_ticker | string (optional) | Stock ticker if public |
| industry | string | Primary industry |
| sub_industry | string | Specific sub-sector |
| geography | string | Primary operating geography |
| company_size | enum | large_cap, mid_cap, small_cap, private |
| company_stage | enum | early, growth, mature, turnaround, declining |
| user_role | enum | investor, employee, competitor, student, advisor, executive |
| strategic_question | string | The core question to answer |
| time_horizon | string | Analysis time horizon (e.g., "3-5 years") |
| output_preferences | object | generate_slides, generate_report flags |

**Output Key:** `foundation_context`

---

#### 3.0.2 Document Processor Agent

**Purpose:** Extract strategic insights from user-provided documents

**Triggers:** Only runs if user uploads documents

**Supported Document Types:**
| Type | What We Extract |
|------|-----------------|
| Strategy presentations | Strategic priorities, competitive positioning, target segments |
| Customer research | JTBD insights, satisfaction drivers, churn reasons |
| Financial reports | Segment margins, cost structure, investment priorities |
| Board decks | Strategic concerns, risk assessments, key initiatives |
| Competitive analyses | Competitor positioning, threat assessment |
| Partnership agreements | Complement relationships, dependencies |

**Output Fields:**
| Field | Type | Description |
|-------|------|-------------|
| documents_processed | list[DocumentSummary] | Summary of each document |
| extracted_strategy | StrategyInsights | Strategic positioning from docs |
| extracted_customers | CustomerInsights | Customer/JTBD insights from docs |
| extracted_financials | FinancialInsights | Margin and cost insights |
| extracted_competitive | CompetitiveInsights | Competitive landscape from docs |
| extracted_partnerships | PartnershipInsights | Complement/partner insights |
| confidence_boost | float | How much docs improve confidence (0-0.4) |
| remaining_gaps | list[string] | What's still missing for each domain |

**Output Key:** `document_insights`

---

#### 3.0.3 Guided Interview Agent

**Purpose:** Conduct adaptive conversational Q&A to extract insider knowledge not available publicly or in documents

**Behavior:**
- Adapts questions based on user role (investor asks different questions than employee)
- Skips questions already answered by documents
- Prioritizes high-value questions first
- Allows "I don't know" / "Skip" responses
- Explains why each question matters for the analysis

**Interview Domains and Questions:**

##### Domain: Market Structure (Complements)
| Question | Why It Matters | Fallback if Unknown |
|----------|---------------|---------------------|
| Who are your most critical partners/complements? | Identifies ecosystem dependencies | Use public partnership announcements |
| How would you rate each relationship? (strengthening/stable/weakening) | Reveals relationship trajectory | Assume stable, lower confidence |
| What happens if [key partner] vertically integrates or leaves? | Uncovers existential risks | Flag as unknown risk |
| Which partnerships are strategic vs. transactional? | Distinguishes critical from replaceable | Treat all as moderate importance |

##### Domain: Market Structure (Substitutes)
| Question | Why It Matters | Fallback if Unknown |
|----------|---------------|---------------------|
| What do customers actually switch to when they leave? | Reveals real competitive threats | Use competitor analysis |
| What triggers customers to start looking for alternatives? | Identifies vulnerability moments | Infer from public reviews |
| What's your estimated customer switching cost (time, money, effort)? | Quantifies defensibility | Use industry benchmarks |
| Are there emerging substitutes that concern leadership? | Surfaces non-obvious threats | Research emerging players |

##### Domain: Comparative Advantage
| Question | Why It Matters | Fallback if Unknown |
|----------|---------------|---------------------|
| What does leadership believe is the company's moat? | Internal perception of advantage | Infer from public positioning |
| What can you do that competitors genuinely cannot replicate? | Identifies durable advantages | Use patent/capability analysis |
| What advantages are eroding or under threat? | Reveals vulnerability | Assume all advantages contested |
| What would it take for a well-funded competitor to match you? | Tests durability | Use industry analysis |

##### Domain: Value Chain
| Question | Why It Matters | Fallback if Unknown |
|----------|---------------|---------------------|
| Which activities generate the highest margins? | Identifies profit engines | Use segment financials |
| What's outsourced vs. kept in-house, and why? | Reveals make-vs-buy strategy | Assume industry standard |
| Where are the biggest cost centers? | Identifies efficiency opportunities | Use public cost disclosures |
| Which activities are strategically important vs. commodity? | Distinguishes core from context | Infer from investments |

##### Domain: Customer Jobs (JTBD)
| Question | Why It Matters | Fallback if Unknown |
|----------|---------------|---------------------|
| Why do customers "hire" your product? (functional, emotional, social jobs) | Core value proposition | Use review sentiment analysis |
| What are customers' top frustrations with current solutions (yours or competitors)? | Identifies underserved jobs | Mine public complaints |
| What would make customers pay significantly more? | Reveals willingness-to-pay drivers | Assume price sensitivity |
| What causes customers to "fire" your product? | Churn drivers | Use public churn benchmarks |
| Which customer segments are underserved by current offerings? | Growth opportunities | Use market research |

##### Domain: Competitive Strategy
| Question | Why It Matters | Fallback if Unknown |
|----------|---------------|---------------------|
| What is leadership's explicit strategic positioning? (cost leader, differentiator, focused) | Deliberate strategy | Infer from pricing/marketing |
| Who is the target customer, specifically? | Segment focus | Use customer demographic data |
| What strategic bets is the company making for the next 3-5 years? | Future direction | Use earnings call guidance |
| Where is there internal disagreement about strategy? | Reveals tensions | Assume coherent strategy |
| What would you do differently if you had unlimited resources? | Reveals constrained ambitions | Assume current path optimal |

**Output Fields:**
| Field | Type | Description |
|-------|------|-------------|
| interview_completed | bool | Whether interview was completed |
| questions_asked | int | Number of questions asked |
| questions_answered | int | Number answered (vs. skipped) |
| user_context:complements | ComplementsContext | Partner/ecosystem insights |
| user_context:substitutes | SubstitutesContext | Switching/alternatives insights |
| user_context:advantages | AdvantagesContext | Moat/capability insights |
| user_context:value_chain | ValueChainContext | Margin/cost insights |
| user_context:jtbd | JTBDContext | Customer job insights |
| user_context:strategy | StrategyContext | Positioning/intent insights |
| context_completeness | object | Completeness score per domain (0-1) |
| analysis_mode | enum | document_enriched, guided_context, public_only |

**Context Output Schema (per domain):**

| Field | Type | Description |
|-------|------|-------------|
| raw_responses | list[QAPair] | Question-answer pairs |
| synthesized_insights | list[string] | Key insights extracted |
| confidence_level | float | 0-1 based on response quality |
| gaps_remaining | list[string] | What we still don't know |
| source | enum | user_response, document, inferred, unknown |

**Output Key:** `user_context:{domain}` for each domain

---

#### 3.0.4 Analysis Mode Determination

Based on context gathered, the system determines the analysis mode:

| Mode | Criteria | Confidence Multiplier |
|------|----------|----------------------|
| **document_enriched** | User provided documents + answered key questions | 1.0x (full confidence) |
| **guided_context** | No documents, but user answered most questions | 0.8x |
| **public_only** | User skipped most questions, no documents | 0.5x (explicit confidence penalty) |

**Output Key:** `analysis:mode`

---

### 3.0.5 Deep Research Agent (Phase 0.5) — AUTONOMOUS WEB RESEARCH

**Purpose:** Conduct comprehensive, parallel web research based on Phase 0 context to pre-populate a research cache with cited reports that all downstream analysis agents consume.

**Type:** `ParallelAgent` managing multiple Deep Research interactions via the Interactions API

**Why This Phase Exists:**

| Without Phase 0.5 | With Phase 0.5 |
|-------------------|----------------|
| Each agent does ad-hoc web searches | Research happens once, shared by all |
| Shallow, quick lookups | Deep, multi-step research with citations |
| Inconsistent source quality | Verified, cited sources |
| Redundant searches across agents | Cached results, no duplication |
| Variable research depth | Comprehensive coverage per domain |

**Architecture:**
```
Deep Research Agent (Phase 0.5)
├── Research Task Planner
│   └── Analyzes: Phase 0 context (company, industry, strategic question)
│   └── Selects: Which research tasks to run based on analysis needs
│   └── Prioritizes: Core tasks always run; conditional tasks based on context
├── Parallel Deep Research Executor
│   └── Launches: Multiple Deep Research interactions in parallel
│   └── Uses: Gemini Interactions API with background=true
│   └── Streams: Progress updates for UX feedback
│   └── Handles: Reconnection for long-running research
└── Research Cache Builder
    └── Collects: All research results with citations
    └── Indexes: By domain for fast agent lookup
    └── Stores: research_cache:{domain} in shared state
```

---

#### 3.0.5.1 Research Task Templates

**Core Tasks (Always Run):**

| Task Key | Query Template | Target Domain |
|----------|---------------|---------------|
| `company_overview` | "Comprehensive business analysis of {company}: business model, revenue streams, key products/services, recent strategic moves, financial performance, and market position in {industry}" | All agents |
| `competitor_landscape` | "Detailed competitive landscape analysis for {company} in {industry}: top 5 competitors, their strengths/weaknesses, market share, strategic positioning, and competitive dynamics" | Substitutes, Competitive Strategy |
| `industry_trends` | "Industry trends and market dynamics in {industry} 2024-2025: growth drivers, disruption risks, regulatory changes, technology shifts, and emerging opportunities" | Macro Economy, all agents |

**Conditional Tasks (Based on Strategic Question):**

| Task Key | Query Template | Condition | Target Domain |
|----------|---------------|-----------|---------------|
| `value_chain_economics` | "Value chain and supply chain analysis for {industry}: key activities, cost structures, margin distribution, make-vs-buy trends, and vertical integration patterns" | Strategic question mentions costs, margins, supply chain | Value Chain |
| `customer_segments` | "Customer segmentation and jobs-to-be-done analysis for {company} in {industry}: customer types, purchase drivers, switching costs, and unmet needs" | Strategic question mentions customers, growth, segments | JTBD |
| `partnership_ecosystem` | "Partner ecosystem and complement analysis for {company}: key partnerships, integration dependencies, platform relationships, and ecosystem health" | Strategic question mentions partnerships, ecosystem, platform | Complements |
| `macro_regulatory` | "Macroeconomic and regulatory factors affecting {industry} in {geography}: economic indicators, policy changes, trade dynamics, and geopolitical risks" | Strategic question mentions regulation, policy, macro | Macro Economy |
| `emerging_threats` | "Emerging competitive threats and disruptors in {industry}: new entrants, technology disruption, business model innovation, and substitute threats" | Strategic question mentions disruption, threats, future | Substitutes |

---

#### 3.0.5.2 Deep Research Execution

**Invocation via Interactions API:**

| Parameter | Value | Notes |
|-----------|-------|-------|
| `agent` | `deep-research-pro-preview-12-2025` | Deep Research agent |
| `background` | `true` | Required for async execution |
| `stream` | `true` | Enable progress tracking |
| `agent_config.thinking_summaries` | `auto` | Get research progress updates |

**Parallel Execution Pattern:**

1. Launch all selected research tasks simultaneously
2. Each task runs as independent Deep Research interaction
3. Collect results as they complete (not waiting for all)
4. Build research cache incrementally
5. Timeout: 30 minutes max wait (typical: 15-20 min for all tasks)

**Error Handling:**

| Scenario | Behavior |
|----------|----------|
| Single task fails | Continue with other tasks; flag gap in cache |
| All tasks fail | Fall back to quick search mode; warn user |
| Timeout exceeded | Use partial results; flag incomplete research |
| Rate limit hit | Retry with exponential backoff |

---

#### 3.0.5.3 Research Cache Structure

**Cache Entry Schema:**

| Field | Type | Description |
|-------|------|-------------|
| task_key | string | Research task identifier |
| query | string | The research query sent |
| report | string | Full research report with markdown |
| citations | list[Citation] | Source URLs with titles |
| research_time_seconds | int | Time taken for research |
| completed_at | datetime | Completion timestamp |
| word_count | int | Report length |
| confidence_indicators | object | Quality signals from Deep Research |

**Citation Schema:**

| Field | Type | Description |
|-------|------|-------------|
| url | string | Source URL |
| title | string | Page/article title |
| domain | string | Source domain |
| accessed_at | datetime | When accessed |
| relevance | float | Relevance score (0-1) |

**Output Keys:**

| Key | Description |
|-----|-------------|
| `research_cache` | Full cache object with all results |
| `research_cache:company_overview` | Company overview research |
| `research_cache:competitor_landscape` | Competitor analysis |
| `research_cache:industry_trends` | Industry trends research |
| `research_cache:{task_key}` | Domain-specific research |
| `research_metadata` | Timing, task count, cache stats |
| `research_gaps` | Tasks that failed or timed out |

---

#### 3.0.5.4 Agent Cache Consumption

Each analysis agent queries the research cache for relevant information:

**Lookup Pattern:**

| Agent | Primary Cache Keys | Fallback |
|-------|-------------------|----------|
| Macro Economy | `industry_trends`, `macro_regulatory` | Quick web search |
| Complements | `partnership_ecosystem`, `company_overview` | Quick web search |
| Substitutes | `competitor_landscape`, `emerging_threats` | Quick web search |
| Comparative Advantage | `company_overview`, `competitor_landscape` | Quick web search |
| Value Chain | `value_chain_economics`, `company_overview` | Quick web search |
| JTBD | `customer_segments`, `company_overview` | Quick web search |
| Competitive Strategy | `competitor_landscape`, `industry_trends` | Quick web search |

**Cache Miss Handling:**

When an agent needs information not in the cache:
1. Check cache with keyword relevance search
2. If relevant cache exists → synthesize from cached reports
3. If no relevant cache → use Market Intel Tool for quick search
4. Flag the gap in `research_gaps` for potential future improvement

---

#### 3.0.5.5 User Experience During Deep Research

**Progress Display:**

| Stage | User Feedback |
|-------|--------------|
| Planning | "Analyzing your strategic question to plan research..." |
| Launching | "Starting {n} parallel research tasks..." |
| In Progress | Progress bars per task with estimated time |
| Completing | "Research complete. {n} comprehensive reports ready." |

**Time Expectations:**

| Research Scope | Typical Duration | Tasks |
|----------------|------------------|-------|
| Minimal (public company) | 10-15 min | 3 core tasks |
| Standard | 15-20 min | 4-5 tasks |
| Comprehensive | 20-30 min | 6-7 tasks |

---

#### 3.0.5.6 Analysis Mode with Deep Research

**Updated Mode Determination:**

| Mode | Criteria | Confidence Multiplier |
|------|----------|----------------------|
| **comprehensive** | Documents + Interview + Deep Research | 1.2x (highest confidence) |
| **deep_research_enriched** | Interview + Deep Research (no docs) | 1.0x |
| **document_enriched** | Documents + Interview (no deep research) | 0.9x |
| **guided_context** | Interview only (no docs, no deep research) | 0.7x |
| **quick_public** | Quick search only (skipped everything) | 0.5x |

**Output Key:** `analysis:mode`

---

### 3.1 Strategy Orchestrator (Root Agent)

**Type:** `SequentialAgent` containing `ParallelAgent` sub-agents for concurrent phases

**Responsibilities:**
- Coordinate the 8-phase execution pipeline (Phase 0, 0.5, 1-6)
- Gather user context before analysis begins (Phase 0)
- Trigger deep research with predictive task selection (Phase 0.5)
- Manage research cache for downstream agent consumption
- Inject foundation context into shared state
- Run cross-agent coherence checks
- Synthesize final narrative
- Generate executive summary and comprehensive report
- Trigger presentation generation

**Sub-Agents:**
1. `phase0_context_agent` — Context Gathering (sequential)
2. `phase0_5_research_agent` — Deep Research (parallel tasks)
3. `phase1_parallel_agent` — Macro + Market Structure (parallel)
4. `phase2_parallel_agent` — Comparative Advantage + Value Chain + JTBD (parallel)
5. `phase3_competitive_agent` — Competitive Strategy (sequential)
6. `phase4_synthesis_agent` — SWOT + Coherence + Narrative (sequential)
7. `phase5_report_agent` — Executive Summary + Full Report (sequential)
8. `phase6_presentation_agent` — Slides + Visuals + PDF (sequential)

**Coherence Checks (Phase 4):**

| Condition A | Condition B | Flag |
|-------------|-------------|------|
| Macro: contraction | Strategy: growth investment | `timing_risk` |
| Comp. Adv: technical depth | Strategy: cost leadership | `strategy_mismatch` |
| Substitutes: format shift | Strategic moves: no mention | `blind_spot` |
| Complements: high dependency | No partnership strategy | `execution_gap` |
| JTBD: wants simplicity | Differentiation: feature richness | `value_mismatch` |

---

### 3.2 Framework Agents (Specialists)

Each specialist agent is an `LlmAgent` with:
- Specific `instruction` defining its analytical framework
- **Primary data source: Research Cache** (populated by Phase 0.5 Deep Research)
- Fallback: Market Intel Tool for quick searches on cache misses
- Access to user context from Phase 0 interview
- Defined `output_key` for storing results in session state
- Structured output via `output_schema`
- Model: `gemini-3-pro-preview`

**Data Priority for All Agents:**
1. **Research Cache** — Deep research reports with citations (highest quality)
2. **User Context** — Insider knowledge from Phase 0 interview
3. **Document Insights** — Extracted from user-provided documents
4. **Quick Search** — Fallback for cache misses (lowest priority)

#### 3.2.1 Macro Economy Agent

**Purpose:** Analyze business cycle, interest rates, FX, sector spending patterns

**Research Cache Keys:** `industry_trends`, `macro_regulatory`

**Output Fields:**
| Field | Type | Description |
|-------|------|-------------|
| cycle_phase | enum | expansion, peak, contraction, trough |
| rate_environment | string | rising, stable, falling |
| consumer_spending_outlook | string | Assessment of consumer trends |
| fx_impact | string (optional) | Foreign exchange implications |
| sector_trends | list[string] | Key sector movements |
| strategic_implications | list[string] | Business implications |
| confidence | ConfidenceMetadata | Confidence scoring |
| sources | list[Citation] | Citations from research cache |

**Tools:** `research_cache.get()`, `search_economic_data` (fallback)

**Output Key:** `macro_economy_analysis`

---

#### 3.2.2 Market Structure Agent

**Type:** `ParallelAgent` orchestrating Complements + Substitutes

**Purpose:** Analyze ecosystem (complements) and competitive alternatives (substitutes)

##### Complements Analysis Agent

**Research Cache Keys:** `partnership_ecosystem`, `company_overview`

**Output Fields:**
| Field | Type | Description |
|-------|------|-------------|
| key_complements | list[ComplementItem] | Category, players, integration depth |
| ecosystem_health | enum | strong, moderate, weak, fragmented |
| vertical_integration_risks | list[string] | Integration risk factors |
| strategic_implications | list[string] | Business implications |
| sources | list[Citation] | Citations from research cache |

**ComplementItem:** category, key_players, integration_depth (critical/deep/moderate/shallow), relationship_trend (strengthening/stable/weakening/adversarial)

**Tools:** `research_cache.get()`, `search_partnerships` (fallback)

**Output Key:** `complements_analysis`

##### Substitutes Analysis Agent

**Research Cache Keys:** `competitor_landscape`, `emerging_threats`

**Output Fields:**
| Field | Type | Description |
|-------|------|-------------|
| direct_substitutes | list[SubstituteItem] | Direct competitive alternatives |
| indirect_substitutes | list[SubstituteItem] | Indirect alternatives |
| switching_costs | enum | high, moderate, low |
| substitution_triggers | list[string] | What causes switching |
| defensibility_zones | list[string] | Protected areas |
| sources | list[Citation] | Citations from research cache |

**SubstituteItem:** name, type (direct/indirect/potential), threat_level (high/medium/low), switching_barrier

**Tools:** `research_cache.get()`, `search_competitors` (fallback)

**Output Key:** `substitutes_analysis`

---

#### 3.2.3 Comparative Advantage Agent

**Purpose:** Identify what the firm does that others cannot easily replicate

**Research Cache Keys:** `company_overview`, `competitor_landscape`

**Output Fields:**
| Field | Type | Description |
|-------|------|-------------|
| primary_advantages | list[AdvantageItem] | Core competitive advantages |
| areas_of_parity | list[string] | Where firm matches competitors |
| areas_of_disadvantage | list[string] | Where firm lags |
| durability_assessment | string | Overall durability analysis |
| sources | list[Citation] | Citations from research cache |

**AdvantageItem:** advantage, source (scale/network_effects/ip/brand/capabilities/data/relationships), durability (durable/eroding/temporary), replicability (very_hard/hard/moderate/easy), monetization_status

**Tools:** `research_cache.get()`, `search_patents` (fallback)

**Output Key:** `comparative_advantage_analysis`

**Dependencies:** Reads `market_structure_analysis`, `user_context:advantages`

---

#### 3.2.4 Value Chain Agent

**Purpose:** Analyze activities and margin drivers

**Research Cache Keys:** `value_chain_economics`, `company_overview`

**Output Fields:**
| Field | Type | Description |
|-------|------|-------------|
| primary_activities | list[ActivityItem] | Core value-creating activities |
| support_activities | list[ActivityItem] | Supporting activities |
| margin_drivers | list[string] | What drives margins |
| cost_centers | list[string] | Major cost areas |
| vertical_integration_level | enum | high, moderate, low |
| sources | list[Citation] | Citations from research cache |

**ActivityItem:** activity, margin_contribution (high/medium/low/negative), strategic_importance (critical/important/supporting), outsourced (bool)

**Tools:** `research_cache.get()`, `analyze_cost_structure` (fallback)

**Output Key:** `value_chain_analysis`

**Dependencies:** Reads `user_context:value_chain`

---

#### 3.2.5 JTBD Agent (Jobs To Be Done)

**Purpose:** Understand customer needs and hiring criteria

**Research Cache Keys:** `customer_segments`, `company_overview`

**Output Fields:**
| Field | Type | Description |
|-------|------|-------------|
| jobs | list[JobItem] | Customer jobs being done |
| underserved_jobs | list[string] | Unmet needs |
| overserved_jobs | list[string] | Over-delivered areas |
| hiring_criteria | list[string] | Why customers choose |
| firing_triggers | list[string] | Why customers leave |
| opportunities | list[string] | Market opportunities |
| sources | list[Citation] | Citations from research cache |

**JobItem:** job, job_type (functional/emotional/social), importance (critical/important/nice_to_have), current_satisfaction (high/medium/low/unmet)

**Tools:** `research_cache.get()`, `search_customer_reviews` (fallback)

**Output Key:** `jtbd_analysis`

**Dependencies:** Reads `user_context:jtbd`

---

#### 3.2.6 Competitive Strategy Agent

**Purpose:** Determine cost leadership vs. differentiation positioning

**Research Cache Keys:** `competitor_landscape`, `industry_trends`

**Output Fields:**
| Field | Type | Description |
|-------|------|-------------|
| current_position | enum | cost_leadership, differentiation, focus_cost, focus_diff, stuck_in_middle |
| target_segment | string | Primary customer segment |
| scope | enum | broad, narrow |
| coherence_assessment | object | score (0-1), gaps, tensions |
| recommended_position | string (optional) | Suggested positioning |
| strategic_moves | list[StrategicMove] | Recommended actions |
| sources | list[Citation] | Citations from research cache |

**StrategicMove:** move, rationale, risk (high/medium/low), dependencies

**Tools:** `research_cache.get()`, `analyze_pricing` (fallback)

**Output Key:** `competitive_strategy_analysis`

**Dependencies:** Reads `comparative_advantage_analysis`, `value_chain_analysis`, `user_context:strategy`

---

#### 3.2.7 SWOT Synthesis Agent

**Purpose:** Integrate all findings into SWOT framework with cross-references

**Output Fields:**
| Field | Type | Description |
|-------|------|-------------|
| strengths | list[SWOTItem] | Internal positives |
| weaknesses | list[SWOTItem] | Internal negatives |
| opportunities | list[SWOTItem] | External positives |
| threats | list[SWOTItem] | External negatives |
| key_tensions | list[TensionItem] | Strategic tensions |
| strategic_priorities | list[string] | Prioritized actions |

**SWOTItem:** item, source_agent, importance (critical/high/medium/low), cross_references

**TensionItem:** tension, between (tuple of agent names), severity (high/medium/low), resolution_suggestion

**Tools:** None (reads from other agents' outputs)

**Output Key:** `swot_synthesis`

**Dependencies:** Reads ALL other agent outputs

---

### 3.3 Report Agent (Phase 5)

**Purpose:** Generate executive summary and comprehensive strategic analysis report

**Type:** `SequentialAgent` with two sub-components

**Architecture:**
```
Report Agent
├── Executive Summary Generator (LlmAgent)
│   └── Produces concise C-suite ready summary with verdict
└── Full Report Generator (LlmAgent)
    └── Produces comprehensive analysis document with all sections
```

#### 3.3.1 Executive Summary Generator

**Purpose:** Synthesize all analysis into a 2-minute-readable executive summary that directly answers the strategic question

**Output Fields:**
| Field | Type | Description |
|-------|------|-------------|
| summary_headline | string | Single sentence capturing verdict |
| summary_body | string | 2-3 paragraph synthesis |
| key_takeaways | list[KeyTakeaway] | 3-5 prioritized findings |
| strategic_verdict | StrategicVerdict | Clear recommendation |
| critical_risks | list[RiskItem] | Top 3 risks |
| immediate_actions | list[ActionItem] | Next 90 days actions |
| confidence | ConfidenceMetadata | Confidence scoring |

**StrategicVerdict:** verdict (strongly_favorable / favorable_with_caveats / neutral_depends_on_execution / unfavorable_with_opportunities / strongly_unfavorable), reasoning, key_conditions, time_horizon_caveat

**KeyTakeaway:** takeaway, source_agent, confidence_level, supporting_evidence

**RiskItem:** risk, severity, likelihood, mitigation_available, source_analysis

**ActionItem:** action, rationale, owner_type (executive/strategy_team/operations/board), dependencies, success_metric

**Tools:** None (pure synthesis)

**Output Key:** `executive_summary`

**Dependencies:** Reads ALL prior agent outputs

---

#### 3.3.2 Full Report Generator

**Purpose:** Generate comprehensive strategic analysis report for detailed reading

**Output Fields:**
| Field | Type | Description |
|-------|------|-------------|
| report_title | string | Report title |
| report_subtitle | string | Report subtitle |
| generated_for | string | User role context |
| analysis_date | string | Generation date |
| table_of_contents | list[TOCEntry] | Document structure |
| introduction | ReportSection | Opening section |
| external_environment | ExternalEnvironmentSection | Macro + market analysis |
| firm_analysis | FirmAnalysisSection | Comp advantage + value chain + JTBD |
| strategic_position | StrategicPositionSection | Competitive strategy |
| synthesis_and_tensions | SynthesisTensionsSection | SWOT + coherence |
| recommendations | RecommendationsSection | Action recommendations |
| appendices | list[AppendixSection] | Supporting material |
| source_attribution | SourceAttribution | Traceability |
| confidence | ConfidenceMetadata | Confidence scoring |

**ReportSection:** section_id, title, content (list of ContentBlock), key_insight, confidence_note

**ContentBlock:** block_type (paragraph/bullet_list/numbered_list/quote/callout/data_table/framework_reference/cross_reference), content, source_agent, emphasis

**RecommendationsSection:** primary_recommendation, supporting_recommendations, implementation_sequence, success_metrics, monitoring_triggers

**Tools:** None (pure synthesis)

**Output Key:** `full_report`

**Dependencies:** Reads ALL prior outputs + `executive_summary`

---

### 3.4 Presentation Agent (Phase 6)

**Purpose:** Generate professional presentation deck leveraging executive summary and full report

**Type:** `SequentialAgent` with three sub-components

**Architecture:**
```
Presentation Agent
├── Slide Structure Generator (Gemini 3 Pro)
│   └── Plans slide sequence, content, visual descriptions
│   └── Uses executive_summary.key_takeaways for highlight slides
│   └── Uses full_report.recommendations for recommendations slides
├── Visual Generator (Nano Banana Pro)
│   └── Creates infographics, charts, diagrams
│   └── Uses full_report.swot_matrix_data for SWOT visual
└── PDF Assembler (PyMuPDF)
    └── Composes final presentation PDF
    └── Generates report PDF from full_report structure
```

#### 3.4.1 Slide Structure Generator

**Purpose:** Plan presentation structure and content for each slide

**Output Fields:**
| Field | Type | Description |
|-------|------|-------------|
| title | string | Presentation title |
| subtitle | string | Presentation subtitle |
| total_slides | int | Number of slides |
| slides | list[SlideSpec] | Slide specifications |
| color_scheme | ColorScheme | Brand colors |

**SlideSpec:** slide_number, slide_type (title/executive_summary/section_header/key_findings/framework/data_visualization/swot_matrix/recommendations/appendix), title, content, visual_description, speaker_notes, source_agents

**SlideContent:** headline, body_points, data_points, quote, footer

**Slide Sequence:**
1. Title slide
2. Executive summary (1-2 slides)
3. Macro context
4. Market structure (complements & substitutes)
5. Firm analysis (comparative advantage, value chain)
6. Customer insights (JTBD)
7. Competitive positioning
8. SWOT matrix
9. Key tensions and coherence issues
10. Strategic recommendations
11. Appendix (optional)

**Output Key:** `presentation_structure`

---

#### 3.4.2 Visual Generator

**Model:** Nano Banana Pro (`gemini-3-pro-image-preview`)

**Purpose:** Generate infographics, charts, and diagrams for each slide

**Capabilities:**
- Slide visuals from natural language descriptions
- Framework diagrams (Porter's, SWOT, Value Chain, competitive position)
- Data visualizations with brand colors
- 16:9 aspect ratio, 2K resolution

**Output Key:** `slide_visuals` (list of image paths)

---

#### 3.4.3 PDF Assembler

**Purpose:** Assemble final PDF presentation with consistent styling

**Design System:**
| Element | Value |
|---------|-------|
| Primary Blue | #2265C0 |
| Light Blue | #E3F2FD |
| Accent Blue | #4285F4 |
| Dark Text | #212121 |
| Light Text | #757575 |
| Title Size | 44pt |
| Heading Size | 28pt |
| Body Size | 14pt |
| Page Size | A4 Landscape (841.89 × 595.28 pt) |
| Margins | 50pt standard, 80pt content |

**Output Keys:** `presentation_pdf`, `report_pdf`

---

## Part 4: Shared Memory Architecture

### 4.1 Foundation Context

All agents share access to foundation context stored in session state:

| Key Pattern | Example | Description |
|-------------|---------|-------------|
| `user:{key}` | `user:role`, `user:strategic_question` | User-level data |
| `company:{key}` | `company:name`, `company:ticker` | Company being analyzed |
| `analysis:{key}` | `analysis:mode`, `analysis:documents` | Session metadata |
| `output:{key}` | `output:generate_slides` | Output configuration |
| `user_context:{domain}` | `user_context:complements` | Domain-specific insider knowledge (Phase 0) |
| `research_cache:{task_key}` | `research_cache:competitor_landscape` | Deep Research results with citations (Phase 0.5) |
| `document_insights` | `document_insights` | Extracted document insights |

**User Roles:** investor, employee, competitor, student, advisor, executive

**Analysis Modes (Updated with Deep Research):**
| Mode | Description | Confidence Impact |
|------|-------------|-------------------|
| comprehensive | Documents + Interview + Deep Research | 1.2x (highest) |
| deep_research_enriched | Interview + Deep Research (no docs) | 1.0x |
| document_enriched | Documents + Interview (no deep research) | 0.9x |
| guided_context | Interview only | 0.7x |
| quick_public | Quick search only (skipped everything) | 0.5x |

### 4.1.1 User Context Domains (from Phase 0)

Each domain captures insider knowledge that agents consume:

| Domain Key | Consuming Agent | Key Insights Captured |
|------------|-----------------|----------------------|
| `user_context:complements` | Complements Agent | Partner criticality, relationship health, integration risks |
| `user_context:substitutes` | Substitutes Agent | Churn destinations, switching triggers, emerging threats |
| `user_context:advantages` | Comparative Advantage Agent | Perceived moat, capability assessment, durability concerns |
| `user_context:value_chain` | Value Chain Agent | Activity margins, outsourcing rationale, cost centers |
| `user_context:jtbd` | JTBD Agent | Hiring/firing triggers, underserved segments, satisfaction drivers |
| `user_context:strategy` | Competitive Strategy Agent | Strategic intent, target customer, positioning choices |

**Each domain contains:**
- `raw_responses` — Original Q&A pairs
- `synthesized_insights` — Key takeaways
- `confidence_level` — 0-1 based on response quality
- `gaps_remaining` — What we still don't know
- `source` — user_response, document, inferred, or unknown

### 4.1.2 Research Cache (from Phase 0.5)

Deep Research results are stored in the research cache for consumption by all analysis agents:

| Cache Key | Consuming Agents | Content |
|-----------|------------------|---------|
| `research_cache:company_overview` | All agents | Business model, revenue, products, market position |
| `research_cache:competitor_landscape` | Substitutes, Competitive Strategy, Comparative Advantage | Top competitors, market share, strategic positioning |
| `research_cache:industry_trends` | Macro Economy, all agents | Growth drivers, disruption risks, technology shifts |
| `research_cache:value_chain_economics` | Value Chain | Cost structures, margin distribution, make-vs-buy |
| `research_cache:customer_segments` | JTBD | Customer types, purchase drivers, unmet needs |
| `research_cache:partnership_ecosystem` | Complements | Key partnerships, integration dependencies |
| `research_cache:macro_regulatory` | Macro Economy | Policy changes, regulatory factors |
| `research_cache:emerging_threats` | Substitutes | New entrants, disruption, substitute threats |

**Each cache entry contains:**
- `task_key` — Research task identifier
- `query` — Original research query
- `report` — Full research report with markdown formatting
- `citations` — List of source URLs with titles and relevance scores
- `research_time_seconds` — Time taken for research
- `completed_at` — Completion timestamp
- `word_count` — Report length for quality assessment

**Cache Metadata Keys:**
| Key | Description |
|-----|-------------|
| `research_cache` | Full cache object with all results |
| `research_metadata` | Timing, task count, completion stats |
| `research_gaps` | Tasks that failed or timed out |

### 4.2 State Key Naming Convention

| Pattern | Description |
|---------|-------------|
| `{agent_name}_analysis` | Agent's structured output |
| `{agent_name}_raw` | Raw LLM response (debugging) |
| `research_cache` | Full Deep Research cache object |
| `research_cache:{task_key}` | Individual research result with citations |
| `research_metadata` | Research timing and completion stats |
| `research_gaps` | Research tasks that failed or timed out |
| `executive_summary` | Executive summary from Report Agent |
| `full_report` | Comprehensive report from Report Agent |
| `presentation_structure` | Slide structure from Gemini 3 Pro |
| `slide_visuals` | Generated image paths |
| `presentation_pdf` | Final slides PDF path |
| `report_pdf` | Detailed report PDF path |
| `temp:{key}` | Intermediate calculations |
| `cache:search:{hash}` | Cached quick search results (fallback) |
| `cache:document:{id}` | Cached document extractions |
| `cache:visual:{hash}` | Cached generated visuals |

### 4.3 State Flow Between Agents

```
Phase 0: User Input → [Context Gathering Agent] ★ CRITICAL FOUNDATION
              │
              ├── Foundation Setup Agent
              │       ↓
              │   foundation_context (company, user role, strategic question)
              │
              ├── Document Processor Agent (if docs provided)
              │       ↓
              │   document_insights (extracted strategy, customer, financial insights)
              │
              └── Guided Interview Agent
                      ↓
                  user_context:complements  → consumed by Complements Agent
                  user_context:substitutes  → consumed by Substitutes Agent
                  user_context:advantages   → consumed by Comparative Advantage Agent
                  user_context:value_chain  → consumed by Value Chain Agent
                  user_context:jtbd         → consumed by JTBD Agent
                  user_context:strategy     → consumed by Competitive Strategy Agent
                  analysis:mode             → confidence multiplier for all agents
              ↓
Phase 0.5: Foundation Context → [Deep Research Agent] ★ AUTONOMOUS WEB RESEARCH
              │
              ├── Research Task Planner (selects tasks based on strategic question)
              │       ↓
              │   research_tasks (3-7 parallel research queries)
              │
              ├── Parallel Deep Research Executor (Interactions API)
              │       ↓
              │   research_cache:company_overview      → consumed by ALL agents
              │   research_cache:competitor_landscape  → consumed by Substitutes, Comp Strategy
              │   research_cache:industry_trends       → consumed by Macro, all agents
              │   research_cache:value_chain_economics → consumed by Value Chain
              │   research_cache:customer_segments     → consumed by JTBD
              │   research_cache:partnership_ecosystem → consumed by Complements
              │   research_cache:macro_regulatory      → consumed by Macro Economy
              │   research_cache:emerging_threats      → consumed by Substitutes
              │
              └── Research Cache Builder
                      ↓
                  research_cache (full cache with all results)
                  research_metadata (timing, completion stats)
                  research_gaps (failed/timed out tasks)
              ↓
Phase 1: User Context + Research Cache → [Macro Agent, Market Structure Agent]
              │
              ├── Macro Economy Agent
              │   ├── reads: research_cache:industry_trends, research_cache:macro_regulatory
              │   └── fallback: quick search via Market Intel Tool
              │       ↓ macro_economy_analysis (with citations)
              │
              └── Market Structure Agent
                  ├── Complements Agent
                  │   ├── reads: user_context:complements, research_cache:partnership_ecosystem
                  │   └── fallback: quick search
                  │       ↓ complements_analysis (with citations)
                  └── Substitutes Agent
                      ├── reads: user_context:substitutes, research_cache:competitor_landscape
                      └── fallback: quick search
                          ↓ substitutes_analysis (with citations)
              ↓
Phase 2: + Phase 1 outputs + Research Cache → [Comp. Advantage, Value Chain, JTBD]
              │
              ├── Comparative Advantage Agent
              │   ├── reads: user_context:advantages, research_cache:company_overview
              │   └── fallback: quick search
              │       ↓ comparative_advantage_analysis (with citations)
              │
              ├── Value Chain Agent
              │   ├── reads: user_context:value_chain, research_cache:value_chain_economics
              │   └── fallback: quick search
              │       ↓ value_chain_analysis (with citations)
              │
              └── JTBD Agent
                  ├── reads: user_context:jtbd, research_cache:customer_segments
                  └── fallback: quick search
                      ↓ jtbd_analysis (with citations)
              ↓
Phase 3: + Phase 2 outputs + Research Cache → [Competitive Strategy]
              │
              └── Competitive Strategy Agent
                  ├── reads: user_context:strategy, research_cache:competitor_landscape
                  └── fallback: quick search
                      ↓ competitive_strategy_analysis (with citations)
              ↓
Phase 4: + All outputs → [SWOT Synthesis, Coherence Check, Narrative]
              ↓
         swot_synthesis, coherence_flags, final_narrative
              ↓
Phase 5: + All outputs → [Report Agent]
              ↓
         executive_summary, full_report (includes source citations from research cache)
              ↓
Phase 6: + Report outputs → [Presentation Agent]
              ↓
         presentation_structure, slide_visuals, presentation_pdf, report_pdf
```

**Key Insight:** Each analysis agent receives THREE data sources in priority order:
1. **Research Cache** — Deep research reports with verified citations (highest quality)
2. **User Context** — Insider knowledge from Phase 0 interview
3. **Quick Search** — Fallback for cache misses (Market Intel Tool)

This combination enables dramatically higher-confidence analysis than any single source approach.

---

## Part 5: Tool Architecture

### 5.1 Research Cache Manager (NEW)

**Purpose:** Manage Deep Research cache populated by Phase 0.5, providing fast access to cited research reports for all analysis agents

**Methods:**
| Method | Description |
|--------|-------------|
| `get(task_key)` | Retrieve specific research result by task key |
| `get_relevant(keywords)` | Find research results matching keywords |
| `get_citations(task_key)` | Get citations for a specific research result |
| `has_coverage(domain)` | Check if domain has cached research |
| `get_gaps()` | Return list of research gaps (failed/incomplete tasks) |

**Usage Pattern:**
```python
# Primary: Check research cache first
research = cache.get("competitor_landscape")
if research:
    citations = research.citations
    report = research.report
else:
    # Fallback: Use quick search
    result = market_intel.search_web(query)
```

**Output:** ResearchResult with report, citations, timing, and quality metadata

---

### 5.2 Market Intel Tool (Fallback)

**Purpose:** Quick web search fallback when Research Cache has gaps. Used for cache misses during analysis.

**Methods:**
| Method | Description |
|--------|-------------|
| `search_web` | Search with source_type (news/general/financial/academic) and recency filters |
| `get_company_financials` | Retrieve financial metrics by ticker |
| `search_sec_filings` | Search SEC filings (10-K, 10-Q, 8-K, DEF14A) |
| `search_earnings_transcripts` | Retrieve earnings call transcripts |

**Note:** This tool is the FALLBACK when Research Cache doesn't have relevant data. Deep Research results are preferred for their depth and citations.

**Caching:** Results cached by query hash with configurable TTL (default 24h)

### 5.3 Document Processor Tool (Docling)

**Purpose:** PDF and document processing with structured extraction

**Methods:**
| Method | Description |
|--------|-------------|
| `process_document` | Extract sections, tables, and metadata |

**Output:** document_id, title, sections[], tables[], metadata

### 5.4 Knowledge Base Tool

**Purpose:** Unified access to all knowledge sources with priority ranking

**Source Priority:** research_cache → documents → user_context → quick_search

### 5.5 Visual Generator Tool

**Purpose:** Generate presentation visuals using Nano Banana Pro

**Methods:**
| Method | Description |
|--------|-------------|
| `generate_slide_visual` | Generate visual from description |
| `generate_framework_diagram` | Generate Porter's, SWOT, Value Chain diagrams |

### 5.6 PDF Generator Tool

**Purpose:** Generate PDF presentations and reports using PyMuPDF

**Methods:**
| Method | Description |
|--------|-------------|
| `create_presentation` | Assemble slide deck PDF |
| `generate_report_pdf` | Generate detailed text report PDF |

---

## Part 6: Session & Caching Architecture

### 6.1 Database Schema

**ADK-Managed Tables:** sessions, user_state, app_state, raw_events

**Custom Tables:**

| Table | Purpose |
|-------|---------|
| `research_cache` | Deep Research results with citations (Phase 0.5) |
| `research_interactions` | Deep Research interaction IDs for resumption |
| `search_cache` | Quick search results with TTL (fallback) |
| `document_cache` | Cached document extractions |
| `visual_cache` | Cached generated visuals |
| `analysis_sessions` | Analysis metadata and status |
| `generated_outputs` | Generated PDFs and files |
| `report_outputs` | Executive summary and full report content |
| `logs` | Comprehensive logging |
| `agent_executions` | Agent execution metrics |
| `coherence_checks` | Coherence check results |

### 6.2 Cache Strategy

| Cache Type | Key Pattern | TTL |
|------------|-------------|-----|
| Deep Research results | task_key | Session-bound (persistent for session) |
| Quick search results | query hash | 24 hours (configurable) |
| Documents | file hash | Persistent |
| Visuals | prompt hash | Persistent |

---

## Part 7: Logging & Observability

### 7.1 Log Categories

| Category | Description |
|----------|-------------|
| `context.foundation` | Foundation setup (company, role, question) |
| `context.document` | Document processing and extraction |
| `context.interview` | Guided interview Q&A |
| `context.mode` | Analysis mode determination |
| `research.plan` | Deep Research task planning |
| `research.start/progress/complete/error` | Deep Research execution lifecycle |
| `research.cache_hit/cache_miss` | Research cache access patterns |
| `research.citation` | Citation tracking |
| `agent.start/complete/error` | Agent lifecycle |
| `llm.request/response/error/tokens` | LLM interactions |
| `tool.call/result/error/cache_hit` | Tool executions |
| `state.read/write` | State changes |
| `phase.start/complete` | Phase transitions |
| `coherence.check/flag` | Coherence checking |
| `document.upload/process/extract` | Document handling |
| `report.executive_summary/full/section/verdict/confidence` | Report generation |
| `presentation.structure` | Slide planning |
| `visual.generate` | Image generation |
| `pdf.generate` | PDF assembly |
| `performance.latency/memory` | Performance metrics |

### 7.2 Log Levels

DEBUG, INFO, WARNING, ERROR, CRITICAL

---

## Part 8: Frontend Design System

### 8.1 Color Palette

| Color | Hex | Usage |
|-------|-----|-------|
| Primary 50 | #E3F2FD | Backgrounds |
| Primary 500 | #2196F3 | Primary actions |
| Primary 800 | #1565C0 | Headers |
| Gray 50 | #FAFAFA | Page backgrounds |
| Gray 900 | #212121 | Headings |
| Success | #4CAF50 | Positive states |
| Warning | #FF9800 | Caution states |
| Error | #F44336 | Error states |

### 8.2 Typography

| Element | Size | Weight |
|---------|------|--------|
| Headings | 24-36px | Bold (700) |
| Body | 14-16px | Regular (400) |
| Captions | 12px | Regular (400) |
| Font Family | Inter, system fonts | — |

### 8.3 Component Patterns

- **Cards:** White background, subtle border, 12px radius, hover elevation
- **Confidence Indicators:** 5-dot scale with color coding (green/yellow/red)
- **Progress Tracker:** Phase icons with completed/in-progress/pending states
- **Chat Interface:** Agent messages (blue-50 bg), User messages (gray-100 bg)

---

## Part 9: Project Structure

```
strategy_analysis_agent/
├── src/strategy_agent/
│   ├── agents/
│   │   ├── orchestrator.py
│   │   ├── context_gathering/     # Phase 0
│   │   │   ├── __init__.py
│   │   │   ├── foundation.py      # Foundation Setup Agent
│   │   │   ├── document.py        # Document Processor Agent
│   │   │   ├── interview.py       # Guided Interview Agent
│   │   │   └── questions.py       # Domain-specific question banks
│   │   ├── deep_research/         # Phase 0.5 (NEW)
│   │   │   ├── __init__.py
│   │   │   ├── planner.py         # Research Task Planner
│   │   │   ├── executor.py        # Parallel Deep Research Executor
│   │   │   ├── cache_builder.py   # Research Cache Builder
│   │   │   └── task_templates.py  # Research task templates
│   │   ├── macro_economy.py       # Phase 1
│   │   ├── market_structure.py    # Phase 1
│   │   ├── complements.py
│   │   ├── substitutes.py
│   │   ├── comparative_advantage.py # Phase 2
│   │   ├── value_chain.py
│   │   ├── jtbd.py
│   │   ├── competitive_strategy.py # Phase 3
│   │   ├── swot.py                # Phase 4
│   │   ├── report.py              # Phase 5
│   │   └── presentation.py        # Phase 6
│   ├── tools/
│   │   ├── research_cache.py      # Research Cache Manager (NEW)
│   │   ├── market_intel.py        # Quick search fallback
│   │   ├── document_processor.py
│   │   ├── knowledge_base.py
│   │   ├── visual_generator.py
│   │   ├── pdf_generator.py
│   │   └── analysis_tools.py
│   ├── schemas/
│   │   ├── foundation.py
│   │   ├── context.py             # User context schemas
│   │   ├── research.py            # Research cache schemas (NEW)
│   │   ├── outputs.py
│   │   ├── report.py
│   │   ├── presentation.py
│   │   └── common.py
│   ├── services/
│   │   ├── session.py
│   │   ├── cache.py
│   │   ├── deep_research.py       # Deep Research service (NEW)
│   │   ├── logging.py
│   │   └── database.py
│   ├── coherence/
│   │   ├── checker.py
│   │   └── rules.py
│   └── main.py
├── frontend/
│   └── [Next.js application]
├── data/prompts/
│   ├── foundation_setup.md        # Phase 0
│   ├── guided_interview.md        # Phase 0
│   ├── research_task_templates.md # Phase 0.5 (NEW)
│   ├── macro_economy.md
│   ├── complements.md
│   ├── executive_summary.md
│   ├── full_report.md
│   ├── presentation_structure.md
│   └── ...
├── tests/
│   ├── test_context_gathering/    # Phase 0 tests
│   │   ├── test_foundation.py
│   │   ├── test_document.py
│   │   └── test_interview.py
│   ├── test_deep_research/        # Phase 0.5 tests (NEW)
│   │   ├── test_planner.py
│   │   ├── test_executor.py
│   │   └── test_cache.py
│   └── ...
└── scripts/
```

---

## Part 10: Configuration

### 10.1 Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `GOOGLE_API_KEY` | Yes | Gemini API key |
| `STRATEGY_AGENT_DB_PATH` | No | Database path (default: ./data/strategy_agent.db) |
| `STRATEGY_AGENT_LOG_LEVEL` | No | Log level (default: INFO) |
| `STRATEGY_AGENT_CACHE_TTL_HOURS` | No | Cache TTL (default: 24) |
| `TAVILY_API_KEY` | No | Search API key |

### 10.2 Application Settings

| Setting | Default | Description |
|---------|---------|-------------|
| model | gemini-3-pro-preview | Analysis model |
| image_model | gemini-3-pro-image-preview | Image generation model |
| temperature | 0.7 | LLM temperature |
| max_tokens | 8192 | Max output tokens |
| image_resolution | 2K | Image resolution |
| image_aspect_ratio | 16:9 | Image aspect ratio |
| agent_timeout_seconds | 300 | Agent timeout |
| generate_slides | true | Generate presentation |
| generate_report | true | Generate report |

---

## Part 11: Execution Flow

### 11.1 Analysis Lifecycle

**Phase 0 — Context Gathering (Sequential) ★ CRITICAL FOUNDATION:**
1. **Foundation Setup:**
   - Capture company info, user role, strategic question, time horizon
   - → `foundation_context`
2. **Document Processing (if documents provided):**
   - Extract strategic insights from uploaded docs
   - → `document_insights`
3. **Guided Interview:**
   - Conduct adaptive Q&A based on user role and document gaps
   - Ask domain-specific questions (complements, substitutes, advantages, value chain, JTBD, strategy)
   - → `user_context:{domain}` for each domain
4. **Mode Determination:**
   - Calculate context completeness per domain
   - Determine if Deep Research should run
   - → `analysis:mode`

**Phase 0.5 — Deep Research (Parallel) ★ AUTONOMOUS WEB RESEARCH:**
1. **Research Task Planning:**
   - Analyze strategic question and foundation context
   - Select 3-7 research tasks (core + conditional)
   - → `research_tasks`
2. **Parallel Deep Research Execution:**
   - Launch all research tasks via Interactions API
   - Stream progress updates to user
   - Timeout: 30 min max (typical: 15-20 min)
   - → `research_cache:{task_key}` for each completed task
3. **Research Cache Building:**
   - Collect all research results with citations
   - Index by domain for fast agent lookup
   - → `research_cache`, `research_metadata`, `research_gaps`

**Phase 1 — External Context (Parallel):**
- Macro Economy Agent
  - reads: `research_cache:industry_trends`, `research_cache:macro_regulatory`
  - fallback: quick search → `macro_economy_analysis`
- Market Structure Agent:
  - Complements Agent (reads: `user_context:complements`, `research_cache:partnership_ecosystem`) → `complements_analysis`
  - Substitutes Agent (reads: `user_context:substitutes`, `research_cache:competitor_landscape`) → `substitutes_analysis`

**Phase 2 — Firm Analysis (Parallel):**
- Comparative Advantage Agent (reads: `user_context:advantages`, `research_cache:company_overview`) → `comparative_advantage_analysis`
- Value Chain Agent (reads: `user_context:value_chain`, `research_cache:value_chain_economics`) → `value_chain_analysis`
- JTBD Agent (reads: `user_context:jtbd`, `research_cache:customer_segments`) → `jtbd_analysis`

**Phase 3 — Strategy (Sequential):**
- Competitive Strategy Agent (reads: `user_context:strategy`, `research_cache:competitor_landscape`) → `competitive_strategy_analysis`

**Phase 4 — Synthesis (Sequential):**
- SWOT Agent → `swot_synthesis`
- Coherence Checker → `coherence_flags`
- Narrative Synthesis → `final_narrative`

**Phase 5 — Report Generation (Sequential):**
- Executive Summary Generator → `executive_summary`
- Full Report Generator → `full_report` (includes citations from research cache)

**Phase 6 — Presentation (Sequential):**
- Slide Structure Generator → `presentation_structure`
- Visual Generator → `slide_visuals`
- PDF Assembler → `presentation_pdf`, `report_pdf`

**Completion:**
- Store outputs, log metrics, return results with confidence scores and source citations

### 11.2 Context Gathering Flow Detail

```
User Starts Session
        ↓
┌───────────────────────────────────────────────────────────────┐
│ Foundation Setup                                              │
│ "What company? What's your role? What question to answer?"    │
└───────────────────────────────────────────────────────────────┘
        ↓
┌───────────────────────────────────────────────────────────────┐
│ Document Upload (Optional)                                    │
│ "Do you have strategy docs, customer research, financials?"   │
│ If yes → Extract insights, identify remaining gaps            │
└───────────────────────────────────────────────────────────────┘
        ↓
┌───────────────────────────────────────────────────────────────┐
│ Guided Interview (Adaptive)                                   │
│                                                               │
│ For each domain with gaps:                                    │
│   - Explain why question matters                              │
│   - Ask question                                              │
│   - Accept answer OR "I don't know" OR "Skip"                 │
│   - Adapt follow-ups based on response                        │
│                                                               │
│ Domains: Complements, Substitutes, Advantages,                │
│          Value Chain, JTBD, Strategy                          │
└───────────────────────────────────────────────────────────────┘
        ↓
┌───────────────────────────────────────────────────────────────┐
│ Mode Determination                                            │
│                                                               │
│ Calculate completeness score per domain (0-1)                 │
│ Set analysis:mode based on overall context quality            │
│ Inform user of confidence implications                        │
└───────────────────────────────────────────────────────────────┘
        ↓
┌───────────────────────────────────────────────────────────────┐
│ Deep Research (Phase 0.5) ★ AUTONOMOUS                        │
│                                                               │
│ Planning: Analyze strategic question → select research tasks  │
│                                                               │
│ Execution (Parallel):                                         │
│   ├── [████████████████░░░░] Company Overview (80%)          │
│   ├── [██████████████░░░░░░] Competitor Landscape (70%)      │
│   ├── [████████░░░░░░░░░░░░] Industry Trends (40%)           │
│   └── [██████░░░░░░░░░░░░░░] Value Chain Economics (30%)     │
│                                                               │
│ ⏱️  Typical duration: 15-20 minutes                           │
│                                                               │
│ Cache Building: Index all research by domain for fast lookup  │
└───────────────────────────────────────────────────────────────┘
        ↓
    Analysis Begins (Phase 1-6) — Uses Research Cache
```

### 11.3 How Agents Use Context

Each analysis agent follows this pattern:

1. **Query research cache:** `research_cache.get("{domain_key}")` for pre-researched reports
2. **Read user context:** `state.get("user_context:{domain}")` for insider knowledge
3. **Check document insights:** `state.get("document_insights")` if documents provided
4. **Fallback search (if cache miss):** Use Market Intel tool for quick search
5. **Synthesize:** Combine research cache + user context + document insights
6. **Include citations:** Preserve source URLs from research cache in output
7. **Score confidence:** Apply `analysis:mode` multiplier (1.2x for comprehensive, 0.5x for quick_public)
8. **Flag gaps:** Note where research cache had gaps or fallback was used

---

## Part 12: Testing Strategy

### 12.1 Test Categories

| Category | Focus |
|----------|-------|
| Unit Tests | Agent outputs, tool functions, schema validation, coherence rules |
| Integration Tests | Agent-to-agent state passing, caching, database persistence |
| End-to-End Tests | Full pipeline with mock/real data, complete report + presentation |
| Evaluation Tests | Output quality, coherence detection accuracy, verdict accuracy |
| Context Tests | Phase 0 interview flow, document extraction, mode determination |
| Research Tests | Phase 0.5 task planning, parallel execution, cache building |

### 12.2 Key Test Scenarios

**Phase 0 — Context Gathering:**
- Foundation setup captures all required fields
- Document processor extracts relevant insights per domain
- Interview adapts questions based on user role
- Interview skips questions answered by documents
- "I don't know" responses handled gracefully with fallbacks
- Mode determination correctly calculates confidence multiplier
- Context completeness scores are accurate per domain

**Phase 0.5 — Deep Research:**
- Research task planner selects correct tasks based on strategic question
- Core tasks (company_overview, competitor_landscape, industry_trends) always run
- Conditional tasks only run when strategic question matches condition
- Parallel execution launches all tasks simultaneously
- Streaming progress updates work correctly
- Research cache stores results with citations
- Cache miss handling triggers quick search fallback
- Timeout handling uses partial results gracefully
- Research gaps are correctly identified and logged

**Phase 1-6 — Analysis:**
- Agents correctly query research cache first
- Agents fall back to quick search on cache miss
- Agents correctly read user_context:{domain} for insider knowledge
- Confidence scores reflect analysis:mode multiplier (1.2x to 0.5x)
- Source citations from research cache are preserved in outputs
- Public-only mode explicitly flags lower confidence
- Executive summary answers the strategic question
- Strategic verdict is decisive (not hedged)
- All takeaways trace to source agents
- Critical risks incorporate coherence flags
- Report sections are complete and cross-referenced
- Report includes source citations from research cache
- Presentation uses report data accurately

### 12.3 Context Quality Scenarios

| Scenario | Expected Behavior |
|----------|------------------|
| Rich context (docs + full interview) | High confidence, specific insights |
| Partial context (some questions skipped) | Medium confidence, flag gaps |
| Minimal context (public only) | Low confidence, explicit warnings |
| Contradictory context (user vs. docs) | Flag tension, explain discrepancy |

---

## Part 13: Future Considerations

### 13.1 Deferred for MVP

- Real-time streaming updates
- Multi-user support
- Strategy Map integration
- Collaborative editing

### 13.2 Scaling Considerations

- PostgreSQL for production
- Redis for distributed caching
- Rate limiting for external APIs
- CDN for generated outputs

---

## Appendix A: Confidence Scoring Guidelines

| Score | Criteria |
|-------|----------|
| 1.0 | Direct user documents + verification |
| 0.8 | User verbal context + public confirmation |
| 0.6 | Public structured data (SEC, financials) |
| 0.4 | Public unstructured (news, articles) |
| 0.2 | Inference from limited data |

**ConfidenceMetadata:** overall (0-1), limiting_factors, high_confidence_claims, low_confidence_claims, would_improve_with

---

*Document Version: 5.0*
*Last Updated: 2025-12-12*

**Changelog:**
- **v5.0** (2025-12-12): Added Phase 0.5 Deep Research Agent — autonomous web research via Gemini Deep Research Agent (`deep-research-pro-preview-12-2025`). System now has 8 phases (0, 0.5, 1-6) and 14 agents. Key changes:
  - Added Deep Research Agent with Research Task Planner, Parallel Executor, and Cache Builder
  - Added Research Cache Manager to Shared Services (primary data source for all analysis agents)
  - All analysis agents now query research_cache first, with quick search fallback
  - Updated analysis modes: comprehensive (1.2x), deep_research_enriched (1.0x), document_enriched (0.9x), guided_context (0.7x), quick_public (0.5x)
  - Added research task templates (core: company_overview, competitor_landscape, industry_trends; conditional: value_chain_economics, customer_segments, etc.)
  - All agent outputs now include source citations from research cache
  - Added research_cache:{task_key} state keys and research_metadata
- **v4.0** (2025-12-11): Added Phase 0 Context Gathering Agent — the critical foundation that extracts insider knowledge before analysis begins. System now has 7 phases (0-6) and 13 agents. Added guided interview with domain-specific questions, document processing, analysis mode determination, and user_context:{domain} state keys consumed by each analysis agent.
- **v3.0** (2025-12-11): Added Phase 5 Report Agent, removed implementation code, converted to specification tables
- **v2.0** (2025-01-15): Added Gemini 3 Pro, Phase 5 Presentation Agent, Nano Banana Pro, PyMuPDF
- **v1.0** (Initial): Core 4-phase analysis pipeline with 9 agents
