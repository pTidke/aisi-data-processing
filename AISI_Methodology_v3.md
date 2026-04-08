# AISI — AI Supremacy Index: Methodology (v3 — Post-Data Audit)

## What Is It?

The **AI Supremacy Index (AISI)** is a composite score that measures how structurally resilient a country is in the global AI landscape — not just how active it is today, but how defensible and self-sustaining its AI position is against geopolitical, supply chain, and competitive shocks.

---

## Data Sources

Five datasets feed the index:

| Dataset | What It Captures | Scoreable Countries |
|---|---|---|
| **Country AI Activity Metrics** | Annual publications, citations, patent applications & grants, and estimated investment by country and research field | ~192 (pubs), ~73 (patents), ~117 (investment) |
| **Advanced Semiconductor Supply Chain** | Provider market shares across semiconductor inputs and fabrication stages | ~23 |
| **Cross-Border Tech Research Metrics** | Co-authored research papers between country pairs, broken down by topic and year | ~94 |
| **Private-Sector AI Indicators (PARAT)** | Corporate AI activity — company counts, AI publication output, and workforce figures | 17 |
| **AGORA** | AI governance documents — laws, regulations, standards, and their thematic coverage | 2 (US, China) |

---

## Country Name Harmonization

The five source datasets use inconsistent country naming. Before any scoring, all datasets are harmonized to a single canonical name per country using an explicit lookup table. Key mappings include:

| Canonical Name | Semiconductor | Cross-Border | Country AI Activity | PARAT | AGORA |
|---|---|---|---|---|---|
| China | `CHN` | `China` | `China (mainland)` | `China` | `China` |
| United States | `USA` | `United States` | `United States` | `United States` | `United States` |
| South Korea | `KOR` | `South Korea` | `South Korea` | `South Korea` | — |
| Taiwan | `TWN` | `Taiwan` | `Taiwan` | `Taiwan` | — |
| Japan | `JPN` | `Japan` | `Japan` | `Japan` | — |
| Germany | `DEU` | `Germany` | `Germany` | `Germany` | — |
| United Kingdom | `GBR` | `United Kingdom` | `United Kingdom` | `United Kingdom` | — |
| Netherlands | `NLD` | `Netherlands` | `Netherlands` | `Netherlands` | — |

The full lookup table covers all countries present in any dataset, mapping ISO 3166 three-letter codes (semiconductor data) and variant name strings to canonical names. Countries that cannot be matched are flagged in the output with `harmonization_warning = True` for analyst review.

### Group Entity Exclusion

The Country AI Activity dataset includes 12 aggregate entities that are not individual countries: ASEAN, Africa, Asia, EU, Europe, Five Eyes, Global Partnership on Artificial Intelligence, NATO, North America and the Caribbean, OECD, Oceania, and Quad. These are excluded from all AISI scoring. They are identified and removed before any metric computation.

---

## The Seven Dimensions

The index is built from seven dimensions, each capturing a distinct aspect of resilience. They are calculated independently and then combined.

### 1. Research Capacity (20%)

Measures the depth and momentum of a country's scientific AI output. Three sub-components are combined with explicit weights:

- **Publication volume** (35%) — average annual AI articles over the most recent three complete years of data (filtering on `complete = true`; currently 2021–2023). Log-transformed (log(1 + x)) before normalisation. Raw data skewness is 8.3; after log transform, skewness drops to 0.2.
- **Citation impact** (45%) — citations per article, averaged over a three-year lagged window: **2021–2023** (years max-4 to max-2, where max citation year = 2025). This lag is deliberate: the Country AI Activity dataset warns that citation data is incomplete in recent years and does not yet include a `complete` flag for citations. Using a lagged window ensures the metric reflects genuine research quality rather than data pipeline latency.
- **Growth trend** (20%) — five-year publication trajectory, standardised using a z-score across all countries so that fast-growing smaller nations are not unfairly penalised against large incumbents.

Countries with fewer than five years of data are scored only on the available sub-components, with weights redistributed accordingly.

