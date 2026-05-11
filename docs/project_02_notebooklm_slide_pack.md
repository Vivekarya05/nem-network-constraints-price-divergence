# Project 02 NotebookLM Slide Pack

# NEM Network Constraints & Price Divergence Intelligence System

## Purpose Of This Document

This document is a complete study and slide-generation pack for NotebookLM.

It summarises the full Project 2 workflow, including:

- project objective
- business question
- data sources
- notebook flow
- feature engineering
- output tables
- Plotly chart outputs
- key findings
- analyst interpretation
- dashboard concept
- final capstone relevance

This file can be uploaded to NotebookLM to generate:

- presentation slides
- a study guide
- speaker notes
- project summary
- interview preparation notes

---

# 1. Project Overview

## Project Name

NEM Network Constraints & Price Divergence Intelligence System

## Business Question

How do network constraints and interconnector flows drive price spikes, congestion, and regional price divergence in the Australian National Electricity Market?

## Project Objective

The objective of this project is to build a professional energy market analysis workflow that explains how network conditions influence regional price behaviour in the NEM.

The project focuses on:

- price spikes
- regional price divergence
- dispatch constraint activity
- interconnector flow behaviour
- congestion-style signals
- spike classification
- decision intelligence recommendations

## Market Context

In the NEM, regional prices can diverge when transmission limits, constraints, or interconnector flow conditions prevent energy from moving freely between regions.

This project investigates whether high-price events are explained only by demand and supply fundamentals, or whether network conditions also contributed.

## Study Period

- Start date: 2026-02-01
- End date: 2026-03-01

## Focus Regions

- NSW1
- VIC1

## Main Interconnector Focus

- VIC1-NSW1

## Technology Stack

- Python
- pandas
- numpy
- SQLAlchemy
- PostgreSQL
- Plotly
- Jupyter Notebook

## Data Source

The raw data comes from AEMO NEMWeb and is loaded into PostgreSQL through an existing ETL pipeline.

NEMOSIS was not used.

---

# 2. End-To-End Analytical Flow

The project was built progressively through notebooks:

```text
ETL completed externally
→ PostgreSQL raw tables
→ data extraction
→ data cleaning
→ base market features
→ constraint features
→ interconnector features
→ regional price divergence
→ spike classification
→ decision intelligence
→ Plotly visualisation pack
```

## Full Notebook Sequence

| Notebook | Name | Purpose |
|---|---|---|
| 01 | Data Extraction And Market Context | Extract and merge price, demand, generation, and interchange data. |
| 02 | Base Market Feature Engineering | Create price spike, supply-demand gap, price change, volatility, and time features. |
| 03 | Constraint Activity Analysis | Add dispatch constraint activity features. |
| 04 | Interconnector Flow Analysis | Add interconnector flow and congestion-style signals. |
| 05 | Regional Price Divergence | Calculate NSW1-VIC1 price spread and divergence flags. |
| 06 | Network Spike Classification | Classify spike intervals by likely driver. |
| 07 | Decision Intelligence | Convert classifications into recommendations, risks, and confidence. |
| 08 | Plotly Visualisation Pack | Create interactive HTML charts for reporting and dashboard design. |

---

# 3. Raw Tables Used

The project used the following PostgreSQL raw tables:

| Raw Table | Purpose |
|---|---|
| `raw.dispatch_price` | Regional RRP and price outcomes. |
| `raw.dispatch_regionsum` | Demand, available generation, and net interchange. |
| `raw.dispatch_constraints` | Dispatch constraint marginal values and violation degrees. |
| `raw.interconnector_results` | Interconnector flows by dispatch interval. |

Optional tables loaded but not deeply used in this stage:

- `raw.constraint_rhs`
- `raw.constraint_details`
- `raw.constraint_equations`

These optional tables can be used later to explain constraint equations in more detail.

---

# 4. Notebook 01: Data Extraction And Market Context

## Objective

Notebook 01 extracted NSW1 and VIC1 dispatch price and region summary data from PostgreSQL.

## Business Question

