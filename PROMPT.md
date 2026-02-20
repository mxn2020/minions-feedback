**MINIONS FEEDBACK — IMPLEMENTATION SPEC**

You are tasked with creating the complete initial foundation for `minions-feedback` — a structured feedback and survey system that enables systematic collection, analysis, and synthesis of user feedback. This is part of the Minions ecosystem, a universal structured object system designed for building AI-native tools.

---

**PROJECT OVERVIEW**

`minions-feedback` provides structured survey creation, response collection, theme detection, and sentiment analysis. It allows developers and agents to collect feedback systematically, identify emerging patterns, and prioritize product improvements based on real user input.

The core concept: feedback should be structured, analyzable, and actionable. Agents can synthesize responses, detect recurring themes, analyze sentiment, and generate prioritized recommendations that feed directly into product roadmaps. This is product intelligence infrastructure for AI agents.

---

**CONCEPT OVERVIEW**

This project is built on the Minions SDK (`minions-sdk`), which provides the foundational primitives: Minion (structured object instance), Minion Type (schema), and Relation (typed link between minions).

Surveys contain questions that collect responses. Agents analyze responses to extract themes and calculate sentiment scores. The system links directly with `minions-roadmap` to turn feedback into actionable product initiatives.

The system supports both TypeScript and Python SDKs with cross-language interoperability (both serialize to the same JSON format). All documentation includes dual-language code examples with tabbed interfaces.

---

**CORE PRIMITIVES**

This project defines the following Minion Types:

- `survey` — A structured feedback collection instrument with questions and targeting
- `question` — A single survey question with type (text, rating, multiple choice, etc.)
- `response` — An individual survey response with answers mapped to questions
- `theme` — An emergent pattern or topic extracted from response analysis
- `sentiment-score` — Quantified sentiment analysis for responses or themes

---

**MINIONS SDK REFERENCE — REQUIRED DEPENDENCY**

This project depends on `minions-sdk`, a published package that provides the foundational primitives. The GH Agent building this project MUST install it from the public registries and use the APIs documented below — do NOT reimplement minions primitives from scratch.

**Installation:**
```bash
# TypeScript (npm)
npm install minions-sdk
# or: pnpm add minions-sdk

# Python (PyPI) — package name is minions-sdk, but you import as "minions"
pip install minions-sdk
```

**TypeScript SDK — Core Imports:**
```typescript
import {
  // Core types
  type Minion, type MinionType, type Relation,
  type FieldDefinition, type FieldValidation, type FieldType,
  type CreateMinionInput, type UpdateMinionInput, type CreateRelationInput,
  type MinionStatus, type MinionPriority, type RelationType,
  type ExecutionResult, type Executable,
  type ValidationError, type ValidationResult,

  // Validation
  validateField, validateFields,

  // Built-in Schemas (10 MinionType instances — reuse where applicable)
  noteType, linkType, fileType, contactType,
  agentType, teamType, thoughtType, promptTemplateType, testCaseType, taskType,
  builtinTypes,

  // Registry — stores and retrieves MinionTypes by id or slug
  TypeRegistry,

  // Relations — in-memory directed graph with traversal utilities
  RelationGraph,

  // Lifecycle — CRUD operations with validation
  createMinion, updateMinion, softDelete, hardDelete, restoreMinion,

  // Evolution — migrate minions when schemas change (preserves removed fields in _legacy)
  migrateMinion,

  // Utilities
  generateId, now, SPEC_VERSION,
} from 'minions-sdk';
```

**Python SDK — Core Imports:**
```python
from minions import (
    # Types
    Minion, MinionType, Relation, FieldDefinition, FieldValidation,
    CreateMinionInput, UpdateMinionInput, CreateRelationInput,
    ExecutionResult, Executable, ValidationError, ValidationResult,
    # Validation
    validate_field, validate_fields,
    # Built-in Schemas (10 types)
    note_type, link_type, file_type, contact_type,
    agent_type, team_type, thought_type, prompt_template_type,
    test_case_type, task_type, builtin_types,
    # Registry
    TypeRegistry,
    # Relations
    RelationGraph,
    # Lifecycle
    create_minion, update_minion, soft_delete, hard_delete, restore_minion,
    # Evolution
    migrate_minion,
    # Utilities
    generate_id, now, SPEC_VERSION,
)
```

