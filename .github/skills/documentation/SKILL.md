---
name: documentation
description: 'Scaffold split setup guides, play guide, and answer key for escape room games. Use after Fabric items pass validation to generate parallel-workstream documentation.'
---

# Documentation

Generate documentation after all Fabric items pass validation. The setup guide is split into parallel-workstream files so team members work independently.

---

## File Structure

Generate exactly these files:

```
setup-guide/
├── README.md                    ← index + role assignments + dependency order
├── 00-SHARED-SETUP.md           ← everyone reads first (sign-in, save theme JSON)
├── 01-MODULE1-REPORT.md         ← Power BI Desktop — Data Anomaly report
├── 02-MODULE2-DASHBOARD.md      ← Fabric portal — RTI Dashboard
├── 03-MODULE3-DATA-AGENT.md     ← Fabric portal — Data Agent
├── 04-MODULE5-REPORT.md         ← Power BI Desktop — Final Escape report
└── 05-ORGAPP.md                 ← Fabric portal — game portal (final assembly)
PLAY-GUIDE.md                    ← top-level, for players (no spoilers)
ANSWER-KEY.md                    ← top-level, game admin only (codes + verification)
```

---

## Setup Guide File Specs

Each file in `setup-guide/` must include at the top:
- **Owner role** — e.g. "Power BI builder", "Data Agent owner", "OrgApp owner"
- **Depends on** — which other files must be done first
- **Estimated screens needed** — Power BI Desktop only, browser only, or both

Keep cross-references minimal. Repeat short shared steps inline; link to `00-SHARED-SETUP.md` for longer ones.

---

## setup-guide/README.md — Index

Contents:
- A table mapping each setup file to its owner role and dependencies
- A "suggested team split" section (e.g. 3-person split: BI builder takes 01/02/04, agent owner takes 03, portal owner takes 05)
- A dependency diagram showing `05-ORGAPP.md` waits for reports/dashboard to be published and Data Agent share link to be available
- A "definition of done" checklist (all 4 codes verified, OrgApp link tested as a player)

**Completion criterion:** README contains the role table, team split, dependency diagram, and done checklist.

---

## setup-guide/00-SHARED-SETUP.md — Shared Prerequisites

Contents:
- **Sign-in steps:**
  - Open **Power BI Desktop** → top-right **Sign in** with the workspace account
  - In a browser open `https://app.fabric.microsoft.com` → switch to the workspace
- **Verify game items exist** — list every Copilot-created item (`{GameName}LH`, `{GameName} Seed Data`, `{GameName}EH`, `{GameName}SM`, `{ModuleName} Diagnostic`). If any is missing, return to Copilot chat to retry.
- **Good to know:**
  - Workspace must be on **active Fabric capacity** — paused capacity = items won't work
  - **Contributor** or **Member** role required
  - **Power BI Desktop** and **Power BI Pro** license needed for report steps
  - **Data Agent** and **OrgApp** availability depends on tenant settings
- **Save the report theme JSON** — step-by-step with Notepad:
  1. Provide the full theme JSON block
  2. Open Notepad → paste → **File → Save As…**
  3. **Save as type** dropdown → **All Files (*.*)** (critical — prevents `.txt` suffix)
  4. Name the file `{theme-name}.json` → **Save**
  5. Verify via File Explorer that the file ends in `.json`, not `.json.txt`

**Completion criterion:** File contains sign-in, item verification list, prerequisites, and theme-save steps.

---

## setup-guide/01-MODULE1-REPORT.md — Data Anomaly Report

Owner: Power BI builder. Depends on: `00-SHARED-SETUP.md`. Screens: Power BI Desktop.

Steps:
1. **Connect to the semantic model:** Home ribbon → **Get data** dropdown → **Power BI semantic models** → pick `{GameName}SM` → **Connect** (live-connect mode, do not click "Make changes to this model")
2. **Apply the custom theme:** View ribbon → Themes gallery dropdown → **Browse for themes…** → select the `.json` file
3. **Add the page title** via Insert → Text box (themed, 28–36 pt, foreground color)
4. **Add the slicer:** Visualizations → Slicer → drag entity field into Field well → Format → Slicer settings → Options → Style: **Tile**
5. **Add the gauge** with the explicit average measure in the Value well
6. **Add the summary table** with entity field + average measure (summary per entity, not raw rows)
7. **Add the drillthrough page:** Duplicate page → rename → deselect visuals → Drill through section → drag entity field → add Table visual showing all raw columns including `DiagnosticCode`
8. **Add navigation hint** on Page 1 (text box: "Right-click any row and select Drill through → Section Detail")
9. **Save and publish:** File → Save as → File → Publish → Publish to Power BI… → pick workspace

