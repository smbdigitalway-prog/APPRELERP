# Pre-Production Module — Review Findings

**Review marker:** `REVIEW-2026-11-19`
**Scope:** `apparel_erp` app as of commit `02b5c64` (21 Sep 2026)
**How to use:** each finding has a stable ID (`FND-xx`). Reference the ID in commit messages,
PRs and issues (e.g. `Fixes FND-11`). Update the **Status** column as work progresses:
`Open` → `In Progress` → `Done` / `Won't Do`. Add new review rounds as new sections below;
never renumber existing IDs.

---

## A. Functional / domain gaps

| ID | Priority | Finding | Suggestion | Status |
|---|---|---|---|---|
| FND-01 | P1 | `Apparel Order` is a dead end — nothing downstream creates Sales Order / Production Plan / Material Request. `Style TNA.po_reference` is free text, not a link. | Link Apparel Order → Sales Order → Material Request / Production Plan; make TNA link to Apparel Order. | Open |
| FND-02 | P1 | No material requirement planning (order qty × consumption × wastage → fabric/trim requirement), no fabric booking, lab-dip/strike-off/bulk approvals against suppliers, no lead times. | Add requirement-planning and material-approval tracking. | Open |
| FND-03 | P2 | Costing too basic: hard-coded INR/USD currency with no exchange rate; flat CMT/pc not derived from SMV × cost-per-minute; no size-wise costing; no quantity-slab costing; no FOB/CIF build-up; no quote version comparison or costed-vs-actual. | Costing v2: multi-currency + exchange rate, CMT from SMV, size-weighted cost, versioned quotes. | Open |
| FND-04 | P2 | Model assumes ~one order / one TNA per style; repeat orders, split shipments, per-colour delivery dates not handled. | Support multiple orders and TNAs per style. | Open |
| FND-05 | P2 | TNA is generic: only 3 auto milestones, no buyer-specific templates back-calculated from ex-factory date, no dependencies / critical path, no delay alerts. | TNA templates per buyer, backward scheduling, dependencies, notifications. | Open |
| FND-06 | P2 | No real Buyer/Brand master for buyer-specific size charts, tolerances, labelling/packing rules, compliance and test requirements. | Add Buyer master with these profiles. | Open |
| FND-07 | P3 | Measurements are a flat grid: no grading rules, tolerances (±), base size, or measured-vs-spec during fit sampling. | Extend Tech Pack measurements with grading + tolerance + fit comparison. | Open |
| FND-08 | P3 | Order matrix lacks prepacks / assortment ratios, third dimension (inseam/length), and carton/packing. | Extend order matrix model. | Open |

## B. Architecture / technical risks

| ID | Priority | Finding | Suggestion | Status |
|---|---|---|---|---|
| FND-09 | P2 | Entire UI is one ~3,550-line jQuery file (`page/style_workspace/style_workspace.js`) with string templates and manual event binding; bypasses Frappe form features (field permissions, mandatory checks, list views). Already caused known bugs (handler accumulation, pin filtering). | Modularise per tab; consider Frappe UI / Vue, or standard forms + lighter dashboard. | Open |
| FND-10 | P3 | Pin-annotation logic implemented three times independently (sketches, tech-pack reference images, sample photos). | One shared annotation component. | Open |
| FND-11 | P1 | Weak permissions: only System Manager / Manufacturing Manager roles; ~60 `ignore_permissions` calls (heaviest in `style_bom.py`, `sampling.py`); no Merchandiser/Designer/Sampling/Costing-Approver/Buyer roles; no user-level restrictions. Approvals are custom status pills, not Frappe Workflows. | Define roles + permission matrix, remove unnecessary `ignore_permissions`, use Frappe Workflow for cost approval / style confirmation. | Open |
| FND-12 | P1 | Zero automated tests. BOM generation, cost computation and sampling status derivation are untested. | Add unit/integration tests for these core flows first. | Open |
| FND-13 | P1 | `hooks.py` `required_apps` lists only `frappe` although the app depends on ERPNext (`Item`, `BOM`). | Add `erpnext` to `required_apps`. | Open |
| FND-14 | P1 | `hooks.py` Custom Field fixture filter lists 4 BOM fields but code and `fixtures/custom_field.json` use 5 — `custom_size` is missing from the filter and will be dropped on next `export-fixtures`. | Add `custom_size` to the fixture filter. | Open |
| FND-15 | P2 | Production BOMs are generate-once and locked forever; no change-request / revision flow for post-confirmation changes (trim swap, buyer change). | Add controlled BOM revision / change-request flow. | Open |
| FND-16 | P2 | Per-Colourway mode creates a synthetic non-sellable "carrier" Item with averaged consumption — MRP/purchasing will plan on averaged numbers. | Default to Per-SKU for production; restrict Per-Colourway to costing/estimation. | Open |
| FND-17 | P2 | No reporting layer (no Script/Query Reports or dashboards): style pipeline, sampling turnaround/rejections, TNA delays, costing margins. | Add core reports and a workspace dashboard. | Open |
| FND-18 | P3 | Multi-company / multi-season semantics unclear (style carry-over via `base_style`, inheritance rules, per-company masters). | Document and enforce carry-over and company scoping. | Open |
| FND-19 | P2 | Release hygiene: `main` historically diverged from working branch; "Remove all files" commit in history; no CI; `patches.txt` empty (no data migration path for schema changes). | Branch strategy, CI (lint + tests), use patches for schema/data changes, versioning. | Open |

## Suggested sequencing

1. **Foundations:** FND-12, FND-11, FND-13, FND-14, FND-19
2. **Close the loop:** FND-01, FND-02, FND-05
3. **Costing v2:** FND-03
4. **Buyer master & measurements:** FND-06, FND-07
5. **UI refactor (parallel):** FND-09, FND-10
6. **Reports, then sampling M4:** FND-17

## Change log

| Date | Change |
|---|---|
| 2026-11-19 | Initial review (`REVIEW-2026-11-19`), FND-01 … FND-19 recorded. |
