---
name: ui-testing
description: >
  Compares a live web application page against a Figma design to detect UI inconsistencies.
  User provides a Figma design link (with node ID) and the skill fetches the design via Figma MCP,
  captures the live app page via Browser Tools MCP, then produces a structured mismatch report in chat.
  Handles dynamic content (amounts, names, dates, IDs) by checking presence/structure only, not values.
  Trigger when the user says "ui test", "compare with figma", "check design", "test this page against figma",
  "run ui check", or provides a Figma link and asks to compare.
compatibility: >
  Requires Figma MCP (with FIGMA_ACCESS_TOKEN) and Browser Tools MCP (@agentdeskai/browser-tools-mcp)
  connected. User must already be logged into the application in their browser before running.
---

# UI Testing Skill

Automates visual and structural comparison between a Figma design node and the live application page currently open in the browser.

---

## Prerequisites Check

Before starting, verify:
1. Browser Tools MCP is connected (tool `browser_action` or `take_screenshot` is available)
2. Figma MCP is connected (tool `get_figma_data` or `figma_get_file_nodes` is available)
3. User has provided a Figma URL containing a `node-id` parameter

If any prerequisite is missing, stop and tell the user what to fix. Do not proceed partially.

---

## Input

The user provides a Figma URL in one of these formats:
- `https://www.figma.com/design/<FILE_KEY>/...?node-id=<NODE_ID>`
- `https://www.figma.com/file/<FILE_KEY>/...?node-id=<NODE_ID>`

Parse from the URL:
- `FILE_KEY` — the alphanumeric segment after `/design/` or `/file/`
- `NODE_ID` — value of the `node-id` query parameter (may use `-` or `:` as separator, normalize to `:`)

If no Figma URL is provided, ask the user: "Please provide the Figma design link with the node ID for the page you want to compare."

---

## Workflow

Execute all steps in order. Do not pause between steps unless an error occurs.

---

### Step 1 — Fetch Figma Design Data

Use Figma MCP to fetch the node. Call `get_figma_data` (or equivalent) with:
- `fileKey`: extracted FILE_KEY
- `nodeId`: extracted NODE_ID

From the response, extract and store:

**A. Text Inventory** — all visible text strings in the design:
- Collect every text layer recursively within the node
- Tag each with its purpose using these rules:

| Tag | Rule |
|---|---|
| `STATIC` | Short labels, button text, nav items, column headers, section titles, form field labels — text that never changes per user or data |
| `DYNAMIC` | Amounts with currency symbols, dates, user names/emails, IDs, counts, percentages, status values pulled from data |
| `PATTERN` | Greeting text like "Welcome, [Name]" or "Hello, [Name]" — verify prefix only |

Classification heuristics:
- Contains `$`, `€`, `₹`, `%` → `DYNAMIC`
- Matches date pattern (`DD/MM/YYYY`, `MMM DD`, `YYYY-MM-DD`) → `DYNAMIC`
- All-caps short string (`ACTIVE`, `PENDING`, `PAID`) → `DYNAMIC` (status badge)
- Long alphanumeric (length > 8, mixed chars) → `DYNAMIC` (ID/reference)
- Contains `@` → `DYNAMIC` (email)
- 1–4 word label ending with `:` or followed by an input → `STATIC`
- Nav/menu item, button, heading with known app-domain words → `STATIC`

**B. Layout Inventory** — structural elements:
- Top navigation / sidebar present?
- Page title / heading
- Cards, tables, grids — note count and arrangement
- Buttons and their labels
- Form fields and their labels
- Empty states or placeholder text

**C. Color & Typography (surface level)**:
- Primary heading font size (approximate)
- Button colors (note if primary/secondary/danger)
- Any obvious color-coded status indicators

---

### Step 2 — Capture Live Application State

Use Browser Tools MCP to:

1. **Take a screenshot** of the current browser page
   - Call `take_screenshot` or `browser_action` with action `screenshot`
   - Store for visual reference

2. **Read the DOM** — extract visible text and structure:
   - Call `get_page_content` or `execute_script` to get:
     - All visible text nodes (exclude hidden elements, `display:none`, `visibility:hidden`)
     - Page `<title>`
     - All `<h1>`, `<h2>`, `<h3>` text content
     - All `<button>`, `<a>` (nav links) text content
     - All `<label>`, `<th>`, `<td>` text content
     - All `<input>` placeholder values
     - All status badge / chip text (elements with class names containing `badge`, `chip`, `status`, `tag`)

3. **Extract the current URL** — note the page path for context in the report.

If Browser Tools MCP cannot take a screenshot or read DOM, stop and tell the user:
> "Browser Tools MCP is not responding. Make sure the Chrome extension is active and the BrowserTools server is running."

---

### Step 3 — Intelligent Comparison

Compare Figma design data (Step 1) against live app data (Step 2).

#### 3A. Static Text Comparison
For every `STATIC` tagged text from Figma:
- Search for an exact or case-insensitive match in the live DOM text list
- If NOT found → flag as **MISSING**
- If found but different casing → flag as **CASE MISMATCH**
- If found but truncated (e.g., design shows "Transaction History", app shows "Transactions") → flag as **TEXT MISMATCH**

