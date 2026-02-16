# Cards & Aggregated Data — Feature Spec

> Customer Graph Agent: How every question becomes a card.

**Version:** 2.0 | **Date:** 2026-02-13 | **Author:** John Tan

---

## Principle

The Customer Graph has no pages, no dashboards, no navigation. Every answer is a **card** rendered inside the conversation. Leadership dashboards are just pinned cards. Scheduled reports are just cards delivered to Slack.

**One surface. Cards as the atomic unit.**

---

## Card Architecture

### 4 Archetypes

Every card the agent renders fits one of four archetypes. New questions don't require new card types — just new data piped into existing archetypes.

| Archetype | Purpose | When to use | Example |
|-----------|---------|-------------|---------|
| **Entity Card** | Deep view of one thing | "Tell me about Adlibris" | CompanyCard, CallPrepCard |
| **List Card** | Ranked/filtered set of items | "Which accounts are at risk?" | RiskCard, SearchResultsCard, RenewalList |
| **Brief Card** | Multi-section synthesis | "Give me the Monday report" | PortfolioCard, DigestCard |
| **Insight Card** | One key finding + evidence | Proactive cross-reference | "Adlibris usage dropped 40% but renewal is in 45 days" |

### Shared Card Components (Built)

- **MetricCell** — label + value + trend arrow + sub-text
- **Plan badge** — colored pill (Light=gray, Core=blue, Scale=purple)
- **Health dot** — green/yellow/red status indicator
- **formatSEK()** — consistent currency formatting (42K, 1.2M, etc.)

---

## Existing Cards (Built)

| # | Card | Archetype | Trigger Tool | What It Shows |
|---|------|-----------|-------------|---------------|
| 1 | **CompanyCard** | Entity | `get_customer_profile` | Name, plan, health, ARR, renewal, WAC, seats, industry, meetings, procurement |
| 2 | **PortfolioCard** | Brief | `get_portfolio_overview` | Total ARR, NRR, at-risk count, expansion count, plan mix bar, at-risk rows |
| 3 | **RiskCard** | List | `detect_churn_risks` | Flagged accounts by severity, health score, ARR, risk signal pills |
| 4 | **SearchResultsCard** | List | `search_customers` | Matching companies with plan, industry, ARR, usage trend |
| 5 | **CallPrepCard** | Entity | `generate_call_prep` | Company brief, plan, ARR, last meeting, talking points, watch outs |
| 6 | **ActionCard** | Entity | Action tools | Confirmation card for Slack/task/email/flag/meeting actions |

---

## 50-Question Stress Test (10 per Team)

### Grading Scale
- **WORKS** = Existing skill + card handles this fully
- **PARTIAL** = Data exists but no dedicated skill/card; agent can hack an answer
- **GAP** = Data exists in mock but no skill to query it at aggregate level
- **BLOCKED** = Data doesn't exist in mock; needs new data fields or external source

---

### TEAM 1: Leadership (Hugo, Charlotte, Oskar)

These are the "Monday morning" and "board meeting" questions. They want portfolio-level numbers, trends, and strategic health indicators.

| # | Question | Data Needed | Skill Today | Card Today | Verdict |
|---|----------|-------------|-------------|------------|---------|
| L1 | "What's our total ARR and how has it changed?" | Total ARR + delta from previous period | `get_portfolio_overview` (has total ARR) | PortfolioCard | **PARTIAL** — has current ARR, no historical trend |
| L2 | "What's NRR this quarter?" | NRR decomposition: expansion, contraction, churn | `get_portfolio_overview` (has NRR %) | PortfolioCard | **PARTIAL** — has NRR number, no decomposition |
| L3 | "How many customers on each plan tier?" | Count by plan | `get_portfolio_overview` (has plan mix) | PortfolioCard | **WORKS** |
| L4 | "What's our churn rate this quarter?" | Churned count + revenue over period | None | None | **BLOCKED** — no `churned_date` or temporal events |
| L5 | "Give me the Monday commercial report" | All aggregates combined | Multiple skills needed | No DigestCard | **GAP** — no single "digest" skill or card |
| L6 | "How many customers did we add in 2026?" | Customer start dates | None | None | **BLOCKED** — no `start_date` on subscriptions |
| L7 | "How does Sweden compare to rest of Nordics?" | Regional breakdown: ARR, count, health | None at aggregate level | None | **GAP** — data exists per-company, no segment query |
| L8 | "Are we on track for 100M SEK?" | Current ARR + growth rate + projection | None | None | **BLOCKED** — no historical ARR, no target tracking |
| L9 | "What's our biggest risk right now?" | Risk-ranked accounts by ARR impact | `detect_churn_risks` | RiskCard | **WORKS** — but card doesn't highlight THE biggest |
| L10 | "How concentrated is our revenue in top accounts?" | Revenue ranking, top-10 %, HHI | None | None | **GAP** — ARR exists per-company, no concentration analysis |

