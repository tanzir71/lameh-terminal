# Lameh Lean Implementation Plan: One Week and Two Weeks

Goal: maximize visible stakeholder value with a 2 to 3 person team by building only on what Lameh already has: parsed filings, validated financial statements, news filtering, graph building, and LLM responses.

Do not try to build a full ontology, full agent system, perfect forecasting engine, or institutional-grade research platform in two weeks. The near-term win is evidence-backed insight generation that feels smarter than summaries but is still simple to implement.

---

## Team Assumption

| Role | If 2 Devs | If 3 Devs |
|---|---|---|
| Dev A | data/backend enrichment | data/backend enrichment |
| Dev B | LLM workflows + UI/output | LLM workflows |
| Dev C | not available | UI polish + QA/demo scripts |

Use feature flags for every add-on.

---

## One-Week Plan: Evidence-Backed Insight MVP

### Objective

Ship 3 high-value workflows that combine validated financials, news, charts, and LLM reasoning:

1. Earnings quality check.
2. News sentiment and catalyst brief.
3. Sector-specific investor prompt output with suggested charts.

### Recurring Product Loop

The feature should create a loop users return to every week or after every new filing.

```text
new filing/news arrives
-> update financial flags and news signals
-> compare with previous run
-> classify insight as new, worsening, improving, confirmed, rejected, or resolved
-> generate watchlist brief
-> suggest graph/export
-> save next proof metric
```

This makes the output operational: users are not just reading an answer, they are monitoring a thesis.

### What To Build

| Feature | Minimal Version | Why It Matters |
|---|---|---|
| Financial signal flags | rule-based flags from existing financial tables | instantly makes filings feel analytical |
| News sentiment/event tags | sentiment + relevance + event type + expected line item | improves news filtering without heavy ontology |
| Evidence bundle builder | one JSON object combining financials, news, source ids, graph-ready metrics | gives LLM grounded context |
| LLM answer contract | fact base, hypothesis, proof metric, suggested chart | prevents generic summaries |
| 6 recurring prompts | cash quality, news confirmation, real estate financing, healthcare/pharmacy, retail, contractors | creates repeatable user workflows |

### Financial Signal Flags

Implement transparent rules, not ML. Each flag should be reusable as a saved watchlist condition, not only a one-time demo insight.

Each signal should output:

```text
flag_id
company
sector
period
severity: low | medium | high
trend: new | repeated | worsening | improving | resolved
evidence_metric
comparison_period
source_table
suggested_chart
analyst_hypothesis
confirming_metric_next_period
```

#### Core Signal Catalog

| Signal | Rule | Purpose | Suggested Chart |
|---|---|---|---|
| `cash_quality_gap` | net income up YoY while operating cash flow down YoY | detect profit not backed by cash | net income vs operating cash flow |
| `receivables_pressure` | receivables growth exceeds revenue growth by 15 percentage points | detect weak collection or aggressive revenue recognition | revenue growth vs receivables growth |
| `inventory_build` | inventory growth exceeds revenue growth by 15 percentage points | detect demand slowdown, supply build, or stocking ahead of demand | revenue vs inventory |
| `margin_compression` | gross margin down YoY while revenue up YoY | detect discounting, COGS pressure, mix change | revenue vs gross margin |
| `cogs_pressure` | COGS / revenue up YoY by more than 2 percentage points | isolate input-cost pressure | COGS % revenue |
| `opex_drag` | opex growth exceeds revenue growth by 10 percentage points | detect expansion not yet operating-leveraged | revenue growth vs opex growth |
| `finance_cost_pressure` | finance cost / revenue up YoY | detect rate/debt burden | finance cost / revenue |
| `debt_load_rising` | debt up YoY while cash flow down YoY | detect balance sheet stretch | debt vs operating cash flow |
| `dividend_cash_gap` | dividends exceed operating cash flow or free cash flow proxy | detect unsustainable income profile | dividends vs cash flow |
| `segment_mix_shift` | lower-margin segment grows faster than company revenue | explain margin changes through mix | segment revenue + margin |
| `one_off_profit_dependence` | profit growth driven by non-operating income, fair-value gain, impairment reversal, or disposal gain | separate recurring from accounting profit | recurring vs non-recurring profit bridge |
| `capex_burden` | capex/depreciation high while cash flow weak | detect expansion cash drain | capex vs operating cash flow |
| `lease_burden` | lease liabilities or rent cost rising faster than revenue | useful for retail, healthcare, education | revenue vs lease/rent cost |
| `working_capital_stretch` | receivables + inventory up while cash flow down | broad quality warning | working capital vs cash flow |
| `peer_divergence` | target metric diverges from sector median by threshold | distinguish company-specific vs sector issue | target vs peer median |

