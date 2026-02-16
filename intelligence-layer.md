# Tendium Customer Graph — Intelligence Layer

## The 11-Star Experience

Applying Chesky's framework: imagine a 1-star through 11-star version of the Tendium Customer Graph. The 7-star version is the **minimum lovable product** — the floor beneath which people revert to spreadsheets within 2 weeks.

| Star | Experience | Status |
|------|-----------|--------|
| 1-star | Spreadsheet dump from 15+ systems. Hugo spends Monday mornings assembling data manually. | **Today** |
| 2-star | Single dashboard with charts per department. Better than spreadsheets, but still requires interpretation. Everyone draws different conclusions from the same charts. | Generic BI |
| 3-star | Unified customer entity view. One company, one card. But you still have to click around, compare, and figure out what it means. | Dashboard |
| 4-star | Department lenses on the entity. CS sees health, Sales sees pipeline, Marketing sees ICP fit. Data unified, but still read-only. | Current Compass scope |
| 5-star | Data quality is visible. Every metric shows its source and confidence level. Definitions are tracked (agreed/proposed/disputed). You trust the data because you can see why. | Compass + quality layer |
| 6-star | Cross-source signals highlighted. The compass flags when billing data contradicts usage data. You see compound signals (usage drop + negative call + renewal approaching) without manual assembly. | Compass + detection |
| **7-star** | **Punch cards. Before every call, before every Monday, before every QBR — the compass hands you a pre-built intelligence brief for your role. 10 seconds to scan what took 30 minutes to assemble. Powered by procurement data signals that exist nowhere else.** | **Ship this** |
| 8-star | The compass proactively tells you what to do. Not just "here's the data" but "call Bygg AB today — their usage dropped 40% and renewal is in 45 days. Here's what to say." Actions, not just information. | Compass + agents |
| 9-star | Each department gets a personalized daily action plan. CS sees their priority list ranked by urgency. Sales sees expansion opportunities ranked by likelihood. Marketing gets a prospect list from Ghost TAM. All generated overnight from the latest data. | Autonomous daily ops |
| 10-star | The compass learns from outcomes. When CS acts on a recommendation and the customer renews, that signal feeds back. The scoring model tightens. Patterns accumulate. The tool gets smarter every week. | Self-improving system |
| 11-star | The compass runs customer operations autonomously. It identifies risk, drafts and sends the right outreach, schedules the right meeting, and reports the outcome — across CS, Sales, and Marketing. The human approves or overrides, but the system does the work. | Autonomous customer ops |

---

## The 7-Star Product: Department Punch Cards

The 7-star version is the minimum that changes daily behavior. Below this, people revert to their spreadsheets.

### What makes it 7-star (not 4-star)

A 4-star product shows you data. A 7-star product **hands you the answer before you ask the question**.

The difference:
- **4-star**: Open the compass → navigate to a company → read 5 tabs → draw a conclusion → decide what to do
- **7-star**: Open the compass → see your role-specific brief already synthesized → scan for 10 seconds → act

The LLM is the bridge. It reads all the unified data and synthesizes it into **pre-built intelligence briefs tailored to each role's daily decision**. Not a chatbot. Not a query box. A punch card waiting for you when you arrive.

---

### Sales: The Customer Punch Card

*Before every call. Before every outreach. One page. 10 seconds to scan.*

**Existence proof: Jens's Sales Graph Explorer** — A working hack that already changed Sales workflows. Uses 90,000 structured award decisions (with 14,000 more in pipeline) to build a co-bidding graph: companies connected by bidding on the same procurements. Features: ranked leads by competitive overlap, company investigation by org number, competitive landscape scatter plots (win rate vs B2G revenue), and auto-generated Google Slides decks. Example output: Adlibris (82% win rate, 0.9 MSEK B2G) vs 5 competitors, head-to-head cards — all generated from procurement data. **The behavior change is the proof**: Sales actually adopted this tool. The Compass must preserve this capability while extending it from single-lens to whole-elephant.

**Sections:**

1. **Procurement Profile** — Bid volume (last 90 days), win rate, top sectors, payment trend
   - Source: Procurement Data
   - Why 0.1%: Nobody else can show this. A company bidding on 40% more tenders this quarter is GROWING — that's your upsell signal before any CRM flag.

2. **Current Relationship** — Plan tier, ARR, contract end date, expansion gap (difference between current spend and what peer companies pay)
   - Source: Younium
   - Table stakes, but contextualized by procurement profile

3. **Usage Snapshot** — Find active? BidFlow active? Intel active? Last active date. Platform: legacy or new.
   - Source: PostHog
   - Directional (tracking incomplete). Shows what they USE vs. what they PAY for.

