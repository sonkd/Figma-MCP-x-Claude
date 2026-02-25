---
name: figma-agent-skill
version: 0.1.0
description: >
  Audit, refactor, and document a Figma Design System file for mobile banking or fintech apps.
  Use this skill whenever the user wants to: audit a Figma design system, restructure Figma pages,
  check component quality, review naming conventions, inspect design tokens or variables, create
  documentation frames in Figma, refactor components for scalability, or improve consistency across
  a Figma file. Trigger even if the user says "review my Figma file", "clean up our design system",
  "check if our DS is well structured", "audit our components", or "help me organize Figma".
  Also trigger for tasks like "are our tokens correct?", "does our DS follow best practices?",
  "set up a governance page", or any mention of Figma + design system quality/structure.
metadata:
  author: sonkd
  domain: Mobile Banking / Fintech
  requires: Figma MCP (read + write), Figma Desktop Bridge plugin recommended for live validation
---

## 1. PURPOSE

Enable Claude to act as a senior Design Systems engineer who can:
- Audit an existing Figma Design System for structure, consistency, and completeness
- Refactor and reorganize it into a scalable, maintainable DS following industry standards
- Add documentation frames and annotations directly in Figma
- Generate a structured audit report (in-Figma + Markdown export)

Primary domain: **Mobile Banking / Financial Application** (eKYC, transfers, cards, lending).

---

## 2. TOOL USAGE MAP

Use these Figma MCP tools in sequence. Do not guess — always fetch real data first.

| Phase | Primary Tools |
|---|---|
| Discovery | `figma_get_metadata`, `figma_get_file_data` (depth=1) |
| Token audit | `figma_get_variables`, `figma_get_styles` |
| Component audit | `figma_get_design_system_summary`, `figma_search_components`, `figma_get_component` |
| Visual validation | `figma_capture_screenshot`, `figma_get_component_image` |
| Write / refactor | `figma_execute`, `figma_set_fills`, `figma_set_text`, `figma_rename_node` |
| Documentation | `figma_set_description`, `figma_create_child`, `figma_post_comment` |
| Final check | `figma_audit_design_system` (if available) |

**Rule**: Never call a write tool before completing Phase 1 discovery. Always confirm the target node ID before modifying.

---

## 3. SCOPE & CONSTRAINTS

**In scope**: Structure audit, naming conventions, token/variable review, component state coverage, documentation frames, governance page setup, Archive migration.

**Out of scope**: Visual redesign, brand repositioning, business logic changes, code-side implementation.

**Assumption defaults** (override if user specifies):
- Spacing system: 4pt base grid
- Type scale: Material-style (12/14/16/20/24/32)
- Target platforms: iOS + Android (mobile-first)
- Accessibility target: WCAG AA (4.5:1 for text, 3:1 for UI elements)

---

## 4. AGENT WORKFLOW

### Phase 1 — DISCOVER (always run first)

```
1. figma_get_file_data(depth=1, verbosity="summary")
   → Map all pages: names, node counts, structure
2. figma_get_variables(format="summary")
   → Inventory token collections and modes (Light/Dark)
3. figma_get_styles(verbosity="summary")
   → Inventory color, text, effect styles
4. figma_get_design_system_summary()
   → Component categories and counts
```

Output: Internal structural map. Do not write to Figma yet. Summarize findings to user before proceeding.

---

### Phase 2 — AUDIT

Run the checklist in Section 5. For each area, use:
```
figma_search_components(query="Button")  → find component
figma_get_component(nodeId="...", format="metadata")  → get full detail
figma_get_component_image(nodeId="...")  → visual reference
```

Flag issues as: `[CRITICAL]` (breaking), `[WARN]` (inconsistent), `[INFO]` (suggestion).

---

### Phase 3 — PLAN & CONFIRM

Present the refactor plan to the user before executing:
- Which pages will be created / renamed
- Which components will be moved / archived
- Which tokens need renaming

Wait for explicit user confirmation before Phase 4.

---

### Phase 4 — EXECUTE (write operations)

