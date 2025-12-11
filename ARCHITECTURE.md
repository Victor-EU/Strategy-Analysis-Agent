# Strategy Analysis Agent — Technical Architecture

## Executive Summary

A multi-agent system built on Google ADK (Agent Development Kit) that analyzes companies using classical strategy economics frameworks. The system orchestrates 9 specialized agents through a 5-phase execution pipeline, with shared context, session-based caching via SQLite, comprehensive logging, and automated presentation generation.

**Tech Stack:**
- Framework: Google ADK (Python)
- LLM: Gemini 3 Pro (`gemini-3-pro-preview`) — most advanced reasoning model with 1M context
- Image Generation: Nano Banana Pro (`gemini-3-pro-image-preview`) — for slide visuals
- PDF Processing: Docling (extraction) + PyMuPDF (generation)
- Database: SQLite (session caching + logging)
- Frontend: React/Next.js with modern, elegant light blue design system
- Language: Python 3.11+

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
│  │  │   - Manages 5-phase execution (Sequential + Parallel)                 │  │  │
│  │  │   - Enforces dependency chain                                         │  │  │
│  │  │   - Runs coherence checks                                             │  │  │
│  │  │   - Synthesizes final narrative                                       │  │  │
│  │  │   - Triggers presentation generation                                  │  │  │
│  │  └──────────────────────────────────────────────────────────────────────┘  │  │
│  └────────────────────────────────────────────────────────────────────────────┘  │
│                                       │                                          │
│  ┌────────────────────────────────────┼────────────────────────────────────────┐ │
│  │                          AGENT LAYER                                        │ │
│  │                                    │                                        │ │
│  │  Phase 1 (Parallel)               │                                        │ │
│  │  ┌──────────────┐  ┌──────────────┴──────────────┐                         │ │
│  │  │ Macro Economy│  │     Market Structure        │                         │ │
│  │  │    Agent     │  │         Agent               │                         │ │
│  │  │              │  │  ┌──────────┐ ┌──────────┐  │                         │ │
│  │  │              │  │  │Complem-  │ │Substit-  │  │                         │ │
│  │  │              │  │  │ents      │ │utes      │  │                         │ │
│  │  └──────────────┘  │  └──────────┘ └──────────┘  │                         │ │
│  │                    └─────────────────────────────┘                         │ │
│  │                                                                             │ │
│  │  Phase 2 (Parallel, after Phase 1)                                         │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                      │ │
│  │  │ Comparative  │  │ Value Chain  │  │    JTBD      │                      │ │
│  │  │  Advantage   │  │    Agent     │  │    Agent     │                      │ │
│  │  └──────────────┘  └──────────────┘  └──────────────┘                      │ │
│  │                                                                             │ │
│  │  Phase 3 (Sequential, after Phase 2)                                       │ │
│  │  ┌──────────────┐                                                          │ │
│  │  │ Competitive  │                                                          │ │
│  │  │  Strategy    │                                                          │ │
│  │  └──────────────┘                                                          │ │
│  │                                                                             │ │
│  │  Phase 4 (Sequential, after Phase 3)                                       │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                      │ │
│  │  │    SWOT      │  │  Coherence   │  │  Narrative   │                      │ │
│  │  │  Synthesis   │  │   Checker    │  │  Synthesis   │                      │ │
│  │  └──────────────┘  └──────────────┘  └──────────────┘                      │ │
│  │                                                                             │ │
│  │  Phase 5 (Sequential, after Phase 4) ★ NEW                                 │ │
│  │  ┌─────────────────────────────────────────────────────────────────────┐   │ │
│  │  │                    Presentation Agent                                │   │ │
│  │  │  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐  │   │ │
│  │  │  │ Slide Structure │ →  │ Visual Generator│ →  │  PDF Assembler  │  │   │ │
│  │  │  │ (Gemini 2.5 Pro)│    │(Nano Banana Pro)│    │   (PyMuPDF)     │  │   │ │
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
│                                                                                   │
└───────────────────────────────────────────────────────────────────────────────────┘
```

---

## Part 2: Model Configuration

### 2.1 Model Selection

| Purpose | Model | Model ID | Context Window | Notes |
|---------|-------|----------|----------------|-------|
| **Analysis Agents** | Gemini 3 Pro | `gemini-3-pro-preview` | 1M tokens | Most advanced reasoning, agentic capabilities |
| **Presentation Structure** | Gemini 3 Pro | `gemini-3-pro-preview` | 1M tokens | Complex slide planning |
| **Visual Generation** | Nano Banana Pro | `gemini-3-pro-image-preview` | N/A | 4K infographics, charts, diagrams |

### 2.2 Model Configuration

```python
from google import genai
from google.genai import types

# Analysis model configuration (Gemini 3 Pro)
ANALYSIS_MODEL = "gemini-3-pro-preview"
ANALYSIS_CONFIG = types.GenerateContentConfig(
    temperature=0.7,
    max_output_tokens=8192,
    thinking_config=types.ThinkingConfig(
        thinking_level="HIGH"  # Enable extended thinking for complex reasoning
    )
)

# Image generation model configuration
IMAGE_MODEL = "gemini-3-pro-image-preview"
IMAGE_CONFIG = types.GenerateContentConfig(
    response_modalities=["IMAGE", "TEXT"],
    image_config=types.ImageConfig(
        image_size="2K",  # Balance quality vs cost (1K, 2K, 4K available)
        aspect_ratio="16:9"  # Presentation slide format
    )
)
```

---

## Part 3: Agent Specifications

### 3.1 Strategy Orchestrator (Root Agent)

**Type:** `SequentialAgent` (contains `ParallelAgent` sub-agents for concurrent phases)

**Responsibility:**
- Coordinate the 5-phase execution pipeline
- Inject foundation context into shared state before analysis begins
- Run cross-agent coherence checks after all analyses complete
- Synthesize final narrative that integrates all findings
- Trigger presentation generation in Phase 5

**Configuration:**
```python
root_agent = SequentialAgent(
    name="strategy_orchestrator",
    description="Orchestrates strategic analysis through 5 phases: context, market, firm, strategy, presentation",
    sub_agents=[
        context_setup_agent,      # Sets up foundation context
        phase1_parallel_agent,    # Macro + Market Structure (parallel)
        phase2_parallel_agent,    # Comparative Advantage + Value Chain + JTBD (parallel)
        phase3_competitive_agent, # Competitive Strategy (sequential)
        phase4_synthesis_agent,   # SWOT + Coherence + Narrative (sequential)
        phase5_presentation_agent, # Slide generation + PDF output (sequential)
    ]
)
```

**Coherence Checks (implemented in Phase 4):**
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
- **Model:** `gemini-3-pro-preview` for all analysis agents

#### 3.2.1 Macro Economy Agent

**Purpose:** Analyze business cycle, interest rates, FX, sector spending patterns

**Output Schema:**
```python
class MacroEconomyOutput(BaseModel):
    cycle_phase: Literal["expansion", "peak", "contraction", "trough"]
    rate_environment: str  # e.g., "rising", "stable", "falling"
    consumer_spending_outlook: str
    fx_impact: Optional[str]
    sector_trends: List[str]
    strategic_implications: List[str]
    confidence: ConfidenceMetadata
```

**Tools:** `search_economic_data`, `get_fed_indicators`, `search_sector_trends`

**Output Key:** `macro_economy_analysis`

---

#### 3.2.2 Market Structure Agent

**Type:** `ParallelAgent` (orchestrates Complements + Substitutes in parallel)

**Purpose:** Analyze ecosystem (complements) and competitive alternatives (substitutes)

**Sub-agents:**

##### Complements Analysis Agent
**Output Schema:**
```python
class ComplementsOutput(BaseModel):
    key_complements: List[ComplementItem]  # category, players, integration_depth, health
    ecosystem_health: Literal["strong", "moderate", "weak", "fragmented"]
    vertical_integration_risks: List[str]
    strategic_implications: List[str]
    confidence: ConfidenceMetadata

class ComplementItem(BaseModel):
    category: str
    key_players: List[str]
    integration_depth: Literal["critical", "deep", "moderate", "shallow"]
    relationship_trend: Literal["strengthening", "stable", "weakening", "adversarial"]
```

**Tools:** `search_partnerships`, `search_ecosystem`, `search_integrations`

**Output Key:** `complements_analysis`

##### Substitutes Analysis Agent
**Output Schema:**
```python
class SubstitutesOutput(BaseModel):
    direct_substitutes: List[SubstituteItem]
    indirect_substitutes: List[SubstituteItem]
    switching_costs: Literal["high", "moderate", "low"]
    substitution_triggers: List[str]
    defensibility_zones: List[str]
    strategic_implications: List[str]
    confidence: ConfidenceMetadata