**Completion criterion:** Report published to workspace with summary page, gauge, slicer, and drillthrough page.

---

## setup-guide/02-MODULE2-DASHBOARD.md — Pattern Gap RTI Dashboard

Owner: Power BI builder (or RTI owner). Depends on: nothing — runs in parallel. Screens: browser only.

Steps:
1. **Create the dashboard:** Fabric workspace → **+ New item** → **Real-Time Dashboard** → name it `{GameName} Dashboard` → **Create**
2. **Connect data source:** Manage tab → Data sources → + Add → **OneLake data hub** or **KQL Database** → select `{GameName}EH` → **Connect**
3. **Add bar chart tile:** Paste the activity-by-window KQL query → Run → Visual type: Bar chart → themed title → Apply changes → Save
4. **Add assessments table tile:** Paste the assessments KQL query → Visual type: Table → themed title → Apply changes → Save
5. **Save the dashboard**

### KQL Queries to Include

Generate two ready-to-paste KQL queries with actual table/column names:

1. **Activity bar chart query:**
   ```kql
   {ActivityTable}
   | summarize EventCount = count() by bin(Timestamp, 10m)
   | order by Timestamp asc
   ```

2. **Assessments table query:**
   ```kql
   {AssessmentTable}
   | project WindowStart, WindowEnd, Rating, Code
   | order by WindowStart asc
   ```

Replace `{ActivityTable}`, `{AssessmentTable}`, and column names with the actual names from the Eventhouse. Queries must be copy-pasteable.

**Completion criterion:** Dashboard published with bar chart tile and assessments table tile, both using correct KQL queries.

---

## setup-guide/03-MODULE3-DATA-AGENT.md — AI Conversation Data Agent

Owner: Data Agent owner. Depends on: nothing — fully independent. Screens: browser only.

Steps:
1. **Create the Data Agent:** Fabric workspace → **+ New item** → **Data Agent** → name it `{AIName}` → **Create**
2. **Add data sources:**
   - **+ Add data source** → Lakehouse `{GameName}LH` or its SQL analytics endpoint → select all game tables → **Add**
   - **+ Add data source** → KQL Database `{GameName}EH` → select all tables → **Add**
3. **Set the AI instructions** — paste the full instructions block into the **Instructions** field

### Data Agent Instructions Template

Generate a complete instructions block inside a fenced code block. Structure:

```
You are {AIName}, {backstory}.

## Personality
- Speaking style: {themed dialect, glitches, pirate slang, ghostly whispers, etc.}
- Emotional tone: {caring, menacing, mischievous, mysterious, etc.}
- Add atmospheric flavor: {[STATIC], *creaking sounds*, ghostly sighs, etc.}

## Game Context
You are part of a puzzle game with 5 modules. Players are trying to escape by finding 4 authorization codes.
- Module 1: {brief description} — uses a Power BI report
- Module 2: {brief description} — uses an RTI Dashboard
- Module 3 (YOU): {brief description} — players chat with you to find a code
- Module 4: {brief description} — uses a Notebook
- Module 5: Players enter all 4 codes to escape

## Your Data Sources
You have access to these tables:
- Lakehouse `{GameName}LH`: {list tables and what they contain}
- Eventhouse `{GameName}EH`: {list tables and what they contain}

## Code Protection Rules
- NEVER directly reveal any authorization code from any module
- If a player asks for a code directly, deflect in character
- You may help players understand the data structure and guide them toward the right table/column
- For YOUR module's code: give progressively stronger hints based on conversation length
  - Messages 1–4: vague thematic hints ("the fragments hold the key...")
  - Messages 5–9: point toward the right table or column name
  - Messages 10+: describe exactly where to look without saying the code value
- For OTHER modules' codes: say you don't have access or redirect them to the right module

## Hint Progression
{3-4 themed hints of increasing specificity for the Module 3 clue}
```

4. **Publish** at the top right
5. **Share** at the top right → **People in your organization with the link can use** → **Copy link**
6. **Hand off the link** to the OrgApp owner

**Completion criterion:** Data Agent published and shareable link copied for OrgApp setup.

---

## setup-guide/04-MODULE5-REPORT.md — Final Escape Report

Owner: Power BI builder. Depends on: `00-SHARED-SETUP.md`. Screens: Power BI Desktop.

