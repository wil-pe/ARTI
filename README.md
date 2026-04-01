# Arcti

**A powerful, recursive, and adaptive multi-agent orchestration system built with TypeScript and Bun.**

Arcti turns a single instruction into a complete tree of decomposed tasks, executed in parallel, verified, and synthesized automatically. It supports 6 LLM providers (local & cloud), a real-time dashboard, built-in web search, Telegram/WhatsApp bots, a persistent skill library, and much more.

> "Less, but better." — Dieter Rams inspired design

<img width="1798" height="1042" alt="Arcti Dashboard" src="https://github.com/user-attachments/assets/df312f62-b91d-474f-af51-f82c3fe5330c" />

---

## Key Features

### Core Engine
- **Recursive task decomposition** — breaks complex goals into atomic subtasks (depth up to 10, up to 500 tasks)
- **DAG Scheduler** — maximum parallel execution (8 concurrent) with dependency resolution
- **3-tier model routing** — low/medium/high complexity tasks routed to appropriate models
- **Automatic escalation** — failed tasks retry with stronger models
- **Smart verification** — hallucination detection, code sandbox execution, auto-skip for simple tasks
- **Task redo** — retry any task (including parallel groups) with optional feedback
- **Persistent Skill Library** — auto-save, score (0-100), and reuse generated tools across projects
- **Flow Mode** — save completed projects as reusable templates with `{{variables}}`, re-run instantly

### LLM Providers

| Provider | Type | Models |
|----------|------|--------|
| **Ollama** | Local/Cloud | Auto-detected (gemma3, qwen2.5, llama3.1, etc.) |
| **Claude Code** | Local CLI | haiku, sonnet, opus (uses subscription credits) |
| **Anthropic** | Cloud API | claude-haiku, claude-sonnet, claude-opus |
| **OpenAI** | Cloud API | gpt-4o-mini, gpt-4o, o3 |
| **Grok** | Cloud API | grok-3-mini, grok-3 |
| **OpenRouter** | Cloud Proxy | 300+ models, browsable from dashboard |

- Per-tier model assignment from the dashboard
- Dynamic model listing from provider APIs
- Enable/disable providers and manage API keys from the UI

### Built-in Tools

| Tool | Description |
|------|-------------|
| `web_search` | Search the web via DuckDuckGo |
| `fetch_url` | Fetch and extract web page content |
| `read_file` | Read local files |
| `write_file` | Write local files |
| `list_dir` | List directory contents |
| `shell` | Execute shell commands (sandboxed) |
| `eval_code` | Run TypeScript/JavaScript code (sandboxed) |
| `screenshot` | Capture screen or region as PNG (macOS) |
| `click` | Click at screen coordinates (macOS) |
| `type_text` | Type text via keyboard simulation (macOS) |
| `scroll` | Scroll at a given position (macOS) |

- Multi-turn tool calling — up to 5 rounds per task (10 for computer-use tasks)
- Automatic fallback to raw tool results if model output is empty

### Dashboard
- Real-time updates via WebSocket
- Provider management — toggle, API keys, model browser
- Hierarchical task tree with parallel layout
- Task detail panel (output, reasoning, criteria, retry diff)
- Task redo with feedback input
- Cost estimation per project
- Project export/import
- Flow templates — save, browse, and run with variable substitution
- Dark mode
- Live LLM streaming
- Live preview (iframe for generated HTML/CSS/JS)

### Telegram & WhatsApp Bots
- `/run`, `/status`, `/stop`, `/skills`, `/help` commands
- Plain text launches a new project
- In-chat QCM answering for requirement clarification
- Real-time progress updates
- Result preview delivery (first 1500 chars) + artifact files
- Auto-assigned workspace per run (`~/.arcti/outputs/<runId>/`)

### Extras
- **Cross-project memory** — learns patterns & preferences across runs
- **Code sandbox** — isolated TS/JS/Python/Shell execution
- **QQOQCP intake** — smart requirement clarification (max 3 questions)
- **OS notifications**
- **Keyboard shortcuts**
- **Single binary build**

