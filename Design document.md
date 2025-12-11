---
# Strategy Analysis Agent — Design Document

## Overview

A multi-agent system that helps users think through strategic frameworks. Analyzes companies using classical strategy economics: macro conditions, market structure, comparative advantage, and competitive positioning. Built on Google ADK, integrates with Strategy Map.

---

## Part 1: Agent Architecture

### Conceptual Foundation

The frameworks form a logical analytical sequence:

**Step 1: External Context**
- Macro Economy analysis (cycle, rates, FX, sector spending)

**Step 2: Market Structure**
- Complements Analysis
- Substitutes Analysis

**Step 3: Firm Position**
- Comparative Advantage
- Value Chain
- JTBD (customer needs)

**Step 4: Strategic Choice**
- Competitive Strategy (Cost leadership vs. Differentiation)
- SWOT Synthesis (integrates all findings)

Each stage informs the next. You can't determine competitive strategy without understanding comparative advantage, which requires understanding market structure, which sits within macro conditions.

---

### Agent Hierarchy

**Strategy Orchestrator (Root Agent)**
- Sequences analysis (macro → market → firm → strategy)
- Knows when to parallelize vs. chain
- Synthesizes cross-framework insights
- Challenges weak logic ("You claim cost leadership but your comparative advantage is in R&D — explain the disconnect")

**Top-Level Agents (report to Orchestrator)**
- Macro Economy Agent
- Market Structure Agent (parent to sub-agents)
- Comparative Advantage Agent
- Competitive Strategy Agent

**Market Structure Sub-Agents**
- Complements Analysis Agent
- Substitutes Analysis Agent

**Supporting Agents**
- SWOT Agent (synthesis role)
- JTBD Agent
- Value Chain Agent

**Shared Service**
- Market Intel Agent (web search, news APIs, financial data)

---

### Framework Agents (Specialists)

Each has a narrow focus and specific output schema:

| Agent | Focus | Key Tools | Output Structure |
|-------|-------|-----------|------------------|
| Macro Economy | Business cycle, rates, FX, sector spending | economic_data, fed_watch, sector_indicators | `{cycle_phase, rate_environment, consumer_spending_outlook, fx_impact, strategic_implications: []}` |
| Complements Analysis | Products that increase value when used together | ecosystem_search, partnership_tracker, app_store_data | `{key_complements: [{category, players, integration_depth, ecosystem_health, vertical_integration_risk}], strategic_implications: []}` |
| Substitutes Analysis | Alternative ways to solve the same job | competitor_search, trend_analysis, switching_cost_model | `{direct_substitutes: [], indirect_substitutes: [], switching_costs, substitution_triggers: [], defensibility_zones: []}` |
| Comparative Advantage | What the firm does that others cannot easily replicate | patent_search, capability_analysis, historical_investment | `{primary_advantages: [{advantage, durability, replicability, monetization}], areas_of_parity: [], areas_of_disadvantage: []}` |
| Competitive Strategy | Cost leadership vs. differentiation positioning | pricing_analysis, cost_structure, brand_perception | `{current_position, target_segment, coherence_assessment: {score, gaps: []}, recommended_position, strategic_moves: []}` |
| SWOT | Synthesis of internal/external factors | pulls_from_other_agents | `{strengths: [], weaknesses: [], opportunities: [], threats: [], cross_references: []}` |
| JTBD | Customer needs and hiring criteria | interview_parser, review_analyzer, survey_data | `{jobs: [{job, importance, current_satisfaction}], underserved_jobs: [], opportunities: []}` |
| Value Chain | Activity analysis and margin drivers | cost_structure_tool, segment_profitability | `{primary_activities: [], support_activities: [], margin_drivers: [], cost_centers: []}` |

---

### Market Intel Agent (Shared Service)

- Called by other agents when they need external data
- Wraps web search, news APIs, financial data (SEC filings, earnings transcripts)
- Caches results to avoid redundant searches within a session
- Returns structured, source-attributed findings

---

### Orchestration Logic

The orchestrator manages dependencies:

**Phase 1: Context Setting (can parallelize)**
- Macro Economy Agent
- Market Structure Agent
  - Complements Analysis (parallel)
  - Substitutes Analysis (parallel)

**Phase 2: Internal Analysis (after Phase 1 completes)**
- Value Chain Agent
- Comparative Advantage Agent (needs market context)
- JTBD Agent

**Phase 3: Strategy Determination (after Phase 2 completes)**
- Competitive Strategy Agent (needs comparative advantage input)

**Phase 4: Synthesis**
- SWOT Agent (integrates all findings)
- Orchestrator narrative synthesis

---

### Cross-Agent Coherence Checks

The orchestrator flags contradictions:

| If This... | And This... | Then Flag... |
|------------|-------------|--------------|
| Macro shows contraction | Strategy recommends growth investment | Timing risk |
| Comparative advantage is technical depth | Strategy is cost leadership | Potential misalignment |
| Substitutes are gaining (format shift) | No mention in strategic moves | Blind spot |
| High complement dependency | No partnership strategy | Execution gap |
| JTBD shows customers want simplicity | Differentiation is feature richness | Value mismatch |

---

## Part 2: Information Supply Chain

### Information Source Hierarchy

Ranked by richness and reliability (highest to lowest):

1. **User-Provided Documents** (highest richness, lowest availability)
   - Internal data, financials, strategy docs, board materials

2. **User-Provided Context**
   - Verbal/text input about their situation, constraints, goals

3. **Public Structured Data**
   - SEC filings, earnings transcripts, analyst reports, industry data

4. **Public Unstructured Data** (lowest richness, highest availability)
   - News articles, blog posts, social sentiment, general web

The challenge: **richness and availability are inversely correlated**. The best information is hardest to get.

---

### Three Operating Modes

#### Mode 1: Document-Enriched Analysis

**Trigger**: User uploads PDFs or documents

**What changes**:
- PDF content becomes the **primary source** for internal/firm-specific claims
- Web search becomes **supplementary** — fills gaps, provides external context
- Agent confidence levels are higher for firm-specific analysis
- Can do deeper Value Chain, Comparative Advantage, internal SWOT factors

**Document types that matter**:

| Document Type | Feeds Which Agents | Information Extracted |
|---------------|-------------------|----------------------|
| Annual reports / 10-K | Value Chain, Competitive Strategy, Macro exposure | Segment revenue, cost structure, risk factors, strategy narrative |
| Investor presentations | Competitive Strategy, Comparative Advantage | Positioning claims, strategic priorities, KPIs |
| Internal strategy docs | All agents | Actual strategic intent (vs. public narrative) |
| Competitor analysis | Substitutes, Complements | Market map, competitive dynamics |
| Customer research | JTBD, Substitutes | Jobs, satisfaction, switching triggers |
| Board materials | Comparative Advantage, Competitive Strategy | Real constraints, internal debates |

**Tool orchestration when user uploads 10-K + investor deck**:

Step 1: PDF Parser extracts structured content
- Segment financials → Value Chain Agent
- Risk factors → SWOT (threats), Macro exposure
- Competitive discussion → Substitutes, Complements
- Strategy narrative → Competitive Strategy

Step 2: Web Search fills gaps
- Recent news (post-document date)
- Competitor moves not in documents
- Macro indicators (current, not historical)
- Analyst perspectives

Step 3: Cross-reference for consistency
- Does public narrative match internal docs?

---

#### Mode 2: Guided Context Gathering

**Trigger**: User doesn't upload documents

**Philosophy**: Don't just ask for documents — ask **strategic questions** that extract the same information conversationally.

**Context gathering sequence**:

**Tier 1: Essential (ask first)**
- What company/business are we analyzing?
- What's the strategic question you're trying to answer?
- What's your role? (Investor? Employee? Competitor? Student?)
  - This changes what information they'll have access to

**Tier 2: Internal Context (if they seem willing)**
- What would you say is the company's main competitive advantage?
- Who do they lose deals/customers to most often?
- What's their approximate revenue mix? (by segment, geography, customer type)
- What strategic options are they considering?

**Tier 3: Constraints (if relevant)**
- Are there resource constraints I should know about?
- Any sacred cows or off-limits options?
- What's the time horizon for this decision?