#### Sector-Specific Flags

| Sector | Signal | Rule | End-User Use |
|---|---|---|---|
| Real estate developers | `inventory_to_cash_delay` | inventory up, receivables up, operating cash flow weak | spot land/project stories not converting to cash |
| REITs/property | `income_quality_gap` | dividends high but rental cash proxy/operating cash flow weak | detect yield traps |
| Contractors/projects | `contract_cash_gap` | contract/news momentum but receivables or contract assets rising faster than revenue | avoid treating awards as cash earnings |
| Banks | `credit_cost_turn` | provisions rising faster than loan growth or management language worsens | early stress signal |
| Insurers | `claims_pressure` | claims ratio/claims cost rising faster than premiums | underwriting quality signal |
| Pharmacy/healthcare retail | `store_growth_stretch` | revenue up but inventory/receivables/leases rise faster | expansion quality signal |
| Hospitals | `utilization_quality_gap` | revenue up but receivables and capex burden worsen | payer mix or expansion risk |
| Food/retail | `promotion_margin_tradeoff` | revenue up, gross margin down, inventory up | demand may be promotional not structural |
| Education | `capacity_not_matured` | capex/leases up before margin or cash collection improves | new campus utilization risk |
| Telecom/payments | `adoption_not_monetized` | usage/adoption news positive but ARPU/segment revenue/cash flow not improving | separates hype from monetization |
| Logistics/importers | `freight_cost_pass_through` | COGS/revenue or inventory up after logistics news | supply-chain exposure |

#### Severity Rules

```text
low:
one-period flag, small magnitude, no negative news support

medium:
two consecutive periods or one high-magnitude move with related news/management commentary

high:
three consecutive periods, peer divergence, cash-flow impact, or management/news confirmation
```

#### Recurring Watchlist Behavior

```text
new:
flag appears for first time in 4 quarters

worsening:
same flag appears again and metric deteriorates

improving:
flag remains but metric improves

resolved:
flag disappears and confirming metric improves
```

This turns the feature into a weekly/monthly monitoring product: "what changed, what is still unresolved, and what should I watch next?"

### News Sentiment MVP

The MVP should be more than positive/negative labels. It should convert news into recurring, filterable market signals tied to financial proof metrics.

```text
sentiment: positive | negative | neutral | mixed | uncertain
relevance: high | medium | low
event_type: see taxonomy below
time_horizon: immediate | 1q | 2-4q | long_term | unknown
financial_line_item: revenue | gross_margin | COGS | opex | receivables | inventory | debt | finance_cost | provisions | dividends | capex | cash_flow | unknown
confidence: high | medium | low
source_id
linked_company_ids
```

Implementation options:

- Fastest: LLM structured extraction with JSON schema.
- More scalable: transformer classifier later.
- Do not train a model in week one.

#### Event Taxonomy

| Event Type | Examples | Financial Proof Metric |
|---|---|---|
| `earnings` | results, profit warning, restatement | revenue, margin, net income, cash flow |
| `dividend` | dividend cut/increase, policy change | dividend coverage, cash balance, free cash flow proxy |
| `contract_project` | awards, backlog, project milestones | revenue, receivables, contract assets, gross margin |
| `rates_financing` | mortgage offers, rate cuts, refinancing | finance cost, debt, receivables, real estate sales |
| `real_estate_demand` | transactions, launches, occupancy, rents | segment revenue, inventory, receivables, rental income |
| `healthcare_demand` | hospital expansion, pharmacy growth, insurance changes | revenue, receivables, inventory, margins |
| `retail_consumer` | promotions, inflation, footfall, store expansion | revenue, gross margin, inventory, lease cost |
| `bank_credit` | provisions, NPL commentary, loan growth | provisions, loans, deposits, finance income |
| `insurance_claims` | medical/motor claims, pricing, regulation | premiums, claims cost, underwriting margin |
| `logistics_supply_chain` | freight, ports, Red Sea, import delays | COGS/revenue, inventory, logistics revenue |
| `regulation_policy` | Saudization, tariffs, fees, licensing, tax | opex, COGS, revenue, margin |
| `governance_related_party` | board changes, related parties, resignations | related-party amounts, governance disclosures |
| `litigation_compliance` | lawsuits, penalties, investigations | provisions, contingent liabilities, opex |
| `macro_commodity` | oil, petrochemicals, FX, inflation | revenue, COGS, margins, capex, provisions |