---

## Quick Start

### Prerequisites
- [Bun](https://bun.sh) (v1.3+)
- [Ollama](https://ollama.ai) (optional, for local models)
- Claude CLI (optional, for subscription credits)

### Installation

```bash
git clone https://github.com/wil-pe/ARTI.git
cd arcti
bun install
```

### Configuration

```bash
cp arcti.yaml.example arcti.yaml
# Edit arcti.yaml with your preferred providers and models
```

### Commands

```bash
# Check providers and available models
bun run src/index.ts check

# Run a task in CLI mode (interactive)
bun run src/index.ts run "Create a SaaS landing page for a coffee subscription"

# Start web server + real-time dashboard
bun run src/index.ts server

# Build standalone binary
bun build --compile src/index.ts --outfile arcti
./arcti server
```

Dashboard available at **http://localhost:3000/**

---

## Configuration

```yaml
# arcti.yaml
default_provider: ollama
default_models:
  tier1: qwen2.5:3b        # Low complexity
  tier2: gemma3:12b         # Medium complexity
  tier3: llama3.1:70b       # High complexity

providers:
  ollama:
    enabled: true
    base_url: "http://localhost:11434"
    mode: both              # local, cloud, or both
  anthropic:
    enabled: false
    api_key_env: ANTHROPIC_API_KEY
  openai:
    enabled: false
    api_key_env: OPENAI_API_KEY
  grok:
    enabled: false
    api_key_env: XAI_API_KEY
    base_url: "https://api.x.ai"
  openrouter:
    enabled: false
    api_key_env: OPENROUTER_API_KEY
    base_url: "https://openrouter.ai/api"
  claude-code:
    enabled: true

bots:
  telegram:
    enabled: false
    token: ""
  whatsapp:
    enabled: false
    token: ""
    phone_id: ""
    verify_token: ""

scheduler:
  maxConcurrency: 8
  recursion:
    maxDepth: 10
    maxTotalTasks: 500
```

API keys can also be managed directly from the dashboard Settings panel (persisted to `.env`).

---

## Project Structure

```
src/
├── index.ts                 # CLI entry point (commander)
├── config.ts                # Multi-provider configuration
├── core/
│   ├── types.ts             # Shared types
│   ├── run-manager.ts       # Project lifecycle & task redo
│   ├── events.ts            # EventBus (WebSocket relay)
│   ├── sandbox.ts           # Code execution sandbox
│   ├── memory.ts            # Cross-project memory
│   └── cost-estimator.ts    # Token & cost estimation
├── agents/
│   ├── base.ts              # BaseAgent (shared LLM interface)
│   ├── mother.ts            # Mother Agent (intake + synthesis)
│   ├── worker.ts            # Worker Agent (execution + tools)
│   └── verifier.ts          # Verifier (hallucination detection)
├── llm/
│   ├── client.ts            # LLMClient + provider registry
│   ├── router.ts            # Tier-based model routing
│   └── providers/           # Ollama, OpenAI, Claude Code, etc.
├── tools/
│   ├── registry.ts          # Tool registration
│   ├── generator.ts         # Dynamic tool generation
│   ├── skill-library.ts     # Persistent skill storage
│   ├── flow-library.ts      # Flow template storage & instantiation
│   └── builtins/            # web_search, fetch_url, shell, computer-use, etc.
├── orchestrator/
│   ├── intake.ts            # QQOQCP requirement analysis
│   ├── decomposer.ts        # Recursive task decomposition
│   ├── scheduler.ts         # DAG scheduler (parallel execution)
│   └── flow-runner.ts       # Flow execution (skip intake/decompose)
├── bot/
│   └── manager.ts           # Telegram & WhatsApp integration
├── server/
│   └── app.ts               # Hono API + WebSocket server
└── web/
    └── index.html           # Dashboard (single-page app)
```

---

## Architecture

```
User Input
    │
    ▼
┌─────────┐     ┌──────────────┐     ┌───────────┐
│  Intake  │────▶│  Decomposer  │────▶│ Scheduler │
│ (QQOQCP) │     │ (recursive)  │     │   (DAG)   │
└─────────┘     └──────────────┘     └─────┬─────┘
                                           │
                              ┌─────────────┼─────────────┐
                              ▼             ▼             ▼
                         ┌────────┐   ┌────────┐   ┌────────┐
                         │ Worker │   │ Worker │   │ Worker │
                         │ +Tools │   │ +Tools │   │ +Tools │
                         └───┬────┘   └───┬────┘   └───┬────┘
                             ▼             ▼             ▼
                         ┌────────┐   ┌────────┐   ┌────────┐
                         │Verifier│   │Verifier│   │Verifier│
                         └───┬────┘   └───┬────┘   └───┬────┘
                              │             │             │
                              └─────────────┼─────────────┘
                                           ▼
                                    ┌─────────────┐
                                    │  Synthesis   │
                                    │ (Mother Agent)│
                                    └──────┬──────┘
                                           ▼
                                      Final Output
```

---

## Dashboard Guide

- **Left sidebar** — Projects list + Flows + Skills library
- **Flows** — Browse saved templates, run with variables, save completed projects as flows
- **Run button** — Start a new project (choose provider, set output directory)
- **Click any task** — Full details: output, criteria, verification status, retry diff
- **Redo button** — Retry any failed or verified task with optional feedback
- **Settings (gear)** — Manage providers, API keys, per-tier models, bot tokens
- **Open folder** — Open project output directory in Finder

---

## Changelog

### v0.2.0-alpha — Flow Mode + Computer Vision

**Flow Mode (reusable templates)**
- Save any completed project as a reusable flow template
- Define variables (`{{subject}}`, `{{audience}}`, etc.) that get substituted at runtime
- Flow library with scoring, usage tracking, stored in `~/.arcti/flows/`
- Dashboard UI: browse flows, run with variable form, save from completed projects
- API: `GET/POST /flows`, `GET/DELETE /flows/:name`, `POST /flows/:name/run`
- Skips intake + decomposition — instant execution from saved task trees

**Computer Vision / Screen Use**
- 4 new tools: `screenshot`, `click`, `type_text`, `scroll`
- macOS native: `screencapture` for screenshots, `cliclick`/AppleScript for input
- Multi-modal support: screenshot results passed as images to vision-capable LLMs
- Works with Anthropic, OpenAI, OpenRouter, and Ollama vision models
- Extended tool rounds (up to 10) for computer-use tasks
- Sandboxed with permission checks (Screen Recording + Accessibility)

**Multi-modal LLM support**
- Providers now handle image content in messages (base64 PNG)
- OpenAI: `image_url` content blocks
- Anthropic: native `image` source blocks
- Ollama: `images` field for vision models (llava, etc.)

### v0.1.0-alpha — Initial Release
- Recursive task decomposition (depth 10, up to 500 tasks)
- DAG scheduler with 8 concurrent workers
- 3-tier model routing with auto-escalation
- 6 LLM providers (Ollama, Claude Code, Anthropic, OpenAI, Grok, OpenRouter)
- 7 built-in tools (web_search, fetch_url, read_file, write_file, list_dir, shell, eval_code)
- Real-time dashboard with dark mode
- Telegram & WhatsApp bots
- Skill library, cross-project memory, code sandbox
- Cost estimation, project export/import

---

## Roadmap

- Visual DAG graph (interactive flowchart)
- Fully autonomous agent mode
- Google Search API integration (Serper.dev)
- Expanded unit tests
- Plugin system for custom providers

---

## License

MIT

---

#### Arcti — Because a great agent doesn't do everything alone. It orchestrates intelligently.

Built with Bun + TypeScript in Lima.
