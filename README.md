# AROMA — Cross-Functional Revenue Operations Transformation

**AROMA English for Working Professionals**

---

## 1. Project Summary

Designed and led the rollout of an enterprise **Revenue Operations (RevOps)** system for AROMA — a multi-branch English-training company operating across Hanoi and Ho Chi Minh City — replacing fragmented, branch-specific sales processes and siloed spreadsheets with a **single, standardized customer process running end-to-end across five functions: Marketing → Telesales → Sales → Training → Accounting.**

The project went well beyond redrawing a process map. It delivered:

- A standardized **Lead-to-Cash pipeline** with clearly defined stages and enforced transition rules.
- A **relational, multi-object data model** (Deal ↔ Appointment ↔ Contact ↔ Receipt ↔ Customer) with automated cross-object synchronization.
- An automated **lead-routing engine** with source-based distribution and a built-in **data recycling** mechanism for un-worked leads.
- A **revenue attribution model** based on UTM tracking and per-stage contribution.
- A **sales compensation engine** wired directly to operational data, with multi-touch allocation, time-decay attribution, and quality adjustment.
- A **cross-functional BI reporting layer** giving executives full-funnel visibility from marketing spend to revenue and receivables.

The system was built on a heavily configured no-code / low-code, issue-based work-management platform (workflows, custom fields, automations, JQL, screens, cross-object sync) — effectively a **self-built CRM / Revenue Operations platform**, rather than an off-the-shelf SaaS CRM.

**Role:** COO — architect and delivery lead for the entire system, holding executive decision authority over process, data, and compensation design.

**Timeline:** Designed and implemented from **January 2020**; operational results measured through **August 2022** (~2 years of production data).

**Core project team:**

- COO / RevOps Lead (myself)
- Technology / System configuration partner
- Function leads across Marketing, Telesales, Sales, Training, and Accounting

**Executive context:** Unlike an early-career analyst project, this was executed at COO level — I owned the business logic end-to-end and led the cross-functional change effort directly.

---

## 2. Business Problem

Before the project, the company's revenue operations suffered from systemic issues documented directly in the operating records.

**Fragmented, uncontrolled process**

- Each branch ran a different customer-handling process; executive management had no reliable view of what was actually happening at each site.
- Branch managers preferred to self-direct, resisted oversight, and tended to externalize blame ("customer too difficult," "not enough classrooms," "telesales calls weren't good enough") when deals didn't close.
- Reporting differed by region — every location used its own template, making consolidated comparison impossible.

**A leaking, unmeasured funnel**

- The appointment-booking rate against total contacts was very low, around 12–13%.
- Telesales throughput was slow, with 10–20% of leads left unworked; low talk-time (0.6–0.9 hours/day); no defined process and no performance pressure.
- No visibility into stage-by-stage conversion (Marketing data → appointment → action → payment), so no way to locate where the funnel leaked.

**Wasted customer data**

- Sales reps had no consistent way to manage their personal pipelines; there was no systematic rotation or continuous nurturing of leads.
- Branches were "losing" qualified prospects — customers with genuine demand who were never scheduled into a class — without noticing.
- Large volumes of high-value leads converted poorly, wasting marketing spend.

**Compensation disconnected from operational data**

- Revenue depended heavily on individual reps; the source of each customer and who did which step of the process could not be traced, making fair incentive calculation impossible.
- There was no way to credit the work of *sourcing* customers (by Marketing or by proactively prospecting functions).

---

## 3. Business Objectives

- Standardize a single customer process applied consistently across all branches.
- Digitize the full Lead-to-Cash funnel on one system so every lead and transaction leaves a data trail.
- Make stage-by-stage conversion measurable as the basis for optimization.
- Enable revenue attribution down to the content creator, the ad operator, the telesales rep, and the closer.
- Eliminate data waste through automated rotation, recovery, and recycling of un-closed leads.
- Wire compensation directly to operational data, with fair allocation across each stage of contribution.
- Give leadership near-real-time, cross-functional BI reporting.

---

## 4. Stakeholders

**Direct owner / decision-maker:**

- COO (myself) — held authority over process design, data taxonomy, and compensation logic

**Cross-functional participants (design and rollout):**

- Marketing (lead generation, UTM/attribution inputs)
- Telesales (first-touch qualification, appointment booking)
- Sales / Branch teams (in-person closing)
- Training / Academic (trial teaching, assessment, retention/renewal selling)
- Accounting (receipts, revenue recognition, receivables)

**Technology partner:**

- System configuration (workflows, automations, cross-object sync, permissions)

**Report consumers:**

- Executive leadership, branch managers, and function leads

---

## 5. Decision-Making Authority