**Key Concepts:**
- A `MinionType` defines a schema (list of `FieldDefinition`s) — each field has `name`, `type`, `label`, `required`, `defaultValue`, `options`, `validation`
- A `Minion` is an instance with `id`, `title`, `minionTypeId`, `fields` (dict), `status`, `tags`, timestamps
- A `Relation` is a typed directional link (12 types: `parent_of`, `depends_on`, `implements`, `relates_to`, `inspired_by`, `triggers`, `references`, `blocks`, `alternative_to`, `part_of`, `follows`, `integration_link`)
- Field types: `string`, `number`, `boolean`, `date`, `select`, `multi-select`, `url`, `email`, `textarea`, `tags`, `json`, `array`
- `TypeRegistry` auto-loads 10 built-in types; register custom types with `registry.register(myType)`
- `createMinion(input, type)` validates fields against the schema and returns `{ minion, validation }` (TS) or `(minion, validation)` tuple (Python)
- Both SDKs serialize to identical camelCase JSON; Python provides `to_dict()` / `from_dict()` for conversion

**IMPORTANT:** Do NOT recreate these primitives. Import them from `minions-sdk` (npm) / `minions` (PyPI). Build your domain-specific types and utilities ON TOP of the SDK.

---

**WHAT YOU NEED TO CREATE**

**1. THE SPECIFICATION** (`/spec`)

Write a complete markdown specification document covering:

- Motivation and goals — why structured feedback matters for product development
- Glossary of terms specific to feedback analysis (sentiment, theme, NPS, etc.)
- Core type definitions for all five minion types with full field schemas
- Question type system — text, rating scale, multiple choice, NPS, CSAT, CES
- Response aggregation semantics — how responses roll up to surveys
- Theme extraction algorithm — detecting recurring patterns across responses
- Sentiment analysis model — scoring methodology and ranges
- Integration with `minions-roadmap` — feedback-driven prioritization
- Best practices for survey design and response analysis
- Conformance checklist for implementations

**2. THE CORE LIBRARY** (`/packages/core`)

A framework-agnostic TypeScript library built on `minions-sdk`. Must include:
- **Unified Client Architecture**:
  - A standalone `MinionsFeedback` client class that wraps all primitives and utilities in a unified facade.
  - A `FeedbackPlugin` class that implements `MinionPlugin` for mounting onto the core `Minions` client (e.g. `minions.feedback`).
  - Both modular (direct imports) and centralized (client instance) usage must be supported.

- Full TypeScript type definitions for all feedback-specific types
- `SurveyBuilder` class — fluent API for survey creation
  - `addQuestion(question)` — append question to survey
  - `setAudience(criteria)` — define target respondents
  - `schedule(startDate, endDate)` — set survey window
  - `build()` — create survey minion with linked questions
- `ResponseCollector` class — response capture and validation
  - `submit(surveyId, answers)` — create response minion
  - `validate(response)` — check answer completeness and formats
  - `aggregateStats(surveyId)` — response counts, completion rates
- `ThemeExtractor` class — pattern detection across responses
  - `extractThemes(surveyId)` — analyze all responses for recurring topics
  - `cluster(responses, k)` — group similar responses
  - `keywords(themeId)` — identify defining terms for theme
  - `prevalence(themeId)` — calculate theme occurrence rate
- `SentimentAnalyzer` class — emotion and tone scoring
  - `analyzeResponse(responseId)` — score individual response
  - `analyzeSurvey(surveyId)` — aggregate sentiment across survey
  - `trend(surveyIds[])` — track sentiment changes over time
  - Support for multiple sentiment dimensions (positive, negative, neutral, intensity)
- `FeedbackSynthesizer` class — actionable insight generation
  - `synthesize(surveyId)` — generate summary with themes, sentiment, quotes
  - `prioritize(themes[])` — rank themes by impact and prevalence
  - `recommendations(surveyId)` — suggest product actions based on feedback
- Clean public API with comprehensive JSDoc documentation
- Zero storage opinions — works with any backend

**3. THE PYTHON SDK** (`/packages/python`)

