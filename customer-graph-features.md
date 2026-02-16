# Tendium Customer Graph — Feature Specification

**Version:** 1.0
**Last Updated:** Feb 12, 2026
**Status:** Feature refinement complete, ready for implementation planning

---

## Overview

The Tendium Customer Graph is an AI-first internal customer intelligence layer that unifies fragmented data across Sales, CS, Marketing, Product, Leadership, Commercial Ops, and RevOps. It eliminates data silos, conflicting definitions, and enables teams to see "the whole elephant" — complete customer context in seconds.

**Core Philosophy:**
- **Search-first, not dashboard-first** — Ask questions, don't scroll through charts
- **Source traceability** — Every data point links back to Younium, PostHog, Garba, Postmark
- **Visual + Text intelligence** — Combine AI-generated summaries with visual storytelling
- **People-centric** — Distinguish buyers from users, champions from dormant accounts
- **Progressively intelligent** — Phase 1 = answers questions, Phase 2 = detects patterns, Phase 3 = takes action

---

## Feature Roadmap

| Phase | Features | Status | Target |
|-------|----------|--------|--------|
| **Phase 1** | Search, Glossary, Profile, People, Timeline, Briefs, Relationships, Export | 🟢 Buildable | 6-week sprint (Q1 2026) |
| **Phase 2** | Signals, Insights, Agents | 🟡 Roadmap | Post-MVP |
| **Phase 3** | Tendium Activity, Predict | 🔵 Advanced | Future |

**Total:** 13 features (8 Phase 1 + 3 Phase 2 + 2 Phase 3)

---

## Phase 1 Features — Buildable in 6 Weeks

### 1. Search 🔍

**One-line:** Semantic search across all customer data + contextual AI Q&A

**Description:**
Semantic search that handles both finding ("Norwegian customers with high procurement spend") and understanding ("Why did X churn?"). Includes Perplexity-style contextual suggestions and drill-down search within specific customers.

**Key Capabilities:**
- Global search across all customers
- Natural language queries (not SQL, not filters)
- Contextual suggestions ("People also asked...")
- Drill-down search within a specific customer
- Answer questions with citations to source data

**Data Sources:**
- All unified data (Procurement, Younium, PostHog, Garba, Postmark)
- Vector embeddings for semantic matching
- LLM for natural language understanding

**Why It Matters:**
- Replaces digging through multiple tools
- Ask questions in plain English, get instant answers
- Find customers you didn't know existed (semantic matching > keyword search)

**Teams Benefiting:**
- 🎯 **All Teams** — Universal utility, everyone searches

---

### 2. Glossary 📖

**One-line:** Single source of truth for metric definitions, showing aligned and conflicting terms

**Description:**
Living dictionary of all customer metrics and terms used across teams. Shows aligned definitions (ARR, MRR, churn) and conflicts where teams disagree (MQL ≠ MQL). Manual entry in Phase 1, AI conflict detection added in Phase 2. Solves the "elephant problem" by making definition drift visible.

**Key Capabilities:**
- **Aligned definitions** — Metrics everyone agrees on
- **Conflict highlighting** — Where teams use different definitions for same term
- **Source attribution** — Which team owns which definition
- **Usage tracking** — How often each definition is referenced
- **Proposed canonical** — (Phase 2) AI-suggested standard definitions
- **Definition history** — Track changes over time

**Data Sources:**
- Manual team input (Phase 1)
- Search query analysis (Phase 2 - AI detects usage patterns)
- Brief generation logs (Phase 2 - see which definitions are used)
- Tool usage metadata (Phase 2 - detect drift)

**Why It Matters:**
- **Solves elephant problem** — "MQL ≠ MQL" made visible and resolvable
- **Organizational alignment** — Get teams on same page
- **Trust foundation** — Search and Briefs need canonical definitions
- **Reduces miscommunication** — Sales and CS use same churn definition

**Teams Benefiting:**
- 🎯 **All Teams** — Universal reference, eliminates definition confusion
- 🎯 **Leadership** — Organizational alignment tool
- 🎯 **Commercial Ops** — Define BOWTIE funnel stages consistently
- ✅ **Product** — Align activation/retention definitions

**Phase 1 Implementation:**
- Simple table: Term | Definition | Team | Last Updated
- Manual CRUD operations (add, edit, delete definitions)
- Conflict detection: flag when same term has 2+ definitions
- Search integration: link to glossary when terms appear

**Phase 2 Enhancement:**
- AI analyzes search queries to detect unstated definitions
- Propose canonical definitions based on majority usage
- Track definition drift over time
- Alert when team starts using term differently

