# Drug Development White Space Analyzer

A pharmaceutical landscape analysis tool that identifies underserved diseases and therapeutic areas where new drug development could have the greatest impact. Built as a Claude skill for use with [Anthropic's Claude](https://claude.ai).

## What It Does

The analyzer maintains a living Excel database of diseases mapped against their approved treatments, clinical pipeline, basic science activity, and economic burden. It scores each disease across five dimensions to produce a **White Space Matrix** — a ranked view of where the biggest drug development opportunities exist.

**Example questions it answers:**
- "Where is the most open space for a new drug right now?"
- "What drug developments are occurring for ALS?"
- "Rank diseases by patient population relative to number of treatments"
- "Which rare diseases have regulatory advantages but no pipeline?"

## Architecture

```
drug-whitespace-analyzer/
├── SKILL.md                          # Main skill instructions (v2.0)
├── scripts/
│   ├── build_workbook.py             # Creates/populates the Excel database
│   ├── score_diseases.py             # 5-dimension scoring algorithm
│   └── visualize_matrix.py           # Bubble charts, bar charts, heatmaps
├── references/
│   ├── data_schema.md                # Excel workbook schema (8 tabs)
│   └── scoring_methodology.md        # Scoring formulas and weights
└── evals/
    └── evals.json                    # Benchmark test cases
```

### Data Flow

```
  Data Sources                    Processing                    Outputs
┌──────────────┐
│ ClinicalTrials│──┐
│    .gov       │  │         ┌──────────────┐          ┌──────────────────┐
├──────────────┤  ├────────▶│build_workbook │─────────▶│  Excel Database   │
│   PubMed     │──┤         │    .py        │          │   (8 tabs)       │
├──────────────┤  │         └──────────────┘          └────────┬─────────┘
│  WebSearch   │──┤                                            │
│ (CDC/FDA/WHO)│  │         ┌──────────────┐                   │
├──────────────┤  │         │score_diseases │◀──────────────────┤
│  ICD-10      │──┘         │    .py        │                   │
└──────────────┘            └──────┬───────┘                   │
                                   │                            │
                                   ▼                            │
                          ┌──────────────┐          ┌──────────▼─────────┐
                          │  Scored Data  │─────────▶│ visualize_matrix   │
                          │  + Historical │          │      .py           │
                          │    Trends     │          └──────────┬─────────┘
                          └──────────────┘                     │
                                                               ▼
                                                    ┌──────────────────┐
                                                    │  6 Chart Types   │
                                                    │  (PNG output)    │
                                                    └──────────────────┘
```

## Scoring Model (v2.0)

Each disease is scored 0–10 on five dimensions, then combined into a composite White Space Score:

| Dimension | Weight | What It Measures |
|-----------|--------|------------------|
| **Unmet Patient Need** | 30% | Prevalence × severity × mortality, penalized by existing treatment count |
| **Drug Coverage Gap** | 25% | Ratio of patient population to available drugs (approved + phase-weighted pipeline) |
| **Scientific Complexity** | 15% | Inverse of R&D activity — less activity = harder problem = higher score |
| **Regulatory Advantage** | 15% | Fraction of pipeline with orphan/breakthrough/fast-track designations + rare disease bonus |
| **Health Economics** | 15% | Annual cost per patient, total economic burden, and QALY burden with gap amplification |

Pipeline drugs are phase-weighted: Phase 1 = 0.1, Phase 2 = 0.3, Phase 3 = 0.6, Phase 4 = 0.9.

The model also computes **approval probability** per disease using therapeutic-area-specific base rates (from BIO/QLS industry data), with boosts for orphan and breakthrough designations.

## Database Schema

The Excel workbook contains 8 tabs:

1. **Disease Epidemiology** — Prevalence, incidence, mortality, severity, cost, economic burden, QALY
2. **Approved Drugs** — FDA-approved treatments with MOA, regulatory designations, pricing
3. **Pipeline Drugs** — Active clinical trials with phase, sponsor, NCT ID, regulatory status
4. **Basic Science Activity** — PubMed publications, NIH/DARPA grants, funding data
5. **White Space Scores** — Computed scores across all 5 dimensions + composite
6. **Therapeutic Area Summary** — Aggregated stats by therapeutic area
7. **Historical Trends** — Quarterly snapshots for longitudinal tracking
8. **Approval Probability** — Reference table of phase-transition rates by therapeutic area

## Current Coverage

The included database covers **50 diseases** across these therapeutic areas:

- **Oncology** (10): Lung, breast, colorectal, pancreatic, liver, melanoma, prostate, ovarian, glioblastoma, bladder
- **Neurology** (7): Alzheimer's, Parkinson's, ALS, MS, epilepsy, migraine, Huntington's
- **Cardiovascular** (4): Heart failure, atrial fibrillation, PAD, pulmonary hypertension
- **Endocrine/Metabolic** (4): Type 2 diabetes, obesity, type 1 diabetes, NAFLD/NASH
- **Respiratory** (4): COPD, asthma, IPF, cystic fibrosis
- **Infectious Disease** (6): HIV/AIDS, tuberculosis, hepatitis B, hepatitis C, measles, RSV
- **Autoimmune** (5): Rheumatoid arthritis, lupus, Crohn's, psoriasis, ulcerative colitis
- **Hematology** (2): Sickle cell disease, hemophilia A
- **Rare Disease** (3): DMD, SMA, Rett syndrome
- **Mental Health** (3): MDD, schizophrenia, PTSD
- **Other** (2): CKD, osteoarthritis

## Visualizations

The tool generates six chart types:

1. **Bubble Chart** — X: coverage gap, Y: unmet need, size: prevalence, color: complexity
2. **Health Economics Bubble** — Same axes, colored by health economics score
3. **Ranked Bar Chart** — Top 20 diseases by composite score, stacked by dimension
4. **Therapeutic Area Summary** — Average scores grouped by therapeutic area
5. **Specialty Segments** — Comparison across pediatric/geriatric/rare/general populations
6. **Regulatory Landscape** — Diseases ranked by regulatory advantage score

## Skill Modes

The analyzer supports five operating modes:

| Mode | Description |
|------|-------------|
| **1. Build Database** | Pull data for a set of diseases and populate the workbook |
| **2. Query & Analyze** | Answer natural-language questions against the existing database |
| **3. Score & Rank** | Recompute scores and generate fresh visualizations |
| **4. Deep Dive Report** | Full analysis of a single disease (competitive landscape, pipeline map, regulatory path) |
| **5. Quarterly Refresh** | Update pipeline data, check new approvals, append historical snapshot |

## Requirements

- Python 3.8+
- `openpyxl` (Excel read/write)
- `matplotlib` (chart generation)
- Claude with MCP access to PubMed, ClinicalTrials.gov, and WebSearch

## Usage

**As a Claude Skill:** Install the `.skill` package and the skill will trigger automatically when you ask about drug development gaps, unmet medical needs, or pharmaceutical pipeline analysis.

**Scripts standalone:**

```bash
# Create a new empty database
python scripts/build_workbook.py database.xlsx

# Add disease data from a JSON file
python scripts/build_workbook.py database.xlsx --add-diseases data.json

# Score all diseases
python scripts/score_diseases.py database.xlsx

# Generate visualizations
python scripts/visualize_matrix.py database.xlsx --output-dir ./charts
```

## Data Sources

All data is pulled from public, authoritative sources:

- **ClinicalTrials.gov** — Pipeline drugs, trial phases, sponsors, regulatory designations
- **PubMed / PMC** — Basic science publications, funding announcements
- **CDC / NCI SEER** — US prevalence, incidence, mortality statistics
- **WHO / GBD** — Global disease burden data
- **FDA** — Approved drug lists, orphan/breakthrough/accelerated approval designations
- **ACS / disease-specific foundations** — Supplementary epidemiology data
- **BIO/QLS** — Clinical trial success rate benchmarks by therapeutic area

## License

This project is provided as-is for research and educational purposes. The underlying data is sourced from public databases and should be independently verified before use in any commercial or clinical decision-making context.

---

*Built with Claude by Anthropic*
