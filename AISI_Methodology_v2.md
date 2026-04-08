# AISI — AI Supremacy Index: Methodology

## What Is It?

The **AI Supremacy Index (AISI)** is a composite score that measures how structurally resilient a country is in the global AI landscape — not just how active it is today, but how defensible and self-sustaining its AI position is against geopolitical, supply chain, and competitive shocks.

---

## Data Sources

Five datasets feed the index:

| Dataset | What It Captures |
|---|---|
| **Country AI Activity Metrics** | Annual publications, citations, patent applications & grants, and estimated investment by country and research field |
| **Advanced Semiconductor Supply Chain** | Provider market shares across semiconductor inputs and fabrication stages |
| **Cross-Border Tech Research Metrics** | Co-authored research papers between country pairs, broken down by topic and year |
| **Private-Sector AI Indicators (PARAT)** | Corporate AI activity — company counts, AI publication output, and workforce figures |
| **AGORA** | AI governance documents — laws, regulations, standards, and their thematic coverage |

---

## The Seven Dimensions

The index is built from seven dimensions, each capturing a distinct aspect of resilience. They are calculated independently and then combined.

### 1. Research Capacity (18%)

Measures the depth and momentum of a country's scientific AI output. Three sub-components are combined with explicit weights:

- **Publication volume** (35%) — average annual AI articles over the most recent three complete years of data (filtering on `complete = true` to avoid undercounting from data lag).
- **Citation impact** (45%) — citations per article, averaged over a three-year window ending at least two years before the current date. This lag is deliberate: the Country AI Activity dataset warns that citation data is incomplete in recent years and does not yet include a `complete` flag for citations. Using a lagged window ensures the metric reflects genuine research quality rather than data pipeline latency.
- **Growth trend** (20%) — five-year publication trajectory, standardised using a z-score across all countries so that fast-growing smaller nations are not unfairly penalised against large incumbents.

Countries with fewer than five years of data are scored only on the available sub-components, with weights redistributed accordingly.

**Log transformation:** publication volume is log-transformed (log(1 + x)) before normalisation to prevent extreme right-skew from compressing most countries to the bottom of the scale. The US and China dominate raw counts; log transformation preserves rank order while spreading the distribution.

---

### 2. Innovation Output (18%)

Measures the translation of research into protected intellectual property across diverse fields:

- **Patent volume** (45%) — total AI patent applications filed, log-transformed before normalisation to address the same skew problem as publication counts.
- **Field diversity** (55%) — number of distinct patent thematic fields with active patent activity. Only fields with at least one patent application count, preventing inflated scores from data artifacts.

