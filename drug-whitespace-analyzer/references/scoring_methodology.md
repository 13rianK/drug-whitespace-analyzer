# White Space Scoring Methodology v2.0

## Overview

Each disease is scored on five dimensions (0-10 scale), then combined into a composite White Space Score. v2.0 adds regulatory advantage and health economics dimensions, plus an approval probability model.

## Dimension 1: Unmet Patient Need (0-10) — Weight: 30%

Same calculation as v1 (see below). Captures patient suffering relative to available solutions.

### Inputs
- `us_prevalence`, `severity_category`, `approved_drug_count`, `mortality_rate_per_100k`

### Calculation
```
prevalence_score = normalize_log(us_prevalence, 1000, 50_000_000) * 10
severity_weight = {life-threatening: 1.0, debilitating: 0.75, chronic-manageable: 0.5, acute-treatable: 0.25}
treatment_adequacy = min(approved_drug_count / 5, 1.0)
mortality_factor = normalize_log(mortality_rate_per_100k, 0.1, 100) * 10

unmet_need = prevalence_score * 0.30 + severity_weight * 10 * 0.35 + (1 - treatment_adequacy) * 10 * 0.25 + mortality_factor * 0.10
```

## Dimension 2: Drug Coverage Gap (0-10) — Weight: 25%

Same calculation as v1. Measures drug availability relative to patient population.

### Calculation
```
total_coverage = approved_drug_count + weighted_pipeline_count
coverage_per_million = total_coverage / (us_prevalence / 1_000_000)
raw_gap = 10 - normalize_log(coverage_per_million, 0.01, 1000) * 10
# MOA diversity adjustment: +1.5 if pipeline lacks diverse mechanisms
drug_coverage_gap = clamp(raw_gap + diversity_adjustment, 0, 10)
```

## Dimension 3: Scientific Complexity (0-10) — Weight: 15% (inverted)

Same as v1. Lower complexity = more actionable. Inverted in composite: `(10 - complexity) × 0.15`.

## Dimension 4: Regulatory Advantage (0-10) — Weight: 15% ← NEW

Captures how favorable the regulatory environment is for new drugs in this disease area. Higher scores mean more regulatory tailwinds — orphan designations, breakthrough therapies, fast-track grants — which lower the barrier to entry.

### Inputs
- Approved drugs: count with orphan_drug=yes, breakthrough_therapy=yes, accelerated_approval=yes
- Pipeline drugs: count with orphan_designation=yes, breakthrough_designation=yes, fast_track=yes
- Disease: is it a rare disease (<200K US prevalence)?

### Calculation
```
# What fraction of approved + pipeline drugs have regulatory designations?
total_drugs = approved_count + pipeline_count
if total_drugs == 0:
    regulatory_advantage = 5.0  # neutral default

orphan_fraction = count(orphan=yes) / total_drugs
breakthrough_fraction = count(breakthrough=yes) / total_drugs
fast_track_fraction = count(fast_track=yes) / total_drugs

# Rare disease bonus
rare_bonus = 2.0 if us_prevalence < 200000 else 0.0

regulatory_advantage = clamp(
    orphan_fraction * 10 * 0.35 +
    breakthrough_fraction * 10 * 0.35 +
    fast_track_fraction * 10 * 0.15 +
    rare_bonus * 0.15,
    0, 10
)
```

### Interpretation
- 8-10: Strong regulatory tailwinds — many orphan/breakthrough designations, FDA actively encouraging
- 5-7: Moderate — some designations present
- 2-4: Standard regulatory path — no special designations
- 0-1: No regulatory advantage signals

### Why This Matters
Orphan drug designation grants 7 years market exclusivity, tax credits, and reduced user fees. Breakthrough therapy speeds review. Fast track enables rolling review. These create economic moats that make drug development more attractive for investors.

## Dimension 5: Health Economics Score (0-10) — Weight: 15% ← NEW

Captures the economic desperation — diseases where the cost burden is enormous relative to treatment availability. High scores signal diseases where payers, employers, and health systems would enthusiastically adopt new treatments.

### Inputs
- `annual_cost_per_patient_usd`: Average annual treatment cost
- `total_us_economic_burden_usd`: Total direct + indirect costs
- `qaly_burden`: QALYs lost per patient per year
- `approved_drug_count`: Existing treatment options

### Calculation
```
cost_score = normalize_log(annual_cost_per_patient_usd, 1000, 500000) * 10
burden_score = normalize_log(total_us_economic_burden_usd, 1_000_000, 500_000_000_000) * 10
qaly_score = normalize_log(qaly_burden, 0.01, 1.0) * 10 if qaly_burden > 0 else 5.0

# Treatment gap amplifier: high costs with few drugs = very high econ score
gap_amplifier = 1.0 + (1 - min(approved_drug_count / 5, 1.0)) * 0.3

health_econ = clamp(
    (cost_score * 0.30 + burden_score * 0.40 + qaly_score * 0.30) * gap_amplifier,
    0, 10
)
```

