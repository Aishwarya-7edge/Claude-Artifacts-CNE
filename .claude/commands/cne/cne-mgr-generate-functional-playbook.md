# Cloud Native Engineering I - Aishwarya

# cne-mgr-generate-functional-playbook.md
Generate or update the CNE Functional Playbook for: $ARGUMENTS

---

## WHAT THIS COMMAND DOES

| Mode | When to Use | What It Produces |
|---|---|---|
| **First Run** | No prior version exists | Full A–Z playbook from scratch |
| **Quarterly Update** | End of every quarter | Only changed sections + changelog entry |
| **Section Fix** | A specific section needs an urgent fix mid-quarter | Targeted section rewrite only |

---

## HOW TO USE

### Mode 1 — First Run
```
/cne-mgr-generate-functional-playbook
  mode=first-run
  version=1.0
  quarter=Q1-2026
  owner=Aishwarya K
```

### Mode 2 — Quarterly Update
```
/cne-mgr-generate-functional-playbook
  mode=quarterly-update
  version=2.0
  quarter=Q2-2026

  OBSERVATIONS:
  - Timesheet corrections keep happening because engineers confuse
    opex vs internal cross-charge accounts
  - KS sessions have no consistent structure — some are 10 min,
    some are 90 min with no agenda
  - New joiners still taking 3+ weeks to raise first PR

  SECTIONS TO ADD:
  - Add a "Common Timesheet Mistakes" table to Part G
  - Add KS session prep checklist to Part H

  SECTIONS TO UPDATE:
  - Part G: Cross-charging rules — add clearer examples
  - Part H: KS structure — enforce 5-stage format
  - Part B: Onboarding — add buddy check-in schedule
```

### Mode 3 — Section Fix
```
/cne-mgr-generate-functional-playbook
  mode=section-fix
  section=G3
  reason=Cross-charging rules are causing repeated Tempo errors
  fix=Add a table with 5 real examples of what gets logged where
```

---

## STEP 1 — Parse Inputs

Extract from $ARGUMENTS:

| Field | Required | Description |
|---|---|---|
| `mode` | Yes | `first-run` / `quarterly-update` / `section-fix` |
| `version` | Yes | Playbook version number (e.g. 1.0, 2.0) |
| `quarter` | Yes | Quarter being published (e.g. Q2-2026) |
| `owner` | First run only | Manager name |
| `observations` | Quarterly update | What broke, what was unclear, what changed |
| `sections_to_add` | Optional | New sections to create |
| `sections_to_update` | Optional | Existing sections to revise |
| `section` | Section fix only | Which part/section (e.g. G3, H2) |
| `reason` | Section fix only | Why this needs fixing |
| `fix` | Section fix only | What the fix should be |

**If `mode` is missing**, ask:
> "Is this a first run, a quarterly update, or a targeted section fix?
> For a quarterly update, please share your observations from the quarter."

**If `mode=quarterly-update` but no observations provided**, ask:
> "What happened this quarter that needs to change in the playbook?
> Even 2–3 bullet points of what caused confusion or errors is enough."

---

## STEP 2 — FIRST RUN: Generate Full Playbook

When `mode=first-run`, generate the complete CNE Functional Playbook
using the structure below. Every section must be fully written —
no placeholders, no "fill this in later."

Output the playbook with this exact header:

```
╔══════════════════════════════════════════════════════════════╗
  CNE FUNCTIONAL PLAYBOOK
  Cloud Native Engineering — 7EDGE
  Version: <X.0>    Quarter: <QX-YYYY>
  Owner: <name>, Manager I – Cloud Native Engineering
  Last Updated: <today's date>
  Next Review:  End of <next quarter>
╚══════════════════════════════════════════════════════════════╝

"This is your A–Z reference for how CNE operates.
 If it is not in here, raise it so we can add it."
```

Then generate all 14 parts in full:

### PLAYBOOK STRUCTURE

