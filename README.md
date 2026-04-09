# 🌐 AI Supremacy Index — Data Processing Pipeline

> *Complete documentation of how 5 ETO datasets containing 31 source files were cleaned, combined, harmonized, and transformed into 8 analysis-ready master datasets feeding the AISI scoring pipeline.*

---

```
5 Raw Datasets → 31 CSV Files → Clean & Combine → Harmonize Names → 8 Master Files → 7 Dimensions → AISI Score
```

---

## 📊 Pipeline at a Glance

| Stat | Value |
|------|-------|
| 📁 Source Files | **31** across 5 datasets |
| 📤 Master Outputs | **8** analysis-ready CSVs |
| 🗂️ Total Rows Processed | **~50,000** |
| 🌍 Countries (after harmonization) | **195** |
| 📐 Dimensions Scored per Country | **7** |

---

## 📦 Datasets

### 1 · Advanced Semiconductor Supply Chain
`5 source files → semiconductor_master.csv (1,305 rows × 16 columns)`

| Metric | Value |
|--------|-------|
| Unique Inputs (tools, materials, processes) | 126 |
| Organizations across 21 countries | 374 |
| Master rows (provider–input pairs) | 1,305 |
| Inputs with market share data | 96 |

**Source Files**

| File | Rows | Cols | Role |
|------|------|------|------|
| `inputs.csv` | 126 | 10 | Catalog of chip production inputs (tools 90, materials 17, processes 11, designs 7) |
| `providers.csv` | 397 | 5 | Countries + organizations (includes aliases → 374 unique after dedup) |
| `provision.csv` | 1,305 | 7 | Core linkage: which providers supply which inputs, with market share % |
| `sequence.csv` | 139 | 6 | Supply chain relationships: 53 "goes into" + 86 "is type of" |
| `stages.csv` | 3 | 6 | Three production stages: Design, Fabrication, ATP |

**Processing Pipeline**

```
Step 1 — Deduplicate Providers     providers.csv: 397 → 374 unique (drop aliases on provider_id)
Step 2 — Join Provision + Providers  Left join on provider_id → adds provider_type, provider_hq_country
Step 3 — Join Input Metadata        Left join on provided_id → adds input_type, stage_id, market_size
Step 4 — Join Stage Metadata        Left join on stage_id → adds production_stage (Design/Fab/ATP)
Step 5 — Derive Effective Country   country-type → provider_name; orgs → provider_hq_country
Step 6 — Final Cleanup & Reorder    Drop redundant cols, rename for clarity, reorder to 16 columns
```

**Quality Checks**
- ✅ Market share sums: country-level shares per input average ~100% (0 outliers outside 90–110%)
- ⚠️ "Various" entries: aggregated small providers (<1% share each) — excluded from country-level analysis
- ⚠️ Missing shares: 30 inputs have no market share data → only **96 inputs** used for breadth denominator
- 📌 HQ Attribution: all country assignments are HQ-based (e.g., TSMC → Taiwan even with US fabs)

**Output Schema**
```
semiconductor_master.csv — 1,305 rows × 16 columns
├── provider_name, provider_id, provider_type, provider_hq_country, effective_country
├── input_name, input_id, input_name_full, input_type
├── production_stage, stage_id, input_data_year
├── share_provided, provision_year
└── input_market_size, source
```

---

### 2 · Cross-Border Tech Research Collaborations
`8 field-specific CSVs → collaboration_master.csv (21,118 rows × 8 columns)`

| Metric | Value |
|--------|-------|
| Source Files | 8 (one per research field) |
| Master Rows (country-pair × field × year) | 21,118 |
| Countries covered | 94 |
| Year range (complete through) | 2015–2023 |
| Minimum article threshold (per pair/year) | ≥25 |

**Source Files by Field**

| File | Rows | Field | Note |
|------|------|-------|------|
| `Artificial_intelligence.csv` | 10,569 | AI (general) | Largest — broadest coverage |
| `Computer_vision.csv` | 3,335 | Computer Vision | Mature subfield |
| `Chip_design_and_fabrication.csv` | 2,418 | Chip Design | Strategically weighted **2.0×** |
| `Cybersecurity.csv` | 1,709 | Cybersecurity | National security relevance |
| `Robotics.csv` | 1,561 | Robotics | Mature subfield |
| `Natural_language_processing.csv` | 921 | NLP | Broad applied AI |
| `Large_language_models.csv` | 432 | LLMs | Frontier AI, weighted **1.8×** |
| `AI_safety.csv` | 181 | AI Safety | Smallest — emerging field, weighted **1.6×** |

