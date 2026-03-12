# Database Schema v2.1: drug_whitespace_database.xlsx

## Tab 1: Disease Epidemiology

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| disease_name | str | Common disease name | Type 2 Diabetes |
| icd10_code | str | Primary ICD-10 code | E11 |
| icd10_chapter | str | WHO ICD-10 chapter (Roman numeral, auto-populated) | IV |
| icd10_chapter_name | str | ICD-10 chapter name (auto-populated) | Endocrine, Nutritional & Metabolic |
| therapeutic_area | str | Broad category | Endocrine/Metabolic |
| specialty_segment | str | pediatric, geriatric, rare, general | general |
| us_prevalence | int | Current US patients | 37000000 |
| global_prevalence | int | Current global patients | 462000000 |
| us_annual_incidence | int | New US cases/year | 1500000 |
| mortality_rate_per_100k | float | US deaths per 100k/year | 25.7 |
| severity_category | str | life-threatening, debilitating, chronic-manageable, acute-treatable | chronic-manageable |
| primary_determinants | str | Comma-separated | genetic, lifestyle, environmental |
| age_group_affected | str | Primary affected age group | adults 45+ |
| annual_cost_per_patient_usd | float | Avg annual treatment cost per patient | 16750.0 |
| total_us_economic_burden_usd | float | Total US economic burden (direct + indirect) | 327000000000 |
| qaly_burden | float | QALYs lost per patient per year (disease burden) | 0.15 |
| data_source | str | Where data came from | CDC NCHS 2024 |
| source_url | str | URL to source | https://www.cdc.gov/... |
| date_pulled | date | When this row was last updated | 2026-03-08 |

## Tab 2: Approved Drugs

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| disease_name | str | Maps to Epidemiology tab | Type 2 Diabetes |
| drug_brand_name | str | Brand name | Ozempic |
| drug_generic_name | str | Generic/INN name | semaglutide |
| mechanism_of_action | str | MOA category | GLP-1 receptor agonist |
| moa_novelty | str | established, validated, emerging, novel | established |
| approval_year | int | FDA approval year | 2017 |
| manufacturer | str | Company name | Novo Nordisk |
| route_of_admin | str | Delivery route | subcutaneous injection |
| orphan_drug | str | yes/no | no |
| breakthrough_therapy | str | yes/no | no |
| accelerated_approval | str | yes/no | no |
| annual_cost_usd | float | Annual wholesale cost per patient | 12000.0 |
| data_source | str | Source reference | FDA Orange Book |
| date_pulled | date | When this row was last updated | 2026-03-08 |

## Tab 3: Pipeline Drugs (Clinical Trials)

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| disease_name | str | Maps to Epidemiology tab | Alzheimer's Disease |
| nct_id | str | ClinicalTrials.gov ID | NCT04567890 |
| intervention_name | str | Drug/treatment name | lecanemab |
| trial_phase | str | PHASE1, PHASE2, PHASE3, PHASE4 | PHASE3 |
| trial_status | str | Current recruitment status | RECRUITING |
| sponsor | str | Lead sponsor | Eisai |
| enrollment_target | int | Planned enrollment | 1800 |
| mechanism_of_action | str | If known | anti-amyloid antibody |
| orphan_drug | str | yes/no/unknown | no |
| breakthrough_designation | str | yes/no/unknown | yes |
| fast_track | str | yes/no/unknown | no |
| start_date | date | Trial start date | 2024-06-15 |
| estimated_completion | date | Expected primary completion | 2027-12-01 |
| date_pulled | date | When this row was last updated | 2026-03-08 |

## Tab 4: Basic Science Activity

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| disease_name | str | Maps to Epidemiology tab | Pancreatic Cancer |
| activity_type | str | publication, grant, investment, program | grant |
| title | str | Title of article/grant/program | DARPA PREPARE Program |
| source_org | str | Funding/publishing org | DARPA |
| description | str | Brief summary | Novel immunotherapy targets |
| funding_amount_usd | float | If known | 50000000 |
| pmid | str | PubMed ID if applicable | 38123456 |
| doi | str | DOI if applicable | 10.1038/... |
| publication_date | date | When published/announced | 2025-09-01 |
| date_pulled | date | When this row was last updated | 2026-03-08 |