**Leadership Score: 2 WORKS, 2 PARTIAL, 3 GAP, 3 BLOCKED**

---

### TEAM 2: Sales (AEs, SDRs, Thibaud)

These are "who should I call" and "prepare me for meetings" questions. They want targets, briefs, and competitive intel.

| # | Question | Data Needed | Skill Today | Card Today | Verdict |
|---|----------|-------------|-------------|------------|---------|
| S1 | "Who's renewing in the next 90 days?" | Renewals sorted by date, ARR, risk | None at aggregate | None | **GAP** — renewal dates exist, no pipeline query |
| S2 | "Which accounts can we upsell?" | Expansion signals: high usage + low plan | `get_portfolio_overview` (lists expansion opps) | PortfolioCard (partial) | **PARTIAL** — listed in portfolio but not rich |
| S3 | "Brief me for my call with Stockholm Municipality" | Full profile + talking points | `generate_call_prep` | CallPrepCard | **WORKS** |
| S4 | "Which Light customers should upgrade to Core?" | Light plans + high usage + not using Core features | None | None | **GAP** — need segment filter with conditions |
| S5 | "Show me prospects with the most procurement activity" | Procurement volume by non-customer companies | None | None | **BLOCKED** — mock data only has procurement for customers, not prospects |
| S6 | "What's our procurement win rate?" | Awards vs. total bids | None | None | **BLOCKED** — no bid/loss data, only awards |
| S7 | "Which companies in pipeline are related to existing customers?" | Company group x non-customer overlap | `get_company_group` (per-company) | None | **GAP** — works per-company, no portfolio-wide group scan |
| S8 | "Who haven't we talked to in 60 days that's up for renewal?" | Meeting recency x renewal proximity | None | None | **GAP** — both data points exist, no compound query |
| S9 | "What are the top objections from recent sales calls?" | Meeting summaries + MEDDIC insights across accounts | None | None | **GAP** — meeting data exists, no cross-portfolio text analysis |
| S10 | "Which Scale customers don't have Intelligence?" | Scale plan + intelligence_addon = false | None | None | **GAP** — data exists, no filtered aggregate query |

**Sales Score: 1 WORKS, 1 PARTIAL, 6 GAP, 2 BLOCKED**

---

### TEAM 3: Customer Success

These are "who needs help" and "how's engagement" questions. They want health monitoring, proactive alerts, and engagement tracking.

| # | Question | Data Needed | Skill Today | Card Today | Verdict |
|---|----------|-------------|-------------|------------|---------|
| C1 | "Which accounts are at risk?" | Health-sorted risk scan | `detect_churn_risks` | RiskCard | **WORKS** |
| C2 | "Who has declining usage this month?" | WAC trend = "down" across portfolio | None | None | **GAP** — wac_trend exists per-company, no aggregate filter |
| C3 | "Which customers haven't had a meeting in 90+ days?" | Last meeting date per company | None | None | **GAP** — meeting dates exist, no engagement gap scan |
| C4 | "What's the average health score by plan tier?" | Health x plan aggregation | None | None | **GAP** — both data points exist per-company, no cross-cut |
| C5 | "Show me all accounts I need to check in on" | Compound: low health OR no recent meeting OR approaching renewal | None | None | **GAP** — needs personalized CS queue combining signals |
| C6 | "What themes come up in negative meetings?" | Negative-sentiment meetings, summaries aggregated | None | None | **GAP** — meeting sentiment + summaries exist, no text aggregation |
| C7 | "Which onboarding customers aren't activating?" | New customers (recent start) with low usage | None | None | **BLOCKED** — no `start_date` to identify "new" customers |
| C8 | "What actions have been taken on at-risk accounts?" | Action log filtered by company + date | `get_action_log` | None (raw JSON) | **PARTIAL** — log exists but no filtering or card |
| C9 | "Which accounts have the best expansion potential?" | Same as S2 but from CS perspective | `get_portfolio_overview` | PortfolioCard (partial) | **PARTIAL** — listed but not CS-tailored |
| C10 | "Who has renewal in 30d with health under 50?" | Compound: renewal proximity x health threshold | None | None | **GAP** — both fields exist, no compound query |

**CS Score: 1 WORKS, 2 PARTIAL, 6 GAP, 1 BLOCKED**

---

### TEAM 4: Product (Gustav)

These are "what are customers doing in the product" questions. They want usage patterns, feature adoption, and activation funnels.

