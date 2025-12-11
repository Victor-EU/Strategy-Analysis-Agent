# Strategy Analysis Agent — Technical Architecture

## Executive Summary

A multi-agent system built on Google ADK (Agent Development Kit) that analyzes companies using classical strategy economics frameworks. The system orchestrates 13 specialized agents through a 7-phase execution pipeline (Phase 0-6), beginning with intelligent context gathering that extracts insider knowledge through guided conversation, followed by parallel analysis, synthesis, report generation, and presentation output.

**Tech Stack:**
- Framework: Google ADK (Python)
- LLM: Gemini 3 Pro (`gemini-3-pro-preview`) — 1M context window
- Image Generation: Nano Banana Pro (`gemini-3-pro-image-preview`)
- PDF Processing: Docling (extraction) + PyMuPDF (generation)
- Database: SQLite (session caching + logging)
- Frontend: React/Next.js with light blue design system
- Language: Python 3.11+

**Key Design Principle:** Analysis quality is bounded by context quality. Phase 0 (Context Gathering) systematically extracts insider knowledge that cannot be found in public sources, enabling dramatically higher-confidence analysis.

---

## Part 1: System Architecture Overview

```
┌──────────────────────────────────────────────────────────────────────────────────┐
│                           STRATEGY ANALYSIS AGENT                                 │
├──────────────────────────────────────────────────────────────────────────────────┤
│                                                                                   │
│  ┌────────────────────────────────────────────────────────────────────────────┐  │
│  │                         ORCHESTRATION LAYER                                 │  │
│  │  ┌──────────────────────────────────────────────────────────────────────┐  │  │
│  │  │               Strategy Orchestrator (Root Agent)                      │  │  │
│  │  │   - Manages 7-phase execution (Sequential + Parallel)                 │  │  │
│  │  │   - Gathers user context before analysis (Phase 0)                    │  │  │
│  │  │   - Enforces dependency chain                                         │  │  │
│  │  │   - Runs coherence checks                                             │  │  │
│  │  │   - Generates executive summary and full report                       │  │  │
│  │  │   - Triggers presentation generation                                  │  │  │
│  │  └──────────────────────────────────────────────────────────────────────┘  │  │
│  └────────────────────────────────────────────────────────────────────────────┘  │
│                                       │                                          │
│  ┌────────────────────────────────────┼────────────────────────────────────────┐ │
│  │                          AGENT LAYER                                        │ │
│  │                                                                              │ │
│  │  Phase 0 (Sequential) — CONTEXT GATHERING ★ CRITICAL                        │ │
│  │  ┌─────────────────────────────────────────────────────────────────────┐   │ │
│  │  │                    Context Gathering Agent                           │   │ │
│  │  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────┐  │   │ │
│  │  │  │ Foundation      │→ │ Document        │→ │ Guided Interview    │  │   │ │
│  │  │  │ Setup           │  │ Processor       │  │ Agent               │  │   │ │
│  │  │  │ (basic info)    │  │ (if docs)       │  │ (conversational)    │  │   │ │
│  │  │  └─────────────────┘  └─────────────────┘  └─────────────────────┘  │   │ │
│  │  │                              ↓                                       │   │ │
│  │  │                    Writes: user_context:{domain}                     │   │ │
│  │  └─────────────────────────────────────────────────────────────────────┘   │ │
│  │                                                                             │ │
│  │  Phase 1 (Parallel) — EXTERNAL CONTEXT                                     │ │
│  │  ┌──────────────┐  ┌─────────────────────────────┐                         │ │
│  │  │ Macro Economy│  │     Market Structure        │                         │ │
│  │  │    Agent     │  │  ┌──────────┐ ┌──────────┐  │                         │ │
│  │  │              │  │  │Complem-  │ │Substit-  │  │                         │ │
│  │  │              │  │  │ents      │ │utes      │  │                         │ │
│  │  └──────────────┘  │  └──────────┘ └──────────┘  │                         │ │
│  │                    └─────────────────────────────┘                         │ │
│  │                                                                             │ │
│  │  Phase 2 (Parallel) — FIRM ANALYSIS                                        │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                      │ │
│  │  │ Comparative  │  │ Value Chain  │  │    JTBD      │                      │ │
│  │  │  Advantage   │  │    Agent     │  │    Agent     │                      │ │
│  │  └──────────────┘  └──────────────┘  └──────────────┘                      │ │
│  │                                                                             │ │
│  │  Phase 3 (Sequential) — STRATEGY                                           │ │
│  │  ┌──────────────┐                                                          │ │
│  │  │ Competitive  │                                                          │ │
│  │  │  Strategy    │                                                          │ │
│  │  └──────────────┘                                                          │ │
│  │                                                                             │ │
│  │  Phase 4 (Sequential) — SYNTHESIS                                          │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                      │ │
│  │  │    SWOT      │  │  Coherence   │  │  Narrative   │                      │ │
│  │  │  Synthesis   │  │   Checker    │  │  Synthesis   │                      │ │
│  │  └──────────────┘  └──────────────┘  └──────────────┘                      │ │
│  │                                                                             │ │
│  │  Phase 5 (Sequential) — REPORT GENERATION                                  │ │
│  │  ┌─────────────────────────────────────────────────────────────────────┐   │ │
│  │  │  ┌─────────────────────────┐    ┌─────────────────────────────────┐ │   │ │
│  │  │  │  Executive Summary      │ →  │      Full Report Generator      │ │   │ │
│  │  │  │     Generator           │    │   (Comprehensive Analysis Doc)  │ │   │ │
│  │  │  └─────────────────────────┘    └─────────────────────────────────┘ │   │ │
│  │  └─────────────────────────────────────────────────────────────────────┘   │ │
│  │                                                                             │ │
│  │  Phase 6 (Sequential) — PRESENTATION                                       │ │
│  │  ┌─────────────────────────────────────────────────────────────────────┐   │ │
│  │  │  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐  │   │ │
│  │  │  │ Slide Structure │ →  │ Visual Generator│ →  │  PDF Assembler  │  │   │ │
│  │  │  │ (Gemini 3 Pro)  │    │(Nano Banana Pro)│    │   (PyMuPDF)     │  │   │ │
│  │  │  └─────────────────┘    └─────────────────┘    └─────────────────┘  │   │ │
│  │  └─────────────────────────────────────────────────────────────────────┘   │ │
│  └─────────────────────────────────────────────────────────────────────────────┘ │
│                                       │                                          │
│  ┌────────────────────────────────────┼────────────────────────────────────────┐ │
│  │                         SHARED SERVICES                                     │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │ │
│  │  │ Market Intel │  │  Document    │  │   Context    │  │     PDF      │    │ │
│  │  │    Tool      │  │  Processor   │  │   Manager    │  │  Generator   │    │ │
│  │  │ (Web Search) │  │  (Docling)   │  │              │  │  (PyMuPDF)   │    │ │
│  │  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘    │ │
│  └─────────────────────────────────────────────────────────────────────────────┘ │
│                                       │                                          │
│  ┌────────────────────────────────────┼────────────────────────────────────────┐ │
│  │                       INFRASTRUCTURE LAYER                                  │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                      │ │
│  │  │   Session    │  │   Logging    │  │   Shared     │                      │ │
│  │  │   Service    │  │   Service    │  │   Memory     │                      │ │
│  │  │  (SQLite)    │  │  (SQLite)    │  │   (State)    │                      │ │
│  │  └──────────────┘  └──────────────┘  └──────────────┘                      │ │
│  └─────────────────────────────────────────────────────────────────────────────┘ │
└───────────────────────────────────────────────────────────────────────────────────┘
```

