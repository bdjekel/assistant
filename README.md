# Personal Assistant System

A personal operating system for a data engineer, designed for use with Kiro CLI and Kiro IDE. This system provides AI-assisted coding context, career tracking, and quick capture workflows through progressive context loading.

## What This Is

This directory (`~/assistant/`) contains the **content layer** for an AI-powered personal assistant system. The **control layer** lives in `~/.kiro/steering/` as steering files that determine when and how this content is loaded.

### Content Layer (This Directory)

- **Coding context** — Tech stack knowledge (Python, PySpark, SQL, Terraform, Go, Databricks, AWS)
- **Career tracking** — Append-only achievement logging, goals, and skills inventory
- **Quick capture** — Frictionless note-taking via `now.md`

### Control Layer (`~/.kiro/steering/`)

- **router.md** — Always loaded, provides system overview and quick commands
- **coding.md** — Auto-loads when working with code files
- **career.md** — Auto-loads when working in `~/assistant/career/`
- **data-*.md** — Domain-specific coding standards (errors, organization, review, documentation, testing, performance, security, stack, types)

## System Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                         Kiro CLI / IDE                              │
├─────────────────────────────────────────────────────────────────────┤
│  Quick Commands            │  Knowledge Base Search                 │
│  @qnote, @career-win       │  /knowledge search                     │
└──────────────┬─────────────┴──────────────┬────────────────────────┘
               │                             │
               ▼                             ▼
┌──────────────────────────┐   ┌──────────────────────────────────────┐
│ Steering Files (Control) │   │    Knowledge Bases (Indexed Search)  │
│  ~/.kiro/steering/       │   │  ┌────────────────────────────────┐  │
│  ├── router.md (always)  │   │  │ coding-context (Best/semantic) │  │
│  ├── coding.md (auto)    │   │  │ career-achievements (Fast/bm25)│  │
│  ├── career.md (auto)    │   │  │ quick-capture (Fast/bm25)      │  │
│  └── data-*.md (auto)    │   │  └────────────────────────────────┘  │
└──────────────┬───────────┘   └──────────────────────────────────────┘
               │                                    │
               └────────────────┬───────────────────┘
                                ▼
┌─────────────────────────────────────────────────────────────────────┐
│              Content Files (~/assistant/)                           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌───────────┐  │
│  │   coding/   │  │   career/   │  │  obsidian/  │  │  now.md   │  │
│  │ tech-stack  │  │ achievements│  │  templates  │  │ (scratch) │  │
│  │   stack/    │  │   goals     │  │             │  │           │  │
│  │  patterns   │  │   skills    │  │             │  │           │  │
│  │  snippets/  │  │             │  │             │  │           │  │
│  └─────────────┘  └─────────────┘  └─────────────┘  └───────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

## Directory Structure

### Content Files (`~/assistant/`)

```
~/assistant/                    # This directory
├── coding/
│   ├── tech-stack.md           # Index of available tech context
│   ├── stack/                  # Per-technology guidance
│   │   ├── python.md           # Python patterns, typing, testing
│   │   ├── pyspark.md          # DataFrames, performance, UDFs
│   │   ├── sql.md              # Query optimization, modeling
│   │   ├── terraform.md        # Modules, state, AWS patterns
│   │   ├── go.md               # Idioms, error handling, concurrency
│   │   ├── databricks.md       # Unity Catalog, jobs, Delta Lake
│   │   └── aws.md              # S3, Glue, Lambda, Step Functions
│   ├── patterns.md             # Code review, testing, documentation
│   └── snippets/               # Reusable code templates
├── career/
│   ├── achievements.jsonl      # Append-only wins log
│   ├── goals.yaml              # Current OKRs
│   └── skills.md               # Skills inventory
├── obsidian/
│   └── templates/              # Note templates (future)
└── now.md                      # Quick capture scratch pad
```

### Steering Files (`~/.kiro/steering/`)

```
~/.kiro/steering/               # Control layer (not in this repo)
├── router.md                   # Always loaded - system overview
├── coding.md                   # Auto-loads for code files
├── career.md                   # Auto-loads in career directory
├── data-errors.md              # Error handling guidelines
├── data-organization.md        # Code organization patterns
├── data-review.md              # Code review checklist
├── data-documentation.md       # Documentation standards
├── data-testing.md             # Testing requirements
├── data-performance.md         # Performance optimization
├── data-security.md            # Security requirements
├── data-stack.md               # Tech stack reference
└── data-types.md               # Type safety guidelines
```

## Quick Commands

These are defined as reusable prompts in `~/.kiro/steering/router.md`:

| Command | Description |
|---------|-------------|
| `@qnote [text]` | Append thought to `now.md` with timestamp |
| `@career-win [description]` | Log achievement to `achievements.jsonl` |
| `@kb-add [path]` | Generate command to add new knowledge base |

## Slash Commands

Manual steering files available as slash commands:

| Command | Description |
|---------|-------------|
| `/asst` | Load system documentation and README reference |
| `/health` | Run system health check and diagnostics |

To use: Type `/` in chat and select from the menu, or type the command directly.

## Setup

### 1. Enable Knowledge Management

```bash
kiro-cli settings chat.enableKnowledge true
```

### 2. Create Knowledge Bases

```bash
# Career achievements (keyword search for structured data)
/knowledge add --name "career-achievements" \
  --path ~/assistant/career/ \
  --index-type Fast \
  --include "*.jsonl"

# Coding context (semantic search for documentation)
/knowledge add --name "coding-context" \
  --path ~/assistant/coding/ \
  --index-type Best \
  --include "*.md"

# Quick capture
/knowledge add --name "quick-capture" \
  --path ~/assistant/now.md \
  --index-type Fast
```