| # | Question | Data Needed | Skill Today | Card Today | Verdict |
|---|----------|-------------|-------------|------------|---------|
| P1 | "What's our WAC trend?" | Portfolio-wide WAC current + previous | None | None | **GAP** — per-company WAC exists, no aggregate |
| P2 | "Which features have the highest adoption?" | Features_used across all companies, ranked | None | None | **GAP** — features_used exists per-company, no aggregate |
| P3 | "How does BidFlow usage correlate with retention?" | BidFlow adoption x health score | None | None | **GAP** — both exist per-company, no correlation |
| P4 | "What % of Core customers use Ask Tendium AI?" | Feature x plan cross-tabulation | None | None | **GAP** — both exist per-company, no cross-tab |
| P5 | "What's the usage gap between Light and Core?" | WAC/logins/features by plan tier | None | None | **GAP** — usage data exists, no segment comparison |
| P6 | "Which features do churned customers NOT use?" | Churned accounts x features_used | None | None | **BLOCKED** — no churn status to identify churned |
| P7 | "What does the activation funnel look like?" | Signup → first login → first search → filter set → BidFlow | None | None | **BLOCKED** — no event-level funnel data in mock |
| P8 | "How many companies use Search but not Filters?" | Feature-pair gap analysis | None | None | **GAP** — features_used arrays exist, no aggregate |
| P9 | "What features drive the strongest health scores?" | Feature adoption x health correlation | None | None | **GAP** — both exist, no correlation query |
| P10 | "How does login frequency relate to plan tier?" | Logins_last_30d by plan | None | None | **GAP** — both exist per-company, no aggregate |

**Product Score: 0 WORKS, 0 PARTIAL, 8 GAP, 2 BLOCKED**

---

### TEAM 5: Marketing (Patrik)

These are "who should we target" and "where's the white space" questions. They want ICP analysis, market segmentation, and lead qualification.

| # | Question | Data Needed | Skill Today | Card Today | Verdict |
|---|----------|-------------|-------------|------------|---------|
| M1 | "What industries are our best customers in?" | Industry x ARR x health aggregation | None | None | **GAP** — industry + ARR + health exist per-company |
| M2 | "Which regions should we target for expansion?" | Region x growth x penetration | None | None | **GAP** — region data exists, no aggregate analysis |
| M3 | "What's the profile of a high-value customer?" | Attributes of top-20% ARR customers | None | None | **GAP** — all data exists, no ICP synthesis |
| M4 | "How many prospects come from company groups of existing customers?" | Group subsidiaries that aren't customers | `get_company_group` (per-company) | None | **GAP** — group data exists, no portfolio-wide warm lead scan |
| M5 | "What procurement categories have the most activity?" | Tender titles/categories across all awards | None | None | **GAP** — procurement data exists, no category aggregation |
| M6 | "Which industries have the highest win rate?" | Awards by industry | None | None | **BLOCKED** — no win/loss data, only awards |
| M7 | "What's the average time from prospect to customer?" | Prospect date → subscription start date | None | None | **BLOCKED** — no temporal lifecycle data |
| M8 | "Which content/outreach drives the most signups?" | Attribution data from marketing platform | None | None | **BLOCKED** — no marketing attribution in data model |
| M9 | "How does company size relate to plan choice?" | Employees x plan cross-tabulation | None | None | **GAP** — employees + plan exist, no aggregate |
| M10 | "What are the biggest untapped segments?" | Total addressable market minus current customers | None | None | **BLOCKED** — no TAM data, only existing customers |

**Marketing Score: 0 WORKS, 0 PARTIAL, 6 GAP, 4 BLOCKED**

---

## Aggregate Scorecard

### By Team

| Team | WORKS | PARTIAL | GAP | BLOCKED | Total Answerable After Fix |
|------|-------|---------|-----|---------|---------------------------|
| Leadership (10) | 2 | 2 | 3 | 3 | 7/10 (70%) |
| Sales (10) | 1 | 1 | 6 | 2 | 8/10 (80%) |
| Customer Success (10) | 1 | 2 | 6 | 1 | 9/10 (90%) |
| Product (10) | 0 | 0 | 8 | 2 | 8/10 (80%) |
| Marketing (10) | 0 | 0 | 6 | 4 | 6/10 (60%) |
| **TOTAL (50)** | **4** | **5** | **29** | **12** | **38/50 (76%)** |

### By Verdict

| Verdict | Count | What it means |
|---------|-------|--------------|
| **WORKS** | 4 (8%) | Existing skill + card handles this today |
| **PARTIAL** | 5 (10%) | Can hack an answer but no dedicated skill/card |
| **GAP** | 29 (58%) | Data exists in mock, needs aggregate skill + card |
| **BLOCKED** | 12 (24%) | Data doesn't exist — needs new fields or external sources |

