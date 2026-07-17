# 🚨 05 — The Game Portal (Org App)

**Owner role:** Player 5 (OrgApp Creator)
**Depends on:** [01](01-MODULE1-REPORT.md), [02](02-MODULE2-DASHBOARD.md), [04](04-MODULE5-REPORT.md) all **published**, and the **Data Agent shareable link** from [03](03-MODULE3-DATA-AGENT.md) in hand.
**Screens needed:** Browser only (Fabric portal)

> *Every piece of evidence is in place. Now assemble the case file — one portal where investigators walk the crime scene module by module, from the badge logs to the final accusation. This is the front door players will open.*

This guide wires all five modules into a single **Org app** called **Murder at Microsoft**.

**You will link these items** (confirm each exists before starting):

| Module | Item to link | Type |
|--------|--------------|------|
| 🔬 Module 1 | `Module 1 - Badge Anomaly` | Report |
| 📹 Module 2 | `Murder at Microsoft Dashboard` | Real-Time Dashboard |
| 🔮 Module 3 | **Frontier AI** share link | Link (new tab) |
| 📓 Module 4 | `MurderAtMicrosoft Diagnostic` | Notebook |
| 🕵️ Module 5 | `Module 5 - Final Accusation` | Report |

---

## Steps

### 1. 🆕 Create the Org app
1. Open **https://app.fabric.microsoft.com** → **next-bis-day-3** workspace.
2. **+ New item** → find **Org app** → click it.
3. Name it **`Murder at Microsoft`** → **Create**.

> **Recovery:** If **Org app** isn't listed, it isn't enabled for your tenant. Ask a Fabric admin to enable the *Org apps* tenant setting.

### 2. ⚙️ Customize tab
1. Open the **Customize** tab.
2. **App theme:** set to the report background **`#15151C`** (dark) to match the murder-mystery theme.
3. Optionally **upload a logo** (a 🔪 or 🕵️ image if you have one).

### 3. 👥 App elements
1. **Add Overview** → write the **Name** `Murder at Microsoft`. Then **Add header** → **Add title and description**, and paste the story hook into the description —
   > *The CEO of Microsoft has been murdered at Build — the biggest conference of the year — moments before unveiling a frontier AI feature that would have changed everything. Four authorization codes are scattered across the conference's data systems. Work the evidence, question Frontier AI, and unmask the killer before the closing keynote.*
   >
   > Add a one-line briefing per module (see structure below).
2. *(Optional)* **+ Add section** → name it after the module (e.g. **`Module 1`**).
3. Now add the five modules **in order**, each via **+ Add content**:
   - **🔬 Module 1** → **Add workspace content** → pick the report **`Module 1 - Badge Anomaly`**. Rename the nav label to `🔬 Module 1 · Badge Anomaly`.
   - **📹 Module 2** → **Add workspace content** → pick the dashboard **`Murder at Microsoft Dashboard`**. Label it `📹 Module 2 · CCTV Gap`.
   - **🔮 Module 3** → **Add link** → **Name:** `🔮 Module 3 · Frontier AI` → **URL:** paste the **Data Agent shareable link** from [03](03-MODULE3-DATA-AGENT.md) → set **Open in new tab** ✔ (Data Agents don't embed inline).
   - **📓 Module 4** → **Add workspace content** → pick the notebook **`MurderAtMicrosoft Diagnostic`**. Label it `📓 Module 4 · Core Diagnostic`.
   - **🕵️ Module 5** → **Add workspace content** → pick the report **`Module 5 - Final Accusation`**. Label it `🕵️ Module 5 · Final Accusation`.
4. Confirm the five items appear in module order in the left navigation preview.

### 4. 🚀 Publish save
1. Click **Save**.
2. Click **View app**.

### 5. 🧪 Test as a player
1. Walk all five modules:
   - Module 1 report loads and you can drill into the Green Room.
   - Module 2 dashboard shows the bar chart gap and assessments table.
   - Module 3 opens **Frontier AI in a new tab** and it responds.
   - Module 4 notebook opens and is readable.
   - Module 5 report accepts the four codes and shows the verdict card.
2. Fix any broken link or permission before sharing broadly.

---

## 🗺️ Target OrgApp Structure

```
Overview (landing)
  → Title: 🚨 MURDER AT MICROSOFT
  → Body: story hook + one-line briefing per module

Section: "🏢 The Build Crime Scene"
  → 🔬 Module 1 · Badge Anomaly        → Report:    Module 1 - Badge Anomaly
  → 📹 Module 2 · CCTV Gap             → Dashboard: Murder at Microsoft Dashboard
  → 🔮 Module 3 · Frontier AI          → Link (new tab): Data Agent share link
  → 📓 Module 4 · Core Diagnostic      → Notebook:  MurderAtMicrosoft Diagnostic
  → 🕵️ Module 5 · Final Accusation     → Report:    Module 5 - Final Accusation
```

Theme color: **`#15151C`** (matches the report theme background).

---

## ✅ Completion criterion
Org app **`Murder at Microsoft`** is **published** with all **five modules** wired (Module 3 as a new-tab link), and a test player has opened the app link and successfully reached every module.