---

## Part 2: Model Configuration

| Purpose | Model | Model ID | Context | Notes |
|---------|-------|----------|---------|-------|
| Analysis Agents | Gemini 3 Pro | `gemini-3-pro-preview` | 1M tokens | Extended thinking enabled |
| Report Generation | Gemini 3 Pro | `gemini-3-pro-preview` | 1M tokens | Executive summary + full report |
| Presentation Structure | Gemini 3 Pro | `gemini-3-pro-preview` | 1M tokens | Slide planning |
| Visual Generation | Nano Banana Pro | `gemini-3-pro-image-preview` | N/A | 2K/4K infographics |

**Analysis Model Settings:** temperature=0.7, max_output_tokens=8192, thinking_level=HIGH

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

### 3.1 Strategy Orchestrator (Root Agent)

**Type:** `SequentialAgent` containing `ParallelAgent` sub-agents for concurrent phases

**Responsibilities:**
- Coordinate the 7-phase execution pipeline (Phase 0-6)
- Gather user context before analysis begins (Phase 0)
- Inject foundation context into shared state
- Run cross-agent coherence checks
- Synthesize final narrative
- Generate executive summary and comprehensive report
- Trigger presentation generation

**Sub-Agents:**
1. `phase0_context_agent` — Context Gathering (sequential)
2. `phase1_parallel_agent` — Macro + Market Structure (parallel)
3. `phase2_parallel_agent` — Comparative Advantage + Value Chain + JTBD (parallel)
4. `phase3_competitive_agent` — Competitive Strategy (sequential)
5. `phase4_synthesis_agent` — SWOT + Coherence + Narrative (sequential)
6. `phase5_report_agent` — Executive Summary + Full Report (sequential)
7. `phase6_presentation_agent` — Slides + Visuals + PDF (sequential)

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
- Access to shared tools (Market Intel, Knowledge Base)
- Defined `output_key` for storing results in session state
- Structured output via `output_schema`
- Model: `gemini-3-pro-preview`

