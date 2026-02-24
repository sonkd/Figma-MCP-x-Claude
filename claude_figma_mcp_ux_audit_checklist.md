# Claude + Figma MCP — UX Audit Checklist (Reusable .md)

**Purpose**
This checklist is a reusable Markdown template for running agentic UX audits with **Claude + Figma MCP** for banking/finance mobile screens. It enforces *Plan Mode*, captures Figma frames/screens, and writes structured audit results into the existing Figma page named **"UX Audit report template"**. Follow exactly.

---

## Quick facts & conventions
- **Audit lens:** Key Gestalt Principles (proximity, similarity, continuity, closure, figure–ground, symmetry/Prägnanz) + heuristic checks relevant to fintech (trust, clarity, error prevention).
- **Output format:** Markdown (and written directly into the Figma page template).
- **Severity → label mapping (use hex colors below for Figma pins / labels):**
  - **Usability catastrophe (P0)** — `#E60A32` (red)
  - **Major problem (P1)** — `#F58220` (orange)
  - **Minor problem (P2)** — `#2F6BFF` (blue)
  - **Cosmetic problem (P3)** — `#00BC3C` (green)

---

## Pre-flight (required before running the agent)
- [ ] You are an **Editor** on the target Figma file.
- [ ] Figma Desktop Bridge plugin is installed and **Write Mode enabled** (if you intend agent to write). If only reading, Write Mode not required.
- [ ] Provide Frame IDs or select frames and confirm selection in MCP command.
- [ ] Provide: `SCREEN_NAME`, `FLOW`, `PERSONA` (1-line), `RISK_LEVEL` (High/Medium/Low), `KPIs` list, and any `CONSTRAINTS` (tokens, platform, compliance)
- [ ] Confirm page **"UX Audit report template"** exists on the current Figma page and is writable.

---

## High-level Plan Mode instruction (MANDATORY for every run)
1. **Enter Plan Mode first.** Produce a 1–2 paragraph plan describing: scope, frames to inspect, who will review, and verification steps.
2. Await human confirmation ("Plan approved") before executing any reads/writes.
3. If anything goes sideways, **re-enter Plan Mode**, re-plan, and seek approval.

> After implementation and every correction: append an entry to `http://CLAUDE.md` (project notes) summarizing what changed and why. End with: "Update your http://CLAUDE.md so you don't make that mistake again." 

---

## Step-by-step checklist (to be executed by Claude when permitted)

### STEP A — Planning (Plan Mode)
- [ ] 1. Summarize the audit scope (frames, persona, flow, KPIs, risk level).
- [ ] 2. List the Gestalt Principles and fintech heuristics you will apply.
- [ ] 3. Provide a 1-paragraph verification plan (how you will validate findings and what "evidence" means: layer names, tokens, spacing values).
- [ ] 4. Output: short plan block (2–4 bullets). Wait for "Plan approved."

### STEP B — Pre-capture checks (after plan approval)
- [ ] 1. Confirm file + frames are accessible. If not, return an error block with next steps.
- [ ] 2. Confirm Write Mode enabled (if you will write into Figma). If not, request user to enable it.
- [ ] 3. Capture screenshots of selected frames using Figma MCP `screencapture` (store into the current page under the frame name).

### STEP C — Visual audit (read-only inspection)
For each captured frame:
- [ ] 1. Inspect visual hierarchy & spacing, list violations of Gestalt rules.
- [ ] 2. Check fintech-specific risks: ambiguous amounts, unclear confirmation, PII exposure, error messages, masking, and trust signals.
- [ ] 3. Gather evidence: exact layer names, token names (colors, typography), spacing values (px), and any overrides.
- [ ] 4. Create 1–3 concise findings per frame (max 6) mapping to the Output Template below.

### STEP D — Write results into Figma
*(Only after Plan approved & Write Mode enabled)*
- [ ] 1. Locate the page named **"UX Audit report template"** in the same Figma file.
- [ ] 2. For each frame, create a copy of the template frame named: `UX Audit — [SCREEN_NAME]`.
- [ ] 3. Paste the captured screenshot into the template placeholder.
- [ ] 4. For each finding, add an item in the template with:
  - Numbered bullet
  - Short title
  - Severity label (text + color dot using specified hex)
  - One-line description
  - Evidence (layer name(s), token name(s))
  - One concrete recommendation (token change, spacing px, copy change)
  - Quick win? (yes/no)
  - Business impact (map to KPI)
  - Testable hypothesis + suggested metric to track