A complete Python port of the core library with identical functionality:
- **Unified Client Architecture**:
  - `MinionsFeedback` standalone client class.
  - `FeedbackPlugin` class for mounting onto the core `Minions` client.

- Python type hints for all classes and methods
- `SurveyBuilder`, `ResponseCollector`, `ThemeExtractor`, `SentimentAnalyzer`, `FeedbackSynthesizer` classes
- Same method signatures as TypeScript version (following Python naming conventions)
- Serializes to identical JSON format as TypeScript SDK (cross-language interoperability)
- Full docstrings compatible with Sphinx documentation generation
- Published to PyPI as `minions-feedback`

**4. THE CLI** (`/packages/cli`)

A command-line tool called `feedback` that provides:

```bash
feedback survey new "Post-launch survey"
# Interactively create a new survey with questions

feedback question add <survey-id> --type rating --text "How likely are you to recommend us?"
# Add question to survey

feedback survey publish <id>
# Make survey available for responses

feedback response submit <survey-id> --answers answers.json
# Submit a response (for testing/import)

feedback analyze <survey-id>
# Run full analysis: themes, sentiment, synthesis

feedback themes <survey-id>
# Extract and display recurring themes

feedback sentiment <survey-id>
# Show sentiment breakdown and trends

feedback synthesis <survey-id>
# Generate actionable insights report

feedback export <survey-id> --format csv
# Export responses to CSV or JSON

feedback import <file> --format typeform
# Import responses from external survey tools
```

Additional features:
- Interactive survey builder with question templates
- Colored output for sentiment (green=positive, red=negative, gray=neutral)
- ASCII charts for response distributions and sentiment trends
- JSON output mode for programmatic usage
- Config file support (`.feedbackrc.json`) for default settings

**5. THE DOCUMENTATION SITE** (`/apps/docs`)

Built with Astro Starlight. Must include:

- Landing page — "Structured feedback analysis" positioning, emphasize agent synthesis
- Getting started guide with both TypeScript and Python examples
- Core concepts:
  - Survey design principles
  - Question types and their use cases
  - Theme extraction methodology
  - Sentiment analysis model
  - Response aggregation and statistics
- API reference for both TypeScript and Python
  - Dual-language code tabs for all examples
  - Auto-generated from JSDoc/docstrings where possible
- Guides:
  - Designing effective surveys
  - Analyzing qualitative feedback at scale
  - Integrating feedback into product roadmaps
  - Building agents that synthesize user insights
  - Setting up feedback loops and continuous listening
  - Importing from Typeform, Google Forms, SurveyMonkey
- CLI reference with example commands
- Integration examples:
  - Email survey distribution
  - In-app feedback widgets
  - Slack feedback bots
  - Roadmap integration via `minions-roadmap`
  - Agent-driven feedback synthesis workflow
- Best practices for response rates and bias reduction
- Contributing guide

**6. OPTIONAL: THE WEB APP** (`/apps/web`)

A visual feedback interface (optional but recommended):

- Survey builder with drag-and-drop question ordering
- Response viewer with filtering and search
- Theme visualization (word clouds, cluster maps)
- Sentiment dashboard with trend charts
- Synthesis report generator
- Export interface for multiple formats
- Built with Next.js or SvelteKit (your choice based on ecosystem consistency)

---

**PROJECT STRUCTURE**

Standard Minions ecosystem monorepo structure:

```
minions-feedback/
  packages/
    core/                 # TypeScript core library
      src/
        types.ts          # Type definitions
        SurveyBuilder.ts
        ResponseCollector.ts
        ThemeExtractor.ts
        SentimentAnalyzer.ts
        FeedbackSynthesizer.ts
        index.ts          # Public API surface
      test/
      package.json
    python/               # Python SDK
      minions_feedback/
        __init__.py
        types.py
        survey_builder.py
        response_collector.py
        theme_extractor.py
        sentiment_analyzer.py
        feedback_synthesizer.py
      tests/
      pyproject.toml
    cli/                  # CLI tool
      src/
        commands/
          survey.ts
          question.ts
          response.ts
          analyze.ts
          themes.ts
          sentiment.ts
          synthesis.ts
          export.ts
          import.ts
        index.ts
      package.json
  apps/
    docs/                 # Astro Starlight documentation
      src/
        content/
          docs/
            index.md
            getting-started.md
            concepts/
            guides/
            api/
              typescript/
              python/
            cli/
      astro.config.mjs
      package.json
    web/                  # Optional feedback interface
      src/
      package.json
  spec/
    v0.1.md              # Full specification
  examples/
    typescript/
      simple-survey.ts
      theme-extraction.ts
      sentiment-analysis.ts
      roadmap-integration.ts
    python/
      simple_survey.py
      theme_extraction.py
      sentiment_analysis.py
      roadmap_integration.py
  .github/
    workflows/
      ci.yml             # Lint, test, build for both TS and Python
      publish.yml        # Publish to npm and PyPI
  README.md
  LICENSE                # AGPL-3.0
  package.json           # Workspace root
```