#### 3.2.1 Macro Economy Agent

**Purpose:** Analyze business cycle, interest rates, FX, sector spending patterns

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

**Tools:** `search_economic_data`, `get_fed_indicators`, `search_sector_trends`

**Output Key:** `macro_economy_analysis`

---

#### 3.2.2 Market Structure Agent

**Type:** `ParallelAgent` orchestrating Complements + Substitutes

**Purpose:** Analyze ecosystem (complements) and competitive alternatives (substitutes)

##### Complements Analysis Agent

**Output Fields:**
| Field | Type | Description |
|-------|------|-------------|
| key_complements | list[ComplementItem] | Category, players, integration depth |
| ecosystem_health | enum | strong, moderate, weak, fragmented |
| vertical_integration_risks | list[string] | Integration risk factors |
| strategic_implications | list[string] | Business implications |

**ComplementItem:** category, key_players, integration_depth (critical/deep/moderate/shallow), relationship_trend (strengthening/stable/weakening/adversarial)

**Tools:** `search_partnerships`, `search_ecosystem`, `search_integrations`

**Output Key:** `complements_analysis`

##### Substitutes Analysis Agent

**Output Fields:**
| Field | Type | Description |
|-------|------|-------------|
| direct_substitutes | list[SubstituteItem] | Direct competitive alternatives |
| indirect_substitutes | list[SubstituteItem] | Indirect alternatives |
| switching_costs | enum | high, moderate, low |
| substitution_triggers | list[string] | What causes switching |
| defensibility_zones | list[string] | Protected areas |

**SubstituteItem:** name, type (direct/indirect/potential), threat_level (high/medium/low), switching_barrier

**Tools:** `search_competitors`, `search_alternatives`, `analyze_trends`

**Output Key:** `substitutes_analysis`

---

#### 3.2.3 Comparative Advantage Agent

**Purpose:** Identify what the firm does that others cannot easily replicate

