# BRIEF — Vis4Cat
**Last updated:** 2026-07-06  
**Status:** TRL 2 → 3 (active development)  
**Lead:** Mohammad Khatamirad (BasCat / NFDI4Cat TA3-4)  
**Consortium measure:** Measure 3.5 — Basic Tools

---

## 1. Project Context

Vis4Cat is a visualization extension to **Repo4Cat**, the NFDI4Cat central data repository (DataVerse-based). It is developed as part of the Digital Catalysis Task Force within the NFDI4Cat consortium.

The core thesis: **plotting is not the goal — it is the entry point.** The long-term goal is interoperable, semantics-aware exploration of heterogeneous catalysis datasets across the community. Vis4Cat reduces the initial burden on users by delivering immediate value (visualization) before asking them to invest effort (metadata, semantic mapping).

---

## 2. Ecosystem Dependencies

| Tool | Role | Status |
|---|---|---|
| **CoreMeta4Cat** | LinkML-based metadata model (4 data classes) | In parallel development |
| **Voc4Cat** | NFDI4Cat controlled catalysis vocabulary (ontology) | Existing, actively maintained |
| **Repo4Cat / DataVerse** | Central repository — Vis4Cat integrates as a file previewer | Target integration platform |

Vis4Cat must be architected to accommodate CoreMeta4Cat and Voc4Cat from the start, even when not yet actively using them. Internal data structures must reserve slots for semantic annotations (e.g. `voc4cat_uri`) that are initially empty and populated at higher maturity tiers.

---

## 3. Data Classes

CoreMeta4Cat defines four data classes. Vis4Cat priority order:

| Class | Format | Current focus |
|---|---|---|
| **Reaction** | Tabular (CSV, XLSX) | ✅ Active |
| **Characterization** | Tabular or text files | 🔜 Next |
| Synthesis | Tabular | 🔜 Later |
| Simulation | Tabular | 🔜 Later |

---

## 4. Maturity Tier Model

Vis4Cat follows a step-wise maturity model. Each tier is a superset of the previous.

### Tier 1 — Basic ✅ COMPLETE (Reaction data class only)
- Read CSV / XLSX into a homogeneous data structure
- Auto-detect numeric and categorical columns (see §7 — Column Classification Logic)
- Render interactive scatter and line plot with variable selection via dropdowns
- Color / group-by support with high-cardinality warnings
- Dataset-level query filtering: range sliders for dual columns, checkbox lists for categorical columns
- Export plot as PNG

> **Scope note:** Tier 1 is validated against two Reaction datasets (STEU, catalyticData_b). Characterization, Synthesis, and Simulation data classes are not yet tested. Each data class may require adjustments to the classification logic, supported file formats, or default visualization behaviour before Tier 1 can be declared complete for that class.

### Tier 2 — Checkpoint (partially delivered ahead of schedule)
- ✅ Missing value detection and highlighting per column
- ✅ Descriptive statistics panel: mean, std, min, median, max, unique count, most frequent value, missing count — sortable, exportable as CSV
- ⬜ Template matching: suggest compatible plot types based on detected variable roles
- ⬜ Unit validation

### Tier 3 — Advanced (Basic Calculator)
- User-defined descriptors: compute new columns from existing ones via formula input
- Formula is evaluated as-is — no dimensional validation (that is a semantic-tier feature)
- Unsupervised ML: clustering (start), then PCA, UMAP
- Annotate data points / clusters interactively

### Tier 4 — Semantic
- Map raw column headers → Voc4Cat CURIEs (exact match, fuzzy, LLM-assisted)
- Use CoreMeta4Cat slot definitions to validate units and resolve synonyms
- Merge heterogeneous datasets via shared CURIEs as join keys
- Data transformation: aggregation, filtering, unit conversion
- Reusable visualization templates tied to CoreMeta4Cat data classes

**Architecture note:** The semantic layer must be considered in the architecture even at Tier 1. Columns should carry a `voc4cat_uri` field (nullable) from the start.

---

## 5. Software Architecture Principle

Vis4Cat follows a lightweight MVC-inspired separation of concerns — not as a strict framework, but as a discipline applied even within the single-file HTML demo:

| Layer | In Vis4Cat | Rule |
|---|---|---|
| **Model** | `state` object, `filterState` object, pure computation functions (`classifyColumns`, `computeStats`, `applyFilters`) | Never touches the DOM. Single source of truth. |
| **View** | Render functions (`renderPlot`, `renderStatsTable`, `renderFilterControls`, etc.) | Only reads from state. Never mutates data. |
| **Controller** | Event handlers (`onFilterColToggle`, `onRangeChange`, `onCatToggle`, `renderPlot` trigger) | Updates state first, then calls render. |

This discipline costs nothing at the current scale and prevents a structural rewrite when we move to a production framework for DataVerse integration.

---

## 6. Demo Datasets

### Dataset 1 — STEU (CO Hydrogenation, High-Throughput Screening)
- Source: Repo4Cat (open access)
- Structure: ~2100 rows, 257+ columns. High-throughput parallel reactor data. Conditions (T, P, GHSV, gas composition), catalyst descriptors (ID, support, metal loadings), time on stream, selectivity columns (naming pattern: `S_[Product]_[Reactant] [%]`)
- Key scientific insight: Selectivity columns (`S_Propane_CO [%]` etc.) are not immediately parseable by a machine without semantic mapping — live demonstration of why Voc4Cat matters
- **Demo plot:** Line — TOS [h] vs. selectivity column, colored by catalyst ID → stability over time across catalysts

### Dataset 2 — catalyticData_b (CO₂ Hydrogenation Performance)
- Source: Repo4Cat (open access)
- Structure: ~60 rows, tabular. Independent variables: temperature [degC], pressure [bar], feed composition. Dependent variable: X_CO2 (conversion). Product distribution: full alkane/alkene/alcohol series to C15. Categorical: channel, condition, catalyst_loading [g]
- No explicit selectivity column — raw outlet flows only
- **Demo plot:** Scatter — temperature [degC] vs. X_CO2, colored by catalyst_loading [g]
- **Tier 3 showcase:** User defines `S_CH4 = C1an_out / (CO2_in - CO2_out)`, plots derived descriptor → demonstrates Tier 1 → Tier 3 transition in one session

---

## 7. Column Classification Logic ✅ LOCKED

Agreed and implemented. Do not revisit without a concrete breaking case.

**Step 1 — dtype gate**
- String / object dtype → `categorical` (always)
- Numeric → proceed to Step 2

**Step 2 — numeric columns**
- Monotonically increasing integers with ratio > 0.95 → `index` (excluded from all dropdowns)
- `unique_count / n_rows < 0.05` AND `unique_count < 20` → `dual` (offered as axis AND color/group)
- Otherwise → `continuous`

**Step 3 — display limit (independent of classification)**
- Any column with unique_count > 15 offered as color → ⚠ high-cardinality warning shown
- Column is still available; user decides

**Role → UI mapping:**

| Role | X axis | Y axis | Color/group | Filter panel |
|---|---|---|---|---|
| `continuous` | ✅ | ✅ | — | — |
| `dual` | ✅ | ✅ | ✅ | ✅ range slider |
| `categorical` | — | — | ✅ | ✅ checkbox list |
| `index` | — | — | — | — |

---

## 8. Visualization Templates (Reaction Data Class)

Defined for Tier 2 (checkpoint) implementation. Not yet enforced at Tier 1.

| Template | X | Y | Color/Group |
|---|---|---|---|
| Scatter / Line | independent_variable | dependent_variable | categorical_variable |
| Distribution (Histogram) | — | dependent_variable | — |
| Aggregated Bar | categorical_variable | mean(dependent_variable) | — |
| Condition Sweep | independent_variable | dependent_variable | condition_variable (multiple lines) |

---

## 9. Demo Delivery Modes

| Mode | Purpose | When to use |
|---|---|---|
| **Self-contained HTML** | Shareable, no deployment, built in Claude | Fast iteration, internal review, this project space |
| **Lovable** | Rapid stakeholder feedback, polished UI | Community demos, external feedback rounds |
| **Streamlit / Dash** | More capable prototypes with Python backend | When JS-only is insufficient (e.g. ML tier) |
| **Local DataVerse instance** | Integration testing before central repo | TRL 3 technical integration phase |
| **Central Repo4Cat** | Production | Post TRL 3, requires maintainer coordination |