**Processing Pipeline**

```
Step 1 — Concatenate          Stack all 8 CSVs vertically (identical schema)
Step 2 — Standardize Pairs    Alphabetical ordering → (US, China) == (China, US)
Step 3 — Completeness Filter  Only complete=True rows; exclude 2024–2025 (incomplete)
Step 4 — Validation           Consistent country names, no duplicate pairs per field/year
```

**Key Data Characteristics**
- 🇺🇸🇨🇳 US–China axis dominates: 197K co-authored articles — 2.4× the next largest pair
- 📈 Growth trajectory: 39K articles in 2015 → 174K in 2023 (+342% overall)
- 🌐 Partner diversity: mean 19.1 partners, median 12, 95th percentile = 54 (used as cap in AISI)

---

### 3 · Country AI Activity Metrics
`9 CSVs → 4 master datasets (unified + 3 pillar-level)`

| Metric | Value |
|--------|-------|
| Source Files | 9 (3 publication + 2 patent + 4 investment) |
| Countries (after excluding groups) | 189 |
| Raw entities (including OECD, NATO, EU…) | 204 |
| Group entities excluded | 12 |

**Source Files**

| File | Rows | Pillar | Key Metrics |
|------|------|--------|-------------|
| `publications_yearly_articles.csv` | 8,983 | PUBLICATIONS | `num_articles` by country × field × year |
| `publications_yearly_citations.csv` | ~8K | PUBLICATIONS | `num_citations` (no complete flag — lagged) |
| `publications_yearly_highly_cited.csv` | ~8K | PUBLICATIONS | Highly cited article counts |
| `patents_yearly_applications.csv` | 5,832 | PATENTS | `num_patent_applications` (complete thru 2020) |
| `patents_yearly_grants.csv` | ~5K | PATENTS | Granted patents (**removed** — grant rate bias) |
| `companies_yearly_disclosed.csv` | ~12K | INVESTMENT | Disclosed investment ($M) |
| `companies_yearly_estimated.csv` | 15,339 | INVESTMENT | Estimated investment ($M) — **used for scoring** |
| `companies_yearly_num_transactions.csv` | ~12K | INVESTMENT | Transaction counts |
| `companies_yearly_num_companies.csv` | ~12K | INVESTMENT | Active company counts |

**Processing Pipeline**

```
Step 1 — Exclude Groups        Remove 12 aggregate entities (ASEAN, EU, NATO…): 204 → 192
Step 2 — Harmonize China       "China (mainland)" → canonical "China" via lookup table
Step 3 — Completeness Filter   Publications: 2015–2023 | Patents: 2015–2020 | Investment: 2015–2024
Step 4 — Build Pillar Masters  Aggregate by country per pillar across complete years
Step 5 — Build Unified Master  Join all 3 pillars on country name; set boolean availability flags
```

**Critical Data Findings**
- 📉 Extreme skewness: Publications skew=8.3, Patents skew=6.9, Investment skew=10.4 → `log(1+x)` reduces all to <1.0
- ⚠️ Patent lag: complete patent data ends at **2020** — Innovation Output measures pre-pandemic activity only
- 🔍 Coverage varies: 192 countries have publications, only 73 have patents, 117 have investment; **65 countries have all 3**

**Output Files**
```
country_ai_unified_master.csv        — 189 rows × 10 cols (one row per country, all pillars)
country_ai_publications_master.csv   — 8,983 rows (yearly articles by country × field)
country_ai_patents_master.csv        — 5,832 rows (yearly applications by country × field)
country_ai_investment_master.csv     — 15,339 rows (yearly estimated investment by country × field)
```

---

### 4 · Private-Sector AI Indicators (PARAT)
`5 source files → parat_master.csv + parat_country_agg.csv`

| Metric | Value |
|--------|-------|
| Companies (startups to multinationals) | 691 |
| Countries represented | 17 |
| US-based companies | 85% |
| Core columns | 63 |
| Master columns (after enrichment) | 71 |

**Source Files**

| File | Rows | Role |
|------|------|------|
| `core.csv` | 691 | Main metrics: AI pubs, patents, workforce, company metadata (HQ, stage, sector) |
| `yearly_publication_counts.csv` | ~6K | Disaggregated yearly publication & patent data per company |
| `alias.csv` | varies | Alternate company names for matching |
| `ticker.csv` | varies | Stock exchange symbols for public companies |
| `id.csv` | varies | Cross-references: LinkedIn, Crunchbase, ROR, PermID |

**Processing Pipeline**