**Output Fields:**
| Field | Type | Description |
|-------|------|-------------|
| primary_advantages | list[AdvantageItem] | Core competitive advantages |
| areas_of_parity | list[string] | Where firm matches competitors |
| areas_of_disadvantage | list[string] | Where firm lags |
| durability_assessment | string | Overall durability analysis |

**AdvantageItem:** advantage, source (scale/network_effects/ip/brand/capabilities/data/relationships), durability (durable/eroding/temporary), replicability (very_hard/hard/moderate/easy), monetization_status

**Tools:** `search_patents`, `search_capabilities`, `analyze_historical_investment`

**Output Key:** `comparative_advantage_analysis`

**Dependencies:** Reads `market_structure_analysis`

---

#### 3.2.4 Value Chain Agent

**Purpose:** Analyze activities and margin drivers

**Output Fields:**
| Field | Type | Description |
|-------|------|-------------|
| primary_activities | list[ActivityItem] | Core value-creating activities |
| support_activities | list[ActivityItem] | Supporting activities |
| margin_drivers | list[string] | What drives margins |
| cost_centers | list[string] | Major cost areas |
| vertical_integration_level | enum | high, moderate, low |

**ActivityItem:** activity, margin_contribution (high/medium/low/negative), strategic_importance (critical/important/supporting), outsourced (bool)

**Tools:** `analyze_cost_structure`, `search_segment_data`, `analyze_financials`

**Output Key:** `value_chain_analysis`

---

#### 3.2.5 JTBD Agent (Jobs To Be Done)

**Purpose:** Understand customer needs and hiring criteria

**Output Fields:**
| Field | Type | Description |
|-------|------|-------------|
| jobs | list[JobItem] | Customer jobs being done |
| underserved_jobs | list[string] | Unmet needs |
| overserved_jobs | list[string] | Over-delivered areas |
| hiring_criteria | list[string] | Why customers choose |
| firing_triggers | list[string] | Why customers leave |
| opportunities | list[string] | Market opportunities |

**JobItem:** job, job_type (functional/emotional/social), importance (critical/important/nice_to_have), current_satisfaction (high/medium/low/unmet)

**Tools:** `search_customer_reviews`, `search_forums`, `analyze_surveys`

**Output Key:** `jtbd_analysis`

---

#### 3.2.6 Competitive Strategy Agent

**Purpose:** Determine cost leadership vs. differentiation positioning

**Output Fields:**
| Field | Type | Description |
|-------|------|-------------|
| current_position | enum | cost_leadership, differentiation, focus_cost, focus_diff, stuck_in_middle |
| target_segment | string | Primary customer segment |
| scope | enum | broad, narrow |
| coherence_assessment | object | score (0-1), gaps, tensions |
| recommended_position | string (optional) | Suggested positioning |
| strategic_moves | list[StrategicMove] | Recommended actions |

**StrategicMove:** move, rationale, risk (high/medium/low), dependencies

**Tools:** `analyze_pricing`, `analyze_brand_perception`, `search_positioning`

**Output Key:** `competitive_strategy_analysis`

**Dependencies:** Reads `comparative_advantage_analysis`, `value_chain_analysis`

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
| `user_context:{domain}` | `user_context:complements` | Domain-specific insider knowledge |
| `document_insights` | `document_insights` | Extracted document insights |

**User Roles:** investor, employee, competitor, student, advisor, executive

**Analysis Modes:**
| Mode | Description | Confidence Impact |
|------|-------------|-------------------|
| document_enriched | User provided docs + answered questions | Full confidence |
| guided_context | No docs, but answered most questions | 0.8x confidence |
| public_only | Minimal user input | 0.5x confidence penalty |

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

### 4.2 State Key Naming Convention

