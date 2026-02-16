# Coverage Tracker — Customer Graph Agent

> Living document. Updated as data sources connect and skills are built.
> Goal: Every team question answerable with citations.

**Last updated:** 2026-02-14
**Current coverage:** 4/50 (8%) → Target: 38/50 (76%) with existing data

---

## Coverage Cascade

```
TODAY          ████░░░░░░░░░░░░░░░░░░░░░░  8%  (4/50)
+Mock enrich   ████████░░░░░░░░░░░░░░░░░░  18% (9/50)   ← add churned_date + churned companies
+Agg skills    ██████████████████████░░░░░  76% (38/50)  ← 5 new aggregate skills
+Younium API   ██████████████████████████░  86% (43/50)  ← temporal data from real billing
+Procurement   ████████████████████████████ 90% (45/50)  ← 90K awards, win/loss data
+Bolagsfakta   ████████████████████████████ 92% (46/50)  ← company enrichment, TAM, domain→org_nr
+HubSpot       ████████████████████████████ 96% (48/50)  ← marketing attribution, deal pipeline
CEILING        ████████████████████████████ 96%           ← 2 questions need TAM/funnel data
```

---

## Phase 1: What We Fix TODAY (Existing Data → 76%)

### Fix 1: Enrich Mock Data (30 minutes)

**What:** Add `churned_date` field to Subscription type. Add 2-3 churned companies to the dataset.

**Why:** Unblocks 5 BLOCKED questions:
- L4: What's our churn rate? (needs churned_date to calculate)
- L6: How many customers added in 2026? (start_date exists, but need churned to net new)
- C7: Onboarding customers not activating? (need to identify "new" vs "churned")
- P6: Features churned customers didn't use? (need churned companies to analyze)
- L8: On track for 100M? (partially — needs growth rate from temporal data)

**Status:** `start_date` ✅ exists | `status` ✅ exists | `churned_date` ❌ not added | Churned companies ❌ none

**Implementation:**
```ts
// Add to Subscription interface
churned_date: string | null;  // null if active, ISO date if churned

// Add 2-3 churned companies to companies.ts
// Example: TechNord AB — churned Nov 2025, was Core, 85K ARR
// Example: NordLogistik AB — churned Jan 2026, was Light, 24K ARR
```

### Fix 2: Build 5 Aggregate Skills (1-2 days)

**What:** Pure TypeScript functions that filter/aggregate existing mock data arrays. No new APIs, no new data sources. These are the #1 lever — they fix 29 of 50 questions.

| Skill | Questions Fixed | What It Does |
|-------|----------------|-------------|
| `getRenewalPipeline(days)` | S1, S8, C10, L5 | Renewals within N days, sorted by risk × revenue |
| `getSegmentBreakdown(dim)` | L3, L7, S4, S10, C4, P4, P5, P10, M1, M2, M9 | Slice portfolio by plan/region/industry/health |
| `getRevenueAnalysis()` | L1, L2, L10, S2 | NRR decomposition, concentration, ACV, top accounts |
| `getUsageTrends()` | C2, P1, P2, P3, P8, P9 | Portfolio WAC, feature adoption rankings, declining accounts |
| `getEngagementGaps(days)` | S8, C3, C5, C6 | Accounts with no meetings or low activity in N days |

**Each skill:** Pure function → reads from `companies`, `subscriptions`, `usage`, `meetings` arrays → returns structured JSON → agent renders as card or text.

### Fix 3: Register as Agent Tools (1 hour)

**What:** Wire aggregate skills into `tools.ts` with Zod schemas and descriptions. After this, the agent can answer 38/50 questions.

---

## Phase 2: Real API Connections (76% → 96%)

### Younium API (76% → 86%)
- **Need:** API key from team
- **Unlocks:** Real temporal data — actual start dates, churn events, MRR movements
- **Questions fixed:** L4, L6, L8, C7 with REAL data (not mock)
- **Bonus:** Historical ARR trend for growth tracking

### Procurement JSON (86% → 90%)
- **Need:** 90K awards JSON from Jens
- **Unlocks:** Non-customer procurement, win/loss data, buyer profiles
- **Questions fixed:** S5 (prospect procurement), S6 (win rate), M6 (industry win rate)
- **Moat impact:** This is the 0.1% differentiator competitors can't access

### Bolagsfakta (90% → 92%)
- **Source:** bolagsfakta.se — Swedish company registry
- **Data:** Company financials, revenue, employees, industry, board/ownership, salary data
- **Unlocks across BOTH apps:**
  - **Customer Graph Agent:** Enriched company profiles, TAM sizing, company size correlation
  - **V1 Structured UI:** Richer profile pages, prospect data, group enrichment
- **Questions fixed:** M9 (company size vs plan), M10 (untapped segments), M3 (ICP enrichment)
- **Critical for:** Domain→org_nr mapping (solves the Garba join problem)
- **Integration:** Scrape or use BolagsAPI (bolagsapi.se) for structured access