---

**BEYOND STANDARD PATTERN**

These utilities and classes are specific to `@minions-feedback/sdk`:

**SurveyBuilder**
- Fluent API for constructing surveys with validation
- Question templates for common patterns (NPS, CSAT, CES)
- Audience targeting criteria (user segment, behavior triggers)
- Survey scheduling and automatic open/close

**ResponseCollector**
- Validation of answer formats against question types
- Partial response support (save progress)
- Response deduplication and fraud detection
- Real-time aggregation statistics (completion rate, response count)

**ThemeExtractor**
- Natural language processing for open-text responses
- Clustering algorithms (k-means, hierarchical) for grouping similar responses
- Keyword extraction using TF-IDF or similar techniques
- Theme prevalence calculation (% of responses mentioning theme)
- Confidence scoring for detected themes

**SentimentAnalyzer**
- Multi-dimensional sentiment scoring (positive, negative, neutral, intensity)
- Question-level and survey-level aggregation
- Trend analysis across multiple survey iterations
- Sentiment distribution visualization data
- Support for both rule-based and ML-based sentiment models

**FeedbackSynthesizer**
- Agent-friendly synthesis of key insights from responses
- Theme prioritization based on prevalence and sentiment
- Representative quote selection for each theme
- Actionable recommendation generation
- Integration hooks for `minions-roadmap` initiative creation

---

**CLI COMMANDS**

All commands with detailed specifications:

**`feedback survey new <title>`**
- Interactive survey creation wizard
- Asks for: description, target audience, survey window
- Optionally adds questions inline
- Returns created survey ID

**`feedback question add <survey-id> [options]`**
- Adds question to survey
- Options: `--type`, `--text`, `--required`, `--options` (for multiple choice)
- Question types: text, rating, nps, multiple-choice, yes-no, csat, ces
- Can add multiple questions in batch via `--file questions.json`

**`feedback survey publish <id>`**
- Marks survey as active/published
- Sets survey status and opens for responses
- Returns survey URL or integration instructions

**`feedback response submit <survey-id> --answers <data>`**
- Creates response minion
- Accepts JSON with question IDs → answers mapping
- Validates answer formats
- Used for importing or testing

**`feedback analyze <survey-id>`**
- Runs full analysis pipeline
- Executes theme extraction, sentiment analysis, synthesis
- Creates `theme` and `sentiment-score` minions
- Displays summary report

**`feedback themes <survey-id> [options]`**
- Shows extracted themes with prevalence
- Options: `--min-prevalence`, `--sort-by` (prevalence, sentiment)
- Displays keywords for each theme
- Can output JSON with `--json`

**`feedback sentiment <survey-id> [options]`**
- Shows sentiment breakdown
- Overall scores plus per-question breakdown
- Options: `--trend` (compare to previous surveys)
- Displays distribution chart

**`feedback synthesis <survey-id> [options]`**
- Generates actionable insights report
- Includes: top themes, sentiment summary, representative quotes, recommendations
- Options: `--output` (file path), `--format` (markdown, json, html)
- Integration with `minions-roadmap` via `--create-initiatives`

**`feedback export <survey-id> --format <csv|json|xlsx>`**
- Exports all responses
- Includes question text and answers in columnar format
- Option to include analysis results (themes, sentiment)

**`feedback import <file> --format <typeform|google-forms|surveymonkey>`**
- Imports responses from external survey tools
- Creates survey and response minions
- Maps question types to internal schema