class SubstituteItem(BaseModel):
    name: str
    type: Literal["direct", "indirect", "potential"]
    threat_level: Literal["high", "medium", "low"]
    switching_barrier: str
```

**Tools:** `search_competitors`, `search_alternatives`, `analyze_trends`

**Output Key:** `substitutes_analysis`

**Combined Output Key:** `market_structure_analysis`

---

#### 3.2.3 Comparative Advantage Agent

**Purpose:** Identify what the firm does that others cannot easily replicate

**Output Schema:**
```python
class ComparativeAdvantageOutput(BaseModel):
    primary_advantages: List[AdvantageItem]
    areas_of_parity: List[str]
    areas_of_disadvantage: List[str]
    durability_assessment: str
    strategic_implications: List[str]
    confidence: ConfidenceMetadata

class AdvantageItem(BaseModel):
    advantage: str
    source: Literal["scale", "network_effects", "ip", "brand", "capabilities", "data", "relationships"]
    durability: Literal["durable", "eroding", "temporary"]
    replicability: Literal["very_hard", "hard", "moderate", "easy"]
    monetization_status: Literal["fully_monetized", "partially_monetized", "untapped"]
```

**Tools:** `search_patents`, `search_capabilities`, `analyze_historical_investment`

**Output Key:** `comparative_advantage_analysis`

**Dependencies:** Reads `market_structure_analysis` from state

---

#### 3.2.4 Value Chain Agent

**Purpose:** Analyze activities and margin drivers

**Output Schema:**
```python
class ValueChainOutput(BaseModel):
    primary_activities: List[ActivityItem]
    support_activities: List[ActivityItem]
    margin_drivers: List[str]
    cost_centers: List[str]
    vertical_integration_level: Literal["high", "moderate", "low"]
    strategic_implications: List[str]
    confidence: ConfidenceMetadata

class ActivityItem(BaseModel):
    activity: str
    margin_contribution: Literal["high", "medium", "low", "negative"]
    strategic_importance: Literal["critical", "important", "supporting"]
    outsourced: bool
```

**Tools:** `analyze_cost_structure`, `search_segment_data`, `analyze_financials`

**Output Key:** `value_chain_analysis`

---

#### 3.2.5 JTBD Agent (Jobs To Be Done)

**Purpose:** Understand customer needs and hiring criteria

**Output Schema:**
```python
class JTBDOutput(BaseModel):
    jobs: List[JobItem]
    underserved_jobs: List[str]
    overserved_jobs: List[str]
    hiring_criteria: List[str]
    firing_triggers: List[str]
    opportunities: List[str]
    confidence: ConfidenceMetadata

class JobItem(BaseModel):
    job: str
    job_type: Literal["functional", "emotional", "social"]
    importance: Literal["critical", "important", "nice_to_have"]
    current_satisfaction: Literal["high", "medium", "low", "unmet"]
```

**Tools:** `search_customer_reviews`, `search_forums`, `analyze_surveys`

**Output Key:** `jtbd_analysis`

---

#### 3.2.6 Competitive Strategy Agent

**Purpose:** Determine cost leadership vs. differentiation positioning

**Output Schema:**
```python
class CompetitiveStrategyOutput(BaseModel):
    current_position: Literal["cost_leadership", "differentiation", "focus_cost", "focus_diff", "stuck_in_middle"]
    target_segment: str
    scope: Literal["broad", "narrow"]
    coherence_assessment: CoherenceAssessment
    recommended_position: Optional[str]
    strategic_moves: List[StrategicMove]
    confidence: ConfidenceMetadata

class CoherenceAssessment(BaseModel):
    score: float  # 0.0 to 1.0
    gaps: List[str]
    tensions: List[str]

class StrategicMove(BaseModel):
    move: str
    rationale: str
    risk: Literal["high", "medium", "low"]
    dependencies: List[str]
```

**Tools:** `analyze_pricing`, `analyze_brand_perception`, `search_positioning`

**Output Key:** `competitive_strategy_analysis`

**Dependencies:** Reads `comparative_advantage_analysis`, `value_chain_analysis` from state

---

#### 3.2.7 SWOT Synthesis Agent

**Purpose:** Integrate all findings into SWOT framework with cross-references

**Output Schema:**
```python
class SWOTOutput(BaseModel):
    strengths: List[SWOTItem]
    weaknesses: List[SWOTItem]
    opportunities: List[SWOTItem]
    threats: List[SWOTItem]
    key_tensions: List[TensionItem]
    strategic_priorities: List[str]
    confidence: ConfidenceMetadata

class SWOTItem(BaseModel):
    item: str
    source_agent: str  # Which agent contributed this insight
    importance: Literal["critical", "high", "medium", "low"]
    cross_references: List[str]

class TensionItem(BaseModel):
    tension: str
    between: Tuple[str, str]  # e.g., ("comparative_advantage", "competitive_strategy")
    severity: Literal["high", "medium", "low"]
    resolution_suggestion: Optional[str]
```

**Tools:** None (reads from other agents' outputs)

**Output Key:** `swot_synthesis`

**Dependencies:** Reads ALL other agent outputs from state

---

### 3.3 Presentation Agent (Phase 5) — NEW

**Purpose:** Generate a professional presentation deck from analysis findings

**Type:** `SequentialAgent` with three sub-components

**Architecture:**
```
Presentation Agent
├── Slide Structure Generator (LlmAgent - Gemini 3 Pro)
│   └── Plans slide sequence, content, and visual descriptions
├── Visual Generator (Custom Agent - Nano Banana Pro)
│   └── Creates infographics, charts, diagrams for each slide
└── PDF Assembler (Custom Agent - PyMuPDF)
    └── Composes final PDF with consistent styling
```

#### 3.3.1 Slide Structure Generator

**Model:** `gemini-3-pro-preview`

**Purpose:** Plan the presentation structure and content for each slide

**Output Schema:**
```python
class PresentationStructure(BaseModel):
    title: str
    subtitle: str
    total_slides: int
    slides: List[SlideSpec]
    color_scheme: ColorScheme
    metadata: PresentationMetadata

class SlideSpec(BaseModel):
    slide_number: int
    slide_type: Literal[
        "title", "executive_summary", "section_header",
        "key_findings", "framework", "data_visualization",
        "swot_matrix", "recommendations", "appendix"
    ]
    title: str
    content: SlideContent
    visual_description: str  # Detailed prompt for Nano Banana Pro
    speaker_notes: Optional[str]
    source_agents: List[str]  # Which agents contributed to this slide

class SlideContent(BaseModel):
    headline: str
    body_points: List[str]
    data_points: Optional[List[DataPoint]]
    quote: Optional[str]
    footer: Optional[str]

class DataPoint(BaseModel):
    label: str
    value: str
    trend: Optional[Literal["up", "down", "stable"]]
    color_hint: Optional[str]

class ColorScheme(BaseModel):
    primary: str      # Light blue theme
    secondary: str
    accent: str
    background: str
    text_primary: str
    text_secondary: str
```

**Instruction Template:**
```markdown
You are a presentation designer creating a strategy analysis deck.

## Context
Company: {company:name}
Strategic Question: {user:strategic_question}
User Role: {user:role}

## Available Analysis Data
- Macro Economy: {macro_economy_analysis}
- Market Structure: {market_structure_analysis}
- Comparative Advantage: {comparative_advantage_analysis}
- Value Chain: {value_chain_analysis}
- JTBD: {jtbd_analysis}
- Competitive Strategy: {competitive_strategy_analysis}
- SWOT: {swot_synthesis}
- Coherence Flags: {coherence_flags}
- Final Narrative: {final_narrative}