4. **Peer Comparison** — "Companies like this one (same sector, same bid volume bracket) are typically on Scale tier and use Intel features."
   - Source: Procurement Data + Younium cross-reference
   - The killer section. "Your competitors use features you're not paying for."
   - **Already working in Sales Graph:** Jens's hack shows win rate vs B2G revenue scatter plots with co-bidding competitors. The Compass extends this with subscription + usage context.

5. **Recent Signals** — Last 30 days: usage trend (up/down/flat), support tickets (count + topics), email digest engagement (opens/clicks), call sentiment (if available)
   - Source: All sources combined
   - Cross-source compound signal — no single system shows this

6. **Suggested Talking Points** — LLM-generated from all of the above
   - Example: "Their bid volume grew 30% in construction tenders. They're on Find-only. Ask about Scale tier for team workflows — peer companies in construction at this bid volume have 70% Scale adoption."

**The question Sales asks:** "Give me the punch card for Bygg AB before my 2pm call."

---

### CS: The Health Brief

*Before a check-in. Before a QBR. When an account feels "off."*

**Sections:**

1. **Health Traffic Light** — Green/yellow/red — but EXPLAINED in plain language
   - Not a magic score. A narrative: "Yellow because: usage flat for 3 weeks, bid volume up 20% (they're busy winning tenders — good sign, but they might not be using Tendium features for those bids), and they haven't opened tender digest emails since Jan 28."
   - The procurement context changes the interpretation. Usage drop + bid volume drop = their business is shrinking (external churn risk). Usage drop + bid volume UP = they're busy but might be outgrowing current plan (expansion opportunity, not churn).

2. **Procurement Context** — Bid volume trend, win rate trend, sector activity
   - Why it matters: If a customer's procurement bid volume drops 50%, churn risk isn't about your product — it's about their business. CS should respond differently (empathetic check-in) vs. product-friction churn (fix the problem).

3. **Interaction Timeline** — Chronological, cross-source. Last 3 calls (date + sentiment), last 5 support tickets (date + topic), last email engagement, last product session. All on one timeline.
   - Today this lives in 4 different systems. The brief assembles it in one view.

4. **Renewal Forecast** — Days to renewal + risk factors + recommended timing for outreach
   - "Renewal in 62 days. Risk: medium. Recommended: schedule QBR in next 2 weeks to address declining Intel usage."

5. **"What would I say?"** — LLM drafts a check-in message that references REAL context
   - Example: "Hi [name], I noticed your team hasn't used the BidFlow features since January, but it looks like you've been winning more construction tenders recently — congratulations! Want to walk through how BidFlow can help manage those bids more efficiently?"

**The question CS asks:** "Which accounts need attention this week, and what should I say to each one?"

---

### Leadership: The Monday Compass

*One screen. State of the business. What to worry about. What to celebrate.*

**Sections:**

1. **Path to 100M** — Current ARR, monthly run rate, trajectory projection
   - "Current: 42M SEK subscription ARR (Younium). 1,376 active subs. Avg ACV ~30.5K SEK. NRR 110-120%. Need ~50-54M net new ARR (after NRR expansion). At avg ACV, ~1,640-1,770 new companies."
   - Total revenue is higher than subscription ARR (includes professional services, one-time fees — TBD from Younium).
   - Not a dashboard chart. A sentence that tells you if you're winning or losing.

2. **WAC by Market** — Weekly Active Companies trend, 4-week rolling, per market
   - Geographic priority: ~35% Sweden (saturating), 65% rest of Nordics. **Norway and Finland are the key expansion markets.** Hugo: "If we don't crack Norway and Finland, we fail."
   - 2026 targets: 67% revenue from new markets.
   - WAC is the leading indicator. If WAC grows, revenue follows. If WAC stalls, revenue will stall in 4-8 weeks.

3. **PCM Scorecard** — Paying Companies by Market with key ratios
   - WAC:PCM ratio = trial conversion health (are engaged companies paying?)
   - ARR:PCM = effective ACV (is expansion working?)
   - "Germany: 45 WAC, 15 PCM — conversion rate 33%. Sweden: 1,200 WAC, 950 PCM — conversion rate 79%. Germany conversion is healthy for early market."

4. **Top 3 Risks** — Compound signals, not single metrics
   - Example: "Nordea AB (120K ARR) — usage dropped 40% + negative call sentiment + contract renews in 45 days. CS flagged but no action taken yet."
   - Example: "Finnish WAC has been flat for 6 weeks. 8 trials started but 0 converted. Possible market-product fit issue."