### 3. Verify Steering Files

Steering files should already exist in `~/.kiro/steering/`. Verify with:

```bash
ls -la ~/.kiro/steering/
```

Expected files:
- `router.md` (always loaded)
- `coding.md` (auto-loads for code)
- `career.md` (auto-loads in career dir)
- `data-*.md` (domain-specific standards)

## Usage

### In Kiro CLI

**Search knowledge bases** instead of loading full files:

```bash
/knowledge search coding-context "pyspark broadcast join"
/knowledge search career-achievements "Q4 2025"
/knowledge search coding-context "terraform module structure"
```

**Use quick commands** for common operations:

```bash
@qnote "Remember to refactor the ETL pipeline next week"
@career-win "Reduced pipeline runtime by 40% using broadcast joins"
```

### In Kiro IDE

**Reference files directly** with `#File`:

```
#File ~/assistant/coding/stack/pyspark.md
#File ~/assistant/career/goals.yaml
```

Or configure MCP servers for external documentation access.

### Progressive Context Loading

The system automatically loads relevant context based on what you're working on:

- **Always loaded**: `router.md` (lightweight overview)
- **Working on Python/SQL/Terraform**: `coding.md` + `data-*.md` files auto-load
- **In `~/assistant/career/`**: `career.md` auto-loads
- **Everything else**: Only router is loaded

This prevents context bloat while ensuring relevant information is always available.

## Steering File Details

### router.md (Always Loaded)

- System overview (~50 lines)
- Quick command reference
- Module descriptions
- Environment context (CLI vs IDE)

### coding.md (Auto-loads for Code Files)

Triggers on: `**/*.{py,sql,tf,go,scala,json,yaml}`

- Tech stack index
- Knowledge base search instructions
- Links to patterns and snippets

### career.md (Auto-loads in Career Directory)

Triggers on: `~/assistant/career/**`

- Achievement logging schema
- Type inference rules
- Goals and skills reference
- History search instructions

### data-*.md (Domain-Specific Standards)

Auto-load with `coding.md`:

- **data-errors.md** — Error messages, logging, exception handling
- **data-organization.md** — Project structure, anti-patterns
- **data-review.md** — Review checklist, high-scrutiny changes
- **data-documentation.md** — Docstring requirements, schema docs
- **data-testing.md** — Test coverage, structure, property-based testing
- **data-performance.md** — PySpark optimization, monitoring
- **data-security.md** — Secrets management, input validation
- **data-stack.md** — Approved languages, frameworks, tools
- **data-types.md** — Type hints, schema definitions

## Achievement Schema

Achievements are stored in JSONL format (append-only):

```json
{
  "date": "2026-03-03",
  "type": "project",
  "title": "Shipped new data pipeline",
  "details": "Reduced processing time by 40% using broadcast joins",
  "evidence": "https://link-to-pr"
}
```

**Types**: `project` | `skill` | `recognition` | `impact` | `learning`

**Type Inference Rules** (from `career.md`):
- **project** — Shipped, deployed, launched, completed, delivered
- **skill** — Learned, certified, mastered, practiced
- **recognition** — Promoted, awarded, praised, recognized
- **impact** — Improved, reduced, increased, saved, optimized
- **learning** — Studied, read, attended, completed course

## Adding New Tech Context

1. **Create a new file** in `coding/stack/`:
   ```bash
   touch ~/assistant/coding/stack/airflow.md
   ```

2. **Add entry** to `coding/tech-stack.md`:
   ```markdown
   | Airflow | `stack/airflow.md` | DAGs, operators, scheduling |
   ```

3. **Re-index** the knowledge base:
   ```bash
   /knowledge add --name "coding-context" \
     --path ~/assistant/coding/ \
     --index-type Best \
     --include "*.md"
   ```

## Design Principles

1. **Minimize maintenance burden** — If you won't update it, don't build it
2. **Progressive disclosure** — Load context only when needed
3. **Separate reference and operational data** — Reference data is stable, operational data is append-only
4. **Append-only for operational data** — JSONL prevents accidental data loss
5. **Search over load** — Use knowledge bases instead of file references to save context window

## Tech Stack

### Languages & Frameworks
- **Python** — PySpark, pytest, hypothesis (property-based testing)
- **Go** — APIs with Prometheus/Grafana monitoring
- **SQL** — Query optimization, data modeling
- **Scala** — Limited use

### Data Technologies
- **Databricks** — Unity Catalog, jobs, notebooks
- **Delta Lake** — Table format
- **Parquet** — File format

### Infrastructure
- **Terraform** — Infrastructure as code
- **AWS** — S3, Glue, Lambda, Step Functions, IAM

## Migration History

This system was migrated from the `digital-brain-skill` example in the [Agent-Skills-for-Context-Engineering](https://github.com/bdjekel/Agent-Skills-for-Context-Engineering) repository. The migration specs are located in:

```
~/Developer/Repositories - bdjekel (github)/Agent-Skills-for-Context-Engineering/
  .kiro/specs/kiro-cli-migration/
```

Key changes from the original:
- Migrated from Claude Code to Kiro CLI
- Simplified from 6 modules to 2 (coding, career)
- Replaced SKILL.md/AGENT.md hierarchy with steering files
- Reduced operational logs from 11 JSONL files to 1 (achievements)
- Removed Python automation scripts (replaced with knowledge bases)
- Separated control layer (steering) from content layer (this directory)

## Related Documentation

- [Kiro CLI Documentation](https://kiro.dev/docs/)
- [Steering Files Guide](https://kiro.dev/docs/steering)
- [Knowledge Management](https://kiro.dev/docs/knowledge)
- [Custom Agents](https://kiro.dev/docs/agents)