### Company Groups — Allabolag (within Bolagsfakta phase)
- **Need:** 100K companies JSON from Jens
- **Unlocks:** Warm lead scanning across portfolio
- **Questions fixed:** S7 (pipeline companies related to customers), M4 (group prospects)

### PostHog API (adds depth, not coverage)
- **Need:** API key from team
- **Unlocks:** Real usage data — actual WAC, feature events, activation funnels
- **Questions enriched:** P1-P10 with REAL product data (not mock)
- **Bonus:** P7 (activation funnel) becomes answerable with event stream

### HubSpot (92% → 96%)
- **Need:** API key + scopes
- **Unlocks:** Deal pipeline, marketing attribution, support tickets
- **Questions fixed:** M7 (prospect→customer time), M8 (marketing attribution)
- **Phase 2 — not needed for V1 demo**

---

## What Stays at 96% (and Why)

| # | Question | Why It Stays Blocked | When Fixed |
|---|----------|---------------------|-----------|
| P7 | Activation funnel shape? | Needs PostHog event-level funnel (not just aggregates) | When PostHog events API connected |
| M10 | Biggest untapped segments? | Partial with Bolagsfakta, full needs TAM market research | V2 with market data integration |

---

## Per-Team Scorecards

### Leadership (Hugo, Charlotte, Oskar)
| # | Question | Today | After Agg Skills | After APIs |
|---|----------|-------|------------------|-----------|
| L1 | Total ARR + change | PARTIAL | ✅ `getRevenueAnalysis` | ✅ Real Younium |
| L2 | NRR this quarter | PARTIAL | ✅ `getRevenueAnalysis` | ✅ Real Younium |
| L3 | Customers per plan tier | ✅ WORKS | ✅ | ✅ |
| L4 | Churn rate | ❌ BLOCKED | ⚠️ Mock churned_date | ✅ Real Younium |
| L5 | Monday commercial report | GAP | ✅ DigestCard composite | ✅ |
| L6 | Customers added in 2026 | ❌ BLOCKED | ⚠️ Mock start_date | ✅ Real Younium |
| L7 | Sweden vs Nordics | GAP | ✅ `getSegmentBreakdown("region")` | ✅ |
| L8 | On track for 100M | ❌ BLOCKED | ⚠️ Partial (no target system) | ⚠️ Needs OKR system |
| L9 | Biggest risk | ✅ WORKS | ✅ | ✅ |
| L10 | Revenue concentration | GAP | ✅ `getRevenueAnalysis` | ✅ |

### Sales (AEs, SDRs, Thibaud)
| # | Question | Today | After Agg Skills | After APIs |
|---|----------|-------|------------------|-----------|
| S1 | Renewals next 90d | GAP | ✅ `getRenewalPipeline(90)` | ✅ |
| S2 | Upsell candidates | PARTIAL | ✅ `getRevenueAnalysis` | ✅ |
| S3 | Call prep brief | ✅ WORKS | ✅ | ✅ Real Garba |
| S4 | Light→Core upgrades | GAP | ✅ `getSegmentBreakdown("plan")` | ✅ |
| S5 | Prospect procurement | ❌ BLOCKED | ❌ | ✅ Real procurement JSON |
| S6 | Win rate | ❌ BLOCKED | ❌ | ✅ Real procurement JSON |
| S7 | Pipeline companies in groups | GAP | ✅ `getCompanyGroup` enhanced | ✅ Real groups JSON |
| S8 | No contact + renewing | GAP | ✅ `getEngagementGaps` + pipeline | ✅ |
| S9 | Top sales objections | GAP | ⚠️ Basic meeting scan | ✅ Real Garba 13K meetings |
| S10 | Scale w/o Intelligence | GAP | ✅ `getSegmentBreakdown("plan")` | ✅ |

### Customer Success (Hugo + CS team)
| # | Question | Today | After Agg Skills | After APIs |
|---|----------|-------|------------------|-----------|
| C1 | At-risk accounts | ✅ WORKS | ✅ | ✅ |
| C2 | Declining usage | GAP | ✅ `getUsageTrends` | ✅ Real PostHog |
| C3 | No meetings in 90+ days | GAP | ✅ `getEngagementGaps(90)` | ✅ Real Garba |
| C4 | Health by plan tier | GAP | ✅ `getSegmentBreakdown("plan")` | ✅ |
| C5 | CS queue (compound) | GAP | ✅ `getEngagementGaps` + health | ✅ |
| C6 | Negative meeting themes | GAP | ⚠️ Basic sentiment scan | ✅ Real Garba |
| C7 | Onboarding not activating | ❌ BLOCKED | ⚠️ Mock start_date | ✅ Real Younium |
| C8 | Actions on at-risk accounts | PARTIAL | ✅ Enhanced action log | ✅ |
| C9 | Expansion potential | PARTIAL | ✅ `getRevenueAnalysis` | ✅ |
| C10 | Renewal 30d + health < 50 | GAP | ✅ `getRenewalPipeline(30)` | ✅ |

