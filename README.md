# 🧠 Universal Context-Bridge & Autonomous Browser Agent

> **Bridge your local codebase with free AI platforms — no backend, no subscription, no copy-paste.**

A production-grade Chrome Extension (Manifest V3) that connects your local development workspace directly to free AI chat interfaces (Claude, ChatGPT, Gemini). Built on top of proven open-source foundations and extended with unique capabilities that don't exist anywhere else.

---

## 📋 Table of Contents

- [The Problem We Solve](#-the-problem-we-solve)
- [What Makes This Different](#-what-makes-this-different)
- [Architecture Overview](#-architecture-overview)
- [The 4 Core Pillars](#-the-4-core-pillars)
- [Unique Features (Not Available Elsewhere)](#-unique-features-not-available-elsewhere)
- [Tech Stack & Dependencies](#-tech-stack--dependencies)
- [Open-Source Foundations We Build On](#-open-source-foundations-we-build-on)
- [Project Structure](#-project-structure)
- [Build Roadmap](#-build-roadmap)
- [Use Case Scenarios](#-use-case-scenarios)
- [Installation (Development)](#-installation-development)
- [Contributing](#-contributing)
- [License](#-license)

---

## 🔥 The Problem We Solve

Every developer using AI in 2025 faces the same friction loop:

1. Open Claude or ChatGPT in the browser
2. Manually copy 300 lines of code from VS Code
3. Paste, explain the context, wait for response
4. Copy the response back into the editor
5. Repeat — for every file, every question, every session

Existing solutions either require a paid IDE plugin (Cursor, GitHub Copilot), a $200/month subscription (Claude Max), or they only bridge AI-to-AI context without touching local files at all.

**This extension eliminates that entire loop.**

---

## ✨ What Makes This Different

| Feature | This Project | BridgeContext | Nanobrowser | Cursor/Copilot |
|---|---|---|---|---|
| Local filesystem access (no IDE) | ✅ | ❌ | ❌ | ✅ (IDE only) |
| Injects context into free AI UIs | ✅ | ✅ | ❌ | ❌ |
| Git-aware context (only diffs) | ✅ | ❌ | ❌ | ❌ |
| Autonomous browser agent | ✅ | ❌ | ✅ | ❌ |
| Multi-AI consensus mode | ✅ | ❌ | ❌ | ❌ |
| Zero backend / zero server cost | ✅ | ✅ | ✅ | ❌ |
| PII stripping before cloud upload | ✅ | ❌ | ❌ | ❌ |
| PDF/ZIP export (client-side) | ✅ | ❌ | ❌ | ❌ |

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    Chrome Side Panel (UI)                    │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │ Context  │  │  Bridge  │  │  Agent   │  │ Exports  │   │
│  │ Engine   │  │ & Inject │  │ Control  │  │ PDF/ZIP  │   │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └──────────┘   │
└───────┼─────────────┼─────────────┼────────────────────────┘
        │             │             │
        ▼             ▼             ▼
┌───────────┐  ┌─────────────┐  ┌──────────────────────┐
│ FileSystem│  │ Content     │  │ Tab Orchestration    │
│ Access API│  │ Scripts     │  │ (chrome.tabs +       │
│(local disk)│  │(DOM inject) │  │  chrome.scripting)   │
└───────────┘  └──────┬──────┘  └──────────────────────┘
                      │
              ┌───────▼────────┐
              │  AI Platforms  │
              │ Claude.ai      │
              │ ChatGPT        │
              │ Gemini         │
              └────────────────┘
```

The extension runs entirely client-side. No data leaves the machine except through the AI platform's own input field — which the user controls explicitly.

---

## 🧱 The 4 Core Pillars

### Pillar 1 — Local Context Engine & Smart Chunking

Reads and processes your local codebase directly from the browser's side panel using the native `FileSystem Access API` — no file uploads, no server, no IDE required.

**How it works:**

```
Local Project Folder
        │
        ▼
┌───────────────────┐     ┌──────────────────────┐
│ Directory Scanner │────▶│  Arch-Map Generator  │
│ (FileSystem API)  │     │  architecture_summary│
└───────────────────┘     │  .txt (minified)     │
        │                 └──────────────────────┘
        ▼
┌───────────────────┐     ┌──────────────────────┐
│  Git-Aware Filter │────▶│  Only changed files  │
│  (git diff HEAD)  │     │  go into the prompt  │
└───────────────────┘     └──────────────────────┘
        │
        ▼
┌───────────────────┐     ┌──────────────────────┐
│   PII Stripper    │────▶│  API keys, DB strings│
│   (local scan)    │     │  replaced with mocks │
└───────────────────┘     └──────────────────────┘
        │
        ▼
┌───────────────────┐
│  Token Counter    │
│  (gpt-tokenizer)  │
│  Pre-flight check │
└───────────────────┘
```

**Key capabilities:**
- `window.showDirectoryPicker()` — secure, one-click folder binding
- Arch-Map generation: lightweight project tree with key imports extracted, fed to AI first to minimize token usage
- **Git-aware context** (unique): reads `git diff HEAD` to send only what changed — reduces token usage by ~90% vs full codebase
- **Dependency graph** (unique): parses `import/require` statements to understand which files depend on each other, providing richer structural context
- Smart chunking: splits large files into semantically-aware segments that preserve function/class boundaries
- Local PII stripping: scans for patterns matching API keys (`sk-...`, `eyJ...`), Supabase URLs, database connection strings — replaces with `[REDACTED_API_KEY]` before any content is injected

---

### Pillar 2 — Live Hybrid Bridge & Context Injection

Scrapes live web content (documentation pages, GitHub repos, Supabase dashboards) and merges it with selected local files into a structured prompt — then injects it directly into the AI platform's input field.

**Injection targets (DOM selectors maintained per platform):**

| Platform | Input Selector | Method |
|---|---|---|
| Claude.ai | `div[contenteditable="true"]` | `innerHTML` + `dispatchEvent` |
| ChatGPT | `#prompt-textarea` | `value` setter + React synthetic event |
| Gemini | `.ql-editor` | Quill API or direct DOM mutation |

**Super-Prompt structure generated:**

```
=== ARCHITECTURE SUMMARY ===
[minified project tree + key imports]

=== LOCAL FILE: database.js (git diff only) ===
[only the lines that changed since last commit]

=== WEB CONTEXT: [page title + URL] ===
[scraped documentation content, sanitized]

=== TASK ===
[user's instruction]
```

**Context Versioning** (unique): every prompt sent is saved locally in `IndexedDB` with a timestamp and the AI response. Works like git for your AI conversations — you can restore any previous context and continue from there.

---

### Pillar 3 — Autonomous Browser Agent

Accepts natural language multi-step tasks and executes them autonomously across browser tabs. Built on top of [Nanobrowser](https://github.com/nanobrowser/nanobrowser)'s multi-agent architecture.

**Agent system:**

```
User Prompt: "Search Supabase v2 docs, cross-reference my local
             database.js, then refactor it"
                        │
                        ▼
              ┌─────────────────┐
              │  Planner Agent  │  Breaks task into steps,
              │  (high-context  │  self-corrects on failure
              │   LLM call)     │
              └────────┬────────┘
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
   ┌────────────┐ ┌──────────┐ ┌──────────┐
   │ Navigator  │ │Extractor │ │Injector  │
   │ Agent      │ │ Agent    │ │ Agent    │
   │(tab control│ │(scraping)│ │(prompt   │
   │ navigation)│ │          │ │injection)│
   └────────────┘ └──────────┘ └──────────┘
```

**Human-like interaction simulation:**
- Programmatic mouse paths with Bézier curves (not straight-line teleports)
- Smooth scroll with variable speed
- Random delays between actions (150–800ms, normally distributed)
- Keyboard typing simulation with per-character timing

**Multi-AI Consensus Mode** (unique): sends the same task to Claude, ChatGPT, and Gemini in parallel tabs, then aggregates responses side-by-side in the side panel. No other free tool offers this.

---

### Pillar 4 — Live Visual Overlay & Professional Deliverables

**Visual Overlay:**
- SVG-based overlay rendered on top of the active tab showing the agent's current step
- Glowing virtual cursor following the agent's programmatic mouse path
- Step counter, progress bar, and real-time status log
- Designed for demo recordings and product transparency

**Client-side export (zero server cost):**

| Output | Library | Contents |
|---|---|---|
| `Report_[timestamp].pdf` | `jsPDF` | Full-page screenshots, comparison tables, timestamped audit trail |
| `Refactored_[timestamp].zip` | `JSZip` | All modified source files, change log, original backups |

---

## 🚀 Unique Features (Not Available Elsewhere)

### 1. Git-Aware Context
Instead of sending entire files, reads `git diff HEAD` and sends only changed lines. Reduces token consumption by ~90% and focuses the AI on exactly what's relevant.

```javascript
// Pseudocode
const diff = await runGitCommand('git diff HEAD');
const changedFiles = parseDiff(diff); // Only files with changes
const context = buildContext(changedFiles, archMap);
// Result: 200 tokens instead of 20,000
```

### 2. Dependency Graph Context
Parses `import`/`require` statements across the project and builds a graph. When you select `auth.js`, the extension automatically includes snippets from `userModel.js` and `tokenService.js` because they're direct dependencies — without you specifying them.

### 3. Context Versioning
Every prompt + response pair is saved locally in `IndexedDB` with full metadata. Browse history, restore any past state, and continue a conversation exactly where you left off — even in a new browser session.

### 4. Multi-AI Consensus Mode
One task, three AIs, one panel. Sends your prompt simultaneously to Claude, ChatGPT, and Gemini. Displays responses side-by-side with visual diff highlighting where they disagree.

### 5. Live Documentation Watcher
Monitor a documentation URL (e.g., Supabase changelog, a library's GitHub releases page). When content changes, the extension notifies you and suggests which local files may need updating based on the diff.

---

## 🛠️ Tech Stack & Dependencies

### Core Runtime
| Technology | Role | Version |
|---|---|---|
| Chrome Extension Manifest V3 | Core architecture | MV3 |
| `chrome.sidePanel` | Persistent UI controller | Chrome 114+ |
| `chrome.tabs` | Tab management & navigation | - |
| `chrome.scripting` | Content script injection | - |
| `chrome.storage` | Local settings persistence | - |
| `IndexedDB` (native) | Context versioning & history | - |

### Web APIs (Native, Zero Cost)
| API | Role |
|---|---|
| `window.showDirectoryPicker()` | Secure local folder access |
| `FileSystemDirectoryHandle` | Directory traversal |
| `FileSystemFileHandle` | File reading |

### Client-Side Libraries (No Server Required)

```json
{
  "dependencies": {
    "gpt-tokenizer": "^2.x",
    "jspdf": "^2.x",
    "jszip": "^3.x"
  }
}
```

| Library | Purpose | Size |
|---|---|---|
| `gpt-tokenizer` | Local token counting + pre-flight billing estimation | ~50KB |
| `jsPDF` | Client-side PDF generation with screenshots | ~300KB |
| `JSZip` | Client-side ZIP packaging of refactored code | ~100KB |

### Development Tools
| Tool | Role |
|---|---|
| TypeScript | Type safety across all modules |
| `pnpm` | Package management (faster than npm) |
| Vite + CRXJS | Chrome extension bundler with hot reload |
| ESLint + Prettier | Code quality |

---

## 📦 Open-Source Foundations We Build On

This project is not built from scratch. We stand on the shoulders of three excellent open-source projects:

### 1. [Nanobrowser](https://github.com/nanobrowser/nanobrowser) — Browser Agent Engine
> ⭐ 13,000+ stars · Apache 2.0 · TypeScript

Provides the complete multi-agent browser automation system. We use its Planner + Navigator agent architecture and add the FileSystem context layer on top.

**What we take:** Agent orchestration, tab management, LLM integration layer, side panel shell.  
**What we add:** FileSystem binding, prompt injection into AI platforms, context chunking, git-awareness.

### 2. [BridgeContext](https://github.com/anavvathi/Bridgecontext) — Prompt Injection Engine
> MIT License · JavaScript

Provides battle-tested DOM injection logic for Claude.ai, ChatGPT, and Gemini — including handling React synthetic events and Quill editor quirks.

**What we take:** Platform-specific injection selectors, event dispatch logic, context capture from AI outputs.  
**What we add:** Local file context, structured super-prompt assembly, auto-retry on DOM changes.

### 3. [ExtensionOS](https://github.com/albertocubeddu/extensionOS) — Extension Shell & Prompt Factory
> MIT License · JavaScript

Provides a clean Manifest V3 shell with side panel, settings UI, and a prompt workflow system.

**What we take:** Extension boilerplate, side panel layout, prompt library UI pattern.  
**What we add:** File tree browser, token counter display, context versioning UI, overlay renderer.

---

## 📁 Project Structure

```
universal-context-bridge/
├── manifest.json                    # MV3 manifest with all permissions
├── src/
│   ├── background/
│   │   └── service-worker.ts        # Extension lifecycle, tab orchestration
│   ├── sidepanel/
│   │   ├── index.html               # Side panel entry point
│   │   ├── App.tsx                  # Root React component
│   │   └── components/
│   │       ├── FileTree.tsx         # Local directory browser
│   │       ├── ContextBuilder.tsx   # Prompt assembly UI
│   │       ├── AgentConsole.tsx     # Agent status + step log
│   │       ├── ConsensusView.tsx    # Multi-AI response comparison
│   │       └── HistoryPanel.tsx     # Context versioning browser
│   ├── content-scripts/
│   │   ├── claude-injector.ts       # Claude.ai DOM injection
│   │   ├── chatgpt-injector.ts      # ChatGPT DOM injection
│   │   └── gemini-injector.ts       # Gemini DOM injection
│   ├── core/
│   │   ├── filesystem/
│   │   │   ├── DirectoryReader.ts   # FileSystem Access API wrapper
│   │   │   ├── ArchMapGenerator.ts  # Project tree + key imports
│   │   │   ├── GitDiffReader.ts     # git diff HEAD parser
│   │   │   └── DependencyGraph.ts   # import/require graph builder
│   │   ├── context/
│   │   │   ├── ChunkManager.ts      # Smart semantic chunking
│   │   │   ├── PIIStripper.ts       # API key + credential sanitizer
│   │   │   ├── TokenCounter.ts      # gpt-tokenizer wrapper
│   │   │   └── ContextVersioner.ts  # IndexedDB history manager
│   │   ├── agent/
│   │   │   ├── Planner.ts           # High-level task decomposition
│   │   │   ├── Navigator.ts         # Tab navigation + interaction
│   │   │   ├── Extractor.ts         # Web content scraping
│   │   │   └── HumanSimulator.ts    # Mouse paths, scroll, typing
│   │   ├── bridge/
│   │   │   ├── SuperPromptBuilder.ts # Assembles final hybrid prompt
│   │   │   └── WebScraper.ts        # Live documentation scraping
│   │   └── export/
│   │       ├── PDFGenerator.ts      # jsPDF wrapper + screenshots
│   │       └── ZIPPackager.ts       # JSZip wrapper for code export
│   ├── overlay/
│   │   ├── overlay.ts               # SVG overlay renderer
│   │   └── cursor.ts                # Virtual cursor animation
│   └── utils/
│       ├── storage.ts               # chrome.storage wrapper
│       └── platform-detector.ts    # Detect active AI platform
├── public/
│   └── icons/                       # Extension icons (16, 48, 128px)
├── package.json
├── tsconfig.json
├── vite.config.ts
└── README.md
```

---

## 🗺️ Build Roadmap

### Phase 1 — Foundation (Weeks 1–3)
**Goal:** Working extension that reads local files and injects context into one AI platform.

- [ ] Initialize Manifest V3 with permissions: `sidePanel`, `tabs`, `scripting`, `storage`, `activeTab`
- [ ] Set up TypeScript + Vite + CRXJS build pipeline
- [ ] Build side panel shell (from ExtensionOS base)
- [ ] Implement `DirectoryReader.ts` using `FileSystem Access API`
- [ ] Build `ArchMapGenerator.ts` — project tree with import extraction
- [ ] Implement `PIIStripper.ts` — regex patterns for common credential formats
- [ ] Implement `TokenCounter.ts` — real-time token count display
- [ ] Build Claude.ai content script injector (start with one platform)
- [ ] End-to-end test: select folder → generate arch-map → inject into Claude

### Phase 2 — The Unique Value (Weeks 4–6)
**Goal:** Ship the features that no other tool has.

- [ ] Build `GitDiffReader.ts` — parse `git diff HEAD` output into file/line structure
- [ ] Build `DependencyGraph.ts` — AST-lite import parser (no full AST needed, regex-based)
- [ ] Build `ContextVersioner.ts` — IndexedDB schema + CRUD for prompt history
- [ ] Build `SuperPromptBuilder.ts` — structured prompt assembly with all context layers
- [ ] Add ChatGPT and Gemini injectors
- [ ] Build `ConsensusView.tsx` — parallel prompt dispatch + side-by-side response UI
- [ ] Implement `WebScraper.ts` — documentation page scraper + change detection

### Phase 3 — Agent & Exports (Weeks 7–10)
**Goal:** Full autonomous agent + professional output generation.

- [ ] Integrate Nanobrowser's Planner + Navigator agents
- [ ] Build `HumanSimulator.ts` — Bézier mouse paths, scroll simulation, typing delay
- [ ] Build SVG overlay renderer + virtual cursor animation
- [ ] Implement `PDFGenerator.ts` — jsPDF with full-page screenshots via `chrome.tabs.captureVisibleTab`
- [ ] Implement `ZIPPackager.ts` — JSZip with modified files + change log
- [ ] Build `AgentConsole.tsx` — real-time step-by-step agent status UI
- [ ] End-to-end test: multi-step autonomous task with PDF report export

### Phase 4 — Polish & Launch (Weeks 11–12)
**Goal:** Chrome Web Store submission ready.

- [ ] Live Documentation Watcher — URL monitoring + local file change suggestions
- [ ] Performance optimization: lazy loading, worker threads for heavy parsing
- [ ] Chrome Web Store assets: screenshots, promo video, description
- [ ] Security audit: CSP headers, permission minimization, data flow review
- [ ] Submit to Chrome Web Store

---

## 🎬 Use Case Scenarios

### Scenario A — Updating Local Code via Live Documentation

```
1. Developer opens newly released Supabase v2 docs in Chrome
2. Opens extension side panel → binds local project folder
3. Selects database.js from file tree
4. Extension runs git diff → finds 12 changed lines
5. Clicks "Bridge & Compare"
6. Extension scrapes the docs page + merges with the 12-line diff
7. Injects structured prompt into open Claude tab
8. Claude returns targeted refactoring — no irrelevant context
```

**Token usage:** ~800 tokens (vs ~15,000 for full file send)

---

### Scenario B — Autonomous Market Research

```
User types: "Browse Amazon and two hardware sites, extract
            4K monitor pricing, compile a comparison report"

Agent:
  Step 1 → Opens Amazon tab, navigates to 4K monitors category
  Step 2 → Smooth-scrolls product listing, extracts name/price/specs
  Step 3 → Repeats on BestBuy and Newegg
  Step 4 → Closes automated tabs
  Step 5 → Downloads Comparison_Report_2025-05-31.pdf + data.zip
```

---

### Scenario C — Multi-AI Code Review

```
1. Developer selects auth.js + its dependencies (auto-detected)
2. Types: "Review this authentication flow for security issues"
3. Clicks "Consensus Mode"
4. Extension opens Claude, ChatGPT, Gemini tabs in parallel
5. Injects identical prompt into all three
6. Side panel shows 3 responses with color-coded diff:
   - Green: all three agree
   - Yellow: two agree, one differs
   - Red: all three give different answers
```

---

## 🔧 Installation (Development)

### Prerequisites
- Node.js 18+
- pnpm (`npm install -g pnpm`)
- Chrome 114+ (for `chrome.sidePanel` API)
- Git (for git-diff features)

### Setup

```bash
# Clone the repository
git clone https://github.com/YOUR_USERNAME/universal-context-bridge.git
cd universal-context-bridge

# Install dependencies
pnpm install

# Start development build with hot reload
pnpm dev
```

### Load in Chrome

```
1. Open Chrome → chrome://extensions/
2. Enable "Developer mode" (top right toggle)
3. Click "Load unpacked"
4. Select the dist/ folder generated by pnpm dev
5. Pin the extension → click the icon → open Side Panel
```

### Environment

No API keys required in the extension itself. The extension injects prompts into AI platforms where the user is already logged in. If using agent features with direct API calls (optional), add your keys in the extension's Settings panel — they are stored locally in `chrome.storage.local` and never transmitted elsewhere.

---

## 🤝 Contributing

This project is in active development. Contributions welcome in these areas:

- **New platform injectors** — Perplexity, Mistral, Grok
- **Language parsers** — better dependency graph for Python, Rust, Go
- **Agent recipes** — pre-built automation workflows for common dev tasks
- **Test coverage** — unit tests for core parsing modules

Please read `CONTRIBUTING.md` before opening a PR.

---

## ⚠️ Legal & Ethical Notes

- This extension injects into AI platforms' web interfaces. It does not bypass authentication, rate limits, or any security mechanism — it only automates input into fields the user has access to.
- The "human-like simulation" features are intended for automation of tasks the user would perform manually. Use responsibly and in accordance with each platform's Terms of Service.
- The extension never stores or transmits your code to any third-party service. All processing is local.

---

## 📄 License

MIT License — see `LICENSE` for details.

Built on top of:
- [Nanobrowser](https://github.com/nanobrowser/nanobrowser) — Apache 2.0
- [BridgeContext](https://github.com/anavvathi/Bridgecontext) — MIT
- [ExtensionOS](https://github.com/albertocubeddu/extensionOS) — MIT

---

<p align="center">
  <strong>Built for developers who are tired of copy-pasting their codebase into a chat box.</strong>
</p>