Do NSW1 and VIC1 have clean 5-minute dispatch data showing price, demand, available generation, and interchange for February 2026?

## Inputs

- `raw.dispatch_price`
- `raw.dispatch_regionsum`

## Main Steps

1. Loaded database credentials from `.env`.
2. Connected to PostgreSQL.
3. Checked raw schema tables.
4. Extracted dispatch price data.
5. Extracted dispatch region summary data.
6. Cleaned timestamps and region IDs.
7. Filtered intervention intervals.
8. Removed duplicate interval-region rows.
9. Merged price with region summary data.
10. Saved the clean base market context table.

## Output Tables

| Output | Purpose |
|---|---|
| `outputs/01_base_market_context.csv` | Clean merged price, demand, generation, and interchange table. |
| `outputs/01_base_market_context_summary.csv` | Regional summary of the base market context. |

## Analyst Interpretation

Notebook 01 created the foundation dataset for the project. Each row represents one region and one dispatch interval, with RRP, total demand, available generation, and net interchange aligned at 5-minute resolution.

This table does not yet classify spikes or explain network behaviour. It is the clean market base used by later notebooks.

---

# 5. Notebook 02: Base Market Feature Engineering

## Objective

Notebook 02 created the first layer of market features before introducing network constraints and interconnector flows.

## Business Question

Before introducing network effects, what does normal market behaviour tell us about price spikes, volatility, demand conditions, and regional tightness?

## Input

- `outputs/01_base_market_context.csv`

## Features Created

| Feature | Formula / Rule | Meaning |
|---|---|---|
| `supply_demand_gap` | `availablegeneration - totaldemand` | Measures regional headroom between available generation and demand. |
| `price_spike_flag` | `rrp > 300` | Flags intervals above $300/MWh. |
| `price_change` | current RRP minus previous interval RRP | Measures 5-minute price movement. |
| `rolling_rrp_volatility_1h` | rolling standard deviation over 12 intervals | Measures one-hour price instability. |
| `date` | extracted from timestamp | Calendar date. |
| `hour` | extracted from timestamp | Time-of-day analysis. |
| `weekday` | extracted from timestamp | Weekday pattern. |
| `is_weekend` | Saturday or Sunday | Weekend flag. |

## Important Note On Missing Values

Some `NaN` values are expected:

- `price_change` is missing for the first interval of each region because there is no previous interval.
- `rolling_rrp_volatility_1h` is missing at the start of each region because rolling windows require prior observations.

These missing values are analytically correct.

## Output Tables

| Output | Purpose |
|---|---|
| `outputs/02_base_market_features.csv` | Base market feature table. |
| `outputs/02_regional_feature_summary.csv` | Summary by region. |
| `outputs/02_spike_summary_by_region.csv` | Spike count and price summary by region. |
| `outputs/02_highest_price_intervals.csv` | Highest price intervals for review. |

## Analyst Interpretation

Using a $300/MWh threshold, NSW1 recorded price spike intervals during the study period, while VIC1 did not breach the selected spike threshold.

This does not mean VIC1 is unimportant. It means that, for this period and threshold, NSW1 is the main high-price event focus, while VIC1 remains critical for regional price divergence and interconnector analysis.

---

# 6. Notebook 03: Constraint Activity Analysis

## Objective

Notebook 03 added the first network layer by analysing dispatch constraint activity.

## Business Question

Were price spikes associated with elevated constraint activity, marginal value signals, or violation signals?

## Inputs

- `outputs/02_base_market_features.csv`
- `raw.dispatch_constraints`

## Key Market Concepts

| Concept | Meaning |
|---|---|
| Dispatch constraint | A mathematical limit used by AEMO to maintain secure dispatch. |
| Marginal value | Price impact of relaxing a binding constraint by one unit. |
| Violation degree | Indicates whether a constraint was violated. |
| Active constraint | In this project, a constraint with non-zero marginal value or violation degree. |

## Constraint Features Created