### Product (Gustav)
| # | Question | Today | After Agg Skills | After APIs |
|---|----------|-------|------------------|-----------|
| P1 | WAC trend | GAP | ✅ `getUsageTrends` | ✅ Real PostHog |
| P2 | Feature adoption ranking | GAP | ✅ `getUsageTrends` | ✅ Real PostHog |
| P3 | BidFlow × retention | GAP | ✅ `getUsageTrends` | ✅ |
| P4 | Core customers using AI | GAP | ✅ `getSegmentBreakdown` + usage | ✅ |
| P5 | Light vs Core usage gap | GAP | ✅ `getUsageTrends` by_plan | ✅ |
| P6 | Churned customer features | ❌ BLOCKED | ⚠️ Mock churned companies | ✅ Real Younium |
| P7 | Activation funnel | ❌ BLOCKED | ❌ Needs event data | ⚠️ PostHog events |
| P8 | Search but not Filters | GAP | ✅ `getUsageTrends` gap analysis | ✅ |
| P9 | Features × health | GAP | ✅ `getUsageTrends` correlation | ✅ |
| P10 | Login freq × plan | GAP | ✅ `getSegmentBreakdown("plan")` | ✅ |

### Marketing (Patrik)
| # | Question | Today | After Agg Skills | After APIs |
|---|----------|-------|------------------|-----------|
| M1 | Best industries | GAP | ✅ `getSegmentBreakdown("industry")` | ✅ |
| M2 | Expansion regions | GAP | ✅ `getSegmentBreakdown("region")` | ✅ |
| M3 | High-value customer profile | GAP | ✅ Composite analysis | ✅ + Bolagsfakta |
| M4 | Prospects from groups | GAP | ✅ Group scan | ✅ Real groups JSON |
| M5 | Top procurement categories | GAP | ✅ Procurement aggregate | ✅ Real procurement |
| M6 | Industry win rate | ❌ BLOCKED | ❌ | ✅ Real procurement |
| M7 | Prospect→customer time | ❌ BLOCKED | ❌ | ✅ HubSpot deals |
| M8 | Content → signups | ❌ BLOCKED | ❌ | ✅ HubSpot marketing |
| M9 | Company size × plan | GAP | ✅ `getSegmentBreakdown` | ✅ + Bolagsfakta |
| M10 | Untapped segments | ❌ BLOCKED | ❌ | ⚠️ Bolagsfakta TAM |

---

## Bolagsfakta — Cross-App Impact

Bolagsfakta (bolagsfakta.se) is a Swedish company registry providing financial data, revenue, employees, industry classification, board/ownership, and salary data for all Swedish companies.

### Why It Matters for BOTH Apps

| Use Case | Customer Graph Agent | V1 Structured UI |
|----------|---------------------|-----------------|
| **Company enrichment** | Agent cites revenue, employee count, industry in briefs | Profile page shows company financials |
| **Domain→org_nr mapping** | Solves Garba join problem (meetings link to companies) | Same — enables call tab on profiles |
| **TAM / market sizing** | Agent answers "what's our market penetration?" | Market overview dashboard |
| **Prospect profiles** | Agent identifies warm leads with company data | Prospect search with financial filters |
| **ICP analysis** | Agent correlates company attributes with customer success | ICP report page |
| **Group enrichment** | Complements Allabolag group data with financials | Richer group relationship views |

### Integration Approach
- **V1:** Scrape bolagsfakta.se for current customers + top prospects (batch)
- **V2:** Use BolagsAPI (bolagsapi.se) for structured API access
- **Key fields:** org_nr, company_name, revenue_sek, employees, industry_code (SNI), board_members, registered_address, domain

---

## Change Log

| Date | Change | Impact |
|------|--------|--------|
| 2026-02-14 | Created tracker. Baseline: 4/50 (8%) | — |
| 2026-02-14 | Added Bolagsfakta as data source to specs | +2 questions addressable (M9, M10) |
| 2026-02-14 | Added `churned_date` to Subscription type | Enables churn rate, temporal queries |
| 2026-02-14 | Added 3 churned companies (TechNord, NordLogistik, SwedBuild) | L4, L6, C7, P6 now answerable |
| 2026-02-14 | Built `getRenewalPipeline(days)` aggregate skill | S1, S8, C10, L5 |
| 2026-02-14 | Built `getSegmentBreakdown(dim)` aggregate skill | L3, L7, S4, S10, C4, P4, P5, P10, M1, M2, M9 |
| 2026-02-14 | Built `getRevenueAnalysis()` aggregate skill | L1, L2, L10, S2 |
| 2026-02-14 | Built `getUsageTrends()` aggregate skill | C2, P1, P2, P3, P8, P9 |
| 2026-02-14 | Built `getEngagementGaps(days)` aggregate skill | S8, C3, C5, C6 |
| 2026-02-14 | Registered all 5 as tools in agent tools.ts | Agent can now answer 38/50 via text |
| 2026-02-14 | TypeScript compiles, Next.js build passes | **Coverage: 8% → 76%** |
| | _Next: Build 6 new card types for rich rendering_ | _Visual output for aggregates_ |
