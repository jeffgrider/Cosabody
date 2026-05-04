# Cosabody™ ROI Calculator

Single-page interactive calculator that projects Cosabody™'s documented field performance onto a customer's operating numbers. Built for sales conversations, customer follow-ups, and self-serve scenarios.

---

## What it does

Three species (turkey, broiler, layer) on a single page, each with its own tab. The customer enters their baseline operating numbers; the calculator applies Cosabody™'s locked, documented performance lift and shows projected additional output. An optional price input enables a conservative dollar projection.

Each result box updates independently as inputs are filled. No "submit" button — projections appear live.

---

## File

- **`Cosabody_ROI_Calculator.html`** — single self-contained file (~140 KB)
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

The page is designed to fit a 720 px tall viewport without scrolling, which covers virtually any laptop screen with browser chrome accounted for. Layout is pixel-stable across all three species and all input states at browser widths of 1100 px and up; below that it degrades gracefully toward the 820 px responsive single-column breakpoint.

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
INCLUSION_RATE   = 0.5 lb Cosabody™ per ton (2,000 lb) of finished feed
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

The bottom-line decomposition is shown directly in the calculator as an audit trail. Top-line revenue is gross. Bottom-line subtracts only Cosabody™ product cost and the net change in feed cost (extra feed needed minus FCR efficiency savings); it does NOT subtract incremental processing, placement, labor, or other variable costs that scale with output.

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
| Audit trail: Top-line revenue | birds + livability + weight + price |
| Audit trail: Feed cost saved | birds + livability + weight + FCR + feed cost |
| Audit trail: Extra feed needed | birds + livability + weight + FCR + feed cost |
| Audit trail: Product cost | birds + livability + weight + FCR + Cosabody cost |
| Bottom-line impact (meat) | all of the above + price |
| Feed consumed (Cosabody/baseline) | birds + livability + weight + FCR |
| Additional hens (layer) | pullets + livability |
| Additional dozens of eggs (layer) | pullets + livability + eggs/hen |
| Feed cost of extra output (layer) | pullets + livability + eggs/hen + FCR + feed cost |
| Top-line revenue impact (layer) | pullets + livability + eggs/hen + price |
| Bottom-line impact (layer) | all of the above + Cosabody cost + feed cost |
| Feed cost per dozen (layer, fin-detail) | FCR + feed cost (hidden until both entered) |
| Eggs-to-dozens hint (layer, under eggs input) | always shows; example by default, populates with user's value |

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

Two static italic helper lines accompany inputs:
- Under each species' livability input: "Industry typical range: X–Y%"
- Under the layer eggs input: live eggs-to-dozens conversion. Default text "e.g. 320 eggs ≈ 26.7 dozen per hen"; populates with the user's value as they type, e.g. "300 eggs ≈ 25.0 dozen per hen"

---

## Layout stability

The calculator is designed to render without any vertical shift across species tabs or as inputs are filled. The cred strip and footer hold a fixed y-position regardless of state. Several CSS rules enforce this and should not be casually weakened:

- **`.stat-box { min-height: 95px }`** — reserves space for a 2-line stat-label so wrapping at narrow widths doesn't grow the row. Also matches the natural height of `.stat-value.large` (used by the dollar-box) so that toggling between dollar-placeholder and dollar-box doesn't shift the row by 4 px.
- **`.fin-detail { min-height: 32px }`** — reserves 2 lines for the species-specific detail line below the financials. Big dollar values can push turkey/broiler's "Feed consumed" line to wrap; reserved space prevents that wrap from shifting the cred strip.
- **`.financials { grid-template-columns: auto 1fr }`** — the financial inputs column takes its natural width (constrained by labels + 130 px input), and the outputs column gets all remaining space. Replaces an earlier `1fr 1.15fr` ratio that left outputs too narrow.
- **`.fin-row .fin-label`** and **`.fin-row .fin-value`** both have `white-space: nowrap` — prevents long dollar values from breaking inside the value span and forcing a 2-line row.
- **Layer audit trail phantom row** — between "Top-line revenue" and "Feed cost of extra output" there's a `visibility: hidden` `.fin-row` slot. Reserves the y-position where turkey/broiler's "Feed cost saved" line lives, keeping the bottom-line impact row at the same y-coordinate across species. Search `Spacer row` to find it.
- **Layer feed-cost-per-dozen line** — uses `visibility: hidden` (not `display: none`) so the slot is reserved before FCR + feed cost are entered. The JS toggle in `renderLayer` flips visibility, not display.

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
| The layer eggs-to-dozens hint logic | `eggsHint` |
| Head-to-head footnote | `class="head-to-head"` |
| Placeholder example values | `placeholder="e.g.` |
| The closing-CTA email contact | `class="closing-row"` |
| The footer website link | `class="contact-line"` |
| The footer address line | `class="address-line"` |
| Per-species color tokens | `body[data-species=` |
| Stat-box height reservation | `.stat-box {` (look for min-height) |
| Fin-detail height reservation | `.fin-detail {` (look for min-height) |
| Layer audit-trail phantom row | `Spacer row` |
| Layer feed-cost-per-dozen toggle | `l-feedcost-detail` |