#### Sentiment Scoring Rules

```text
positive:
news likely improves revenue, margin, cash flow, dividend coverage, or balance sheet quality

negative:
news likely worsens revenue, margin, cash flow, credit quality, claims, leverage, or governance risk

mixed:
news has both upside and downside channels, such as lower rates helping real estate demand but pressuring bank spreads

neutral:
news is factual but no clear financial line item is affected

uncertain:
news is vague, speculative, duplicated, or lacks company-specific connection
```

#### Relevance Scoring Rules

```text
high:
named company, official source, direct financial line item, or large contract/regulatory impact

medium:
sector-level effect, customer/supplier/channel implication, or repeated news cluster

low:
general macro/news background with no company-specific evidence
```

#### Aggregated News Signals

For each company/week:

```text
news_sentiment_score =
  weighted sentiment by relevance, source quality, recency, and event type

news_pressure_flag:
high-relevance negative or mixed news cluster with no filing confirmation yet

news_confirmation_flag:
news signal later appears in financial statements or board/earnings commentary

news_filing_gap:
high-relevance news exists but no related official disclosure or financial proof metric yet
```

This creates recurring value because the user can ask: "What news is confirmed by numbers, what remains a hypothesis, and what should I monitor next quarter?"

#### News Tag JSON Schema

```json
{
  "article_id": "news_123",
  "company_ids": ["4300"],
  "sector_tags": ["real_estate"],
  "sentiment": "mixed",
  "relevance": "high",
  "event_type": "rates_financing",
  "expected_financial_line_items": ["finance_cost", "receivables", "inventory"],
  "time_horizon": "2-4q",
  "summary_fact": "Banks launched lower-rate mortgage offers.",
  "analyst_hypothesis": "Developers may see improved collections before earnings recover.",
  "proof_metric": "receivables growth vs revenue growth",
  "disconfirming_metric": "receivables continue growing faster than revenue",
  "confidence": "medium",
  "needs_human_review": false
}
```

Use this schema for search filters, LLM context, watchlist triggers, and later model training.

### Evidence Bundle Shape

```json
{
  "company": "string",
  "sector": "string",
  "periods": ["Q1 2025", "Q2 2025"],
  "financial_metrics": {
    "revenue": [],
    "gross_margin": [],
    "cogs_to_revenue": [],
    "net_income": [],
    "operating_cash_flow": [],
    "receivables": [],
    "inventory": [],
    "debt": [],
    "finance_cost": [],
    "dividends": []
  },
  "signal_flags": [
    {
      "flag_id": "cash_quality_gap",
      "severity": "medium",
      "trend": "worsening",
      "evidence_metric": "operating_cash_flow",
      "suggested_chart": "net_income_vs_operating_cash_flow"
    }
  ],
  "news_items": [
    {
      "title": "string",
      "date": "string",
      "sentiment": "mixed",
      "event_type": "rates",
      "relevance": "high",
      "financial_line_item": "finance_cost",
      "source_id": "news_123"
    }
  ],
  "hypotheses": [
    {
      "claim": "Lower mortgage offers may improve collections before earnings recover.",
      "proof_metric": "receivables growth vs revenue growth",
      "disconfirming_metric": "receivables continue rising faster than revenue"
    }
  ],
  "available_charts": [],
  "source_documents": [],
  "watchlist_rules": []
}
```

### Recurring Insight Record

Save generated insights as structured records so users can revisit them weekly or after the next filing.

```text
insight_id
company_id
sector
created_at
insight_type: cash_quality | sentiment_shift | peer_divergence | project_cash_conversion | margin_pressure | dividend_quality
status: active | monitoring | confirmed | rejected | resolved
fact_base
hypothesis
proof_metric
next_check_date
source_ids
chart_ids
watchlist_rule
```

