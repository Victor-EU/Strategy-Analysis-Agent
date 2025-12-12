# Deep Research Agent Integration Analysis

## Executive Summary

This document analyzes how Gemini's Deep Research Agent could be integrated into the Strategy Analysis Agent architecture to dramatically improve research quality through autonomous, multi-step web research with cited sources.

**Key Decision:** Recommend **Hybrid Approach** — predictive deep research in Phase 0.5 with on-demand fallback.

---

## Part 1: Deep Research Agent Overview

### What It Does

The Gemini Deep Research Agent is an autonomous research tool that plans, executes, and synthesizes multi-step research tasks. Rather than generating immediate responses, it iteratively searches the web to produce detailed, cited reports.

| Characteristic | Detail |
|----------------|--------|
| **Function** | Autonomous multi-step web research → detailed cited reports |
| **Execution** | Async (`background=true`), up to 60 min, typically ~20 min |
| **Output** | Structured reports with citations |
| **Best For** | Market analysis, due diligence, competitive landscaping, literature reviews |
| **Cannot Do** | Real-time chat, custom tools/MCP, quick lookups |

### API Access

Deep Research operates exclusively through the **Interactions API** (not `generate_content`):

```python
from google import genai

client = genai.Client()

# Launch deep research (async)
interaction = client.interactions.create(
    input="Research the competitive landscape for electric vehicles",
    agent='deep-research-pro-preview-12-2025',
    background=True
)

# Poll for completion
while True:
    interaction = client.interactions.get(interaction.id)
    if interaction.status == "completed":
        report = interaction.outputs[-1].text
        break
    time.sleep(10)
```

### Key Parameters

| Parameter | Purpose |
|-----------|---------|
| `input` | Research query or task description |
| `background` | Must be `true` for async execution |
| `stream` | Enable real-time progress updates |
| `tools` | Add `file_search` for private document access |
| `agent_config` | Configure thinking summaries with `"thinking_summaries": "auto"` |

### Limitations

- **Max research time:** 60 minutes (typical: ~20 minutes)
- **No custom tools:** Cannot add custom function calling or MCP servers
- **No audio inputs** supported
- **Beta status:** API schemas may change

---

## Part 2: Current Architecture Analysis

### Where Web Research Currently Happens

The current architecture (v4.0) has:

- **Market Intel Tool** in Shared Services for web search
- Each analysis agent (Phase 1-3) can call this tool for lookups

### Fundamental Mismatch

| Our Current Approach | Deep Research Agent |
|---------------------|---------------------|
| Synchronous, quick lookups | Asynchronous, deep dives |
| Agent stays in control | Hands off to Deep Research |
| Shallow but fast | Deep but slow (20 min avg) |
| Agent decides what to search | Deep Research decides autonomously |

---

## Part 3: Integration Options

### Option A: Dedicated Research Phase (Phase 0.5)

```
Phase 0: Context Gathering (user interview)
Phase 0.5: Deep Research (async, parallel research tasks)
   ├── Company Deep Dive
   ├── Industry Landscape
   ├── Competitor Profiles (parallel)
   └── Market Trends
Phase 1-6: Analysis using pre-gathered research
```

| Pros | Cons |
|------|------|
| Research happens once, shared by all agents | Adds 20+ minutes upfront |
| No 20-minute waits during analysis phases | Research is speculative |
| Clean separation: research vs. analysis | Can't adapt based on analysis findings |
| Multiple Deep Research calls in parallel | — |

---

### Option B: On-Demand Tool (Lazy Invocation)

Each agent can invoke Deep Research when it encounters a complex question:

```python
# Inside Value Chain Agent
if needs_deep_research(question):
    research = await deep_research_tool.invoke(
        query=f"Detailed analysis of {company}'s supply chain economics",
        timeout=30*60
    )
```

| Pros | Cons |
|------|------|
| Research only when needed | 20+ min per agent (sequential bottleneck) |
| Contextual — agent knows exactly what it needs | Unpredictable total analysis time |
| Can handle unexpected deep dives | UX nightmare if multiple agents need it |

---

### Option C: Hybrid — Pre-Research + On-Demand Cache (RECOMMENDED)

```
Phase 0: Context Gathering
Phase 0.5: Predictive Deep Research
   └── Based on Phase 0 context, predict what research is needed
   └── Run multiple Deep Research tasks in parallel
   └── Cache results in state

Phase 1-3: Analysis (use cached research, can request more if needed)
   └── If cache miss → quick web search (not Deep Research)
   └── Flag for potential Deep Research refinement

Phase 4+: Synthesis & Report
```