| Feature | Meaning |
|---|---|
| `active_constraint_count` | Number of active constraints in a dispatch interval. |
| `max_constraint_marginal_value` | Highest marginal value in the interval. |
| `total_constraint_marginal_value` | Sum of marginal values in the interval. |
| `violation_flag` | Whether any constraint had a non-zero violation degree. |
| `max_violation_degree` | Highest violation degree in the interval. |

## Key Result

Constraint activity comparison showed:

| Region | Spike Flag | Intervals | Average RRP | Average Active Constraints | Violation Intervals |
|---|---:|---:|---:|---:|---:|
| NSW1 | False | 8010 | 68.32 | 19.26 | 54 |
| NSW1 | True | 53 | 2417.10 | 24.42 | 0 |
| VIC1 | False | 8063 | 45.06 | 19.29 | 54 |

## Analyst Interpretation

NSW1 price spikes occurred during periods with a higher average number of active constraints.

However, the maximum constraint marginal value signal did not directly support a simple constraint-driven explanation. The largest marginal value events occurred outside the NSW1 spike intervals, and no violation intervals were observed during NSW1 spikes.

This suggests that constraints may have contributed to tighter or more complex dispatch conditions, but the spikes should not be classified as purely constraint-driven based on constraint count, marginal value, and violation signals alone.

## Output Tables

| Output | Purpose |
|---|---|
| `outputs/03_constraint_features.csv` | Interval-level constraint features. |
| `outputs/03_constraint_event_summary.csv` | Highest constraint activity intervals. |
| `outputs/03_spike_constraint_detail.csv` | Spike intervals with constraint signals. |
| `outputs/03_market_features_with_constraints.csv` | Market feature table joined with constraint features. |

---

# 7. Notebook 04: Interconnector Flow Analysis

## Objective

Notebook 04 added interconnector flow behaviour and congestion-style signals.

## Business Question

Were price spikes associated with high interconnector flow, sharp flow changes, or congestion-style network conditions?

## Inputs

- `outputs/03_market_features_with_constraints.csv`
- `raw.interconnector_results`

## Interconnector Features Created

| Feature | Formula / Rule | Meaning |
|---|---|---|
| `flow` | `mwflow` | MW flow on the interconnector. |
| `absolute_flow` | `abs(flow)` | Flow size regardless of direction. |
| `flow_change` | current flow minus previous flow | Dispatch interval movement in flow. |
| `absolute_flow_change` | `abs(flow_change)` | Size of flow movement. |
| `high_flow_flag` | absolute flow above 90th percentile for interconnector | High usage relative to own history. |
| `high_flow_change_flag` | absolute flow change above 90th percentile | Sharp flow movement. |
| `congestion_signal_flag` | high flow or high flow change | Practical congestion-style signal. |

## Important Analytical Note

The congestion signal is not a formal AEMO congestion classification.

It is a practical analytical indicator based on observed flow and flow-change behaviour.

Formal congestion analysis would require directional limits, constraint equations, and RHS data.

## Interconnector Summary Result

| Interconnector | Max Absolute Flow | Max Absolute Flow Change | Congestion Signal Share |
|---|---:|---:|---:|
| N-Q-MNSP1 | 201.60 MW | 117.70 MW | 19.32% |
| NSW1-QLD1 | 1298.52 MW | 674.01 MW | 19.30% |
| T-V-MNSP1 | 443.35 MW | 220.99 MW | 17.94% |
| V-S-MNSP1 | 213.25 MW | 117.00 MW | 19.51% |
| V-SA | 698.41 MW | 686.14 MW | 19.57% |
| VIC1-NSW1 | 1872.63 MW | 1447.09 MW | 19.83% |

## Analyst Interpretation

The `VIC1-NSW1` interconnector was especially important because it directly links the two focus regions.

It recorded:

- maximum absolute flow of approximately 1873 MW
- maximum absolute flow change of approximately 1447 MW
- congestion-style signal share of approximately 19.8%

This indicates that the NSW-VIC transfer path experienced large directional and magnitude changes during the study period.

## Spike Vs Non-Spike Interconnector Result

During NSW1 spike intervals:

- average maximum absolute interconnector flow increased from around 694 MW to 878 MW
- average maximum absolute flow change increased from around 108 MW to 143 MW
- average congestion signal count increased from around 1.15 to 1.94
- 49 of 53 NSW1 spike intervals had at least one congestion-style signal

However, direct `VIC1-NSW1` absolute flow was lower during NSW1 spike intervals than during non-spike intervals.

This suggests NSW1 price spikes were associated with broader interconnector stress across the NEM, but not necessarily direct stress on the `VIC1-NSW1` interconnector alone.

## Output Tables

| Output | Purpose |
|---|---|
| `outputs/04_interconnector_features.csv` | Interconnector-level flow features. |
| `outputs/04_interconnector_flow_summary.csv` | Summary by interconnector. |
| `outputs/04_interconnector_interval_features.csv` | Interval-level interconnector features. |
| `outputs/04_interconnector_vs_spike_summary.csv` | Spike vs non-spike flow comparison. |
| `outputs/04_market_features_with_network.csv` | Market, constraint, and interconnector features joined together. |

---

# 8. Notebook 05: Regional Price Divergence Analysis

## Objective

Notebook 05 measured NSW1-VIC1 price separation.

## Business Question

When do NSW1 and VIC1 prices separate, and can regional price divergence be linked to network constraint activity or interconnector flow behaviour?

## Input

- `outputs/04_market_features_with_network.csv`

## Divergence Features Created

| Feature | Formula / Rule | Meaning |
|---|---|---|
| `nsw_rrp` | NSW1 RRP | NSW price signal. |
| `vic_rrp` | VIC1 RRP | VIC price signal. |
| `price_spread` | `nsw_rrp - vic_rrp` | Positive means NSW1 higher than VIC1. |
| `absolute_price_spread` | `abs(price_spread)` | Size of price separation. |
| `divergence_flag` | spread above $100/MWh | Material divergence. |
| `extreme_divergence_flag` | spread above $300/MWh | Severe divergence. |
| `divergence_direction` | NSW higher / VIC higher | Direction of price separation. |

## Key Result

| Metric | Value |
|---|---:|
| Intervals | 8063 |
| Average NSW1 RRP | 83.76 |
| Average VIC1 RRP | 45.06 |
| Average price spread | 38.71 |
| Average absolute price spread | 44.93 |
| Maximum positive spread | 20308.00 |
| Maximum negative spread | -234.02 |
| Divergence intervals | 307 |
| Extreme divergence intervals | 60 |
| Divergence share | 3.81% |
| Extreme divergence share | 0.74% |

## Analyst Interpretation

NSW1 prices were higher than VIC1 on average during the study period.

Material NSW1-VIC1 price divergence occurred in 307 intervals, representing about 3.8% of the study period. Extreme divergence occurred in 60 intervals, representing about 0.74% of the period.

The maximum positive spread was very large, indicating intervals where NSW1 prices separated sharply above VIC1.

This supports the need for spike classification and decision intelligence, because price spikes should be reviewed alongside regional separation and interconnector flow signals.

## Output Tables

| Output | Purpose |
|---|---|
| `outputs/05_regional_price_divergence.csv` | Interval-level NSW1-VIC1 spread table. |
| `outputs/05_divergence_summary.csv` | Overall divergence summary. |
| `outputs/05_divergence_direction_summary.csv` | Direction of divergence summary. |
| `outputs/05_divergence_event_summary.csv` | Region-level divergence events. |
| `outputs/05_divergence_interval_summary.csv` | Interval-level divergence events. |
| `outputs/05_spike_divergence_summary.csv` | Spike vs divergence comparison. |
| `outputs/05_market_features_with_divergence.csv` | Full dataset with divergence features. |

---

# 9. Notebook 06: Network Spike Classification

## Objective

Notebook 06 converted price spike flags into market explanations.

## Business Question

Can price spikes in NSW1 and VIC1 be classified by their likely market driver using demand, volatility, constraint, interconnector, and price divergence features?

## Input

- `outputs/05_market_features_with_divergence.csv`

## Classification Categories