---

## Gap Analysis: 4 Root Causes

### Root Cause 1: No Aggregate Query Layer (29 questions)

**The problem:** Every skill today operates on a single company. There's `getCustomerProfile("Adlibris")` but no `getAccountsByPlan("light")` or `getFeatureAdoption()`. The mock data IS there — subscriptions, usage, meetings, procurement are all arrays we could filter and aggregate. We just don't have functions that do it.

**The fix:** 5 new aggregate skills that query across the portfolio.

| Skill | What it does | Questions it answers |
|-------|-------------|---------------------|
| `get_renewal_pipeline(days)` | Renewals within N days, sorted by risk x revenue | S1, S8, C10, L5 (partial) |
| `get_segment_breakdown(dimension)` | Slice by plan/region/industry/health | L3, L7, S4, S10, C4, P4, P5, P10, M1, M2, M9 |
| `get_revenue_analysis()` | NRR decomposition, concentration, ACV, top accounts | L1, L2, L10, S2 |
| `get_usage_trends()` | Portfolio WAC, feature adoption rankings, declining accounts | C2, P1, P2, P3, P8, P9 |
| `get_engagement_gaps(days)` | Accounts with no meetings or low activity in N days | S8, C3, C5, C6 |

**Impact:** Fixes 29 of 50 questions (58%). This is the #1 priority.

### Root Cause 2: No Temporal Data (8 questions)

**The problem:** The mock data is a point-in-time snapshot. There's `arr_sek` and `previous_arr_sek` but no `start_date`, no `churned_date`, no event log. This blocks all "over time" questions.

**Questions blocked:** L4 (churn rate), L6 (customers added in 2026), L8 (on track for 100M), C7 (onboarding activation), P6 (churned customer features), P7 (activation funnel), M7 (time to customer).

**The fix:** Add 3 fields to mock data:

```ts
// On Subscription
start_date: string;      // "2024-06-15" — when they became a customer
churned_date: string | null;  // null if active, date if churned

// New: SubscriptionEvent[] (optional, for richer history)
// {date, event: "new"|"upgrade"|"downgrade"|"churn", plan_before, plan_after, arr_before, arr_after}
```

**Impact:** Unblocks 5 more questions (L4, L6, C7, P6 + enriches L1, L2, L8). Partial fix — full timeseries needs event log which is V2.

### Root Cause 3: No Cross-Portfolio Text Analysis (3 questions)

**The problem:** Meeting summaries, MEDDIC insights, and action items exist per-meeting — but we can't search or aggregate across all meetings. "What are the top objections?" requires scanning all meeting text.

**Questions blocked:** S9 (top objections), C6 (negative meeting themes).

**The fix:** A `get_meeting_insights()` skill that:
- Filters meetings by sentiment/team/date
- Aggregates common themes from summaries
- Extracts top MEDDIC patterns

For V1 with 15 companies and ~30 meetings, this is a simple in-memory filter + keyword extraction. No LLM needed.

**Impact:** Unblocks 2 questions. Nice-to-have for V1, important for V2 with real Garba data.

### Root Cause 4: No External/Prospect Data (5 questions)

**The problem:** Some questions require data we don't have and can't mock:
- S5 (prospects with procurement) — procurement data only covers existing customers
- S6/M6 (win rate) — no bid/loss tracking, only award data
- M8 (marketing attribution) — no marketing platform integration
- M10 (untapped segments) — no TAM data

**The fix:** These are real data source integrations, not mock data problems.
- **Procurement prospecting** → when real procurement data is loaded (90K awards), we'll have non-customer companies
- **Win rate** → needs bid tracking from procurement source
- **Marketing attribution** → needs HubSpot or PostHog marketing events
- **TAM** → needs Allabolag company database (100K companies)

**Impact:** Not fixable in V1 prototype. These are V2 integrations. Deferred.

---

## The Recommendation: What to Fix Before Building Cards

### Order of Operations

**Step 1 (Day 1): Enrich mock data** — Add `start_date` and `churned_date` to subscriptions. Add 2-3 churned companies to the dataset. This is 30 minutes of work but unblocks 5 blocked questions.

**Step 2 (Day 1-2): Build 5 aggregate skills** — Pure TypeScript functions that filter/aggregate existing arrays. No new APIs, no new data sources. This is the highest-leverage fix: 29 questions go from GAP to WORKS.

**Step 3 (Day 2-3): Register 5 new tools** — Wire aggregate skills into agent tools.ts with proper Zod schemas and descriptions. Agent can now answer 38/50 questions via text alone (no new cards yet).

