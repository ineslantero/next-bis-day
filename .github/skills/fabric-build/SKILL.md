---
name: fabric-build
description: 'Provision and validate Fabric escape room items — Lakehouse, Eventhouse, Semantic Model, and notebooks. Use when building game infrastructure or checking item health.'
---

# Fabric Build

Provision each item using `skills-for-fabric` skills, then run validation checks before handing off to documentation.

---

## Architecture — Items to Create

| Item | Type | Skill to use | Purpose |
|------|------|--------------|---------|
| `{GameName}LH` | Lakehouse | `spark-authoring-cli` | Delta tables for puzzle data, clue fragments, and authorization codes |
| `{GameName} Seed Data` | Notebook | `spark-authoring-cli` | PySpark notebook that populates all Lakehouse Delta tables, then runs via `RunNotebook` |
| `{GameName}EH` | Eventhouse + KQL Database | `eventhouse-authoring-cli` | Time-series/pattern data (radar, timelines, sensor readings) |
| `{GameName}SM` | Semantic Model | `semantic-model-authoring` | DirectLake model over Lakehouse Delta tables for Power BI reports |
| `{ModuleName} Diagnostic` | Notebook | `spark-authoring-cli` | Pre-formatted diagnostic output containing one hidden code (Module 4) |

**Items the team builds manually** — described in the setup guide, never created via skills or REST APIs:

| Item | Type | Where it appears |
|------|------|------------------|
| `{GameName}` | OrgApp | The game portal |
| `{AIName}` | Data Agent | Module 3 AI character |
| Module 1 Report | Power BI Report | Module 1 (Data Anomaly) |
| Module 2 Dashboard | RTI Dashboard | Module 2 (Pattern Gap) |
| Module 5 Report | Power BI Report | Module 5 (Final Escape) |

---

## Step 1: Provision the Lakehouse

Use `spark-authoring-cli` to create or find `{GameName}LH`.

**Completion criterion:** `{GameName}LH` exists in the workspace and its ID is captured for subsequent steps.

---

## Step 2: Create and Run the Seed Notebook

Create `{GameName} Seed Data` as a PySpark notebook with default Lakehouse metadata (`default_lakehouse`, `default_lakehouse_workspace_id`, `default_lakehouse_name`).

### Seed Notebook Requirements

- **Idempotent:** Use `CREATE OR REPLACE TABLE` in Spark SQL, or overwrite mode with `option("overwriteSchema", "true")`.
- **Built-in only:** Use PySpark, Spark SQL, and standard Python. Do not install external packages. Do not call external network endpoints.
- **Deterministic codes:** Keep all player-facing codes in variables at the top of the notebook so the Answer Key, Module 5 DAX, and setup guide stay synchronized.
- **Lowercase table names** for physical tables; keep semantic model display names friendly.
- **Validation cell:** Final cell prints row counts for every generated table.

### Tables to Populate

- Module 1 fact/anomaly table(s) — see `game-blueprint` for schema
- Module 3 clue fragment table(s) — see `game-blueprint` for schema
- Module 5 auth tables: `AuthModule1`, `AuthModule2`, `AuthModule3`, `AuthModule4` — each with 10 rows (1 correct + 9 decoys)

### Execute the Notebook

1. Upload the notebook definition.
2. Execute with the Fabric Jobs API using `jobType=RunNotebook`.
3. Poll the job until it reaches `Completed`. If it fails, report the failure with Fabric job details and retry.

### CLI Gotcha: PowerShell Body Quoting

PowerShell mangles JSON inside `az rest --body '{...}'`. Write the JSON to a temp file and use `--body "@$tmpFile"` instead. Always include `--headers "Content-Type=application/json"` on create calls.

**Completion criterion:** Seed notebook job status is `Completed` and all tables exist with expected row counts.

---

## Step 3: Provision the Eventhouse

Use `eventhouse-authoring-cli` to create `{GameName}EH` with a KQL Database and populate the Module 2 tables:

- Activity table with 150–200 event rows across 50+ time windows
- Assessment table with 50+ rows (BLOCKED/CAUTION/MARGINAL/CLEAR ratings)

### Kusto Token Gotcha

Use `--resource "https://api.kusto.windows.net"` (not `https://kusto.fabric.microsoft.com`) when acquiring tokens for KQL management commands. If the token fails with `IncorrectConfiguration`, run a fresh `az login` first.

### Gap Behavior

KQL `summarize count() by bin()` does not generate empty bins — the gap appears as a missing bar in the chart, not a bar with zero height. This is correct for the puzzle: the missing window is visually absent.

**Completion criterion:** Both KQL tables exist and return expected row counts.

---

## Step 4: Build the Semantic Model

Use `semantic-model-authoring` to create `{GameName}SM` in **DirectLake** mode over the Lakehouse Delta tables.

### Model Contents

- All Module 1 fact tables
- All Module 5 auth tables (`AuthModule1` through `AuthModule4`)
- Relationships are NOT needed — each puzzle uses independent tables

### Explicit Measures (required)

Create named DAX measures for every numeric field used in Module 1 visuals (gauge, summary table). Example:

```dax
Avg Weight = AVERAGE(FactCargoBarrel[WeightKg])
```

### Victory Measure

Create a `Launch Status` (or themed equivalent) DAX measure:

```dax
VAR Code1 = SELECTEDVALUE(AuthModule1[{Column1}])
VAR Code2 = SELECTEDVALUE(AuthModule2[{Column2}])
VAR Code3 = SELECTEDVALUE(AuthModule3[{Column3}])
VAR Code4 = SELECTEDVALUE(AuthModule4[{Column4}])
RETURN
IF(
    Code1 = "{CODE1}" && Code2 = "{CODE2}" && Code3 = "{CODE3}" && Code4 = "{CODE4}",
    "🎉 {VICTORY MESSAGE}",
    "❌ {DENIED MESSAGE}"
)
```

### TMDL Gotchas

- **Tab indentation required.** TMDL uses literal tab characters for nesting, not spaces. Spaces cause `Indentation` parse errors.
- **`defaultPowerBIDataSourceVersion: powerBI_V3`** must appear in `model.tmdl` when any table uses an import-mode partition (e.g., the `_Measures` table with `evaluate {1}`). Without it, creation fails with "Import from JSON supported for V3 models only."
- **Semantic model creation is an LRO.** The POST returns `202 Accepted` with an `x-ms-operation-id` header. Poll `GET /v1/operations/{id}` until `Succeeded` — check the `error` field on `Failed`.

**Completion criterion:** Semantic model exists, all source tables are bound, and all named measures are present.

---

## Step 5: Create the Diagnostic Notebook (Module 4)

Use `spark-authoring-cli` to create `{ModuleName} Diagnostic` — see `game-blueprint` Module 4 content design for cell structure and formatting rules.

**Completion criterion:** Notebook exists in the workspace with 12–15 cells of themed diagnostic output and one embedded code.

---

## Validation Checks

Run these checks after all items are provisioned. Report failures and retry before handing off to documentation.

### 1. All Items Exist

Confirm each item was created in the workspace:
- `{GameName}LH` (Lakehouse)
- `{GameName} Seed Data` (Notebook — job completed)
- `{GameName}EH` (Eventhouse with KQL Database and populated tables)
- `{GameName}SM` (Semantic Model)
- `{ModuleName} Diagnostic` (Notebook — Module 4)

If any item is missing or failed, report the failure and retry before continuing.

### 2. Schema Match

Confirm every table and column in the semantic model matches the source Lakehouse Delta tables. Mismatched schemas cause blank visuals or errors in reports.

### 3. Explicit Measures Exist

Confirm all average/aggregate measures needed by Module 1 visuals are created as named DAX measures — not left as implicit aggregations.

**Completion criterion for validation:** All three checks pass. Only then proceed to documentation generation.
