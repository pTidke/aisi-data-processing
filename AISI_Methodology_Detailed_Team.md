# AISI Methodology — Internal Technical Reference

## What Is the AISI Score?

The AI Supremacy Index (AISI) is a single number from 0 to 100 that measures how **resilient** a country's AI position is — not just how much AI activity it has today, but how well it could sustain that position if supply chains broke, sanctions hit, or talent moved. Think of it as a stress-test score for national AI ecosystems.

The score is built from **7 dimensions**, each measuring a different aspect of AI resilience. Each dimension is scored 0–100, then combined into one final number using weights that reflect how important each aspect is for resilience.

---

## The 7 Dimensions — What They Are, How They're Calculated, and Why

---

### 1. Research Capacity (20% of final score)

**What it measures:** How much high-quality AI research is a country producing, and is it growing?

**Why it matters for resilience:** Research is the foundation of everything else. A country that produces significant AI research can develop its own models, train its own talent, and doesn't depend on importing knowledge from others. It's the hardest capability to sanction or embargo.

**How it's calculated — 3 sub-components:**

- **Publication volume (35% of this dimension):** We take the average number of AI research articles a country published per year over 2021–2023 (the most recent three years where data is considered complete). We only count years the source marks as `complete = true` to avoid undercounting from data pipeline lag. The raw counts are then **log-transformed** — more on why below.

- **Citation impact (45% of this dimension):** How much does other research reference this country's work? We calculate citations per article, but critically, we use a **lagged window of 2021–2023** rather than the latest year. This is because the source dataset warns that citation data in recent years is incomplete (new papers haven't had time to be cited yet, and there's no `complete` flag for citations). Using the latest year would measure how fast the data pipeline runs, not how influential the research is.

- **Growth trend (20% of this dimension):** Is the country's research output accelerating or stalling? We compute a compound annual growth rate over the last 5 complete years, then **z-score standardise** it across all countries. Z-scoring means we're measuring "how fast is this country growing *relative to the global norm*" rather than raw growth rate — this prevents large countries with modest percentage growth from being penalised against small countries with high percentage growth off a tiny base.

**Why log-transform publication counts?** Because the raw data is extraordinarily skewed. China publishes ~85,700 AI articles per year. The median country publishes 76. That's a 1,127× gap. If we normalised on raw values, 90%+ of countries would be compressed into the bottom 5% of the scale, and the index would only differentiate between the top 5 nations. Log(1+x) transformation brings the skewness from 8.3 down to 0.2, spreading the distribution so that meaningful differences between mid-tier countries (like Brazil vs. Turkey) actually show up.

**Data coverage:** ~192 countries — the broadest of any dimension.

---

### 2. Innovation Output (18% of final score)

**What it measures:** Is a country translating its research into protected intellectual property, and is it doing so across diverse fields?

**Why it matters for resilience:** Patents represent applied knowledge — the step between "we published a paper" and "we built something defensible." A country with broad patent activity across many AI subfields has more options and more fallback positions than one concentrated in a single area.

**How it's calculated — 2 sub-components:**

- **Patent volume (45% of this dimension):** Total AI patent applications filed across all complete data years. Log-transformed for the same skewness reasons as publications (raw skewness: 6.9, China alone files 164,582 vs. median of 33).

- **Field diversity (55% of this dimension):** How many of the 11 distinct AI patent fields does the country have active filings in? A country filing patents in life sciences AI, autonomous vehicles, cybersecurity, AND natural language processing demonstrates broader innovative capacity than one filing exclusively in computer vision. Score = fields with ≥1 application ÷ 11.

**Why field diversity is weighted higher than volume:** We originally included patent **grant rate** (granted ÷ applications) as a quality signal. We removed it after discovering it's confounded by patent office behaviour: China's patent office historically grants at >90%, while the USPTO and EPO are far stricter. A high grant rate in China reflects a permissive examiner, not better filings. Field diversity turned out to be a more robust quality proxy — it measures breadth of innovative capacity rather than a metric gamed by jurisdictional differences.

**Critical data limitation:** Complete patent data ends at **2020** — a 5-year lag. This dimension is essentially scoring pre-pandemic patenting activity. Any country that massively scaled AI patenting after 2020 (through national AI strategies, for example) gets zero credit. This is flagged in every output.

