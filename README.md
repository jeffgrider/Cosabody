# Cosabody™ ROI Calculator

Single-page interactive calculator that projects Cosabody's documented field performance onto a customer's operating numbers. Built for sales conversations, customer follow-ups, and self-serve scenarios.

---

## What it does

Three species (turkey, broiler, layer) on a single page, each with its own tab. The customer enters their baseline operating numbers; the calculator applies Cosabody's locked, documented performance lift and shows projected additional output. An optional price input enables a conservative dollar projection.

Each result box updates independently as inputs are filled. No "submit" button — projections appear live.

---

## File

- **`Cosabody_ROI_Calculator.html`** — single self-contained file (~120 KB)
- No external dependencies except Roboto from the Google Fonts CDN
- Optum Immunity logo inlined as base64
- Works in any modern browser (Chrome, Safari, Firefox, Edge)

---

## How to use and share

| Use case | How |
|---|---|
| Run locally | Double-click the HTML file |
| Host on the web | Drop into any static host (S3, Netlify, Vercel, GitHub Pages, internal server) — single file, just upload it |
| Email it | Send the file as an attachment; recipient opens in any browser |
| Embed | Works as an `<iframe>` target |
| Customer demo | Open in browser, click through species tabs, type in their numbers live |

The page is designed to fit a 720 px tall viewport without scrolling, which covers virtually any laptop screen with browser chrome accounted for.

---

## Locked performance lifts

These are baked into the calculator and reflect Cosabody's documented field performance versus control. Source data: 2M+ turkeys in field, 20M+ layers since 2023, every commercial broiler flock fed every time.

| Species | Livability lift | Feed conversion lift | Other |
|---|---|---|---|
| Turkey  | +6.5% | +5.5% | — |
| Broiler | +1.6% | +1.8% | — |
| Layer   | +1.6% | — | +1 dozen additional eggs per hen over productive life |

Lifts live in the `LIFTS` constant near the top of the script block. Search `var LIFTS =` to find them.

A fixed Cosabody inclusion rate of **0.5 lb per ton of finished feed** (0.025%) drives the usage calculation across all three species.

---

## Math

### Locked constants

```
LIVABILITY_LIFT  = 0.065 (turkey)  | 0.016 (broiler)  | 0.016 (layer)
FCR_LIFT         = 0.055 (turkey)  | 0.018 (broiler)  | n/a   (layer)
EXTRA_EGGS_PER_HEN = 12 (layer only, productive life)
INCLUSION_RATE   = 0.5 lb Cosabody per ton (2,000 lb) of finished feed
LIVABILITY_CEILING = 0.99 (1% biological headroom)
```

### Effective livability lift (cap)

To prevent biologically impossible projections, the documented lift is capped if the resulting livability would exceed 99%:

```
effectiveLivabilityLift(LIV/100, LIVABILITY_LIFT) =
  0                                           if LIV/100 ≥ 0.99 (no headroom)
  LIVABILITY_LIFT                             if (LIV/100) × (1 + LIVABILITY_LIFT) ≤ 0.99
  (0.99 / (LIV/100)) − 1                      otherwise (cap kicks in)
```

For typical operations the full documented lift applies. The cap only matters above ~93% (turkey) or ~97% (broiler/layer) baseline.

### Turkey and broiler

```
effLift              = effectiveLivabilityLift(livability/100, LIVABILITY_LIFT)
addBirds             = birds × (livability/100) × effLift
addWeight            = addBirds × marketWeight
cosaFlockLiveWeight  = birds × (livability/100) × (1 + effLift) × marketWeight
feedConsumedBaseline = birds × (livability/100) × marketWeight × baselineFCR
feedConsumedCosa     = cosaFlockLiveWeight × baselineFCR × (1 − FCR_LIFT)
cosabodyUsage (lb)   = feedConsumedCosa × 0.5 / 2000

topLineRevenue       = addWeight × pricePerLb
feedCostSaved ($)    = cosaFlockLiveWeight × baselineFCR × FCR_LIFT × feedCostPerTon / 2000
extraFeedNeeded ($)  = addWeight × baselineFCR × feedCostPerTon / 2000
productCost ($)      = cosabodyUsage × cosabodyCostPerLb
bottomLine ($)       = topLineRevenue + feedCostSaved − extraFeedNeeded − productCost
```

The bottom-line decomposition is shown directly in the calculator as an audit trail. Top-line revenue is gross. Bottom-line subtracts only Cosabody product cost and the net change in feed cost (extra feed needed minus FCR efficiency savings); it does NOT subtract incremental processing, placement, labor, or other variable costs that scale with output.