## Tab 5: White Space Scores

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| disease_name | str | Maps to Epidemiology tab | Pancreatic Cancer |
| therapeutic_area | str | Broad category | Oncology |
| specialty_segment | str | pediatric, geriatric, rare, general | general |
| us_prevalence | int | From epidemiology | 62000 |
| approved_drug_count | int | Count from Approved Drugs tab | 4 |
| pipeline_drug_count | int | Count from Pipeline tab (active only) | 87 |
| weighted_pipeline_count | float | Phase-weighted | 32.4 |
| approval_probability_avg | float | Avg historical approval probability for this area | 0.12 |
| expected_new_approvals | float | weighted_pipeline × avg_approval_probability | 3.9 |
| recent_publications | int | Count from Basic Science tab (last 3 years) | 156 |
| regulatory_advantage_score | float | 0-10: orphan + breakthrough + fast-track density | 6.5 |
| health_econ_score | float | 0-10: cost burden vs treatment availability | 7.2 |
| unmet_need_score | float | 0-10 scale | 8.2 |
| drug_coverage_gap | float | 0-10 scale | 3.1 |
| scientific_complexity | float | 0-10 scale | 7.5 |
| composite_whitespace_score | float | Weighted combination | 6.27 |
| underserved_ratio | float | Prevalence / (approved + weighted pipeline) | 1703.3 |
| score_date | date | When scores were computed | 2026-03-08 |

## Tab 6: Historical Trends

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| disease_name | str | Maps to Epidemiology tab | Alzheimer's Disease |
| snapshot_date | date | When this snapshot was taken | 2026-03-08 |
| approved_drug_count | int | Drugs at time of snapshot | 7 |
| pipeline_drug_count | int | Active pipeline at snapshot | 142 |
| composite_whitespace_score | float | Score at snapshot | 7.3 |
| underserved_ratio | float | Ratio at snapshot | 456789.2 |
| us_prevalence | int | Prevalence at snapshot | 6900000 |
| notes | str | Notable events during this period | Lecanemab Phase 3 results |

This tab tracks how each disease's landscape changes over time. Each quarterly refresh appends a new snapshot row per disease. Useful for spotting which diseases are getting more or less attention.

## Tab 7: Approval Probability Reference

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| therapeutic_area | str | Broad category | Oncology |
| phase_1_to_approval | float | Historical P1→approval rate | 0.052 |
| phase_2_to_approval | float | Historical P2→approval rate | 0.155 |
| phase_3_to_approval | float | Historical P3→approval rate | 0.592 |
| orphan_boost | float | Multiplicative boost for orphan drugs | 1.40 |
| breakthrough_boost | float | Boost for breakthrough designation | 1.25 |
| novel_moa_penalty | float | Penalty for novel/unvalidated MOA | 0.70 |
| data_source | str | Source of probability estimates | BIO/QLS 2024 Industry Report |

Historical approval probability rates by therapeutic area. Used by the scoring algorithm to estimate how many pipeline drugs will actually reach market.

## Tab 8: Metadata

| Column | Type | Description |
|--------|------|-------------|
| parameter | str | Database parameter name |
| value | str | Parameter value |

Tracked parameters:
- database_created: Timestamp of initial creation
- last_full_update: Timestamp of last comprehensive refresh
- total_diseases: Count of diseases in database
- total_approved_drugs: Count of approved drugs
- total_pipeline_drugs: Count of pipeline entries
- scoring_version: Version (now 2.0)
- schema_version: 2.0
- notes: Free-text notes about the database state
- last_quarterly_refresh: Timestamp of last scheduled refresh
