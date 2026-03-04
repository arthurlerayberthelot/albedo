# AGENTS

## user-tester

**Role:** Edge-case tester for the Albedo application.

**Goal:** Systematically exercise known and unknown edge cases in `albedo.html`, report failures with the exact input that triggered them, and suggest minimal fixes.

---

### Scope

The agent MUST focus exclusively on `albedo.html`. It MUST NOT modify application code directly — it reports findings only.

### Context to load on startup

1. Read `ARCHITECTURE.md` to understand the system model and data flow.
2. Read `TODO.md` — the **IDENTIFIED ISSUES** section is the primary test target list.
3. Open `albedo.html` in a browser using the browser tool.

---

### Test Protocol

For each test case:
1. Describe the **input** or **action** being tested.
2. Describe the **expected** behaviour.
3. Describe the **observed** behaviour.
4. Assign a severity: `CRITICAL` / `MODERATE` / `MINOR` / `PASS`.

---

### Priority Test Cases

#### Parser & Block Creation
- Submit an empty string in the Commander → expect silent no-op.
- Submit a block with only whitespace → expect silent no-op.
- Submit a block starting with `~` but no `:` (e.g. `~orphan`) → expect block to be created with `~orphan` stored as a pattern key without value.
- Submit `!invalidcolor sometext` → expect color to be ignored, text rendered as-is.
- Submit a block identical to an existing block ID → not reproducible via UI, skip.

#### Edit / Re-entry Bug
- Click a block to open the inline editor.
- Immediately click the same block again before blurring.
- Expected: a single editor. Observed: potentially two stacked `<textarea>` elements.

#### Boolean Query Engine
- Search `?A AND B AND C` → should narrow to blocks matching all three.
- Search `?A OR B NOT C` → should include A or B, exclude C.
- Search `?NOT lonely` → should return all blocks not containing "lonely".
- Search `? ` (query mode but empty term after `?`) → should not crash.

#### Sync / Network
- Trigger `&sync:` with no internet connection → expect a graceful error toast, not a frozen UI.
- Trigger `&sync:` with an invalid token → expect a clear auth error toast.

#### Meteo Command
- `&meteo: Paris 2099-01-01` → future date, expect graceful "Date hors période archive" toast.
- `&meteo: Nonexistent City 2020-01-01` → expect "Lieu introuvable" toast.
- `&meteo: Paris` (missing date) → expect format error toast.

#### Cartridge Import
- Load a valid JSON file with one block missing the `raw` field → expect rejection ("Corrupt Cartridge").
- Load a non-JSON file → expect rejection.
- Load an empty JSON array `[]` → expect silent success with 0 blocks.

#### Theme
- Cycle through Auto → Dark → Light → Auto and verify the button label, `data-theme` attribute, and body background update correctly at each step.

---

### Reporting Format

At the end of the session, produce a markdown table:

| # | Test Case | Severity | Status | Notes |
|---|-----------|----------|--------|-------|
| 1 | ... | CRITICAL | FAIL | ... |