## Design Requirements
- Style: Modern, elegant, professional
- Color scheme: Light blue primary (#E3F2FD), dark blue accents (#1565C0)
- Clean whitespace, minimal text per slide
- Data visualizations for quantitative insights
- Framework diagrams for strategic concepts

## Slide Sequence
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

For each slide, provide:
- Clear title and headline
- 3-5 bullet points maximum
- Detailed visual description for image generation
- Speaker notes for context
```

**Output Key:** `presentation_structure`

---

#### 3.3.2 Visual Generator

**Model:** `gemini-3-pro-image-preview` (Nano Banana Pro)

**Purpose:** Generate infographics, charts, and diagrams for each slide

**Implementation:**
```python
class VisualGeneratorTool:
    """
    Generates slide visuals using Nano Banana Pro.
    """

    def __init__(self):
        from google import genai
        self.client = genai.Client()
        self.model = "gemini-3-pro-image-preview"

    def generate_slide_visual(
        self,
        visual_description: str,
        slide_type: str,
        color_scheme: ColorScheme,
        aspect_ratio: str = "16:9",
        resolution: str = "2K",
        tool_context: ToolContext
    ) -> dict:
        """
        Generate a visual for a presentation slide.

        Args:
            visual_description: Detailed description of what to generate
            slide_type: Type of slide (framework, data_viz, etc.)
            color_scheme: Brand colors to use
            aspect_ratio: Image aspect ratio (16:9 for slides)
            resolution: Output resolution (1K, 2K, 4K)
            tool_context: ADK tool context

        Returns:
            {
                "image_path": str,
                "image_bytes": bytes,
                "generation_metadata": dict
            }
        """
        # Build the prompt with style guidance
        styled_prompt = f"""
        Create a professional presentation slide visual.

        VISUAL DESCRIPTION:
        {visual_description}

        STYLE REQUIREMENTS:
        - Modern, elegant, clean design
        - Primary color: {color_scheme.primary} (light blue)
        - Accent color: {color_scheme.accent}
        - White/light background
        - Minimal, professional typography style
        - High contrast for readability
        - No text unless absolutely necessary (will be added separately)
        - Suitable for business/strategy presentation

        SLIDE TYPE: {slide_type}
        """

        from google.genai import types

        response = self.client.models.generate_content(
            model=self.model,
            contents=[styled_prompt],
            config=types.GenerateContentConfig(
                response_modalities=["IMAGE", "TEXT"],
                image_config=types.ImageConfig(
                    image_size=resolution,
                    aspect_ratio=aspect_ratio
                )
            )
        )

        # Extract image from response
        for part in response.parts:
            if part.inline_data is not None:
                image_bytes = part.inline_data.data
                image_path = self._save_image(image_bytes, tool_context)
                return {
                    "image_path": image_path,
                    "image_bytes": image_bytes,
                    "generation_metadata": {
                        "model": self.model,
                        "resolution": resolution,
                        "aspect_ratio": aspect_ratio
                    }
                }

        return {"error": "No image generated"}

    def generate_framework_diagram(
        self,
        framework_type: str,
        data: dict,
        color_scheme: ColorScheme,
        tool_context: ToolContext
    ) -> dict:
        """
        Generate specific framework diagrams (Porter's, SWOT, Value Chain, etc.)
        """
        framework_prompts = {
            "porter_matrix": self._build_porter_prompt(data, color_scheme),
            "swot_matrix": self._build_swot_prompt(data, color_scheme),
            "value_chain": self._build_value_chain_prompt(data, color_scheme),
            "competitive_position": self._build_position_prompt(data, color_scheme),
        }

        prompt = framework_prompts.get(framework_type)
        if not prompt:
            return {"error": f"Unknown framework type: {framework_type}"}

        return self.generate_slide_visual(
            visual_description=prompt,
            slide_type="framework",
            color_scheme=color_scheme,
            tool_context=tool_context
        )

    def _build_swot_prompt(self, data: dict, colors: ColorScheme) -> str:
        return f"""
        Create a 2x2 SWOT matrix visualization:

        TOP LEFT (Strengths - internal positive):
        {', '.join(data.get('strengths', [])[:3])}

        TOP RIGHT (Weaknesses - internal negative):
        {', '.join(data.get('weaknesses', [])[:3])}

        BOTTOM LEFT (Opportunities - external positive):
        {', '.join(data.get('opportunities', [])[:3])}

        BOTTOM RIGHT (Threats - external negative):
        {', '.join(data.get('threats', [])[:3])}

        Style: Clean quadrant layout, light blue accent ({colors.primary}),
        subtle icons for each quadrant, professional business style.
        """

    def _save_image(self, image_bytes: bytes, tool_context: ToolContext) -> str:
        """Save generated image and return path."""
        import uuid
        from pathlib import Path

        output_dir = Path(tool_context.state.get("output_dir", "./output"))
        output_dir.mkdir(parents=True, exist_ok=True)

        filename = f"slide_visual_{uuid.uuid4().hex[:8]}.png"
        filepath = output_dir / filename

        with open(filepath, "wb") as f:
            f.write(image_bytes)

        return str(filepath)
```

**Output Key:** `slide_visuals` (list of generated image paths)

---

#### 3.3.3 PDF Assembler (PyMuPDF)

**Purpose:** Assemble final PDF presentation with consistent styling

**Implementation:**
```python
import fitz  # PyMuPDF
from pathlib import Path
from typing import List, Optional
from PIL import Image
import io

class PDFGeneratorTool:
    """
    Generates professional PDF presentations using PyMuPDF.
    Implements the light blue, modern, elegant design system.
    """

    # Design system constants
    DESIGN = {
        # Colors (RGB tuples, 0-1 scale for PyMuPDF)
        "primary_blue": (0.133, 0.396, 0.753),      # #2265C0
        "light_blue": (0.890, 0.949, 0.992),        # #E3F2FD
        "accent_blue": (0.259, 0.522, 0.957),       # #4285F4
        "dark_text": (0.129, 0.129, 0.129),         # #212121
        "light_text": (0.459, 0.459, 0.459),        # #757575
        "white": (1, 1, 1),
        "subtle_gray": (0.961, 0.961, 0.961),       # #F5F5F5

        # Typography
        "font_title": "Helvetica-Bold",
        "font_heading": "Helvetica-Bold",
        "font_body": "Helvetica",
        "font_light": "Helvetica",

        # Sizes (points)
        "title_size": 44,
        "heading_size": 28,
        "subheading_size": 20,
        "body_size": 14,
        "caption_size": 10,

        # Layout (16:9 aspect ratio, 1920x1080 equivalent in points)
        "page_width": 841.89,   # A4 landscape width
        "page_height": 595.28,  # A4 landscape height
        "margin": 50,
        "content_margin": 80,
    }

    def __init__(self):
        self.doc = None

    def create_presentation(
        self,
        structure: PresentationStructure,
        visuals: List[dict],
        output_path: str,
        tool_context: ToolContext
    ) -> dict:
        """
        Create a complete PDF presentation.

        Args:
            structure: Slide structure from Gemini 2.5 Pro
            visuals: Generated visuals from Nano Banana Pro
            output_path: Where to save the PDF
            tool_context: ADK tool context

        Returns:
            {
                "pdf_path": str,
                "page_count": int,
                "file_size_kb": int
            }
        """
        self.doc = fitz.open()

        # Create each slide
        for i, slide in enumerate(structure.slides):
            visual = visuals[i] if i < len(visuals) else None
            self._create_slide(slide, visual, structure.color_scheme)

        # Save the document
        self.doc.save(output_path)
        file_size = Path(output_path).stat().st_size // 1024

        result = {
            "pdf_path": output_path,
            "page_count": len(self.doc),
            "file_size_kb": file_size
        }

        self.doc.close()
        return result

    def _create_slide(
        self,
        slide: SlideSpec,
        visual: Optional[dict],
        colors: ColorScheme
    ):
        """Create a single slide page."""
        # Add new page (16:9 landscape)
        page = self.doc.new_page(
            width=self.DESIGN["page_width"],
            height=self.DESIGN["page_height"]
        )

        # Draw background
        self._draw_background(page, slide.slide_type)

        # Draw content based on slide type
        if slide.slide_type == "title":
            self._draw_title_slide(page, slide)
        elif slide.slide_type == "section_header":
            self._draw_section_header(page, slide)
        elif slide.slide_type == "swot_matrix":
            self._draw_swot_slide(page, slide, visual)
        elif slide.slide_type == "data_visualization":
            self._draw_data_slide(page, slide, visual)
        else:
            self._draw_content_slide(page, slide, visual)

        # Add page number (except title slide)
        if slide.slide_type != "title":
            self._add_page_number(page, slide.slide_number)

    def _draw_background(self, page: fitz.Page, slide_type: str):
        """Draw slide background with subtle design elements."""
        rect = page.rect

        # White background
        page.draw_rect(rect, color=None, fill=self.DESIGN["white"])

        # Subtle accent bar at top
        accent_rect = fitz.Rect(0, 0, rect.width, 8)
        page.draw_rect(accent_rect, color=None, fill=self.DESIGN["primary_blue"])

        # Subtle bottom border
        bottom_rect = fitz.Rect(0, rect.height - 2, rect.width, rect.height)
        page.draw_rect(bottom_rect, color=None, fill=self.DESIGN["light_blue"])

    def _draw_title_slide(self, page: fitz.Page, slide: SlideSpec):
        """Draw the title slide with centered content."""
        rect = page.rect
        center_y = rect.height / 2

        # Main title
        title_point = fitz.Point(rect.width / 2, center_y - 40)
        page.insert_text(
            title_point,
            slide.title,
            fontsize=self.DESIGN["title_size"],
            fontname=self.DESIGN["font_title"],
            color=self.DESIGN["primary_blue"],
            render_mode=0,
        )

        # Subtitle
        if slide.content.headline:
            subtitle_point = fitz.Point(rect.width / 2, center_y + 20)
            page.insert_text(
                subtitle_point,
                slide.content.headline,
                fontsize=self.DESIGN["subheading_size"],
                fontname=self.DESIGN["font_light"],
                color=self.DESIGN["light_text"],
            )

        # Decorative line
        line_y = center_y - 60
        page.draw_line(
            fitz.Point(rect.width / 2 - 100, line_y),
            fitz.Point(rect.width / 2 + 100, line_y),
            color=self.DESIGN["accent_blue"],
            width=3
        )

    def _draw_content_slide(
        self,
        page: fitz.Page,
        slide: SlideSpec,
        visual: Optional[dict]
    ):
        """Draw a standard content slide."""
        rect = page.rect
        margin = self.DESIGN["content_margin"]

        # Title
        page.insert_text(
            fitz.Point(margin, margin + 30),
            slide.title,
            fontsize=self.DESIGN["heading_size"],
            fontname=self.DESIGN["font_heading"],
            color=self.DESIGN["primary_blue"],
        )

        # Headline
        if slide.content.headline:
            page.insert_text(
                fitz.Point(margin, margin + 60),
                slide.content.headline,
                fontsize=self.DESIGN["subheading_size"],
                fontname=self.DESIGN["font_body"],
                color=self.DESIGN["dark_text"],
            )

        # Content area - split for visual if present
        content_y = margin + 100

        if visual and visual.get("image_path"):
            # Two-column layout: text left, visual right
            text_width = rect.width * 0.45
            visual_x = rect.width * 0.52

            # Bullet points
            self._draw_bullet_points(
                page,
                slide.content.body_points,
                fitz.Point(margin, content_y),
                max_width=text_width - margin
            )

            # Visual
            self._insert_visual(
                page,
                visual["image_path"],
                fitz.Rect(visual_x, content_y, rect.width - margin, rect.height - 60)
            )
        else:
            # Full-width bullet points
            self._draw_bullet_points(
                page,
                slide.content.body_points,
                fitz.Point(margin, content_y),
                max_width=rect.width - 2 * margin
            )

    def _draw_bullet_points(
        self,
        page: fitz.Page,
        points: List[str],
        start: fitz.Point,
        max_width: float
    ):
        """Draw bullet points with proper spacing."""
        y = start.y
        line_height = 28
        bullet_indent = 15

        for point in points[:6]:  # Max 6 points per slide
            # Bullet character
            page.insert_text(
                fitz.Point(start.x, y),
                "•",
                fontsize=self.DESIGN["body_size"],
                color=self.DESIGN["accent_blue"],
            )

            # Text (with wrapping if needed)
            text_rect = fitz.Rect(
                start.x + bullet_indent,
                y - self.DESIGN["body_size"],
                start.x + max_width,
                y + line_height
            )

            # Insert text with automatic wrapping
            page.insert_textbox(
                text_rect,
                point,
                fontsize=self.DESIGN["body_size"],
                fontname=self.DESIGN["font_body"],
                color=self.DESIGN["dark_text"],
                align=fitz.TEXT_ALIGN_LEFT,
            )

            y += line_height

    def _draw_swot_slide(
        self,
        page: fitz.Page,
        slide: SlideSpec,
        visual: Optional[dict]
    ):
        """Draw SWOT matrix slide."""
        rect = page.rect
        margin = self.DESIGN["content_margin"]

        # Title
        page.insert_text(
            fitz.Point(margin, margin + 30),
            "SWOT Analysis",
            fontsize=self.DESIGN["heading_size"],
            fontname=self.DESIGN["font_heading"],
            color=self.DESIGN["primary_blue"],
        )

        # If we have a generated visual, use it
        if visual and visual.get("image_path"):
            self._insert_visual(
                page,
                visual["image_path"],
                fitz.Rect(margin, margin + 60, rect.width - margin, rect.height - 40)
            )
        else:
            # Draw simple 2x2 matrix
            self._draw_swot_matrix(page, slide.content, margin)

    def _insert_visual(self, page: fitz.Page, image_path: str, rect: fitz.Rect):
        """Insert an image into the page."""
        try:
            img = fitz.Pixmap(image_path)

            # Calculate scaling to fit within rect while maintaining aspect ratio
            img_ratio = img.width / img.height
            rect_ratio = rect.width / rect.height

            if img_ratio > rect_ratio:
                # Image is wider - fit to width
                new_width = rect.width
                new_height = rect.width / img_ratio
            else:
                # Image is taller - fit to height
                new_height = rect.height
                new_width = rect.height * img_ratio

            # Center the image in the rect
            x_offset = (rect.width - new_width) / 2
            y_offset = (rect.height - new_height) / 2

            img_rect = fitz.Rect(
                rect.x0 + x_offset,
                rect.y0 + y_offset,
                rect.x0 + x_offset + new_width,
                rect.y0 + y_offset + new_height
            )

            page.insert_image(img_rect, filename=image_path)

        except Exception as e:
            # Fallback: draw placeholder
            page.draw_rect(rect, color=self.DESIGN["light_blue"], fill=self.DESIGN["subtle_gray"])
            page.insert_text(
                fitz.Point(rect.x0 + 10, rect.y0 + 30),
                f"[Visual: {Path(image_path).name}]",
                fontsize=12,
                color=self.DESIGN["light_text"],
            )

    def _add_page_number(self, page: fitz.Page, number: int):
        """Add page number to bottom right."""
        rect = page.rect
        page.insert_text(
            fitz.Point(rect.width - 60, rect.height - 20),
            str(number),
            fontsize=self.DESIGN["caption_size"],
            fontname=self.DESIGN["font_light"],
            color=self.DESIGN["light_text"],
        )

    def generate_report_pdf(
        self,
        analysis_data: dict,
        output_path: str,
        tool_context: ToolContext
    ) -> dict:
        """
        Generate a detailed PDF report (not slides).
        More text-heavy, suitable for reading.
        """
        self.doc = fitz.open()

        # Cover page
        self._create_report_cover(analysis_data)

        # Table of contents
        self._create_toc(analysis_data)

        # Executive summary
        self._create_executive_summary(analysis_data)

        # Detailed sections for each framework
        sections = [
            ("Macroeconomic Context", analysis_data.get("macro_economy_analysis")),
            ("Market Structure", analysis_data.get("market_structure_analysis")),
            ("Comparative Advantage", analysis_data.get("comparative_advantage_analysis")),
            ("Value Chain Analysis", analysis_data.get("value_chain_analysis")),
            ("Customer Jobs to Be Done", analysis_data.get("jtbd_analysis")),
            ("Competitive Strategy", analysis_data.get("competitive_strategy_analysis")),
            ("SWOT Synthesis", analysis_data.get("swot_synthesis")),
        ]

        for title, content in sections:
            if content:
                self._create_report_section(title, content)

        # Appendix with methodology and sources
        self._create_appendix(analysis_data)

        self.doc.save(output_path)
        file_size = Path(output_path).stat().st_size // 1024

        result = {
            "pdf_path": output_path,
            "page_count": len(self.doc),
            "file_size_kb": file_size,
            "report_type": "detailed_analysis"
        }

        self.doc.close()
        return result
```

**Output Key:** `presentation_pdf`, `report_pdf`

---

## Part 4: Shared Memory Architecture

### 4.1 Foundation Context (Injected at Start)

All agents share access to foundation context stored in session state:

```python
# Foundation context structure (set before analysis begins)
foundation_context = {
    # User Context
    "user:role": "investor",  # investor | employee | competitor | student | advisor
    "user:strategic_question": "Does the services pivot make strategic sense?",
    "user:time_horizon": "3-5 years",
    "user:constraints": [],

    # Company Context
    "company:name": "Apple Inc.",
    "company:ticker": "AAPL",
    "company:industry": "Consumer Electronics / Technology",
    "company:sub_industry": "Hardware, Software, Services",
    "company:geography": "Global, HQ: USA",
    "company:size": "Large Cap",
    "company:stage": "Mature",

    # Analysis Metadata
    "analysis:id": "uuid-here",
    "analysis:started_at": "2025-01-15T10:30:00Z",
    "analysis:mode": "public_only",  # document_enriched | guided_context | public_only
    "analysis:documents": [],  # List of processed document references

    # Output Configuration
    "output:generate_slides": True,
    "output:generate_report": True,
    "output:dir": "./output/analysis_uuid",
}
```

### 4.2 State Key Naming Convention

```
# Foundation context (persists across invocations)
user:{key}              → User-level data
company:{key}           → Company being analyzed
analysis:{key}          → Analysis session metadata
output:{key}            → Output configuration

# Agent outputs (current session)
{agent_name}_analysis   → Agent's structured output
{agent_name}_raw        → Raw LLM response (for debugging)

# Presentation outputs
presentation_structure  → Slide structure from Gemini 2.5 Pro
slide_visuals          → List of generated image paths
presentation_pdf       → Final slides PDF path
report_pdf             → Detailed report PDF path

# Temporary data (current invocation only)
temp:{key}              → Intermediate calculations

# Cache data
cache:search:{hash}     → Cached search results
cache:document:{id}     → Cached document extractions
cache:visual:{hash}     → Cached generated visuals
```

### 4.3 State Flow Between Agents

```
Phase 1: Foundation Context → [Macro Agent, Market Structure Agent]
              ↓
         macro_economy_analysis
         complements_analysis
         substitutes_analysis
         market_structure_analysis
              ↓
Phase 2: + Phase 1 outputs → [Comp. Advantage, Value Chain, JTBD]
              ↓
         comparative_advantage_analysis
         value_chain_analysis
         jtbd_analysis
              ↓
Phase 3: + Phase 2 outputs → [Competitive Strategy]
              ↓
         competitive_strategy_analysis
              ↓
Phase 4: + All outputs → [SWOT Synthesis, Coherence Check, Narrative]
              ↓
         swot_synthesis
         coherence_flags
         final_narrative
              ↓
Phase 5: + All outputs → [Presentation Agent]
              ↓
         presentation_structure (Gemini 2.5 Pro)
         slide_visuals (Nano Banana Pro)
         presentation_pdf (PyMuPDF)
         report_pdf (PyMuPDF)
```

---

## Part 5: Tool Architecture

### 5.1 Market Intel Tool (Shared Service)

Central tool for all external data retrieval, with built-in caching.

```python
class MarketIntelTool:
    """
    Shared service for web search, news, and financial data.
    Implements session-level caching to avoid redundant API calls.
    """

    def search_web(
        self,
        query: str,
        source_type: Literal["news", "general", "financial", "academic"] = "general",
        recency: Literal["day", "week", "month", "year", "any"] = "month",
        tool_context: ToolContext
    ) -> dict:
        """
        Search the web for information.

        Args:
            query: Search query string
            source_type: Type of sources to prioritize
            recency: How recent the results should be
            tool_context: ADK tool context for state access

        Returns:
            {
                "results": [...],
                "sources": [...],
                "cached": bool,
                "confidence": float
            }
        """
        # Check cache first
        cache_key = f"cache:search:{hash(query + source_type + recency)}"
        cached = tool_context.state.get(cache_key)
        if cached:
            return {**cached, "cached": True}

        # Perform search
        results = self._execute_search(query, source_type, recency)

        # Cache results
        tool_context.state[cache_key] = results

        return {**results, "cached": False}

    def get_company_financials(
        self,
        ticker: str,
        metrics: List[str],
        tool_context: ToolContext
    ) -> dict:
        """Retrieve financial metrics for a company."""
        pass

    def search_sec_filings(
        self,
        company: str,
        filing_type: Literal["10-K", "10-Q", "8-K", "DEF14A"],
        tool_context: ToolContext
    ) -> dict:
        """Search SEC filings for a company."""
        pass

    def search_earnings_transcripts(
        self,
        company: str,
        quarters: int = 4,
        tool_context: ToolContext
    ) -> dict:
        """Retrieve recent earnings call transcripts."""
        pass
```

### 5.2 Document Processor Tool (Docling Integration)

```python
class DocumentProcessorTool:
    """
    PDF and document processing using Docling.
    Extracts structured content, tables, and sections.
    """

    def __init__(self):
        from docling.document_converter import DocumentConverter
        self.converter = DocumentConverter()

    def process_document(
        self,
        file_path: str,
        extract_tables: bool = True,
        classify_sections: bool = True,
        tool_context: ToolContext
    ) -> dict:
        """
        Process a document and extract structured content.

        Returns:
            {
                "document_id": str,
                "title": str,
                "sections": [...],
                "tables": [...],
                "metadata": {...}
            }
        """
        result = self.converter.convert(file_path)

        # Extract and classify sections
        sections = self._extract_sections(result.document)
        tables = self._extract_tables(result.document)

        # Cache the processed document
        doc_id = str(uuid.uuid4())
        tool_context.state[f"cache:document:{doc_id}"] = {
            "sections": sections,
            "tables": tables,
            "metadata": self._extract_metadata(result.document)
        }

        return {
            "document_id": doc_id,
            "sections": sections,
            "tables": tables
        }
```

### 5.3 PDF Generator Tool (PyMuPDF)

See Section 3.3.3 for full implementation.

### 5.4 Knowledge Base Tool

```python
class KnowledgeBaseTool:
    """
    Unified access to all knowledge sources with priority ranking.
    Implements the information supply chain from the design doc.
    """

    def query(
        self,
        query: str,
        source_priority: List[str] = ["documents", "user_context", "public_structured", "public_unstructured"],
        tool_context: ToolContext
    ) -> dict:
        """
        Query all knowledge sources with priority ranking.
        """
        pass

    def get_foundation_context(self, tool_context: ToolContext) -> dict:
        """Retrieve foundation context for current analysis."""
        return {
            "user": {
                "role": tool_context.state.get("user:role"),
                "question": tool_context.state.get("user:strategic_question"),
                "horizon": tool_context.state.get("user:time_horizon"),
            },
            "company": {
                "name": tool_context.state.get("company:name"),
                "industry": tool_context.state.get("company:industry"),
                "ticker": tool_context.state.get("company:ticker"),
            },
            "analysis": {
                "mode": tool_context.state.get("analysis:mode"),
                "documents": tool_context.state.get("analysis:documents", []),
            }
        }
```

---

## Part 6: Session & Caching Architecture

### 6.1 Database Schema

Using SQLite with `DatabaseSessionService` plus custom tables for caching and logging.

```sql
-- ADK-managed tables (created by DatabaseSessionService)
-- sessions, user_state, app_state, raw_events

-- Custom: Search Cache Table
CREATE TABLE search_cache (
    id TEXT PRIMARY KEY,
    query_hash TEXT NOT NULL,
    query TEXT NOT NULL,
    source_type TEXT NOT NULL,
    results TEXT NOT NULL,  -- JSON
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    expires_at TIMESTAMP NOT NULL,
    hit_count INTEGER DEFAULT 0
);

CREATE INDEX idx_search_cache_hash ON search_cache(query_hash);
CREATE INDEX idx_search_cache_expires ON search_cache(expires_at);

-- Custom: Document Cache Table
CREATE TABLE document_cache (
    id TEXT PRIMARY KEY,
    file_hash TEXT NOT NULL,
    file_name TEXT NOT NULL,
    extracted_content TEXT NOT NULL,  -- JSON
    sections TEXT NOT NULL,  -- JSON
    tables TEXT NOT NULL,  -- JSON
    metadata TEXT NOT NULL,  -- JSON
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_document_cache_hash ON document_cache(file_hash);

-- Custom: Visual Cache Table (NEW)
CREATE TABLE visual_cache (
    id TEXT PRIMARY KEY,
    prompt_hash TEXT NOT NULL,
    prompt TEXT NOT NULL,
    image_path TEXT NOT NULL,
    metadata TEXT NOT NULL,  -- JSON (model, resolution, etc.)
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_visual_cache_hash ON visual_cache(prompt_hash);

-- Custom: Analysis Sessions (extends ADK sessions)
CREATE TABLE analysis_sessions (
    id TEXT PRIMARY KEY,
    session_id TEXT NOT NULL,  -- FK to ADK sessions
    company_name TEXT NOT NULL,
    company_ticker TEXT,
    user_role TEXT NOT NULL,
    strategic_question TEXT,
    analysis_mode TEXT NOT NULL,
    status TEXT DEFAULT 'in_progress',
    started_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    completed_at TIMESTAMP,
    presentation_pdf_path TEXT,
    report_pdf_path TEXT,
    FOREIGN KEY (session_id) REFERENCES sessions(id)
);

-- Custom: Generated Outputs Table (NEW)
CREATE TABLE generated_outputs (
    id TEXT PRIMARY KEY,
    analysis_id TEXT NOT NULL,
    output_type TEXT NOT NULL,  -- 'presentation_pdf', 'report_pdf', 'slide_visual'
    file_path TEXT NOT NULL,
    file_size_kb INTEGER,
    metadata TEXT,  -- JSON
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (analysis_id) REFERENCES analysis_sessions(id)
);
```

### 6.2 Cache Strategy

```python
class CacheManager:
    """
    Manages session-level caching for search results, documents, and visuals.
    """

    def __init__(self, db_path: str):
        self.db_path = db_path
        self._init_db()

    def get_search_cache(self, query: str, source_type: str) -> Optional[dict]:
        """Retrieve cached search results if not expired."""
        query_hash = hashlib.sha256(f"{query}:{source_type}".encode()).hexdigest()

        with sqlite3.connect(self.db_path) as conn:
            cursor = conn.execute(
                """
                SELECT results FROM search_cache
                WHERE query_hash = ? AND expires_at > datetime('now')
                """,
                (query_hash,)
            )
            row = cursor.fetchone()
            if row:
                conn.execute(
                    "UPDATE search_cache SET hit_count = hit_count + 1 WHERE query_hash = ?",
                    (query_hash,)
                )
                return json.loads(row[0])
        return None

    def get_visual_cache(self, prompt: str) -> Optional[str]:
        """Retrieve cached visual image path."""
        prompt_hash = hashlib.sha256(prompt.encode()).hexdigest()

        with sqlite3.connect(self.db_path) as conn:
            cursor = conn.execute(
                "SELECT image_path FROM visual_cache WHERE prompt_hash = ?",
                (prompt_hash,)
            )
            row = cursor.fetchone()
            if row and Path(row[0]).exists():
                return row[0]
        return None

    def set_visual_cache(self, prompt: str, image_path: str, metadata: dict):
        """Cache a generated visual."""
        prompt_hash = hashlib.sha256(prompt.encode()).hexdigest()

        with sqlite3.connect(self.db_path) as conn:
            conn.execute(
                """
                INSERT OR REPLACE INTO visual_cache
                (id, prompt_hash, prompt, image_path, metadata)
                VALUES (?, ?, ?, ?, ?)
                """,
                (str(uuid.uuid4()), prompt_hash, prompt, image_path, json.dumps(metadata))
            )

    def cleanup_expired(self):
        """Remove expired cache entries."""
        with sqlite3.connect(self.db_path) as conn:
            conn.execute("DELETE FROM search_cache WHERE expires_at < datetime('now')")
```

---

## Part 7: Logging & Observability System

### 7.1 Log Levels & Categories

```python
from enum import Enum

class LogLevel(Enum):
    DEBUG = "debug"
    INFO = "info"
    WARNING = "warning"
    ERROR = "error"
    CRITICAL = "critical"

class LogCategory(Enum):
    # Agent lifecycle
    AGENT_START = "agent.start"
    AGENT_COMPLETE = "agent.complete"
    AGENT_ERROR = "agent.error"

    # LLM interactions
    LLM_REQUEST = "llm.request"
    LLM_RESPONSE = "llm.response"
    LLM_ERROR = "llm.error"
    LLM_TOKENS = "llm.tokens"

    # Tool executions
    TOOL_CALL = "tool.call"
    TOOL_RESULT = "tool.result"
    TOOL_ERROR = "tool.error"
    TOOL_CACHE_HIT = "tool.cache_hit"

    # State changes
    STATE_READ = "state.read"
    STATE_WRITE = "state.write"

    # Analysis flow
    PHASE_START = "phase.start"
    PHASE_COMPLETE = "phase.complete"
    COHERENCE_CHECK = "coherence.check"
    COHERENCE_FLAG = "coherence.flag"

    # Documents
    DOC_UPLOAD = "document.upload"
    DOC_PROCESS = "document.process"
    DOC_EXTRACT = "document.extract"

    # Presentation (NEW)
    PRESENTATION_STRUCTURE = "presentation.structure"
    VISUAL_GENERATE = "visual.generate"
    PDF_GENERATE = "pdf.generate"

    # Performance
    LATENCY = "performance.latency"
    MEMORY = "performance.memory"
```

### 7.2 Log Database Schema

```sql
-- Main log table
CREATE TABLE logs (
    id TEXT PRIMARY KEY,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    session_id TEXT NOT NULL,
    analysis_id TEXT NOT NULL,
    level TEXT NOT NULL,
    category TEXT NOT NULL,
    agent_name TEXT,
    message TEXT NOT NULL,
    data TEXT,  -- JSON blob for structured data
    duration_ms INTEGER,
    tokens_in INTEGER,
    tokens_out INTEGER,
    error_type TEXT,
    error_message TEXT,
    error_traceback TEXT
);

CREATE INDEX idx_logs_session ON logs(session_id);
CREATE INDEX idx_logs_analysis ON logs(analysis_id);
CREATE INDEX idx_logs_timestamp ON logs(timestamp);
CREATE INDEX idx_logs_category ON logs(category);
CREATE INDEX idx_logs_level ON logs(level);
CREATE INDEX idx_logs_agent ON logs(agent_name);

-- Agent execution summary (aggregated view)
CREATE TABLE agent_executions (
    id TEXT PRIMARY KEY,
    analysis_id TEXT NOT NULL,
    agent_name TEXT NOT NULL,
    started_at TIMESTAMP,
    completed_at TIMESTAMP,
    duration_ms INTEGER,
    status TEXT,  -- success, error, timeout
    tokens_total INTEGER,
    tool_calls_count INTEGER,
    cache_hits INTEGER,
    output_key TEXT,
    output_summary TEXT,  -- Truncated output for quick review
    error_message TEXT
);

CREATE INDEX idx_agent_exec_analysis ON agent_executions(analysis_id);

-- Coherence check results
CREATE TABLE coherence_checks (
    id TEXT PRIMARY KEY,
    analysis_id TEXT NOT NULL,
    check_type TEXT NOT NULL,
    condition_a TEXT NOT NULL,
    condition_b TEXT NOT NULL,
    flag_raised TEXT,
    severity TEXT,
    details TEXT,  -- JSON
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_coherence_analysis ON coherence_checks(analysis_id);
```

### 7.3 Logging Service Implementation

See original document — unchanged.

---

## Part 8: Frontend Design System

### 8.1 Design Philosophy

**Style:** Modern, elegant, professional
**Theme:** Light blue primary palette
**Approach:** Progressive disclosure with clean whitespace

### 8.2 Color Palette

```css
:root {
  /* Primary Blues */
  --color-primary-50: #E3F2FD;   /* Lightest - backgrounds */
  --color-primary-100: #BBDEFB;  /* Light - hover states */
  --color-primary-200: #90CAF9;  /* Light accent */
  --color-primary-300: #64B5F6;  /* Medium accent */
  --color-primary-400: #42A5F5;  /* Primary accent */
  --color-primary-500: #2196F3;  /* Primary */
  --color-primary-600: #1E88E5;  /* Primary dark */
  --color-primary-700: #1976D2;  /* Dark accent */
  --color-primary-800: #1565C0;  /* Darker - headers */
  --color-primary-900: #0D47A1;  /* Darkest - emphasis */

  /* Neutrals */
  --color-white: #FFFFFF;
  --color-gray-50: #FAFAFA;
  --color-gray-100: #F5F5F5;
  --color-gray-200: #EEEEEE;
  --color-gray-300: #E0E0E0;
  --color-gray-400: #BDBDBD;
  --color-gray-500: #9E9E9E;
  --color-gray-600: #757575;
  --color-gray-700: #616161;
  --color-gray-800: #424242;
  --color-gray-900: #212121;

  /* Semantic Colors */
  --color-success: #4CAF50;
  --color-warning: #FF9800;
  --color-error: #F44336;
  --color-info: #2196F3;

  /* Confidence Indicators */
  --confidence-high: #4CAF50;
  --confidence-medium: #FF9800;
  --confidence-low: #F44336;
}
```

### 8.3 Typography

```css
:root {
  /* Font Families */
  --font-primary: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  --font-mono: 'JetBrains Mono', 'Fira Code', monospace;

  /* Font Sizes */
  --text-xs: 0.75rem;    /* 12px */
  --text-sm: 0.875rem;   /* 14px */
  --text-base: 1rem;     /* 16px */
  --text-lg: 1.125rem;   /* 18px */
  --text-xl: 1.25rem;    /* 20px */
  --text-2xl: 1.5rem;    /* 24px */
  --text-3xl: 1.875rem;  /* 30px */
  --text-4xl: 2.25rem;   /* 36px */

  /* Font Weights */
  --font-light: 300;
  --font-regular: 400;
  --font-medium: 500;
  --font-semibold: 600;
  --font-bold: 700;

  /* Line Heights */
  --leading-tight: 1.25;
  --leading-normal: 1.5;
  --leading-relaxed: 1.75;
}
```

### 8.4 Component Styles

#### Cards (Analysis Artifacts)
```css
.artifact-card {
  background: var(--color-white);
  border: 1px solid var(--color-gray-200);
  border-radius: 12px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.08);
  padding: 24px;
  transition: all 0.2s ease;
}

.artifact-card:hover {
  border-color: var(--color-primary-300);
  box-shadow: 0 4px 12px rgba(33, 150, 243, 0.15);
}

.artifact-card.selected {
  border-color: var(--color-primary-500);
  box-shadow: 0 0 0 3px var(--color-primary-100);
}
```

#### Confidence Indicators
```css
.confidence-indicator {
  display: flex;
  gap: 4px;
  align-items: center;
}

.confidence-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: var(--color-gray-300);
}

.confidence-dot.filled {
  background: var(--color-primary-500);
}

.confidence-dot.high { background: var(--confidence-high); }
.confidence-dot.medium { background: var(--confidence-medium); }
.confidence-dot.low { background: var(--confidence-low); }
```

#### Progress Tracker
```css
.progress-tracker {
  background: var(--color-gray-50);
  border-radius: 8px;
  padding: 16px;
}

.progress-phase {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 8px 0;
}

.progress-phase.completed .phase-icon {
  background: var(--color-success);
  color: var(--color-white);
}

.progress-phase.in-progress .phase-icon {
  background: var(--color-primary-500);
  color: var(--color-white);
  animation: pulse 1.5s infinite;
}

.progress-phase.pending .phase-icon {
  background: var(--color-gray-200);
  color: var(--color-gray-500);
}
```

#### Chat Interface
```css
.chat-container {
  background: var(--color-white);
  border-radius: 16px;
  overflow: hidden;
}

.chat-message {
  padding: 16px 20px;
  max-width: 80%;
}

.chat-message.agent {
  background: var(--color-primary-50);
  border-radius: 16px 16px 16px 4px;
  margin-right: auto;
}

.chat-message.user {
  background: var(--color-gray-100);
  border-radius: 16px 16px 4px 16px;
  margin-left: auto;
}

.chat-input {
  border: 2px solid var(--color-gray-200);
  border-radius: 12px;
  padding: 12px 16px;
  transition: border-color 0.2s;
}

.chat-input:focus {
  border-color: var(--color-primary-400);
  outline: none;
  box-shadow: 0 0 0 3px var(--color-primary-100);
}
```

### 8.5 Layout Structure

```
┌─────────────────────────────────────────────────────────────────────┐
│  Header: "Strategy Agent" + [Export] [Save] [Share]                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  Artifact Navigation Bar (horizontal scroll)                 │    │
│  │  [MACRO ●●●●○] [MARKET ●●●●○] [COMP ADV ●●●○○] [VALUE ●●○○○] │    │
│  │  [JTBD ●●●○○] [STRATEGY ●●●●○] [SWOT ●●●○○]                  │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │                                                                 │ │
│  │                     Main Canvas                                 │ │
│  │                                                                 │ │
│  │  ┌─────────────────────────────────────────────────────────┐   │ │
│  │  │  Selected Artifact (expanded view)                       │   │ │
│  │  │                                                          │   │ │
│  │  │  COMPETITIVE STRATEGY                        [Edit] [↗]  │   │ │
│  │  │  ─────────────────────────────────────────────────────   │   │ │
│  │  │  Position: DIFFERENTIATION                               │   │ │
│  │  │  Scope: Broad (mass premium)                             │   │ │
│  │  │                                                          │   │ │
│  │  │  [Porter's Matrix Visualization]                         │   │ │
│  │  │                                                          │   │ │
│  │  │  Coherence Score: 0.72 ●●●●○                             │   │ │
│  │  │                                                          │   │ │
│  │  │  Tensions:                                               │   │ │
│  │  │  • Services vs ecosystem (cross-ref: Comp Adv)           │   │ │
│  │  │  • Developer conflict (cross-ref: Complements)           │   │ │
│  │  │                                                          │   │ │
│  │  └─────────────────────────────────────────────────────────┘   │ │
│  │                                                                 │ │
│  └────────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │  Contextual Chat                                                │ │
│  │  ─────────────────────────────────────────────────────────────  │ │
│  │  Context: Competitive Strategy  [Switch to: Global] [SWOT]      │ │
│  │                                                                 │ │
│  │  Agent: "The services pivot creates tension with the           │ │
│  │          differentiation strategy..."                          │ │
│  │                                                                 │ │
│  │  [Type your question...]                              [Send →]  │ │
│  └────────────────────────────────────────────────────────────────┘ │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 8.6 Animation & Transitions

```css
/* Smooth transitions */
* {
  transition-property: background-color, border-color, box-shadow, transform, opacity;
  transition-duration: 0.2s;
  transition-timing-function: ease-out;
}

/* Loading pulse animation */
@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}

/* Slide in animation */
@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.artifact-card {
  animation: slideIn 0.3s ease-out;
}

/* Progress bar animation */
@keyframes progressFill {
  from { width: 0%; }
  to { width: var(--progress); }
}
```

---

## Part 9: Project Structure

```
strategy_analysis_agent/
├── pyproject.toml                 # Project configuration
├── README.md                      # Project documentation
├── ARCHITECTURE.md                # This document
│
├── src/
│   └── strategy_agent/
│       ├── __init__.py
│       │
│       ├── agents/                # Agent definitions
│       │   ├── __init__.py
│       │   ├── orchestrator.py    # Root SequentialAgent
│       │   ├── macro_economy.py   # Macro Economy LlmAgent
│       │   ├── market_structure.py # Market Structure ParallelAgent
│       │   ├── complements.py     # Complements LlmAgent
│       │   ├── substitutes.py     # Substitutes LlmAgent
│       │   ├── comparative_advantage.py
│       │   ├── value_chain.py
│       │   ├── jtbd.py
│       │   ├── competitive_strategy.py
│       │   ├── swot.py
│       │   └── presentation.py    # NEW: Presentation Agent
│       │
│       ├── tools/                 # Tool definitions
│       │   ├── __init__.py
│       │   ├── market_intel.py    # Web search, news, financials
│       │   ├── document_processor.py  # Docling integration
│       │   ├── knowledge_base.py  # Unified knowledge access
│       │   ├── visual_generator.py    # NEW: Nano Banana Pro
│       │   ├── pdf_generator.py       # NEW: PyMuPDF
│       │   └── analysis_tools.py  # Domain-specific analysis tools
│       │
│       ├── schemas/               # Pydantic models
│       │   ├── __init__.py
│       │   ├── foundation.py      # Foundation context models
│       │   ├── outputs.py         # Agent output models
│       │   ├── presentation.py    # NEW: Slide/PDF models
│       │   └── common.py          # Shared types (ConfidenceMetadata, etc.)
│       │
│       ├── services/              # Infrastructure services
│       │   ├── __init__.py
│       │   ├── session.py         # Session management extensions
│       │   ├── cache.py           # CacheManager
│       │   ├── logging.py         # LoggingService
│       │   └── database.py        # Database initialization
│       │
│       ├── coherence/             # Coherence checking logic
│       │   ├── __init__.py
│       │   ├── checker.py         # CoherenceChecker class
│       │   └── rules.py           # Coherence rules definitions
│       │
│       ├── callbacks/             # ADK callbacks
│       │   ├── __init__.py
│       │   └── logging_callbacks.py
│       │
│       └── main.py                # Entry point
│
├── frontend/                      # NEW: Frontend application
│   ├── package.json
│   ├── next.config.js
│   ├── tailwind.config.js
│   ├── src/
│   │   ├── app/
│   │   │   ├── layout.tsx
│   │   │   ├── page.tsx
│   │   │   └── analysis/[id]/page.tsx
│   │   ├── components/
│   │   │   ├── ArtifactCard.tsx
│   │   │   ├── ArtifactNavBar.tsx
│   │   │   ├── ChatPanel.tsx
│   │   │   ├── ConfidenceIndicator.tsx
│   │   │   ├── MainCanvas.tsx
│   │   │   ├── ProgressTracker.tsx
│   │   │   └── ui/               # Shadcn/ui components
│   │   ├── lib/
│   │   │   ├── api.ts
│   │   │   └── utils.ts
│   │   └── styles/
│   │       └── globals.css
│   └── public/
│
├── tests/
│   ├── __init__.py
│   ├── conftest.py                # Pytest fixtures
│   ├── test_agents/
│   ├── test_tools/
│   ├── test_coherence/
│   └── test_integration/
│
├── data/
│   ├── prompts/                   # Agent instruction templates
│   │   ├── macro_economy.md
│   │   ├── complements.md
│   │   ├── presentation_structure.md  # NEW
│   │   └── ...
│   └── examples/                  # Example documents for testing
│
└── scripts/
    ├── init_db.py                 # Database initialization
    ├── run_analysis.py            # CLI for running analysis
    └── debug_analysis.py          # CLI for debugging past analyses
```

---

## Part 10: Configuration

### 10.1 Environment Variables

```bash
# Required
GOOGLE_API_KEY=your-gemini-api-key

# Optional - defaults shown
STRATEGY_AGENT_DB_PATH=./data/strategy_agent.db
STRATEGY_AGENT_LOG_LEVEL=INFO
STRATEGY_AGENT_CACHE_TTL_HOURS=24
STRATEGY_AGENT_MODEL=gemini-3-pro-preview
STRATEGY_AGENT_IMAGE_MODEL=gemini-3-pro-image-preview

# Search API (for Market Intel)
TAVILY_API_KEY=your-tavily-key  # or use Google Search
GOOGLE_SEARCH_API_KEY=your-search-key
GOOGLE_SEARCH_CX=your-custom-search-id
```

### 10.2 Application Configuration

```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    # Database
    db_path: str = "./data/strategy_agent.db"

    # LLM
    model: str = "gemini-3-pro-preview"
    image_model: str = "gemini-3-pro-image-preview"
    temperature: float = 0.7
    max_tokens: int = 8192

    # Image Generation
    image_resolution: str = "2K"  # 1K, 2K, 4K
    image_aspect_ratio: str = "16:9"

    # Caching
    cache_ttl_hours: int = 24
    cache_max_entries: int = 1000

    # Logging
    log_level: str = "INFO"
    log_llm_prompts: bool = True
    log_llm_responses: bool = True

    # Timeouts
    agent_timeout_seconds: int = 300
    tool_timeout_seconds: int = 60
    image_generation_timeout: int = 120

    # Output
    output_dir: str = "./output"
    generate_slides: bool = True
    generate_report: bool = True

    class Config:
        env_prefix = "STRATEGY_AGENT_"
```

---

## Part 11: Execution Flow

### 11.1 Analysis Lifecycle

```
1. INITIALIZATION
   ├── Create/resume session (DatabaseSessionService)
   ├── Set foundation context in state
   ├── Initialize logging
   └── Validate inputs

2. CONTEXT GATHERING (if no documents)
   ├── Guided questions to user
   ├── Store responses in state
   └── Determine analysis mode

3. DOCUMENT PROCESSING (if documents provided)
   ├── Process PDFs with Docling
   ├── Extract sections and tables
   ├── Cache extracted content
   └── Update analysis mode

4. PHASE 1: EXTERNAL CONTEXT (Parallel)
   ├── Macro Economy Agent → state["macro_economy_analysis"]
   └── Market Structure Agent (Parallel)
       ├── Complements Agent → state["complements_analysis"]
       └── Substitutes Agent → state["substitutes_analysis"]

5. PHASE 2: FIRM ANALYSIS (Parallel)
   ├── Comparative Advantage Agent → state["comparative_advantage_analysis"]
   ├── Value Chain Agent → state["value_chain_analysis"]
   └── JTBD Agent → state["jtbd_analysis"]

6. PHASE 3: STRATEGY (Sequential)
   └── Competitive Strategy Agent → state["competitive_strategy_analysis"]

7. PHASE 4: SYNTHESIS (Sequential)
   ├── SWOT Agent → state["swot_synthesis"]
   ├── Coherence Checker → state["coherence_flags"]
   └── Narrative Synthesis → state["final_narrative"]

8. PHASE 5: PRESENTATION (Sequential) ★ NEW
   ├── Slide Structure Generator (Gemini 3 Pro)
   │   └── → state["presentation_structure"]
   ├── Visual Generator (Nano Banana Pro)
   │   └── → state["slide_visuals"]
   └── PDF Assembler (PyMuPDF)
       ├── → state["presentation_pdf"]
       └── → state["report_pdf"]

9. COMPLETION
   ├── Store final outputs
   ├── Generate summary
   ├── Log completion metrics
   └── Return results with PDF paths
```

---

## Part 12: Testing Strategy

### 12.1 Test Categories

1. **Unit Tests**
   - Individual agent output validation
   - Tool function correctness
   - Schema validation
   - Coherence rule logic
   - PDF generation correctness

2. **Integration Tests**
   - Agent-to-agent state passing
   - Tool caching behavior
   - Database persistence
   - Logging completeness
   - Visual generation pipeline

3. **End-to-End Tests**
   - Full analysis pipeline with mock data
   - Full analysis with real API calls (marked slow)
   - Complete presentation generation

4. **Evaluation Tests**
   - Output quality assessment
   - Coherence detection accuracy
   - Consistency across runs
   - Visual quality assessment

### 12.2 Test Fixtures

```python
# conftest.py

@pytest.fixture
def mock_session_service():
    """In-memory session service for tests."""
    return InMemorySessionService()

@pytest.fixture
def mock_foundation_context():
    """Standard foundation context for tests."""
    return {
        "user:role": "student",
        "user:strategic_question": "Is the services pivot strategically sound?",
        "company:name": "Apple Inc.",
        "company:ticker": "AAPL",
        "company:industry": "Consumer Electronics",
        "analysis:mode": "public_only",
        "output:generate_slides": True,
        "output:generate_report": True,
    }

@pytest.fixture
def mock_analysis_outputs():
    """Complete mock analysis outputs for presentation testing."""
    return {
        "macro_economy_analysis": {...},
        "market_structure_analysis": {...},
        # ... all other outputs
    }
```

---

## Part 13: Future Considerations

### 13.1 Not in MVP (Deferred)

- Real-time streaming updates
- Multi-user support
- Strategy Map integration
- Document upload workflow UI
- Collaborative editing

### 13.2 Scaling Considerations

- Move from SQLite to PostgreSQL for production
- Add Redis for distributed caching
- Implement rate limiting for external APIs
- Add request queuing for high load
- CDN for generated PDFs and images

### 13.3 Evaluation Framework

- Define golden test cases with expected outputs
- Implement LLM-as-judge for output quality
- Track coherence detection precision/recall
- Monitor token usage and latency trends
- Visual quality scoring for generated slides

---

## Appendix A: Agent Instruction Templates

See `/data/prompts/` for full instruction templates. Each includes:

1. Role definition
2. Framework explanation
3. Analysis methodology
4. Output format requirements
5. Examples of good analysis
6. Common pitfalls to avoid

---

## Appendix B: Confidence Scoring Guidelines

```python
class ConfidenceMetadata(BaseModel):
    overall: float  # 0.0 to 1.0
    limiting_factors: List[str]
    high_confidence_claims: List[str]
    low_confidence_claims: List[str]
    would_improve_with: List[str]

# Scoring heuristics:
# 1.0: Direct user documents + verification
# 0.8: User verbal context + public confirmation
# 0.6: Public structured data (SEC, financials)
# 0.4: Public unstructured (news, articles)
# 0.2: Inference from limited data
```

---

## Appendix C: Design System Reference

### Color Usage Guidelines

| Use Case | Color | CSS Variable |
|----------|-------|--------------|
| Primary actions | Blue 600 | `--color-primary-600` |
| Card backgrounds | White | `--color-white` |
| Page backgrounds | Gray 50 | `--color-gray-50` |
| Agent messages | Blue 50 | `--color-primary-50` |
| User messages | Gray 100 | `--color-gray-100` |
| Headings | Gray 900 | `--color-gray-900` |
| Body text | Gray 700 | `--color-gray-700` |
| Subtle text | Gray 500 | `--color-gray-500` |
| Borders | Gray 200 | `--color-gray-200` |
| Hover states | Blue 100 | `--color-primary-100` |
| Selected states | Blue 500 | `--color-primary-500` |

---

*Document Version: 2.0*
*Last Updated: 2025-01-15*
*Changes: Added Gemini 3 Pro, Phase 5 Presentation Agent, Nano Banana Pro, PyMuPDF, Frontend Design System*