**Coverage:** ~192 countries have publication data. This is the broadest-coverage dimension.

---

### 2. Innovation Output (18%)

Measures the translation of research into protected intellectual property across diverse fields:

- **Patent volume** (45%) — total AI patent applications filed across all complete years. Log-transformed before normalisation. Raw skewness is 6.9; after log transform, 0.8.
- **Field diversity** (55%) — number of distinct patent thematic fields (out of 11 non-"All" fields) with active patent activity. Only fields with at least one patent application count, preventing inflated scores from data artifacts.

**Why grant rate was removed:** Grant rates are heavily confounded by patent office behaviour — the Chinese patent office historically grants at rates above 90%, while the USPTO and EPO are far stricter. A high grant rate in one jurisdiction reflects a permissive examiner, not better filings. Since the dataset attributes patents to the filing country (not the inventor's country), this confound cannot be corrected. Field diversity is a more robust quality proxy.

**Data lag warning:** Complete patent data currently covers only **2015–2020** — a 5-year lag. The Innovation Output dimension therefore measures pre-pandemic patenting activity. Countries that significantly scaled AI patenting after 2020 (e.g., through national AI strategies) receive no credit for that growth. This lag is flagged in the output with `innovation_output_latest_year = 2020`.

**Coverage:** ~73 countries have patent data.

---

### 3. Commercial Ecosystem (20%)

Measures the private-sector depth behind a country's AI position:

- **Investment** (50%) — total estimated AI investment (in USD millions), using the `companies_yearly_estimated` table rather than disclosed-only values. The dataset provides estimated investment that accounts for undisclosed deal values using median imputation by investment stage, target country, and year. Disclosed-only data systematically undercounts countries where deal confidentiality norms are stronger. Log-transformed before normalisation. Raw skewness is 10.4; after log transform, 0.3.
- **Company count** (30%) — number of active AI companies per PARAT, log-transformed. Only available for the 17 countries with PARAT coverage; for countries without PARAT data, this sub-component's weight is redistributed to Investment.
- **Publication intensity** (20%) — average AI publications per company (total PARAT AI publications ÷ company count), reflecting a research-active corporate culture rather than just corporate presence. Not log-transformed, as this ratio is already naturally bounded. Same PARAT-dependent coverage caveat applies.

**Coverage:** ~117 countries have investment data. PARAT-dependent sub-components (company count, publication intensity) cover only 17 countries; weights are redistributed for the rest.

---

### 4. Hardware Sovereignty (22%)

The highest-weighted dimension, reflecting that semiconductor access is the primary geopolitical chokepoint in AI. It measures a country's foothold in the global semiconductor supply chain:

- **Average market share** (35%) — mean market share across all semiconductor inputs where the country has a provider presence.
- **Peak market share** (30%) — the country's strongest position in any single critical input. This captures dominant positions in chokepoint inputs (e.g., EUV lithography tools — 100% Netherlands).
- **Supply chain breadth** (35%) — the fraction of semiconductor inputs where the country has any provider presence. **The denominator is 96** (the number of inputs with at least one market share data point), not 126 (total inputs in the dataset). Using 126 would penalise countries for not participating in inputs where no share data exists for any country.

Countries with no mapped semiconductor provider presence score zero. Country names are matched using the harmonization lookup table (ISO 3166 codes → canonical names). Unmatched countries are flagged for analyst review.

**Note on HQ-based attribution:** the semiconductor dataset assigns country affiliation based on the ultimate parent company's headquarters location, not physical production location. TSMC's market share counts toward Taiwan even though it operates fabs in the US and Japan. Countries where HQ-based attribution is likely distortive are flagged in the output.

**Coverage:** ~23 countries have semiconductor provider data. The remaining ~170+ countries score zero on this dimension, and the coverage-weighted composite adjusts accordingly.

---

### 5. Collaboration Network (5%)

Measures the breadth and strategic quality of a country's international research partnerships:

- **Partner diversity** (40%) — number of unique countries collaborated with, capped at **54** (the empirical 95th percentile of partner counts; only the US at 79, China, UK, Germany, and India exceed this). The cap prevents the sub-component from being dominated by a handful of hyper-connected countries while preserving meaningful variation below the threshold. The cap value is recomputed per run and reported in the output.
- **Weighted volume** (35%) — total co-authored articles, weighted by the strategic importance of the research topic:

  | Topic | Multiplier | Rationale |
  |---|---|---|
  | Chip design and fabrication | 2.0× | Direct hardware relevance, highest geopolitical sensitivity |
  | Large language models | 1.8× | Frontier AI capability |
  | AI safety | 1.6× | Governance and alignment signal |
  | Cybersecurity | 1.4× | National security relevance |
  | NLP | 1.2× | Broad applied AI |
  | AI (general) | 1.2× | Baseline |
  | Computer vision | 1.0× | Mature subfield |
  | Robotics | 1.0× | Mature subfield |

- **Partner quality** (25%) — collaboration volume weighted by each partner country's own Research Capacity score. This creates a one-step dependency: Research Capacity scores are computed first, then used here. Collaborating with the US or China adds more resilience signal than collaborating with a minimal-output country.

**Data note:** Only complete years (`complete = true`, currently 2015–2023) are used. Country pairs with fewer than 25 joint publications per year are already excluded by the source dataset. The US–China axis dominates with 197K co-authored articles — 2.4× the next pair.

**Coverage:** ~94 countries appear in cross-border collaboration data.

---

### 6. Talent Base (10%)

Measures the human capital underpinning AI activity, using a tiered fallback approach depending on data availability:

- **Tier 1** — AI workforce headcount from PARAT (highest confidence). Used when the country has at least **3** PARAT companies with non-zero workforce data. (Threshold lowered from 5 to 3 based on data audit: with only 17 PARAT countries total, a threshold of 5 effectively restricts Tier 1 to the US alone.)
- **Tier 2** — Total AI publications from PARAT companies, as a proxy for active research talent. Used when Tier 1 data is insufficient but the country has PARAT companies with publication data.
- **Tier 3** — Company count from PARAT (lowest confidence, flagged in output). Used as last resort for the 17 PARAT countries.

For countries with no PARAT presence (175+ countries), this dimension has zero coverage and the coverage-weighted composite excludes it from their score entirely.

The tier used is recorded alongside the score so analysts know which countries have reliable talent data.

**Weight rationale:** talent at 10% reflects its importance for resilience (human capital is harder to sanction than chips or capital) while acknowledging severe data limitations. PARAT workforce data is derived from LinkedIn, which is blocked in China and Russia and far less popular outside the US. The coverage-weighted composite automatically downweights this dimension for countries with poor or absent data.

**Coverage:** 17 countries via PARAT. Effectively US-only for Tier 1 workforce data.

---

### 7. Governance Readiness (5%)

Measures the maturity and breadth of a country's AI governance framework. This dimension is derived from AGORA.

- **Document count** (30%) — number of enacted AI governance documents for the country (filtering on `Most recent activity = "Enacted"`). Log-transformed before normalisation.
- **Thematic breadth** (40%) — number of distinct AGORA taxonomy tags covered across all of the country's enacted documents (out of 77 total tags spanning risk factors, harms, governance strategies, incentives, and application domains). This rewards countries whose governance covers more of the AI policy landscape.
- **Maturity ratio** (30%) — ratio of enacted documents to total documents (enacted + proposed + defunct). A higher ratio indicates a governance pipeline that converts proposals into law.

**Data reality:** AGORA currently contains **973 documents** but is overwhelmingly US-focused:
- United States: 390 enacted documents, broad thematic coverage
- China: 18 enacted documents, moderate coverage
- Multinational: 17 enacted documents (cannot be attributed to specific countries)
- Other countries: 10 enacted documents (aggregated, not individually mappable)

This means only **2 countries** (US and China) are individually scoreable on this dimension. "Multinational" and "Other countries" cannot be mapped to specific nations and are excluded from country-level scoring.

**Weight rationale:** reduced to 5% (from 9% in v2) because the effective country coverage is minimal. The coverage-weighted composite ensures that the ~190 countries without AGORA data are not penalised — they are simply scored without this dimension. As AGORA expands coverage to more jurisdictions, this dimension's impact will grow without requiring methodology changes.

**Mapping AGORA to countries:** AGORA documents are mapped to countries via the `authorities` table, using the `Jurisdiction` field. Sub-national authorities (e.g., California, New York City) are counted toward the parent country (United States).

---

## Scoring & Normalisation

### Sub-Component Scoring

Each sub-component is computed as a **0–1 fraction** and combined into a dimension score using its explicit sub-weights, producing a **0–100 dimension score**. Missing sub-components do not zero out a dimension — their weights are redistributed proportionally to the components that are present.

### Log Transformations

The following metrics are log-transformed (log(1 + x)) before normalisation, based on measured skewness from the data audit:

| Metric | Raw Skewness | Post-Log Skewness | Top-1/Median Ratio (Raw → Log) |
|---|---|---|---|
| Publication volume | 8.3 | 0.2 | 1,127× → 2.6× |
| Patent volume | 6.9 | 0.8 | 4,987× → 3.4× |
| Estimated investment | 10.4 | 0.3 | 6,741× → 2.8× |
| Company count | Moderate | Low | Applied for consistency |
| Enacted governance docs | Moderate | Low | Applied for consistency |

Without log transformation, min-max normalisation compresses >90% of countries into the bottom 5% of the scale, making the index uninformative for all but the top 5 nations.

### Dimension Normalisation

Dimension scores are normalised to a common 0–100 scale using one of two methods:

- **Anchored normalisation** — used where a stable global reference exists (e.g., semiconductor market share is inherently 0–100%). Scores are comparable across different runs of the index.
- **Relative min-max normalisation** — used for dimensions without a fixed reference. Applied across the current country set. The normalisation method used is flagged in the output for each dimension.

### Data Completeness Filters

Before scoring, the following completeness filters are applied:

| Data Source | Filter Rule | Effective Range |
|---|---|---|
| Publications (articles) | `complete = true` only | 2015–2023 |
| Publications (citations) | Years max-4 to max-2 (no `complete` flag available) | 2021–2023 |
| Patents | `complete = true` only | 2015–2020 (5-year lag) |
| Investment | `complete = true` only | 2015–2024 |
| Cross-border collaborations | `complete = true` only; pairs <25 articles already excluded by source | 2015–2023 |
| AGORA | `Most recent activity = "Enacted"` for primary scoring | All years |
| Semiconductor | Point-in-time snapshot (mostly 2024 market shares) | Snapshot |

---

## Composite Index & Confidence

### Dimension Weights

| Dimension | Weight | Coverage | Rationale for Weight |
|---|---|---|---|
| Hardware Sovereignty | 22% | ~23 countries | Chip access is the single most acute geopolitical bottleneck |
| Research Capacity | 20% | ~192 countries | Broadest coverage; foundational to all AI activity |
| Commercial Ecosystem | 20% | ~117 countries | Private-sector depth drives real-world AI deployment |
| Innovation Output | 18% | ~73 countries | IP protection matters, though 5-year data lag limits recency |
| Talent Base | 10% | 17 countries | Most sanction-resistant asset, but severe data limitations |
| Collaboration Network | 5% | ~94 countries | Strategic partnerships matter but less than direct capacity |
| Governance Readiness | 5% | 2 countries | Institutional infrastructure matters; minimal current coverage |
| **Total** | **100%** | | |

**Weight change log (v2 → v3):**
- Research Capacity: 18% → 20% (+2%). Broadest coverage of any dimension; marginal weight increase rewards the most data-rich signal.
- Commercial Ecosystem: 18% → 20% (+2%). Good coverage and high relevance for deployment-stage resilience.
- Governance Readiness: 9% → 5% (-4%). Only 2 countries individually scoreable; weight now proportional to actual data utility.

### Coverage-Weighted Composite

The final **AISI Resilience Score** is not a simple weighted average. It is a **coverage-weighted composite**: each dimension's contribution is scaled by how much reliable data was actually available for that country in that dimension. A country missing two entire datasets is not penalised as if it scored zero — it is simply scored on the dimensions where data exists, and its overall data coverage (0–1) is reported alongside its index score.

```
AISI Score = Σ (dimension_score × dimension_weight × dimension_coverage)
             ─────────────────────────────────────────────────────────────
                   Σ (dimension_weight × dimension_coverage)
```

Countries with overall data coverage below 50% are flagged as **low confidence** and their rankings should be treated with caution.

### Availability Flags

For each country, boolean `has_*` flags are set **before any NaN filling or imputation**:

- `has_publications` — country appears in publications data (after group entity exclusion)
- `has_citations` — country has citation data in the usable year range (2021–2023)
- `has_patents` — country appears in patent data (after group entity exclusion)
- `has_investment` — country appears in estimated investment data
- `has_semiconductor` — country has at least one mapped semiconductor provider (via harmonization table)
- `has_collaboration` — country appears in cross-border collaboration data
- `has_parat` — country has at least one company in PARAT
- `has_workforce` — country has PARAT companies with non-zero workforce data
- `has_governance` — country has at least one enacted document in AGORA (individually mappable — excludes "Multinational" and "Other countries")

These flags ensure that zeros in the scored data represent genuine absences (the country has data but the value is zero) rather than missing data that was silently filled.

### HQ Attribution Flags

Countries where headquarters-based attribution is likely distortive are flagged in the output. The flag is triggered when a country meets any of the following conditions:
- Classified as a tax haven or offshore financial centre by the OECD or IMF
- Has a ratio of PARAT-listed company HQs to population that exceeds three standard deviations above the global mean
- Has known discrepancies between corporate domicile and operational presence (maintained as a manual list, initially including Ireland, Luxembourg, the Cayman Islands, Bermuda, and Singapore)

Flagged countries are not removed from rankings but are annotated so analysts can apply judgment.

---

## Key Design Choices & Their Rationale

| Choice | Rationale |
|---|---|
| Country name harmonization table | Five datasets use different naming conventions (ISO codes, full names, variants like "China (mainland)"); without explicit mapping, cross-dataset joins fail silently |
| Group entity exclusion | 12 aggregate entities (OECD, NATO, EU, etc.) in Country AI Activity data are not countries and must be removed before scoring |
| Hardware Sovereignty weighted highest at 22% | Chip access is the single most acute geopolitical bottleneck — cannot be substituted by research excellence alone |
| Research Capacity and Commercial Ecosystem at 20% each | Best data coverage (192 and 117 countries); most reliable signals in the index |
| Governance Readiness reduced to 5% | Only 2 countries individually scoreable; weight now proportional to data utility |
| Citation impact uses a lagged window (2021–2023) | Source dataset warns citation data is incomplete in recent years; latest year measures data lag, not quality |
| Patent grant rate removed | Grant rates confounded by patent office behaviour across jurisdictions |
| Innovation Output flagged as lagged | Complete patent data ends at 2020 — 5 years behind. Countries that scaled patenting post-2020 get no credit |
| Estimated investment over disclosed | Disclosed-only systematically undercounts countries with stronger deal confidentiality norms |
| Log(1+x) on skewed metrics | Skewness of 6.9–10.4 compresses 90%+ of countries to the bottom; log transform reduces to 0.2–0.8 |
| Supply chain breadth denominator = 96 | Only 96 of 126 inputs have market share data; using 126 penalises for non-existent data |
| Partner diversity cap = 54 (95th percentile) | Only 5 countries exceed this; cap preserves variation without letting the US at 79 partners dominate |
| Talent Tier 1 threshold lowered to 3 companies | With only 17 PARAT countries, threshold of 5 restricted Tier 1 to the US alone |
| Coverage-weighted composite | Distinguishes "low capability" from "no data" — critical when dimensions cover 2 to 192 countries |
| `has_*` flags set before NaN filling | Ensures zeros represent genuine absences, not missing data silently filled |

---

## What the Index Does Not Capture

- **Classified or undisclosed R&D** — sovereign AI programmes that do not publish or patent publicly.
- **Talent quality** — only quantity proxies are available; no measure of skill level, educational quality, or researcher impact beyond citation counts.
- **Compute access beyond semiconductors** — cloud infrastructure, energy availability, data centre capacity, and sovereign compute initiatives.
- **Domestic regulatory enforcement** — AGORA captures the existence of governance documents but not whether they are effectively enforced.
- **Absolute capability** — because normalisation is partly relative, the index measures position within the current cohort, not absolute level. Two countries scoring 60 and 40 are not necessarily in a 3:2 ratio of absolute capability.
- **Non-English-language governance** — AGORA's US skew (91% of documents) means governance activity in non-English-speaking countries is severely undercounted.
- **Physical production geography** — HQ-based corporate attribution does not reflect where manufacturing, R&D, or deployment physically occurs.
- **Private-market completeness** — Crunchbase-derived investment data excludes public companies, government spending, non-equity funding, and companies not in Crunchbase.
- **Chinese-language publications** — the Merged Academic Corpus underrepresents Chinese-language research, which may undercount China's actual research output.
- **Post-2020 patent activity** — the 5-year patent lag means the Innovation Output dimension is blind to recent patenting surges.
- **Non-US/China governance activity** — 188+ countries have no individually scoreable AGORA presence.

---

## Output Schema

The final output for each country includes:

| Field | Type | Description |
|---|---|---|
| `country` | text | Canonical country name (post-harmonization) |
| `aisi_score` | float | Final AISI Resilience Score (0–100) |
| `data_coverage` | float | Overall data coverage (0–1) |
| `low_confidence` | boolean | True if data coverage < 50% |
| `hq_attribution_flag` | boolean | True if HQ-based attribution is likely distortive |
| `harmonization_warning` | boolean | True if country name required non-trivial matching |
| `dim_research_capacity` | float | Research Capacity dimension score (0–100) |
| `dim_innovation_output` | float | Innovation Output dimension score (0–100) |
| `dim_commercial_ecosystem` | float | Commercial Ecosystem dimension score (0–100) |
| `dim_hardware_sovereignty` | float | Hardware Sovereignty dimension score (0–100) |
| `dim_collaboration_network` | float | Collaboration Network dimension score (0–100) |
| `dim_talent_base` | float | Talent Base dimension score (0–100) |
| `dim_governance_readiness` | float | Governance Readiness dimension score (0–100) |
| `talent_tier` | integer | Which tier (1/2/3) was used for talent scoring |
| `innovation_output_latest_year` | integer | Latest complete patent year (currently 2020) |
| `norm_method_*` | text | Normalisation method used per dimension (anchored / relative) |
| `has_publications` | boolean | Data availability flag |
| `has_citations` | boolean | Data availability flag |
| `has_patents` | boolean | Data availability flag |
| `has_investment` | boolean | Data availability flag |
| `has_semiconductor` | boolean | Data availability flag |
| `has_collaboration` | boolean | Data availability flag |
| `has_parat` | boolean | Data availability flag |
| `has_workforce` | boolean | Data availability flag |
| `has_governance` | boolean | Data availability flag |

---

## Version History

| Version | Date | Changes |
|---|---|---|
| v1 (GAIRI) | — | Original 6-dimension methodology |
| v2 | — | Added Governance Readiness (7th dimension), log transforms, supply chain breadth, partner quality, citation lag, removed patent grant rate, switched to estimated investment |
| v3 | — | Post-data audit: added country harmonization section, group entity exclusion, revised weights (Research 20%, Commercial 20%, Governance 5%), fixed breadth denominator to 96, set partner cap to 54, lowered Tier 1 threshold to 3, added skewness statistics, documented effective date ranges, added innovation lag flag, added harmonization warning to output |