This section reflects the actual level of seniority and autonomy held during the project.

| Area | Decision Authority |
|---|---|
| Process & pipeline design | Full ownership — defined stages, transition rules, and cross-functional handoffs |
| Data model & taxonomy | Full ownership — defined objects, fields, lead-source classification, and scoring criteria |
| Lead routing & recycling logic | Full ownership — defined distribution rules and recovery cadence |
| Attribution & compensation | Full ownership — designed the revenue allocation and incentive mechanics |
| System implementation | Directed the technology partner; validated configuration against business logic |
| Change management | Led rollout directly, including handling branch-manager resistance |

---

## 6. Project Scope

### Domain 1 — Standardized Lead-to-Cash Pipeline

The entire customer journey was modeled as a single unified workflow on the **Deal** object, with transitions enforced by validators:

```
New Lead → Processing → Consulting → Appointment Set → Action →
Contract/Deposit → Awaiting Class Assignment → Closed
      ↘ On Hold / Return / Recall → Recycle
```

Each transition requires the corresponding fields to be complete before it can proceed (e.g., moving to "Consulting" requires the five lead-scoring criteria; moving to "Appointment Set" requires a created appointment; moving to "Contract" requires branch and class). This guarantees complete data at every funnel stage, independent of any individual's data-entry discipline.

A parallel **Appointment** workflow (Booked → Confirmed → Received → Interaction → Done, with Return / Reschedule / Cancel / No-show branches) synchronizes bidirectionally with the Deal and with the shared calendar.

### Domain 2 — Relational, Multi-Object Data Model

Instead of one flat customer table, the system uses linked objects that auto-generate one another through automation:

| Object | Role | Key relationships |
|---|---|---|
| Deal | A single sales opportunity / customer engagement | Generates Appointment; on close, generates Contact & Receipt |
| Appointment | A scheduled on-site visit | Two-way sync with Deal & calendar |
| Contact | Durable customer identity (company asset) | One Contact links many Deals and Receipts |
| Receipt | Payment record; where revenue and allocation are booked | Links Contact & Contract; generates Receivable |
| Customer / Class | Enrolled student | Created at the "Awaiting Class Assignment" stage |

**Key design decision:** revenue is not booked directly from the Deal but flows through the Contact. A single Contact can generate multiple renewal Deals over time, and every Receipt inherits a fixed "acquisition date" from the first Deal. This is the data foundation that makes time-based attribution possible (see Domain 6) — something a flat model cannot support.

### Domain 3 — Lead-Source Classification & Routing Engine

Every customer is tagged with an acquisition source at intake, across six scenarios:

| Scenario | Source | Routing |
|---|---|---|
| K1 | Marketing | Auto-created from Facebook Lead / web form → email → Deal; round-robin (weighted) to the telesales pool |
| K2 | Sales (proactive) | Prospector keeps the lead, runs both call and close, no distribution |
| K3 | Telesales (proactive) | Prospector runs the call; appointment stage still round-robins to a branch |
| K5 | Training (proactive) | Trainer runs trial + consult; may invite a closer and split 50% of revenue |
| K5/K6 | Service (renewal / proactive) | Prospector keeps the lead |

The routing engine auto-assigns owners by weighted round-robin and tracks each rep's "accepting / paused" status so leads are never assigned to an overloaded rep.

### Domain 4 — Revenue Attribution Model

Every marketing Deal carries a standardized UTM set (source, medium, campaign, term, content) plus two custom fields to separate **who created the content** from **who ran the ad**. These fields are locked from editing after creation to preserve attribution integrity.

The result: every unit of revenue is traceable to a campaign, content, ad operator, telesales rep, closer, and branch — the foundation for both marketing-effectiveness reporting and compensation.

### Domain 5 — Lead Scoring & Lifecycle Management

Lead potential is scored on **five ABCD criteria** (Need, Product/Method fit, Financial capacity, Location, Timing), each weighted, with an aggregate potential score computed automatically. Any criterion at C/D level maps to a specific "objection reason," linked to a bank of standard objection-handling playbooks.

In parallel, each customer carries a dynamic temperature state:

- **Hot (A)** — highest priority, daily contact; drops to B if no appointment within ~7 days; a marketing lead with no logged interaction is **auto-recalled and reassigned**.
- **Warm (B)** — standard cadence, minimum twice weekly; drops to C after ~2 weeks without progress.
- **Cold (C/D)** — scheduled for follow-up at an expected return date.

This turns "hundreds-to-thousands of records in the system" into a clearly prioritized daily worklist for each rep.

### Domain 6 — Data Recycling Engine

The answer to the data-waste problem:

- Leads left un-worked or un-closed beyond ~3 months are **auto-recalled** into a shared pool.
- Aged data is packaged into blocks (~100 leads) and **rotated weekly** to telesales/sales, recovered on fixed cadences, then redistributed.
- Leads are gradually re-warmed (D→C→B→A) through re-engagement programs before being re-assigned.
- Critically, the **original acquirer still receives revenue credit** even if someone else later closes the lead — so recycling does not kill the incentive to prospect.

### Domain 7 — Sales Compensation Engine

The system separates **two kinds of revenue**:

**(a) Realized revenue** — actual cash collected, allocated by stage:

| Stage | Weight |
|---|---|
| Telesales (initial consult + booking) | 50% |
| Sales (meet & close) | 50% |

This is multiplied by a tiered bonus rate based on quota attainment (4 salary tiers against 4 quota levels), then by a **quality-adjustment coefficient** — each contract/class is graded ABCD on class size & revenue (group classes) or price & payment terms (private/enterprise).

**(b) Acquisition revenue** — a separately booked credit for *sourcing* the customer:

- Allocated to content creator, ad operator, manager (marketing leads) or the direct prospector (proactive leads).
- **Time-decay attribution:** revenue within 0–3 months of acquisition counts 100%, 3–6 months 60%, 6–12 months 30%, beyond 12 months 0%. Thanks to the durable Contact model, renewal sales still credit the original acquirer.
- Applied at a fixed base rate (2.5% for reps, 1% for managers), with no quota, to avoid distorting the realized-revenue bonus structure.
- **Manager override** on the acquisition revenue of their direct reports.

**Total income = base salary + quality-adjusted realized-revenue bonus + acquisition bonus.** One consistent framework applies across Sales, Telesales, and Marketing.

### Domain 8 — Standardized Reporting & Cross-Functional BI

A shared reporting framework (situation & direction → key issue areas → overall assessment of strengths/weaknesses/solutions), built on the now-clean, connected data, powering a cross-functional BI dashboard suite (see Business Impact).

---

## 7. Scale

**Functional coverage:** Marketing, Telesales, Sales, Training/Academic, Accounting — the full revenue chain.

**Geographic coverage:** multiple branches across Hanoi (Cau Giay, Hoang Cau, and others) and Ho Chi Minh City (Phu Nhuan and others), plus functional centers (Marketing, Telesales, Service).

**System footprint:** a cluster of linked projects (Customer CRM, Appointment Management, Receipts, Revenue Management, Classes & Teachers), regionally partitioned, connected via cross-object links and automations.

**Operational data:** the system captured tens of thousands of Deals and thousands of closed transactions over multiple years, with reporting data spanning 2016–2022.

---

## 8. My Responsibilities

**Business Analysis & Process Design**

- Diagnosed the current state and mapped funnel leakage across every function
- Designed the end-to-end Lead-to-Cash pipeline, stage definitions, and transition rules

**Data Architecture**

- Designed the relational multi-object data model and field taxonomy
- Defined lead-source classification and the ABCD lead-scoring framework

**RevOps Systems**

- Directed configuration of workflows, automations, screens, permissions, cross-object sync, and JQL logic
- Designed the routing engine and the data-recycling cadence

**Attribution & Compensation**

- Built the UTM-based, per-stage attribution model with time-decay
- Designed the full compensation engine (multi-touch allocation, quality adjustment, manager override, acquisition bonus)

**Analytics & Reporting**

- Standardized the reporting framework and specified the cross-functional BI dashboards

**Leadership & Change Management**

- Led cross-functional rollout, managed change resistance, piloted at one branch before scaling

---

## 9. Governance & Operational Continuity

- **Enforced data integrity at the source** — validators require complete fields at each stage transition; attribution fields are locked from editing.
- **Single source of truth** — one connected data model replaced disparate branch spreadsheets, so leadership reviewed the same numbers everyone operated on.
- **Stable, repeatable operation** — because the process, routing, and reporting are embedded in the system rather than in individuals, day-to-day operation became stable and less dependent on specific people, and continued running consistently across branches.
- **Recruitment & onboarding support** — the standardized process and clearly defined roles/steps made it materially easier to hire and train new sales and telesales staff: new joiners follow a defined workflow and a fixed data standard rather than an undocumented, manager-specific way of working.

---

## 10. Process & Technical Workflow

```
Current-State Diagnosis
        ↓
  Process Standardization (Lead-to-Cash)
        ↓
  Data Model & Field Taxonomy
        ↓
  System Configuration
  (workflows · automations · routing · sync · permissions)
        ↓
  Attribution & Compensation Logic
        ↓
  Pilot at One Branch
        ↓
  Cross-Branch Rollout
        ↓
  Cross-Functional BI Reporting
        ↓
  Continuous Funnel Optimization
```

