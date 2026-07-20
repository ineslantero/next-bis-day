---
name: fabric-build
description: 'Provision and validate Fabric escape room items — Lakehouse, Eventhouse, Semantic Model, and notebooks. Use when building game infrastructure or checking item health.'
---

# Fabric Build

Provision each item using `skills-for-fabric` skills, then run validation checks before handing off to documentation.

---

## Architecture — Items to Create

> The **Skill to use** column names skills from the sibling **[microsoft/skills-for-fabric](https://github.com/microsoft/skills-for-fabric)** repo, which must be open in the same VS Code workspace. Each skill owns the exact REST/CLI calls for its item type — invoke it instead of hand-writing API calls. If a named skill isn't available, that repo isn't loaded: stop and ask the user to add it to the workspace.

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

## Build Discipline (applies to every step)

Every item obeys the **game contract** — the tables, codes, and bindings the blueprint promised must all agree. Three rules keep deployments repeatable:

- **List before create.** Before creating any item, list existing items by `displayName` + type and reuse the existing ID if present. Repeated create calls otherwise pile up duplicate artifacts. Give diagnostic probes dedicated names and delete them when done — the workspace ends on the canonical item set only.
- **One source of truth per item.** Each notebook and model definition lives in exactly one file the build reads from. Keep no stale embedded copies inside the build script.
- **Validate JSON before upload.** Parse every definition payload (notebook `.ipynb`, TMDL parts, `definition.pbism`) before the create/update call.

### Identity Preflight (do this first)

Before discovering the workspace or creating any item, confirm which account is signed in and get the user's approval:

```pwsh
az account show --query "{user:user.name, tenant:tenantId, subscription:name}" -o json
```

Report the `user` and `tenant` back to the user and wait for them to confirm it is the correct account. If it is wrong, or no account is signed in, stop and ask them to run `az login` with the right identity. Deploying to the wrong tenant is hard to reverse, so treat this as a required gate.

**Completion criterion:** User has confirmed the signed-in account before any workspace lookup runs.

---

## Step 1: Provision the Lakehouse

Use `spark-authoring-cli` to create or find `{GameName}LH`.

**The Lakehouse name must not contain spaces** — spaces break the default Lakehouse binding and Delta table paths.

**Completion criterion:** `{GameName}LH` exists in the workspace and its ID is captured for subsequent steps.

---

## Step 2: Create and Run the Seed Notebook

Use `spark-authoring-cli` to create `{GameName} Seed Data` as a PySpark notebook with default Lakehouse metadata (`default_lakehouse`, `default_lakehouse_workspace_id`, `default_lakehouse_name`).

### Seed Notebook Requirements

- **Idempotent:** Use `CREATE OR REPLACE TABLE` in Spark SQL, or overwrite mode with `option("overwriteSchema", "true")`.
- **Built-in only:** Use PySpark, Spark SQL, and standard Python. Do not install external packages. Do not call external network endpoints.
- **Deterministic codes:** Keep all player-facing codes in variables at the top of the notebook so the Answer Key, Module 5 DAX, and setup guide stay synchronized.
- **Lowercase table names** for physical tables; keep semantic model display names friendly. Spark `saveAsTable` writes lowercase folders — the semantic model must bind to that exact case (see Step 4).
- **Valid notebook JSON:** The source file must be a real `.ipynb` document — top-level `nbformat`, `nbformat_minor`, `metadata`, and `cells`, each cell carrying `metadata.language` and a valid `cell_type`/`source`. Editor-export formats (e.g. `<VSCode.Cell …>`) fail deployment. Parse the JSON before calling create/update.
- **Validation cell:** Final cell prints row counts for every generated table.

### Tables to Populate

- Module 1 fact/anomaly table(s) — see `game-blueprint` for schema
- Module 3 clue fragment table(s) — see `game-blueprint` for schema
- Module 5 auth tables: `AuthModule1`, `AuthModule2`, `AuthModule3`, `AuthModule4` — each with 10 rows (1 correct + 9 decoys)

### Resolve IDs, Attach the Lakehouse, then Execute

1. **Resolve the binding IDs dynamically — never hardcode.** Confirm `{GameName}LH` exists in the target workspace, then list items filtered by type `Lakehouse` and match by display name to capture its item GUID, its workspace GUID, and its display name. The binding metadata below requires these real GUIDs, so discover them before attaching or uploading.
2. **Attach `{GameName}LH` to the notebook and confirm it is attached.** Using `spark-authoring-cli`, set the resolved IDs as the notebook's default Lakehouse (`default_lakehouse`, `default_lakehouse_workspace_id`, `default_lakehouse_name`) and verify the attachment resolves before running. Without an attached Lakehouse the notebook has no default catalog to write Delta tables into, and the run fails or writes nowhere.
3. **Upload the notebook definition.** When posting the body with `az rest`, write the JSON to a temp file and pass `--body "@$tmpFile"` — PowerShell mangles JSON inside `--body '{...}'`. Always include `--headers "Content-Type=application/json"` on create calls.
4. **Execute** with the Fabric Jobs API using `jobType=RunNotebook`.
5. **Poll** the job until it reaches `Completed`. If it fails, report the failure with Fabric job details and retry.

**Completion criterion:** `{GameName}LH` is attached to the notebook, the job status is `Completed`, and all tables exist with expected row counts.

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

### Table Bindings (DirectLake partitions)

Each table's partition is what binds the model to a physical Lakehouse Delta table. Every table `.tmdl` uses a DirectLake `entity` partition whose `source` points at the physical table (`entityName`) through the model's named expression (`expressionSource`):

```tmdl
table AuthModule1
	column LabCode
		dataType: string
		sourceColumn: LabCode

	partition AuthModule1 = entity
		mode: directLake
		source
			entityName: authmodule1
			expressionSource: DL_{GameName}
```

- `entityName` = the **physical** Lakehouse table name (lowercase, as Spark `saveAsTable` wrote it), even when the table's display name is `AuthModule1`.
- `expressionSource` = the named expression defined in `model.tmdl` (see the named-expression gotcha below).
- `sourceColumn` = the physical Delta column name; the display column name above it can differ.

The **Deployment Gotchas** below refine each of these — read them before generating the TMDL.

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

### Deployment Gotchas (hard-won — each one broke a real run)

**Definition & creation**

- **Create via the semantic model endpoint with a full definition.** `POST /v1/workspaces/{workspaceId}/semanticModels` carrying the complete envelope — `definition.pbism`, `definition/database.tmdl`, `definition/model.tmdl`, and `definition/tables/*.tmdl` as InlineBase64 parts, with `Content-Type: application/json`. A create without the full body returns `MissingDefinition`.
- **Creation and update are LROs.** The POST returns `202 Accepted` with an `x-ms-operation-id` header. Poll `GET /v1/operations/{id}` until `Succeeded` (inspect the `error` field on `Failed`) before discovering or updating the item. A `202` is not a finished create.

**`definition.pbism`**

- **Literal JSON, simple shape.** Build it as single-quoted literal text — PowerShell interpolation in a double-quoted here-string corrupts the `$schema`/`version` keys. Use the Items-API-compatible form and nothing fancier:
  ```json
  { "version": "4.2", "settings": { "qnaEnabled": true } }
  ```
  Keep the Direct Lake connection in the `model.tmdl` named expression — not in a `byConnection` or `datasetReference` pbism variant.

**`model.tmdl` named expression**

- **Generate from a template, then token-replace** the workspace/lakehouse IDs — hand-escaped interpolation produces a malformed M expression (`syntax error in expression DL_…`). Expected shape:
  ```tmdl
  expression DL_{GameName} =
  	let
  		Source = AzureStorage.DataLake("https://onelake.dfs.fabric.microsoft.com/{WORKSPACE_ID}/{LAKEHOUSE_ID}", [HierarchicalNavigation=true])
  	in
  		Source
  ```
- **Tab indentation required.** TMDL nests with literal tab characters, not spaces. Spaces cause `Indentation` parse errors.
- **`defaultPowerBIDataSourceVersion: powerBI_V3`** must appear in `model.tmdl` when any table uses an import-mode partition (e.g. a `_Measures` table with `evaluate {1}`). Without it, creation fails with "Import from JSON supported for V3 models only."

**Direct Lake table bindings**

- **`entityName` must match the physical OneLake table name exactly, including case.** Spark `saveAsTable` writes lowercase folders (`authmodule1`), so bind partitions to the lowercase name even when the display name is `AuthModule1`. Mixed-case bindings fail with "source tables do not exist or access denied." Display names stay friendly for UX; only the partition `entityName` must be case-correct.
- **Set `entityName` + `expressionSource`, and omit `schemaName`** unless a schema-enabled Lakehouse layout is explicitly verified — an unverified `schemaName: dbo` produces table warning icons in the UI.
- **Validate every binding against the actual Lakehouse table names before publish.**

**Token audience for runtime calls**

- Items-API CRUD (create/update/get semantic model) uses the **Fabric** audience: `https://api.fabric.microsoft.com`.
- Dataset runtime calls — refresh, `executeQueries`, datasource/parameters/users — use the **Power BI** audience: `https://analysis.windows.net/powerbi/api`. A Fabric token on these hangs or fails. Set the resource explicitly per call and keep the two API families separate.

**Completion criterion:** Semantic model exists, every partition `entityName` resolves to a real case-correct Lakehouse table, all named measures are present, and the creation LRO reached `Succeeded`.

---

## Step 5: Create the Diagnostic Notebook (Module 4)

Use `spark-authoring-cli` to create `{ModuleName} Diagnostic` — see `game-blueprint` Module 4 content design for cell structure and formatting rules.

**Reference example:** Model the new notebook on the validated `MurderAtMicrosoft Diagnostic` notebook captured in [`references/diagnostic-notebook-example.md`](references/diagnostic-notebook-example.md). Follow its **format and markdown conventions** — the same cell structure, heading levels, status glyphs (✅ / ⚠️ / 🔧), blockquotes, ASCII banner, and code-cell style. It shows the proven pattern: an ASCII banner intro cell, a "how to read this report" glyph legend, ~7 themed sections (mostly ✅ NOMINAL) with 2 ⚠️ WARNING red herrings, one 🔧 AUDIT section whose dense frozen output hides the real code, and a "Diagnostic Complete" outro cell with an in-character closing hint. Reuse this exact shape, tone, and formatting; swap in only the current game's theme, subsystem names, and embedded authorization code.

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

### 2. Schema Match

Confirm every table and column in the semantic model matches the source Lakehouse Delta tables. Mismatched schemas cause blank visuals or errors in reports. Confirm each partition `entityName` matches the physical (lowercase) Lakehouse table name exactly — case mismatches show as "source tables do not exist" errors.

### 3. Explicit Measures Exist

Confirm all average/aggregate measures needed by Module 1 visuals are created as named DAX measures — not left as implicit aggregations.

### 4. Canonical Inventory (no duplicates)

Confirm the workspace holds exactly one of each canonical item and no leftover probe artifacts from iterative create attempts. Delete any duplicate notebooks or semantic models created during testing.

**Completion criterion for validation:** All four checks pass. Only then proceed to documentation generation.