**Step 4 (Day 3-4): Build 4 new card types** — Comparison, Breakdown, Renewal Pipeline, Feature Adoption. These render the aggregate skill output visually.

**Step 5 (Day 4-5): Digest Card + card enhancements** — Hugo's Monday report card. Adaptive headers. Embedded action buttons.

### Why This Order

The temptation is to jump straight to building pretty cards. But cards without data are empty shells. The aggregate skills are invisible to the user but they're the foundation — the agent literally can't answer 29 of 50 questions without them.

**Skills first. Cards second. Enhancements third.**

After Step 3, the agent can answer 38/50 questions in text. After Step 4, it renders them beautifully. After Step 5, it's the 0.1% experience.

---

## 5 New Aggregate Skills (Detailed)

### Skill 1: `get_renewal_pipeline(days_ahead)`

Highest impact — Sales, CS, AND Leadership all ask about renewals.

```ts
// Input
days_ahead: number  // default 90

// Output
{
  window_days: number;
  total_arr_at_stake: number;
  total_renewals: number;
  critical_count: number;     // health < 30
  warning_count: number;      // health 30-60
  healthy_count: number;      // health > 60
  renewals: Array<{
    company: string;
    org_nr: string;
    days_until_renewal: number;
    arr_sek: number;
    health_score: number;
    health_status: "strong" | "moderate" | "at_risk";
    plan: string;
    risk_signals: string[];   // from detect_churn logic
  }>;
}
```

**Implementation:** Filter subscriptions by renewal_date within window. For each, compute health score (reuse getCustomerProfile logic). Sort by health ascending, then ARR descending.

**Questions answered:** S1, S8 (combined with engagement), C10, L5 (partial)

---

### Skill 2: `get_segment_breakdown(dimension)`

Most versatile — answers 11 different questions across every team.

```ts
// Input
dimension: "plan" | "region" | "industry" | "health_status"

// Output
{
  dimension: string;
  total_companies: number;
  segments: Array<{
    label: string;              // "Core", "Stockholm", "Fintech", "at_risk"
    count: number;
    total_arr: number;
    avg_arr: number;
    avg_health: number;
    avg_wac: number;
    bidflow_adoption_pct: number;
    intelligence_pct: number;
    avg_logins_30d: number;
    features_avg_count: number; // avg number of features adopted
  }>;
}
```

**Implementation:** Group companies by dimension field. For each group, compute averages across subscription, usage, and health data.

**Questions answered:** L3, L7, S4 (plan="light" + high engagement), S10 (plan="scale" + no intelligence), C4, P4, P5, P10, M1, M2, M9

---

### Skill 3: `get_revenue_analysis()`

Revenue story — expansion, contraction, concentration, ACV.

```ts
// Output
{
  total_arr: number;
  total_arr_formatted: string;
  nrr_percent: number;
  expansion_total: number;      // sum of positive deltas
  contraction_total: number;    // sum of negative deltas (absolute)
  net_arr_change: number;       // expansion - contraction
  churn_arr: number;            // ARR from churned accounts (needs churned_date)
  new_arr: number;              // ARR from new accounts (needs start_date)
  avg_acv: number;
  acv_by_plan: Record<string, { avg: number; min: number; max: number; count: number }>;
  concentration: {
    top_5_arr: number;
    top_5_pct: number;
    top_10_arr: number;
    top_10_pct: number;
  };
  top_accounts: Array<{
    company: string;
    arr_sek: number;
    pct_of_total: number;
    plan: string;
  }>;
  movers: {
    expansions: Array<{ company: string; from: number; to: number; delta: number; plan: string }>;
    contractions: Array<{ company: string; from: number; to: number; delta: number; plan: string }>;
  };
}
```

**Implementation:** Iterate subscriptions. Compare arr_sek vs previous_arr_sek for expansion/contraction. Sort by ARR for concentration. Group by plan for ACV.

**Questions answered:** L1, L2, L10, S2 (enriched expansion view)

---

### Skill 4: `get_usage_trends()`

Product team's primary tool — feature adoption and engagement patterns.

```ts
// Output
{
  portfolio_wac: { current: number; previous: number; trend: "up"|"down"|"flat"; delta_pct: number };
  portfolio_logins: { total_30d: number; avg_per_company: number };
  feature_adoption: Array<{
    feature: string;
    adopted_count: number;
    total_eligible: number;  // companies on plans that include this feature
    adoption_pct: number;
    by_plan: Record<string, { adopted: number; total: number; pct: number }>;
    avg_health_of_adopters: number;
    avg_health_of_non_adopters: number;
  }>;
  declining_accounts: Array<{
    company: string;
    org_nr: string;
    wac: number;
    wac_prev: number;
    delta_pct: number;
    health_score: number;
    plan: string;
    arr_sek: number;
  }>;
  growing_accounts: Array<{
    company: string;
    org_nr: string;
    wac: number;
    wac_prev: number;
    delta_pct: number;
    health_score: number;
    plan: string;
  }>;
}
```

