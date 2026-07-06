# Vis4Cat

> Interactive visualization extension for [Repo4Cat](https://www.nfdi4cat.org), the central data repository of the [NFDI4Cat](https://www.nfdi4cat.org) consortium.

**Status:** TRL 2 → 3 — active development  
**Consortium measure:** Measure 3.5 — Basic Tools (TA 3-4)  
**Lead:** Mohammad Khatamirad — BasCat (BASF & TU Berlin JointLab) / NFDI4Cat

---

## What is Vis4Cat?

Catalysis datasets deposited in Repo4Cat are currently static — researchers can download them but cannot explore them in place. Vis4Cat changes that by providing an interactive visualization layer directly within the repository.

The goal is **not** plotting. Plotting is the entry point. The long-term goal is interoperable, semantics-aware exploration of heterogeneous catalysis datasets across the community — enabling dataset comparison, cross-experiment analysis, and reusable visualization templates grounded in the NFDI4Cat data infrastructure.

---

## Features (current — basic tier)

- Upload CSV or XLSX datasets directly in the browser
- Automatic column classification: continuous, categorical, dual-role, index
- Interactive scatter and line plots with variable selection via dropdowns
- Color / group-by support with high-cardinality warnings
- Dataset report: row/column summary, missing value detection, suggested axes
- Export plot as PNG (via Plotly toolbar)
- Zero backend — fully self-contained HTML, runs in any browser

---

## Maturity Roadmap

Vis4Cat is developed incrementally across four maturity tiers:

| Tier | Name | Key capabilities |
|---|---|---|
| 1 | **Basic** | File ingestion, column detection, interactive scatter/line plot |
| 2 | **Checkpoint** | Validation, missing value detection, descriptive statistics, template matching |
| 3 | **Advanced** | User-defined descriptors (basic calculator), unsupervised ML (clustering, PCA) |
| 4 | **Semantic** | Voc4Cat column mapping, dataset merging, unit conversion, reusable templates |

---

## Ecosystem

Vis4Cat is part of a broader NFDI4Cat toolchain:

- **[CoreMeta4Cat](https://github.com/nfdi4cat/CoreMeta4Cat)** — LinkML-based metadata standard for catalysis data (Reaction, Synthesis, Characterization, Simulation)
- **Voc4Cat** — NFDI4Cat controlled vocabulary for catalysis concepts
- **Repo4Cat** — DataVerse-based central repository; Vis4Cat targets integration as a file previewer

---

## Repository Structure

```
vis4cat/
├── BRIEF.md          # Full project brief, architecture decisions, decisions log
├── README.md         # This file
├── demos/            # Self-contained HTML demos (runnable in browser)
└── data/             # Test dataset references (see data/README.md)
```

---

## Demo Datasets

Two open-access datasets from Repo4Cat are used for development and testing:

| Dataset | Reaction type | Focus |
|---|---|---|
| STEU | CO hydrogenation (high-throughput screening) | Time-series stability, catalyst comparison |
| catalyticData_b | CO₂ hydrogenation | Conversion vs. temperature, derived descriptor showcase |

Datasets are not committed to this repository. They are available via Repo4Cat.

---

## Running the Demo

No installation required. Open any file from `demos/` directly in a browser:

```bash
open demos/vis4cat_demo_v1.html
```

Then upload a CSV or XLSX file. The tool runs entirely client-side.

---

## Development Notes

This project is developed with AI assistance. Claude (Anthropic) is used as an active co-development tool for architecture decisions, code generation, and documentation. All design decisions are reviewed and owned by the human project lead.

Contributions and feedback from the NFDI4Cat community are welcome — please open an issue.

---

## License

MIT License — see [LICENSE](LICENSE) for details.

---

*NFDI4Cat is funded by the Deutsche Forschungsgemeinschaft (DFG) — project number 441926934.*
