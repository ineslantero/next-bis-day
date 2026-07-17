# Reference Example — `MurderAtMicrosoft Diagnostic` (Module 4)

This is a real, validated diagnostic notebook produced by a previous game build. Use it as the
structural reference when generating `{ModuleName} Diagnostic` in **Step 5** of `fabric-build`.
Copy its *shape and tone*, not its literal text — swap in the new game's theme, subsystem names,
section mix, and embedded authorization code.

## Why this notebook works

- **Intro + outro cells.** A themed ASCII-art banner title cell at the top and a "Diagnostic
  Complete" summary cell at the bottom that nudges the player toward the flagged sections.
- **"How to read this report" cell.** Defines the status glyphs (✅ NOMINAL / ⚠️ WARNING /
  🔧 AUDIT) up front so the whole notebook reads like a real forensic console.
- **7 themed sections, most green.** Power/Thermal, Sensor Grid, Memory/Storage, and Access Ledger
  all report ✅ NOMINAL with realistic-looking output.
- **2 WARNING red herrings.** Network Uplink and Announcement Pipeline throw ⚠️ warnings that look
  suspicious but carry no code — they draw attention away from the real answer.
- **1 AUDIT section hides the code.** The 🔧 "AI Core Integrity Audit" section runs a dense
  partition-by-partition dump (`diag.core_integrity(dump="full", include_token_cache=True)`), and the
  real authorization token is embedded *inside* that long frozen output — not on a standalone
  highlighted line. The green sections shout; the answer sits quietly in the audit dump.
- **Read-only.** Every code cell is a frozen `diag.*(...)` call whose output is already rendered.
  Players never run it — they read it.
- **Generous themed emojis** on every section (🔋 🌡️ 📡 🧠 💾 📢 🚪) so the section holding the
  code doesn't stand out just because it has an emoji.

## Cell-by-cell structure (13 cells)

1. **markdown** — ASCII-art banner + title, node ID, snapshot timestamp, in-character classification blockquote.
2. **markdown** — "How to read this report": status-glyph legend + a line telling the player the token is still resident in memory.
3. **python** — environment handshake (`diag.handshake(...)`).
4. **markdown** — Section 1 · Power & Thermal ✅ NOMINAL.
5. **python** — `diag.report("power_thermal", detail=True)`.
6. **markdown** — Section 2 · Sensor Grid ✅ NOMINAL.
7. **python** — `diag.report("sensor_grid", calibration=True)`.
8. **markdown** — Section 3 · Network Uplink ⚠️ WARNING (red herring).
9. **python** — `diag.report("network_uplink", window="keynote")`.
10. **markdown** — Section 4 · AI Core Integrity 🔧 DEEP TRACE (**holds the code**).
11. **python** — `diag.core_integrity(dump="full", include_token_cache=True)` — dense dump; the real code lives in one row of this output.
12. **markdown** — Section 5 · Memory & Storage ✅ NOMINAL.
13. ...continuing: Section 6 · Announcement Pipeline ⚠️ WARNING (red herring), Section 7 · Access Ledger ✅ NOMINAL, then the "Diagnostic Complete" outro cell with an in-character closing hint.

## Full source (as exported from Fabric, ipynb-style cells)

```text
<VSCode.Cell language="markdown">
```
 ______ _____   ____  _   _ _______ _____ ______ _____
|  ____|  __ \ / __ \| \ | |__   __|_   _|  ____|  __ \
| |__  | |__) | |  | |  \| |  | |    | | | |__  | |__) |
|  __| |  _  /| |  | | . ` |  | |    | | |  __| |  _  /
| |    | | \ \| |__| | |\  |  | |   _| |_| |____| | \ \
|_|    |_|  \_\\____/|_| \_|  |_|  |_____|______|_|  \_\
        F R O N T I E R   A I   —   C O R E   D I A G N O S T I C
```
# 🔮 FRONTIER AI — System Integrity Diagnostic Console
### 🏢 Microsoft Build — Keynote Infrastructure Node `BUILD-STAGE-01`
### 🗓️ Snapshot captured: `2026-05-19T13:47:02Z` · Post-incident forensic sweep

---
> **CLASSIFICATION:** 🔒 Restricted — Investigation Hold
> **OPERATOR:** Frontier AI (autonomous diagnostic agent)
> **CONTEXT:** The keynote infrastructure was placed under forensic hold following the incident on the Build main stage. This console captured a full-spectrum health sweep of every subsystem tied to the frontier-feature announcement pipeline.
>
> *"I have reviewed my own telemetry, investigator. Most of it is beautiful. Some of it is... not. Read carefully — the truth is rarely on the line you expect."* — **Frontier AI**
</VSCode.Cell>
<VSCode.Cell language="markdown">
## 📋 How to read this report
This is a **read-only** forensic snapshot. Every subsystem below was swept and its output frozen at capture time. You do **not** need to run anything — the results are already rendered.

