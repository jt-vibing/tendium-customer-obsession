# Tendium Customer Graph

*AI-first customer intelligence layer for internal teams*

**Status:** Planning Phase → Build Phase (Starting Feb 12, 2026)
**Target:** V1 by March 6, 2026 (6-week sprint)
**Owner:** John Tan (Product Advisor) + Jens Egeland (Engineering Driver)

---

## What Is This?

The Customer Graph unifies 7 fragmented data sources into one intelligent layer that Sales, CS, and Leadership can query for real-time customer intelligence.

**The Problem:** 7 teams (Sales, CS, Marketing, Product, Leadership, C.Ops, RevOps) each have different versions of customer truth. Hugo (Commercial Ops) spends 5 hours every Monday manually joining data from Younium, PostHog, and Garba.

**The Solution:** Unified customer intelligence delivered in 30 seconds (not 5 hours), powered by procurement data (90k awards + 14k pipeline) that competitors can't access.

---

## Success Metric

> **By March 6, 2026:** Sales, CS, and Leadership independently ask and get answers to take action on with the same unified data. No more conflicting versions of truth.

---

## 7 Data Sources (The Moat)

**MCP-First Architecture:** PostHog and HubSpot integrate as MCP servers for composable intelligence.

1. **Procurement Data** (90k awards + 14k pipeline) — Nordic procurement decisions, 0.1% differentiator
2. **Company Groups** (100k companies from Allabolag) — Warm lead identification via corporate group analysis
3. **Younium** (SoT: 42M SEK ARR, 1,376 subscriptions) — Billing, subscriptions, revenue (Direct API)
4. **PostHog** (Product analytics) — Usage data, feature adoption, health signals (**MCP Server**)
5. **Garba** (Call recordings) — CS/Sales conversations, sentiment (Direct API)
6. **Postmark** (Email delivery) — Tender digest engagement (Direct API)
7. **HubSpot** (CRM) — Support tickets, deal history, contact records (**MCP Server**)

---

## Core Constraints

1. **6-week sprint** — V1 by March 6, 2026 (end of Q1)
2. **Procurement data moat** — 90k awards + 14k pipeline, competitors can't access
3. **Company Groups strategy** — 100k companies, ultra-warm lead hypothesis
4. **Speed bar** — Hugo's 5 hours → 30 seconds
5. **Behavioral change required** — Jens's Sales Graph Explorer set the adoption bar
6. **Ground truth principle** — Raw data access, no abstraction layers
7. **Read-only architecture** — No writes to source systems, Supabase query-time stitching

---

## Non-Goals

