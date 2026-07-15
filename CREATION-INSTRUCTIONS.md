# 🎮 Escape Room Game Creator — Getting Started

This guide walks you through creating a complete Fabric escape room game using GitHub Copilot. The game includes 5 puzzle modules built on Fabric items (Lakehouse, Eventhouse, Semantic Model, Notebook, Data Agent, and OrgApp).

**What's in this folder:**

| File | Purpose |
|------|---------|
| [CREATION-INSTRUCTIONS.md](CREATION-INSTRUCTIONS.md) | This file — setup steps and the game creation prompt |
| [.github/agents/](..%2F.github/agents/escape-room-builder.agent.md) | Agent + skills that tell Copilot how to build escape room games |
| [EXAMPLE-THEMES.md](EXAMPLE-THEMES.md) | Theme ideas with story hooks for inspiration |

---
## Team Roles

Assign these 5 roles to your team before starting:

| Player | Role | Phase 1 | Phase 2 | What They Build |
|--------|------|--------|--------|------------------|
| **1** | GitHub Copilot Player | ✅ Runs prompt | ✅ Power BI modules | Lakehouse, Eventhouse, Semantic Model, Seed + Diagnostic notebooks, Module 1 & 5 reports |
| **2** | Power BI Player | — | ✅ Power BI modules | Module 1 & 5 reports (or splits work with Player 1) |
| **3** | RTI Dashboard Creator | — | ✅ Builds dashboard | Module 2 RTI Dashboard |
| **4** | Data Agent Owner | — | ✅ Configures AI | Module 3 Data Agent |
| **5** | OrgApp Creator | — | ✅ Final assembly | Wires all items into OrgApp (Phase 2) |

**Key dependencies:**
- Player 1 must complete Phase 1 **before** any player starts Phase 2
- Player 4 (Data Agent Owner) must publish and share their Data Agent link **before** Player 5 can complete the OrgApp
- Players 1–4 should publish their items **before** Player 5 wires them into the OrgApp
- Player 2 is optional: Player 1 can handle all Power BI reports alone, or delegate to Player 2

---
## Workshop Flow

In a team event, the 5-person team follows this sequence:

| Phase | Who | What they do | Time |
|-------|-----|-------------|------|
| **1. Initial Build** | Player 1 (GitHub Copilot Player) | Alone: Runs the creation prompt. Copilot creates Lakehouse, Eventhouse, Semantic Model, seed notebook, diagnostic notebook, and generates the setup guide | ~1 hour |
| **2. Manual Build & Assemble** | All 5 players | In parallel: Players 1–4 each pick a setup guide file and build their assigned item (Module 1 & 5 reports, Module 2 dashboard, Module 3 Data Agent). Player 5 wires all published items into the OrgApp and publishes it | ~45 min |
| **3. Test & Reset** | Any player | Verify: Test the game end-to-end, solve all puzzles, reset filters | ~15 min |
| **4. Play & Escape** | Other team | Players: Teams swap OrgApp links and try to escape each other's games | ~30 min |

> **Tip:** Player 5 (OrgApp Creator) is blocked until Player 4 (Data Agent Owner) publishes their Data Agent and shares the link. Recommend: have Player 5 set up the OrgApp structure while waiting for the Data Agent link during phase 2.

---

## Prerequisites

