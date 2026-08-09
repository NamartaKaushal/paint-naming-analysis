# Does Luxury Sell You a Feeling, Not a Color?
### A linguistic analysis of paint naming conventions, a generator built from the findings, and a Power BI dashboard

## Problem Statement
Does the language paint brands use to name their colors signal (or help
construct) a luxury/premium perception — independent of the paint itself?

This project has three parts:
1. **Analysis** — quantify how luxury and mass-market paint brands differ
   in their color-naming conventions, using real catalog data.
2. **Application** — use whatever naming pattern actually distinguishes
   luxury brands to generate new, on-brand color names for any input color.
3. **Dashboard** — a Power BI report visualizing the core findings.

## Method (Part 1: Analysis)
Built a dataset of 6,433 real paint colors across 4 brands, split into
two tiers:
- **Luxury** (n=425): Farrow & Ball, Little Greene
- **Mass-Market** (n=6,008): Behr, Sherwin-Williams

Engineered four naming features per color:
1. **Literal color word presence** — regex word-boundary match against
   a curated color-word list
2. **Name length** — word count and character count
3. **Vocabulary rarity** — average word frequency score via the
   [`wordfreq`](https://github.com/rspeer/wordfreq) library (lower = rarer/less common English word)
4. **Place-name references** — curated place/nationality word list,
   matched via regex (originally attempted with spaCy NER, but abandoned
   after finding ~50% false-positive rate on short product names —
   see Limitations)

## Key Findings

| Feature | Luxury (n=425) | Mass-Market (n=6,008) |
|---|---|---|
| Vocabulary rarity (avg score, lower = rarer) | 0.000101 | 0.000212 |
| Contains a real place reference | 5.2% | 2.2% |
| Contains a literal color word | 35.5% | 32.6% |
| Avg word count | 1.79 | 1.84 |

**Luxury brands don't use more literal color words or longer names than
mass-market brands** — those two features showed no reliable pattern.
What *does* hold up consistently: luxury names use **vocabulary roughly
half as common** as mass-market names, and reference **real places 2.3x
as often**. At the brand level, this forms a clean gradient from rarest
to most common vocabulary: Farrow & Ball → Little Greene →
Sherwin-Williams → Behr — no brand crosses out of its tier's ordering.

**Conclusion:** naming sophistication in luxury paint branding comes
from specific, measurable linguistic choices — rare vocabulary and place
references — not from a vague "sounds fancier" effect, and not from
simply avoiding literal color words or writing longer names.

## Application (Part 2: Color Name Generator)
Since the analysis found luxury brands specifically lean on **rare
vocabulary** and **real place references**, the second part of this
project builds a generator that applies those exact patterns: given any
color (extracted from an uploaded photo), it produces new name
suggestions styled after real luxury naming conventions, using a
character-level Markov chain trained on the actual Farrow & Ball /
Little Greene dataset.

- Correctly detects the color's hue family (a navy blue photo only ever
  gets "blue"-family names, never green or purple)
- Every generated name is checked against the real dataset to guarantee
  it's genuinely novel, not reproduced from training data
- Interactive photo-upload interface (`ipywidgets`) — no hex code
  knowledge required

## Dashboard (Part 3: Power BI)
A Power BI report visualizing the naming analysis findings: vocabulary
commonness by brand, place-word references by tier and brand, and
headline stats (6,433 colors analyzed; luxury brands reference places
2.32x as often as mass-market).

See `paint_analysis_dashboard.pbix` — open in Power BI Desktop to explore.

## Data Quality & Limitations
- Benjamin Moore was excluded after discovering systematic name
  truncation in the source data (e.g. "Paper Mache" recorded as "Paper"),
  confirmed against the brand's official Affinity collection naming.
- 1,744 duplicate Benjamin Moore names and 217 duplicate Behr names were
  found and removed prior to that brand's exclusion.
- Place-name detection uses a manually curated word list rather than a
  general NLP model, after spaCy's NER proved unreliable on short,
  context-free product names.
- The name generator uses a character-level Markov chain on a small
  (~230-name) per-hue-family training set — this produces plausible but
  occasionally imperfect results; a small curated "safe" word list
  guarantees at least one clean, recognizable result per generation.
- The generator's `ipywidgets` upload/generate interface is interactive
  and requires running the notebook locally in Jupyter — GitHub's
  notebook preview only renders a static snapshot and cannot run
  interactive elements.

## Data Sources
- Behr, Sherwin-Williams color catalogs: [colornerd](https://github.com/jpederson/colornerd)
- Farrow & Ball, Benjamin Moore reference data: [Paint Color HQ](https://www.paintcolorhq.com)
- Little Greene color catalog: [PaintDB](https://paintdb.com/brands/little-greene)
- Pricing references: brand websites and industry pricing guides, 2026

## Tech Stack
Python (pandas, wordfreq, ipywidgets, PIL), Excel, Power BI, DAX

## Running This
```bash
pip install -r requirements.txt
jupyter notebook paint_industry_analysis.ipynb
```
Run all cells in order top to bottom. When you reach the generator
section near the end, click **Upload**, select a photo, then click
**Generate Names**.

For the dashboard, open `paint_analysis_dashboard.pbix` in Power BI
Desktop.