**Why This Is Optimal:**

1. **Front-loads the slow work** — User waits once at the start
2. **Predictive research** — Phase 0 context tells us what to research
3. **Parallel execution** — Multiple Deep Research tasks run simultaneously
4. **Graceful degradation** — If something isn't pre-researched, quick search fallback
5. **High cache hit rate** — If we predict well, analysis phases are fast

---

## Part 4: Recommended Architecture

### Phase 0.5: Deep Research Agent

**Purpose:** Conduct comprehensive, parallel web research based on Phase 0 context to pre-populate research cache for all downstream agents.

**Type:** `ParallelAgent` managing multiple Deep Research interactions

**Position:** After Phase 0 (Context Gathering), before Phase 1 (External Context)

### Research Task Planning

After Phase 0 completes, predict research needs based on the strategic question:

| Analysis Domain | Predictable Research Tasks |
|-----------------|---------------------------|
| Complements | Partner ecosystem research, API/integration landscape |
| Substitutes | Competitor deep dives (top 3-5), alternative solutions |
| Macro Economy | Industry trends, regulatory landscape, economic factors |
| Value Chain | Supply chain analysis, industry cost structures |
| JTBD | Customer research synthesis, market segmentation studies |
| Competitive Strategy | Strategic positioning analysis, competitive dynamics |

### Research Task Templates

```python
RESEARCH_TASK_TEMPLATES = {
    "company_overview": {
        "query": "Comprehensive business analysis of {company}: business model, revenue streams, key products/services, recent strategic moves, financial performance, and market position in {industry}",
        "priority": 1,
        "always_run": True
    },
    "competitor_landscape": {
        "query": "Detailed competitive landscape analysis for {company} in {industry}: top 5 competitors, their strengths/weaknesses, market share, strategic positioning, and competitive dynamics",
        "priority": 1,
        "always_run": True
    },
    "industry_trends": {
        "query": "Industry trends and market dynamics in {industry} 2024-2025: growth drivers, disruption risks, regulatory changes, technology shifts, and emerging opportunities",
        "priority": 2,
        "always_run": True
    },
    "value_chain_economics": {
        "query": "Value chain and supply chain analysis for {industry}: key activities, cost structures, margin distribution, make-vs-buy trends, and vertical integration patterns",
        "priority": 2,
        "condition": "value_chain in strategic_question"
    },
    "customer_segments": {
        "query": "Customer segmentation and jobs-to-be-done analysis for {company} in {industry}: customer types, purchase drivers, switching costs, and unmet needs",
        "priority": 2,
        "condition": "jtbd in strategic_question OR customer in strategic_question"
    },
    "macro_factors": {
        "query": "Macroeconomic and regulatory factors affecting {industry} in {geography}: economic indicators, policy changes, trade dynamics, and geopolitical risks",
        "priority": 3,
        "condition": "macro in strategic_question OR regulation in strategic_question"
    }
}
```

### Invocation Pattern

```python
async def run_deep_research_phase(context: FoundationContext) -> ResearchCache:
    """Phase 0.5: Parallel Deep Research execution."""

    client = genai.Client()
    research_tasks = select_research_tasks(context)

    # Launch all research tasks in parallel
    interactions = []
    for task in research_tasks:
        query = task["query"].format(
            company=context.company_name,
            industry=context.industry,
            geography=context.geography
        )

        interaction = client.interactions.create(
            input=query,
            agent='deep-research-pro-preview-12-2025',
            background=True,
            stream=True,
            agent_config={"type": "deep-research", "thinking_summaries": "auto"}
        )
        interactions.append({
            "key": task["key"],
            "interaction": interaction,
            "started_at": time.time()
        })

    # Collect results with progress tracking
    research_cache = ResearchCache()
    for item in interactions:
        result = await poll_for_completion(
            client,
            item["interaction"],
            progress_callback=lambda p: emit_progress(item["key"], p)
        )
        research_cache.set(item["key"], result)

    return research_cache
```

### Research Cache Structure

```python
@dataclass
class ResearchResult:
    query: str
    report: str
    citations: list[Citation]
    research_time_seconds: int
    completed_at: datetime

@dataclass
class ResearchCache:
    results: dict[str, ResearchResult]

    def get(self, key: str) -> Optional[ResearchResult]:
        return self.results.get(key)

    def get_relevant(self, keywords: list[str]) -> list[ResearchResult]:
        """Retrieve research results relevant to given keywords."""
        relevant = []
        for key, result in self.results.items():
            if any(kw.lower() in result.report.lower() for kw in keywords):
                relevant.append(result)
        return relevant
```