**Why grant rate was removed:** the original methodology used patent grant rate (granted / applications) as a quality signal. However, grant rates are heavily confounded by patent office behaviour — the Chinese patent office historically grants at rates above 90%, while the USPTO and EPO are far stricter. A high grant rate in one jurisdiction reflects a permissive examiner, not better filings. Since the Country AI Activity dataset attributes patents to the filing country (not the inventor's country), this confound cannot be corrected within the available data. Field diversity is a more robust quality proxy: countries filing across many distinct thematic fields demonstrate broad innovative capacity rather than narrow specialisation.

---

### 3. Commercial Ecosystem (18%)

Measures the private-sector depth behind a country's AI position:

- **Investment** (50%) — total estimated AI investment (in USD millions), using the `companies_yearly_estimated` table rather than disclosed-only values. The Country AI Activity dataset provides estimated investment that accounts for undisclosed deal values using median imputation by investment stage, target country, and year. Disclosed-only data systematically undercounts countries where deal confidentiality norms are stronger. Log-transformed before normalisation.
- **Company count** (30%) — number of active AI companies per PARAT, log-transformed.
- **Publication intensity** (20%) — average AI publications per company (total PARAT AI publications ÷ company count), reflecting a research-active corporate culture rather than just corporate presence. Not log-transformed, as this ratio is already naturally bounded.

---

### 4. Hardware Sovereignty (22%)

The highest-weighted dimension, reflecting that semiconductor access is the primary geopolitical chokepoint in AI. It measures a country's foothold in the global semiconductor supply chain:

- **Average market share** (35%) — mean market share across all semiconductor inputs and stages where the country has a provider presence.
- **Peak market share** (30%) — the country's strongest position in any single critical input. This captures dominant positions in chokepoint inputs (e.g., EUV lithography).
- **Supply chain breadth** (35%) — the fraction of total semiconductor inputs/stages where the country has any provider presence. This sub-component was added to prevent narrow specialists from outscoring countries with broad, moderate presence across the supply chain. A country with 15% share across 30 inputs is more resilient than one with 90% in a single niche.

Countries with no mapped semiconductor provider presence score zero. Country names are matched to semiconductor data using an explicit lookup table (ISO 3166 codes to provider names), and unmatched countries are flagged by name for the analyst to resolve.

**Note on HQ-based attribution:** the semiconductor dataset assigns country affiliation based on the ultimate parent company's headquarters location, not physical production location. This means, for example, that TSMC's market share counts toward Taiwan even though it operates fabrication plants in the United States and Japan. This is a known limitation that affects all country-level metrics derived from corporate data in AISI (including Commercial Ecosystem and Talent Base). Countries where HQ-based attribution is likely distortive — particularly small open economies and tax-haven jurisdictions — are flagged in the output.

---

### 5. Collaboration Network (5%)

Measures the breadth and strategic quality of a country's international research partnerships:

- **Partner diversity** (40%) — number of unique countries collaborated with, capped at the empirical 95th percentile of partner counts across all countries in the dataset. This cap prevents the sub-component from being dominated by a handful of hyper-connected countries while preserving meaningful variation below the threshold. The exact cap value is computed dynamically per run and reported in the output.
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

- **Partner quality** (25%) — collaboration volume weighted by each partner country's own Research Capacity score. This creates a one-step dependency: a country's Collaboration Network score depends partly on its partners' Research Capacity scores, which are computed first as an independent dimension. Collaborating with the United States or China adds more resilience signal than collaborating with a country that publishes ten papers per year.

---

### 6. Talent Base (10%)

Measures the human capital underpinning AI activity, using a tiered fallback approach depending on data availability:

- **Tier 1** — AI workforce headcount from PARAT (highest confidence). Used when the country has at least five PARAT companies with workforce data.
- **Tier 2** — Total AI publications from PARAT companies, as a proxy for active research talent. Used when Tier 1 data is insufficient.
- **Tier 3** — Company count from PARAT (lowest confidence, flagged in output). Used as last resort.

The tier used is recorded alongside the score so analysts know which countries have reliable talent data.

**Weight rationale:** talent was raised from 5% to 10% (relative to the original draft) because, for a resilience index, human capital is arguably the most durable asset — it is harder to sanction, embargo, or disrupt than chips, capital, or IP. The concern about data quality (PARAT workforce data is unreliable outside the U.S.) is real, but the coverage-weighted composite already handles this: countries with poor talent data will have low coverage on this dimension, and it will be downweighted automatically in the final composite.

**Known limitation:** workforce data from PARAT is derived from LinkedIn profiles and is unreliable outside the United States. LinkedIn is blocked in China and Russia. The Tier 2 and Tier 3 fallbacks partially mitigate this, but talent scores for non-U.S. countries should be interpreted with caution.

---

### 7. Governance Readiness (9%)

Measures the maturity and breadth of a country's AI governance framework. This dimension is derived from AGORA and captures institutional infrastructure — a country with enacted regulations, safety frameworks, and standards is better positioned to respond to crises, attract cautious investors, and maintain international legitimacy.

- **Document count** (30%) — number of enacted AI governance documents for the country (filtering on `Most recent activity = "Enacted"`). Log-transformed before normalisation.
- **Thematic breadth** (40%) — number of distinct AGORA taxonomy tags covered across all of the country's enacted documents (spanning risk factors, harms, governance strategies, incentives, and application domains). This rewards countries whose governance covers more of the AI policy landscape, not just those that produce many narrowly focused documents.
- **Maturity ratio** (30%) — ratio of enacted documents to total documents (enacted + proposed + defunct). A higher ratio indicates a governance pipeline that converts proposals into law, rather than one stuck in perpetual deliberation.

**Known limitations:** AGORA currently skews heavily toward U.S. federal law and policy. Coverage of other countries — particularly non-English-speaking jurisdictions — is incomplete. This means the Governance Readiness dimension will systematically undercount governance activity in countries not yet well-covered by AGORA. To mitigate this:
- The dimension receives a moderate weight (9%) rather than a higher one.
- The coverage-weighted composite ensures that countries with minimal AGORA presence are scored on other dimensions rather than penalised with a zero.
- As AGORA coverage expands, this dimension's reliability will improve without requiring methodological changes.

**Mapping AGORA to countries:** AGORA documents are mapped to countries via the `authorities` table, using the `Jurisdiction` field. Documents issued by sub-national authorities (e.g., California) are counted toward the parent country (e.g., United States).

---

## Scoring & Normalisation

### Sub-Component Scoring

Each sub-component is computed as a **0–1 fraction**. Where specified above, raw values are log-transformed (log(1 + x)) before normalisation to handle right-skewed distributions. Sub-components are then combined into a dimension score using their explicit sub-weights, producing a **0–100 dimension score**. Missing sub-components do not zero out a dimension — their weights are redistributed proportionally to the components that are present.

### Dimension Normalisation

Dimension scores are normalised to a common 0–100 scale using one of two methods:

- **Anchored normalisation** — used where a stable global reference exists (e.g., semiconductor market share is inherently 0–100%). Scores are comparable across different runs of the index.
- **Relative min-max normalisation** — used for dimensions without a fixed reference. Applied across the current country set. The normalisation method used is flagged in the output for each dimension so analysts know which scores are stable across runs and which are relative to the current cohort.

### Data Completeness Filters

Before scoring, the following completeness filters are applied:

- **Publications:** only rows where `complete = true` are used for volume calculations.
- **Citations:** only years at least two years behind the current date are used, pending the addition of a `complete` flag to citation data.
- **Patents:** only rows where `complete = true` are used.
- **Investment:** only rows where `complete = true` are used.
- **Cross-border collaborations:** only rows where `complete = true` are used. Country pairs with fewer than 25 joint publications (which are already omitted by the source dataset) are not imputed.
- **AGORA:** only documents with `Most recent activity = "Enacted"` are used for the primary scoring. Proposed and defunct documents are used only for the maturity ratio calculation.

---

## Composite Index & Confidence

### Dimension Weights

| Dimension | Weight |
|---|---|
| Hardware Sovereignty | 22% |
| Research Capacity | 18% |
| Innovation Output | 18% |
| Commercial Ecosystem | 18% |
| Talent Base | 10% |
| Governance Readiness | 9% |
| Collaboration Network | 5% |
| **Total** | **100%** |

### Coverage-Weighted Composite

The final **AISI Resilience Score** is not a simple weighted average. It is a **coverage-weighted composite**: each dimension's contribution is scaled by how much reliable data was actually available for that country in that dimension. A country missing two entire datasets is not penalised as if it scored zero on those dimensions — it is simply scored on the dimensions where data exists, and its overall data coverage (0–1) is reported alongside its index score.

```
AISI Score = Σ (dimension_score × dimension_weight × dimension_coverage)
             ─────────────────────────────────────────────────────────────
                   Σ (dimension_weight × dimension_coverage)
```

Countries with overall data coverage below 50% are flagged as **low confidence** and their rankings should be treated with caution.

### Availability Flags

For each country, boolean `has_*` flags are set **before any NaN filling or imputation**:

- `has_publications` — country appears in publications data
- `has_citations` — country has citation data in the usable year range
- `has_patents` — country appears in patent data
- `has_investment` — country appears in investment data
- `has_semiconductor` — country has at least one mapped semiconductor provider
- `has_collaboration` — country appears in cross-border collaboration data
- `has_parat` — country has at least one company in PARAT
- `has_workforce` — country has PARAT companies with workforce data
- `has_governance` — country has at least one enacted document in AGORA

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
| Hardware Sovereignty weighted highest at 22% | Chip access is the single most acute geopolitical bottleneck — it cannot be substituted by research excellence alone |
| Citation impact uses a lagged window | The source dataset warns citation data is incomplete in recent years; using the latest year would measure data lag, not quality |
| Patent grant rate removed | Grant rates are confounded by patent office behaviour across jurisdictions; field diversity is a more robust quality proxy |
| Estimated investment used over disclosed | Disclosed-only data systematically undercounts countries with stronger deal confidentiality norms |
| Log transformation on skewed metrics | Investment, publication, and patent counts are extremely right-skewed; without log transform, min-max normalisation compresses most countries to the bottom |
| Supply chain breadth added to Hardware Sovereignty | Prevents narrow specialists from outscoring countries with broad, moderate presence across the supply chain |
| Strategic topic multipliers in collaboration | A chip-design partnership is more strategically valuable than a general AI co-authorship |
| Partner quality weighting in collaboration | Collaborating with a major AI power contributes more to resilience than collaborating with a minimal-output country |
| Partner diversity cap set empirically | Fixed at the 95th percentile rather than an arbitrary number; computed dynamically per run |
| Talent raised to 10% | Human capital is the most sanction-resistant AI asset; data quality concerns are handled by the coverage-weighted composite |
| Governance Readiness added at 9% | Institutional infrastructure matters for resilience; AGORA is the only dataset of the five that was unused in the original draft |
| Coverage-weighted composite | Distinguishes between "low capability" and "no data" — a distinction that matters for smaller and developing nations |
| `has_*` availability flags set before NaN filling | Ensures zeros represent genuine absences, not missing data that was silently filled |
| HQ attribution flagging | Alerts analysts to countries where corporate domicile diverges from operational reality |
| Seven dimensions instead of six | Adding Governance Readiness uses all five available datasets and captures a resilience facet (institutional infrastructure) that no other dimension addresses |

---

## What the Index Does Not Capture

- **Classified or undisclosed R&D** — sovereign AI programmes that do not publish or patent publicly.
- **Talent quality** — only quantity proxies are available; no measure of skill level, educational quality, or researcher impact beyond citation counts.
- **Compute access beyond semiconductors** — cloud infrastructure, energy availability, data centre capacity, and sovereign compute initiatives.
- **Domestic regulatory enforcement** — AGORA captures the existence of governance documents but not whether they are effectively enforced.
- **Absolute capability** — because normalisation is partly relative, the index measures position within the current cohort, not absolute level. Two countries scoring 60 and 40 are not necessarily in a 3:2 ratio of absolute capability.
- **Non-English-language governance** — AGORA's current U.S. skew means governance activity in non-English-speaking countries is undercounted.
- **Physical production geography** — HQ-based corporate attribution does not reflect where manufacturing, R&D, or deployment physically occurs.
- **Private-market completeness** — Crunchbase-derived investment data excludes public companies, government spending, non-equity funding, and companies not in Crunchbase.
- **Chinese-language publications** — the Merged Academic Corpus underrepresents Chinese-language research, which may undercount China's actual research output.

---

## Output Schema

The final output for each country includes:

| Field | Type | Description |
|---|---|---|
| `country` | text | Country name |
| `aisi_score` | float | Final AISI Resilience Score (0–100) |
| `data_coverage` | float | Overall data coverage (0–1) |
| `low_confidence` | boolean | True if data coverage < 50% |
| `hq_attribution_flag` | boolean | True if HQ-based attribution is likely distortive |
| `dim_research_capacity` | float | Research Capacity dimension score (0–100) |
| `dim_innovation_output` | float | Innovation Output dimension score (0–100) |
| `dim_commercial_ecosystem` | float | Commercial Ecosystem dimension score (0–100) |
| `dim_hardware_sovereignty` | float | Hardware Sovereignty dimension score (0–100) |
| `dim_collaboration_network` | float | Collaboration Network dimension score (0–100) |
| `dim_talent_base` | float | Talent Base dimension score (0–100) |
| `dim_governance_readiness` | float | Governance Readiness dimension score (0–100) |
| `talent_tier` | integer | Which tier (1/2/3) was used for talent scoring |
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