**Data coverage:** ~73 countries.

---

### 3. Commercial Ecosystem (20% of final score)

**What it measures:** How deep is the private sector behind a country's AI position? Is there real money flowing in, are companies actively building, and are those companies doing research?

**Why it matters for resilience:** Government research programs can be defunded. Academic output can stall. But a deep commercial ecosystem — venture capital, active companies, corporate R&D labs — creates self-sustaining momentum. Companies hire talent, attract investment, generate revenue, and reinvest. It's the flywheel.

**How it's calculated — 3 sub-components:**

- **Investment (50% of this dimension):** Total estimated AI investment in USD millions. We deliberately use **estimated** investment rather than **disclosed** investment. The source dataset provides estimated values that account for undisclosed deal amounts using median imputation by investment stage, country, and year. Disclosed-only data systematically undercounts countries where deal confidentiality norms are stronger (e.g., some Asian and Middle Eastern markets). Log-transformed (raw skewness: 10.4 — the US alone accounts for $849B vs. a median of $126M).

- **Company count (30% of this dimension):** Number of active AI companies tracked in the PARAT dataset. Log-transformed. This is only available for 17 countries (PARAT is a curated set of 691 companies, 85% US-based). For the ~175 countries without PARAT data, this sub-component's weight is **redistributed to Investment** — so they're scored on investment alone rather than penalised for missing corporate data.

- **Publication intensity (20% of this dimension):** Average AI publications per company (total PARAT publications ÷ company count). This rewards countries where companies actually *do* research, not just exist. A country with 50 companies publishing 10,000 papers scores higher than one with 200 companies publishing 2,000 papers. Not log-transformed because it's a ratio and already naturally bounded. Same PARAT-coverage caveat.

**Data coverage:** ~117 countries for investment; 17 for PARAT-dependent sub-components.

---

### 4. Hardware Sovereignty (22% of final score — highest weighted)

**What it measures:** How much control does a country have over the physical infrastructure required to build AI systems — specifically, the semiconductor supply chain?

**Why it gets the highest weight:** You can have the best researchers, the most patents, and the deepest venture capital — but if you can't access advanced chips, none of it translates to frontier AI capability. Semiconductor access is the single most acute geopolitical chokepoint in AI today. Export controls, TSMC concentration, and the ASML monopoly on EUV lithography make this the dimension most likely to determine AI outcomes during a geopolitical crisis.

**How it's calculated — 3 sub-components:**

- **Average market share (35% of this dimension):** Mean market share percentage across all semiconductor inputs (tools, materials, processes, designs) where the country has any provider presence. If a country has firms in 20 different input categories averaging 15% share each, it scores higher than a country in 3 categories averaging 30%.

- **Peak market share (30% of this dimension):** The country's single strongest position in any input category. This captures chokepoint dominance — the Netherlands scores 100% here because ASML has a complete monopoly on EUV lithography tools. The US scores 100% because Nvidia/AMD monopolise discrete GPUs. These monopoly positions matter enormously for resilience.

- **Supply chain breadth (35% of this dimension):** What fraction of the 96 semiconductor inputs with data does the country have *any* provider in? The denominator is 96, not 126 (total inputs in the dataset), because 30 inputs have no market share data for any country — penalising countries for not being present in categories where no data exists would be punishing data gaps, not measuring capability.

**Why breadth is weighted heavily:** We added breadth specifically to prevent a scoring artifact. Without it, a country with 90% share in one niche input would outscore a country with 15% share across 30 critical inputs. The second country is far more resilient — it has fallback positions across the supply chain — but without the breadth component, it would lose on both average and peak metrics.

**HQ attribution caveat:** All country assignments are based on where the ultimate parent company is headquartered, not where production physically happens. TSMC's market share counts for Taiwan even though it operates fabs in the US and Japan. This is a known limitation. Countries where HQ-attribution is likely distortive (tax havens, small open economies) are flagged.

**Data coverage:** ~23 countries. The remaining ~170 countries score zero — they have no mapped semiconductor provider presence. The coverage-weighted composite handles this (see below).

---

### 5. Collaboration Network (5% of final score)

**What it measures:** How broadly and strategically does a country collaborate on AI research with other nations?

