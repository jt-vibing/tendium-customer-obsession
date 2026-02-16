# Technical Specification: Tendium Customer Graph

**Version:** 1.1 (2-tier search, Garba API confirmed)
**Date:** Feb 13, 2026
**Status:** Build Phase
**Target:** V1 by March 6, 2026

---

## Architecture Overview

### System Purpose

Unified customer intelligence layer that stitches data from 7 sources at query time, delivering answers in 30 seconds (not 5 hours of manual joins).

### Core Principle: Ground Truth

- **Raw data access** — No abstraction layers
- **Every insight traces to source** — Citations required
- **Query-time stitching** — Not pre-aggregated views
- **Read-only architecture** — No writes to source systems

---

## Data Sources

| Source | Type | Records | Priority | Integration Method |
|--------|------|---------|----------|-------------------|
| Procurement Data | Awards | 90k + 14k pipeline | Phase 1 | JSON dump |
| Company Groups | Corporate Groups | 100k companies | Phase 1 | Allabolag scrape |
| Younium | Billing | 1,376 subscriptions, 42M SEK | Phase 1 | Direct API |
| PostHog | Product Analytics | 5,000 WAC | Phase 1 | **MCP Server** |
| Garba | Call Intelligence | 13,437 meetings | Phase 1 | REST API (confirmed) |
| Bolagsfakta | Company Registry | All Swedish companies | Phase 1 | Scrape/API |
| Postmark | Email Delivery | Tender digest | Phase 2 | Direct API |
| HubSpot | CRM | Support tickets, deals | Phase 2 | **MCP Server** |

**Join Key:** Organization number (org_nr) — UUID standardization across all sources

**Note on Bolagsfakta:** Swedish company registry data (bolagsfakta.se) provides company financials, revenue, employee count, industry classification, board/ownership, and salary data. Critical for: (1) enriching customer and prospect profiles, (2) domain→org_nr mapping for Garba join, (3) TAM market sizing for expansion targeting, (4) company size correlation analysis.

---

## Tech Stack

### Layer 1: Data Access (MCP-First Architecture)
- **JSON dumps** — Procurement data (90k+14k), Company Groups (100k) from Jens
- **MCP Servers** — PostHog MCP, HubSpot MCP (composable intelligence)
- **Direct APIs** — Younium, Garba, Bolagsfakta, Postmark
- **Import scripts** — Next.js API routes to fetch and transform source data
- **MCP Composition** — Customer Graph MCP acts as composition layer over specialized MCP servers

### Layer 2: Store & Join
- **Supabase (Postgres)** — Managed database with org_nr as primary join key
- **pgvector** — Available for future vector embeddings (not needed for V1 at 1,376 customers)
- **Query-time stitching** — Real-time joins across tables via org_nr
- **Auth** — None for prototype; Supabase Auth (email/password) for V1

### Layer 3: Intelligence
- **Claude API (Sonnet 4.5)** — LLM for punch card generation AND natural language search
- **2-tier search** — Keyword (client-side `.filter()`) + Claude (NL→reasoning+filters)
- **No Voyage AI for V1** — Claude handles similarity/discovery at current scale (1,376 customers fit in context window). Add embeddings when >5K customers or sub-100ms similarity needed.

### Layer 4: Interface
- **Next.js + Vercel** — React frontend + API routes, deployed to Vercel
- **Search** — Entity-first with 2 tiers (keyword instant, Claude NL reasoning)
- **Design** — Linear-inspired (Inter font, CSS custom properties, dark mode)

---

## Data Model

### Core Entities

#### Customer
```
customer_id (UUID from org_nr)
name
org_nr (join key)
acquisition_date
current_tier (Younium)
arr (Younium)
health_score (computed from PostHog + Younium)
```

#### Company Group
```
group_id (UUID)
parent_company
subsidiaries[] (org_nr list)
relationship_type (concern, subsidiary, etc.)
source (Allabolag)
```

#### Procurement Award
```
award_id (UUID)
org_nr (join key to customer)
tender_title
award_date
award_value
sector (CPV code)
win/loss
source (Nordic procurement data)
```

#### Usage Event (PostHog)
```
event_id
customer_id (org_nr)
event_name
timestamp
properties{}
```

#### Call / Meeting (Garba)
```
id (UUID from Garba)
title
date (ISO timestamp)
duration (seconds)
status (done/ongoing/processing)
attendeeType (external/internal)
user {id, name, email, teamName}     — Tendium rep
companies [{id, name, domain}]       — Associated company (join via domain→org_nr lookup)
participants [{name, email, type, speakerId}]
summaries {
  shortSummary                        — 1 paragraph
  markdownSummary                     — Structured markdown
  insights [{title, value, sortOrder}] — MEDDIC analysis (16 sections)
}
actionItems []
transcript {languageCode, paragraphs[{sentences, speakerId, start, end}]}
```
**Note:** Garba provides company `domain` not `org_nr`. Requires domain→org_nr mapping table.
**Volume:** 13,437 meetings (as of Feb 13, 2026). Paginated at 50/page.
**API:** `https://api.garba.ai/v1/meetings` (Bearer token auth, scopes: read:data, read:meeting_video)