```
Step 1 — Load & Merge         Join core + yearly_publication_counts on company ID; enrich with alias/ticker
Step 2 — Flag Key Attributes  Identify S&P 500, Global Big Tech, GenAI Contenders; flag zero-pub/patent companies
Step 3 — Country Aggregation  Group by HQ country → company_count, total AI pubs/patents/workers, etc.
Step 4 — Talent Tier          Tier 1: ≥3 companies with workforce data | Tier 2: has AI pubs | Tier 3: count only
```

**Known Limitations**
- 🇺🇸 US dominance: 85% of companies are US-based; LinkedIn workforce data is essentially US-only
- 📋 Curated, not exhaustive: 691 selected companies — biased toward well-known firms
- 🏢 Maturity skew: 93% of companies at "mature" stage; limited startup representation

---

### 5 · AGORA — AI Governance Archive
`4 source files → agora_master.csv (NLP stance classification)`

| Metric | Value |
|--------|-------|
| Documents (laws, regulations, standards) | 973 |
| Taxonomy Tags across 5 domains | 77 |
| US-focused documents | 91% (882 of 973) |
| Scoreable countries | 2 (US: 390 enacted, China: 18) |

**Source Files**

| File | Rows | Role |
|------|------|------|
| `documents.csv` | 973 | Core metadata: authority, status, dates, summaries, 77 tag columns |
| `segments.csv` | 8,116 | Sub-document segments with granular annotations |
| `authorities.csv` | 105 | Issuing bodies with jurisdiction and parent authority |
| `collections.csv` | 10 | Thematic groupings |

**Processing Pipeline**

```
Step 1 — Merge Documents + Authorities   Left join on Authority = Name → adds Jurisdiction field
Step 2 — Map to Countries               Sub-national (California, NYC) → "United States"
Step 3 — NLP Stance Classification      Dual-signal approach:
                                          • Tag-based scoring (2× weight): 77 taxonomy tags → enabling vs restrictive
                                          • Keyword analysis (1× weight): summary scan ("promote" → enabling, "restrict" → restrictive)
Step 4 — Boolean Tag Normalization      Convert 77 tag cols to proper booleans (TRUE/"True"/1)
Step 5 — Governance Metrics            enacted_docs (log), thematic_breadth (/77), maturity_ratio
```

**Stance Classification Results**

| Stance | Count |
|--------|-------|
| 🟢 Enabling | 346 |
| 🔴 Restrictive | 293 |
| ⚪ Neutral | 191 |
| 🔵 Balanced | 137 |
| ❔ Unclassified | 6 |

**Document Status Breakdown:** 450 Enacted · 400 Defunct · 123 Proposed

---

## 🌍 Country Name Harmonization

Each dataset uses different country naming conventions — ISO 3166 codes (semiconductor), full names with variants (Country AI Activity), and standard full names (cross-border, PARAT, AGORA). Without explicit harmonization, cross-dataset joins fail silently and countries lose data across dimensions.

**Key Mappings**

| Canonical Name | Semiconductor | Cross-Border | Country AI | PARAT | AGORA |
|---------------|---------------|--------------|------------|-------|-------|
| China | CHN | China | China (mainland) | China | China |
| United States | USA | United States | United States | United States | United States |
| South Korea | KOR | South Korea | South Korea | South Korea | — |
| Taiwan | TWN | Taiwan | Taiwan | Taiwan | — |
| Japan | JPN | Japan | Japan | Japan | — |
| Germany | DEU | Germany | Germany | Germany | — |
| Netherlands | NLD | Netherlands | Netherlands | Netherlands | — |
| United Kingdom | GBR | United Kingdom | United Kingdom | United Kingdom | — |

**Harmonization Function**

```python
def harmonize_country(name):
    if name in GROUP_ENTITIES:        return None              # Exclude 12 group entities
    if name in ISO_TO_NAME:           return ISO_TO_NAME[name] # CHN → China
    if name in VARIANT_TO_CANONICAL:  return VARIANT_TO_CANONICAL[name] # "China (mainland)" → China
    if name == 'Various countries':   return None              # Semiconductor aggregates
    return name                                                # Already canonical
```

**Group Entities Excluded (12)**

`ASEAN` · `Africa` · `Asia` · `EU` · `Europe` · `Five Eyes` · `GPAI` · `NATO` · `N. America & Caribbean` · `OECD` · `Oceania` · `Quad`

---

## 🏆 AISI Scoring Pipeline

### Dimension Weights (v3)