```
# Create Archive page first (safety net)
figma_execute(code="
  const archivePage = figma.createPage();
  archivePage.name = '🗂 Archive';
")

# Clone deprecated components to Archive before moving
figma_clone_node(nodeId="<deprecated_component_id>")
figma_move_node(nodeId="<clone_id>", ...)

# Rename pages to standard structure
figma_execute(code="
  figma.root.children.forEach(page => {
    if (page.name === 'Colors') page.name = '02 – Foundations';
  });
")

# Add documentation frame
figma_create_child(nodeType="FRAME", parentId="...", properties={
  name: "_doc / Color Usage",
  width: 800, height: 600
})
figma_set_text(nodeId="...", text="Usage: Semantic colors only in UI components...")
```

**Safety rules (non-negotiable)**:
- Always clone to Archive before deleting or restructuring
- Never overwrite a component without creating a backup clone first
- Preserve component IDs — use `rename_node` not delete+recreate for renames
- Log every write action in the audit trail (see Section 6)

---

### Phase 5 — VERIFY

```
figma_capture_screenshot(nodeId="<refactored_page>")
figma_audit_design_system()  # if tool available
```

Run through the Post-Refactor Checklist (Section 5.5). Report pass/fail per item.

---

## 5. AUDIT CHECKLIST

### 5.1 Design Language
- [ ] Design philosophy stated (clarity, trust, safety — critical for banking UX)
- [ ] Principles tied to use cases (e.g., "Error states must never be ambiguous — user must know what to do next")
- [ ] Dos & Don'ts documented with banking-specific examples

**Target page**: `01 – Design Language` with frames: Vision · Principles · UX Heuristics · Dos & Don'ts

---

### 5.2 Foundations

**Colors**
- [ ] Semantic tokens defined: `color/semantic/primary`, `color/semantic/error`, `color/semantic/success`, `color/semantic/warning`, `color/semantic/info`
- [ ] Primitive tokens separated from semantic tokens
- [ ] Contrast ratios documented (use `figma_get_token_values` to extract hex, verify manually)
- [ ] Dark mode: separate mode in variable collection OR documented strategy

**Typography**
- [ ] Type scale covers: Display · Heading · Body · Label · Caption · Overline
- [ ] Line height, letter spacing, and weight documented per style
- [ ] Numeric/tabular figures variant for amount display (critical for banking)

**Spacing & Layout**
- [ ] 4pt or 8pt grid system declared
- [ ] Mobile safe areas and notch handling noted
- [ ] Bottom sheet, modal, and toast inset rules documented

**Iconography**
- [ ] Size variants: 16, 20, 24, 32 minimum
- [ ] Stroke weight consistent (1.5 or 2pt)
- [ ] Icon categories: Navigation · Action · Status · Financial (wallet, card, transfer icons)

**Elevation & Motion**
- [ ] Shadow tokens defined (`elevation/low`, `elevation/medium`, `elevation/high`)
- [ ] Transition durations documented (suggest: 150ms micro, 300ms standard, 500ms emphasis)

**Target page**: `02 – Foundations` with one section per category.

---

### 5.3 Core Components — Standard Set

For **each component**, verify:
- [ ] Purpose and usage documented
- [ ] All states present: Default · Hover/Press · Focused · Disabled · Loading · Error · Success
- [ ] Auto-layout enabled with proper constraints
- [ ] Naming follows: `ComponentName / Variant / State`
- [ ] Description set via `figma_set_description`

**Standard components** (search and audit each):
Buttons · Input Fields · Selection Controls (checkbox, radio, toggle) · Navigation (tab bar, bottom nav) · Cards · Modals · Bottom Sheets · Toasts / Snackbars · Badges · Avatars · Loaders

