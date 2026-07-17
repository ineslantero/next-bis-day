# 🩸 00 — Shared Setup (Everyone Reads First)

**Owner role:** Everyone on the build team
**Depends on:** nothing
**Screens needed:** Browser for all roles; **Power BI Desktop** additionally for the Module 1 & 5 report builders

> *Before you can catch a killer, you need access to the crime scene. This page gets every investigator signed in, confirms the evidence is in place, and prepares the one shared file the Power BI builders need.*

---

## 1. 🔐 Sign In

### Power BI Desktop (Module 1 & 5 builders only)
1. Install **Power BI Desktop** from the Microsoft Store or https://powerbi.microsoft.com/desktop if you don't have it.
2. Open **Power BI Desktop**.
3. Top-right corner → click **Sign in**.
4. Enter the **workspace account** (the same account used to run the Copilot build) → **Continue** → complete authentication.
5. You should now see your name in the top-right corner.

### Fabric portal (all roles)
1. Open a browser and go to **https://app.fabric.microsoft.com**.
2. Sign in with the **same workspace account**.
3. Bottom-left → **workspace switcher** → select **next-bis-day-3**.
4. You should now see the workspace item list.

---

## 2. 🔎 Verify the Evidence Exists

In the **next-bis-day-3** workspace, confirm every Copilot-created item below is present. Use the **search box** at the top of the item list if the workspace is crowded.

| Item name | Type | Used by |
|-----------|------|---------|
| `MurderAtMicrosoftLH` | Lakehouse | Modules 1, 3, 5 |
| `MurderAtMicrosoft Seed Data` | Notebook | (data loader — already run) |
| `MurderAtMicrosoftEH` | Eventhouse (+ KQL Database) | Modules 2, 3 |
| `MurderAtMicrosoftSM` | Semantic model | Modules 1, 5 |
| `MurderAtMicrosoft Diagnostic` | Notebook | Module 4 |

> 🧩 You will also see a `MurderAtMicrosoftLH` **SQL analytics endpoint** and a `MurderAtMicrosoftEH` **KQL Database** with matching names — those are automatic companions, not duplicates. Leave them alone.

**Recovery:** If any item in the table above is **missing**, do not build around it — return to the GitHub Copilot chat that created the workspace and re-run the build. The docs assume all five exist.

---

## 3. ℹ️ Good to Know (prerequisites)

- ⚡ **Active Fabric capacity** — the workspace must be on a running capacity. If items show errors like *"capacity paused"*, resume the capacity (or ask an admin) before continuing.
- 👤 **Contributor** or **Member** role on **next-bis-day-3** is required to build reports, dashboards, agents, and the org app.
- 🖥️ **Power BI Desktop** + a **Power BI Pro** (or Premium Per User) license is required for the Module 1 & 5 report steps and to publish.
- 🤖 **Data Agent** and **Org app** availability depend on **tenant settings**. If **+ New item** doesn't list *Data Agent* or *Org app*, ask your Fabric admin to enable them.

---

## 4. 🎨 Save the Report Theme File

> **Only the Power BI builder (Modules 1 & 5) needs this.** Dashboard, Data Agent, and OrgApp owners can skip to their guide.

A ready-made dark murder-mystery theme lives next to this file: **`murder-at-microsoft.json`**. If you cloned the repo you already have it. If you need to recreate it from scratch on the build machine, follow these steps with **Notepad**:

1. Open **Notepad** (Start menu → type *Notepad* → Enter).
2. Open the theme file `setup-guide/murder-at-microsoft.json` from this repo and **copy its entire contents**, then paste into Notepad. *(The full JSON is reproduced at the bottom of this page for convenience.)*
3. In Notepad: **File → Save As…**
4. In the **Save as type** dropdown, choose **All Files (\*.\*)** — this is **critical**; it prevents Windows from adding a hidden `.txt` suffix.
5. In **File name**, type exactly `murder-at-microsoft.json` → click **Save**.
6. Open **File Explorer**, navigate to where you saved it, and confirm the file name ends in **`.json`** and **not** `.json.txt`. If the `.txt` snuck on, rename it.

You will load this file in **Power BI Desktop → View → Themes → Browse for themes…** in Modules 1 and 5.

---

### 📋 Theme JSON (for reference)

```json
{
  "name": "Murder at Microsoft",
  "dataColors": ["#C81E3A", "#8B0000", "#DC143C", "#6C6C7A", "#3A3A48", "#A31621", "#5A5A6E", "#E63946"],
  "background": "#15151C",
  "foreground": "#EDEDF2",
  "tableAccent": "#C81E3A",
  "good": "#2ECC71",
  "neutral": "#E1B12C",
  "bad": "#C81E3A",
  "maximum": "#C81E3A",
  "center": "#E1B12C",
  "minimum": "#2ECC71",
  "textClasses": {
    "title": { "color": "#EDEDF2", "fontFace": "Segoe UI Semibold", "fontSize": 20 },
    "header": { "color": "#C81E3A", "fontFace": "Segoe UI Semibold", "fontSize": 14 },
    "label": { "color": "#EDEDF2", "fontFace": "Segoe UI", "fontSize": 11 }
  },
  "visualStyles": {
    "*": {
      "*": {
        "background": [{ "show": true, "color": { "solid": { "color": "#1E1E28" } }, "transparency": 0 }],
        "border": [{ "show": true, "color": { "solid": { "color": "#3A3A48" } }, "radius": 6 }],
        "title": [{ "show": true, "fontColor": { "solid": { "color": "#EDEDF2" } }, "background": { "solid": { "color": "#1E1E28" } }, "fontSize": 13, "fontFamily": "Segoe UI Semibold" }],
        "visualHeader": [{ "show": false }]
      }
    },
    "page": {
      "*": {
        "background": [{ "color": { "solid": { "color": "#15151C" } }, "transparency": 0 }],
        "outspace": [{ "color": { "solid": { "color": "#15151C" } } }]
      }
    }
  }
}
```

✅ **You're set up.** Head to your assigned guide in [README.md](README.md).