| Classification | Meaning |
|---|---|
| Constraint-driven | Spike occurred with elevated constraint activity or marginal value signals. |
| Interconnector/congestion-driven | Spike occurred with interconnector stress or regional price separation. |
| Demand-driven | Spike occurred during tight demand and supply conditions. |
| Volatility-driven | Spike occurred during unstable price movement. |
| Unknown | No clear dominant driver. |

## Driver Signals

| Signal | Inputs Used |
|---|---|
| Demand tightness signal | high demand or low supply-demand gap |
| Volatility signal | high rolling volatility or large price change |
| Constraint signal | high active constraint count, high marginal value, or violation flag |
| Interconnector congestion signal | congestion-style signal, divergence flag, or extreme divergence flag |

## Priority Order

The rule-based classifier used this priority order:

1. Constraint-driven
2. Interconnector/congestion-driven
3. Demand-driven
4. Volatility-driven
5. Unknown

## Important Analytical Note

This is a rule-based classification, not a causal model.

It is designed as decision-support evidence, not proof of causality.

## Key Result

| Region | Classification | Spike Intervals | Average RRP | Max RRP | Average Congestion Signal Count | Average Absolute Price Spread |
|---|---|---:|---:|---:|---:|---:|
| NSW1 | Constraint-driven | 27 | 1522.01 | 9914.02 | 1.70 | 1478.04 |
| NSW1 | Interconnector/congestion-driven | 26 | 3346.61 | 20300.00 | 2.19 | 3323.22 |

## Analyst Interpretation

NSW1 spike intervals were split almost evenly between:

- constraint-driven signals
- interconnector/congestion-driven signals

The interconnector/congestion-driven group had a higher average RRP and included the maximum RRP event of $20,300/MWh.

This suggests that the most severe price spike events were more strongly associated with interconnector stress and regional price separation than with constraint signals alone.

## Output Tables

| Output | Purpose |
|---|---|
| `outputs/06_network_spike_explanation.csv` | Event-level spike explanation table. |
| `outputs/06_spike_classification_summary.csv` | Classification summary. |
| `outputs/06_market_features_with_spike_classification.csv` | Full dataset with classification fields. |

---

# 10. Notebook 07: Decision Intelligence Recommendations

## Objective

Notebook 07 translated classified spike events into decision intelligence.

## Business Question

How can classified price spike and network signals be translated into practical recommendations for energy traders, analysts, and market operators?

## Inputs

- `outputs/06_network_spike_explanation.csv`
- `outputs/06_market_features_with_spike_classification.csv`

## Decision Fields Created

| Field | Meaning |
|---|---|
| `market_situation` | Short description of market condition. |
| `insight` | Analyst interpretation. |
| `recommendation` | Practical monitoring or review action. |
| `risk` | Main risk associated with the condition. |
| `decision_confidence` | Confidence based on classification evidence. |

## Key Result

| Region | Classification | Confidence | Event Count | Average RRP | Max RRP |
|---|---|---|---:|---:|---:|
| NSW1 | Constraint-driven | High | 27 | 1522.01 | 9914.02 |
| NSW1 | Interconnector/congestion-driven | High | 26 | 3346.61 | 20300.00 |

## Example Recommendation: Constraint-Driven

Market situation:

Price spike occurred during elevated constraint activity.

Insight:

Network constraint signals were present during the spike interval, suggesting dispatch limitations may have contributed to price formation.

Recommendation:

Review binding or high-impact constraints around the interval and monitor whether similar constraints recur during peak demand or outage conditions.

Risk:

Constraint-driven price outcomes can repeat quickly if the underlying network limitation remains active.

## Example Recommendation: Interconnector/Congestion-Driven

Market situation:

Price spike occurred during interconnector or regional separation stress.

Insight:

Congestion-style signals or NSW1-VIC1 price divergence were present, indicating that regional transfer conditions may have influenced the price outcome.

Recommendation:

Monitor interconnector flow, flow changes, and regional price spreads. Prioritise review of NSW1-VIC1 and neighbouring interconnector behaviour.

Risk:

Regional prices may separate sharply if transfer capability is limited or flows cannot respond to price differences.

## Output Tables

| Output | Purpose |
|---|---|
| `outputs/07_network_decision_recommendations.csv` | Event-level recommendation table. |
| `outputs/07_decision_recommendation_summary.csv` | Summary of recommendation themes. |

---

# 11. Notebook 08: Plotly Visualisation Pack

## Objective

Notebook 08 created interactive visualisations to support portfolio presentation, dashboard design, and analyst storytelling.

## Input Files

- `outputs/06_market_features_with_spike_classification.csv`
- `outputs/07_network_decision_recommendations.csv`
- `outputs/05_regional_price_divergence.csv`
- `outputs/04_interconnector_features.csv`

## Chart Outputs

The following Plotly HTML charts were created:

| Chart | File | Purpose |
|---|---|---|
| RRP by region with spike markers | `outputs/charts/01_rrp_by_region_with_spikes.html` | Shows NSW1 and VIC1 RRP over time with spike events highlighted. |
| NSW1 vs VIC1 RRP comparison | `outputs/charts/02_nsw_vic_rrp_comparison.html` | Shows regional prices on the same time axis. |
| NSW1-VIC1 price spread | `outputs/charts/03_nsw_vic_price_spread.html` | Shows regional price separation and divergence thresholds. |
| Constraint activity vs NSW1 RRP | `outputs/charts/04_constraint_activity_vs_nsw_rrp.html` | Compares RRP with active constraint count. |
| VIC1-NSW1 interconnector flow | `outputs/charts/05_vic_nsw_interconnector_flow.html` | Shows flow behaviour and congestion-style signals. |
| Congestion signal count | `outputs/charts/06_congestion_signal_count.html` | Shows broader interconnector stress over time. |
| Spike classification summary | `outputs/charts/07_spike_classification_summary.html` | Shows spike count by driver classification. |

## Visual Storyline

The visualisation pack supports this narrative:

1. NSW1 experienced high-price spike events.
2. VIC1 did not breach the same spike threshold.
3. NSW1 and VIC1 prices separated during material divergence intervals.
4. Constraint activity was higher during NSW1 spike intervals, but marginal value evidence was not enough to explain all spikes.
5. Broader interconnector congestion-style signals were present during most NSW1 spike intervals.
6. The largest spike outcomes were linked more strongly with interconnector/congestion and regional divergence signals.

---

# 12. Dashboard Mockups

Two dashboard mockups were created to explore future dashboard design.

## Project 2 Dashboard Mockup

File:

`mockups/nem_network_dashboard_mockup.html`

Purpose:

Shows how this Project 2 analysis could look as a single interactive dashboard.

Includes:

- slicer-style controls
- KPI cards
- RRP chart
- price spread chart
- spike classification chart
- constraint activity chart
- interconnector flow chart
- congestion signal chart
- decision recommendation cards
- event recommendation table

## Full Platform Dashboard Mockup

File:

`mockups/nem_market_intelligence_platform_mockup.html`

Purpose:

Shows how Project 2 can later become one module inside a larger NEM Market Intelligence Platform.

Includes:

- business case selector
- shared dashboard filters
- executive overview
- market price drivers module
- network divergence module
- rooftop PV module
- demand ramp module
- unified recommendation table

---

# 13. Project Output Inventory

## Core Output CSVs