---

## 11. Major Challenges & Solutions

**1. Change resistance from branch managers.** Managers wanted autonomy and disliked oversight. → Renamed roles to match actual scope (focus on selling + report supervision), added periodic staffing reports, piloted at one branch first, and handled the psychological side of change alongside the process change.

**2. Uneven data-entry discipline.** Left to goodwill, data would be incomplete and the funnel unmeasurable. → Used validators to force complete fields at each transition; locked attribution fields; auto-populated derivable fields.

**3. Fair compensation when a customer passes through many hands over time.** → Built the durable-Contact data model plus time-decay attribution, so every sale on a customer traces back to the correct original acquirer and the correct owner of each stage.

**4. Wasted historical data.** → An automated recall-and-recycle engine on a fixed cadence, with lead re-warming before redistribution.

---

## 12. Business Impact

Running on the standardized process, the funnel became both **visible and improvable** — and the measured improvements are documented in the cross-functional BI reporting through August 2022.

**Funnel conversion improvements**

- **Appointment-booking rate** rose to roughly **20%**, reaching close to **30%** in strong periods — up from a starting point around 12–13%.
- **Close rate** (deals closed / actioned) improved from about **36% to roughly 50%**, reaching approximately **67%** at peak.
- These conversion gains translated into **higher revenue and profit**, as more of the same top-of-funnel volume converted through each stage.

**Full-funnel and cross-functional visibility (previously impossible)**

- Deal Created → Actioned → Closed → Revenue, with conversion rates at every tier, sliced by region, product line, and individual rep.
- Marketing effectiveness and unit economics: revenue and close rate by campaign source/content/term, with **cost-per-deal** and **cost-to-revenue** ratios per channel.
- Detailed sales performance (revenue per action, revenue per close) by region and by rep — the basis for the compensation engine.
- Receivables and cash-flow tracking by owner; goal-vs-actual dashboards by function.

**Operational stability and organizational leverage**

- Because process, routing, and reporting live in the system rather than in individuals, operations became **stable and repeatable** across branches and less exposed to individual turnover.
- The standardized workflow and clear role definitions **materially supported recruitment and training** — new sales/telesales hires ramp on a defined process and data standard instead of an undocumented, manager-specific way of working.
- Data waste was replaced by a recycling engine that turns aged leads into repeatable revenue rather than letting them die in personal pipelines.

> *Interpretation note:* The BI reporting evidences the **measurement and optimization capability the standardized data layer created** — i.e., a funnel that was previously invisible became fully visible and tunable. The conversion improvements are best presented as outcomes the system enabled the business to measure and drive, framed alongside the broader business context rather than as attributable to the system alone.

---

## 13. Handover & Continuity

- The standardized process, data model, and reporting were embedded in the system and used as the operating backbone across branches.
- Documentation of the process, field taxonomy, and compensation logic supported onboarding of new sales, telesales, and support staff.
- The clearly defined roles and workflow reduced dependency on specific individuals and eased ongoing recruitment and training.

---

## 14. Lessons Learned

1. **A process only improves what it can measure** — enforcing complete data at each stage is what turned an invisible funnel into a tunable one.
2. **Put integrity at the source, not in goodwill** — validators and locked fields beat after-the-fact data cleaning.
3. **The data model is the real product** — a durable Contact model (not a flat table) is what made time-based attribution and fair compensation possible.
4. **Incentives are part of the system** — wiring compensation to operational data is what made the process stick behaviorally, not just structurally.
5. **Standardization is an organizational asset** — it stabilizes operations and de-risks hiring and training, beyond its direct conversion impact.
6. **Change management is half the work** — the hardest part was not the configuration but aligning branch managers and shifting the operating culture.

---

## 15. Skills Demonstrated

**Revenue Operations**

- Lead-to-Cash pipeline design
- Cross-functional revenue process standardization (Marketing–Sales–Service–Finance)
- Lead routing, lead scoring, lifecycle management, data recycling

**CRM / Systems Administration**

- Configuring an issue-based platform as a CRM: workflows, custom fields, screens, permissions, automation, cross-object sync, JQL
- Relational multi-object data modeling

**Revenue Attribution & Analytics**

- UTM-based and per-stage attribution, time-decay attribution
- Funnel conversion measurement, CAC / cost-to-revenue analysis
- Cross-functional BI dashboard design (Power BI background from the Golden Gate project)

**Sales Compensation Design**

- Data-driven incentive design: multi-touch allocation, quality adjustment, manager override, acquisition bonus

**Cross-Functional Leadership**

- Executive-level ownership of a cross-functional transformation
- Change management, pilot-then-scale rollout, hiring and training enablement
