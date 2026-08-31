# CSR

Source: https://chatgpt.com/share/6a94fd0e-5848-83e8-a33b-d3ef7da05789
Model: ChatGPT
Exported: 2026-08-31 14:52 +07

---


## You

How do you usually approach designing UIs for large queues are there UX patterns for that?

## ChatGPT

Yes. Large queues are a well-established UX problem, and the best designs usually treat the queue less like a “big list” and more like an **operational workspace for deciding what to do next**.

A useful starting point is to design around four jobs: **triage, focus, act, and recover context**.

* **Triage:** Help users quickly distinguish urgent, blocked, stale, high-value, or unusual items. Common patterns are priority indicators, SLA/age columns, smart sorting, grouped sections, filters, saved views, and exception-based highlighting.
* **Focus:** Don’t make users mentally process the whole queue. Give them views such as “Assigned to me,” “Due today,” “Needs review,” or “Unassigned.” For very large queues, default views should generally show the _most actionable subset_, not everything.
* **Act:** Optimize repeated work. Bulk selection, batch actions, keyboard shortcuts, inline status changes, quick assign, and “next item” workflows are extremely valuable when someone processes dozens or hundreds of items.
* **Recover context:** Queue work gets interrupted. Preserve filters, scroll position, selection, and opened-item state. Show what has changed since the user last looked.

A common layout looks like this:

**Queue health / summary**  
→ 24 overdue · 81 unassigned · 12 blocked

**View controls**  
→ My work | Unassigned | Due soon | Saved views  
→ Search · Filters · Sort

**Queue**  
→ Dense rows with only decision-critical information  
→ Checkbox · priority · title · owner · age · status · key metadata

**Detail pane**  
→ Selecting a row opens a side panel instead of navigating away

The **list + detail pane** pattern is especially strong for large queues because users can inspect an item without losing their place. Email clients, ticketing systems, moderation tools, CRMs, and admin consoles use this extensively.

For scale, I’d usually avoid pure infinite scrolling when users need to understand position, revisit items, or work methodically. Pagination, virtualized tables, or “load more” can provide better orientation. Infinite scroll works better for browsing than for operational work.

Another important pattern is **progressive disclosure**. A queue row should answer only the questions needed to decide, “Do I open this?” Put secondary fields, history, notes, and controls in the detail view. Otherwise large queues become unreadable spreadsheets.

For queues that involve teams, there are a few additional patterns that matter a lot: clear ownership, presence/locking if two people might act on the same item, assignment/reassignment, audit history, and visible queue-level metrics such as oldest item, throughput, backlog, and SLA risk.

One principle I find particularly useful is:

> **Highlight exceptions, not normality.**

If 98% of items are “normal,” avoid decorating all 98%. Make overdue, blocked, escalated, or unusual items visually distinct. That dramatically improves scanability.

And I’d generally design for **three usage modes** separately: a manager asking “Is the queue healthy?”, an operator asking “What should I work on next?”, and an investigator asking “What’s going on with this particular item?” Trying to serve all three with one overloaded table is where many queue UIs become painful.

## You

What are great UXes in that? Say it’s incoming tickets are there’s millions of tickets to few tickets and some designer if they’ve ecompassed all that?

## ChatGPT

Yes — and the key insight is that **“3 tickets” and “3 million tickets” should not really be the same UI**. The strongest systems progressively change the abstraction level as volume grows.

I’d study these five:

| Product                     | Particularly good at              | Pattern worth stealing                                                                  |
| --------------------------- | --------------------------------- | --------------------------------------------------------------------------------------- |
| **Linear**                  | Small → medium queues             | Extremely fast triage, keyboard navigation, peek/detail without losing position         |
| **Intercom**                | Medium → large support operations | Inbox → team inboxes → saved views → automated routing                                  |
| **Zendesk**                 | Large customer-support operations | Separates an agent's personal workload from global queue health                         |
| **Jira Service Management** | Complex queues / many teams       | Explicit SLA queues, custom queue definitions, bulk operations                          |
| **ServiceNow**              | Very large enterprises            | Users increasingly receive _assigned work_ rather than browsing the universe of tickets |

### The UX should effectively transform with scale

Imagine the same system growing.

**3 tickets**

```
Inbox                         3

○ Can't reset password
○ Invoice appears incorrect
○ API authentication error
```

You probably don't need dashboards, routing, saved filters or fancy prioritization. Every extra control is noise.

**300 tickets**

```
INBOX

My tickets         12
Unassigned         38
Urgent              4
Waiting on customer 21

─────────────────────────────
⚠ API authentication    18m
  Enterprise · P1

  Cannot reset password 42m
  Free · P3
```