---

**DUAL SDK REQUIREMENTS**

Critical cross-language compatibility requirements:

**Serialization Parity**
- Both TypeScript and Python SDKs must serialize minions to identical JSON format
- Field names, types, and structure must match exactly
- Relation types and metadata must be interchangeable

**API Consistency**
- Same method names (adjusted for language conventions: TypeScript camelCase, Python snake_case)
- Same parameters and return types
- Same class hierarchies and interfaces

**Documentation Parity**
- Every code example in docs must have both TypeScript and Python versions
- Use Astro Starlight's code tabs: `<Tabs><TabItem label="TypeScript">...</TabItem><TabItem label="Python">...</TabItem></Tabs>`
- API reference must document both languages side by side

**Testing Parity**
- Shared test fixtures (JSON files) that both SDKs can consume
- Identical test case coverage
- Cross-language integration tests (TypeScript SDK creates survey, Python SDK analyzes responses)

---

**FIELD SCHEMAS**

Define these Minion Types with full JSON Schema definitions:

**`survey`**
```typescript
{
  id: string;
  title: string;
  description?: string;
  status: 'draft' | 'active' | 'closed';
  targetAudience?: string;          // Description of target respondents
  startDate?: Date;
  endDate?: Date;
  responseCount: number;            // Cached count of responses
  completionRate?: number;          // % of started surveys completed
  tags?: string[];
  createdAt: Date;
  updatedAt: Date;
}
```
Relations: `parent_of` → `question` minions

**`question`**
```typescript
{
  id: string;
  title: string;                    // The question text
  description?: string;             // Additional context
  questionType: 'text' | 'rating' | 'nps' | 'multiple-choice' | 'yes-no' | 'csat' | 'ces';
  required: boolean;
  options?: string[];               // For multiple-choice questions
  scaleMin?: number;                // For rating questions (default 1)
  scaleMax?: number;                // For rating questions (default 5)
  order: number;                    // Position in survey
  createdAt: Date;
  updatedAt: Date;
}
```
Relations: `part_of` → `survey`

**`response`**
```typescript
{
  id: string;
  title: string;                    // Auto-generated (e.g., "Response from user@example.com")
  answers: Record<string, any>;     // questionId → answer mapping
  respondentEmail?: string;
  respondentId?: string;            // External user identifier
  completedAt?: Date;               // If fully completed
  metadata?: Record<string, any>;   // IP, user agent, source, etc.
  createdAt: Date;
  updatedAt: Date;
}
```
Relations: `references` → `survey`

**`theme`**
```typescript
{
  id: string;
  title: string;                    // Theme name (e.g., "Onboarding friction")
  description?: string;             // Theme explanation
  keywords: string[];               // Defining terms for theme
  prevalence: number;               // % of responses mentioning this (0-100)
  confidence: number;               // Confidence in theme detection (0-100)
  representativeQuotes: string[];   // Example response excerpts
  createdAt: Date;
  updatedAt: Date;
}
```
Relations: `references` → `survey` (source of theme)
Relations: `references` → `response` minions (responses contributing to theme)

**`sentiment-score`**
```typescript
{
  id: string;
  title: string;                    // Auto-generated description
  positive: number;                 // Positive sentiment score (0-100)
  negative: number;                 // Negative sentiment score (0-100)
  neutral: number;                  // Neutral sentiment score (0-100)
  intensity: number;                // Emotional intensity (0-100)
  overallScore: number;             // Composite score (-100 to +100)
  method: string;                   // Analysis method used
  createdAt: Date;
  updatedAt: Date;
}
```
Relations: `references` → `response` or `survey` or `theme` (what was analyzed)

---

**TONE AND POSITIONING**

This is a serious tool for systematic feedback analysis. Position it as:

- **Structured feedback collection** — not just forms, but analyzable data
- **Agent-powered synthesis** — let AI find patterns and insights at scale
- **Actionable intelligence** — feedback that feeds directly into product decisions
- **Production-ready** — built for continuous user listening programs