| Pattern | Description |
|---------|-------------|
| `{agent_name}_analysis` | Agent's structured output |
| `{agent_name}_raw` | Raw LLM response (debugging) |
| `executive_summary` | Executive summary from Report Agent |
| `full_report` | Comprehensive report from Report Agent |
| `presentation_structure` | Slide structure from Gemini 3 Pro |
| `slide_visuals` | Generated image paths |
| `presentation_pdf` | Final slides PDF path |
| `report_pdf` | Detailed report PDF path |
| `temp:{key}` | Intermediate calculations |
| `cache:search:{hash}` | Cached search results |
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
Phase 1: Foundation + User Context → [Macro Agent, Market Structure Agent]
              │
              ├── Macro Economy Agent (uses foundation_context)
              │       ↓ macro_economy_analysis
              │
              └── Market Structure Agent
                  ├── Complements Agent (uses user_context:complements)
                  │       ↓ complements_analysis
                  └── Substitutes Agent (uses user_context:substitutes)
                          ↓ substitutes_analysis
              ↓
Phase 2: + Phase 1 outputs → [Comp. Advantage, Value Chain, JTBD]
              │
              ├── Comparative Advantage Agent (uses user_context:advantages)
              │       ↓ comparative_advantage_analysis
              │
              ├── Value Chain Agent (uses user_context:value_chain)
              │       ↓ value_chain_analysis
              │
              └── JTBD Agent (uses user_context:jtbd)
                      ↓ jtbd_analysis
              ↓
Phase 3: + Phase 2 outputs → [Competitive Strategy]
              │
              └── Competitive Strategy Agent (uses user_context:strategy)
                      ↓ competitive_strategy_analysis
              ↓
Phase 4: + All outputs → [SWOT Synthesis, Coherence Check, Narrative]
              ↓
         swot_synthesis, coherence_flags, final_narrative
              ↓
Phase 5: + All outputs → [Report Agent]
              ↓
         executive_summary, full_report
              ↓
Phase 6: + Report outputs → [Presentation Agent]
              ↓
         presentation_structure, slide_visuals, presentation_pdf, report_pdf
```

**Key Insight:** Each analysis agent receives both public data (via tools) AND insider context (via user_context:{domain}). The combination enables dramatically higher-confidence analysis than public-only approaches.

---

## Part 5: Tool Architecture

### 5.1 Market Intel Tool

**Purpose:** Central service for web search, news, and financial data with built-in caching

**Methods:**
| Method | Description |
|--------|-------------|
| `search_web` | Search with source_type (news/general/financial/academic) and recency filters |
| `get_company_financials` | Retrieve financial metrics by ticker |
| `search_sec_filings` | Search SEC filings (10-K, 10-Q, 8-K, DEF14A) |
| `search_earnings_transcripts` | Retrieve earnings call transcripts |

**Caching:** Results cached by query hash with configurable TTL (default 24h)

### 5.2 Document Processor Tool (Docling)

**Purpose:** PDF and document processing with structured extraction

**Methods:**
| Method | Description |
|--------|-------------|
| `process_document` | Extract sections, tables, and metadata |

**Output:** document_id, title, sections[], tables[], metadata

### 5.3 Knowledge Base Tool

**Purpose:** Unified access to all knowledge sources with priority ranking

**Source Priority:** documents → user_context → public_structured → public_unstructured

### 5.4 Visual Generator Tool

**Purpose:** Generate presentation visuals using Nano Banana Pro

**Methods:**
| Method | Description |
|--------|-------------|
| `generate_slide_visual` | Generate visual from description |
| `generate_framework_diagram` | Generate Porter's, SWOT, Value Chain diagrams |

### 5.5 PDF Generator Tool

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
| `search_cache` | Cached search results with TTL |
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
| Search results | query hash | 24 hours (configurable) |
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
│   │   ├── market_intel.py
│   │   ├── document_processor.py
│   │   ├── knowledge_base.py
│   │   ├── visual_generator.py
│   │   ├── pdf_generator.py
│   │   └── analysis_tools.py
│   ├── schemas/
│   │   ├── foundation.py
│   │   ├── context.py             # User context schemas
│   │   ├── outputs.py
│   │   ├── report.py
│   │   ├── presentation.py
│   │   └── common.py
│   ├── services/
│   │   ├── session.py
│   │   ├── cache.py
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
   - Set analysis mode: document_enriched (1.0x) / guided_context (0.8x) / public_only (0.5x)
   - → `analysis:mode`

**Phase 1 — External Context (Parallel):**
- Macro Economy Agent (reads: `foundation_context`) → `macro_economy_analysis`
- Market Structure Agent:
  - Complements Agent (reads: `user_context:complements`) → `complements_analysis`
  - Substitutes Agent (reads: `user_context:substitutes`) → `substitutes_analysis`

**Phase 2 — Firm Analysis (Parallel):**
- Comparative Advantage Agent (reads: `user_context:advantages`) → `comparative_advantage_analysis`
- Value Chain Agent (reads: `user_context:value_chain`) → `value_chain_analysis`
- JTBD Agent (reads: `user_context:jtbd`) → `jtbd_analysis`

**Phase 3 — Strategy (Sequential):**
- Competitive Strategy Agent (reads: `user_context:strategy`) → `competitive_strategy_analysis`

**Phase 4 — Synthesis (Sequential):**
- SWOT Agent → `swot_synthesis`
- Coherence Checker → `coherence_flags`
- Narrative Synthesis → `final_narrative`

**Phase 5 — Report Generation (Sequential):**
- Executive Summary Generator → `executive_summary`
- Full Report Generator → `full_report`

**Phase 6 — Presentation (Sequential):**
- Slide Structure Generator → `presentation_structure`
- Visual Generator → `slide_visuals`
- PDF Assembler → `presentation_pdf`, `report_pdf`

**Completion:**
- Store outputs, log metrics, return results with confidence scores

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
    Analysis Begins (Phase 1-6)
```

