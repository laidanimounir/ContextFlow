# 🧠 ContextFlow — Local Codebase × Free AI, Bridged in Your Browser

> **Connect your local codebase to Claude, ChatGPT, and Gemini — no backend, no subscription, no copy-paste.**

A production-grade Chrome Extension (Manifest V3) that reads your local project files directly from the browser side panel and injects structured, git-aware context into free AI chat interfaces. Built on proven open-source foundations, extended with capabilities that exist nowhere else.

---

## 📋 Table of Contents

- [The Problem We Solve](#-the-problem-we-solve)
- [What Makes This Different](#-what-makes-this-different)
- [Architecture Overview](#-architecture-overview)
- [The 4 Core Pillars](#-the-4-core-pillars)
- [Unique Features](#-unique-features)
- [Tech Stack & Dependencies](#-tech-stack--dependencies)
- [Open-Source Foundations We Build On](#-open-source-foundations-we-build-on)
- [Project Structure](#-project-structure)
- [Build Roadmap](#-build-roadmap)
- [Use Case Scenarios](#-use-case-scenarios)
- [Installation (Development)](#-installation-development)
- [Future Vision](#-future-vision)
- [Contributing](#-contributing)
- [Legal & Ethical Notes](#-legal--ethical-notes)
- [License](#-license)

---

## 🔥 The Problem We Solve

Every developer using AI in 2025 faces the same friction loop:

1. Open Claude or ChatGPT in the browser
2. Manually copy hundreds of lines of code from the editor
3. Paste, re-explain the project context, wait for response
4. Copy the response back, apply it manually
5. Repeat — for every file, every question, every session

Existing solutions either require a paid IDE plugin (Cursor, GitHub Copilot at $10–19/month), a $200/month subscription (Claude Max for browser agent), or they only bridge AI-to-AI conversations without ever touching local files.

**ContextFlow eliminates that entire loop — for free.**

---

## ✨ What Makes This Different

| Feature | ContextFlow | BridgeContext | Nanobrowser | Cursor/Copilot |
|---|---|---|---|---|
| Local filesystem access (no IDE needed) | ✅ | ❌ | ❌ | ✅ IDE only |
| Injects context into free AI web UIs | ✅ | ✅ | ❌ | ❌ |
| **Git-aware context (diffs only)** | ✅ | ❌ | ❌ | ❌ |
| **Simplified dependency awareness** | ✅ | ❌ | ❌ | ❌ |
| **Context versioning (prompt history)** | ✅ | ❌ | ❌ | ❌ |
| **Doc change watcher** | ✅ | ❌ | ❌ | ❌ |
| Autonomous browser agent | ✅ | ❌ | ✅ | ❌ |
| Zero backend / zero server cost | ✅ | ✅ | ✅ | ❌ |
| PII stripping before AI upload | ✅ | ❌ | ❌ | ❌ |
| PDF/ZIP export (client-side) | ✅ | ❌ | ❌ | ❌ |

---

## 🏗️ Architecture Overview

```
┌──────────────────────────────────────────────────────────────┐
│                    Chrome Side Panel (UI)                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────────┐  │
│  │ Context  │  │  Bridge  │  │  Agent   │  │  Exports   │  │
│  │ Engine   │  │ & Inject │  │ Control  │  │  PDF / ZIP │  │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────────────┘  │
└───────┼─────────────┼─────────────┼───────────────────────  ┘
        │             │             │
        ▼             ▼             ▼
┌─────────────┐  ┌───────────────┐  ┌──────────────────────┐
│ FileSystem  │  │ Content       │  │ Tab Orchestration    │
│ Access API  │  │ Scripts       │  │ chrome.tabs +        │
│ (local disk)│  │ (DOM inject)  │  │ chrome.scripting     │
└─────────────┘  └──────┬────────┘  └──────────────────────┘
                         │
                 ┌───────▼────────┐
                 │  AI Platforms  │
                 │  Claude.ai     │
                 │  ChatGPT       │
                 │  Gemini        │
                 └────────────────┘
```

Everything runs client-side. No data leaves the machine except through the AI platform's own input field — which the user controls explicitly.

---

## 🧱 The 4 Core Pillars

### Pillar 1 — Local Context Engine & Smart Chunking

Reads and processes your local codebase directly from the browser side panel using the native `FileSystem Access API`. No file uploads, no server, no IDE required.

**Processing pipeline:**

```
Local Project Folder
        │
        ▼
┌──────────────────────┐     ┌─────────────────────────┐
│  Directory Scanner   │────▶│   Arch-Map Generator    │
│  (FileSystem API)    │     │   architecture_summary  │
│                      │     │   .txt  (minified tree) │
└──────────────────────┘     └─────────────────────────┘
        │
        ▼
┌──────────────────────┐     ┌─────────────────────────┐
│  Git-Aware Filter    │────▶│  Only changed lines go  │
│  (git diff HEAD)     │     │  into the prompt        │
└──────────────────────┘     └─────────────────────────┘
        │
        ▼
┌──────────────────────┐     ┌─────────────────────────┐
│  Import Scanner      │────▶│  "auth.js is imported   │
│  (shallow resolver)  │     │   by 4 other files"     │
└──────────────────────┘     └─────────────────────────┘
        │
        ▼
┌──────────────────────┐     ┌─────────────────────────┐
│  PII Stripper        │────▶│  API keys, DB strings   │
│  (local regex scan)  │     │  → [REDACTED]           │
└──────────────────────┘     └─────────────────────────┘
        │
        ▼
┌──────────────────────┐
│  Token Counter       │
│  (gpt-tokenizer)     │
│  Pre-flight display  │
└──────────────────────┘
```

**Key capabilities:**

- `window.showDirectoryPicker()` — secure one-click folder binding, no upload
- Arch-Map generation: lightweight project tree with top-level imports, fed to the AI first to orient it before the actual code
- Git-aware context: reads `git diff HEAD`, sends only changed lines — reduces token consumption by ~90% vs full file
- Shallow import scanner: detects which files import the selected file and surfaces that as one-line context note ("imported by: userController.js, authMiddleware.js")
- Smart chunking: splits large files at function/class boundaries, never mid-block
- PII stripping: matches patterns for `sk-...`, `eyJ...`, Supabase URLs, connection strings — replaced with `[REDACTED_API_KEY]` locally before injection

---

### Pillar 2 — Live Hybrid Bridge & Context Injection

Scrapes live web content (documentation pages, GitHub repos, changelogs) and merges it with local file context into a structured super-prompt — injected directly into the AI platform's input field.

**Injection targets (DOM selectors per platform):**

| Platform | Selector | Injection method |
|---|---|---|
| Claude.ai | `div[contenteditable="true"]` | `innerHTML` + `dispatchEvent` |
| ChatGPT | `#prompt-textarea` | `value` setter + React synthetic event |
| Gemini | `.ql-editor` | Quill API or direct DOM mutation |

**Super-Prompt format:**

```
=== ARCHITECTURE SUMMARY ===
[minified project tree + top-level imports]

=== LOCAL FILE: database.js ===
[git diff only — lines changed since last commit]
[note: imported by userController.js, authMiddleware.js]

=== WEB CONTEXT: [page title] ([URL]) ===
[scraped documentation content, sanitized]

=== TASK ===
[user's natural language instruction]
```

**Context Versioning:** every prompt sent is saved in `IndexedDB` with a timestamp and the full AI response text. Browse history, restore any previous state, and continue a conversation from any past point — even in a new browser session.

**Doc Change Watcher:** register any documentation URL (e.g. a library's changelog or GitHub releases page). The extension polls it on a configurable interval and notifies you when content changes — so you know when to re-bridge your local files.

---

### Pillar 3 — Autonomous Browser Agent

Accepts natural language multi-step tasks and executes them across browser tabs autonomously. Built on top of [Nanobrowser](https://github.com/nanobrowser/nanobrowser)'s multi-agent architecture.

**Agent system:**

```
User: "Search Supabase v2 docs, cross-reference my local
       database.js, then refactor it"
                    │
                    ▼
          ┌─────────────────┐
          │  Planner Agent  │  Decomposes task into steps.
          │  (LLM call)     │  Self-corrects on failure.
          └────────┬────────┘
                   │
      ┌────────────┼────────────┐
      ▼            ▼            ▼
┌──────────┐ ┌──────────┐ ┌──────────┐
│Navigator │ │Extractor │ │Injector  │
│Agent     │ │Agent     │ │Agent     │
│(tab nav) │ │(scraping)│ │(prompt   │
│          │ │          │ │injection)│
└──────────┘ └──────────┘ └──────────┘
```

**Human-like interaction simulation:**
- Mouse paths follow Bézier curves — no straight-line teleports
- Smooth variable-speed scrolling
- Random inter-action delays: 150–800 ms, normally distributed
- Per-character keyboard timing simulation

---

### Pillar 4 — Live Visual Overlay & Professional Deliverables

**Visual Overlay:**
- SVG overlay rendered on the active tab showing each agent step in real time
- Glowing virtual cursor tracking the agent's programmatic mouse path
- Step counter, progress bar, live status log in the side panel
- Designed for transparent demos and screen recordings

**Client-side export (zero server cost):**

| Output | Library | Contents |
|---|---|---|
| `Report_[timestamp].pdf` | `jsPDF` | Full-page screenshots, data tables, timestamped audit trail |
| `Refactored_[timestamp].zip` | `JSZip` | All modified source files + change log + originals |

---

## 🚀 Unique Features

### 1 — Git-Aware Context *(Phase 1)*

Reads `git diff HEAD` and sends only the lines that changed since the last commit. The AI sees exactly what's relevant — nothing more.

```
Without Git-Aware:   entire database.js   →  ~18,000 tokens
With Git-Aware:      12 changed lines     →  ~200 tokens
Savings:             ~98%
```

Implementation: parse unified diff output, extract per-file hunks, wrap in structured prompt block. Simple text processing — no binary dependencies.

---

### 2 — Shallow Import Awareness *(Phase 2)*

Scans `import`/`require` statements across the project with a single regex pass — no full AST needed. When you select a file, the extension adds a one-line note listing which other files import it. The AI understands the blast radius of any change without you explaining it.

```javascript
// What gets added automatically to the prompt:
// [context: database.js is imported by userController.js,
//  authMiddleware.js, seedScript.js]
```

No complex graph rendering. One line of context, big jump in AI answer quality.

---

### 3 — Context Versioning *(Phase 2)*

Every prompt + AI response is saved locally in `IndexedDB` with full metadata. Browse the history panel, click any past entry, and resume exactly from that state — same files, same context, same AI conversation thread.

Works like `git log` for your AI sessions. No cloud sync, no account needed.

---

### 4 — Doc Change Watcher *(Phase 2 — simplified)*

Register any URL in the side panel. The extension polls it via `chrome.alarms` on a user-set interval (hourly, daily). When the page content changes, you get a badge notification. One click opens the diff and re-triggers the bridge with updated web context.

No AI analysis of which files are affected — just reliable notification that the docs changed. You decide what to do next.

---

## 🛠️ Tech Stack & Dependencies

### Core Runtime

| Technology | Role | Requirement |
|---|---|---|
| Chrome Extension Manifest V3 | Core architecture | Chrome 114+ |
| `chrome.sidePanel` | Persistent UI controller | Chrome 114+ |
| `chrome.tabs` | Tab management & navigation | — |
| `chrome.scripting` | Content script injection | — |
| `chrome.alarms` | Doc watcher polling | — |
| `chrome.storage.local` | Settings + API key storage | — |
| `IndexedDB` (native) | Context versioning & history | — |

### Native Web APIs (Zero Cost)

| API | Role |
|---|---|
| `window.showDirectoryPicker()` | Secure local folder binding |
| `FileSystemDirectoryHandle` | Recursive directory traversal |
| `FileSystemFileHandle` | Individual file reading |

### Client-Side Libraries

```json
{
  "dependencies": {
    "gpt-tokenizer": "^2.x",
    "jspdf":         "^2.x",
    "jszip":         "^3.x"
  },
  "devDependencies": {
    "typescript":    "^5.x",
    "vite":          "^5.x",
    "@crxjs/vite-plugin": "^2.x"
  }
}
```

| Library | Purpose | Bundle size |
|---|---|---|
| `gpt-tokenizer` | Local token counting, pre-flight estimation | ~50 KB |
| `jsPDF` | Client-side PDF with screenshots | ~300 KB |
| `JSZip` | Client-side ZIP packaging | ~100 KB |

### Development Tools

| Tool | Role |
|---|---|
| TypeScript 5 | Type safety across all modules |
| pnpm | Package management |
| Vite + CRXJS | Chrome extension bundler with hot reload |
| ESLint + Prettier | Code quality |

---

## 📦 Open-Source Foundations We Build On

This project does not start from scratch. Three existing open-source projects provide the heavy lifting:

### 1. [Nanobrowser](https://github.com/nanobrowser/nanobrowser)
> ⭐ 13,000+ stars · Apache 2.0 · TypeScript

Provides the complete multi-agent browser automation system: Planner agent, Navigator agent, LLM provider abstraction, side panel shell.

**What we take:** Agent orchestration, tab management, LLM integration layer, extension shell.
**What we add:** FileSystem binding, super-prompt assembly, git-diff context, injection into AI platforms.

---

### 2. [BridgeContext](https://github.com/anavvathi/Bridgecontext)
> MIT License · JavaScript

Provides battle-tested DOM injection logic for Claude.ai, ChatGPT, and Gemini — including React synthetic event dispatch and Quill editor handling.

**What we take:** Platform-specific selectors, injection event logic, output capture from AI response fields.
**What we add:** Local file context assembly, structured super-prompt format, auto-retry on DOM changes.

---

### 3. [ExtensionOS](https://github.com/albertocubeddu/extensionOS)
> MIT License · JavaScript

Provides a clean Manifest V3 shell with side panel layout, settings UI, and a prompt workflow builder.

**What we take:** Extension boilerplate, side panel layout, prompt library UI pattern.
**What we add:** File tree browser, token counter, context versioning panel, overlay renderer.

---

## 📁 Project Structure

```
contextflow/
├── manifest.json
├── src/
│   ├── background/
│   │   └── service-worker.ts        # Extension lifecycle, alarms, tab orchestration
│   ├── sidepanel/
│   │   ├── index.html
│   │   ├── App.tsx
│   │   └── components/
│   │       ├── FileTree.tsx          # Local directory browser
│   │       ├── ContextBuilder.tsx    # Prompt assembly + token display
│   │       ├── AgentConsole.tsx      # Real-time agent step log
│   │       ├── HistoryPanel.tsx      # Context versioning browser
│   │       └── DocWatcher.tsx        # Registered URLs + change notifications
│   ├── content-scripts/
│   │   ├── claude-injector.ts        # Claude.ai DOM injection
│   │   ├── chatgpt-injector.ts       # ChatGPT DOM injection
│   │   └── gemini-injector.ts        # Gemini DOM injection
│   ├── core/
│   │   ├── filesystem/
│   │   │   ├── DirectoryReader.ts    # FileSystem Access API wrapper
│   │   │   ├── ArchMapGenerator.ts   # Project tree + import extraction
│   │   │   ├── GitDiffReader.ts      # git diff HEAD parser
│   │   │   └── ImportScanner.ts      # Shallow import/require resolver
│   │   ├── context/
│   │   │   ├── ChunkManager.ts       # Function/class boundary chunking
│   │   │   ├── PIIStripper.ts        # Credential sanitizer
│   │   │   ├── TokenCounter.ts       # gpt-tokenizer wrapper
│   │   │   └── ContextVersioner.ts   # IndexedDB prompt history
│   │   ├── agent/
│   │   │   ├── Planner.ts            # Task decomposition (from Nanobrowser)
│   │   │   ├── Navigator.ts          # Tab navigation (from Nanobrowser)
│   │   │   ├── Extractor.ts          # Web content scraping
│   │   │   └── HumanSimulator.ts     # Bézier mouse, scroll, typing
│   │   ├── bridge/
│   │   │   ├── SuperPromptBuilder.ts # Final prompt assembly
│   │   │   ├── WebScraper.ts         # Documentation scraper
│   │   │   └── DocWatcherEngine.ts   # Polling + diff detection
│   │   └── export/
│   │       ├── PDFGenerator.ts       # jsPDF + captureVisibleTab
│   │       └── ZIPPackager.ts        # JSZip code packager
│   ├── overlay/
│   │   ├── overlay.ts                # SVG overlay renderer
│   │   └── cursor.ts                 # Virtual cursor animation
│   └── utils/
│       ├── storage.ts                # chrome.storage wrapper
│       └── platform-detector.ts     # Detect current AI platform
├── public/
│   └── icons/                        # 16px, 48px, 128px
├── package.json
├── tsconfig.json
├── vite.config.ts
└── README.md
```

---

## 🗺️ Build Roadmap

### Phase 1 — Foundation (Weeks 1–3)
**Goal:** Working extension that reads local files and injects git-aware context into Claude.

- [ ] Initialize Manifest V3 — permissions: `sidePanel`, `tabs`, `scripting`, `storage`, `activeTab`, `alarms`
- [ ] Set up TypeScript + Vite + CRXJS build pipeline
- [ ] Build side panel shell (based on ExtensionOS)
- [ ] Implement `DirectoryReader.ts` — `FileSystem Access API` wrapper with error handling
- [ ] Build `ArchMapGenerator.ts` — tree + top-level import extraction
- [ ] Build `GitDiffReader.ts` — parse `git diff HEAD` into per-file hunk structure
- [ ] Implement `PIIStripper.ts` — regex patterns: `sk-`, `eyJ`, Supabase URLs, connection strings
- [ ] Implement `TokenCounter.ts` — real-time count displayed in side panel
- [ ] Build `claude-injector.ts` — content script for Claude.ai DOM injection
- [ ] Build `SuperPromptBuilder.ts` — assemble arch-map + git diff + task into structured prompt
- [ ] End-to-end test: select folder → git diff → inject into Claude → verify response

---

### Phase 2 — Unique Value (Weeks 4–6)
**Goal:** Ship the four features that no other free tool has.

- [ ] Build `ImportScanner.ts` — single regex pass over project, build reverse import map
- [ ] Build `ContextVersioner.ts` — IndexedDB schema: `{ id, timestamp, prompt, response, files[] }`
- [ ] Build `HistoryPanel.tsx` — browse past sessions, restore state with one click
- [ ] Build `DocWatcherEngine.ts` — `chrome.alarms` polling, SHA hash comparison for change detection
- [ ] Build `DocWatcher.tsx` — UI to register/manage watched URLs + notification badge
- [ ] Add `chatgpt-injector.ts` and `gemini-injector.ts`
- [ ] Build `WebScraper.ts` — documentation page content extraction, sanitized for prompt injection
- [ ] End-to-end test: all four unique features working independently

---

### Phase 3 — Agent & Exports (Weeks 7–10)
**Goal:** Full autonomous multi-step agent + professional deliverable export.

- [ ] Integrate Nanobrowser Planner + Navigator agents into extension
- [ ] Build `HumanSimulator.ts` — Bézier mouse path generator, variable scroll, per-char typing delay
- [ ] Build SVG overlay + virtual cursor animation (`overlay.ts`, `cursor.ts`)
- [ ] Implement `PDFGenerator.ts` — jsPDF + `chrome.tabs.captureVisibleTab` for screenshots
- [ ] Implement `ZIPPackager.ts` — JSZip with modified files, change log, and originals
- [ ] Build `AgentConsole.tsx` — real-time step log with success/failure indicators
- [ ] End-to-end test: multi-step autonomous task → PDF export with screenshots

---

### Phase 4 — Polish & Launch (Weeks 11–12)
**Goal:** Chrome Web Store submission ready.

- [ ] Performance pass: lazy-load heavy modules, offload parsing to Web Workers
- [ ] Permissions minimization audit — request only what each feature actually needs
- [ ] CSP headers review across all content scripts
- [ ] Chrome Web Store listing: screenshots, 30-second demo video, description copy
- [ ] Submit to Chrome Web Store

---

## 🎬 Use Case Scenarios

### Scenario A — Update Local Code via Live Documentation

```
1. Developer opens Supabase v2 migration guide in Chrome
2. Opens ContextFlow side panel → binds local project folder
3. Selects database.js from the file tree
4. Extension runs git diff → finds 12 changed lines
5. Clicks "Bridge & Inject"
6. Extension scrapes the docs page + merges with 12-line diff
7. Injects structured super-prompt into open Claude tab
8. Claude returns a targeted refactor — no irrelevant context

Token usage:  ~800  (vs ~18,000 for the full file)
```

---

### Scenario B — Autonomous Research & Report

```
User types: "Browse three hardware sites, extract 4K monitor
            pricing and specs, compile a comparison PDF"

Agent:
  Step 1 → Opens Amazon, navigates to 4K monitors
  Step 2 → Smooth-scrolls listing, extracts name / price / specs
  Step 3 → Repeats on BestBuy and Newegg
  Step 4 → Closes automated tabs
  Step 5 → Downloads Comparison_Report_2025-05-31.pdf
           and data_2025-05-31.zip
```

---

### Scenario C — Resume a Past Session

```
1. Developer opens ContextFlow, clicks "History"
2. Finds session from 3 days ago: "refactor auth flow"
3. Clicks "Restore" — side panel reloads exact file selection,
   arch-map, and git diff from that session
4. Injects into Claude and continues the conversation
   exactly where it was left
```

---

## 🔧 Installation (Development)

### Prerequisites

- Node.js 18+
- pnpm (`npm install -g pnpm`)
- Chrome 114+ (required for `chrome.sidePanel`)
- Git installed and accessible in terminal (for git-diff features)

### Setup

```bash
git clone https://github.com/YOUR_USERNAME/contextflow.git
cd contextflow
pnpm install
pnpm dev          # hot-reload development build → dist/
```

### Load in Chrome

```
1. Open chrome://extensions/
2. Enable "Developer mode" (top-right toggle)
3. Click "Load unpacked" → select the dist/ folder
4. Pin the extension → click icon → open Side Panel
```

### API Keys (Optional)

No API keys are required for the core features — the extension injects prompts into AI platforms where the user is already logged in. If using the autonomous agent with direct API calls (faster, no rate-limit risk), add keys in the Settings panel. They are stored only in `chrome.storage.local` and never transmitted to any third party.

---

## 🔭 Future Vision

These capabilities are planned for after the initial launch — they require additional research or carry higher implementation complexity.

**Multi-AI Consensus Mode**
Send the same prompt to Claude, ChatGPT, and Gemini simultaneously and display responses side-by-side with color-coded agreement/disagreement highlights. High value for architectural decisions. Deferred because maintaining DOM injectors for three platforms in parallel increases fragility — will be tackled once the single-platform injection layer is stable and versioned.

**Mobile Agent Bridge**
Use the extension as a coordinator that sends tasks to a companion Android app running a vision-based agent (Accessibility Service or screenshot-based). The phone becomes an agent node controlled from the Desktop side panel. Research phase — no open-source foundation ready for this integration today.

**VS Code Companion Extension**
Write changes suggested by the AI directly back to local files via a VS Code extension that connects to ContextFlow over Native Messaging. Bidirectional loop: file → AI → suggestion → apply.

---

## 🤝 Contributing

Active development in progress. Contributions welcome in:

- **Platform injectors** — Perplexity, Mistral, Grok support
- **Language parsers** — import scanning for Python, Rust, Go
- **Agent recipes** — pre-built task templates for common dev workflows
- **Test coverage** — unit tests for `GitDiffReader`, `ImportScanner`, `PIIStripper`

Please read `CONTRIBUTING.md` before opening a PR.

---

## ⚠️ Legal & Ethical Notes

- ContextFlow injects text into AI platform input fields via content scripts. It does not bypass authentication, circumvent rate limits, or exploit any security mechanism.
- Human-like simulation is intended to replicate what the user would do manually. Use responsibly and in accordance with each platform's Terms of Service.
- All file processing is local. No code, credentials, or project data are transmitted to any service other than the AI platform the user explicitly chooses.

---

## 📄 License

MIT License — see `LICENSE` for details.

Built on top of:
- [Nanobrowser](https://github.com/nanobrowser/nanobrowser) — Apache 2.0
- [BridgeContext](https://github.com/anavvathi/Bridgecontext) — MIT
- [ExtensionOS](https://github.com/albertocubeddu/extensionOS) — MIT

---

<p align="center">
  <strong>ContextFlow — Built for developers who are tired of copy-pasting their codebase into a chat box.</strong>
</p>