**Part A — Who We Are**
- A1. Department identity & mission
- A2. Service lines (Modern App Dev / App Improvement / AMS / Modernization)
- A3. Team structure & roles (CNE I / II / Senior / Manager)
- A4. Culture code (7 non-negotiable values, stated as behaviours not aspirations)
- A5. Tech stack reference table

**Part B — Joining CNE**
- B1. Onboarding checklist (Day 1 / Week 1 / Month 1)
- B2. Ramp-up guide by role level
- B3. Buddy system (responsibilities, duration, escalation)
- B4. Tools & access setup table (tool, purpose, who sets it up)
- B5. Response time expectations

**Part C — How We Work Day to Day**
- C1. Daily rituals (Kanban standup — format, rules, Tempo logging)
- C2. Weekly rituals (1:1, team meeting, KS session)
- C3. Sprint / flow ceremonies (refinement, replenishment, retro, kick-off)
- C4. Communication standards (async-first, SLAs per channel)
- C5. Meeting etiquette
- C6. Jira board standards (WIP, WIP Age, update frequency)

**Part D — Delivery Standards**
- D1. Definition of Ready (checklist)
- D2. Definition of Done (checklist)
- D3. Git workflow (branch naming, commit format, sync rules)
- D4. Architecture & technical decision standards
- D5. Deployment checklist
- D6. Sev1 incident resolution process and SLAs

**Part E — Testing Standards**
- E1. Test-first development standard (why + rule)
- E2. TDD — Red / Green / Refactor cycle
- E3. BDD — Gherkin format, when to use, who writes it
- E4. Layered testing strategy table (Unit → Integration → API → Component → Smoke → QA → UI)
- E5. Bug fixing workflow (step by step)

**Part F — Code Review & Quality**
- F1. Raising a PR (checklist + rules)
- F2. Reviewing a PR (SLA, standards, approval criteria)
- F3. Coding standards table (naming, comments, logging, secrets, dead code)
- F4. Third-party API standards (sandbox, production, monitoring, fallback)

**Part G — Timesheets & Reporting**
- G1. Logging rules (deadline, tool, card-level logging)
- G2. Account types explained (Billable / Opex / Innovation / PDP / Internal)
- G3. Cross-charging rules (3 scenarios with examples)
- G4. Common timesheet mistakes (table: wrong practice → correct practice)
- G5. Timesheet audit process
- G6. Manager weekly distribution format

**Part H — Knowledge Sharing**
- H1. KS session standard (frequency, duration, owner rotation)
- H2. Mandatory 5-stage session structure (with time allocations)
- H3. How to prepare (checklist, approval, dry run)
- H4. Post-session actions (Slack summary, Jira card, manager feedback SLA)
- H5. Mettle & skills tracker process

**Part I — People & Performance**
- I1. 1:1 meeting standard (frequency, structure, ownership)
- I2. PDP process (format, review cadence)
- I3. Calibration cycle (inputs, ratings, cadence)
- I4. PBP — when triggered, format, duration
- I5. Feedback culture & norms
- I6. KPIs — full table with targets

**Part J — Hiring & Onboarding New Roles**
- J1. Hiring brief process
- J2. Screening & interview standard (3 rounds with focus areas)
- J3. Pre-joining checklist