Before starting, assign the **5 team roles** from the [Team Roles](#team-roles) section above:

**All team members:**
- A **Microsoft Fabric workspace** with capacity assigned (F64 or higher recommended)
- **Contributor** or **Member** role on the workspace

**Player 1 (GitHub Copilot Player) — Additional requirements:**
- **GitHub Copilot** license
- **Azure CLI** installed ([install guide](https://learn.microsoft.com/cli/azure/install-azure-cli))
- **Power BI Pro** license (or start a free trial)
- **Power BI Desktop** installed ([download](https://powerbi.microsoft.com/desktop))

**Player 2 (Power BI Player) — If splitting Power BI work:**
- **Power BI Pro** license (or start a free trial)
- **Power BI Desktop** installed ([download](https://powerbi.microsoft.com/desktop))

**Players 3–5 (RTI Dashboard Creator, Data Agent Owner, OrgApp Creator):**
- Browser access to Fabric workspace only (no special software needed)

---

## Step 1: Set Up Your Environment

> ⚠️ **Important:** This repo and [microsoft/skills-for-fabric](https://github.com/microsoft/skills-for-fabric) do **not** auto-combine. Copilot only sees a repo's agent and skill files when that repo is open in VS Code. To build the game you need **both** repos open in the same VS Code window.

### 1. Clone both repos as siblings

Pick a parent folder (e.g. `C:\Repos\` or `~/repos`). You will end up with this layout:

```
repos/
├── next-bis-day/            ← this repo (the Lakehouse-first game blueprint)
└── skills-for-fabric/       ← Microsoft Fabric authoring skills
```

In **VS Code**:

Repository URLs to paste:

- `https://github.com/ineslantero/next-bis-day`
- `https://github.com/microsoft/skills-for-fabric`

1. Open the **Source Control** view (the branch icon in the left activity bar, or **View → Source Control** in the menu bar).
2. Click **Clone Repository**.
3. Paste this repository URL into the URL field at the top and press Enter.
4. In the folder picker, choose your parent folder (e.g. `C:\Repos`) and click **Select as Repository Destination**.
5. When the popup asks "Would you like to open the cloned repository?", click **Cancel** — you'll open both together in the next step.
6. Repeat steps 1–4 for the Fabric skills:
   - Click **Source Control → Clone Repository** again
   - Paste `https://github.com/microsoft/skills-for-fabric`
   - Select the **same** parent folder you used for the first clone
7. Again, click **Cancel** when asked to open the repository.

> **Already cloned `skills-for-fabric`?** Skip step 6. Instead, open your existing copy in VS Code, click the **Synchronize Changes** button at the bottom-left status bar (the circular arrows next to the branch name) to pull the latest skills, then continue.

### 2. Open both repos in a multi-root workspace

So Copilot picks up the game blueprint **and** the Fabric skills at the same time:

1. In VS Code, click **File → Open Folder…** in the top menu bar, then select your `next-bis-day` folder and click **Select Folder**.
2. Click **File → Add Folder to Workspace…**, then select your `skills-for-fabric` folder and click **Add**.
3. Click **File → Save Workspace As…**, name it (e.g. `escape-room.code-workspace`) and save it anywhere you like. Open this file next time you want to come back.

Both agent files (the escape room builder here, and the Fabric authoring skills there) are now in Copilot's context whenever you chat in this workspace.

### 3. Open Copilot Chat in Agent mode

- Open the Copilot Chat panel (click the **Copilot** icon top-right, or **View → Chat**).
- Set the mode dropdown to **Agent** for the initial build prompt. For follow-up questions, switch to **Auto** — it routes to Agent only when tools are needed, saving tokens on simple clarifications.

---

## Step 2: Create Your Game

Copy the prompt below. Replace the `[BRACKETED]` sections with your choices (or leave them blank and let Copilot come up with them), then paste it into GitHub Copilot **in Agent mode**.

**Note:** Copilot will automatically invoke the escape room builder agent and all necessary skills (game-blueprint, fabric-build, documentation) in the correct order. You don't need to manually trigger skills.

See [EXAMPLE-THEMES.md](EXAMPLE-THEMES.md) for theme ideas and inspiration.

---

```
Create a Fabric escape room game in my workspace called [YOUR WORKSPACE NAME].

THEME: [YOUR THEME — e.g., "haunted mansion", "biolab outbreak", "pirate ship"]

GAME NAME: [SHORT NAME — e.g., "Ravencrest Manor", "Lab Zero", "The Crimson Tide"]

STORY: [2-3 SENTENCES — What happened? Why are players trapped? What's the urgency?
Example: "An asteroid hit our space station. Emergency power is failing. We need to find 4 codes to launch the escape shuttle before power runs out."]

AI CHARACTER NAME: [NAME OF THE AI/GHOST/GUIDE — e.g., "ODIN", "Lady Ravencrest", "NEXUS", "Polly the Parrot"]

AI CHARACTER PERSONALITY: [1-2 SENTENCES — How does the AI talk?
Example: "Damaged space station AI. Clinical but glitchy. Inserts [STATIC] into responses. Cares about crew survival."]

MODULE IDEAS: Generate 4 themed module ideas that fit the story. Each must follow the escape room framework:
1. A data anomaly puzzle for a Power BI report
2. A pattern gap puzzle for an RTI Dashboard
3. An AI conversation puzzle for a Data Agent
4. A notebook discovery puzzle with hidden diagnostic output

Build the complete game following the escape room framework. Use these Fabric skills for each item:

- **Eventhouse + KQL Database** (use the `eventhouse-authoring-cli` skill) — time-series data for the pattern gap puzzle (~150–200 activity rows across 50+ time windows with one gap, and ~50 assessment windows with decoy codes)
- **Lakehouse + seed notebook** (use the `spark-authoring-cli` skill) — create `{GameName}LH`, deploy an attached `{GameName} Seed Data` notebook, execute it via `RunNotebook`, and populate all Module 1, Module 3, and Module 5 Delta tables INCLUDING `AuthModule1`, `AuthModule2`, `AuthModule3`, and `AuthModule4` tables (4 rows each with 1 correct code + 3 decoys) in the Lakehouse
- **Module 4 diagnostic notebook** (use the `spark-authoring-cli` skill) — themed diagnostic output with **12–15 cells** containing 6–8 diagnostic sections mixed with 2–3 warning sections, one section showing the hidden code
- **Semantic Model** (use the `semantic-model-authoring` skill) — DirectLake TMDL over Lakehouse Delta tables with explicit average measures and a victory/denied DAX measure

Do **NOT** create the Data Agent, OrgApp, Power BI reports, or RTI Dashboard via Copilot — those are configured manually in the Fabric portal using the Setup Guide you generate.

After creating the items above, generate documentation **split into multiple files** so different team members can work in parallel:
1. A **`setup-guide/`** folder with one markdown file per workstream — index `README.md` (role assignments + dependency order), `00-SHARED-SETUP.md` (sign-in + theme JSON), `01-MODULE1-REPORT.md`, `02-MODULE2-DASHBOARD.md`, `03-MODULE3-DATA-AGENT.md`, `04-MODULE5-REPORT.md`, `05-ORGAPP.md`
2. A top-level **`PLAY-GUIDE.md`** for players (no spoilers, just hints and code formats)
3. A top-level **`ANSWER-KEY.md`** with all 4 codes and how to find them
```

---

## Step 3: Verify the Build

Before handing off to the team, confirm Copilot created everything:

**Fabric workspace items:**
- [ ] `{GameName}LH` — Lakehouse with tables visible (including `AuthModule1–4`)
- [ ] `{GameName} Seed Data` — Notebook (check job status shows **Completed** — see instructions below)
- [ ] `{GameName}EH` — Eventhouse with KQL database and tables populated
- [ ] `{GameName}SM` — Semantic Model with DAX measures
- [ ] `{ModuleName} Diagnostic` — Notebook with markdown/output cells (12–15 cells)

**How to check seed notebook job status:** In Fabric workspace → open the `{GameName} Seed Data` notebook → click the **Job status** button at the top toolbar (next to the Run button) → wait for the job to show **Completed**. If it fails, expand the failure details and report the error.

**Generated files in the repo:**
- [ ] `setup-guide/` folder with 7 files (`README.md`, `00-SHARED-SETUP.md`, `01-MODULE1-REPORT.md`, `02-MODULE2-DASHBOARD.md`, `03-MODULE3-DATA-AGENT.md`, `04-MODULE5-REPORT.md`, `05-ORGAPP.md`)
- [ ] `PLAY-GUIDE.md` at repo root
- [ ] `ANSWER-KEY.md` at repo root with all 4 codes

If anything is missing, ask Copilot to retry that step before continuing.

---

## Step 4: Follow the Setup Guide

Open `setup-guide/README.md` first — it lists owner roles, dependency order, and task assignments.

**5-person team split:**
- **Player 1 (GitHub Copilot Player)** — `01-MODULE1-REPORT.md`, `04-MODULE5-REPORT.md` (Power BI reports)
- **Player 2 (Power BI Player)** — `01-MODULE1-REPORT.md`, `04-MODULE5-REPORT.md` (optional: splits Power BI work with Player 1, or Player 1 handles alone)
- **Player 3 (RTI Dashboard Creator)** — `02-MODULE2-DASHBOARD.md`
- **Player 4 (Data Agent Owner)** — `03-MODULE3-DATA-AGENT.md` (independent — can run in parallel) ⚠️ **MUST publish and share the Data Agent link before Player 5 proceeds**
- **Player 5 (OrgApp Creator)** — `05-ORGAPP.md` (waits for Players 1–4 to publish items + receive Data Agent link from Player 4)

Everyone reads `00-SHARED-SETUP.md` first (sign-in to Power BI Desktop and Fabric, save the theme JSON).

---

## Step 5: Test & Reset

Before sharing with other teams, do a full end-to-end test:

1. **Open the OrgApp** as a player would — click through each module
2. **Solve each puzzle** using the `ANSWER-KEY.md` — verify each code works:
   - Module 1: use the slicer → drillthrough → find the code
   - Module 2: find the gap in the bar chart → match it in the table
   - Module 3: chat with the Data Agent → confirm it gives hints without revealing codes
   - Module 4: read through the notebook → find the hidden code
3. **Enter all 4 codes in Module 5** — confirm the victory message appears
4. **Reset for players:**
   - In the Module 1 report: clear all slicer selections
   - In the Module 5 report: reset all 4 dropdown slicers to blank
   - Close any Data Agent chat sessions
   - Filters are per-session, so other players will start fresh — but if you published with persistent filters, clear them and re-publish
5. **Share the OrgApp link** with the other team

---

## Troubleshooting

| Issue | Solution |
|-------|---------|
| "I can't create items" | Check workspace role is Contributor or Member |
| Seed notebook won't run | Confirm workspace capacity is active, Spark is available, and tenant policy allows notebook execution |
| Seed notebook has package errors | The generated notebook should use built-in PySpark/Spark SQL only; remove external package installs unless the customer has approved them |
| Copilot REST calls fail | Check sign-in, Azure CLI availability, and HTTPS/443 access to Fabric and Microsoft Entra endpoints |
| Data Agent option missing | Data Agent can depend on tenant settings, region, licensing, and preview feature enablement |
| OrgApp option missing or can't publish | OrgApp and sharing/publishing can be controlled by tenant policy even for workspace Contributors |
| "Report won't publish" | User needs Power BI Pro license |
| Data Agent not responding | Verify data sources are connected and agent is published |
| Gauge shows no data | Ensure the semantic model has an explicit average measure (not implicit aggregation) |
| Drillthrough page shows all rows | Check that the drillthrough field is set in Visualizations → Drill through |
| Custom theme won't apply | Must use Power BI Desktop → View → Themes → Browse for themes |
| Report fields are blank/error | Semantic model schema may not match Lakehouse Delta tables — refresh/update the model |
| Copilot creates a Warehouse | Cancel and remind it: "Do NOT create a Warehouse. Use Lakehouse Delta tables only." |
| Copilot skips generating setup guide files | Re-prompt: "Now generate the setup-guide/ folder with all markdown files as specified in the documentation skill" |
| Copilot generates a single giant setup guide | Re-prompt: "Split the setup guide into separate files per the documentation skill — one file per workstream" |
| Custom theme won't apply to Power BI report | Theme file was saved as `.json.txt` instead of `.json`. Open File Explorer → find the file → rename it to remove the `.txt` suffix → try again in Power BI Desktop |
| Data Agent step in setup guide blocked | Player 3 (Data Agent Owner) must publish the agent and share the link with Player 4 before the OrgApp can be assembled. Check that the agent is published and the shareable link was copied correctly |


