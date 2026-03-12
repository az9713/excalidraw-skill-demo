# Excalidraw Diagram Skill — How It Works

A Claude Code skill that generates production-quality Excalidraw diagrams from natural language descriptions. Instead of generic boxes-and-arrows, it produces diagrams that **argue visually** — where the shape of each element mirrors the concept it represents.

---

## How the Skill Works

### 1. Skill Detection

When the skill is installed at `.claude/skills/excalidraw/`, Claude Code automatically loads `SKILL.md` when you ask it to create a diagram. No special commands needed — just describe what you want to visualize.

### 2. Depth Assessment

The skill first determines what level of detail is needed:

- **Simple/Conceptual** — abstract shapes, labels, and relationships (mental models, high-level overviews)
- **Comprehensive/Technical** — concrete examples, code snippets, real data formats, multi-zoom levels (system architectures, protocol explainers, tutorials)

### 3. Research Phase (Technical Diagrams)

For technical subjects, the skill instructs Claude Code to research actual specifications before drawing anything — real event names, API formats, JSON payloads, method signatures. This makes diagrams accurate and educational, not just decorative.

### 4. Concept-to-Pattern Mapping

Each concept is mapped to a visual pattern that mirrors its behavior:

| If the concept...             | Visual pattern used          |
|-------------------------------|------------------------------|
| Spawns multiple outputs       | **Fan-out** (radial arrows)  |
| Combines inputs into one      | **Convergence** (funnel)     |
| Has hierarchy/nesting         | **Tree** (lines + text)      |
| Is a sequence of steps        | **Timeline** (line + dots)   |
| Loops or iterates             | **Spiral/Cycle**             |
| Transforms input to output    | **Assembly line**            |
| Compares two things           | **Side-by-side**             |

### 5. Section-by-Section JSON Generation

For large diagrams, the skill builds the `.excalidraw` JSON file one section at a time rather than in a single pass. This avoids output token limits and produces higher quality layouts.

### 6. Semantic Color System

All colors come from a single palette file (`references/color-palette.md`) where each color encodes meaning:

| Purpose         | Fill color | Stroke color |
|-----------------|------------|--------------|
| Start/Trigger   | Warm peach | Dark orange  |
| End/Success     | Light green| Dark green   |
| Decision        | Light amber| Dark amber   |
| AI/LLM          | Light purple| Dark purple |
| Error           | Light red  | Dark red     |
| Code snippets   | Dark slate background | Syntax-colored text |

### 7. Render-View-Fix Loop

After generating the JSON, the skill renders the diagram to PNG using a Playwright-based renderer, views the image, and fixes visual issues (text overflow, overlapping elements, misaligned arrows) — repeating until the diagram passes quality checks. This typically takes 2-4 iterations.

### 8. Output