This is the difference between a demo and a product habit: the same hypothesis can be rechecked when new filings or news arrive.

### LLM Output Contract

```text
Return:
1. Fact base from filings and news.
2. Analyst hypothesis.
3. Financial line item that should confirm or reject the hypothesis.
4. Signal flags with severity and trend.
5. News sentiment/event signals.
6. Suggested chart.
7. Watchlist rule to re-run.
8. Source ids used.
9. Confidence: high / medium / low.

Rules:
- Do not make unsupported forecasts.
- Label all assumptions as hypotheses.
- Use financial statements and news as primary evidence.
- Convert useful answers into a reusable watchlist rule.
```

### One-Week Recurring Prompt Pack

These are not demo-only prompts. Each one should produce a reusable insight record and watchlist rule.

#### Prompt 1: Cash Quality Monitor

```text
Question:
Which companies reported stronger profit but weaker cash quality, and what should I monitor next?

Universe:
{{market_or_watchlist}}

Required evidence:
- revenue, net income, operating cash flow, receivables, inventory, debt, dividends
- latest filings and board/earnings commentary
- latest high-relevance news

Analyze:
- flag profit up but cash flow down
- flag receivables or inventory growing faster than revenue
- separate recurring profit from one-off gains if available
- compare with sector median

Return:
ranked table:
company | sector | cash quality flag | severity | trend | filing evidence | news support | hypothesis | proof metric next quarter | chart | watchlist rule.
```

#### Prompt 2: News Confirmed By Numbers

```text
Question:
Which positive or negative news stories are already confirmed by financial statements, and which remain unproven?

Universe:
{{company_or_sector}}

Required evidence:
- latest news items with sentiment, relevance, and event type
- validated financial metrics over 4 to 8 quarters
- management commentary if available

Analyze:
- group news by event type
- map each news cluster to expected line item
- classify as confirmed, partially confirmed, not yet visible, contradicted, or too vague

Return:
news cluster | sentiment | relevance | expected line item | filing evidence | status | hypothesis | next proof metric | source ids.
```

#### Prompt 3: Real Estate Financing Impact Watch

```text
Question:
If financing conditions improve, which real estate-linked companies should show proof first?

Universe:
developers, REITs, banks, cement, home retail, insurers

Required evidence:
- receivables, inventory, rental income, finance cost, debt, dividends, cash flow
- mortgage/rate news and bank financing campaigns
- board/earnings commentary on demand, collections, occupancy, refinancing

Analyze:
- identify first proof metric by company type
- classify current evidence as confirmed, early signal, hypothesis only, or contradicted
- compare with previous easing or financing campaign periods if data exists

Return:
company | sector | financing exposure | current evidence | first proof metric | historical pattern | 2q scenario | disconfirming metric | chart | watchlist rule.
```

#### Prompt 4: Healthcare and Pharmacy Expansion Quality

```text
Question:
Are healthcare and pharmacy companies expanding profitably, or stretching working capital?

Universe:
{{healthcare_watchlist}}

Required evidence:
- revenue, gross margin, inventory, receivables, payables, leases, capex, operating cash flow
- board/earnings commentary on stores, clinics, hospitals, acquisitions, payer mix
- news on regulation, insurance, expansion, M&A

Analyze:
- distinguish same-store/organic growth, acquisition growth, and new capacity
- flag inventory or receivables pressure
- flag margin improvement or deterioration
- identify whether expansion is cash-generative yet

Return:
company | growth source | margin signal | working-capital signal | cash conversion | news/management support | hypothesis | next proof metric | watchlist rule.
```

#### Prompt 5: Retail Inflation Pass-Through

```text
Question:
Which retailers or food companies are passing inflation through without damaging demand or margins?

Universe:
{{retail_or_food_sector}}

Required evidence:
- revenue, gross margin, COGS/revenue, inventory, opex, lease cost, cash flow
- news on inflation, promotions, Ramadan/Eid, footfall, consumer demand
- board/earnings commentary on pricing and suppliers

Analyze:
- separate healthy price pass-through from promotional revenue growth
- flag revenue up with gross margin down
- flag inventory buildup
- adjust commentary for seasonal events

Return:
company | demand signal | pricing power evidence | margin impact | inventory risk | news support | hypothesis | proof metric | chart | watchlist rule.
```

