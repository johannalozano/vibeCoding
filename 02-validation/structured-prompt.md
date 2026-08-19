## CONTEXT

## Context, The Internal Tool Nobody Uses

**Role:**
You are an internal Product Manager working with a startup Customer Success team that operates across the full customer lifecycle. The team helps move sales opportunities toward close, manages onboarding and follow-up tasks, and tracks support tickets and customer issues inside the CRM.

**Hypothesis:**
If we replace the CRM's current workflow-heavy screen with a prioritized daily action queue that tells the Customer Success team what needs attention next, why it matters, and lets them complete common updates with minimal data entry, then the team will use the CRM as part of their daily workflow instead of maintaining shadow spreadsheets.

The new experience should reduce the effort required to understand priorities and complete routine CRM updates. It should prioritize work primarily using **account value** and **committed date**.

**Users say:**

* "It's faster to keep my deals in a spreadsheet than to fight the CRM's eight required fields." (Account Exec, 2 logins / month)
* "Logging activity feels like data entry for management, not something that helps me sell." (Senior AE)
* "If it could just tell me who to call next, I'd open it every morning." (SDR, occasional user)

**Metrics:**

* 18% weekly active adoption across licensed seats.
* 8 required fields to log a single ticket or account update.
* 11 min average task time for "update a deal or ticket" vs. 2-min target.
* 63% of reps keep a shadow spreadsheet alongside the CRM.

**Success criteria:**

* Increase weekly active CRM adoption from **18% to at least 40%**.
* Reduce the average time to complete a routine deal, onboarding, or ticket update from **11 minutes to under 3 minutes**.
* Reduce reliance on shadow spreadsheets by making the CRM the fastest place to understand and complete daily work.

## REFERENCE

**Visual style:**
Editorial / neo-brutalist operational UI. High contrast, bold typography, thick black borders, minimal decoration, compact information density, and intentional signal colors:

* Yellow = due today
* Orange/red = overdue
* Black/white = neutral/status

It should feel like a fast operational command center, not a traditional corporate CRM.

**Filtering & hierarchy:**
The screen is a **daily action queue** that combines different types of Customer Success work in one prioritized view.

Queue items may include:

* Deal follow-ups
* Onboarding tasks
* Support tickets
* Customer/account follow-ups

The system should rank items primarily by:

1. **Committed date / urgency**
2. **Account value**

Overdue or due-today work should surface first, with higher-value accounts prioritized when urgency is comparable.

Include:

**Top counters**

* Left
* Overdue
* Done

These counters represent all actionable queue items for the selected day, regardless of whether they are deals, onboarding tasks, or support tickets.

**Global search**

Search across accounts, contacts, deals, tickets, and tasks.

**Time filters**

* Today
* Overdue
* Upcoming
* All

**Today** represents the complete prioritized work queue recommended for the current day, including overdue items and items committed for today.

Do not include a separate **Due Today** filter, since it would overlap with the Today view and create unnecessary ambiguity.

**Work-type filters**

* Deals
* Onboarding
* Support
* Follow-ups

**Lifecycle / stage filters**

Because the same Customer Success team supports both pre-sale and post-sale work, stage filters should adapt to the item type rather than assume everything is a sales opportunity.

Examples:

* Sales: Qualified, Proposal, Negotiation, Closing
* Onboarding: Not Started, In Progress, Blocked, Complete
* Support: Open, Waiting, Escalated, Resolved

**Each queue card should show:**

* Contact + company/account
* Work type: Deal, Onboarding, Support, or Follow-up
* Current stage/status
* Account value
* Committed date
* Next action / short context
* Last touch or last update
* Urgency state
* Clear reason the item is prioritized

Example priority explanation:
**Due today · $88k account**

**Inline actions:**

* Log Call
* Add Note
* Complete / Update Status
* Snooze

These actions must be lightweight and should **not recreate the current eight-required-field workflow**.

For routine actions, require only the minimum information necessary. Additional CRM fields can be optional, auto-populated from existing context, or edited later.

Completing a common action should ideally take one or two interactions without leaving the queue.

## CONSTRAINTS

**Core design principle:**
Make it possible to scan in seconds:

1. **What needs my attention today?**
2. **Why is this item important?**
3. **What should I do next?**
4. **Can I complete that action without navigating through the CRM?**

The experience should reduce cognitive load and data-entry burden rather than simply redesign the existing CRM interface.

The queue must successfully combine sales, onboarding, and support work without making users mentally switch between separate tools or dashboards.

Priority should always remain understandable. Users should be able to see why one item appears above another based on **committed date and account value**.

The prototype should focus on the **daily execution workflow**, not on recreating the entire CRM.

## OUTPUT

Produce a **high-fidelity, clickable prototype of the CRM daily task screen**.

The prototype should demonstrate:

* A mixed queue containing deal follow-ups, onboarding tasks, and support tickets
* Prioritization based on committed date and account value
* Time, work-type, and lifecycle/status filtering
* Search
* Top-level Left / Overdue / Done counters
* Clear priority explanations on queue items
* Lightweight inline actions
* At least one complete interaction flow showing a user updating or completing an item without entering a long CRM form

**Single page, responsive, and ready to share via link.**
