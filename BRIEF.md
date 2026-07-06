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
| **Characterization** | Tabular or text files | ✅ Active |
| Synthesis | Tabular | 🔜 Later |
| Simulation | Tabular | 🔜 Later |

---

## 4. Maturity Tier Model

Vis4Cat follows a step-wise maturity model. Each tier is a superset of the previous.

### Tier 1 — Basic
- Read CSV / XLSX / JSON into a homogeneous data structure
- Auto-detect numeric and categorical columns (see §6 — Column Classification Logic)
- Render interactive plot (scatter, line)
- Interactions: zoom, hover, filter, select variables via dropdowns
- Export plot as HTML / PNG / SVG

### Tier 2 — Checkpoint
- Validate column types, units, missing values
- Template matching: suggest compatible plot types based on detected variable roles
- Descriptive statistics panel (per-column: mean, std, min, max, missing count, unique count)

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

## 5. Demo Datasets

### Dataset 1 — STEU (CO Hydrogenation, High-Throughput Screening)
- Source: Repo4Cat (open access)
- Structure: High-throughput parallel reactor data. Conditions (T, P, GHSV, gas composition), catalyst descriptors (ID, support, metal loadings), time on stream, and selectivity columns (naming pattern: `S_[Product]_[Reactant] [%]`)
- Key scientific insight: Selectivity columns (`S_Propane_CO [%]` etc.) are not immediately parseable by a machine without semantic mapping — this is a live demonstration of why Voc4Cat matters
- **Intended demo plot:** Line plot — TOS [h] vs. a selectivity column, colored by catalyst ID → stability over time across catalysts

### Dataset 2 — catalyticData_b (CO₂ Hydrogenation Performance)
- Source: Repo4Cat (open access)
- Structure: Clean tabular dataset. Independent variables: temperature [degC], pressure [bar], feed composition. Dependent variable: X_CO2 (conversion). Product distribution: full alkane/alkene/alcohol series to C15. Categorical: channel, condition, catalyst_loading [g]
- No explicit selectivity column — raw outlet flows only
- **Intended demo plot:** Scatter — temperature [degC] vs. X_CO2, colored by catalyst_loading [g]
- **Intended Tier 3 showcase:** User defines selectivity descriptor (e.g. `S_CH4 = C1an_out / (CO2_in - CO2_out)`), then plots S_CH4 vs. X_CO2

### Demo Story Arc (catalyticData_b)
1. Upload → immediate scatter plot, value in seconds
2. User notices no selectivity column
3. User opens basic calculator, defines formula, new column appears
4. User plots derived descriptor — demonstrates Tier 1 → Tier 3 transition in one session

---

## 6. Column Classification Logic ✅ LOCKED

Agreed and implemented. Do not revisit without a concrete breaking case.

**Step 1 — dtype gate**
- String / object dtype → `categorical` (always)
- Numeric → proceed to Step 2

**Step 2 — numeric columns**
- Monotonically increasing integers with ratio > 0.95 → `index` (excluded from all dropdowns)
- `unique_count / n_rows < 0.05` AND `unique_count < 20` → `dual` (offered as both axis and color/group)
- Otherwise → `continuous`

**Step 3 — display limit (independent of classification)**
- Any column with unique_count > 15 offered as color → ⚠ high-cardinality warning shown
- Column is still available, user decides

**Role → dropdown mapping:**
- X axis: `continuous`, `dual`
- Y axis: `continuous`, `dual`
- Color / group by: `categorical`, `dual`
- Excluded: `index`

---

## 7. Visualization Templates (Reaction Data Class)

Defined for Tier 2 (checkpoint) implementation. Not yet enforced at Tier 1.

| Template | X | Y | Color/Group |
|---|---|---|---|
| Scatter / Line | independent_variable | dependent_variable | categorical_variable |
| Distribution (Histogram) | — | dependent_variable | — |
| Aggregated Bar | categorical_variable | mean(dependent_variable) | — |
| Condition Sweep | independent_variable | dependent_variable | condition_variable (multiple lines) |

---

## 8. Demo Delivery Modes

| Mode | Purpose | When to use |
|---|---|---|
| **Self-contained HTML** | Shareable, no deployment, built in Claude | Fast iteration, internal review, this project space |
| **Lovable** | Rapid stakeholder feedback, polished UI | Community demos, external feedback rounds |
| **Streamlit / Dash** | More capable prototypes with Python backend | When JS-only is insufficient (e.g. ML tier) |
| **Local DataVerse instance** | Integration testing before central repo | TRL 3 technical integration phase |
| **Central Repo4Cat** | Production | Post TRL 3, requires maintainer coordination |

Git naming convention: `demo-{context}-{tool-name}` (e.g. `demo-nfdi4cat-vis4cat-basic`)

---

## 9. Decisions Log

| Date | Decision | Rationale |
|---|---|---|
| 2026-07-06 | Column classification logic locked (§6) | Agreed after reviewing both datasets; revisit only with a concrete breaking case |
| 2026-07-06 | Tier 3 formula engine = "basic calculator" (no dimensional validation) | Validation is a semantic-tier concern; don't over-engineer Tier 1→2 |
| 2026-07-06 | Report / statistics → slide-in panel on right, not sidebar box | Sidebar competes with plot controls; panel keeps plot visible alongside stats |
| 2026-07-06 | Scatter/line toggle = user-controlled, not auto-detected | Auto-detection of time axes is a heuristic that fails; one click is cheaper than a wrong default |
| 2026-07-06 | Start with Reaction + Characterization data classes only | Scope control for TRL 2→3; Synthesis and Simulation deferred |

---

## 10. Open Decisions

- [ ] **Statistics panel scope:** Which stats for categorical columns? (unique count, mode, missing — confirmed). Any catalysis-specific checks to add (e.g. constant columns, percentage-range detection)?
- [ ] **Semantic mapping ownership:** Does mapping happen at deposit time (depositor) or at visualization time (viewer)? Where are confirmed mappings stored?
- [ ] **New descriptor scope:** Does "add descriptor" at Tier 3 mean proposing a new Voc4Cat term, or just a free-text label outside the controlled vocabulary?
- [ ] **Characterization data:** Specific file formats and visualization types not yet defined (Tier 1 focus is tabular Reaction data)
- [ ] **Use-case collection:** Additional datasets from Repo4Cat beyond STEU and catalyticData_b

---

## 11. Current Sprint (TRL 2 → 3)

**Done:**
- [x] Scope and maturity tier model defined
- [x] Column classification logic designed and implemented
- [x] Basic visualization engine (HTML demo v1): file upload, auto-detection, scatter/line, color grouping, export
- [x] Dataset report / auto-analysis (integrated, then moved to statistics panel — in progress)

**In progress:**
- [ ] Descriptive statistics slide-in panel (right side, toolbar button)

**Next:**
- [ ] Basic calculator (user-defined descriptors)
- [ ] Identify core software component for DataVerse integration
- [ ] Prepare local DataVerse test environment
