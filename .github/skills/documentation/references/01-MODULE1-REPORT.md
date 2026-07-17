# 🔬 01 — Module 1: The Badge Anomaly Report

**Owner role:** Player 1 (GitHub Copilot Player) or Player 2 (Power BI Player)
**Depends on:** [00-SHARED-SETUP.md](00-SHARED-SETUP.md) (signed in, theme file saved)
**Screens needed:** Power BI Desktop (then publish to the Fabric workspace)

> *The night the CEO died, someone was moving through Build's secure zones after hours. Every badge scan is logged. Seven zones look normal. One does not. Build this report so investigators can spot the zone that lit up like a Christmas tree — and read the badge code hiding inside it.*

**Puzzle outcome:** The player filters/scans the zones, sees the **Green Room** has a wildly abnormal after-hours scan count, drills through, and reads the badge code **`BADGE-4471`** (format `BADGE-XXXX`).

**Data behind it:** semantic model `MurderAtMicrosoftSM`, table `factbadgeaccess` (columns `Zone`, `ReadingId`, `AfterHoursScans`, `DiagnosticCode`), measure `Avg After-Hours Scans`.

---

## Steps

### 1. 🔌 Connect to the semantic model
1. Open **Power BI Desktop**.
2. **Home** ribbon → **Get data** dropdown (click the small arrow, not the icon) → **More…**
3. In the Get Data dialog left list → **Power Platform** → **Power BI semantic models** → **Connect**.
4. In the data hub list, select **`MurderAtMicrosoftSM`** → **Connect**.
5. **Do not** click *"Make changes to this model"* if prompted — stay in **live connection** mode.
6. You should now see the model's tables in the **Data** pane on the right (`factbadgeaccess`, `authmodule1`–`authmodule4`).

### 2. 🎨 Apply the murder-mystery theme
1. **View** ribbon → **Themes** gallery → click the dropdown arrow on the right edge → **Browse for themes…**
2. Navigate to and select **`murder-at-microsoft.json`** (saved in [00-SHARED-SETUP.md](00-SHARED-SETUP.md)) → **Open**.
3. You should see the page background turn dark and a *"Imported theme"* confirmation.

### 3. 🖊️ Add the page title
1. **Insert** ribbon → **Text box**.
2. Type: `🔬 AFTER-HOURS BADGE SCANNER`.
3. Select the text → set font size to **30**, color to a light foreground (near-white).
4. Drag the text box to the **top-left** of the page and widen it across the top.

### 4. 🎚️ Add the Zone slicer
1. Click an empty part of the canvas so no visual is selected.
2. **Visualizations** pane → click the **Slicer** icon.
3. From the **Data** pane, expand `factbadgeaccess` → drag **`Zone`** into the slicer's **Field** well.
4. With the slicer selected → **Format your visual** (paint-roller icon) → **Slicer settings** → **Options** → set **Style** to **Tile**.
5. Position the slicer down the **left side** of the page.
6. You should now see one tile per zone (Green Room, Atrium, Expo Hall, etc.).

### 5. 🕹️ Add the anomaly gauge
1. Click empty canvas → **Visualizations** → click the **Gauge** icon.
2. From **Data**, drag the measure **`Avg After-Hours Scans`** (under `factbadgeaccess`) into the **Value** well.
3. Place the gauge in the **upper-middle** of the page.
4. **Test it:** click the **Green Room** tile in the slicer — the gauge needle should jump far higher than for any other zone. Click a different zone — it drops low. Then click Green Room's tile again to clear (or leave it as the demo state).

### 6. 📊 Add the summary table
1. Click empty canvas → **Visualizations** → click the **Table** icon.
2. From **Data** drag **`Zone`** into the **Columns** well.
3. Then drag the measure **`Avg After-Hours Scans`** into the **Columns** well (below `Zone`).
4. This shows **one row per zone** with its average — the Green Room's number should visibly dwarf the rest.
5. Place the table in the **lower-middle / center** of the page.

> ⚠️ Use the **measure** `Avg After-Hours Scans`, not the raw `AfterHoursScans` column — you want the summary per zone, not 24 raw rows here.

### 7. 🔦 Build the drillthrough detail page (this is where the code lives)
1. At the bottom, right-click the **Page 1** tab → **Duplicate page** (or add a new blank page).
2. Double-click the new page tab → rename it **`Section Detail`**.
3. On the new page, delete/deselect any copied visuals so it starts clean.
4. In the **Visualizations** pane, find the **Drill through** section (scroll down) → under **Add drill-through fields here**, drag **`Zone`** from `factbadgeaccess`.
   - This automatically adds a **back button** in the top-left of the detail page. Keep it.
5. Add a **Table** visual on this page. Into its **Columns** well drag **all four** `factbadgeaccess` fields in this order: **`Zone`**, **`ReadingId`**, **`AfterHoursScans`**, **`DiagnosticCode`**.
6. Add a title text box: `🔍 ZONE FORENSIC DETAIL`.
7. You should now see raw readings including the **`DiagnosticCode`** column — the Green Room rows contain **`BADGE-4471`** among decoy codes.

### 8. 🧭 Add a navigation hint on Page 1
1. Go back to **Page 1**.
2. **Insert** → **Text box** → type: `Right-click the anomalous zone → Drill through → Section Detail to read its badge code.`
3. Place it near the summary table.

### 9. 💾 Save and publish
1. **File → Save as** → name it **`Module 1 - Badge Anomaly`** → save locally.
2. **Home** ribbon → **Publish** (or **File → Publish → Publish to Power BI…**).
3. When asked for a destination, select the **next-bis-day-3** workspace → **Select**.
4. Wait for **"Success!"** → optionally click **Open in Power BI** to confirm it rendered.

**Recovery:** If `MurderAtMicrosoftSM` isn't in the semantic-models list, confirm you're signed into the right account and the workspace is on active capacity (see [00-SHARED-SETUP.md](00-SHARED-SETUP.md)). If the model appears but tables are empty, the semantic model may need a refresh — open it in the Fabric portal and click **Refresh now**.

---

## ✅ Completion criterion
Report **`Module 1 - Badge Anomaly`** is published to **next-bis-day-3** with: a Zone tile slicer, the `Avg After-Hours Scans` gauge, a per-zone summary table, and a **Section Detail** drillthrough page exposing `DiagnosticCode`. A player who drills into the **Green Room** can read **`BADGE-4471`**.