### Interpretation
- 8-10: Massive economic burden, payers desperate for alternatives
- 5-7: Significant cost but some options exist
- 2-4: Moderate economic impact, reasonable treatment costs
- 0-1: Low-cost disease, well-managed economically

## Composite White Space Score v2.0

```
composite = (
    unmet_need × 0.30 +
    drug_coverage_gap × 0.25 +
    (10 - scientific_complexity) × 0.15 +
    regulatory_advantage × 0.15 +
    health_econ × 0.15
)
```

### Weight Rationale (v2.0 changes)
- **Unmet Need (30%, was 40%):** Still most important, but reduced slightly to make room for new dimensions
- **Drug Coverage Gap (25%, was 35%):** Still critical for identifying actual white space
- **Scientific Complexity inverted (15%, was 25%):** Reduced — complexity matters but shouldn't dominate
- **Regulatory Advantage (15%, new):** Captures whether FDA incentives exist to de-risk development
- **Health Economics (15%, new):** Captures economic pull — where payers would welcome new entrants

## Approval Probability Model

### Historical Rates by Therapeutic Area

Based on published industry data (BIO/QLS Advisors/Informa Pharma Intelligence clinical development success rates):

| Therapeutic Area | P1→Approval | P2→Approval | P3→Approval | Orphan Boost | Breakthrough Boost |
|-----------------|-------------|-------------|-------------|--------------|-------------------|
| Oncology | 5.2% | 15.5% | 59.2% | 1.40× | 1.25× |
| Neurology | 6.1% | 13.2% | 54.8% | 1.40× | 1.25× |
| Cardiovascular | 8.3% | 18.1% | 62.5% | 1.40× | 1.25× |
| Metabolic | 9.1% | 20.3% | 65.1% | 1.40× | 1.25× |
| Infectious Disease | 11.5% | 22.8% | 67.3% | 1.40× | 1.25× |
| Respiratory | 7.2% | 16.7% | 58.9% | 1.40× | 1.25× |
| Autoimmune | 7.8% | 17.2% | 56.1% | 1.40× | 1.25× |
| Hematology | 10.2% | 21.5% | 64.8% | 1.40× | 1.25× |
| Rare Disease | 12.5% | 25.3% | 68.7% | built-in | 1.25× |
| Mental Health | 5.8% | 12.1% | 51.2% | 1.40× | 1.25× |
| Other | 8.0% | 17.0% | 60.0% | 1.40× | 1.25× |

### Expected New Approvals Calculation

For each disease:
```
expected_approvals = sum over each pipeline drug:
    base_rate = area_rate[drug.phase]
    if drug.orphan: base_rate *= orphan_boost
    if drug.breakthrough: base_rate *= breakthrough_boost
    if drug.moa_novelty == 'novel': base_rate *= 0.70  # novel MOA penalty

expected_new_approvals = sum(adjusted_rates)
```

This gives a probabilistic estimate of how many current pipeline drugs will actually reach market, which is more informative than raw pipeline count.

## Specialty Segment Analysis

Diseases are tagged with a specialty segment that enables filtered views:

- **Pediatric**: Primarily affects children (<18). Often has fewer drugs due to trial enrollment challenges and ethical constraints. Systematic white space.
- **Geriatric**: Primarily affects elderly (65+). Polypharmacy concerns, comorbidities, and dosing differences create drug development complexity.
- **Rare**: <200,000 US patients. Eligible for orphan drug designation. Often strong regulatory tailwinds but small commercial markets.
- **General**: Broad adult population. Largest commercial markets but often most competitive.

## Historical Trend Tracking

Each quarterly refresh appends a snapshot to the Historical Trends tab containing:
- Disease name, date, approved drug count, pipeline count, composite score, underserved ratio
- This enables year-over-year trend analysis: "Is R&D investment accelerating or declining for this disease?"

Trend flags:
- **Heating up**: Composite score dropped >1 point (more drugs entering = less white space)
- **Cooling down**: Pipeline drugs terminated without replacement
- **Stable**: Score within ±0.5 points
- **Breakthrough event**: New approval or regulatory designation

## Normalization Functions (unchanged from v1)

```python
import math

def normalize_log(value, min_val, max_val):
    if value <= min_val: return 0.0
    if value >= max_val: return 1.0
    return math.log(value / min_val) / math.log(max_val / min_val)

def clamp(value, low=0.0, high=10.0):
    return max(low, min(high, value))
```