#### 3B. Dynamic Content Verification
For every `DYNAMIC` tagged element from Figma:
- Do NOT compare the actual value
- Check only that a non-empty value EXISTS in the corresponding position/field in the live app
- If the field is empty or missing → flag as **EMPTY DYNAMIC FIELD**
- If the field exists with a value → mark as **PASS (dynamic)**

#### 3C. Pattern Text Verification
For every `PATTERN` tagged text:
- Extract the static prefix (e.g., "Welcome, " from "Welcome, John")
- Check that the live app shows text starting with that prefix
- If prefix matches → **PASS (pattern)**
- If prefix missing → flag as **PATTERN MISMATCH**

#### 3D. Layout Structure Comparison
Compare structural presence:
- Navigation present in design → check if nav/sidebar exists in live DOM
- Tables in design → check if `<table>` or grid structure exists in live DOM
- Button count on a section → compare approximate count (±1 tolerance)
- Cards/panels → check if similar container blocks exist

Flag as **LAYOUT MISSING** if a major structural element from the design is absent.

#### 3E. Extra Elements (Bonus Check)
Identify text/elements visible in the live app that are **NOT** in the Figma design:
- Flag these as **NOT IN DESIGN** — could be unimplemented additions or accidental content

---

### Step 4 — Generate Report

Output the comparison report in chat using this format:

---

```
╔══════════════════════════════════════════════════════════╗
║           UI COMPARISON REPORT                          ║
║  Page   : [current URL path]                            ║
║  Figma  : [node-id from URL]                            ║
║  Tested : [timestamp]                                   ║
╚══════════════════════════════════════════════════════════╝

SUMMARY
───────
  Total checks  : XX
  ✅ Passed      : XX
  ❌ Failed      : XX
  ⚠️  Warnings   : XX

────────────────────────────────────────────────────────────
FAILURES  (action required)
────────────────────────────────────────────────────────────

[F1] TEXT MISMATCH
  Element  : Page heading / Section title / Button label
  Figma    : "exact text from design"
  Live App : "text found in DOM"
  Severity : High / Medium / Low

[F2] MISSING ELEMENT
  Element  : [describe what's missing]
  Figma    : Present
  Live App : Not found in DOM
  Severity : High

[F3] EMPTY DYNAMIC FIELD
  Element  : [field name/position]
  Expected : Non-empty value
  Live App : Empty or absent
  Severity : Medium

[F4] LAYOUT MISSING
  Element  : [nav / table / card section etc.]
  Figma    : Present
  Live App : Not detected in DOM
  Severity : High

────────────────────────────────────────────────────────────
WARNINGS  (review recommended)
────────────────────────────────────────────────────────────

[W1] CASE MISMATCH
  Element  : [element description]
  Figma    : "Text In Title Case"
  Live App : "text in lowercase"

[W2] NOT IN DESIGN
  Element  : [unexpected element description]
  Live App : "[text or element found]"
  Note     : Present in app but not in Figma — verify intentional

────────────────────────────────────────────────────────────
PASSED (dynamic fields skipped — structure verified only)
────────────────────────────────────────────────────────────
  ✅ [field name] — value present
  ✅ [field name] — value present
  ...

────────────────────────────────────────────────────────────
NOTES
────────────────────────────────────────────────────────────
- Dynamic fields (amounts, dates, names, IDs) were verified
  for presence only, not value accuracy.
- Screenshot captured and available for visual review.
- To raise these as Jira bugs, provide your Jira project key.
```

---

If there are zero failures and zero warnings:
> "✅ All checks passed. The live app matches the Figma design for node [NODE_ID]. No mismatches found."

---

## Severity Guide

| Severity | Meaning |
|---|---|
| **High** | User-facing text wrong or missing element — breaks UX or trust |
| **Medium** | Field empty or structural gap — may indicate data issue |
| **Low** | Minor label difference, casing, spacing text |

---

## Error Handling

| Situation | Action |
|---|---|
| Figma MCP unavailable | Stop. Tell user to connect Figma MCP with a valid access token. |
| Invalid Figma URL (no node-id) | Ask user to share the URL with node-id parameter included (Right-click frame in Figma → Copy link). |
| Figma node returns empty | Tell user the node ID may point to an empty frame or deleted component. |
| Browser Tools MCP unavailable | Stop. Tell user to start the BrowserTools server and ensure Chrome extension is active. |
| Page requires login (redirected to login) | Stop. Tell user to log in to the app first, then re-run. |
| DOM read returns very little text | Warn user the page may still be loading. Ask them to confirm the page is fully loaded, then re-run. |

---

## Notes

- Never compare dynamic values (amounts, user names, dates, IDs) — only verify they exist.
- Always take a fresh screenshot on each run — do not reuse from prior runs.
- The skill does not modify the application or Figma file in any way.
- Jira bug creation is planned for a future enhancement — the report structure above is intentionally formatted to map 1:1 to a Jira bug card.