| Output | Notebook | Purpose |
|---|---|---|
| `01_base_market_context.csv` | 01 | Clean merged base market data. |
| `01_base_market_context_summary.csv` | 01 | Summary of base market data. |
| `02_base_market_features.csv` | 02 | Market features including price spikes and volatility. |
| `02_regional_feature_summary.csv` | 02 | Region-level market summary. |
| `02_spike_summary_by_region.csv` | 02 | Spike summary including regions with zero spikes. |
| `02_highest_price_intervals.csv` | 02 | Highest price intervals. |
| `03_constraint_features.csv` | 03 | Interval-level constraint features. |
| `03_constraint_event_summary.csv` | 03 | Highest constraint activity events. |
| `03_spike_constraint_detail.csv` | 03 | Spike events with constraint context. |
| `03_market_features_with_constraints.csv` | 03 | Market features joined with constraints. |
| `04_interconnector_features.csv` | 04 | Interconnector-level flow features. |
| `04_interconnector_flow_summary.csv` | 04 | Interconnector summary. |
| `04_interconnector_interval_features.csv` | 04 | Interval-level flow features. |
| `04_interconnector_vs_spike_summary.csv` | 04 | Spike vs non-spike interconnector comparison. |
| `04_market_features_with_network.csv` | 04 | Market, constraint, and interconnector features. |
| `05_regional_price_divergence.csv` | 05 | NSW1-VIC1 price spread table. |
| `05_divergence_summary.csv` | 05 | Overall divergence metrics. |
| `05_divergence_direction_summary.csv` | 05 | Divergence direction summary. |
| `05_divergence_event_summary.csv` | 05 | Region-level divergence events. |
| `05_divergence_interval_summary.csv` | 05 | Interval-level divergence events. |
| `05_spike_divergence_summary.csv` | 05 | Spike vs divergence summary. |
| `05_market_features_with_divergence.csv` | 05 | Full dataset with divergence features. |
| `06_network_spike_explanation.csv` | 06 | Spike event explanation table. |
| `06_spike_classification_summary.csv` | 06 | Spike classification summary. |
| `06_market_features_with_spike_classification.csv` | 06 | Full dataset with spike classifications. |
| `07_network_decision_recommendations.csv` | 07 | Event-level recommendations. |
| `07_decision_recommendation_summary.csv` | 07 | Recommendation summary. |

---

# 14. Final Analyst Findings

## Finding 1: NSW1 Was The Main Price Spike Region

NSW1 recorded 53 intervals above $300/MWh.

VIC1 did not breach the $300/MWh threshold in this study period.

## Finding 2: Constraint Activity Increased During NSW1 Spikes

Average active constraints increased during NSW1 spike intervals.

However, marginal value and violation evidence did not support a simple conclusion that all spikes were purely constraint-driven.

## Finding 3: Interconnector Stress Was Relevant During Spike Events

Most NSW1 spike intervals had at least one congestion-style interconnector signal.

This suggests that network transfer behaviour was relevant during high-price events.

## Finding 4: VIC1-NSW1 Was A Key Interconnector

The `VIC1-NSW1` interconnector showed:

- largest maximum absolute flow
- largest absolute flow change
- high congestion-style signal share

This makes it important for NSW1-VIC1 divergence analysis.

## Finding 5: NSW1-VIC1 Divergence Was Material But Not Constant

Material divergence occurred in 307 intervals.

Extreme divergence occurred in 60 intervals.

This means price separation was significant but concentrated in specific periods.

## Finding 6: Spike Classification Split Between Constraint And Interconnector Drivers

NSW1 spike classification results:

- 27 constraint-driven intervals
- 26 interconnector/congestion-driven intervals

The most severe price outcomes were in the interconnector/congestion-driven group.

## Finding 7: Decision Intelligence Adds Business Value

The project converts technical signals into:

- market situation
- insight
- recommendation
- risk
- confidence

This makes the project useful for analyst and trader decision support.

---

# 15. Final Executive Summary

This project built a professional NEM network and price divergence intelligence workflow for February 2026, focused on NSW1 and VIC1.

The analysis started with clean dispatch price and regional operating data, then progressively added market features, constraint features, interconnector flow features, regional price divergence, spike classification, and decision recommendations.

The results show that NSW1 was the main high-price region in the study period, recording 53 price spike intervals above $300/MWh. VIC1 did not record spike intervals under the same threshold.

Constraint activity was higher during NSW1 spike intervals, but marginal value and violation evidence did not support a simple conclusion that spikes were purely constraint-driven.

Interconnector behaviour was highly relevant. Broader congestion-style signals appeared during most NSW1 spike intervals, and the most severe price outcomes were classified as interconnector/congestion-driven.