Git naming convention: `demo-{context}-{tool-name}` (e.g. `demo-nfdi4cat-vis4cat-basic`)

---

## 10. Decisions Log

| Date | Decision | Rationale |
|---|---|---|
| 2026-07-06 | Column classification logic locked (§7) | Agreed after reviewing both datasets; revisit only with a concrete breaking case |
| 2026-07-06 | Tier 3 formula engine = "basic calculator" (no dimensional validation) | Validation is a semantic-tier concern; don't over-engineer Tier 1→2 |
| 2026-07-06 | Statistics → slide-in panel (right), not sidebar box | Sidebar competes with plot controls; panel keeps plot visible alongside stats |
| 2026-07-06 | Scatter/line toggle = user-controlled, not auto-detected | Auto-detection of time axes is a heuristic that fails; one click is cheaper than a wrong default |
| 2026-07-06 | Start with Reaction data class only for Tier 1 | Scope control for TRL 2→3; Characterization next, Synthesis and Simulation deferred |
| 2026-07-06 | Histogram / distribution plot deferred to visualization panel | Stats panel = numbers only; histogram belongs to Tier 2 visualization templates |
| 2026-07-06 | Filter panel = dropdown below toolbar, not slide-in | Filters are adjusted frequently while looking at the plot; covering the plot defeats the purpose |
| 2026-07-06 | Filter column selection = user-picks (not all-auto) | Wide datasets (STEU: 257 cols) would generate unmanageable filter controls if all dual/cat shown |
| 2026-07-06 | Column inspector removed from sidebar | Redundant with stats panel; sidebar should do one job: control the plot |
| 2026-07-06 | MVC as discipline, not framework | Overkill as strict architecture at TRL 2→3; separation of concerns applied informally within single HTML file |

---

## 11. Open Decisions

- [ ] **Semantic mapping ownership:** Does mapping happen at deposit time (depositor) or at visualization time (viewer)? Where are confirmed mappings stored?
- [ ] **New descriptor scope:** Does "add descriptor" at Tier 3 mean proposing a new Voc4Cat term, or a free-text label outside the controlled vocabulary?
- [ ] **Characterization data:** File formats not yet defined (candidates: tabular CSV/XLSX for e.g. BET isotherms, XRD patterns; potentially proprietary instrument formats). Visualization types likely differ from Reaction (e.g. spectrum plots with wavenumber/2θ on X). Column classification logic may need adjustment. Requires at least one representative dataset before Tier 1 can be declared complete for this class.
- [ ] **Use-case collection:** Additional datasets from Repo4Cat beyond STEU and catalyticData_b
- [ ] **DataVerse integration:** Core software component to be identified; local test environment to be set up

---

## 12. Current Sprint Status

### Tier 1 — Basic ✅ COMPLETE for Reaction data class · ⬜ Not yet validated for Characterization, Synthesis, Simulation
- [x] Scope and maturity tier model defined
- [x] Column classification logic designed and implemented (locked)
- [x] File upload: CSV and XLSX support, drag & drop
- [x] Auto-detection of column roles with high-cardinality warnings
- [x] Scatter / line plot with X, Y, color/group dropdowns
- [x] Scatter ↔ Line toggle (user-controlled)
- [x] Dataset query filtering: range sliders (dual) + checkbox lists (categorical) with live re-render
- [x] Filter panel: column search, active filter badge, reset
- [x] Descriptive statistics slide-in panel (ahead of schedule — checkpoint tier)
- [x] Statistics export as CSV
- [x] Plot export as PNG
- [x] BRIEF.md, README.md, repo structure on GitHub

### Next — Tier 3 (Basic Calculator)
- [ ] Formula input UI for user-defined descriptors
- [ ] Column computation engine (pure JS, no dimensional validation)
- [ ] New descriptor appears in X/Y dropdowns immediately

### Next — DataVerse Integration (TRL 3)
- [ ] Identify core software component for DataVerse previewer
- [ ] Prepare local DataVerse test environment
- [ ] Implement local DataVerse integration
