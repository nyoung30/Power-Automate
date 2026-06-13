# Build Instructions — Error Handling with Adaptive Cards

This guide walks you through building the **Power Automate Error Handling with Adaptive Cards** flow
from scratch. It follows the same five parts used in the session.

- [Part 0 — Build the SharePoint lists](#part-0--build-the-sharepoint-lists)
- [Part 1 — Trigger and variables](#part-1--trigger-and-variables)
- [Part 2 — Try, Catch, Finally scopes](#part-2--try-catch-finally-scopes)
- [Part 3 — Business logic and the end-user card](#part-3--business-logic-and-the-end-user-card)
- [Part 4 — Config, debug, and the admin card](#part-4--config-debug-and-the-admin-card)
- [Part 5 — Logging and the end-to-end run](#part-5--logging-and-the-end-to-end-run)

> **Tip:** Prefer not to build by hand? See [02-import-the-flow.md](02-import-the-flow.md) to import
> the exported package. You still need the lists in Part 0 either way.

---

## The pattern at a glance

A resilient flow follows a simple three-block pattern, each wrapped in a **Scope**:

| Block | Purpose |
|-------|---------|
| **TRY** | Run the primary actions that make up the workflow. |
| **CATCH** | Catch any errors and take follow-up actions for known issues. |
| **FINALLY** | Run a final set of fail-safe actions — **always**, regardless of what happened above. |

The plumbing that makes this work:

- **Scope actions** encapsulate a block of actions and inherit their last terminal status.
- **Configure run after** controls *whether* an action runs based on the previous action's status
  (*is successful · has failed · is skipped · has timed out*).
- **Variables** carry debug info and the final run status across the scopes.

---

## Part 0 — Build the SharePoint lists

Create these four lists on a SharePoint site you own (the demo uses a site called **demo-site**).
Column **internal names** matter — match them exactly so expressions resolve.

### List 1 — `Sales Requests` (the trigger source)

| Column | Type | Notes |
|--------|------|-------|
| `Title` | Single line of text | Built-in |
| `Status` | Choice | Include at least **New** (the flow triggers on `New`) |
| `OpportunityAmount` | Currency (or Number) | |
| `QuoteAmount` | Currency (or Number) | |
| `Requestor` | Person or Group | Single selection |

> The built-in **Link to item** value from the trigger is used for the card button — you don't need
> a custom URL column.

### List 2 — `Sales` (the record the flow creates)

| Column | Type | Notes |
|--------|------|-------|
| `Title` | Single line of text | Built-in |
| `SourceRequestID` | Lookup → `Sales Requests` | Stores the originating request ID |
| `Amount` | Currency (or Number) | |
| `Requestor` | Person or Group | |
| `ClosedDate` | Date and Time (Date only) | |

### List 3 — `Flow Configuration` (data-driven admin routing)

One item per production flow + admin. Drives **who** gets notified when a flow fails.

| Column | Type | Notes |
|--------|------|-------|
| `Title` | Single line of text | The flow name, e.g. **Sales Request** |
| `Stage` | Choice (or text) | e.g. **Prod**, **Dev** |
| `NotificationsEnabled` | Yes/No | Set **Yes** to receive alerts |
| `Admin` | Person or Group | The admin to notify |

> Seed one item: `Title = Sales Request`, `Stage = Prod`, `NotificationsEnabled = Yes`,
> `Admin = <you>`. The Catch scope filters on exactly these three values.

### List 4 — `Flow Log` (central logging)

One item per flow execution.

| Column | Type | Notes |
|--------|------|-------|
| `Title` | Single line of text | |
| `RunDateTime` | Date and Time | |
| `Status` | Choice | **Success**, **Business Rule Failed**, **Flow Failed** |
| `RunLink` | Single line of text (or Hyperlink) | Deep link to the run |
| `Debug` | Multiple lines of text | The action-by-action trail |
| `RequestID` | Lookup → `Sales Requests` | The originating request |

---

## Part 1 — Trigger and variables

### Trigger — When an item is created or modified (SharePoint)

1. Add the **SharePoint → When an item is created or modified** trigger.
2. **Site Address:** your demo site. **List Name:** `Sales Requests`.
3. Add a **Trigger Condition** (trigger → ⚙ Settings) so the flow only runs for new requests:

   ```
   @equals(triggerBody()?['Status']?['Value'],'New')
   ```

### Variables (declare these **outside** any scope, before the Try)

Add three **Initialize variable** actions, in this order:

| Order | Name | Type | Initial value |
|-------|------|------|---------------|
| 1 | `varDebug` | String | `List of action names and statuses:` (followed by a couple of blank lines) |
| 2 | `varRunStatus` | String | `Flow Failed` |
| 3 | `varVariance` | Float | *(leave empty)* |

> **Why default `varRunStatus` to `Flow Failed`?** Defaulting to the pessimistic value means any
> unhandled crash still logs correctly with no extra work. The Try branches override it to
> *Success* or *Business Rule Failed*; the Catch sets it back to *Flow Failed*.

> **Boundary rule:** Variable declarations must live **outside** the Scope actions — never inside them.

---

## Part 2 — Try, Catch, Finally scopes

1. Add three **Scope** actions after the variables and rename them:
   - `Scope - Try`
   - `Scope - Catch`
   - `Scope - Finally`
2. Wire up **Configure run after** (each scope → ⋯ → *Configure run after*):
   - **`Scope - Catch`** runs after `Scope - Try` → check **has failed** *(only)*.
   - **`Scope - Finally`** runs after `Scope - Catch` → check **is successful, has failed,
     is skipped, has timed out** *(all four)*, so it always runs.

You'll fill each scope with actions in the parts below.

---

## Part 3 — Business logic and the end-user card

All actions in this part go **inside `Scope - Try`**, in order.

### 3.1 Get item — Sales Request

- **SharePoint → Get item.** Site = demo site, List = `Sales Requests`,
  **Id** = `ID` (from the trigger).
- Re-reading the item gives you the full set of fields to bind onto the card.

### 3.2 Set variable — varVariance

- **Set variable** → `varVariance`. Value (expression):

  ```
  div(
     sub(
        outputs('Get_item_-_Sales_Request')?['body/OpportunityAmount'],
        outputs('Get_item_-_Sales_Request')?['body/QuoteAmount']
     ),
     outputs('Get_item_-_Sales_Request')?['body/OpportunityAmount']
  )
  ```

### 3.3 Condition — within ±15%?

- Add a **Condition**. Switch it to **edit in advanced mode** and use:

  ```
  @and(greaterOrEquals(variables('varVariance'), -0.15), lessOrEquals(variables('varVariance'), 0.15))
  ```

  (equals `true`)

#### If yes → create the Sales record

1. **SharePoint → Create item** in the `Sales` list:
   - `Title` = `Get item` → **Title**
   - `SourceRequestID Id` = `Get item` → **ID**
   - `Amount` = `Get item` → **QuoteAmount**
   - `Requestor Claims` = `Get item` → **Requestor Claims**
   - `ClosedDate` = expression `utcNow('yyyy-MM-dd')`
2. **Set variable** → `varRunStatus` = `Success`.

#### If no → notify the requestor

1. **Microsoft Teams → Post card in a chat or channel**:
   - **Post as:** Flow bot · **Post in:** Chat with Flow bot
   - **Recipient:** `Get item` → **Requestor Email**
   - **Adaptive Card:** paste [`cards/user-rejected-card.json`](../cards/user-rejected-card.json)
     and confirm the placeholders bind to your dynamic content (see below).
2. **Set variable** → `varRunStatus` = `Business Rule Failed`.

**Dynamic values used by the user card:**

| Card placeholder | Bind to |
|------------------|---------|
| Card title | `Get item` → **Title** |
| Status | `Get item` → **Status Value** |
| Opportunity Amount | `formatNumber(outputs('Get_item_-_Sales_Request')?['body/OpportunityAmount'], 'C')` |
| Quote Amount | `formatNumber(outputs('Get_item_-_Sales_Request')?['body/QuoteAmount'], 'C')` |
| Percent Difference | `concat(formatNumber(mul(variables('varVariance'), 100), 'N2'), '%')` |
| **Open Opportunity** button URL | `Get item` → **Link to item** (`body/{Link}`) |

> Build and tweak the card visually in the **[Adaptive Cards Designer](https://adaptivecards.io/designer)**,
> start from a sample, and use placeholders for dynamic data so it's easy to update inside the flow.

---

## Part 4 — Config, debug, and the admin card

All actions in this part go **inside `Scope - Catch`**, in order. The Catch only runs when the
Try scope fails.

### 4.1 Set variable — varRunStatus (Failed)

- **Set variable** → `varRunStatus` = `Flow Failed`.

### 4.2 Get items — Flow Configuration

- **SharePoint → Get items** from `Flow Configuration` with this **Filter Query**:

  ```
  Title eq 'Sales Request' and Stage eq 'Prod' and NotificationsEnabled eq 1
  ```

  This returns only the admins who should be alerted for this flow.

### 4.3 Apply to each — build varDebug

- **Apply to each** over: expression `result('Scope_-_Try')`
  *(the action-by-action result of the Try scope).*
- Inside, add a **Condition**: `status` **is not equal to** `Failed`.
  - **If yes** (not failed) → **Append to string variable** `varDebug`:

    ```
    - Name: @{items('Apply_to_each_-_varDebug')?['name']}
    - Status: @{items('Apply_to_each_-_varDebug')?['status']}
    ```

  - **If no** (failed) → **Append to string variable** `varDebug`, **bolded** so it stands out:

    ```
    - **Name: @{items('Apply_to_each_-_varDebug')?['name']}**
    - **Status: @{items('Apply_to_each_-_varDebug')?['status']}**
    ```

### 4.4 Compose — Get Workflow Details

- **Compose** with input expression: `workflow()`

### 4.5 Compose — Flow Run URL

- **Compose** that builds a deep link straight to the failed run:

  ```
  https://make.powerautomate.com/manage/environments/@{outputs('Compose_-_Get_Workflow_Details')?['tags']?['environmentName']}/flows/@{outputs('Compose_-_Get_Workflow_Details')?['name']}/runs/@{outputs('Compose_-_Get_Workflow_Details')?['run']?['name']}
  ```

  This is the game-changer for admins — they jump straight to the exact failed run instead of
  hunting through run history.

### 4.6 For each — Notify Admins

- **Apply to each** over: `Get items - Flow Configuration` → **value**.
- Inside, **Microsoft Teams → Post card in a chat or channel**:
  - **Post as:** Flow bot · **Post in:** Chat with Flow bot
  - **Recipient:** current item → **Admin Email**
  - **Adaptive Card:** paste [`cards/admin-error-card.json`](../cards/admin-error-card.json).

**Dynamic values used by the admin card:**

| Card placeholder | Bind to |
|------------------|---------|
| Card title | current config item → **Title** |
| Debug body | `varDebug` |
| **Open Flow** button URL | `Compose - Flow Run URL` (outputs) |

---

## Part 5 — Logging and the end-to-end run

This action goes **inside `Scope - Finally`** — it runs on every execution.

### 5.1 Create item — Flow Log

- **SharePoint → Create item** in `Flow Log`:

  | Field | Value |
  |-------|-------|
  | `Title` | `@{outputs('Get_item_-_Sales_Request')?['body/Title']} - @{outputs('Get_item_-_Sales_Request')?['body/Requestor/DisplayName']} (@{outputs('Get_item_-_Sales_Request')?['body/Requestor/Email']}) - @{formatDateTime(utcNow(), 'yyyy-MM-dd')}` |
  | `RunDateTime` | `utcNow()` |
  | `Status Value` | `varRunStatus` |
  | `RunLink` | `Compose - Flow Run URL` (outputs) |
  | `Debug` | `varDebug` |
  | `RequestID Id` | `Get item` → **ID** |

### 5.2 Test it end to end

1. **Happy path** — add a `Sales Requests` item with `Status = New` where Opportunity and Quote
   amounts are within ±15%. → A `Sales` record is created; `Flow Log` shows **Success**.
2. **Business-rule failure** — add an item where the amounts differ by more than 15%. → The
   requestor gets the rejection card; `Flow Log` shows **Business Rule Failed**.
3. **Flow failure** — force an error (e.g. temporarily break the `Create item - Sales` action).
   → Admins get the debug-rich card with a link to the failed run; `Flow Log` shows **Flow Failed**.

---

## Reference — expressions used

| Expression | Where |
|------------|-------|
| `@equals(triggerBody()?['Status']?['Value'],'New')` | Trigger condition |
| `div(sub(Opportunity, Quote), Opportunity)` | varVariance |
| `@and(greaterOrEquals(varVariance,-0.15), lessOrEquals(varVariance,0.15))` | ±15% condition |
| `utcNow('yyyy-MM-dd')` | Sales `ClosedDate` |
| `result('Scope_-_Try')` | Loop source for varDebug |
| `workflow()` | Current run details |
| `formatNumber(amount, 'C')` | Currency on the user card |
| `concat(formatNumber(mul(varVariance, 100), 'N2'), '%')` | Percent difference on the user card |