- ✕ Not a CRM (complements HubSpot, doesn't replace it)
- ✕ Not a dashboard (search/explore entities, not view charts)
- ✕ Not a data warehouse (real-time stitching, not pre-aggregation)
- ✕ Not a product feature (internal tooling only)
- ✕ Not user analytics (company-level intelligence, not login tracking)
- ✕ Not AI black boxes (every insight traces to source data)

---

## Key Documents

### Planning & Context
- **[Problem Page](problem/index.html)** — The why, the who, the what (user-facing PRD)
- **[Tech Page](tech/index.html)** — The how (architecture, data model, tech stack)
- **[SPEC.md](SPEC.md)** — Consolidated technical specification
- **[Intelligence Layer](intelligence-layer.md)** — LLM-powered punch cards for each role
- **[Customer Graph Features](customer-graph-features.md)** — Complete feature set

### UX/UI Wireframes
- **[Wireframes Hub](wireframes/index.html)** — All wireframes with design system reference
- **[Search](wireframes/search.html)** — 3-tier search (Keyword, NL, Semantic)
- **[Customer Profile](wireframes/profile.html)** — Unified profile with 5 tabs
- **[Sales Punch Card](wireframes/punch-card-sales.html)** — AI-generated sales intelligence brief
- **[CS Health Brief](wireframes/punch-card-cs.html)** — Hugo's Monday spreadsheet replacement

### Reference
- **[AARRR Journey](aarrr/option-a.html)** — Customer journey mapping across 5 stages
- **[OKR Mapping](okr/index.html)** — How this aligns with company OKRs

---

## Build Phase Priorities

### Phase 0: UX/UI Wireframes (Week 1, Day 1)
- [x] HTML wireframes using existing design system
- [x] Search page (3-tier: Keyword, Natural Language, Semantic)
- [x] Customer profile page (5 tabs)
- [x] Sales Punch Card (5 sections, Adlibris example)
- [x] CS Health Brief (Hugo's Monday spreadsheet replacement)

### Phase 1: Interactive Prototype (Week 1, Days 2-5)
- [ ] Next.js + Vercel setup with design system
- [ ] Mock data (10-15 Nordic companies)
- [ ] Tier 1: Keyword search (client-side .filter())
- [ ] Customer profile pages (all 5 tabs)
- [ ] Stakeholder testing session (Feb 17)

### Phase 1.5: Foundation (Weeks 2-3)
- [ ] Supabase database setup (Postgres + pgvector)
- [ ] org_nr standardization (join key across all sources)
- [ ] Procurement data import (90k awards JSON)
- [ ] Company Groups import (100k companies JSON)
- [ ] Younium API connection (Direct)
- [ ] PostHog MCP Server integration
- [ ] HubSpot MCP Server integration (Phase 2)

### Phase 2: Intelligence (Weeks 3-4)
- [ ] Claude API integration (punch cards + Tier 2 NL search)
- [ ] Voyage AI embeddings (Tier 3 semantic search)
- [ ] Sales punch card with real data
- [ ] CS health brief with real data
- [ ] 3-tier search fully operational

### Phase 3: Validation (Weeks 5-6)
- [ ] Alpha testing with Sales, CS, Leadership
- [ ] Success metric validation (same answer from unified data)
- [ ] Behavioral change evidence (Hugo stops using spreadsheet)
- [ ] Procurement moat validation (unique insights)

---

## Success = Passing 3 Critical Tests

1. **The Elephant Test:** Sales, CS, Leadership query same customer → get same answer
2. **The Speed Test:** Hugo stops using Monday spreadsheet (5 hrs → 30 sec)
3. **The Moat Test:** Procurement data surfaces insight competitors can't replicate

**All 3 must pass by March 6, 2026.**

---

## Team

- **John Tan** — Product Advisor (6-week sprint lead)
- **Jens Egeland** — Engineering Driver (Sales Graph Explorer proof-of-concept)
- **Hugo Jennerholm** — Commercial Operations (primary user, Monday spreadsheet owner)
- **Oskar Lundgren** — Co-founder/CPO (product alignment)
- **Charlotte Söderhjelm** — Deputy CEO (operational buy-in)
- **Thibaud Hartwig** — GTM/Sales Leadership (sales adoption validation)

---

## Quick Start

1. **Read the problem page** — Understand the why: [problem/index.html](problem/index.html)
2. **Read the tech spec** — Understand the how: [SPEC.md](SPEC.md)
3. **Review data sources** — See what you're working with (7 sources above)
4. **Check phase 1 priorities** — Know what to build first
5. **Validate success metric** — Keep it simple: unified data, same answers

---

## Questions?

- **What's the 0.1% differentiator?** Procurement data (90k Nordic awards). Competitors don't have this.
- **Why not just fix HubSpot?** HubSpot is a CRM, not an intelligence layer. Different tools.
- **What's the behavioral change bar?** Jens's Sales Graph Explorer changed Sales workflows. That's the bar.
- **Why 6 weeks?** V1 by end of Q1 2026 (March 6 deadline).
- **What if data exports are delayed?** Interactive prototype runs on realistic mock data. Real data integrates when JSON dumps + API access arrive.
- **What's the tech stack?** Supabase (Postgres + pgvector) + Next.js + Vercel + Claude API + Voyage AI. No separate backend.

---

## Tech Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Data Access** | JSON dumps + MCP Servers + Direct APIs | Procurement, Company Groups, Younium, Garba (Direct); PostHog, HubSpot (MCP) |
| **Store & Join** | Supabase (Postgres) | org_nr joins, pgvector embeddings, auth |
| **Intelligence** | Claude API + Voyage AI | Punch cards, NL search, semantic search |
| **Interface** | Next.js + Vercel | React frontend + API routes, deployed |

---

**Last Updated:** Feb 12, 2026
**Build Phase:** In Progress
**Next Milestone:** Interactive prototype by Feb 17, 2026 (stakeholder testing)
