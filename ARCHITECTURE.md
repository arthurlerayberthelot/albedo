# ALBEDO: System Architecture

Albedo is a brutalist, zero-dependency, local-first note-taking engine. It is designed as a single-file application (`albedo.html`), prioritizing portability, offline capability, and text-driven interactions over complex graphical interfaces.

---

## 🧠 Core Philosophy

1. **Portability over Tooling:** Zero build steps (no Webpack, Babel, or React). The application relies exclusively on native browser standards.
2. **Local-First (with Cloud Sync):** The user's device is the primary source of truth. Data is stored entirely in the browser, with explicit I/O mechanisms (JSON/MD exports and loads), and optional GitHub Gist synchronization via a "first in wins" strategy.
3. **Keyboard-Driven Flow:** The unified input bar (`#commander`) acts as a terminal for searching, filtering, and creating content.

---

## 🏗 System Components

The architecture is divided into three logical namespaces inside the root `sys` object:

| Namespace | Responsibility | Key Methods |
| :--- | :--- | :--- |
| **`sys`** (Core) | State management, parsing, rendering, data I/O | `init`, `add`, `update`, `remove`, `parse`, `format`, `render`, `wipe` |
| **`sys.api`** | All external network requests (async, isolated) | `fetchDefinition`, `fetchWikiSummary`, `fetchSources`, `fetchCoordinates`, `fetchMeteo`, `sync`, `generateHelp` |
| **`sys.events`** | DOM event listeners and UI wiring | `init`, `setupCommander`, `setupFileUploader` |
| **`db`** | IndexedDB persistence wrapper | `init`, `getAll`, `put`, `delete`, `clear` |

---

## 💾 Data Model

The atomic unit of Albedo is a **Block**. Every block is parsed upon creation or modification to extract metadata.

```javascript
{
  id: String,        // Unique identifier (Base36 + Math.random)
  ts: Number,        // Unix timestamp of creation
  isPinned: Boolean, // UI state for prioritized sorting
  raw: String,       // The raw string input by the user
  color: String,     // Extracted from '!color' syntax
  tags: Array,       // Extracted from '#tag' syntax
  patterns: Array,   // Extracted from '~key:value' syntax for sorting
  body: String       // Pure text content, stripped of metadata tokens
}
```

Metadata about the notebook itself is stored in a separate `meta` record in IndexedDB:

```javascript
{
  id: 'primary',
  name: String,       // Display name of the notebook
  theme: String,      // 'auto' | 'dark' | 'light'
  gistToken: String,  // GitHub Personal Access Token (optional)
  gistId: String,     // GitHub Gist ID (optional)
  isDirty: Boolean,   // Dirty flag for sync conflict resolution
  lastGistDate: String // ISO timestamp of last successful sync
}
```

---

## ⚙️ Core Mechanisms

### 1. The Parser Pipeline

When data enters via `sys.add()` or `sys.update()`, it runs through the `sys.parse()` tokenizer. The parser sequentially strips Albedo-specific tokens from the front of the string (`#tag`, `!color`, `~key:value`) before pushing the remaining text into `body`.

During rendering, `sys.format()` processes the body text, converting a restricted subset of Markdown into safe HTML, applying the search highlight engine, and rewriting Wikimedia Commons URLs for auto-thumbnailing.

**Input guard:** `sys.add()` performs an early `trim()` check and rejects empty input before touching the database.

### 2. The Commander Engine

The input bar dispatches to different modes based on string prefixes:

| Prefix | Mode | Description |
| :--- | :--- | :--- |
| *(none)* | **Insert** | Parses text into a new block |
| `?` | **Search** | Fuzzy text filter with Boolean operators (`AND`, `OR`, `NOT`) |
| `?~` | **Sort** | Filters and sorts by pattern key (e.g. `?~ d date`) |
| `&` | **Command** | Routes to `sys.api` external fetchers |

The commander uses a shared `resetCommander()` helper to clear input, reset height, dismiss autocomplete, and clear the active suggestion — applied uniformly across all command branches.

### 3. External API Integrations (`sys.api`)

All external network calls are isolated in `sys.api`. They are async, append new blocks to the state upon resolution, and fire `sys.notify()` to provide user feedback at each step.