#### Prompt 6: Contractor Project Cash Conversion

```text
Question:
Which project-exposed contractors or suppliers have real project momentum, and which are only accumulating receivables?

Universe:
contractors, cement, steel, pipes, cables, logistics, facilities management

Required evidence:
- revenue, gross margin, receivables, contract assets, payables, operating cash flow
- contract announcements and project news
- board/earnings commentary on project delays, execution, customer concentration

Analyze:
- distinguish contract/news momentum from cash conversion
- flag receivables or contract assets growing faster than revenue
- compare with peers

Return:
company | project/news evidence | financial confirmation | receivables risk | cash conversion | margin trend | hypothesis | next proof metric | watchlist rule.
```

### One-Week Delivery Checklist

| Day | Output |
|---|---|
| Day 1 | define signal flag SQL/helpers, evidence bundle schema, 6 recurring prompt templates |
| Day 2 | implement financial signal flags and source-linked output |
| Day 3 | implement news sentiment/event tagging via LLM JSON extraction |
| Day 4 | implement LLM answer contract and suggested-chart output |
| Day 5 | polish recurring flows, QA 10 companies, write stakeholder demo script |

### Week One Success Criteria

- User can ask a company/sector question and get a source-backed answer.
- Answer includes at least one financial flag, one news signal, one hypothesis, and one suggested chart.
- Numbers come from validated financials, not free-form LLM extraction.
- Recurring prompt pack works across at least 5 sectors.

---

## Two-Week Plan: Add Relationship-Lite and Historical Context

### Objective

Add enough relationship and historical reasoning to make answers feel expert without building a full ontology.

### What To Add

| Feature | Minimal Version | Avoid |
|---|---|---|
| Relationship-lite extraction | source-backed links from filings/news | full knowledge graph |
| Historical analog search | rule-based metric/event matching | black-box prediction |
| Peer divergence detector | target vs sector median | complex factor model |
| Watchlist rules | save reusable alert conditions | full alerting platform |
| Board/demo export | markdown or simple PDF-ready brief | full presentation builder |

### Relationship-Lite

Store proposed relationships in a simple table first.

```text
relationship_id
source_id
source_type: filing | board_report | earnings_call | news
entity_a
relationship_type
entity_b
evidence_excerpt
confidence: high | medium | low
review_status: proposed | accepted | rejected
```

Initial relationship types:

```text
customer_of
supplier_of
tenant_of
contractor_for
project_related_to
regulated_by
related_party_to
co_mentioned_with
exposed_to
```

Guardrail:

```text
co_mentioned_with is not causality.
Only accepted/source-backed relationships can influence scoring.
```

### Historical Analog MVP

Use simple matching:

```text
Inputs:
- metric flags
- sector
- event type
- direction of change
- 4 to 8 quarter window

Output:
- prior periods with similar pattern
- what happened next 1q / 2q / 4q
- why today may differ
```

Example:

```text
current pattern:
revenue up, gross margin down, receivables up, negative news sentiment

find:
same sector companies with similar pattern over last 5 years
```

### Peer Divergence Detector

```text
If target metric differs from sector median by threshold:
show:
- target trend
- sector median trend
- likely explanation from filings/news
- whether divergence is company-specific or sector-wide
```

Start with:

- revenue growth
- gross margin
- COGS / revenue
- operating cash flow / net income
- receivables growth vs revenue growth
- finance cost / revenue

### Two-Week Delivery Checklist

| Day | Output |
|---|---|
| Day 6 | relationship-lite table and extraction prompt |
| Day 7 | relationship review/admin view or simple accepted/rejected workflow |
| Day 8 | historical analog search using existing metrics and event tags |
| Day 9 | peer divergence detector and chart specs |
| Day 10 | final demo pack: 12 recurring workflows, 20 companies, source-backed outputs |

### Two-Week Recurring Prompt Pack

#### Prompt 7: Peer Divergence Explainer

```text
Question:
Which companies are moving differently from sector peers, and is the divergence explained by filings or news?

Universe:
{{sector_or_watchlist}}

Required evidence:
- target company metrics and sector median for revenue growth, gross margin, COGS/revenue, cash conversion, receivables, inventory, finance cost
- latest filings, board/earnings commentary, and high-relevance news

Analyze:
- detect positive and negative divergence
- classify reason: pricing, cost, mix, working capital, financing, expansion, regulation, one-off, unexplained
- show whether divergence is new, repeated, worsening, improving, or resolved

Return:
company | divergent metric | target trend | peer median | likely explanation | evidence | confidence | next proof metric | chart | watchlist rule.
```

