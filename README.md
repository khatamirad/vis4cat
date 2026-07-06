# Vis4Cat

> Interactive visualization extension for [Repo4Cat](https://www.nfdi4cat.org), the central data repository of the [NFDI4Cat](https://www.nfdi4cat.org) consortium.

**Status:** TRL 2 → 3 — Tier 1 complete for Reaction data class  
**Consortium measure:** Measure 3.5 — Basic Tools (TA 3-4)  
**Lead:** Mohammad Khatamirad — BasCat (BASF & TU Berlin JointLab) / NFDI4Cat

---

## What is Vis4Cat?

Catalysis datasets deposited in Repo4Cat are currently static — researchers can download them but cannot explore them in place. Vis4Cat adds an interactive visualization layer directly within the repository.

Plotting is not the goal. It is the entry point. The long-term goal is interoperable, semantics-aware exploration of heterogeneous catalysis datasets across the community.

For full technical detail — architecture, decisions log, classification logic, sprint status — see [BRIEF.md](./BRIEF.md).

---

## Repository Structure

```
vis4cat/
├── BRIEF.md          # Full project brief, architecture, decisions log, sprint status
├── README.md         # This file
├── demos/            # Self-contained HTML demos (open in any browser, no install needed)
└── data/             # Test dataset references (see data/README.md)
```

---

## Running the Demo

No installation required. Download any file from `demos/` and open it in a browser:

```bash
open demos/vis4cat_demo_v1.html
```

Upload a CSV or XLSX file. The tool runs entirely client-side — no server, no dependencies.

---

## Development Notes

This project is developed with AI assistance. Claude (Anthropic) is used as an active co-development tool for architecture decisions, code generation, and documentation. All design decisions are reviewed and owned by the human project lead.

Contributions and feedback from the NFDI4Cat community are welcome — please open an issue.

---

## License

This repository is currently unlicensed. All rights reserved.  
The code may not be used, copied, modified, or distributed without explicit permission from the project lead and NFDI4Cat consortium.  
Licensing will be revisited as the project matures.

---

*NFDI4Cat is funded by the Deutsche Forschungsgemeinschaft (DFG).*