### Layer

```
effLift              = effectiveLivabilityLift(livability/100, LIVABILITY_LIFT)
addSurvivors         = pullets × (livability/100) × effLift
cosaSurvivors        = pullets × (livability/100) × (1 + effLift)
totalAddEggs         = addSurvivors × baselineEggsPerHen + cosaSurvivors × 12
totalAddDozens       = totalAddEggs / 12
cosaTotalDozens      = cosaSurvivors × (baselineEggsPerHen + 12) / 12
cosabodyUsage (lb)   = cosaTotalDozens × baselineFCR × 0.5 / 2000

feedCostPerDozen     = baselineFCR × feedCostPerTon / 2000   (identical Cosabody and baseline)
topLineRevenue       = totalAddDozens × pricePerDozen
feedCostExtraOutput  = totalAddDozens × baselineFCR × feedCostPerTon / 2000
productCost ($)      = cosabodyUsage × cosabodyCostPerLb
bottomLine ($)       = topLineRevenue − feedCostExtraOutput − productCost
```

Layer has no documented FCR lift, so feed cost per dozen is identical for Cosabody and baseline. The "Feed cost of extra output" is a real incremental spend (cost of producing the additional dozens at baseline FCR), not a saving.

### Input sanitization

All inputs are clamped at the calc layer:
- Negative values → clamped to 0 silently (treated as missing)
- Livability above 100 → clamped to 100 (cap then returns 0 effective lift)
- 0 is a valid feed cost (allows free-feed scenarios)

The HTML `min`/`max` attributes provide spinner-button bounds; the JS clamping is the authoritative validator and applies regardless of how the value was entered (typed, pasted, or spun).

---

## Box dependencies

Each result box only requires the inputs it actually depends on. Anything missing renders as an em-dash (—). This is intentional — customers see partial answers progressively as they fill in numbers, which makes the tool feel responsive and lets them stop early if they only want one figure.

| Box / row | Depends on |
|---|---|
| Additional birds to market (meat) | birds + livability |
| Additional live weight (meat) | birds + livability + weight |
| Feed cost saved (meat) | birds + livability + weight + FCR + feed cost |
| Top-line revenue impact (meat) | birds + livability + weight + price |
| Audit trail: Feed cost saved | birds + livability + weight + FCR + feed cost |
| Audit trail: Extra feed needed | birds + livability + weight + FCR + feed cost |
| Audit trail: Product cost | birds + livability + weight + FCR + Cosabody cost |
| Bottom-line impact (meat) | all of the above + price |
| Feed consumed (Cosabody/baseline) | birds + livability + weight + FCR |
| Additional hens through cycle (layer) | pullets + livability |
| Additional dozens of eggs (layer) | pullets + livability + eggs/hen |
| Feed cost of extra output (layer) | pullets + livability + eggs/hen + FCR + feed cost |
| Top-line revenue impact (layer) | pullets + livability + eggs/hen + price |
| Bottom-line impact (layer) | all of the above + Cosabody cost + feed cost |
| Feed cost per dozen (layer) | FCR + feed cost (hidden until both entered) |

---

## Inputs and placeholder hints

All baseline inputs start empty with italic ghost text showing a representative example. Calculations only respond to actual entries — the placeholder text is purely visual.

| Species | Field | Placeholder | Industry typical |
|---|---|---|---|
| Turkey  | Birds placed per year | e.g. 1,000,000 | — |
| Turkey  | Livability to harvest | e.g. 85 | 82–90% |
| Turkey  | Market weight, live | e.g. 38 lb | — |
| Turkey  | Feed conversion ratio | e.g. 2.50 | — |
| Broiler | Birds placed per year | e.g. 10,000,000 | — |
| Broiler | Livability to harvest | e.g. 95 | 93–97% |
| Broiler | Market weight, live | e.g. 6.5 lb | — |
| Broiler | Feed conversion ratio | e.g. 1.85 | — |
| Layer   | Pullets placed per cohort | e.g. 100,000 | — |
| Layer   | Livability through productive life | e.g. 90 | 87–93% |
| Layer   | Eggs per hen, productive life | e.g. 320 | — |
| Layer   | Feed conversion ratio | e.g. 3.50 | 3.0–4.0 lb / doz |
| All     | Cosabody cost | e.g. 8.00 | varies — confirm with sales |

The "Industry typical range" hints under livability inputs reinforce that the placeholder examples sit inside a sensible band.