**Implementation:** Aggregate WAC across companies. Count features_used frequency. Compare adopter vs non-adopter health scores. Filter by wac_trend.

**Questions answered:** C2, P1, P2, P3 (BidFlow x health), P4 (Core x Ask Tendium), P8 (feature gap), P9 (features x health), P5 (via by_plan breakdown)

---

### Skill 5: `get_engagement_gaps(days_threshold)`

CS monitoring — who's going dark?

```ts
// Input
days_threshold: number  // default 90

// Output
{
  threshold_days: number;
  no_recent_meetings: Array<{
    company: string;
    org_nr: string;
    last_meeting_date: string | null;
    days_since_meeting: number | null;  // null = never met
    arr_sek: number;
    health_score: number;
    plan: string;
    days_until_renewal: number | null;
  }>;
  low_login_activity: Array<{
    company: string;
    org_nr: string;
    logins_last_30d: number;
    wac: number;
    arr_sek: number;
    health_score: number;
    plan: string;
  }>;
  meeting_sentiment_summary: {
    positive: number;
    neutral: number;
    negative: number;
    recent_negative: Array<{
      company: string;
      date: string;
      summary: string;
    }>;
  };
}
```

**Implementation:** For each company, find most recent meeting date. Calculate days since. Filter by threshold. Also scan for low logins (<10 in 30d). Aggregate meeting sentiments.

**Questions answered:** S8 (no meeting + renewing), C3, C5 (CS queue), C6 (negative themes via sentiment_summary)

---

## Mock Data Enrichment Needed

### Add to Subscription type

```ts
interface Subscription {
  // ... existing fields ...
  start_date: string;          // NEW — when subscription began
  churned_date: string | null; // NEW — null if active
  status: "active" | "churned" | "trial";  // NEW
}
```

### Add 2-3 Churned Companies

To make churn queries meaningful, add churned ex-customers to the dataset:

```ts
// Example churned company
{
  name: "TechNord AB",
  org_nr: "556999-0001",
  domain: "technord.se",
  industry: "Technology",
  region: "Stockholm",
  employees: 150,
  // Subscription with churned_date
  subscription: {
    plan: "core",
    arr_sek: 0,           // churned
    previous_arr_sek: 85000,
    start_date: "2024-03-01",
    churned_date: "2025-11-15",
    status: "churned",
    // ... rest
  }
}
```

**Why 2-3, not more:** Enough to make churn queries return data, not so many that they skew "active portfolio" numbers. The agent can reason about patterns even with 2-3 examples.

---

## New Card Types (6)

### Card 7: Comparison Card

**Archetype:** Brief
**Purpose:** Side-by-side segment comparison with metrics table.
**Trigger:** `get_segment_breakdown` output
**Used for:** Plan tier comparison, region breakdown, industry analysis, size correlation.

```
┌─────────────────────────────────────────┐
│  PLAN TIER BREAKDOWN         15 accounts│
│─────────────────────────────────────────│
│            Light     Core      Scale    │
│  Count     7         5         3        │
│  Avg ARR   18K       42K       165K     │
│  Avg WAC   2.1       4.8       8.2      │
│  BidFlow   0%        60%       100%     │
│  Intel     0%        40%       67%      │
│  Health    52        68        81       │
│─────────────────────────────────────────│
│  Core→Scale path: BidFlow + Intelligence│
│  [Show upsell candidates]              │
└─────────────────────────────────────────┘
```

### Card 8: Breakdown Card

**Archetype:** Brief
**Purpose:** Single metric decomposed into components (waterfall-style).
**Trigger:** `get_revenue_analysis` output
**Used for:** NRR decomposition, ACV by tier, revenue concentration.

```
┌─────────────────────────────────────────┐
│  REVENUE ANALYSIS                       │
│─────────────────────────────────────────│
│  Total ARR        42.1M SEK             │
│  + Expansion      +2.3M  ██████         │
│  − Contraction    −890K  ███            │
│  Net Change       +1.4M                 │
│  NRR              113%                  │
│─────────────────────────────────────────│
│  CONCENTRATION                          │
│  Top 5 = 68% of ARR                    │
│  Top 10 = 89% of ARR                   │
│─────────────────────────────────────────│
│  TOP MOVERS                             │
│  ↑ Volvo Group    +480K (→ Scale)       │
│  ↑ Klarna         +120K (Intelligence)  │
│  ↓ Adlibris       −85K (contraction)    │
│─────────────────────────────────────────│
│  [Flag contracting accounts]            │
└─────────────────────────────────────────┘
```

