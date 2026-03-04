# ALBEDO: System Architecture

Albedo is a brutalist, zero-dependency, local-first note-taking engine. It is designed as a single-file application (`index.html`), prioritizing portability, offline capability, and text-driven interactions over complex graphical interfaces.



## 🧠 Core Philosophy
1.  **Portability over Tooling:** Zero build steps (no Webpack, Babel, or React). The application relies exclusively on native browser standards.
2.  **Local-First:** The user's device is the primary source of truth. Data is stored entirely in the browser, with explicit, manual I/O mechanisms (JSON/MD export and load).
3.  **Keyboard-Driven Flow:** The unified input bar (`#commander`) acts as a terminal for searching, filtering, and creating content.

## 🏗 System Components

The architecture is broadly divided into three logical layers contained within the `sys` object:

| Layer | Responsibility | Implementation Details |
| :--- | :--- | :--- |
| **Storage (DB)** | Persistence | IndexedDB via a lightweight Promise-based wrapper. Contains two stores: `blocks` (notes) and `meta` (notebook state). |
| **Engine (Core)** | State & Parsing | Maintains an in-memory array (`sys.state`) for synchronous reads. Parses a custom syntax (tags, colors, patterns, and markdown). |
| **View (UI)** | Rendering | Vanilla DOM manipulation using DocumentFragments. Reactive to `sys.state` changes. |

## 💾 Data Model

The atomic unit of Albedo is a **Block**. Every block is parsed upon creation or modification to extract metadata.

```javascript
{
  id: String,       // Unique identifier (Base36 + Math.random)
  ts: Number,       // Unix timestamp of creation
  isPinned: Boolean,// UI state for prioritized sorting
  raw: String,      // The raw string input by the user
  color: String,    // Extracted from '!color' syntax
  tags: Array,      // Extracted from '#tag' syntax
  patterns: Array,  // Extracted from '~key:value' syntax for sorting
  body: String      // The pure text content, stripped of metadata tokens
}

```

## ⚙️ Core Mechanisms

### 1. The Parser pipeline

When data enters via `sys.add()` or `sys.update()`, it runs through a tokenizer (`sys.parse()`). The parser sequentially extracts Albedo-specific syntax from the front of the string (e.g., `#idea !blue ~status:wip`) before pushing the remaining string into the text body.

During rendering, `sys.format()` processes the body text, converting a restricted subset of Markdown into safe HTML strings, handling syntax highlighting, and rewriting Wikimedia Commons URLs for optimized thumbnail fetching.

### 2. The Commander Engine

The top input bar handles multiple operational modes detected via string prefixes:

* **Insert Mode (Default):** Text is parsed into a new block.
* **Search Mode (`?`):** Triggers a fuzzy text search across all blocks.
* **Query/Sort Mode (`?~`):** Filters and sorts blocks strictly by extracted pattern values (e.g., `?~ d date` sorts descending by the `~date:xyz` pattern).
* **Command Mode (`&`):** Routes text to external API fetchers.

### 3. External API Integrations

Albedo features a modular integration layer for fetching external research data directly into the database. These are executed asynchronously and append new blocks to the state upon resolution:

* **`&dic:`** Scrapes and parses the French Wiktionary REST API (`/api/rest_v1/page/html/`) using the browser's native `DOMParser` to extract definitions.
* **`&wiki:`** Fetches plain-text Wikipedia summaries via the Wikipedia REST API.
* **`&source:`** Queries the Crossref metadata API to return academic literature citations formatted natively for Albedo.

## 🛡️ Constraints & Limitations

* **DOM Scalability:** Renders are currently destructive (`innerHTML = ''`). Large datasets (>2,000 blocks) may experience frame drops during search/filter operations due to DOM thrashing.
* **Memory Footprint:** The entire database is loaded into `sys.state` (RAM) upon initialization.