5. **Top 3 Opportunities** — Procurement-powered
   - Example: "12 companies in DE procurement data match your SE top-10 customer profile. None are in pipeline yet."
   - Example: "3 Core customers showed 50%+ bid volume growth this quarter — Scale upsell candidates."

**The question Leadership asks:** "Where are we on the path to 100M, and what should I worry about this week?"

---

### Marketing: Segment Intelligence

*Before a campaign. Before ICP refinement. Before content planning.*

**Key questions and answers:**

1. **"Who should we target in Germany?"**
   - Answer: Companies in DE procurement data that match best-customer profile (bid volume >10/month, win rate >20%, sectors: construction + IT + healthcare) but aren't in Younium.
   - Output: Ranked prospect list with ICP fit score. Literal names and characteristics.
   - Why 0.1%: This prospect list comes from Tendium's own proprietary data. No other tool can generate it.

2. **"Which sectors convert best?"**
   - Answer: Conversion rate by procurement sector. "Construction companies convert at 12% (trial→paid), IT services at 4%, healthcare at 8%."
   - Procurement data segments by ACTUAL business activity, not self-reported industry tags in HubSpot.

3. **"What content should we create?"**
   - Answer: "Your best customers bid on [categories]. Companies in those categories that aren't customers search for [terms]. Create content about [topic] targeting [sector]."
   - Content strategy driven by actual procurement behavior, not keyword tools.

4. **"Is our ICP right?"**
   - Answer: "Your ICP says companies with 50+ employees in construction. But your highest-LTV customers actually have 15-30 employees, bid on 20+ tenders/month, and have a win rate above 25%. Your ICP is wrong — bid behavior predicts LTV better than company size."
   - Data-driven ICP challenge. The compass tells Marketing when their assumptions don't match reality.
   - **Hugo confirmed the ICP is broken in 5 ways:** (1) Too qualitative — "product complexity" can't be scored for prospecting, (2) Sales ignores it — workshops ran for weeks, salespeople don't use it, (3) SDR targeting contradicts it — Gustav says sell to salespeople in orgs but Marco prospects founders of small companies, (4) 4 departments have 4 different ICPs, (5) Spray-and-pray was necessary for PMF in the Nordics — now need to refine.
   - Hugo's instinct: "Can we have a library of definitions?" → The Compass's Library of Definitions feature answers this directly.
   - Strong presence in: technical consultants, MedTech, professional services + product companies.
   - Hubexo partnership blocks construction (only 5 large companies accessible).

**Already working in Sales Graph:** Jens's "Explore Leads" feature ranks non-customers by co-bidding overlap with existing customers — this IS the Ghost TAM surface. The Compass extends it with conversion data (who actually became a customer?) and content signals.

**The question Marketing asks:** "Show me the Ghost TAM — companies that should be our customers but don't know we exist."

---

### Product: Activation Intelligence

*Sprint planning. Feature prioritization. Platform migration tracking.*

**Key questions and answers:**

1. **"What predicts retention?"**
   - "Companies that save >10 tenders in their first week retain at 85%. Companies that don't retain at 30%. Find saved tenders = your activation moment."
   - Cross-reference with procurement: "Companies with high bid volume that DON'T activate on Find are the biggest missed opportunity — they have the need but aren't discovering the value."

2. **"New platform vs. legacy — how's migration going?"**
   - Side-by-side: activation rates, feature adoption, error rates, support ticket volume
   - "New platform activates 20% faster but has 3x more errors in BidFlow. Fix BidFlow on new platform = highest-impact sprint work."

3. **"What do churned customers have in common?"**
   - "70% of churns had <5 Find sessions in their first 30 days AND their procurement bid volume was declining. External factor (their business is shrinking), not product failure. For the other 30%: bid volume was growing but they never discovered BidFlow — activation failure, not demand failure."
   - Procurement data distinguishes "they left because we failed" from "they left because their business failed." No other product analytics tool can separate these.

4. **"Which features drive upsell?"**
   - "Companies that use Intel features upgrade to Scale 3x more often. But only 12% of Core customers discover Intel. Improving Intel discoverability has more upsell impact than building new features."

**The question Product asks:** "What should we build next to move WAC?"

---

### Hugo / C.Ops: BOWTIE Intelligence

*Pipeline operations. Conversion optimization. Revenue operations.*

**Key questions and answers:**

