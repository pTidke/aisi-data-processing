# AISI Score — How It Works

The AI Supremacy Index measures how **resilient** a country's AI position is — not just current activity, but how well it could withstand supply chain disruptions, sanctions, or talent shifts. Each country is scored 0–100 across 7 dimensions.

---

## The 7 Dimensions

### Hardware Sovereignty — 22%
**What:** How much of the semiconductor supply chain does a country control?
**How:** Combines average market share across chip-making inputs (35%), strongest position in any single input (30%), and what fraction of the supply chain the country participates in at all (35%).
**Why it's #1:** Without chips, nothing else matters. EUV lithography is 100% Netherlands, discrete GPUs are 100% US. This is the most acute geopolitical chokepoint in AI.

### Research Capacity — 20%
**What:** How much AI research is a country producing, how influential is it, and is it growing?
**How:** Average annual publications (35%), citations per paper using a 2021–2023 window (45%), and 5-year growth rate (20%). Publication counts are log-scaled because the US and China publish 1,000× more than the median country — without this adjustment, the score would only differentiate the top 5 nations.
**Key detail:** Citation data uses a deliberate 2-year lag because recent citations are incomplete in the source data.

### Commercial Ecosystem — 20%
**What:** Is there real private-sector depth — investment, companies, and corporate R&D?
**How:** Total estimated AI investment in $M (50%), number of tracked AI companies (30%), and average publications per company (20%). Uses estimated investment rather than disclosed to avoid undercounting countries with stronger deal-confidentiality norms.
**Caveat:** Company-level data (PARAT) only covers 17 countries. For the rest, the score relies on investment data alone.

### Innovation Output — 18%
**What:** Is a country turning research into protected intellectual property across diverse fields?
**How:** Total AI patent filings (45%) and number of distinct AI fields with active patents out of 11 (55%). Field diversity is weighted higher because broad patenting signals robust innovative capacity.
**Caveat:** Patent data is only complete through 2020 — a 5-year lag. This dimension measures pre-pandemic activity.

### Talent Base — 10%
**What:** How deep is the AI workforce?
**How:** Uses a tiered approach: AI workforce headcount if reliable data exists (Tier 1), corporate AI publications as a proxy if not (Tier 2), or company count as last resort (Tier 3). The tier used is recorded per country.
**Caveat:** Workforce data comes from LinkedIn and is only reliable for ~5 countries. LinkedIn is blocked in China and Russia. Countries without any corporate data are scored on the other 6 dimensions instead.

### Collaboration Network — 5%
**What:** How broadly and strategically does a country collaborate internationally on AI research?
**How:** Number of unique partner countries (40%), total co-authored papers weighted by topic importance — chip design counts 2×, LLMs 1.8×, AI safety 1.6× (35%), and collaboration volume weighted by partner quality (25%). Partner quality means collaborating with a top-ranked country contributes more than collaborating with a low-ranked one.
**Key detail:** Partner count is capped at 54 (the 95th percentile) to prevent the US at 79 partners from dominating this metric.

### Governance Readiness — 5%
**What:** Does a country have enacted AI laws, regulations, and standards that cover the policy landscape?
**How:** Number of enacted governance documents (30%), how many of 77 policy topics are covered (40%), and what fraction of proposed policies actually become law (30%).
**Caveat:** Currently only the US (390 enacted docs) and China (18) are individually scoreable. This dimension gets 5% weight because the data only covers 2 countries.

---

## How the Final Score Works

Scores are **not** a simple average. A country missing data for some dimensions isn't penalised as if it scored zero — it's scored on the dimensions where data exists, with weights adjusted accordingly. A **data coverage** percentage (0–100%) is reported alongside every score so you can see how much data backs the number.

A **breadth factor** slightly penalises countries scored on fewer dimensions, because a well-rounded AI ecosystem is more resilient than a narrow one.

---

## Why Some Numbers Might Surprise You

- **Scores are log-scaled** for publications, patents, and investment. Without this, the US and China would be at 100 and everyone else would be below 10.
- **Missing ≠ zero.** A country with no semiconductor data isn't scored zero on hardware — that dimension is excluded from its calculation entirely. But a country that *is* in the semiconductor dataset with no providers genuinely scores zero.
- **Data lags are real.** Patent data is 5 years behind. Citation data uses a deliberate 2-year lag. Governance data only covers 2 countries. The score is transparent about what it can and can't measure.
