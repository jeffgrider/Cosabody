Cosabody™ ROI Calculator
Single-page interactive calculator that projects Cosabody's documented field performance onto a customer's operating numbers. Built for sales conversations, customer follow-ups, and self-serve scenarios.

What it does
Three species (turkey, broiler, layer) on a single page, each with its own tab. The customer enters their baseline operating numbers; the calculator applies Cosabody's locked, documented performance lift and shows projected additional output. An optional price input enables a conservative dollar projection.
Each result box updates independently as inputs are filled. No "submit" button — projections appear live.

File

Cosabody_ROI_Calculator.html — single self-contained file (~120 KB)
No external dependencies except Roboto from the Google Fonts CDN
Optum Immunity logo inlined as base64
Works in any modern browser (Chrome, Safari, Firefox, Edge)


How to use and share
Use caseHowRun locallyDouble-click the HTML fileHost on the webDrop into any static host (S3, Netlify, Vercel, GitHub Pages, internal server) — single file, just upload itEmail itSend the file as an attachment; recipient opens in any browserEmbedWorks as an <iframe> targetCustomer demoOpen in browser, click through species tabs, type in their numbers live
The page is designed to fit a 720 px tall viewport without scrolling, which covers virtually any laptop screen with browser chrome accounted for.

Locked performance lifts
These are baked into the calculator and reflect Cosabody's documented field performance versus control. Source data: 2M+ turkeys in field, 20M+ layers since 2023, every commercial broiler flock fed every time.
SpeciesLivability liftFeed conversion liftOtherTurkey+6.5%+5.5%—Broiler+1.6%+1.8%—Layer+1.6%—+1 dozen additional eggs per hen over productive life
Lifts live in the LIFTS constant near the top of the script block. Search var LIFTS = to find them.

Math
Turkey and broiler
addBirds      = birds × (livability/100) × livabilityRel
addWeight     = addBirds × marketWeight
feedSavings   = birds × (livability/100) × (1 + livabilityRel) × marketWeight × baselineFCR × fcrRel
dollarUplift  = addWeight × pricePerLb        (only if price entered)
The dollar projection is intentionally conservative — yield gain only. Feed savings are excluded from the $ figure but shown separately so a customer can do the full math themselves if they want.
Layer
addSurvivors      = pullets × (livability/100) × livabilityRel
cosaSurvivors     = pullets × (livability/100) × (1 + livabilityRel)
totalAddEggs      = addSurvivors × baselineEggsPerHen + cosaSurvivors × 12
totalAddDozens    = totalAddEggs / 12
dollarUplift      = totalAddDozens × pricePerDozen     (only if price entered)
The +12 eggs benefit accrues to each surviving hen across her productive life, which is added to the eggs gained from the additional surviving hens themselves.

Box dependencies
Each result box only requires the inputs it actually depends on. Anything missing renders as an em-dash (—). This is intentional — customers see partial answers progressively as they fill in numbers, which makes the tool feel responsive and lets them stop early if they only want one figure.
BoxDepends onAdditional birds to marketbirds + livabilityAdditional live weightbirds + livability + weightFeed savingsbirds + livability + weight + FCRProjected revenue uplift (turkey/broiler)birds + livability + weight + priceAdditional hens through cycle (layer)pullets + livabilityAdditional dozens of eggs (layer)pullets + livability + eggs/henProjected revenue uplift (layer)pullets + livability + eggs/hen + price

Inputs and placeholder hints
All baseline inputs start empty with italic ghost text showing a representative example. Calculations only respond to actual entries — the placeholder text is purely visual.
SpeciesFieldPlaceholderIndustry typicalTurkeyBirds placed per yeare.g. 1,000,000—TurkeyLivability to harveste.g. 8582–90%TurkeyMarket weight, livee.g. 38 lb—TurkeyFeed conversion ratioe.g. 2.50—BroilerBirds placed per yeare.g. 10,000,000—BroilerLivability to harveste.g. 9593–97%BroilerMarket weight, livee.g. 6.5 lb—BroilerFeed conversion ratioe.g. 1.85—LayerPullets placed per cohorte.g. 100,000—LayerLivability through productive lifee.g. 9087–93%LayerEggs per hen, productive lifee.g. 320—
The "Industry typical range" hints under livability inputs reinforce that the placeholder examples sit inside a sensible band.

Brand and content rules baked in
These are conscious decisions, not accidents. If the calculator is ever forked or copied, preserve them.

"Real production data. Not pen trials." — credibility wedge, preserved verbatim in the cred strip
Per-species cred line uses each species' main brand color (turkey amber, broiler blue, layer green) for visual signature
Disclaimer required: "Projected outcomes only. Results not guaranteed. No regulatory or therapeutic claims made or implied."
No unit economics — no $/lb cost or sell price assumed beyond what the customer types in
No "Cosabody 2.0" framing in any customer-facing surface
No real producer names without confirmed reference rights. Mountaire is not cleared for external attribution. S&R Eggs / Schimpf is cleared.
No FDA / regulatory claims, even by implication
No absolute percentage points — only relative percentages
Conservative dollar projection — yield gain only, feed savings excluded from $ figure
"Productive life" is the correct term for the layer cohort timeframe. Do not use "lay cycle" — biologically that refers to the ~25-hour cycle for forming a single egg, not the bird's full laying period
No em-dashes in body copy (commas instead)


Brand system
Per-species CSS variables auto-swap when tabs are clicked. Defined on body[data-species="..."]:
TokenTurkeyBroilerLayer--species#D89B2C#1E88E5#2D8659--species-bg#FEF7E4#DDE9F8#EAF5EC--species-bright#F4E3B8#6EB6F5#6EDA9A--species-cred#D89B2C#6EB6F5#6EDA9A
Static brand tokens:

--navy: #1E3A5F (top brand bar)
--navy-deep: #142842 (cred strip, dollar projection box)
--ink: #1F2937 (body text)
--paper: #FFFFFF
--muted: #4B5563 and --muted-light: #6B7280 (secondary text — these were intentionally darkened from default greys for readability)

Typeface: Roboto, loaded from Google Fonts CDN.

Where to update common content
Most edits land in one of a few places. Open the HTML in any text editor and search:
To change…Search forThe locked lift numbersvar LIFTS =The cred-strip messaging per speciescred-line-1The credibility wedge taglineReal production dataThe disclaimerclass="disclaimer"Hint text under inputsclass="hint"Head-to-head footnoteclass="head-to-head"Placeholder example valuesplaceholder="e.g.Contact / address footerclass="contact-line" and class="address-line"Per-species color tokensbody[data-species=

Layout notes

Two-column calc grid: inputs on the left, projected impact stat boxes on the right
The price input lives in the right column directly under the stat boxes — pairs with the dollar projection visually and balances column heights
Two-column footer: closing CTA + Optum logo on the left, disclaimer on the right with a thin species-color vertical separator
Centered contact / address strip below the footer grid
Responsive: stacks to single column below 820 px wide


Version
v1.0 — May 2026

Three species (turkey, broiler, layer) with locked lifts
Empty inputs with placeholder example hints
Dependency-aware result boxes (each updates independently)
Two-column footer with centered contact strip
Layer copy uses "productive life" throughout


Contact
For changes to performance numbers, brand language, or new species:
sales@optumimmunity.com
317.490.0754
www.optumimmunity.com
Optum Immunity™ • 700 Commercial Avenue • Waterloo, WI 53594