### Card 9: Renewal Pipeline Card

**Archetype:** List
**Purpose:** Upcoming renewals sorted by risk x revenue.
**Trigger:** `get_renewal_pipeline` output
**Used for:** Renewal management, risk prioritization.

```
┌─────────────────────────────────────────┐
│  RENEWAL PIPELINE          Next 90 days │
│  7 renewals · 3.2M SEK at stake        │
│─────────────────────────────────────────│
│  🔴 Adlibris      45d   145K    12/100 │
│  🟡 Securitas     78d   380K    48/100 │
│  🟢 SKF           89d   520K    74/100 │
│  🟢 Malmö Mun.    67d   280K    72/100 │
│─────────────────────────────────────────│
│  1 critical · 1 warning · 5 healthy    │
│  [Flag at-risk]  [Schedule calls]      │
└─────────────────────────────────────────┘
```

### Card 10: Feature Adoption Card

**Archetype:** List
**Purpose:** Feature ranking with adoption bars and plan breakdown.
**Trigger:** `get_usage_trends` output (feature_adoption array)
**Used for:** Product adoption analysis, BidFlow tracking.

```
┌─────────────────────────────────────────┐
│  FEATURE ADOPTION           15 accounts │
│─────────────────────────────────────────│
│  Search           15/15  ████████  100% │
│  Tender Filters   13/15  ███████▒   87% │
│  AI Summaries     11/15  ██████▒    73% │
│  BidFlow           7/15  ████▒      47% │
│  Ask Tendium AI    4/15  ██▒        27% │
│  Intelligence      3/15  █▒         20% │
│─────────────────────────────────────────│
│  BidFlow adopters avg health: 74       │
│  Non-adopters avg health: 48           │
│  [Show non-adopters]                   │
└─────────────────────────────────────────┘
```

### Card 11: Digest Card

**Archetype:** Brief
**Purpose:** Multi-section weekly report for Hugo's Monday.
**Trigger:** `get_weekly_digest` (new orchestration skill) or agent combining multiple tools
**Used for:** Scheduled commercial reports.

(Same wireframe as v1 spec — headline, metrics, needs attention, wins, upcoming renewals, action buttons.)

### Card 12: Engagement Gap Card

**Archetype:** List
**Purpose:** Accounts going dark — no meetings, low logins.
**Trigger:** `get_engagement_gaps` output
**Used for:** CS proactive monitoring.

```
┌─────────────────────────────────────────┐
│  ENGAGEMENT GAPS         90-day window  │
│─────────────────────────────────────────│
│  NO RECENT MEETINGS (4)                │
│  Uppsala Mun.    never met    15K  52  │
│  Linköping       142d ago     22K  44  │
│  Securitas        98d ago    380K  48  │
│  ASSA ABLOY      106d ago    320K  61  │
│─────────────────────────────────────────│
│  LOW LOGIN ACTIVITY (2)                │
│  Adlibris        3 logins    145K  12  │
│  Uppsala Mun.    5 logins     15K  52  │
│─────────────────────────────────────────│
│  [Schedule check-ins]  [Send nudges]   │
└─────────────────────────────────────────┘
```

---

## Post-Build Coverage

### After Skills + Data Enrichment + Cards

| Team | Before | After | Remaining Blocked |
|------|--------|-------|-------------------|
| Leadership (10) | 2 WORKS | 7 WORKS | L6 (partially fixed w/ start_date), L8 (needs target), M8 |
| Sales (10) | 1 WORKS | 8 WORKS | S5 (prospect procurement), S6 (win rate) |
| Customer Success (10) | 1 WORKS | 9 WORKS | C7 (mostly fixed w/ start_date) |
| Product (10) | 0 WORKS | 8 WORKS | P6 (fixed w/ churned_date), P7 (funnel) |
| Marketing (10) | 0 WORKS | 6 WORKS | M6, M7, M8, M10 (external data) |
| **TOTAL** | **4/50 (8%)** | **38/50 (76%)** | 12 need external data |

### What Stays Blocked (and Why That's OK for V1)