**Why it matters for resilience:** A country with deep research partnerships across many nations is harder to isolate. If one collaboration channel is cut (e.g., US-China chip design collaboration declining), others remain. The *quality* of partners matters too — collaborating with the US or China provides more resilience than collaborating with a country that publishes 10 papers per year.

**How it's calculated — 3 sub-components:**

- **Partner diversity (40% of this dimension):** Count of unique countries a nation has co-authored research with, **capped at 54** (the empirical 95th percentile). Without the cap, the US at 79 partners would dominate this sub-component so much that variation among the other 93 countries would be compressed. The cap preserves meaningful differentiation in the middle of the distribution while still rewarding broadly connected nations.

- **Weighted volume (35% of this dimension):** Total co-authored articles, but weighted by **strategic topic multipliers**: chip design collaborations count 2.0× (direct hardware relevance), LLM collaborations 1.8× (frontier AI), AI safety 1.6× (governance signal), cybersecurity 1.4× (security), while mature subfields like computer vision and robotics count 1.0×. This means 1,000 chip-design co-publications contribute more to a country's score than 1,000 robotics co-publications — because chip-related collaboration is geopolitically scarcer and strategically more valuable.

- **Partner quality (25% of this dimension):** Collaboration volume weighted by each partner's own Research Capacity score. This creates a deliberate one-step dependency: Research Capacity is computed first as an independent dimension, then its scores feed into this calculation. Collaborating with China (Research Capacity ~48.5) adds more to your score than collaborating with a country scoring 5.0 — because resilience comes from being embedded in high-capability networks, not from having many low-value connections.

**Data coverage:** ~94 countries. Only complete years (2015–2023) are used. The source already filters out country pairs with fewer than 25 joint publications per year.

---

### 6. Talent Base (10% of final score)

**What it measures:** How deep is the AI talent pool available in a country?

**Why it matters for resilience:** Talent is arguably the most sanction-resistant AI asset. You can embargo chips, block investment, and restrict technology transfer — but people are mobile, adaptable, and self-sustaining. A country with a deep AI workforce can rebuild other capabilities; a country without one cannot.

**How it's calculated — tiered fallback:**