**Part K — Innovation & Growth**
- K1. Innovation time (what counts, what doesn't, approval + logging rules)
- K2. PDP learning (approved activities, logging, required outcomes)
- K3. Spike & POC management (7-step process)
- K4. Supported certifications

**Part L — Inter-Departmental Collaboration**
- L1. How CNE works with each team (table: team, how we collaborate)
- L2. Dependency management (when to raise, how to track)
- L3. AWS support & escalation process

**Part M — Non-Negotiables**
- M1. Behaviours that are never acceptable (list)
- M2. Escalation path (table by situation)
- M3. Confidentiality & IP standards (includes AI tool usage rules)

**Part N — Quarterly Playbook Review**
- N1. Why we review quarterly
- N2. How to raise a change (format for submissions)
- N3. What gets reviewed each quarter
- N4. Changelog table (version, quarter, summary, owner)

---

## STEP 3 — QUARTERLY UPDATE: Generate Changed Sections Only

When `mode=quarterly-update`:

**3a. Map each observation to the section it affects:**

| Observation Type | Maps To |
|---|---|
| Timesheet errors / confusion | Part G |
| KS session quality issues | Part H |
| Onboarding / ramp-up gaps | Part B |
| PR / code quality problems | Part F |
| Delivery / sprint issues | Part C or D |
| Testing gaps | Part E |
| People / feedback concerns | Part I |
| Cross-team friction | Part L |
| Tool or access issues | Part B4 |
| Behaviour / culture gaps | Part M or A4 |

**3b. For each affected section:**
- Rewrite only the affected subsection
- Keep all other content identical
- Label each changed subsection clearly:
  `[UPDATED — Q2-2026]` at the top of changed sections
  `[NEW — Q2-2026]` at the top of new sections

**3c. Output a "What Changed" header before the sections:**

```
═══════════════════════════════════════════════════
 QUARTERLY UPDATE — <Quarter>
 Playbook version: <X.0> → <X+1.0>
═══════════════════════════════════════════════════

SECTIONS UPDATED:
→ <Part X.X>: <one-line reason>
→ <Part X.X>: <one-line reason>

SECTIONS ADDED:
→ <Part X.X>: <one-line reason>

WHAT TRIGGERED THESE CHANGES:
<1–2 sentences summarising the pattern of issues observed this quarter>

Next review: End of <next quarter>
═══════════════════════════════════════════════════
```

**3d. Append to the changelog in Part N4:**
```
| <version> | <quarter> | <summary of changes> | <owner> |
```

---

## STEP 4 — SECTION FIX: Rewrite One Section

When `mode=section-fix`:

1. Identify the section from the `section` parameter (e.g. G3, H2)
2. Read the `reason` — understand the real-world problem it caused
3. Apply the `fix` — rewrite only that subsection
4. Output the rewritten subsection with label: `[FIXED — <today's date>]`
5. Add a note: "This fix will be incorporated into the full changelog at end-of-quarter review."

---

## STEP 5 — QUALITY RULES (Apply to All Modes)

Every section generated must follow these rules:

**Tone & Voice:**
- Direct, practical, and specific — not aspirational or vague
- Written for an engineer who is new to CNE, not for leadership
- "You" not "team members" — address the reader directly
- No filler phrases like "it is important to", "please ensure", "we strive to"

**Format Rules:**
- Use tables for: comparisons, tool lists, role summaries, KPIs
- Use checklists for: DoR, DoD, onboarding, PR raising, deployment
- Use numbered steps for: processes with a fixed sequence
- Use bullet points for: culture values, norms, rules without sequence
- No section should be a wall of prose — break it up

**Content Rules:**
- Every checklist item must be actionable — not a vague reminder
- Every rule must have a "why" or an example where it's not obvious
- Every table must have meaningful column headers
- KPIs must always show the target value — never leave it blank
- Culture values must state the behaviour, not just the virtue
  - Wrong: "We value transparency"
  - Right: "Raise blockers within the same working day they appear"

**Length:**
- Each part: as long as it needs to be — no padding, no cutting for brevity
- Each subsection: minimum 3 lines, maximum fits on one screen without scrolling

---

## STEP 6 — POST-GENERATION REMINDER

After generating, always append:

```
════════════════════════════════════════════════════
 BEFORE YOU PUBLISH — REVIEW CHECKLIST
════════════════════════════════════════════════════
[ ] All KPI target values are filled in (no blanks)
[ ] Tool links updated — replace [Google Drive link] with actual URLs
[ ] Certification cost coverage confirmed with People Ops
[ ] Cross-charging examples match current Tempo account structure
[ ] Manager name, version number, and quarter are correct
[ ] Share draft with one CNE engineer to sense-check before publishing
[ ] Announce the update to the team in Slack with a summary of changes
════════════════════════════════════════════════════
```