Avoid:
- Treating this as just another survey tool (emphasize the analysis layer)
- Over-promising AI magic (be clear about what theme extraction can and can't do)
- Feature overload (focus on collection, analysis, synthesis)

The README should open with a concrete example: creating a survey, collecting responses, extracting themes, analyzing sentiment, and generating prioritized recommendations. Make it immediately tangible.

---

**INTEGRATION EXAMPLES**

Include working examples for:

**Agent Feedback Synthesis** (TypeScript)
```typescript
import { FeedbackSynthesizer, ThemeExtractor, SentimentAnalyzer } from '@minions-feedback/sdk';

const synthesizer = new FeedbackSynthesizer();
const themeExtractor = new ThemeExtractor();
const sentimentAnalyzer = new SentimentAnalyzer();

// Extract themes from survey
const themes = await themeExtractor.extractThemes(surveyId);

// Analyze sentiment
const sentiment = await sentimentAnalyzer.analyzeSurvey(surveyId);

// Generate synthesis
const insights = await synthesizer.synthesize(surveyId);

console.log('Top Themes:');
insights.topThemes.forEach(theme => {
  console.log(`- ${theme.title} (${theme.prevalence}% of responses)`);
});

console.log(`\nOverall Sentiment: ${sentiment.overallScore > 0 ? 'Positive' : 'Negative'}`);
console.log(`\nRecommendations:`);
insights.recommendations.forEach(rec => console.log(`- ${rec}`));
```

**Roadmap Integration** (Python)
```python
from minions_feedback import FeedbackSynthesizer
from minions_roadmap import InitiativeCreator

# Synthesize feedback
synthesizer = FeedbackSynthesizer()
insights = synthesizer.synthesize(survey_id)

# Create roadmap initiatives from top themes
creator = InitiativeCreator()
for theme in insights.top_themes[:3]:  # Top 3 themes
    if theme.prevalence > 20 and theme.sentiment_score < -20:
        # High-prevalence negative feedback = priority
        initiative = creator.create(
            title=f"Address: {theme.title}",
            description=theme.description,
            priority="high",
            evidence_source=theme.id
        )
        print(f"Created initiative: {initiative.title}")
```

**Real-time Sentiment Monitoring** (TypeScript)
```typescript
import { SentimentAnalyzer } from '@minions-feedback/sdk';

const analyzer = new SentimentAnalyzer();

// Track sentiment trend across survey iterations
const surveyIds = ['survey-1', 'survey-2', 'survey-3'];
const trend = await analyzer.trend(surveyIds);

console.log('Sentiment Trend:');
trend.forEach((point, idx) => {
  const arrow = idx > 0 ?
    (point.score > trend[idx-1].score ? '↑' : '↓') : '';
  console.log(`${point.date}: ${point.score} ${arrow}`);
});

if (trend[trend.length - 1].score < -50) {
  console.warn('⚠️  Sentiment declining significantly!');
}
```

---

**DELIVERABLES**

Produce all files necessary to bootstrap this project completely:

1. **Full specification** (`/spec/v0.1.md`) — complete enough to implement from
2. **TypeScript core library** (`/packages/core`) — fully functional, well-tested
3. **Python SDK** (`/packages/python`) — feature parity with TypeScript
4. **CLI tool** (`/packages/cli`) — all commands working with helpful output
5. **Documentation site** (`/apps/docs`) — complete with dual-language examples
6. **README** — compelling, clear, with concrete examples
7. **Examples** — working code in both TypeScript and Python
8. **CI/CD setup** — lint, test, and publish workflows for both languages

Every file should be production quality — not stubs, not placeholders. The spec should be complete. The core libraries should be fully functional. The docs should be ready to publish. The CLI should be ready to install and use.

---

**START SYSTEMATICALLY**

1. Write the specification first — nail down survey schemas, analysis algorithms, and sentiment models
2. Implement TypeScript core library with SurveyBuilder, ThemeExtractor, SentimentAnalyzer, and FeedbackSynthesizer
3. Port to Python maintaining exact serialization compatibility
4. Build CLI using the core library
5. Write documentation with dual-language examples throughout
6. Create working examples demonstrating survey creation and feedback synthesis
7. Write the README with concrete agent-powered analysis use cases

This is a foundational tool for product intelligence in the Minions ecosystem. Agents will use this to understand users and prioritize product work. Get it right.
