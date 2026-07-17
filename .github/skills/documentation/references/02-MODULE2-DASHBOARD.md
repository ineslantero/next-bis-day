# 📹 02 — Module 2: The CCTV Gap Dashboard

**Owner role:** Player 3 (RTI Dashboard Creator)
**Depends on:** nothing — runs fully in parallel
**Screens needed:** Browser only (Fabric portal)

> *The murder happened somewhere the cameras couldn't see. Build's stage security ran continuous motion telemetry all afternoon — except for one ten-minute window when a camera went dark. Find the gap, read the assessment the system logged for that window, and you have the camera override code.*

**Puzzle outcome:** The player views a bar chart of camera activity binned every 10 minutes, spots the **missing bar at `13:10`** (the gap), then finds that window's row in the assessments table — the only one rated **`CLEAR`** — and reads the camera code **`CCTV-2058`** (format `CCTV-XXXX`).

**Data behind it:** Eventhouse **`MurderAtMicrosoftEH`**, tables **`SecurityTelemetry`** (activity: `Timestamp`, `CameraId`, `Zone`, `MotionScore`) and **`CameraWindows`** (assessments: `WindowStart`, `WindowEnd`, `Rating`, `Code`).

---

## Steps

### 1. 🆕 Create the Real-Time Dashboard
1. Open **https://app.fabric.microsoft.com** → switch to the **next-bis-day-3** workspace.
2. Top-left → **+ New item**.
3. In the item catalog, find **Real-Time Dashboard** (under *Real-Time Intelligence*) → click it.
4. Name it **`Murder at Microsoft Dashboard`** → **Create**.
5. You should now see an empty dashboard in edit mode.

### 2. 🔌 Connect the Eventhouse data source
1. Top ribbon → **Manage** tab → **Data sources**.
2. Click **+ Add** → choose **OneLake data hub** (or **KQL Database**).
3. Select **`MurderAtMicrosoftEH`** → **Connect**.
4. If prompted for a database, pick **`MurderAtMicrosoftEH`** → **Add** / **Done**.
5. You should now see the data source listed in the Data sources pane.

### 3. 📊 Add the activity bar chart tile (the gap detector)
1. Top ribbon → **Add bar chart**. A query editor opens.
2. Make sure the tile's data source dropdown = **`MurderAtMicrosoftEH`**.
3. Paste this query exactly:
   ```kql
   SecurityTelemetry
   | summarize EventCount = count() by bin(Timestamp, 10m)
   | order by Timestamp asc
   ```
4. Click **Run**. You should see rows of 10-minute buckets and counts.
5. Open the **Visualization** pane → **Data** section → confirm the **Y axis** is set to `EventCount` and the **X axis** is set to the `Timestamp` bucket.
6. Under **General**, add the tile **Title** → **`📡 CAMERA MOTION — 10-MINUTE ACTIVITY`**.
7. Click **Done** (top-right). You should see a run of bars with **one visibly missing bar** around **13:10** — that's the blind spot.

### 4. 🗂️ Add the assessments table tile (where the code lives)
1. Top ribbon → **Add table**.
2. Data source = **`MurderAtMicrosoftEH`**.
3. Paste this query exactly:
   ```kql
   CameraWindows
   | project WindowStart, WindowEnd, Rating, Code
   | order by WindowStart asc
   ```
4. Click **Run**. You should see ~51 window rows with a `Rating` column.
5. Under **General**, change the tile **Title** to **`🛡️ CAMERA WINDOW ASSESSMENTS`**.
6. Click **Done** (top-right). Most rows read `BLOCKED`, with decoy `CAUTION` and `MARGINAL` rows; exactly **one** row is rated **`CLEAR`** — and its `Code` is **`CCTV-2058`**. That `CLEAR` row's window matches the missing bar at **13:10**.

### 5. 💾 Save the dashboard
1. Top-left → **Save**.
2. Confirm both tiles render on the dashboard canvas.
3. Optionally drag the bar chart above the table so players read top-to-bottom: *see the gap → find the CLEAR row*.

**Recovery:** If a query returns *"table not found"*, confirm the data source is **`MurderAtMicrosoftEH`** and that the seed ingestion completed (the tables should have ~176 and ~51 rows). If tiles show no data, check the workspace is on **active capacity**.

---

## ✅ Completion criterion
Dashboard **`Murder at Microsoft Dashboard`** is saved with a **column/bar chart** tile (`SecurityTelemetry` binned 10m, one missing bar at 13:10) and a **table** tile (`CameraWindows` assessments). A player who spots the gap and reads the single **`CLEAR`** row obtains **`CCTV-2058`**.