---

## Key Architectural Decisions

### 1. Supabase (Postgres) — not Modal + SQLite
**Decision:** Supabase (managed Postgres with pgvector)
**Why:** Scales from 10→100 users without rebuild. Managed service = no server ops. Built-in auth, pgvector for embeddings, EU-hosted. Claude Code-friendly (well-documented, standard stack). Free tier for dev, $25/month production. Modal deferred — useful for automation post-V1 if needed.

### 2. 2-Tier Search Architecture (revised Feb 13)
**Decision:** Keyword + Claude NL search — Voyage AI deferred
**Why:** With 1,376 customers, the dataset fits in Claude's context window. Claude can handle both structured filtering ("customers at risk in education") AND similarity/discovery ("find companies like Adlibris") without embeddings. Simpler architecture (2 systems, not 3), one fewer vendor. Add embeddings when: >5K customers, unstructured data search, sub-100ms similarity needed, or Germany/UK expansion.
**Future embedding options:** Cohere embed-v3 (best Nordic multilingual), OpenAI text-embedding-3-small (cheapest), or Supabase built-in.

### 3. Query-time stitching vs. Pre-aggregation
**Decision:** Query-time stitching via org_nr joins in Supabase
**Why:** Ground truth principle (no abstractions), real-time data, Postgres handles joins efficiently

### 4. Claude API for Intelligence
**Decision:** Claude API (Anthropic)
**Why:** Longer context windows, best for citation/reasoning tasks, powers both punch card generation and natural language search (Tier 2)

### 5. org_nr as Join Key — no UUIDs
**Decision:** Organization number as universal join key (no separate UUID)
**Why:** Stable across all Nordic systems, Jens validated with Sales Graph Explorer. Simple normalization (remove dashes, uppercase). UUIDs deferred to international expansion (Germany/UK)

### 6. Next.js + Vercel — no separate backend
**Decision:** Next.js API routes handle all server logic (Claude, Voyage, Supabase queries)
**Why:** One framework, one deployment. No backend server to manage. Built with Claude Code by non-technical team.

---

## Intelligence Layer: Punch Cards

### Sales Punch Card
**Sections:**
1. Procurement Profile (bid volume, win rate, sectors)
2. Current Relationship (tier, ARR, contract end)
3. Usage Signals (features used, engagement trends)
4. Recent Activity (calls, emails, deals)
5. Recommended Actions (upsell opportunities, risks)

**Data Sources:** Procurement (90k), Younium, PostHog, Garba, HubSpot
**Update Frequency:** Daily
**LLM Prompt:** [See intelligence-layer.md]

### CS Health Brief
**Sections:**
1. Customer Health Score (computed)
2. Usage Trends (last 30/60/90 days)
3. Churn Signals (declining usage, support tickets)
4. Procurement Activity (declining bids = growth slowing)
5. Recommended Actions (proactive outreach, QBR prep)