1. **"Where's the funnel leaking?"**
   - BOWTIE stage analysis with cross-source data. Key OKR metric: MQL → opportunity conversion rate (17% → 40% target). Previously "drove super hard creating as many MQLs as possible" → salespeople booked on irrelevant meetings. Quality > volume.
   - Distinguishes ICP-fit leaks from activation leaks. Different problems, different solutions.
   - Hugo confirmed: data fed into HubSpot WITHOUT utilization strategy. "We usually only have the data itself in mind and not how we use it."

2. **"Which trials should we push?"**
   - Ranked by: procurement bid volume (demand signal) + feature engagement (activation signal) + ICP fit score
   - "These 15 trials are high-intent (bid volume >20/month, Find active, ICP fit >70). These 40 are tourists (low bid volume, minimal engagement, poor ICP fit). Invest CS time in the 15."

3. **"What's our real TAM in Germany?"**
   - "47,000 companies in DE procurement data. 12,000 match our ICP. 200 are in pipeline. 15 are paying. TAM coverage: 0.12%. Addressable next quarter at current conversion rate: 180 new customers."

4. **"What's the quality of our pipeline?"**
   - "Current pipeline has 340 opportunities. Based on procurement data, 120 have strong ICP fit (bid volume + sector match). 220 are weak fits. Expected conversion: 40-50 new customers, not 120. Your pipeline is 3x inflated."

**The question Hugo asks:** "Show me pipeline quality — separate real opportunities from noise."

---

## The 0.1% Test

Why this can't be copied:

| Layer | What it is | Why it's defensible |
|-------|-----------|-------------------|
| **Procurement Data** | Bid volume, win rates, payment data, sector activity for every company in public procurement | Tendium's proprietary data. Not available in any CDP, CRM, or data provider. |
| **Customer IS prospect** | Every procurement entity is a potential customer. The gap between procurement data and Younium = literal TAM | Only Tendium can see both sides simultaneously |
| **Tender behavior as intent signal** | Bid volume growth = company expanding. Win rate changes = competitive pressure. Sector shifts = strategic pivot. | Leading indicators that exist nowhere else in SaaS |
| **B2G data quality** | Public procurement is regulated, transparent, structured. Data is inherently cleaner than any CRM input. | The quality advantage compounds — every model built on this data is more accurate |

Without procurement data, these punch cards are just "AI on CRM data" — Gong, Gainsight, and 50 startups do this. With procurement data, they contain intelligence that doesn't exist anywhere else in the world.

---

## Refined Tiers

### Tier 1: Punch Cards (MVP — Week 2-3)

Ship 3 punch cards: **Sales, CS, Leadership.** Pre-rendered, not conversational. Claude batch-generates them nightly from unified data. Each person opens the compass, sees their accounts with intelligence already synthesized.

- No chatbot. No query box. Just: here's what you need to know.
- Nightly batch via Modal: pull latest data from all sources → Claude generates punch cards per company per role → store as structured JSON → render in Compass UI
- Procurement data signals in every card — the 0.1% differentiator

**Win test:** "Did Hugo stop opening his spreadsheet on Monday morning?"

### Tier 2: Cross-Source Detection (Week 3-4)

Only alert on signals that **require multiple sources.** Any tool can flag a usage drop. Only the compass can flag compound signals.

**Specific detectors:**

- **Business health signal**: Procurement bid volume declining → the company itself may be struggling → churn risk is EXTERNAL, not your product's fault. CS responds differently (empathetic outreach) vs. product-friction churn (fix the issue).
- **Expansion readiness**: Bid volume growing + current plan is Find-only + peer companies at this level use Scale → upsell is natural, not pushy.
- **ICP mismatch alarm**: High churn in a segment where procurement data shows low bid activity → these companies don't actually need Tendium. Stop targeting them. Save Marketing spend.
- **Ghost TAM surfacing**: Companies in procurement data matching best-customer profiles that have never entered the pipeline. Net-new demand that Marketing doesn't know exists.

**Win test:** "Did a cross-source alert prevent a churn that single-source monitoring would have missed?"

### Tier 3: Procurement-Powered Actions (Week 4-5)

Actions that **only the compass can take** because they require procurement data:

- **Ghost TAM prospect lists**: Auto-generate target lists from procurement entities matching high-LTV customer profiles. Marketing gets a weekly "companies you should be talking to" list from Tendium's own data moat.
- **Contextual outreach drafts**: "We noticed you've been winning more construction tenders — companies like yours save 3 hours per bid with Scale." Based on REAL procurement data, not a mail merge template.
- **Renewal defense briefs**: 60 days before renewal, auto-generate a brief showing the customer their own ROI: "You used Tendium to find 47 tenders, bid on 23, and won 8 worth 12M SEK this year. Your ROI at current plan cost: 340x."
- **Pipeline quality scoring**: Score every opportunity against procurement data. "This prospect bids on 2 tenders/month — weak fit. This one bids on 40/month — strong fit." Sales prioritizes accordingly.