| Question | Why blocked | When it gets fixed |
|----------|------------|-------------------|
| L8 (track to 100M) | Needs target/goal system | V2 — when OKR integration added |
| S5 (prospect procurement) | Mock only has customer procurement | V1.1 — when real 90K procurement loaded |
| S6 (win rate) | No bid/loss data | V2 — needs procurement source enrichment |
| M6 (industry win rate) | Same as S6 | V2 |
| M7 (time to customer) | Partial fix with start_date, needs prospect data | V2 — needs HubSpot deal pipeline |
| M8 (marketing attribution) | No marketing platform data | V2 — needs HubSpot/PostHog marketing |
| M10 (untapped segments) | No TAM data | V2 — needs Allabolag 100K company DB |
| P7 (activation funnel) | No event-level data | V2 — needs PostHog event stream |

**Pattern:** All 12 blocked questions need external data sources (HubSpot, real procurement DB, Allabolag, PostHog events). These are V2 integrations. The V1 prototype should focus on the 38 questions we CAN answer with existing + enriched mock data.

---

## Build Plan (Revised)

| Day | What | Impact |
|-----|------|--------|
| **Day 1 AM** | Enrich mock data: add `start_date`, `churned_date`, `status` to subscriptions. Add 2-3 churned companies. | Unblocks L4, L6, C7, P6 |
| **Day 1 PM** | Build `get_renewal_pipeline()` + `get_segment_breakdown()` skills | Answers 15 questions |
| **Day 2 AM** | Build `get_revenue_analysis()` + `get_usage_trends()` skills | Answers 10 more questions |
| **Day 2 PM** | Build `get_engagement_gaps()` skill + register all 5 as tools | Answers 4 more questions. Agent can now answer 38/50 in text. |
| **Day 3** | Build Comparison Card + Breakdown Card | 2 highest-reuse card types |
| **Day 4** | Build Renewal Pipeline Card + Feature Adoption Card + Engagement Gap Card | Complete the card set |
| **Day 5** | Build Digest Card + adaptive headers + embedded action buttons | Hugo's Monday report + 0.1% polish |

### Verification: Test All 50 Questions

After build, run each question through the agent and verify:
- [ ] Agent calls the right tool
- [ ] Tool returns correct data
- [ ] Card renders correctly
- [ ] Commentary stays under 80 words
- [ ] Action buttons trigger HITL flow
- [ ] Blocked questions get an honest "I don't have that data yet" response

---

## Card Design Principles

### From the Founder-OS Stress Test (0.1% Quality Bar)

**1. Cards as Decision Units, Not Data Units**

Bad header: "Adlibris AB — Company Profile"
Good header: "SAVE THIS ACCOUNT — Adlibris renews in 45d with health 12"

Adapt header based on data:
- Health < 30 + renewal < 90d → "SAVE THIS ACCOUNT"
- Expansion signals → "EXPANSION READY"
- New customer, low activation → "ACTIVATE THIS ACCOUNT"
- Stable + high health → company name only

**2. Actions Embedded ON Cards**

Every card with an actionable finding has buttons at the bottom:
- Risk Card → `[Flag critical]` `[Draft check-in emails]`
- Renewal Pipeline → `[Schedule renewal calls]` `[Send reminders]`
- Expansion Card → `[Draft upsell proposal]` `[Brief me for call]`
- Digest Card → `[Share to #commercial]` `[Create tasks for risks]`

Buttons send a message to the agent, triggering HITL flow.

**3. Proactive Cross-Reference (Insight Annotations)**

When the agent pulls data, it notices things the user didn't ask about:

```
┌─────────────────────────────────────────┐
│  [Normal card content]                  │
│─────────────────────────────────────────│
│  Adlibris's parent Bonnier Group has 3  │
│  other subsidiaries — none are          │
│  customers. Warm lead opportunity.      │
│  [Tell me more]                         │
└─────────────────────────────────────────┘
```

---

## Delivery Modes (Same Cards, Different Surfaces)

| Mode | How | Who |
|------|-----|-----|
| **On-demand** | User asks in chat → card renders | Everyone |
| **Pinned** | Card persists at top of home, auto-refreshes | Leadership |
| **Scheduled** | Cron generates card, delivers to Slack/email | Hugo (Mon), CS (daily) |
| **Triggered** | Health drops / renewal approaches → alert card | CS, Sales |

### Pinned Cards (Home Screen)

Leadership sees 3-4 pinned aggregate cards above the chat:

1. Portfolio Brief (ARR, NRR, customer count, plan mix)
2. Risk Scan (at-risk accounts, severity sorted)
3. Renewal Pipeline (next 90 days)
4. Weekly Wins (expansions, new activations)

### Scheduled Briefs

| Schedule | Card | Channel |
|----------|------|---------|
| Monday 8am | Digest Card | #commercial Slack |
| Daily 9am | Risk summary | #cs-alerts Slack |
| Friday 4pm | Week-in-review | #leadership Slack |
| 90d before renewal | Renewal brief | CS owner DM |