**Data Sources:** PostHog, Younium, Procurement, Garba, HubSpot
**Update Frequency:** Weekly (Hugo's Monday spreadsheet replacement)
**LLM Prompt:** [See intelligence-layer.md]

### Leadership Monday Compass
**Sections:**
1. State of Business (ARR, WAC, NRR)
2. Top Opportunities (upsells, expansions)
3. Top Risks (churn signals, declining accounts)
4. Procurement Insights (market trends, competitive landscape)
5. Key Decisions Needed (prioritized by impact)

**Data Sources:** All 7 sources
**Update Frequency:** Weekly (Monday morning)
**LLM Prompt:** [See intelligence-layer.md]

---

## API Specifications

### Search API
```
GET /search?q={query}&type={customer|company|procurement}
Response: {
  results: [{
    entity_id, entity_type, name, org_nr,
    snippet, score, source_citations[]
  }]
}
```

### Customer Profile API
```
GET /customer/{org_nr}
Response: {
  customer: {...},
  procurement: [...],
  usage: [...],
  calls: [...],
  billing: {...},
  health: {...},
  punch_card: {generated_at, sections[]}
}
```

### Punch Card Generation API
```
POST /punch-card/generate
Body: {customer_id, card_type: sales|cs|leadership}
Response: {
  card_id, generated_at, sections[], citations[]
}
```

---

## Implementation Phases

### Phase 0: Wireframes + Prototype (Week 1: Feb 12-17)
**Goal:** Interactive prototype with mock data for stakeholder testing

- [x] HTML wireframes (search, profile, punch cards)
- [x] Build Hub with tabbed navigation
- [ ] Next.js prototype with Tailwind design system
- [ ] Tier 1: Keyword search (client-side `.filter()`)
- [ ] Customer profile page (5 tabs)
- [ ] Sales Punch Card + CS Health Brief (mock content)
- [ ] Garba API integration (Calls tab)
- [ ] Deploy to Vercel

**Milestone:** Stakeholders click through prototype independently (Feb 17)

---

### Phase 1: Foundation + Real Data (Weeks 2-3: Feb 18-28)
**Goal:** Replace mock data with real sources

- [ ] Supabase project + schema setup
- [ ] Procurement data import (90k awards JSON from Jens)
- [ ] Company Groups import (100k companies JSON)
- [ ] Younium API connection (need creds)
- [ ] PostHog API connection (need creds)
- [ ] Garba API full sync (13,437 meetings, have API key)
- [ ] Domain→org_nr mapping table (for Garba join)
- [ ] org_nr standardization across all sources

**Milestone:** Query a customer by org_nr, get unified data from 5 sources

---

### Phase 2: Intelligence (Weeks 3-4: Mar 1-14)
**Goal:** LLM-powered punch cards with real data

- [ ] Claude API integration (Sonnet 4.5)
- [ ] Tier 2: Claude NL search (natural language → reasoning + filters)
- [ ] Sales punch card prompt engineering (real data)
- [ ] CS health brief prompt engineering (real data)
- [ ] Leadership Monday Compass prompt engineering
- [ ] Citation system (every claim → source link)
- [ ] Cross-source compound signal detection

**Milestone:** Generate Sales punch card for real customer, all claims cited

---

### Phase 3: Validation (Weeks 5-6)
**Goal:** Prove success metric

- [ ] Alpha testing with Sales (3 reps)
- [ ] Alpha testing with CS (Hugo + 1 other)
- [ ] Alpha testing with Leadership (Charlotte)
- [ ] Success validation:
  - [ ] Elephant Test: Same answer from 3 depts
  - [ ] Speed Test: Hugo stops using spreadsheet
  - [ ] Moat Test: 1 unique insight from procurement
- [ ] Behavioral change documentation
- [ ] V1 launch readiness

**Milestone:** All 3 tests pass by March 6, 2026

---

## Success Metrics

### Primary (Must Pass)
> **By March 6, 2026:** Sales, CS, and Leadership independently ask and get answers to take action on with the same unified data. No more conflicting versions of truth.

### Supporting Metrics
- **Speed:** Query time < 30 seconds (vs. 5 hours manual)
- **Coverage:** 7 data sources integrated and queryable
- **Adoption:** 3+ dept leads use daily
- **Moat:** 1+ unique insight from procurement data validated
- **Traceability:** 100% of claims link to source data

---

## Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Data exports delayed (Younium/Procurement) | High | **Critical path:** Request exports by Week 1, have sample data fallback |
| LLM hallucinations (wrong claims) | High | Citation system (every claim = source link), ground truth enforcement |
| Adoption fails (teams don't use it) | High | Jens's bar: Behavioral change, not feature availability. Alpha testing in Week 5. |
| Query speed > 30 sec | Medium | SQLite cache, optimize joins, index org_nr |
| Procurement data quality issues | Medium | Data validation in Phase 1, flag missing/corrupt records |
| Company Groups mapping incomplete | Low | Start with subset, expand coverage incrementally |

---

## Resolved Questions (Feb 13)

1. ~~**Vector DB choice?**~~ → **Deferred.** pgvector in Supabase available when needed. Not required for V1 (1,376 customers fit in Claude context).
2. ~~**Embedding model?**~~ → **Deferred.** Cohere embed-v3 (Nordic multilingual) or OpenAI text-embedding-3-small (cheapest) when embeddings are needed.
3. **Company Groups refresh cadence?** Weekly scrape vs. monthly vs. static snapshot — TBD
4. ~~**Garba call recording LLM processing?**~~ → **Resolved.** Garba provides AI-generated summaries, MEDDIC insights, and action items via API. No separate LLM processing needed for V1. Use Garba's native analysis.
5. **Hugo's exact Monday spreadsheet?** Get copy to ensure we're replacing the right workflow — TBD

## Open Questions

1. **Garba domain→org_nr mapping:** Garba identifies companies by domain (e.g., "comfort.se"), not org_nr. Need a lookup table to join Garba data to the customer graph.
2. **Company Groups refresh cadence?** Weekly scrape vs. monthly vs. static snapshot

---

## References

- **[Problem Page](problem/index.html)** — The why
- **[Tech Page](tech/index.html)** — The how (visual)
- **[Intelligence Layer](intelligence-layer.md)** — LLM prompts & punch cards
- **[Customer Graph Features](customer-graph-features.md)** — Complete feature set
- **[AARRR Journey](aarrr/option-a.html)** — Customer journey context

---

**Document Owner:** John Tan (Product Advisor)
**Technical Lead:** Jens Egeland (Engineering)
**Last Updated:** Feb 13, 2026