### State Keys

| Key | Type | Description |
|-----|------|-------------|
| `research_cache` | ResearchCache | All deep research results |
| `research_cache:{domain}` | ResearchResult | Domain-specific research |
| `research_metadata` | object | Timing, task count, cache stats |

---

## Part 5: Updated Phase Structure

### Before (v4.0)

| Phase | Name | Agents |
|-------|------|--------|
| 0 | Context Gathering | 3 |
| 1 | External Context | 2 |
| 2 | Firm Analysis | 3 |
| 3 | Strategy | 1 |
| 4 | Synthesis | 3 |
| 5 | Report Generation | 2 |
| 6 | Presentation | 3 |
| **Total** | **7 phases** | **17 sub-agents** |

### After (v5.0 with Deep Research)

| Phase | Name | Agents | Duration |
|-------|------|--------|----------|
| 0 | Context Gathering | 3 | 5-10 min |
| 0.5 | **Deep Research** | 1 (parallel tasks) | 15-25 min |
| 1 | External Context | 2 | 2-3 min |
| 2 | Firm Analysis | 3 | 3-5 min |
| 3 | Strategy | 1 | 2-3 min |
| 4 | Synthesis | 3 | 2-3 min |
| 5 | Report Generation | 2 | 3-5 min |
| 6 | Presentation | 3 | 5-10 min |
| **Total** | **8 phases** | **18 sub-agents** | **40-65 min** |

---

## Part 6: Agent Integration

### How Analysis Agents Use Research Cache

Each analysis agent receives the research cache and queries it for relevant information:

```python
class ComparativeAdvantageAgent(Agent):
    async def analyze(self, state: AnalysisState) -> ComparativeAdvantageOutput:
        # Get relevant research from cache
        research = state.research_cache.get_relevant([
            "competitive advantage",
            "differentiation",
            "market position",
            state.company_name
        ])

        # Combine with user context
        user_context = state.get("user_context:advantages", {})

        # Run analysis with enriched context
        prompt = self.build_prompt(
            company=state.company_name,
            research_reports=research,
            user_insights=user_context
        )

        return await self.llm.generate(prompt)
```

### Fallback for Cache Misses

When an agent needs information not in the cache:

```python
async def get_research(self, query: str, state: AnalysisState) -> str:
    # First: check cache
    cached = state.research_cache.get_relevant(query.split())
    if cached:
        return self.synthesize_from_cache(cached, query)

    # Fallback: quick web search (NOT Deep Research)
    # Deep Research is too slow for on-demand use
    result = await self.market_intel_tool.search(query)

    # Flag for potential future deep research
    state.flag_research_gap(query)

    return result
```

---

## Part 7: User Experience

### Progress Display

Since Deep Research takes time, provide clear progress feedback:

```
┌─────────────────────────────────────────────────────────────┐
│  Strategy Analysis: Acme Corp                                │
├─────────────────────────────────────────────────────────────┤
│  ✓ Phase 0: Context Gathering Complete                      │
│                                                              │
│  🔍 Phase 0.5: Deep Research in Progress...                 │
│                                                              │
│    ├── [████████████████░░░░] Company Overview (80%)        │
│    ├── [██████████████░░░░░░] Competitor Landscape (70%)    │
│    ├── [████████░░░░░░░░░░░░] Industry Trends (40%)         │
│    └── [██████░░░░░░░░░░░░░░] Value Chain Economics (30%)   │
│                                                              │
│  ⏱️  Estimated time remaining: ~8 minutes                    │
│                                                              │
│  💡 Deep research improves analysis quality by gathering     │
│     comprehensive, cited market intelligence.                │
└─────────────────────────────────────────────────────────────┘
```

### Analysis Mode Selection

Offer users a choice at the start:

| Mode | Deep Research | Duration | Quality |
|------|--------------|----------|---------|
| **Quick Analysis** | No | 25-40 min | Good (public data + user context) |
| **Comprehensive Analysis** | Yes | 40-65 min | Excellent (cited research + context) |

