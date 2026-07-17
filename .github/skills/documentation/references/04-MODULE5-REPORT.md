# 🕵️ 04 — Module 5: The Final Accusation Report

**Owner role:** Player 1 (GitHub Copilot Player) or Player 2 (Power BI Player)
**Depends on:** [00-SHARED-SETUP.md](00-SHARED-SETUP.md) (signed in, theme file saved)
**Screens needed:** Power BI Desktop (then publish to the Fabric workspace)

> *You have worked every angle: the badge logs, the camera blind spot, the intercepted messages, the core diagnostic. Four codes. Bring them together here, on the accusation board, and Frontier AI's own verdict engine will confirm whether you have truly unmasked the killer — or whether the evidence still doesn't add up.*

**Puzzle outcome:** The player selects one value in each of **four dropdown slicers** (one per module). When all four match the correct codes, a status **Card** flips to **"🎉 CASE SOLVED — The killer is unmasked. Build is secure and you have escaped."** Otherwise it reads **"❌ ACCESS DENIED — The evidence does not add up. Keep investigating."**

**Data behind it:** semantic model `MurderAtMicrosoftSM`, four independent auth tables and their uniquely named columns, plus the DAX measure `Case Status`:

| Slicer | Table | Column | Correct value |
|--------|-------|--------|---------------|
| Module 1 | `authmodule1` | `BadgeCode` | `BADGE-4471` |
| Module 2 | `authmodule2` | `CameraCode` | `CCTV-2058` |
| Module 3 | `authmodule3` | `ModelCode` | `MODEL-7712` |
| Module 4 | `authmodule4` | `CoreCode` | `CORE-9330` |

---

## Steps

### 1. 🔌 Connect to the semantic model
1. Open **Power BI Desktop**.
2. **Home** ribbon → **Get data** dropdown → **More…** → **Power Platform** → **Power BI semantic models** → **Connect**.
3. Select **`MurderAtMicrosoftSM`** → **Connect** (stay in **live connection**; do not click *"Make changes to this model"*).
4. In the **Data** pane you should see `authmodule1`–`authmodule4` and the measure **`Case Status`**.

### 2. 🎨 Apply the theme
1. **View** ribbon → **Themes** dropdown → **Browse for themes…**
2. Select **`murder-at-microsoft.json`** → **Open**.

### 3. 🖊️ Add the page title
1. **Insert** → **Text box** → type `🕵️ THE FINAL ACCUSATION`.
2. Font size **30**, light color, placed across the top of the page.

### 4. 🎛️ Add the four dropdown slicers (one per module)
Do this **four times**, once per table. Keep them side by side across the page.

**Slicer 1 — Module 1 (Badge):**
1. Click empty canvas → **Visualizations** → **Slicer** icon.
2. From **Data**, drag **`authmodule1` → `BadgeCode`** into the slicer **Field** well.
3. Select the slicer → **Format your visual** → **Slicer settings** → **Options** → **Style: Dropdown**.
4. **Insert → Text box** above it → type `Module 1 · Badge Code`.

**Slicer 2 — Module 2 (Camera):**
1. Empty canvas → **Slicer** → drag **`authmodule2` → `CameraCode`** into **Field**.
2. **Format → Slicer settings → Options → Style: Dropdown**.
3. Text box label above: `Module 2 · Camera Code`.

**Slicer 3 — Module 3 (Model):**
1. Empty canvas → **Slicer** → drag **`authmodule3` → `ModelCode`** into **Field**.
2. **Format → Slicer settings → Options → Style: Dropdown**.
3. Text box label above: `Module 3 · Model Code`.

**Slicer 4 — Module 4 (Core):**
1. Empty canvas → **Slicer** → drag **`authmodule4` → `CoreCode`** into **Field**.
2. **Format → Slicer settings → Options → Style: Dropdown**.
3. Text box label above: `Module 4 · Core Code`.

> 🧩 Each slicer uses a **different column name** (`BadgeCode`, `CameraCode`, `ModelCode`, `CoreCode`) on a **separate table** — this is intentional so the four dropdowns never cross-filter each other.

### 5. 🃏 Add the verdict Card
1. Click empty canvas → **Visualizations** → **Card** icon.
2. From **Data**, drag the measure **`Case Status`** into the card's **Fields** well.
3. Make the card **large** and center it **below** the four slicers — this is the dramatic reveal.
4. Apply conditional color so the verdict reads green when solved, red when denied:
   - Select the card → **Format your visual** → **Callout value** → expand **Color** → click the **fx** (conditional formatting) button.
   - **Format style: Rules**.
   - **What field should we base this on?** → choose the measure **`Case Status`**.
   - Rule 1: **If value contains** `CASE SOLVED` → color **green** (`#2ECC71`).
   - Rule 2 (add rule): **If value contains** `ACCESS DENIED` → color **red** (`#C81E3A`).
   - Click **OK**.
   - *(If your version doesn't offer "contains," use **Field value** conditional formatting instead and drive it from `Case Status`, or leave the default light foreground — the emoji ✅/❌ in the text already signals the result.)*

### 6. 🧪 Test the win condition
1. Set the dropdowns to **`BADGE-4471`**, **`CCTV-2058`**, **`MODEL-7712`**, **`CORE-9330`**.
2. The card should read **"🎉 CASE SOLVED — The killer is unmasked. Build is secure and you have escaped."** in green.
3. Change any one dropdown to a decoy — the card should flip to the red **"❌ ACCESS DENIED…"** message.
4. Reset the dropdowns to blank so players start fresh.

### 7. 💾 Save and publish
1. **File → Save as** → name it **`Module 5 - Final Accusation`**.
2. **Home → Publish** (or **File → Publish → Publish to Power BI…**) → select **next-bis-day-3** → **Select**.
3. Wait for **"Success!"**.

**Recovery:** If the card always shows "ACCESS DENIED" even with correct codes, verify each slicer is bound to the **correct table/column** in the table above and that exactly **one** value is selected per slicer (the measure uses `SELECTEDVALUE`, which returns blank if multiple are selected).

---

## ✅ Completion criterion
Report **`Module 5 - Final Accusation`** is published with **four dropdown slicers** (`BadgeCode`, `CameraCode`, `ModelCode`, `CoreCode`) and a **`Case Status`** verdict card with conditional green/red formatting. Entering all four correct codes triggers the **"🎉 CASE SOLVED"** message.