| Dimension | Weight | Coverage | Sub-Components | Data Source |
|-----------|--------|----------|----------------|-------------|
| 🖥️ Hardware Sovereignty | **22%** | ~23 | Avg share (35%) + Peak share (30%) + Breadth/96 (35%) | SEMICONDUCTOR |
| 🔬 Research Capacity | **20%** | ~192 | Log pub volume (35%) + Lagged citations (45%) + Z-scored growth (20%) | COUNTRY AI |
| 💼 Commercial Ecosystem | **20%** | ~117 | Log investment (50%) + Log company count (30%) + Pub intensity (20%) | COUNTRY AI + PARAT |
| 💡 Innovation Output | **18%** | ~73 | Log patent volume (45%) + Field diversity /11 (55%) | COUNTRY AI |
| 🧠 Talent Base | **10%** | 17 | Tiered fallback: Workforce → Publications → Company count | PARAT |
| 🤝 Collaboration Network | **5%** | ~94 | Partner div capped@54 (40%) + Strategic volume (35%) + Partner quality (25%) | CROSS-BORDER |
| ⚖️ Governance Readiness | **5%** | 2 | Log enacted docs (30%) + Breadth/77 (40%) + Maturity ratio (30%) | AGORA |

### Log Transform Impact

| Metric | Raw Skew | Log Skew | Top/Median (Raw) | Top/Median (Log) |
|--------|----------|----------|------------------|------------------|
| Publication volume | 8.3 | 0.2 | 1,127× | 2.6× |
| Patent volume | 6.9 | 0.8 | 4,987× | 3.4× |
| Estimated investment | 10.4 | 0.3 | 6,741× | 2.8× |

### Coverage-Weighted Composite Formula

```
AISI Score = Σ (dimension_score × dimension_weight × dimension_coverage)
             ─────────────────────────────────────────────────────────────
                     Σ (dimension_weight × dimension_coverage)

           + Breadth factor: 0.75 + 0.25 × (dims_scored / 7)
             → Penalizes countries scoring on fewer dimensions
```

### 🐛 Bug Fixes Applied

| Bug | Problem | Fix |
|-----|---------|-----|
| **Bug #1 — NaN ≠ Zero** | Countries absent from semiconductor/collaboration/governance datasets treated as NaN (missing), not zero (genuine absence). Coverage-weighted formula *rewarded* them with higher scores for fewer dimensions. | Fill genuine-zero dimensions with `0` before scoring |
| **Bug #2 — Breadth Gaming** | Countries scoring on only 2–3 dimensions could outscore broader profiles because the coverage denominator was smaller. | Added breadth factor `0.75 + 0.25 × (dims/7)` to penalize shallow coverage |

### Final Output Files

```
AISI_Final_Rankings.csv                    — All ~195 countries, 7 dimensions + composite
AISI_Final_Rankings_v2_no_governance.csv   — 6-dimension variant (governance excluded)
AISI_High_Confidence_Rankings.csv          — Filtered to ≥50% data coverage only
```

---

## 📁 Repository Structure

```
.
├── data/
│   ├── raw/
│   │   ├── semiconductor/          # 5 source CSVs
│   │   ├── collaboration/          # 8 field-specific CSVs
│   │   ├── country_ai/             # 9 pillar CSVs
│   │   ├── parat/                  # 5 company CSVs
│   │   └── agora/                  # 4 governance CSVs
│   └── processed/
│       ├── semiconductor_master.csv
│       ├── collaboration_master.csv
│       ├── country_ai_unified_master.csv
│       ├── country_ai_publications_master.csv
│       ├── country_ai_patents_master.csv
│       ├── country_ai_investment_master.csv
│       ├── parat_master.csv
│       ├── parat_country_agg.csv
│       └── agora_master.csv
├── outputs/
│   ├── AISI_Final_Rankings.csv
│   ├── AISI_Final_Rankings_v2_no_governance.csv
│   └── AISI_High_Confidence_Rankings.csv
├── pipeline/
│   ├── harmonize.py                # Country name harmonization
│   ├── score.py                    # AISI scoring pipeline
│   └── process_*.py                # Per-dataset processing scripts
└── README.md
```

---

## 🔗 Links

- 🌍 [Live Site — Who's Got AI?](https://aisi-bda600.vercel.app)
- 🥇 [AISI Index Dashbaord](https://aisi-bda600.vercel.app/aisi-index)
- 📊 [Analysis](https://aisi-bda600.vercel.app/analysis)
- 🔍 [SWOT Analysis](https://aisi-bda600.vercel.app/swot-analysis)
- 👥 [Our Team](https://aisi-bda600.vercel.app/our-team)

---

*© 2026 AISI Project. All rights reserved.*