- [ ] 5. Save & commit changes (create a changelog note in the page: `audited-by: Claude | date: YYYY-MM-DD`)

### STEP E — Produce external Markdown report (same content)
- [ ] 1. Output the full audit report as Markdown in the Claude response.
- [ ] 2. Include top 3 prioritized fixes, effort estimate, and suggested experiment specs (sample size, metric, success criteria).
- [ ] 3. End the report with: `Update your http://CLAUDE.md so you don't make that mistake again.`

---

## Output Template (use exactly – for both Figma template and Markdown)
> Use this for each finding. Keep text concise.

```
### [#] Short title — [Severity label] (color: #[HEX])
**Principle(s):** [Gestalt principle(s) – e.g., Proximity, Figure–ground]
**Evidence:** [layer_name(s); token_name(s); spacing: Xpx]
**Description:** 1–2 sentences describing the problem
**Recommendation:** Step-by-step change (token name / px / copy) + example
**Quick win:** [Yes/No]
**Business impact:** [maps to KPI: e.g., +completion rate, -error rate]
**Testable hypothesis:** 1 sentence
**Suggested metric:** [metric name]
```

---

## Examples (one minimal example)
```
### 1. Primary CTA lost in visual noise — Usability catastrophe (color: #E60A32)
**Principle(s):** Proximity, Figure–ground
**Evidence:** layer: btn_send_primary; token: color_brand_400; margin-right: 8px
**Description:** CTA reads visually equal to inline text links; low contrast and insufficient spacing.
**Recommendation:** Increase elevation `elevation_1`, change token to `color_brand_600`, add 16px right margin; change label to "Send $12.50".
**Quick win:** Yes
**Business impact:** Expected +↑ completion rate for payment confirm by ~5%.
**Testable hypothesis:** After changes, completion rate increases by >=5% in moderated tests.
**Suggested metric:** completion_rate.payment_confirm
```

---

## Rules for agent behavior (strict)
1. **Plan Mode first.** Always produce a plan and wait for human approval. Do not proceed until user replies "Plan approved."
2. **Evidence-only findings.** Cite layer names and tokens. If data is missing, label as `Assumption` and propose tests.
3. **Conservative edits.** When writing into Figma, do not auto-rename tokens or move components without explicit "Apply changes" confirmation.
4. **Maintain http://CLAUDE.md.** After every run, append a short note: what was done, one mistake found & fix, and the line: "Update your http://CLAUDE.md so you don't make that mistake again."  
5. **Fail-safe:** If conflict risk > medium (component is used in >N places — where-used count), recommend a fallback: duplicate file & apply changes to copy.

---

## One-line ready-to-send prompt (replace bracketed values)
```
Plan Mode: You are an expert Senior Product Designer conducting a Gestalt-based UX audit. FIGMA_FRAMELIST: [paste frame ids]. SCREEN_NAME: [name]. FLOW: [flow]. PERSONA: [one-line]. RISK_LEVEL: [High]. KPIs: [list]. CONSTRAINTS: [e.g., WCAG AA, iOS]. Use the checklist at /Claude_Figma_MCP_UX_Audit_Checklist.md. First produce a short Plan (2–4 bullets). Wait for "Plan approved" before proceeding. Read-only audit first. If user confirms write, I will enable write mode and you will write findings into page "UX Audit report template". Return Markdown report and write into Figma when permitted.
```

---

## Post-run verification & logs
- [ ] Human reviews Figma page entries and verifies layer references.
- [ ] If edits applied, run smoke-check: rename test frame, open in prototype, verify no break.
- [ ] Update `http://CLAUDE.md` with a short entry describing mistakes avoided and improvements learned.

---

## Where to store & reuse
- Save this file as `Claude_Figma_MCP_UX_Audit_Checklist.md` in your project notes repo.
- Link the file in the Claude project `notes_dir` and point `http://CLAUDE.md` to that path.

---

If you want, I can now:
- Generate a ready-to-run prompt with your frame ids inserted, or
- Produce the `http://CLAUDE.md` entry template to maintain after every run.