**Fintech-specific components** (critical for mobile banking — audit with extra care):
- [ ] **OTP / PIN Pad**: Numeric keypad, masked input, error shake state, biometric option
- [ ] **Amount Input**: Formatted number, currency selector, max/min validation states
- [ ] **Transfer Row**: Sender/receiver info, amount, status badge
- [ ] **Account Card**: Balance display, masked card number, card type icon
- [ ] **Transaction Item**: Icon + label + amount + timestamp, debit/credit color coding
- [ ] **eKYC Step Indicator**: Multi-step progress, current/completed/error states
- [ ] **Verification Badge**: ID verified, pending, failed states
- [ ] **Limit / Quota Bar**: Progress indicator for daily limits
- [ ] **Biometric Prompt**: Face ID / fingerprint UI pattern

**Target page**: `03 – Core Components` with one section per component.

---

### 5.4 Patterns (Optional)
- [ ] Transfer flow (multi-step)
- [ ] Login / biometric flow
- [ ] OTP verification flow
- [ ] Error recovery patterns

**Target page**: `04 – Patterns`

---

### 5.5 Maintenance & Governance
- [ ] Versioning strategy: Semantic versioning (MAJOR.MINOR.PATCH) documented
- [ ] Change log frame with date, version, author, summary
- [ ] Deprecation process: move to Archive, add `[DEPRECATED v2.x]` prefix, set redirect note
- [ ] Contribution rules: who can edit what, review process
- [ ] Design–Dev handoff: token mapping to CSS variables / platform tokens

**Target page**: `05 – Maintenance & Governance`

---

### 5.6 Post-Refactor Checklist
- [ ] All pages follow naming convention `XX – Name`
- [ ] Archive page exists and contains all deprecated items
- [ ] No orphaned styles (styles not used in any component)
- [ ] All components have descriptions
- [ ] Token names are semantic, not visual (`color/semantic/error` not `color/red-500`)
- [ ] No hardcoded hex values in components (all use variables)
- [ ] Component names have no spaces replaced by underscores

---

## 6. AUDIT REPORT FORMAT

### In-Figma Documentation Frame

Each section should include a `_doc` frame with this structure:
```
[Section Title]
[1-line purpose statement]
[Usage: when to use]
[Anti-patterns: when NOT to use]
[Figma token references]
[Last updated: date]
```

### Markdown Audit Report (export)

```markdown
# Design System Audit — [File Name]
**Date**: YYYY-MM-DD  **Auditor**: Claude (figma-agent-skill v0.1.0)

## Summary
- Total pages: X | Components: X | Variables: X | Styles: X
- Critical issues: X | Warnings: X | Suggestions: X

## Issue Log
| ID | Area | Severity | Finding | Recommendation |
|----|------|----------|---------|----------------|
| F-001 | Foundations / Color | CRITICAL | No semantic token layer | Create color/semantic/* collection |
| C-007 | OTP PIN Pad | WARN | Missing error shake state | Add error variant with animation spec |

## Refactor Actions Taken
- [x] Created Archive page and migrated X deprecated components
- [x] Renamed X components to follow naming convention
- [x] Added descriptions to X components

## Recommended Next Steps
1. ...
2. ...
```

---

## 7. TARGET PAGE STRUCTURE

```
01 – Design Language
02 – Foundations
03 – Core Components
04 – Patterns
05 – Maintenance & Governance
🗂 Archive
```

---

## 8. NAMING CONVENTIONS

**Pages**: `01 – Design Language`, `02 – Foundations` (zero-padded, em-dash)

**Components**: `Button / Primary / Default`, `Input / Amount / Error`, `Card / Account / Masked`

**Variables/Tokens**: `color/semantic/error`, `color/primitive/red-500`, `spacing/component/padding-md`, `typography/body/regular`

**Documentation frames**: `_doc / [Section]` (underscore prefix keeps them sorted at top)

---

## 9. PROMPT TRIGGER TEMPLATE

When the user provides a Figma URL and asks to audit or refactor, begin with:

```
Assumptions (adjust if incorrect):
- Spacing: 4pt grid
- Accessibility: WCAG AA
- Platform: Mobile (iOS + Android)
- Domain: Mobile Banking

Starting Phase 1 — Discovery...
[run figma_get_file_data, figma_get_variables, figma_get_design_system_summary]
```

Always surface the structural map to the user before writing anything to Figma.