```
┌─────────────────────────────────────────────────────────────┐
│  Select Analysis Mode                                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ○ Quick Analysis                                           │
│    ~30 minutes | Uses public data and your provided context │
│                                                              │
│  ● Comprehensive Analysis (Recommended)                     │
│    ~50 minutes | Includes deep web research with citations  │
│    Best for: Investment decisions, strategic planning       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Part 8: Quality Impact

### Confidence Score Adjustment

Deep Research affects the confidence limiting factors:

| Factor | Without Deep Research | With Deep Research |
|--------|----------------------|-------------------|
| Public data quality | Variable | High (cited sources) |
| Competitor coverage | Partial | Comprehensive |
| Industry trend accuracy | Moderate | High (recent data) |
| Source reliability | Unknown | Verified citations |

### Analysis Mode Multipliers (Updated)

| Mode | Context Sources | Confidence Multiplier |
|------|----------------|----------------------|
| `comprehensive` | Documents + Interview + Deep Research | 1.2x |
| `document_enriched` | Documents + Deep Research | 1.0x |
| `guided_context` | Interview + Deep Research | 0.9x |
| `public_only` | Deep Research only | 0.7x |
| `quick_public` | Quick search only (no Deep Research) | 0.5x |

---

## Part 9: Cost Considerations

### Deep Research Pricing

- **Google Search calls:** Free until January 5, 2026
- **After January 2026:** Standard pricing applies (see Gemini pricing docs)

### Estimated Cost Per Analysis

| Component | Tokens/Calls | Cost (est.) |
|-----------|-------------|-------------|
| Deep Research (4 tasks) | ~4 searches × 20 iterations | TBD after Jan 2026 |
| Analysis Agents | ~50K tokens | ~$0.50 |
| Report Generation | ~30K tokens | ~$0.30 |
| **Total** | — | ~$1-2 per analysis (estimate) |

---

## Part 10: Implementation Checklist

### Phase 1: Core Integration

- [ ] Add Deep Research Agent to agent layer
- [ ] Implement ResearchCache data structure
- [ ] Create research task selection logic
- [ ] Add state keys for research results
- [ ] Update Strategy Orchestrator for Phase 0.5

### Phase 2: Agent Updates

- [ ] Update all analysis agents to query ResearchCache
- [ ] Implement cache-miss fallback logic
- [ ] Add research gap flagging
- [ ] Update confidence scoring for research quality

### Phase 3: User Experience

- [ ] Add analysis mode selection UI
- [ ] Implement progress tracking for Deep Research
- [ ] Add streaming support for research updates
- [ ] Create research summary in final report

### Phase 4: Testing & Optimization

- [ ] Test parallel research execution
- [ ] Measure cache hit rates
- [ ] Optimize research task templates
- [ ] A/B test Quick vs. Comprehensive modes

---

## Appendix A: API Reference

### Interactions API for Deep Research

```python
# Create interaction
interaction = client.interactions.create(
    input="Research query here",
    agent='deep-research-pro-preview-12-2025',
    background=True,
    stream=True,
    agent_config={
        "type": "deep-research",
        "thinking_summaries": "auto"
    }
)

# Get interaction status
interaction = client.interactions.get(interaction.id)
# Status: "pending", "running", "completed", "failed"

# Stream progress
for chunk in stream:
    if chunk.event_type == "interaction.start":
        interaction_id = chunk.interaction.id
    elif chunk.event_type == "thinking_summary":
        print(f"Research progress: {chunk.text}")
    elif chunk.event_type == "interaction.complete":
        final_report = chunk.interaction.outputs[-1].text
```

### Adding Private Documents

```python
interaction = client.interactions.create(
    input="Compare our strategy docs against market trends",
    agent="deep-research-pro-preview-12-2025",
    background=True,
    tools=[{
        "type": "file_search",
        "file_search_store_names": ['fileSearchStores/strategy-docs']
    }]
)
```

---

## Appendix B: Research Task Examples

### For a SaaS Company Analysis

```python
research_tasks = [
    {
        "key": "company_overview",
        "query": "Comprehensive analysis of Salesforce: business model, product portfolio, revenue breakdown by cloud, recent acquisitions, AI strategy, and competitive positioning in enterprise software"
    },
    {
        "key": "competitor_analysis",
        "query": "Competitive landscape for Salesforce CRM: detailed comparison with Microsoft Dynamics 365, HubSpot, Oracle, SAP, and emerging competitors. Include market share, pricing, feature comparison, and strategic direction"
    },
    {
        "key": "industry_trends",
        "query": "Enterprise SaaS and CRM industry trends 2024-2025: AI integration, vertical solutions, pricing model evolution, consolidation trends, and growth forecasts"
    },
    {
        "key": "customer_analysis",
        "query": "Salesforce customer analysis: customer segments, enterprise vs SMB mix, churn patterns, expansion revenue, and jobs-to-be-done for CRM buyers"
    }
]
```

---

## Document Information

| Field | Value |
|-------|-------|
| Version | 1.0 |
| Status | Proposal |
| Author | Strategy Analysis Agent Team |
| Compatible With | Architecture v4.0+ |
| Decision Required | Approve integration approach |