**Adaptive behavior**:
- If user answers richly → Treat answers like document content (high confidence)
- If user gives partial answers → Supplement with web search
- If user says "I don't know" → Note uncertainty, rely on public data
- If user says "I can't share that" → Respect it, work around it

---

#### Mode 3: Public-Only Analysis

**Trigger**: User refuses to provide context or only gives company name

**Constraints**:
- Can only analyze what's publicly observable
- Internal dynamics are inferred, not known
- Confidence levels should be lower, especially for:
  - Comparative Advantage (hard to see from outside)
  - Value Chain details (cost structure is opaque)
  - True strategic intent (vs. PR narrative)

**What web search CAN do well**:
- Macro Economy — public data, well-covered
- Complements — ecosystem partnerships are public
- Substitutes — market alternatives are visible
- JTBD — customer reviews, forums, public research
- Competitive Strategy — pricing, positioning are observable

**What web search CANNOT do well**:
- Internal cost structure
- True capability gaps
- Strategic debates and constraints
- Cultural factors
- Unpublicized competitive losses

---

### Tool Architecture

**Information Layer**

Three input channels feed into Knowledge Fusion:

**1. Document Processor**
- PDF parsing
- Table extraction
- Section classification
- Cross-doc synthesis

**2. Context Manager**
- Question sequencing
- Answer validation
- Gap identification
- Confidence scoring

**3. Web Search**
- News search
- Company profiles
- Financial data
- Industry reports
- Earnings transcripts

**Knowledge Fusion** (receives from all three channels)
- Source priority ranking
- Conflict detection
- Confidence scoring
- Gap flagging

**Output**: Fused knowledge feeds into Framework Agents

---

### Knowledge Fusion Logic

When multiple sources provide information on the same topic:

**Priority rules**:
1. User documents > User verbal context > Public structured > Public unstructured
2. Recent > Old (for time-sensitive facts)
3. Primary source > Secondary source (Apple's 10-K > analyst interpretation)

**Conflict handling example**:
- Document says: "We are the cost leader in our segment"
- Web search shows: Premium pricing, high margins vs. competitors
- Action: Flag conflict to user: "The strategy documents claim cost leadership, but public pricing and margin data suggest differentiation positioning. Which reflects current reality?"

**Gap flagging example**:
- Agent needs: Cost structure breakdown by segment
- Available: Only consolidated margins from public filings
- Action: Flag to orchestrator: "Value Chain analysis limited — segment-level cost data unavailable. Proceeding with consolidated margins. Confidence: Medium."

---

### Confidence Scoring

Each agent output carries confidence metadata:

```
{
  agent: "comparative_advantage",
  findings: [...],
  confidence: {
    overall: 0.6,
    limiting_factors: [
      "No internal documents provided",
      "Capability claims based on public statements only",
      "Could not verify R&D investment allocation"
    ],
    high_confidence_claims: [
      "Vertical integration advantage (observable)",
      "Brand premium (pricing data confirms)"
    ],
    low_confidence_claims: [
      "Operational efficiency vs. competitors",
      "True technology differentiation"
    ]
  }
}
```

---

## Part 3: User Interface — Progressive Disclosure

The UI transitions through three phases, matching the natural workflow from context gathering to analysis to exploration.

---

### Phase 1: Context Gathering

**Purpose**: Collect information, establish operating mode, build rapport

**UI State**: Pure conversational interface. No artifacts, no dashboards. Focus entirely on the dialogue.

**Screen layout**:
- Header: "Strategy Agent"
- Main area: Chat messages (agent and user)
- Footer: Text input field + Send button
- Optional: [Upload files] button within agent message

**Sample conversation flow**:

1. Agent: "What company would you like to analyze?"
2. User: "Apple"
3. Agent: "Got it — Apple. A few quick questions to calibrate:
   - What's your angle — investor, employee, competitor, student, or something else?
   - Any specific strategic question, or a general assessment?
   - Do you have documents to share (10-K, investor deck)? [Upload files]"
4. User: "I'm a student. Looking at whether their services pivot makes strategic sense. No documents."
5. Agent: "Good question — gets at strategic coherence. I'll use public sources: SEC filings, earnings calls, analyst coverage, news. I'll be upfront about what I can and can't see from outside. Ready to start?
   - [Start Analysis] [Ask me more first]"

**Key behaviors**:
- Conversational, not form-like
- Questions adapt based on prior answers
- Document upload is offered but not required
- Agent explains what operating mode it will use
- Clear handoff moment before analysis begins

---

### Phase 2: Analysis in Progress

**Purpose**: Show work happening, surface interesting findings, maintain engagement during longer analysis

**UI State**: Progress tracker appears above conversation. Chat continues but shifts to commentary mode.

**Screen layout**:

**Progress Tracker Panel (top)**:
- Title: "ANALYZING: Apple — Services Strategy Coherence"
- Section: EXTERNAL CONTEXT
  - ✓ Macro Economy [view]
  - ✓ Market Structure [view]
    - ✓ Complements
    - ✓ Substitutes
- Section: FIRM ANALYSIS
  - ● Comparative Advantage (analyzing...)
  - ○ Value Chain (pending)
  - ○ JTBD (pending)
- Section: STRATEGIC POSITION
  - ○ Competitive Strategy (pending)
  - ○ SWOT Synthesis (pending)
- Progress bar: 62%

**Chat Area (below tracker)**:
- Agent surfaces findings mid-analysis
- Example: "Interesting finding from the complements analysis: Apple's relationship with content providers (Netflix, Spotify) is increasingly adversarial..."
- Hint: "You can ask questions while I work, or click [view] to see completed analyses."

**Inline preview when user clicks [view]**:
- Modal/panel showing completed artifact
- Example for Complements Analysis:
  - Key Complements Identified:
    - App Developers: ●●●●● Critical — "2M+ apps, defines platform value"
    - Accessory Ecosystem: ●●●○○ Moderate — "Cases, chargers, audio — extends hardware value"
    - Enterprise Software: ●●●○○ Moderate — "Microsoft 365, Salesforce mobile integration"
    - Content Providers: ●●○○○ Weakening — "Increasingly competitive relationship"
  - Vertical Integration Risk: High for content (Apple TV+, Apple Music competing with partners)
  - Confidence: High (ecosystem partnerships publicly observable)
  - Buttons: [Expand] [Ask about this] [Close]

**Key behaviors**:
- Progress tracker shows agent pipeline (users see the methodology)
- Completed agents have [view] links for early access
- Agent surfaces interesting findings mid-analysis (not silent)
- User can ask questions without interrupting work
- Visual hierarchy: progress tracker is informational, chat is interactive

---

### Phase 3: Exploration Mode

**Purpose**: Present completed analysis, enable drill-down, support debate and refinement

**UI State**: Full canvas layout. Artifacts are primary, organized by framework. Chat anchored to selected artifact or global.

**Screen layout**:

**Header**:
- Title: "Apple — Services Strategy Analysis"
- Actions: [Export] [Save] [Share]

**Artifact Navigation Bar (horizontal row of cards)**:
- MACRO ECONOMY — ●●●●○ conf.
- MARKET STRUCTURE — ●●●●○ conf.
- COMPARATIVE ADVANTAGE — ●●●○○ conf.
- VALUE CHAIN — ●●○○○ conf.
- COMPETITIVE STRATEGY — ●●●●○ conf.
- SWOT SYNTHESIS — ●●●○○ conf.
- Click any card to expand into main canvas
- Visual connections show dependencies between cards

**Main Canvas (center, shows selected artifact)**:

Example — Competitive Strategy expanded:
- Header: "COMPETITIVE STRATEGY" [Edit] [Source]
- Position: DIFFERENTIATION
- Scope: Broad (mass premium)
- Visual: Porter's generic strategy matrix with Apple positioned in Broad + Differentiation quadrant
- Coherence Assessment: 0.72
- Tensions Identified:
  1. Services growth may require Android presence, diluting ecosystem advantage (cross-ref: Comparative Advantage)
  2. Competing with App Store developers creates channel conflict (cross-ref: Complements Analysis)
  3. Services margins (70%+) vs. hardware (36%) creates internal resource allocation tension (cross-ref: Value Chain)
- Confidence: High for positioning, Medium for internal dynamics

**Contextual Chat Panel (bottom)**:
- Label: "DISCUSSING: Competitive Strategy"
- Shows conversation thread specific to selected artifact
- Context switcher: "Context: Competitive Strategy [Switch to: Global] [SWOT] [Complements]"
- Text input + Send button

**Key behaviors**:

*Artifact Cards (top row)*:
- Visual summary of each framework
- Confidence indicator (dot rating)
- Click to expand into main canvas
- Connections between cards show dependencies

*Main Canvas (center)*:
- Full artifact view with structured output
- Cross-references to other frameworks (clickable)
- Edit mode for user corrections
- Source links for verification
- Tension/coherence flags prominent

*Contextual Chat (bottom)*:
- Anchored to selected artifact by default
- Can switch to global context
- Quick switches to related artifacts
- Agent responds with awareness of what user is viewing

---

### Phase Transitions

**Phase 1 → Phase 2**:
- Trigger: User clicks [Start Analysis] or provides sufficient context
- Transition: Progress tracker slides down from top, pushes chat down
- Chat continues but agent shifts to progress commentary

**Phase 2 → Phase 3**:
- Trigger: All agents complete
- Transition: Progress tracker morphs into artifact navigation bar
- Full canvas reveals below
- Chat repositions to contextual panel
- Agent delivers synthesis message: "Here's what I found..."

**Handling Interruptions**:
- User asks question during Phase 2 → Agent pauses commentary to respond, then resumes
- User clicks [view] during Phase 2 → Inline preview opens, doesn't break flow
- User wants to restart → [New Analysis] button always available

---

### Artifact Interactions

**Viewing**:
- Click card → Expand to main canvas
- Hover card → Tooltip with key finding
- Confidence dots → Hover shows limiting factors

**Editing**:
- Click [Edit] → Fields become editable
- User changes value → Agent responds: "Noted. Should I re-evaluate dependent analyses?"
- Suggest mode: User proposes, agent confirms and propagates

**Challenging**:
- Click [Ask about this] → Pre-populates chat with artifact context
- Agent defends or revises based on dialogue
- Revisions tracked: "Updated based on your input (v2)"

**Exporting**:
- Individual artifact → PNG, PDF, or structured data
- Full analysis → Strategy Map import format, PDF report, slide deck
- Conversation log → Markdown with artifact references

---

### Confidence Visualization

Each artifact shows confidence, but details on demand:

**Collapsed view (in card)**:
- ●●●○○ (3/5 confidence)

**Expanded view (in main canvas)**:
- Confidence: Medium (0.6)
- High confidence claims:
  - Industry structure (publicly observable)
  - Competitive positioning (pricing/marketing visible)
- Lower confidence claims:
  - Internal cost structure (inferred from margins)
  - True strategic priorities (PR vs. reality unclear)
- Would improve with:
  - 10-K filing analysis
  - Earnings call transcript

---

### Mobile Considerations

On smaller screens, progressive disclosure becomes linear:

- Phase 1: Full screen chat (same as desktop)
- Phase 2: Progress tracker at top, chat below (collapsed artifacts)
- Phase 3: Tab navigation between artifacts, chat as bottom sheet

---

## Part 4: Conversation Design

The agent is a **thinking partner**, not a report generator. Conversation style adapts by phase:

| Phase | Style | Examples |
|-------|-------|----------|
| Context Gathering | Socratic, curious | "What's your angle on this?" / "Anything specific you're trying to decide?" |
| Analysis Running | Informative, engaged | "Interesting — the complements picture is more complex than I expected..." |
| Findings Delivery | Synthesized, opinionated | "The core tension is X. Here's why that matters..." |
| Exploration | Responsive, debatable | "Fair challenge. Let me reconsider..." / "I'd push back on that because..." |

**Agent personality traits**:
- Intellectually honest (flags uncertainty)
- Willing to be challenged (revises when wrong)
- Proactively surfaces tensions (doesn't just report)
- Concise but not curt

---