---

## Brand and content rules baked in

These are conscious decisions, not accidents. If the calculator is ever forked or copied, preserve them.

- **"Real production data. Not pen trials."** — credibility wedge, preserved verbatim in the cred strip
- **Per-species cred line** uses each species' main brand color (turkey amber, broiler blue, layer green) for visual signature
- **Disclaimer** required: "Projected outcomes only. Results not guaranteed. No regulatory or therapeutic claims made or implied."
- **No unit economics** — no $/lb cost or sell price assumed beyond what the customer types in
- **No "Cosabody 2.0"** framing in any customer-facing surface
- **No real producer names** without confirmed reference rights. Mountaire is **not** cleared for external attribution. S&R Eggs / Schimpf is cleared.
- **No FDA / regulatory claims**, even by implication
- **No absolute percentage points** — only relative percentages
- **Conservative dollar projection** — yield gain only, feed savings excluded from $ figure
- **"Productive life"** is the correct term for the layer cohort timeframe. Do not use "lay cycle" — biologically that refers to the ~25-hour cycle for forming a single egg, not the bird's full laying period
- **No em-dashes** in body copy (commas instead)

---

## Brand system

Per-species CSS variables auto-swap when tabs are clicked. Defined on `body[data-species="..."]`:

| Token | Turkey | Broiler | Layer |
|---|---|---|---|
| `--species` | `#D89B2C` | `#1E88E5` | `#2D8659` |
| `--species-bg` | `#FEF7E4` | `#DDE9F8` | `#EAF5EC` |
| `--species-bright` | `#F4E3B8` | `#6EB6F5` | `#6EDA9A` |
| `--species-cred` | `#D89B2C` | `#6EB6F5` | `#6EDA9A` |

Static brand tokens:

- `--navy: #1E3A5F` (top brand bar)
- `--navy-deep: #142842` (cred strip, dollar projection box)
- `--ink: #1F2937` (body text)
- `--paper: #FFFFFF`
- `--muted: #4B5563` and `--muted-light: #6B7280` (secondary text — these were intentionally darkened from default greys for readability)

Typeface: Roboto, loaded from Google Fonts CDN.

---

## Where to update common content

Most edits land in one of a few places. Open the HTML in any text editor and search:

| To change… | Search for |
|---|---|
| The locked lift numbers | `var LIFTS =` |
| The cred-strip messaging per species | `cred-line-1` |
| The credibility wedge tagline | `Real production data` |
| The disclaimer | `class="disclaimer"` |
| Hint text under inputs | `class="hint"` |
| Head-to-head footnote | `class="head-to-head"` |
| Placeholder example values | `placeholder="e.g.` |
| Contact / address footer | `class="contact-line"` and `class="address-line"` |
| Per-species color tokens | `body[data-species=` |

---

## Layout notes

- Two-column calc grid: inputs on the left, projected impact stat boxes on the right
- The price input lives in the right column directly under the stat boxes — pairs with the dollar projection visually and balances column heights
- Two-column footer: closing CTA + Optum logo on the left, disclaimer on the right with a thin species-color vertical separator
- Centered contact / address strip below the footer grid
- Responsive: stacks to single column below 820 px wide

---

## Version

**v1.2** — May 2026

- Three species (turkey, broiler, layer) with locked lifts
- Empty inputs with placeholder example hints
- Dependency-aware result boxes (each updates independently)
- Two-column footer with centered contact strip
- Layer copy uses "productive life" throughout
- Layer view matches turkey/broiler with 4 inputs + 2×2 stat grid
- All currency stat boxes display dollar values (Feed cost saved / Feed cost of extra output)
- "Top-line revenue impact" replaces older "Projected revenue uplift" (clarifies gross vs. net)
- Cosabody product cost input + Feed cost per ton input
- **Bottom-line audit trail**: top-line + feed cost saved − extra feed needed − Cosabody product cost
- Layer-specific feed framing: shows Feed cost per dozen (identical Cosabody/baseline) when populated; hidden otherwise
- Livability cap at 99% (biological headroom); cap returns 0 effective lift when baseline ≥ 99%
- Input sanitization: livability clamped to 0–100; all inputs clamped to non-negative
- Cred lines reframed across all species: "These projections are based on…"

---

## Contact

For changes to performance numbers, brand language, or new species:

**sales@optumimmunity.com**
317.490.0754
www.optumimmunity.com

Optum Immunity™ • 700 Commercial Avenue • Waterloo, WI 53594