#### Prompt 8: Historical Analog and Outcome Tracker

```text
Question:
Where have we seen this pattern before, and what happened over the next 1, 2, and 4 quarters?

Pattern:
{{signal_flags + news_event_type + sector}}

Required evidence:
- 5 years minimum of financial metrics if available
- historical signal flags
- historical news event tags
- subsequent quarterly outcomes

Analyze:
- find similar metric/event patterns
- compare current company to historical cases
- avoid a single forecast; return scenario ranges
- state why today's setup may differ

Return:
analog company/period | similarity | matching signals | what happened next 1q | next 2q | next 4q | difference today | base/upside/downside scenario | trigger metrics.
```

#### Prompt 9: Positive News Not Confirmed By Numbers

```text
Question:
Which companies have positive news momentum that is not yet confirmed by filings?

Universe:
{{market_or_watchlist}}

Required evidence:
- high-relevance positive news clusters
- revenue, gross margin, cash flow, receivables, inventory, debt/finance cost
- management commentary if available

Analyze:
- identify the expected line item for each news event
- classify timing: too early, weak evidence, contradicted, confirmed
- create watchlist rule for next filing

Return:
company | positive news cluster | expected line item | current filing evidence | status | why it matters | next proof metric | watchlist rule.
```

#### Prompt 10: Negative News Before Financial Impact

```text
Question:
Which companies have negative or mixed news that may appear in financials later?

Universe:
{{market_or_watchlist}}

Required evidence:
- high-relevance negative/mixed news
- related financial line items
- board/earnings risk commentary
- peer behavior if available

Analyze:
- map news to likely delayed financial impact
- identify whether filings already show early signs
- separate company-specific risk from sector-wide risk

Return:
company | risk event | likely line item | current evidence | expected lag | peer context | disconfirming metric | severity | watchlist rule.
```

#### Prompt 11: Board Brief Generator

```text
Question:
Build a one-page weekly board brief for this watchlist.

Watchlist:
{{company_list}}

Required evidence:
- latest signal flags
- latest sentiment/event tags
- unresolved hypotheses
- new filings or news since last brief
- suggested charts

Analyze:
- summarize only material changes
- group by cash quality, margin pressure, growth quality, news confirmation, and unresolved risks
- show what changed since the last run

Return:
1. Top 5 changes this week.
2. Companies needing attention.
3. Confirmed improvements.
4. Unresolved hypotheses.
5. Charts to open.
6. Watchlist rules created or updated.
```

#### Prompt 12: End-User Saved Monitor

```text
Question:
Create a saved monitor for {{investor_theme}}.

Examples:
- real estate financing recovery
- healthcare expansion quality
- retailers with margin pressure
- contractors with receivables risk
- banks/insurers with early stress

Required evidence:
- selected companies
- signal flags
- event tags
- proof metrics
- source ids

Analyze:
- define the monitor universe
- define trigger rules
- define what counts as confirmation, deterioration, and resolution
- define suggested charts

Return:
monitor name | company universe | trigger rules | proof metrics | refresh frequency | output format | charts | stop conditions.
```

---

## What Not To Build Yet

| Do Not Build | Reason |
|---|---|
| full ontology graph | too much schema debate for two weeks |
| MiroFish-style swarm simulation | interesting, but demo risk is high and validation is weak |
| OptiLLM production integration | evaluate later; prompt contracts and evidence bundles matter first |
| custom ML sentiment model | LLM JSON tagging is enough for MVP |
| automated forecasts | use scenarios and disconfirming metrics instead |
| full PowerPoint generator | markdown/HTML board brief is enough |
| complex RAG framework migration | use current retrieval and structured evidence bundle |

---

## The Small-Team Product Bet

The best two-week outcome is not "AI predicts the market." It is:

```text
Lameh can turn filings + news + charts into evidence-backed analyst hypotheses in minutes,
with source ids, proof metrics, and reusable watchlist rules.
```

That is realistic for 2 to 3 devs, shows clear value to private-sector investors, and creates a foundation for ontology, inference routing, and scenario simulation later.