Now the important abstraction is **views**.

Intercom does this quite explicitly: it provides things like _All_, _Unassigned_, personal inboxes and configurable saved views, while workflows can automatically route incoming tickets to appropriate teams.

**30,000 tickets**

At this point, I don't want:

> Tickets — 30,381

as my primary interface.

I want:

```
YOUR WORK
Needs response             12
SLA risk                    3
Escalated                   1

TEAM HEALTH
Unassigned                126
SLA risk                   28
Blocked                    14

INCIDENTS
Login outage              847 reports
Billing issue             391 reports
```

Zendesk is interesting here. Its Agent Home deliberately surfaces an agent's assigned work and supports sorting based on things like SLA and conversation activity, while its separate real-time queue dashboard deals with backlog, wait times, agent availability, SLA state and distribution.

That's a very good architectural decision:

**worker UX ≠ manager UX.**

### And at 3 million tickets, the queue almost disappears

This is where many designs go wrong.

Nobody should scroll through:

```
1–50 of 3,284,191
```

unless they're doing forensic search.

The UX becomes:

```
                3.28M TICKETS
                     │
              automated routing
                     ↓
       ┌─────────────┼─────────────┐
    Billing       Technical      Abuse
    428k            911k          62k
       │              │
       ↓              ↓
   classifiers     classifiers
       │              │
       ↓              ↓
     teams          teams
       │
       ↓
   individual agent
       │
       ↓
   "Here are your
    next 11 items"
```

ServiceNow is probably the clearest example of this philosophy. Its Advanced Work Assignment can route work based on queue rules, agent availability, capacity and skills, with agents ultimately receiving relevant assignments in their workspace rather than manually searching the entire corpus.

So at massive scale, **routing itself becomes part of the UX**.

### Jira contains a tiny but revealing design detail

Jira Service Management allows customizable queues based on filter criteria, SLA ordering and other attributes. But it stops displaying an exact queue count after **999**, showing `999+` instead.

That's more profound than it seems.

Once the number is sufficiently large:

> **the exact cardinality stops helping the operator make a decision.**

`1,385,729 tickets`

is usually worse information than:

```
27 need attention now
184 breach SLA today
8 have no qualified owner
3 incidents account for 71% of volume
```

That's the transition from **database UX → operational UX**.

---

If I were designing one system that truly encompassed **1 ticket → millions**, I'd make it one information architecture with roughly four zoom levels:

```
ORGANIZATION
“How healthy is the whole system?”
        ↓
QUEUE
“Where does attention need to go?”
        ↓
MY WORK
“What should I do next?”
        ↓
TICKET
“What do I need to know to resolve this?”
```

Crucially, you can drill downward and back upward without losing context.

So I wouldn't attempt to build the world's best giant table. I'd build a **zoomable work system** where the representation naturally changes:

**few → objects**
**hundreds → lists**
**thousands → queues**
**millions → aggregates + routing + exceptions**

Linear is probably the UX I'd steal most from at the **individual operator** level; Intercom/Zendesk for the **team level**; and ServiceNow's underlying model is worth understanding for the **massive-scale orchestration** level. Linear's triage model in particular treats incoming issues as a separate review stage before they enter the team's workflow, and makes that review highly keyboard-driven.

If you're designing this yourself, the most interesting challenge is actually making **the same product gracefully move between these four zoom levels without feeling like four different applications**.

## You

There’s something more elegant than zendesk with same volume?

## ChatGPT

Yes. If **“Zendesk-scale, but substantially more elegant”** is the requirement, I’d look at **Kustomer first, then Intercom**.

### 1. Kustomer — probably the closest answer

Kustomer has a particularly interesting UX because it questions Zendesk’s fundamental abstraction.

Zendesk is roughly:

**ticket → ticket → ticket → customer**

Kustomer is closer to:

**customer → continuous timeline → conversations/issues**

The agent sees the customer's history, transactions, conversations, events, etc. as one timeline instead of opening a pile of disconnected tickets.

And there is real evidence that it can operate at the scale you're talking about. Glovo moved to Kustomer with **3,800+ agents across 14 service centers and 13 languages**, while planning for **millions of customer conversations per month**. They had previously been operating across more rigid support systems.

So if you're asking:

> “Has someone built a cleaner interaction model _and_ actually thrown millions of conversations at it?”

**Kustomer is a very good case study.**

Its most interesting design idea isn't the visual styling. It's eliminating a lot of the concept of **“the giant queue of tickets.”**

---

### 2. Intercom — I think the operator UX is better

If I were purely studying interaction design, I'd probably spend more time in **Intercom**.

Its current Inbox is very deliberately optimized around:

```
┌───────────────┬──────────────────┬────────────────────────┐
│ Work views    │ Conversations    │ Current conversation   │
│               │                  │                        │
│ Your inbox  4 │ Michael      2m  │ Customer context       │
│ Mentions    10│ Alessandra   9m  │                        │
│ Unassigned  18│ Whitney     12m  │ Conversation           │
│               │                  │                        │
│ Email       132│ ...              │ Reply / action         │
│ WhatsApp     63│                  │                        │
└───────────────┴──────────────────┴────────────────────────┘
```

There is very little navigation penalty. You triage, inspect and act within one workspace. It supports keyboard navigation, bulk actions, routing and configurable views rather than making the user bounce between a ticket table and ticket pages.

And this isn't just “nice startup software.” Intercom publicly describes its infrastructure as supporting **millions of records and thousands of active agents**, and in 2026 reported very high peak request volumes across its systems.

So my shorthand would be:

**Kustomer = strongest proof of huge support-operation scale + better information architecture.**

**Intercom = strongest polished agent experience + serious scale.**

---

### 3. Front — perhaps the most visually elegant

Front's 2025 redesign is worth studying. They explicitly simplified the Inbox because increasing capabilities had created context-switching and crowded interfaces.

It feels much more like:

**Superhuman/email client + collaboration + ticketing**

than:

**enterprise helpdesk database.**

And some details are excellent. For example, duplicate conversations entering multiple shared inboxes can collapse into one shared conversation so teams aren't independently processing the same thing.

Front now has proper ticket states, shared inboxes, automation, enterprise controls, analytics and unlimited rules/macros at its enterprise tier.

But there's an important caveat:

**I wouldn't automatically assume Front is equivalent to Zendesk/Kustomer for the absolute extreme end of contact-center complexity.**

For a few hundred agents and very large email/message volumes, absolutely worth investigating. For thousands of agents, dozens of business units, extremely complicated routing/SLA/compliance requirements, I'd validate your exact workload.

---

### And then there's an even more interesting category

Products such as **Plain** are pushing the UI further.

Plain effectively says that B2B support shouldn't even be built around an old-school helpdesk abstraction; it positions support as composable infrastructure, with teams able to build workflows around their own customer model.

The UX tends to feel closer to Linear than Zendesk.

But I would separate:

**“This is where support UX should go”**

from

**“This has publicly demonstrated 3,000 agents processing millions of conversations/month.”**

For the latter, Kustomer has much stronger evidence.

---

## The pattern I'd steal from Kustomer

There's a deeper idea here that I think relates directly to what you were asking earlier.

Suppose there are **2.8 million tickets**.

Zendesk-ish thinking says:

```
TICKETS

2,831,429 total

[ Search                        ]

Views
────────────────
My tickets
Unassigned
Urgent
Billing
Technical
...
```

A more elegant system says:

```
TODAY

14 need your attention
 3 approaching SLA
 2 customer replies
 1 escalation

────────────────────────

CUSTOMER
Acme Corp

     Timeline
        │
 Payment failed
        │
 Support conversation
        │
 Retry succeeded
        │
 Customer replied
        │
       NOW
```

The **2.8 million** exists in the database.

It doesn't need to exist in the agent's head.

That's the biggest UX improvement I'd take away from the newer generation of systems.

### So I'd study these in this order

| If you're studying…                                      | Look at                                                  |
| -------------------------------------------------------- | -------------------------------------------------------- |
| **Millions of conversations + thousands of agents**      | **Kustomer**                                             |
| **Best agent interaction model**                         | **Intercom**                                             |
| **Cleanest inbox/team collaboration UX**                 | **Front**                                                |
| **What a post-Zendesk B2B support tool could feel like** | **Plain**                                                |
| **Extremely complex enterprise operations**              | Zendesk / ServiceNow, but mainly for capability coverage |

And **Kustomer's Glovo implementation** is probably the case I would dissect most carefully for your original question, because _millions of conversations/month + 3,800 agents_ takes this out of the realm of pretty Dribbble concepts.

The really interesting design exercise would be to combine **Linear's command/triage UX + Intercom's three-pane workspace + Kustomer's customer timeline**. That gets surprisingly close to what I think a genuinely excellent million-ticket system should look like.

## You

Can you generate a bunch of images that, like do all three? Take a very, very simple case study, like there is like a bunch of agents that we're handling in enterprise. Then there is human CSR's view, where we combine Linear, Command, TriGUX, Intercoms, three pipeline workspace, and customer timeline, and like what do screens look like? Can you generate the images?

## ChatGPT

*(Image generation in the share did not export as files. The later turns ask for the same screens as a five-screen command center; that text is in `neve-center.md`.)*