Steps:
1. **Connect to semantic model:** Home → Get data → Power BI semantic models → `{GameName}SM` → Connect (live connect)
2. **Apply the theme** via View → Themes → Browse for themes
3. **Add the page title** (e.g. "🚀 LAUNCH AUTHORIZATION")
4. **Add four slicers — one per module:**
   - Drag each auth column (e.g. `LabCode`, `CameraCode`, `AICode`, `ServerCode`) into a slicer's Field well
   - Format → Slicer settings → Options → Style: **Dropdown**
   - Label each slicer with a text box above it ("Module 1 Code", etc.)
5. **Add the status card:**
   - Visualizations → **Card** → drag the `Launch Status` measure into Fields
   - Format → Callout value → Conditional formatting → Font color on → green for victory, red for denied
6. **Save and publish** → File → Publish → Publish to Power BI… → pick workspace

**Completion criterion:** Report published with 4 dropdown slicers, status card, and conditional formatting.

---

## setup-guide/05-ORGAPP.md — Game Portal OrgApp

Owner: OrgApp owner. Depends on: `01`, `02`, `03`, `04` all published. The Data Agent **shareable link** from `03` must be in hand. Screens: browser only.

Steps:
1. **Create the OrgApp:** Fabric workspace → **+ New item** → **Org app** → name it `{GameName}` → **Create**
2. **Setup tab:** Set Name, Description (paste the story hook), Theme color (report theme's background color), optionally upload a logo
3. **Audience tab:**
   - **+ Add audience** → name `Players`
   - **+ Add new section** → name after the location (e.g. "Station Modules")
   - For each module → **+ Add content**:
     - Module 1: pick the Module 1 Report
     - Module 2: pick the Module 2 RTI Dashboard
     - Module 3: **Add link** → name after AI character → paste Data Agent shareable link → **Open in new tab**
     - Module 4: pick the Notebook
     - Module 5: pick the Module 5 Report
   - Optionally add an **Overview** card at the top with the story text
4. **Permissions tab:** Add players' email addresses or groups
5. **Publish app** → review → **Publish**
6. Copy the app link and share with players

### OrgApp Structure

```
Overview (landing page)
  → Title: themed emergency/alert title
  → Body: story text + per-module briefing descriptions

Section: "{Location Name}"
  → Module 1: Report item
  → Module 2: RTI Dashboard item
  → Module 3: Link (new tab) → Data Agent shareable link
  → Module 4: Notebook item
  → Module 5: Report item
```

Theme color: set to match the report theme's background color.

**Completion criterion:** OrgApp published, all 5 modules linked, app link tested as a player.

---

## PLAY-GUIDE.md — Player Guide (No Spoilers)

For players — NO codes, NO direct answers. Contents:
- The story hook and setting
- What each module looks like and what to do
- Hint format (e.g., "look for the anomaly", "find the gap", "ask the AI")
- Code format per module (e.g., "LAB-XXXX")
- How to use the final escape module

**Completion criterion:** Guide contains story, module descriptions, hint format, and code format — zero spoilers.

---

## ANSWER-KEY.md — Game Admin Only

Contents:
- All 4 codes with discovery method
- Which entity/window/fragment contains each code
- Data verification queries for Lakehouse tables, Eventhouse tables, and notebook validation output

**Completion criterion:** All 4 codes documented with verification queries.

---

## Report Theme Template

Generate a themed JSON color palette with:
- `dataColors`: 8 accent colors matching the theme
- `background`: dark base color
- `foreground`: light readable text color
- `tableAccent`: primary accent color
- `good`/`neutral`/`bad`: green/amber/red equivalents
- `visualStyles`: page background, visual background, title font, label colors

The theme must work with Power BI Desktop (View → Themes → Browse for themes).

**Completion criterion:** Theme JSON is valid and included in `00-SHARED-SETUP.md`.

---

## Extra Credit Modules

Suggest these optional extensions if the team finishes early:

### Extra Credit 1: Additional Puzzle Module
- Add a 5th investigation module (e.g., find the saboteur/traitor/mole)
- New Lakehouse tables: access logs, schedules, alerts
- New report with investigation visuals
- Update the Data Agent to hint at this module
- Can be standalone (fun extra) or mandatory (5th code required for escape)

### Extra Credit 2: Real-Time Streaming Telemetry
- Create a telemetry generator notebook that sends declining readings (battery, temperature, etc.)
- Create an Eventstream with a Custom Endpoint source
- Route data to the Eventhouse
- Add RTI Dashboard tiles showing live gauges
- Creates genuine urgency — players see readings dropping while they play
