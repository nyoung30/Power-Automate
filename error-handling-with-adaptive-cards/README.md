# Elevate Your Power Automate Workflows — Error Handling with Adaptive Cards

Session resources for building a resilient Power Automate cloud flow that uses the
**Try – Catch – Finally** pattern and **Adaptive Cards in Microsoft Teams** to turn silent
failures into notifications people actually see and can act on.

> **Speaker:** Norm Young — Microsoft MVP, Power Platform Solution Architect at AvePoint
> **Topics:** Power Automate · Microsoft Teams · Microsoft Lists / SharePoint · Adaptive Cards

---

## Why error handling matters

Everything eventually fails — and even our best-built flows are no exception. Power Automate's
native error messages aren't timely or informative, so failures go unnoticed until a business
process breaks. Three reasons error handling moves from *nice to have* to *essential*:

- **Responsiveness** — A flow is rarely the whole story; it's one step in a larger process.
  Delays cascade downstream. (Imagine a flow that provisions building-security access — any
  delay means staff can't get into the building.)
- **Importance** — Business-critical processes increasingly depend on flow outcomes, and
  citizen development is rising. Error handling adds the resiliency that keeps the business running.
- **Support** — Out of the box, only flow *owners* are notified, and flows that run as a service
  account may never have those notices read. Route errors to the team that supports the process.

---

## The demo scenario

A flow triggered from a SharePoint **Sales Requests** list creates a **Sales** record from a new
item — but **only when the difference between the Opportunity Amount and the Quote Amount is
within ±15%**. We harden this flow so that:

- **Happy path** → the Sales record is created and the run is logged as *Success*.
- **Business-rule failure** (variance outside ±15%) → the **requestor** gets an actionable
  Adaptive Card telling them what to fix, logged as *Business Rule Failed*.
- **Flow failure** (an action errors out) → **Admins** get a debug-rich Adaptive Card with a deep
  link to the failed run, logged as *Flow Failed*.

---

## What's in this repo

| Path | What it is |
|------|------------|
| [`docs/01-build-instructions.md`](docs/01-build-instructions.md) | Build the entire flow from scratch — lists, variables, scopes, and cards |
| [`docs/02-import-the-flow.md`](docs/02-import-the-flow.md) | Import the exported flow package instead of building it by hand |
| [`flow/Power-Automate-Error-Handling-with-Adaptive-Cards.zip`](flow/) | The exported Power Automate cloud flow (legacy ZIP import package) |
| [`cards/user-rejected-card.json`](cards/user-rejected-card.json) | Adaptive Card JSON for the end-user "request rejected" notification |
| [`cards/admin-error-card.json`](cards/admin-error-card.json) | Adaptive Card JSON for the admin "flow error" notification |

---

## Choose your path

- **Want to learn the pattern?** Follow [the build instructions](docs/01-build-instructions.md)
  and assemble the flow step by step.
- **Want it running quickly?** Follow [the import instructions](docs/02-import-the-flow.md) to
  bring in the exported package, then reconnect SharePoint and Teams.

Either way you'll need the four SharePoint lists described in the build guide — the import package
contains the flow logic but **not** the lists.

---

## Prerequisites

- A Microsoft 365 account with **Power Automate**, **SharePoint Online**, and **Microsoft Teams**.
- Permission to create SharePoint lists on a site you own.
- The **Adaptive Cards Designer** — [adaptivecards.io/designer](https://adaptivecards.io/designer) —
  for customizing the cards.

---

## A note on Dataverse

This demo uses SharePoint lists so the data binds straight onto the cards. If your data lives in
**Dataverse**, you'll reach for `first()`, `formatNumber()` / `FormattedValue`, and a working
knowledge of **FetchXML**. The free community **[XrmToolBox](https://www.xrmtoolbox.com/)** and its
*FetchXML Builder* let you build and test the query visually, then paste the FetchXML straight into
your flow.

---

## Connect

Norm Young · Microsoft MVP · Power Platform Solution Architect, AvePoint
[normyoung.ca](https://normyoung.ca) · GitHub [@norm-young](https://github.com/norm-young) · X [@stormin_30](https://x.com/stormin_30)