**Win test:** "Did a Ghost TAM list generate a real pipeline opportunity?"

### Tier 4: Proprietary Scoring Model (Week 5-6+)

The compass builds a **scoring model that gets smarter with every customer:**

- **ICP score from procurement behavior**: "Companies that bid on >10 tenders/month AND have a win rate >25% AND are in sectors [construction, IT, healthcare] convert at 15% and retain at 90%." Proprietary model that competitors cannot replicate.
- **Churn prediction model**: Weighted signals across all sources. Procurement bid volume is the leading indicator nobody else has. "When bid volume drops >20% AND usage drops >15% AND no CS interaction in 30 days → 75% churn probability within 90 days."
- **Feedback loops**: When CS acts on a recommendation, track the outcome. Did the account renew? Did the upsell land? Feed it back. The model tightens.
- **Pattern library**: Accumulated Tendium-specific patterns. "Nordic construction companies with growing bid volumes but declining Tendium usage are being poached by a competitor, not losing interest. The correct response is competitive displacement outreach, not a feature demo."

**Win test:** "Is the scoring model more accurate this month than last month?"

---

## Technical Architecture for the Intelligence Layer

```
NIGHTLY BATCH (Modal cron)
├── Pull latest: PostHog events, Younium status, Procurement Data
├── Run Claude: Generate punch cards per company per role
├── Run detectors: Cross-source signal matching
├── Store: Structured JSON → Compass API
└── Queue: Alerts for CS/Sales/Leadership

REAL-TIME QUERIES (Next.js API → Claude)
├── User asks question in Compass UI
├── API route sends context + tools to Claude
├── Claude calls: query_posthog(), get_company(), get_procurement()
├── Claude returns: narrative answer with evidence + sources
└── UI renders response with data quality indicators

FEEDBACK LOOP
├── User marks recommendation: acted / ignored / wrong
├── Outcome tracked: renewed / churned / upsold / no change
├── Signals fed back to scoring model
└── Pattern library updated
```

**Stack:**

| Component | Tool | Rationale |
|-----------|------|-----------|
| Reasoning engine | Claude API (Anthropic) | Best at nuanced multi-source analysis, tool use, long context. Can hold entire customer data in one prompt. |
| Batch processing | Modal.com | Nightly punch card generation, weekly pattern detection, bulk transcript analysis |
| Embeddings | Voyage AI | Semantic search across customer narratives, transcripts, tickets |
| Vector store | pgvector (Supabase) | Store embeddings for semantic retrieval. Supabase = Postgres, already familiar. |
| Agent framework | Claude tool use (native) | No framework needed. Claude's built-in tool use IS the agent. Tools: `query_posthog()`, `get_company()`, `get_procurement()`, `draft_message()`, `flag_risk()` |
| Orchestration | Next.js API routes (real-time) + Modal cron (batch) | Web-facing queries via API, batch processing via Modal |
| Storage | Structured JSON in Supabase | Punch cards, alerts, feedback, patterns |

---

## What to Ship in 6 Weeks

| Week | Tier | Deliverable | Win Test |
|------|------|-------------|----------|
| 2 | 1 | Sales punch card for top 20 accounts (Claude-generated from real data) | Sales rep reads it before a call and says "this is useful" |
| 2-3 | 1 | CS health brief for all active accounts | Hugo stops opening his Monday spreadsheet |
| 3 | 1 | Leadership Monday Compass (one screen, state of business) | Charlotte reads it Monday morning instead of asking for a report |
| 3-4 | 2 | Cross-source churn detector (usage + billing + procurement) | First compound alert that single-source monitoring would miss |
| 4 | 2 | Ghost TAM surface for Germany market | Marketing gets a prospect list they couldn't have generated otherwise |
| 4-5 | 3 | Renewal defense brief (auto-generated ROI per customer) | CS uses it in a real QBR |
| 5-6 | 3 | Pipeline quality scoring against procurement data | Sales can separate strong-fit from weak-fit opportunities |
| 6+ | 4 | First version of ICP scoring model from procurement patterns | Model predicts trial→customer conversion better than gut feel |

**The honest constraint:** Weeks 2-3 require real data from Younium + Procurement Data + PostHog. If those exports don't arrive, the punch cards run on sample data and the demo is impressive but not useful. The data exports are the critical path, not the LLM features.