---

### 3. Profile 👤

**One-line:** Complete customer view with deep links to source systems

**Description:**
Complete customer 360 view showing current state: customer acquisition date, subscription tier, health indicators, revenue metrics, product usage, procurement activity. Every data point clickable to verify in source system (Younium, PostHog, Garba, Postmark).

**Key Capabilities:**
- Customer acquisition date (lifecycle anchor)
- Current tier, MRR/ARR, contract details
- Health score (usage, engagement, churn risk)
- Deep links to source systems for verification
- Quick stats (users, calls, emails, procurement bids)

**Data Sources:**
- Younium (subscription, revenue)
- PostHog (product usage, activation)
- Procurement data (bid history, win rate)
- Garba (call recordings, sentiment)
- Postmark (email engagement)

**Why It Matters:**
- Single source of truth for "what's the current state?"
- Trust through source traceability (click any metric to see raw data)
- Customer acquisition date = critical lifecycle marker

**Teams Benefiting:**
- 🎯 **All Teams** — Universal reference point for customer state

---

### 4. People 👥

**One-line:** Map buyers, champions, and users within each account

**Description:**
Distinguish who signed the contract (buyer) from who uses the product daily (users). Identify power users for expansion opportunities and dormant buyers at churn risk. Critical B2B account intelligence showing stakeholder map with roles and engagement levels.

**Key Capabilities:**
- Buyer identification (contract signer, billing contact)
- User activity levels (power users, regular, dormant)
- Champion detection (high engagement, feature adoption)
- Org map visualization (hierarchy + usage heatmap)
- Individual engagement timelines