Regional price divergence was also material. NSW1-VIC1 price separation exceeded $100/MWh in 307 intervals and exceeded $300/MWh in 60 intervals.

The final decision intelligence layer translated these findings into recommendation-ready outputs that can support dashboarding, reporting, and market monitoring.

---

# 16. Suggested Slide Deck Structure

## Slide 1: Title

NEM Network Constraints & Price Divergence Intelligence System

## Slide 2: Business Question

How do network constraints and interconnector flows drive price spikes, congestion, and regional price divergence in the NEM?

## Slide 3: Project Architecture

ETL → PostgreSQL → Feature Engineering → Classification → Recommendations → Dashboard

## Slide 4: Data Sources

Dispatch price, dispatch region summary, dispatch constraints, interconnector results.

## Slide 5: Analytical Workflow

Explain notebooks 01 to 08.

## Slide 6: Base Market Features

RRP, demand, supply-demand gap, price spike flag, volatility.

## Slide 7: Constraint Activity

Active constraint count, marginal value, violation flag.

## Slide 8: Interconnector Flow

Flow, absolute flow, flow change, congestion-style signals.

## Slide 9: NSW1-VIC1 Price Divergence

Price spread, divergence flag, extreme divergence flag.

## Slide 10: Spike Classification

Constraint-driven vs interconnector/congestion-driven spikes.

## Slide 11: Key Findings

NSW1 spikes, material divergence, interconnector stress, constraint complexity.

## Slide 12: Decision Intelligence

Market situation, insight, recommendation, risk, confidence.

## Slide 13: Dashboard Outputs

Interactive Plotly charts and dashboard mockups.

## Slide 14: Limitations

Rule-based model, congestion proxy, limited formal constraint equation interpretation.

## Slide 15: Next Steps

Generalise to all NEM regions, add FCAS/BESS modules, build Streamlit dashboard.

---

# 17. Suggested Study Guide Questions

1. What is the difference between a price spike and a price divergence event?
2. Why is constraint marginal value important?
3. Why does high constraint count not automatically prove a constraint-driven price spike?
4. What does interconnector flow change tell an analyst?
5. Why is VIC1-NSW1 important for NSW1-VIC1 price analysis?
6. What is the purpose of the `divergence_flag`?
7. Why is spike classification rule-based rather than causal?
8. How does decision intelligence improve the business value of the project?
9. What are the limitations of using high flow as a congestion-style signal?
10. How could this project be expanded to all NEM regions?

---

# 18. Limitations

The current project has several limitations:

- Focused on NSW1 and VIC1 only.
- Uses practical congestion-style signals rather than formal interconnector limits.
- Constraint RHS and equation details were not deeply interpreted.
- Spike classification is rule-based, not statistical or causal.
- Bidding behaviour and generator unit-level dispatch were not included.
- FCAS and BESS revenue logic are future modules.

These limitations are acceptable for a portfolio project, but they should be clearly stated.

---

# 19. Future Enhancements

Future work should include:

- expand to all NEM regions
- allow dynamic region pair selection
- add all interconnector mapping logic
- use constraint RHS and equation data
- add FCAS price and requirement analysis
- add BESS charge/discharge simulation
- build a Streamlit dashboard
- create a unified trader signal engine
- schedule ETL and analytics refresh
- write analytics outputs back to PostgreSQL

---

# 20. Capstone Positioning

This project can become one module inside a larger capstone:

NEM Market Intelligence & Trader Decision Platform

The platform can include:

- Market Price Drivers
- Network Constraints & Price Divergence
- Rooftop PV & Net Demand
- Ramp & Volatility Risk
- FCAS Market Intelligence
- BESS Dispatch Intelligence
- Trader Decision Console

The final platform should produce a unified trader decision signal:

- market condition
- primary driver
- risk level
- confidence
- recommendation
- supporting evidence

---

# 21. One-Line Portfolio Summary

Built an end-to-end Python and PostgreSQL intelligence system that analyses AEMO dispatch data to explain NEM price spikes, network constraints, interconnector flow behaviour, regional price divergence, and decision recommendations for market analysts.