The challenge with talent data is that the best source (PARAT's workforce data, derived from LinkedIn profiles) is only reliable for the US and a handful of other countries. LinkedIn is blocked in China and Russia, and far less popular outside the Anglosphere. So we use a 3-tier fallback:

- **Tier 1 — AI workforce headcount** (highest confidence): Used when a country has ≥3 PARAT companies with non-zero workforce data. This threshold was lowered from 5 to 3 after our data audit revealed that with only 17 PARAT countries, a threshold of 5 effectively restricted Tier 1 to the US alone.

- **Tier 2 — Total AI publications from PARAT companies** (medium confidence): If workforce data is insufficient, we use corporate publication output as a proxy for active research talent. A company publishing AI papers has AI researchers.

- **Tier 3 — Company count** (lowest confidence): Last resort. Just counting how many AI companies are headquartered in the country. Flagged in the output.

For the 175+ countries with no PARAT presence at all, this dimension has **zero coverage** and is excluded entirely from their composite score — they're scored on the other 6 dimensions only.

**Why only 10% weight?** Talent is critically important for resilience, but the data quality is severely limited. The coverage-weighted composite already handles this mechanically (countries with bad talent data get downweighted automatically), but we also kept the explicit weight moderate to avoid giving outsized influence to a dimension where only 17 countries have any data and only ~5 have reliable data.

**Data coverage:** 17 countries via PARAT. Effectively US-only for Tier 1 workforce data.

---

### 7. Governance Readiness (5% of final score — lowest weighted)

**What it measures:** Does a country have institutional AI governance infrastructure — enacted laws, regulations, and standards that cover the AI policy landscape?

**Why it matters for resilience:** A country with enacted AI regulation, safety frameworks, and standards is better positioned to respond to crises, attract risk-averse investors, and maintain international legitimacy. Institutional infrastructure is slow to build and creates lasting structural advantage.

**How it's calculated — 3 sub-components:**

- **Document count (30% of this dimension):** Number of enacted AI governance documents (filtering on `Most recent activity = "Enacted"`). Log-transformed because the US has 390 enacted documents while China has 18 — without log transform, this sub-component would just measure "how much is this country the US."

- **Thematic breadth (40% of this dimension):** How many of the 77 taxonomy tags in the AGORA dataset are covered across all of a country's enacted documents? The tags span 5 domains: risk factors, harms, governance strategies, incentives, and application domains. A country whose laws cover AI in healthcare, autonomous vehicles, facial recognition, AND military applications scores higher than one that only addresses data privacy — even if the latter has more total documents. Score = tags covered ÷ 77.

- **Maturity ratio (30% of this dimension):** Enacted documents ÷ total documents (enacted + proposed + defunct). A higher ratio means the governance pipeline converts proposals into law rather than getting stuck in perpetual deliberation.

**Why it gets only 5% weight:** This was originally 9% in our v2 methodology. After the data audit, we discovered that only **2 countries** (the US and China) are individually scoreable — AGORA's "Multinational" and "Other countries" categories can't be mapped to specific nations. Giving 9% weight to a dimension where 190+ countries have no data was unjustifiable. We reduced it to 5% so the weight is proportional to actual data utility. If AGORA expands coverage to more countries, this weight could increase without any methodology changes.

**Data coverage:** 2 countries (US: 390 enacted docs, China: 18).

---

## How the Final Score Is Calculated

### Why Not Just Average the 7 Dimensions?

A simple weighted average would mean that a country missing data for 3 dimensions would get zeros for those dimensions, dragging its score down — even though the zeros represent "we don't have data" rather than "this country has zero capability." A country could rank poorly not because it's weak, but because it's data-poor.

### The Coverage-Weighted Composite

Instead, we use a formula that adjusts for data availability:

```
AISI Score = Σ (dimension_score × dimension_weight × coverage)
             ─────────────────────────────────────────────────
                   Σ (dimension_weight × coverage)
```

Where `coverage` is 1 if the country has data for that dimension and 0 if not. In practice: if a country has data for 5 of 7 dimensions, it's scored on those 5, and the weights are renormalised to sum to 1.0 across just those 5. Its **data coverage** (0–1) is reported alongside its score so users know how much data backs the number.

### The Breadth Factor

After the initial scoring run, we discovered a perverse incentive: countries scoring on only 2–3 dimensions could outscore countries with 6–7 dimensions because the coverage-weighted formula gave them a higher per-dimension average (they were only scored on their best dimensions). To fix this, we added a breadth factor:

```
Final Score = Raw Score × (0.75 + 0.25 × dims_scored / 7)
```

A country scored on all 7 dimensions gets the full score. A country scored on only 2 gets a 21% penalty. This rewards breadth — a well-rounded AI ecosystem is more resilient than a narrow one.

### The Genuine-Zero Bug Fix

The most important bug we caught: countries absent from the semiconductor dataset were getting NaN for Hardware Sovereignty, which the formula treated as "missing data, ignore this dimension." But having zero semiconductor presence isn't missing data — it's a genuine zero. Same for collaboration and governance. We fixed this by setting these to 0 (not NaN) for countries where the data source exists but the country simply isn't in it. This moved India from #2 to #9 and Australia from #3 to #11 — both had been inflated by the formula ignoring their zero-hardware scores.

### All Scores Are Normalised to 0–100

Each dimension produces a 0–100 score using one of two normalisation methods:

- **Anchored normalisation:** Used when a natural reference exists (e.g., semiconductor market share is inherently 0–100%). These scores are comparable across different runs of the index.

- **Relative min-max normalisation:** Used when no natural anchor exists. The best country gets 100, the worst gets 0, everyone else is proportional. These scores are relative to the current cohort — if a new country with extreme values is added, existing scores shift.

The normalisation method is recorded per dimension so users know which scores are stable and which are cohort-relative.

---

## What the Score Does NOT Capture

- **Classified or undisclosed R&D** — military AI programs that don't publish or patent
- **Talent quality** — we measure quantity only; no measure of skill level or educational quality
- **Compute infrastructure** — cloud capacity, data centres, energy availability beyond semiconductors
- **Regulatory enforcement** — AGORA captures whether laws exist, not whether they're enforced
- **Post-2020 patent activity** — the 5-year data lag is real and significant
- **Non-English governance** — AGORA's US skew (91% of documents) means governance in non-English countries is severely undercounted
- **Physical production location** — HQ-based attribution means TSMC counts for Taiwan, not for the US/Japan where its fabs also operate