---

## Layout notes

- Two-column calc grid: inputs on the left, projected impact stat boxes on the right
- The price input lives in the right column directly under the stat boxes — pairs with the dollar projection visually and balances column heights
- Two-column footer: closing CTA + Optum logo on the left, disclaimer on the right with a thin species-color vertical separator
- Centered website strip and address line below the footer grid (website at 18 px on top, address smaller underneath)
- Responsive: stacks to single column below 820 px wide
- Pixel-stable across species and fill states at widths ≥ 1100 px; minor wobble between 960 and 1100 with extreme stress values (e.g. tens of millions of birds); below 960 the 2-column zone is fragile and the layout heads toward the responsive break

---

## Version

**v1.3** — May 2026

Changes since v1.2:

- **Layer page layout stability** — cred strip and footer hold a fixed y-position across all species tabs and all input states at typical browser widths
  - Stat-box `min-height: 95px` reserves space for 2-line labels and matches the dollar-box's larger stat-value font, so price entry no longer shifts the row by 4 px
  - Fin-detail `min-height: 32px` reserves 2 lines for species-specific detail line below financials, so big-number wrapping doesn't shift the cred strip
  - Financials grid changed from `1fr 1.15fr` to `auto 1fr` so the outputs column gets all remaining width after inputs claim their natural width
  - `white-space: nowrap` added to `.fin-row .fin-label` and `.fin-row .fin-value` to prevent dollar-value wrapping
  - Layer audit trail gained a `visibility: hidden` phantom row in slot 2 (where turkey/broiler's "Feed cost saved" sits), so all three species' bottom-line impact lands at the same y-coordinate
  - Layer feed-cost-per-dozen line switched from `display: none` to `visibility: hidden` — reserves layout space when hidden
- **Layer stat-label rename** — "ADDITIONAL HENS THROUGH CYCLE" became "ADDITIONAL HENS" with sub "surviving, per cohort"; the original wording was both too long (wrapped at narrow widths) and used "cycle" too close to the deprecated "lay cycle"
- **Layer fin-detail copy shortened** — "Feed cost per dozen: $X / doz, same as baseline. More dozens at the same per-unit feed cost." became "Feed cost per dozen: $X / doz, identical to baseline." Educational point about Cosabody helping with quantity rather than per-unit cost is now lighter; if it needs to come back, place it in the head-to-head footnote rather than fin-detail
- **Eggs-to-dozens hint** — new always-on italic helper under the layer eggs input. Default text "e.g. 320 eggs ≈ 26.7 dozen per hen"; populates live as the user types
- **Footer contact website** — font size bumped from 14.5 px to 18 px so the URL reads as a deliberate anchor rather than fine print

Carrying forward from v1.2 (no change):

- Three species (turkey, broiler, layer) with locked lifts
- Empty inputs with placeholder example hints
- Dependency-aware result boxes (each updates independently)
- Two-column footer with centered website and address strip (website-only, no email or phone)
- Layer copy uses "productive life" throughout
- All currency stat boxes display dollar values
- Bottom-line audit trail with full decomposition
- Livability cap at 99% biological headroom
- Input sanitization (clamp 0–100 on livability, non-negative on all)

---

## Contact

For changes to performance numbers, brand language, or new species:

**sales@optumimmunity.com**
317.490.0754
www.optumimmunity.com

Optum Immunity™ • 700 Commercial Avenue • Waterloo, WI 53594