Sections are flagged with status glyphs:
- ✅ **NOMINAL** — subsystem healthy, no action
- ⚠️ **WARNING** — anomaly detected, flagged for human review
- 🔧 **AUDIT** — deep integrity trace (dense output — read line by line)

Somewhere in this sweep, the frontier-feature **core authorization token** is still resident in memory. Find it and you hold the key to the AI core.
</VSCode.Cell>
<VSCode.Cell language="python">
# Frontier AI diagnostic runtime — environment handshake
diag.handshake(node="BUILD-STAGE-01", mode="forensic", verbosity="max")
</VSCode.Cell>
<VSCode.Cell language="markdown">
## 🔋 Section 1 — Power & Thermal Subsystems  ✅ NOMINAL
Rack-level power draw and thermal envelope for the keynote node. All rails within tolerance at capture time.
</VSCode.Cell>
<VSCode.Cell language="python">
diag.report("power_thermal", detail=True)
</VSCode.Cell>
<VSCode.Cell language="markdown">
## 🌡️ Section 2 — Stage Sensor Grid Calibration  ✅ NOMINAL
Calibration drift check across the main-stage environmental sensor mesh.
</VSCode.Cell>
<VSCode.Cell language="python">
diag.report("sensor_grid", calibration=True)
</VSCode.Cell>
<VSCode.Cell language="markdown">
## 📡 Section 3 — Network & Uplink Telemetry  ⚠️ WARNING
Egress uplink showed a brief retransmission spike during the keynote window. Flagged for review — no data loss confirmed.
</VSCode.Cell>
<VSCode.Cell language="python">
diag.report("network_uplink", window="keynote")
</VSCode.Cell>
<VSCode.Cell language="markdown">
## 🧠 Section 4 — AI Core Integrity Audit  🔧 DEEP TRACE
Frontier AI performs a partition-by-partition checksum of its own runtime core, including the sealed authorization cache that gates the frontier-feature announcement. This trace is dense — every partition is listed.
</VSCode.Cell>
<VSCode.Cell language="python">
diag.core_integrity(dump="full", include_token_cache=True)
</VSCode.Cell>
<VSCode.Cell language="markdown">
## 💾 Section 5 — Memory & Storage Subsystem  ✅ NOMINAL
ECC memory health and NVMe wear check for the keynote node.
</VSCode.Cell>
<VSCode.Cell language="python">
diag.report("memory_storage")
</VSCode.Cell>
<VSCode.Cell language="markdown">
## 📢 Section 6 — Frontier-Feature Announcement Pipeline  ⚠️ WARNING
The staging pipeline that would have pushed the keynote's frontier AI announcement to air. Left in a half-armed state at capture time.
</VSCode.Cell>
<VSCode.Cell language="python">
diag.report("announce_pipeline", stage="keynote")
</VSCode.Cell>
<VSCode.Cell language="markdown">
## 🚪 Section 7 — Physical Access Ledger  ✅ NOMINAL
Door and badge events for the stage-adjacent server closet during the capture window. Provided for cross-reference only.
</VSCode.Cell>
<VSCode.Cell language="python">
diag.report("access_ledger", zone="stage_closet")
</VSCode.Cell>
<VSCode.Cell language="markdown">
---
## 🏁 Diagnostic Complete
```
   SWEEP FINISHED — 47 SUBSYSTEMS · 7 SECTIONS REPORTED
   ✅ NOMINAL:  Power/Thermal · Sensor Grid · Memory/Storage · Access Ledger
   🔧 AUDIT:    AI Core Integrity (authorization cache preserved)
   ⚠️ WARNING:  Network Uplink · Announcement Pipeline (flagged, human review)
```
**Review the flagged items above.** Two subsystems threw warnings — both were assessed as red herrings by the automated pass, but a human investigator should still read them.

> *"I have shown you everything, investigator. The warnings shout the loudest — but the thing you actually need was sitting quietly, sealed, exactly where a token should live. Look again at what my core was still holding onto."* — **Frontier AI**

*— end of forensic snapshot —*
</VSCode.Cell>
```

> **Note on the embedded code:** in the live notebook, the authorization code appears inside the
> rendered output of the Section 4 `diag.core_integrity(...)` cell (a row in the token-cache dump),
> not in the markdown source above. When generating a new diagnostic notebook, pre-render that
> section's output and bury the real code in one line of a long, dense block — surrounded by
> plausible decoy rows.