**Data Sources:**
- Younium (buyer, billing contact, contract owner)
- OneFlow (contract signer)
- PostHog (individual user activity, feature usage)
- Garba (who's on calls, role mentioned)
- Procurement data (decision-makers in bid documents)

**Why It Matters:**
- **Churn risk:** Users love it but buyer doesn't see value = at risk
- **Expansion:** Find power users to champion upsell
- **Sales efficiency:** Know who to talk to (buyer) vs who to enable (users)
- **Product gaps:** Buyer never logs in = missing executive-level features

**Teams Benefiting:**
- 🎯 **Sales** — Know who to talk to (decision-maker vs end-user)
- 🎯 **CS** — Identify champions for expansion, dormant buyers as churn risk
- ✅ **Product** — Understand different needs (buyer vs user features)
- ✅ **Leadership** — Strategic account planning (who influences decisions?)

---

### 5. Timeline 📅

**One-line:** Chronological feed of everything that happened, with citations

**Description:**
Unified timeline showing procurement bids, product usage milestones, calls, invoices, emails — everything in chronological order with source citations. Click any event to jump to source system. Includes "Became customer" lifecycle marker.

**Key Capabilities:**
- Chronological feed (reverse chronological by default)
- Event types: procurement, usage, calls, invoices, emails
- Source citations (click to verify in source system)
- Lifecycle markers ("Became customer", "First user", "Churned")
- Filter by event type, date range, data source

**Data Sources:**
- All sources with timestamps
- Procurement data (bid submitted, award decision)
- Younium (subscription events, invoices)
- PostHog (usage milestones, feature adoption)
- Garba (call recordings with timestamps)
- Postmark (email sent/opened/clicked)

**Why It Matters:**
- "What happened?" vs "What's the current state?" (Profile)
- Context for churn, expansion, support issues
- Understand customer journey end-to-end
- Debug account issues by replaying history

**Teams Benefiting:**
- 🎯 **All Teams** — Universal chronological context (everyone needs history)

---

### 6. Briefs 🃏

**One-line:** AI-generated customer intelligence summaries with citations

**Description:**
One-page punch card with ICP fit, current state, opportunities, risks, recommended actions. Auto-updates as data changes. Every claim cited with source links. Think "AI-generated exec summary" that's always fresh.

**Key Capabilities:**
- ICP fit score (quantitative + reasoning)
- Current state summary (ARR, users, health)
- Opportunities (upsell, cross-sell, referral)
- Risks (churn signals, dormant users, low engagement)
- Recommended actions (prioritized by impact)
- Auto-regenerate when data changes

**Data Sources:**
- All unified data
- LLM for summarization + reasoning
- ICP definition (procurement spend, market, vertical)
- Historical patterns (what led to churn/expansion in similar accounts)

**Why It Matters:**
- Pre-call prep in 30 seconds (not 30 minutes)
- Always up-to-date (not stale deck from 3 months ago)
- Cited claims (trust through source links)
- Pattern recognition across similar accounts (LLM strength)

**Teams Benefiting:**
- 🎯 **Sales** — Pre-call prep, account planning
- 🎯 **CS** — QBR prep, health checks, renewal strategy
- ✅ **Leadership** — Account reviews, strategic decisions
- ✅ **Commercial Ops** — Pipeline quality assessment

---

### 7. Relationships 🔗

**One-line:** Co-bidding network and competitive landscape

**Description:**
Which companies bid on same procurements? Win rate vs B2G revenue scatter plot. Competitive intelligence for sales positioning. Based on Jens's Sales Graph Explorer pattern — proven to change sales team workflow.

**Key Capabilities:**
- Co-bidding graph (companies connected via procurement bids)
- Competitive landscape visualization (win rate vs B2G revenue)
- Shared procurement analysis (which tenders did we both bid on?)
- Win/loss patterns against specific competitors
- Market positioning insights

**Data Sources:**
- 90,000 structured procurement award decisions (Nordic data, with 14,000 more in pipeline)
- Customer database (who's using Tendium)
- Company metadata (industry, size, location)

**Why It Matters:**
- Sales intelligence: "Who else are they working with?"
- Competitive positioning: "How do we stack up?"
- ICP refinement: "Which co-bidders convert to customers?"
- Proven adoption: Sales team changed workflow because of this

**Teams Benefiting:**
- 🎯 **Sales** — Competitive intelligence, lead generation via co-bidding
- 🎯 **Marketing** — ICP refinement, market positioning
- ✅ **Leadership** — Market strategy, expansion planning
- ✅ **Commercial Ops** — Win/loss analysis, competitive benchmarking

---

### 8. Export 📊

**One-line:** Generate presentation decks on demand

**Description:**
Type company name, get slide deck in seconds. Same intelligence as Briefs but formatted for presentations. Auto-generated from unified customer data. Inspired by Jens's Sales Graph Explorer's Google Slides export.

**Key Capabilities:**
- One-click deck generation (company name → full deck)
- Pre-built templates (sales pitch, QBR, account review)
- Auto-populated slides (metrics, charts, competitive landscape)
- Customizable sections (pick what to include)
- Export formats (Google Slides, PowerPoint, PDF)

**Data Sources:**
- All unified data (same as Briefs)
- Visualization templates
- Chart generation (ARR trends, usage charts, competitive plots)

**Why It Matters:**
- 7-star experience: deck in seconds, not hours
- Consistent quality (not manual copy/paste errors)
- Always current (live data, not stale screenshots)
- Behavioral adoption proof: Sales Graph Explorer's deck feature drove usage

**Teams Benefiting:**
- 🎯 **Sales** — Pitch decks, account reviews, prospect presentations
- 🎯 **CS** — QBR decks, health reports, renewal presentations
- ✅ **Leadership** — Board decks, investor updates, strategic reviews
- ✅ **Marketing** — Case studies, competitive analysis, market reports

---

## Phase 2 Features — Coming Soon

### 9. Signals 🎯

**One-line:** AI-detected patterns, anomalies, and opportunities

**Description:**
Not just threshold alerts — reasoned insights with explanations. "Usage dropped 40% after price increase, pattern matches 12 other churn cases." The intelligence engine behind autonomous workflows. Detects patterns humans miss.

**Key Capabilities:**
- Anomaly detection (usage drops, engagement spikes)
- Pattern matching (similar to other churn/expansion cases)
- Opportunity detection (upsell signals, referral potential)
- Risk scoring with reasoning (why is this account at risk?)
- Proactive alerts (not reactive dashboards)

**Data Sources:**
- All unified data
- Historical patterns (what led to churn/expansion)
- LLM for causal reasoning
- Statistical models for anomaly detection

**Why It Matters:**
- Catch issues before they become problems
- Find opportunities before competitors do
- Reasoned insights (not just "metric went down")
- Foundation for autonomous agents (Phase 2)

**Teams Benefiting:**
- 🎯 **CS** — Churn alerts, health deterioration warnings
- 🎯 **Sales** — Upsell opportunities, expansion signals
- ✅ **Product** — Usage anomalies, feature adoption patterns
- ✅ **RevOps** — Revenue risk early warnings
- ✅ **Commercial Ops** — Pipeline quality signals, conversion blockers

---

### 10. Insights 💡

**One-line:** Visual customer storytelling through charts and trends

**Description:**
Charts showing ARR trends, feature adoption curves, procurement win rates, user growth, health score over time. Different from Briefs (text summaries) — this is visual/graphical analysis. Shows customer story through data visualization.

**Key Capabilities:**
- ARR trajectory (historical + forecast)
- Feature adoption curves (S-curve visualization)
- Procurement win rate trends
- User growth within account (by role, by department)
- Health score trend (current vs historical)
- Engagement patterns (calls, emails, usage over time)
- Customizable dashboards (per team, per use case)

**Data Sources:**
- All time-series data from unified sources
- Visualization library (charts, graphs, heatmaps)
- Statistical analysis (trend detection, forecasting)

**Why It Matters:**
- Visual patterns easier to spot than text
- Complement to Briefs (text + visual = complete story)
- Different from Tendium Activity (customer analytics vs tool analytics)
- Answers "How are they trending?" vs "What's the current state?" (Profile)

**Teams Benefiting:**
- 🎯 **Leadership** — Strategic trends, portfolio health, market insights
- 🎯 **RevOps** — Revenue forecasting, retention trends, ARR trajectory
- ✅ **Product** — Usage patterns, feature adoption curves, engagement trends
- ✅ **Commercial Ops** — Conversion analysis, funnel visualization, pipeline trends

---

### 11. Agents 🤖

**One-line:** AI teammates that act autonomously

**Description:**
Starts with Sales Agent (creates upsell lists, drafts outreach, generates decks). Expands to CS, Marketing, Product, Leadership, Commercial Ops, RevOps. Don't just show data — take action. The 11-star experience.

**Key Capabilities:**
- **Sales Agent:** Upsell list generation, outreach drafting, deck creation
- **CS Agent:** Churn risk monitoring, health check automation, QBR prep
- **Marketing Agent:** ICP refinement, content personalization, attribution analysis
- **Product Agent:** Feature request aggregation, usage pattern analysis, roadmap prioritization
- **Leadership Agent:** OKR tracking, market intelligence, strategic reporting
- **Commercial Ops Agent:** Pipeline quality scoring, forecast accuracy, conversion optimization
- **RevOps Agent:** Revenue forecasting, churn prediction, retention playbooks

**Data Sources:**
- All unified data
- LLM for reasoning + action planning
- Tool integrations (email, CRM, calendar, Slack)

**Why It Matters:**
- Move from "show me the data" to "do this for me"
- 10x productivity (agents work 24/7)
- Consistent quality (no manual errors)
- Foundation for autonomous workflows

**Teams Benefiting:**
- 🎯 **All Teams** — Each team gets their own specialized agent:
  - **Sales Agent** → Sales team
  - **CS Agent** → CS team
  - **Marketing Agent** → Marketing team
  - **Product Agent** → Product team
  - **Leadership Agent** → Leadership team
  - **Commercial Ops Agent** → Commercial Ops team
  - **RevOps Agent** → RevOps team

---

## Phase 3 Features — Advanced (Desktop Only)

### 12. Tendium Activity 📈

**One-line:** Usage analytics for the Customer Graph itself

**Description:**
Most searched companies, trending questions, team focus areas. Understand how your team uses the tool to surface knowledge gaps and feature needs. Meta-analytics: analytics about the analytics tool.

**Key Capabilities:**
- Most searched companies (what's trending?)
- Most asked questions (what's the team focused on?)
- Feature usage by team member
- Search pattern analysis (knowledge gaps)
- Suggested improvements (based on usage patterns)

**Data Sources:**
- Customer Graph usage logs
- Search queries, page views, feature clicks
- User activity tracking

**Why It Matters:**
- Product improvement: "What features aren't being used?"
- Knowledge gaps: "What questions can't the team answer?"
- Adoption tracking: "Is the team actually using this?"
- Training opportunities: "Who needs help with advanced features?"

**Teams Benefiting:**
- 🎯 **Product** — Feature usage tracking, adoption analytics
- 🎯 **Leadership** — Tool adoption monitoring, ROI measurement
- ✅ **Commercial Ops** — Workflow optimization, training needs

---

### 13. Predict 🔮

**One-line:** ML-powered forecasts with confidence and explanations

**Description:**
"Will they churn?" "78% probability because usage dropped 40% after price increase, last 3 similar cases churned within 90 days." Prediction with reasoning. Not just "here's a number" — explain why.

**Key Capabilities:**
- Churn prediction (probability + reasoning)
- Expansion prediction (upsell likelihood + why)
- Lifetime value forecasting (with confidence intervals)
- Feature adoption prediction (will they use X feature?)
- Health score forecasting (trend trajectory)

**Data Sources:**
- All unified data
- Historical patterns (train on past churn/expansion)
- ML models (gradient boosting, neural nets)
- LLM for explanation generation

**Why It Matters:**
- Proactive, not reactive (prevent churn before it happens)
- Explainable AI (not black box predictions)
- Prioritization (focus on high-risk/high-opportunity accounts)
- Complement to Signals (Signals = detect patterns, Predict = forecast future)

**Teams Benefiting:**
- 🎯 **CS** — Churn prevention, health score forecasting
- 🎯 **Sales** — Expansion prioritization, upsell likelihood
- 🎯 **RevOps** — Revenue forecasting, retention modeling
- ✅ **Leadership** — Strategic planning, portfolio risk assessment

---

## Feature Comparison Matrix

| Feature | Type | Intelligence Level | Data Sources | Primary Benefit |
|---------|------|-------------------|--------------|-----------------|
| Search | Find + Ask | LLM Q&A | All sources | Answer questions instantly |
| Glossary | Reference | Manual (Phase 1), AI detection (Phase 2) | Team input, usage logs | Organizational alignment |
| Profile | View | Data aggregation | All sources | Complete current state |
| People | View | Role detection | Younium, PostHog, Garba | Buyer vs user distinction |
| Timeline | View | Chronological feed | All sources | Understand customer journey |
| Briefs | Generate | LLM summary | All sources | Pre-call prep in seconds |
| Relationships | Analyze | Graph analysis | Procurement data | Competitive intelligence |
| Export | Generate | Template automation | All sources | Decks in seconds |
| Signals | Detect | Pattern recognition | All sources | Proactive alerts |
| Insights | Visualize | Chart generation | Time-series data | Visual storytelling |
| Agents | Act | Autonomous action | All sources | 10x productivity |
| Tendium Activity | Analyze | Meta-analytics | Usage logs | Product improvement |
| Predict | Forecast | ML prediction | Historical data | Prevent churn |

---

## Data Sources Summary

| Source | Features Using It | What It Provides |
|--------|-------------------|------------------|
| **Procurement Data** | Search, Profile, Timeline, Relationships | Bid history, win rates, co-bidding graph, ICP signals |
| **Younium** | Search, Profile, People, Timeline, Briefs | Subscription data, ARR, billing, contract owner |
| **PostHog** | Search, Profile, People, Timeline, Briefs | Product usage, feature adoption, activation |
| **Garba** | Search, Profile, People, Timeline, Briefs | Call recordings, sentiment, stakeholder roles |
| **Postmark** | Search, Profile, Timeline | Email engagement, tender digest metrics |
| **OneFlow** | People | Contract signer (buyer identification) |
| **Team Input** | Glossary | Manual metric definitions, conflict reporting |
| **Usage Logs** | Glossary (Phase 2), Tendium Activity | Search patterns, definition usage, tool analytics |

---

## Success Metrics

**Phase 1 (6-week sprint):**
- ✅ All 8 features shipped and usable
- ✅ Source traceability working (click any metric → source system)
- ✅ Search handles both finding and understanding queries
- ✅ Briefs generate in <5 seconds with citations
- ✅ Teams stop building custom spreadsheets

**Behavioral Adoption (post-launch):**
- Daily active users (target: 50%+ of GTM team within 30 days)
- Search queries per user per week (target: 10+)
- Briefs generated per week (target: 50+)
- Decks exported per week (target: 20+)
- Time saved per user per week (target: 2+ hours)

**Qualitative Success:**
- "I can't do my job without this anymore" — highest bar
- Teams reference the Graph in meetings (not spreadsheets)
- Sales/CS ask "What does the Graph say?" before calls
- Leadership makes decisions based on Graph insights, not gut feel

---

## Next Steps

1. **Implementation planning** — Break down each feature into buildable chunks
2. **Data pipeline setup** — Unify Procurement + Younium + PostHog + Garba + Postmark
3. **Search infrastructure** — Vector DB + embeddings + LLM integration
4. **UI/UX design** — Linear-inspired design system (already started in build pages)
5. **Alpha testing** — Internal dogfooding with Sales/CS teams first

---

**Document Owner:** John Tan (Product Advisor)
**Stakeholder Review:** Jens Egeland (Driver), Oskar Lundgren (Co-founder/CPO), Hugo Jennerholm (Commercial Ops)
**Target Audience:** Engineering, Product, Leadership