| Command | API | Notes |
| :--- | :--- | :--- |
| `&dic: mot` | French Wiktionary REST API | `DOMParser` HTML extraction |
| `&wiki: sujet` | Wikipedia REST API (fr) | Plain text summary |
| `&source: sujet` | Crossref Metadata API | Academic literature citations |
| `&meteo: Lieu YYYY-MM-DD` | Open-Meteo (Geocoding + Archive) | Two-step: coordinates → archive |
| `&github: TOKEN ID` | GitHub API | Stores credentials in `sys.meta` |
| `&sync:` | GitHub Gist API | Pull if remote changed, push if local dirty |
| `&help` | *(static)* | Injects a formatted cheatsheet block |

### 4. Theme System

Albedo supports three display modes: `auto`, `dark`, and `light`, cycled via the `Theme` button.

- `sys.meta.theme` persists the current preference in IndexedDB.
- `sys.applyTheme()` resolves the effective mode (respecting OS `prefers-color-scheme` in `auto`), and sets the `data-theme` attribute on `<html>`.
- `sys.events` listens for OS theme changes and calls `applyTheme()` if mode is `auto`.
- Dark mode overrides are defined via `html[data-theme="dark"]` CSS variables, completely decoupled from the `@media` query.

---

## 🛡️ Performance & Security

### Performance
- **Lazy Loading:** `renderMore()` batches DOM creation using `DocumentFragment` and an `IntersectionObserver`, supporting large notebooks without blocking the main thread.
- **Render Debouncing:** `requestRender()` coalesces rapid search input events behind a 150ms debounce.
- **Stats Efficiency:** `updateStats()` uses `TextEncoder.encode().length` instead of `Blob` allocation for byte counting.
- **Toast Debouncing:** `notify()` clears any pending hide-timer before displaying a new message, preventing rapid-fire calls from overlapping.

### Security
- **Content-Security-Policy:** All permitted fetch origins are explicitly enumerated in the CSP `connect-src` header. No wildcard origins.
- **HTML Sanitization:** `sys.escape()` is applied before any Markdown processing in `sys.format()`, preventing XSS injection via raw block content.
- **URL Validation:** Markdown hyperlinks are validated against an allowlist of schemes (`http://`, `https://`, `mailto:`); other values fall back to `#`.
- **Cartridge Validation:** `loadCartridge()` validates that every imported block contains the minimum required fields (`id: string`, `ts: number`, `raw: string`) before committing to state.
- **Credentials:** `gistToken` is stored in IndexedDB (client-side only, single-user by design). It is never logged or transmitted except to `api.github.com`.

---

## ⚠️ Constraints & Limitations

- **Memory Footprint:** The entire database is loaded into `sys.state` (RAM) on initialization. This is by design for synchronous read performance, but could be a bottleneck for exceptionally large notebooks (> ~10,000 blocks).
- **Single-User Sync:** The GitHub Gist sync strategy is "first in wins" with no merge conflict UI. This is a conscious simplification for a single-user context.
- **`&news` dropped:** The planned French public domain press feature (`&news`) was researched and prototyped, but the Gallica SRU API (BnF) does not support browser-side CORS fetch requests (returns `403`). The feature remains viable server-side or via a proxy.

---

## 🧪 Testing Strategy (Zero-Dependency)

Because Albedo strictly avoids external dependencies and build tools, a traditional Node.js testing setup is intentionally omitted.

**Recommended Approach:**
1. **In-Browser Test Harness:** Create a separate `tests.html` file. It can fetch `albedo.html`, extract the `<script>` tag content, and evaluate the `sys` object in an isolated context.
2. **Pure Function Unit Testing:** Native `console.assert()` can be used inside `tests.html` to validate the pure functions: `sys.parse()`, `sys.format()`, and the Boolean search logic.
3. **Automated End-to-End:** Utilize the `user-tester` (from `AGENTS.md`) using browser-automation to systematically exercise the UI, specifically targeting edge cases in the parser, memory constraints, and asynchronous sync workflows.
4. **Mocking:** `db` (IndexedDB layer) and `sys.api` (network layer) can be easily mocked in the test harness to simulate database corruption, network latency, or GitHub API rate limits.
