---
name: game-blueprint
description: 'Blueprint escape room puzzles — module patterns, data schemas, authorization codes, and volume requirements. Use when designing a new game or reviewing puzzle structure.'
---

# Game Blueprint

Design reference for escape room puzzles. Consult when designing a new game — every module's puzzle type, data shape, and decoy strategy.

---

## Module Patterns

Every game has exactly **5 modules**.

### Module 1: Data Anomaly (Power BI Report)

- **Puzzle type:** Find the outlier in a dataset
- **Fabric items:** Lakehouse table + Semantic Model + Report
- **Pattern:** A table with 6–10 entities where one has an abnormal reading. Page 1 shows a summary table with the entity name and the average of the key measure (e.g., `CargoSection` + `Avg Weight`), a slicer to filter entities, and a gauge showing the average value. A drillthrough page shows the raw detail rows and the diagnostic code.
- **Data design:**
  - Main table: `Fact{EntityName}` — 20–30 rows across 6–10 entities
  - One entity has readings that are clearly anomalous (e.g., low O2, high temperature, abnormal weight)
  - **Every entity** has a `DiagnosticCode` value — the anomalous entity has the real authorization code, and all other entities have decoy codes in the same format (e.g., if the real code is `HULL-7293`, decoys might be `HULL-4021`, `HULL-8856`, etc.)
  - This makes the puzzle harder: players cannot simply look for "which section has a code" — they must identify the anomaly first, then drillthrough to find the corresponding code

### Module 2: Pattern Gap (RTI Dashboard)

- **Puzzle type:** Find the missing data point in a time series
- **Fabric items:** Eventhouse KQL tables + RTI Dashboard
- **Pattern:** A bar chart showing activity across 50+ time windows where one window has zero activity. A companion table shows assessments per window with various ratings and decoy codes. Only the zero-activity window has the correct code.
- **Data design:**
  - Activity table: `{ActivityName}` — 150–200 rows of events with timestamps across 50+ time windows
  - One 10-minute window has ZERO events (the gap)
  - Assessment table: `{WindowName}` — 50+ rows, one per time window
    - Ratings: BLOCKED (60%), CAUTION (20% — with decoy codes), MARGINAL (15% — with decoy codes), CLEAR (1 row — the answer)
    - Only the CLEAR row has the correct code. CAUTION and MARGINAL rows have decoy codes in the same format.
  - The gap must be in the middle of the data (not at the start or end) so it's not obvious
  - KQL `summarize count() by bin()` does not produce empty bins — the gap shows as a missing bar, not a zero-height bar. The RTI Dashboard KQL queries must account for this.

### Module 3: AI Conversation (Data Agent)

- **Puzzle type:** Extract information through dialog with an AI character
- **Fabric items:** Lakehouse + Eventhouse tables as data sources + Data Agent
- **Pattern:** A themed AI character who has fragmented information about a signal/message/clue. Player must ask the right questions. The AI gives hints but never directly reveals the code. The code is discoverable by querying the right table through conversation.
- **Data design:**
  - Clue table: `{ClueTableName}` — 5–10 rows of fragments/messages
  - One fragment contains the code (embedded in a longer text or as an ID field)
  - The AI should know about all game data sources so it can query them when asked
- **AI personality rules:**
  - Has a themed personality (damaged AI, ghost, parrot, etc.)
  - NEVER directly reveals any code unless player has worked through the logic
  - Gives progressively stronger hints if player is stuck (after 5+ messages)
  - Adds atmospheric flavor (glitches, spooky sounds, pirate slang, etc.)

### Module 4: Read & Discover (Notebook)

- **Puzzle type:** Find the code hidden in formatted output
- **Fabric items:** Notebook
- **Pattern:** A pre-formatted diagnostic/report notebook with **12–15 cells** spanning 6–8 themed diagnostic sections mixed with 2–3 warning sections. The code is buried in one section among a large amount of visual noise. The notebook should feel like a real system log — dense, detailed, and not immediately scannable.
- **Content design:**
  - **Minimum 12–15 cells** — mix of markdown headers, code cells with pre-rendered output, and narrative text cells
  - Use generous themed emojis throughout (✅, ⚠️, 🔧, 📡, 🛡️, 💾, 🔋, 🌡️, etc.) so the FAIL section doesn't stand out just because it has an emoji
  - **6–8 diagnostic/analysis sections**, most showing PASS/NORMAL/OK with realistic-looking output (tables of readings, status lists, metric summaries)
  - Include **2–3 WARNING sections** (not just one FAIL among all-green) — warnings with yellow ⚠️ that look suspicious but are red herrings with no code
  - The real code section should **NOT use a dramatically different format** — embed the code inside a longer block of output text (e.g., in a status table row, inside a log entry, or as a field value in a multi-line diagnostic dump) rather than on a standalone highlighted line
  - Add **intro and outro cells** — a themed title cell at the top with ASCII art or a banner, and a summary cell at the bottom saying "Diagnostic complete — review flagged items above"
  - Include at least one cell with a **long scrollable table** (8+ rows of system metrics) so players have to read carefully
  - Use themed formatting (ASCII art headers, status indicators, horizontal rules, timestamps, etc.)
  - The notebook is read-only — players don't need to run it

### Module 5: Final Escape (Power BI Report)

- **Puzzle type:** Combine all 4 codes to win
- **Fabric items:** Lakehouse Delta tables (4 independent auth tables) + Semantic Model + Report
- **Pattern:** 4 slicer visuals (one per module), each connected to its own independent table. A DAX measure checks if all 4 correct codes are selected and displays a victory/denied status.
- **Data design:**
  - 4 independent tables: `AuthModule1`, `AuthModule2`, `AuthModule3`, `AuthModule4`
  - Each table has 10 rows: 1 correct code + 9 decoys in the same format
  - Each table has a uniquely named column (e.g., `LabCode`, `CameraCode`, `AICode`, `ServerCode`)
  - The unique column names prevent slicer cross-filtering conflicts
  - DAX measure uses SELECTEDVALUE from each table's column to check if all 4 are correct

---

## Data Generation Rules

1. **Authorization codes:** Format `{PREFIX}-{4 digits}`. Each module has a unique prefix matching its theme.
2. **Decoy codes:** Same format as real codes. 9 decoys per module, randomized.
3. **Lakehouse seed data:** Use PySpark DataFrames or Spark SQL `CREATE TABLE ... USING DELTA` / `saveAsTable`. Use deterministic values so the Answer Key and DAX measures match the generated tables.
4. **Eventhouse data:** Use far-future or themed dates. Ensure timestamps are in the correct column order when using `.ingest inline`.
5. **Volume — enough to make puzzles non-trivial:**
   - Module 1: 20–30 fact rows across 6–10 entities
   - Module 2: 150–200 activity rows, 50+ assessment windows
   - Module 3: 5–10 clue fragments
   - Module 4: 12–15 notebook cells
   - Module 5: 4 × 10 = 40 auth rows