### 11.3 How Agents Use Context

Each analysis agent follows this pattern:

1. **Read user context:** `state.get("user_context:{domain}")`
2. **Check confidence level:** If high → prioritize user insights; if low → rely more on public data
3. **Search public sources:** Use Market Intel tool for external data
4. **Synthesize:** Combine insider knowledge + public data
5. **Score confidence:** Apply `analysis:mode` multiplier to final confidence
6. **Flag gaps:** Note where user context was missing in output

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

### 12.2 Key Test Scenarios

**Phase 0 — Context Gathering:**
- Foundation setup captures all required fields
- Document processor extracts relevant insights per domain
- Interview adapts questions based on user role
- Interview skips questions answered by documents
- "I don't know" responses handled gracefully with fallbacks
- Mode determination correctly calculates confidence multiplier
- Context completeness scores are accurate per domain

**Phase 1-6 — Analysis:**
- Agents correctly read and prioritize user_context:{domain}
- Confidence scores reflect analysis:mode multiplier
- Public-only mode explicitly flags lower confidence
- Executive summary answers the strategic question
- Strategic verdict is decisive (not hedged)
- All takeaways trace to source agents
- Critical risks incorporate coherence flags
- Report sections are complete and cross-referenced
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

*Document Version: 4.0*
*Last Updated: 2025-12-11*

**Changelog:**
- **v4.0** (2025-12-11): Added Phase 0 Context Gathering Agent — the critical foundation that extracts insider knowledge before analysis begins. System now has 7 phases (0-6) and 13 agents. Added guided interview with domain-specific questions, document processing, analysis mode determination, and user_context:{domain} state keys consumed by each analysis agent.
- **v3.0** (2025-12-11): Added Phase 5 Report Agent, removed implementation code, converted to specification tables
- **v2.0** (2025-01-15): Added Gemini 3 Pro, Phase 5 Presentation Agent, Nano Banana Pro, PyMuPDF
- **v1.0** (Initial): Core 4-phase analysis pipeline with 9 agents