You get two files:
- `.excalidraw` — editable JSON you can open in [excalidraw.com](https://excalidraw.com) or the VS Code Excalidraw extension
- `.png` — rendered preview image

---

## Prerequisites

Before using the skill, you need the following installed on your system:

### 1. Claude Code

The skill runs inside [Claude Code](https://docs.anthropic.com/en/docs/claude-code), Anthropic's CLI agent. Install it globally via npm:

```bash
npm install -g @anthropic-ai/claude-code
```

You'll need an Anthropic API key or a Claude Pro/Max subscription to use Claude Code.

### 2. uv (Python package manager)

The render pipeline uses [uv](https://docs.astral.sh/uv/) to manage Python dependencies. Install it:

**macOS/Linux:**
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

**Windows (PowerShell):**
```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

**Or via pip:**
```bash
pip install uv
```

uv requires **Python 3.11+**. If you don't have Python installed, uv will manage it for you.

### 3. Playwright + Chromium (render pipeline)

The skill renders diagrams to PNG using Playwright to drive a headless Chromium browser. This is installed as part of the setup steps below.

---

## Installation & Setup

### Step 1: Install the skill into your project

Copy the skill folder into your project's `.claude/skills/` directory:

```bash
mkdir -p .claude/skills
cp -r excalidraw-diagram-skill .claude/skills/excalidraw
```

Your project structure should look like:

```
your-project/
  .claude/
    skills/
      excalidraw/
        SKILL.md
        README.md
        references/
          color-palette.md
          element-templates.md
          json-schema.md
          pyproject.toml
          render_excalidraw.py
          render_template.html
```

### Step 2: Install Python dependencies

```bash
cd .claude/skills/excalidraw/references
uv sync
```

This reads `pyproject.toml` and installs the `playwright` Python package (>=1.40.0) into a local virtual environment.

### Step 3: Install Chromium for rendering

```bash
uv run playwright install chromium
```

This downloads a Chromium binary that Playwright uses to render `.excalidraw` files to PNG. It's a one-time download (~150 MB).

### Step 4: Verify the setup

You can verify everything works by running the renderer manually on any `.excalidraw` file:

```bash
uv run python render_excalidraw.py path/to/your-diagram.excalidraw
```

If successful, it will output a `.png` file next to the `.excalidraw` file.

### Quick setup (all steps at once)

Or just tell Claude Code:

> "Set up the Excalidraw diagram skill renderer by following the instructions in SKILL.md."

It will run all the commands for you.

---

## 10 Example Prompts

Each example below demonstrates a different strength of the skill. Copy-paste any of these into Claude Code to try it.

---

### 1. Microservices Architecture with Real API Contracts

> "Create an Excalidraw diagram of an e-commerce microservices architecture. Show the API Gateway routing to Order Service, Inventory Service, and Payment Service. Include actual REST endpoint examples (POST /orders, GET /inventory/{sku}), show the JSON request/response payloads between services, and diagram how Kafka events propagate order state changes across services."

**What this showcases:** Evidence artifacts (real endpoints, JSON payloads), fan-out pattern from API Gateway, assembly line for order processing, multi-zoom architecture (overview + detail).

---

### 2. Git Branching Strategy

> "Create an Excalidraw diagram explaining the Gitflow branching model. Show main, develop, feature, release, and hotfix branches as parallel timelines. Mark merge points, tag releases, and show the actual git commands used at each transition (git checkout -b, git merge --no-ff, git tag). Use a timeline pattern with dots at each commit."

**What this showcases:** Timeline pattern with parallel tracks, evidence artifacts (real git commands), color-coded branches by semantic purpose (main = success/green, hotfix = error/red, feature = primary/blue).

---

### 3. OAuth 2.0 Authorization Code Flow

> "Create a comprehensive Excalidraw diagram of the OAuth 2.0 Authorization Code flow. Show all parties (User, Browser, App Server, Auth Server, Resource Server) and every HTTP redirect/request between them. Include the actual query parameters (response_type=code, client_id, redirect_uri, scope) and show what the authorization code and access token look like. Make it educational enough that someone could implement OAuth from this diagram alone."

**What this showcases:** Sequence/timeline pattern across multiple actors, evidence artifacts (HTTP parameters, token formats), comprehensive depth with educational intent, research-driven accuracy.

---

### 4. Data Pipeline — Raw to Dashboard

> "Create an Excalidraw diagram showing a data pipeline: raw CSV files are uploaded to S3, triggered by an S3 event to a Lambda function, which transforms and loads into a PostgreSQL data warehouse, then dbt models transform it into analytics tables, and finally Metabase dashboards visualize the results. Show a sample CSV row, the transformed JSON event, a SQL transformation snippet, and a mockup of what the final dashboard chart looks like."

**What this showcases:** Assembly line (transformation) pattern, evidence artifacts at every stage (CSV sample, JSON event, SQL code, UI mockup), clear before/after at each step, multi-zoom from overview to detail.

---

### 5. Decision Tree — Incident Response

> "Create an Excalidraw diagram of an incident response decision tree. Start with 'Alert Triggered', then branch on severity (P1/P2/P3/P4). P1 goes to 'Page On-Call → War Room → Status Page Update → Post-Mortem'. P2 goes to 'Slack Notification → Investigate → Fix or Escalate'. P3/P4 go to 'Ticket Created → Backlog'. Use diamond shapes for decisions and show the actual PagerDuty/Slack actions at each step."

**What this showcases:** Decision diamonds with fan-out, semantic colors (P1 = error/red, P2 = warning/amber, P3-P4 = neutral/blue), tree pattern for branching paths, concrete action names instead of generic labels.

---

### 6. React Component Lifecycle with Hooks

> "Create an Excalidraw diagram mapping the React component lifecycle using hooks. Show the mounting, updating, and unmounting phases as distinct sections. Inside each phase, show which hooks fire (useState, useEffect, useLayoutEffect, useMemo, useRef) with actual code snippets showing common patterns. Include the render cycle as a spiral/loop that re-enters on state change."

**What this showcases:** Cycle/spiral pattern for re-renders, section boundaries for lifecycle phases, evidence artifacts (real hook code snippets), gap/break between phases, educational depth.

---

### 7. Kubernetes Pod Scheduling

> "Create an Excalidraw diagram showing how Kubernetes schedules a pod. Show the journey: kubectl apply → API Server → etcd → Scheduler (with filtering and scoring phases) → kubelet → Container Runtime → Running Pod. Include the actual YAML spec snippet, show the scheduler's node filtering criteria (resource requests, taints/tolerations, node affinity), and show the scoring algorithm visually as a convergence funnel."

**What this showcases:** Assembly line from request to running pod, convergence pattern for scheduler scoring, evidence artifacts (YAML spec, filtering criteria), side-by-side comparison of candidate nodes.

---

### 8. Event Sourcing vs CRUD — Side-by-Side Comparison

> "Create an Excalidraw diagram comparing Event Sourcing with traditional CRUD. Show them side-by-side: on the left, a CRUD flow (UPDATE users SET name='...' WHERE id=1) with the current-state-only database. On the right, an Event Sourcing flow showing the event stream (UserCreated, NameChanged, EmailUpdated) rebuilding state through a projection. Show the actual SQL vs event JSON. Highlight what you lose with CRUD (history, audit trail) and what you gain with Event Sourcing."

**What this showcases:** Side-by-side comparison pattern, evidence artifacts on both sides (SQL vs JSON events), convergence pattern for event projection, visual contrast between approaches.

---

### 9. CI/CD Pipeline with Failure Paths

> "Create an Excalidraw diagram of a CI/CD pipeline: Push to GitHub → GitHub Actions workflow triggers → parallel jobs (lint, unit tests, integration tests, security scan) → merge gate → build Docker image → deploy to staging → smoke tests → manual approval → deploy to production. Show the failure paths: where each step fails, what notification is sent (Slack message example), and what the rollback procedure looks like. Use fan-out for parallel jobs and convergence for the merge gate."

**What this showcases:** Fan-out and convergence patterns, parallel tracks, error/warning colors for failure paths, cycle pattern for rollback, evidence artifacts (GitHub Actions YAML snippet, Slack notification mockup).

---

### 10. LLM RAG Architecture — Query to Answer

> "Create a comprehensive Excalidraw diagram of a RAG (Retrieval-Augmented Generation) architecture. Show the ingestion pipeline (documents → chunking → embedding → vector store) and the query pipeline (user question → embedding → similarity search → context assembly → LLM prompt → response). Include a real embedding vector snippet [0.023, -0.041, ...], show the actual prompt template with {context} and {question} placeholders, and show a sample similarity search result with distance scores. Make it detailed enough for a conference talk."

**What this showcases:** Two assembly lines (ingestion + query), evidence artifacts throughout (embedding vectors, prompt template, search results), multi-zoom architecture, convergence where retrieved chunks merge into the prompt, comprehensive/educational depth.

---

## Customization

To match your brand colors, edit `.claude/skills/excalidraw/references/color-palette.md`. The rest of the skill — design methodology, visual patterns, layout rules — is universal and doesn't need changes.

---

## Key Design Principles

1. **Argue, don't display** — the visual structure should communicate meaning even without text
2. **Evidence over labels** — show real code, real data, real formats instead of generic "Input" / "Output" boxes
3. **Shape = meaning** — fan-outs for one-to-many, diamonds for decisions, timelines for sequences
4. **Minimal containers** — prefer free-floating text over boxing everything
5. **Semantic color** — every color choice maps to a purpose (start, end, error, decision, AI)
6. **Render and validate** — never ship a diagram without visually inspecting the rendered PNG
