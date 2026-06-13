# Import the Flow — Legacy Package (.zip)

Prefer to start from the finished flow instead of building it by hand? Import the exported package
in [`flow/Power-Automate-Error-Handling-with-Adaptive-Cards.zip`](../flow/Power-Automate-Error-Handling-with-Adaptive-Cards.zip).

> **Important:** The package contains the **flow logic only** — it does **not** create the SharePoint
> lists. Build the four lists first (see [Part 0 of the build instructions](01-build-instructions.md#part-0--build-the-sharepoint-lists)),
> then import.

---

## Before you start

- A Microsoft 365 account with **Power Automate**, **SharePoint Online**, and **Microsoft Teams**.
- The four SharePoint lists from Part 0 already created on a site you own:
  `Sales Requests`, `Sales`, `Flow Configuration`, `Flow Log`.
- A **SharePoint** connection and a **Microsoft Teams** connection in your environment (the import
  will ask you to pick or create them).

---

## Step 1 — Download the package

Download [`flow/Power-Automate-Error-Handling-with-Adaptive-Cards.zip`](../flow/Power-Automate-Error-Handling-with-Adaptive-Cards.zip)
from this repo. **Do not unzip it** — Power Automate imports the `.zip` as-is.

> If your browser auto-extracts downloads, right-click → *Save link as…* to keep the `.zip` intact.

## Step 2 — Open the import screen

1. Go to **[make.powerautomate.com](https://make.powerautomate.com)**.
2. Confirm the **environment** selector (top-right) points at the environment where you want the flow.
3. In the left nav, select **My flows**.
4. On the toolbar, choose **Import → Import Package (Legacy)**.

## Step 3 — Upload and review

1. Select **Upload** and choose the `.zip` file.
2. Wait for the package to finish uploading and validating.
3. Under **Review Package Content**, you'll see the flow
   **Power Automate Error Handling with Adaptive Cards**.

## Step 4 — Set the import action for the flow

1. In the **Import setup** column for the flow, click the wrench/edit (**Select during import**).
2. Choose **Create as new** (recommended) and give it a name, **or** select **Update** to overwrite
   an existing flow of the same name.
3. Save.

## Step 5 — Map the connections

The package depends on two connectors — **SharePoint** and **Microsoft Teams**:

1. For each connection listed as *Related resources*, click **Select during import**.
2. Pick an existing connection, or choose **Create new** to sign in and create one.
   - If you create a new connection, you may need to open it in another tab, create it, then return
     and click the **refresh** icon so it appears in the list.
3. Make sure **both** the SharePoint and Teams connections show a valid selection.

## Step 6 — Import

1. Click **Import**.
2. Wait for the success banner, then go to **My flows** to find the imported flow.

---

## Step 7 — Reconnect to *your* lists and Teams

The exported actions still point at the original demo site and list IDs. Open the flow in **Edit**
and update each action to your own environment:

- **Trigger – When an item is created or modified** → set **Site Address** and **List Name**
  (`Sales Requests`).
- **Get item / Create item** actions → repoint **Site Address** and **List Name** to your
  `Sales Requests`, `Sales`, `Flow Configuration`, and `Flow Log` lists.
- Re-select any **Person**, **Lookup**, **Choice**, or **Status** column mappings that show as
  unresolved (they're tied to the original list's internal IDs).
- **Post card in a chat or channel** (Teams) actions → confirm the **Post as / Post in** options and
  the **Recipient** bindings resolve.
- Seed one **Flow Configuration** item: `Title = Sales Request`, `Stage = Prod`,
  `NotificationsEnabled = Yes`, `Admin = <you>`.

Then **Save** and run the three test scenarios from
[Part 5 of the build instructions](01-build-instructions.md#52-test-it-end-to-end).

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| *"Import Package (Legacy)" isn't on the menu* | Use the **My flows** page (not Solutions). Some tenants show it under **Import** → the legacy option. |
| Connection won't appear during mapping | Create it first under **Data → Connections**, then click **refresh** on the import screen. |
| Actions show missing list/column after import | Expected — repoint each SharePoint action to your own site and lists (Step 7). |
| Teams card posts fail | Re-authorize the Teams connection and confirm the recipient expression returns a valid email. |
| Flow doesn't trigger | Check the trigger condition `@equals(triggerBody()?['Status']?['Value'],'New')` and that new items have `Status = New`. |

> **Solutions alternative:** Legacy packages import outside a solution. If your organization works
> in **solutions** with environment variables and connection references, consider rebuilding the
> flow inside a solution using [the build instructions](01-build-instructions.md) rather than the
> legacy import.
