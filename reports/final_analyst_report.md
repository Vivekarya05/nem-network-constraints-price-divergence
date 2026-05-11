# Final Analyst Report

# NEM Network Constraints & Price Divergence Intelligence System

## Executive Summary

This project analysed how network constraints and interconnector flows influenced regional price behaviour in the Australian National Electricity Market for the February 2026 study period.

The analysis focused on NSW1 and VIC1 and examined dispatch price, regional demand and generation conditions, dispatch constraint activity, interconnector flows, NSW1-VIC1 price divergence, price spike classification, and decision intelligence recommendations.

The key finding is that NSW1 recorded material price spike events while VIC1 did not breach the selected $300/MWh spike threshold. NSW1 spike intervals were associated with elevated constraint activity and broader interconnector congestion-style signals. However, constraint marginal value and violation evidence did not support a simple conclusion that all spikes were purely constraint-driven.

The most severe price outcomes were classified as interconnector/congestion-driven, with regional price divergence and broader interconnector stress providing important explanatory evidence.

## Business Question

How do network constraints and interconnector flows drive price spikes, congestion, and regional price divergence in the NEM?

## Study Scope

| Item | Scope |
|---|---|
| Market | Australian National Electricity Market |
| Study period | 2026-02-01 to 2026-03-01 |
| Focus regions | NSW1 and VIC1 |
| Primary network focus | Dispatch constraints and interconnector flows |
| Main price threshold | RRP greater than $300/MWh |
| Divergence threshold | NSW1-VIC1 absolute price spread greater than $100/MWh |
| Extreme divergence threshold | NSW1-VIC1 absolute price spread greater than $300/MWh |

## Data Sources

The analysis used AEMO NEMWeb data loaded into PostgreSQL through an existing ETL pipeline.

Core tables used:

- `raw.dispatch_price`
- `raw.dispatch_regionsum`
- `raw.dispatch_constraints`
- `raw.interconnector_results`

Optional tables available for future enhancement:

- `raw.constraint_rhs`
- `raw.constraint_details`
- `raw.constraint_equations`

NEMOSIS was not used.

## Analytical Methodology

The project was developed through eight notebooks:

| Notebook | Purpose |
|---|---|
| 01 Data Extraction And Market Context | Extracted and merged price, demand, generation, and interchange data. |
| 02 Base Market Feature Engineering | Created price spikes, supply-demand gap, price change, volatility, and time features. |
| 03 Constraint Activity Analysis | Created interval-level constraint activity features. |
| 04 Interconnector Flow Analysis | Created interconnector flow, flow change, and congestion-style signal features. |
| 05 Regional Price Divergence | Calculated NSW1-VIC1 price spread and divergence flags. |
| 06 Network Spike Classification | Classified price spikes by likely market driver. |
| 07 Decision Intelligence | Converted spike classifications into recommendations, risks, and confidence levels. |
| 08 Plotly Visualisation Pack | Created interactive visual outputs for reporting and dashboard design. |

## Base Market Findings

The base market dataset combined regional RRP, total demand, available generation, and net interchange at 5-minute dispatch interval resolution.

Using a $300/MWh spike threshold:

- NSW1 recorded 53 spike intervals.
- VIC1 recorded no spike intervals under the selected threshold.

This indicates that NSW1 was the main high-price event region during the study period, while VIC1 remained important as the comparison region for price divergence and interconnector analysis.

## Constraint-Driven Behaviour

Constraint activity was assessed using:

- active constraint count
- maximum constraint marginal value
- total constraint marginal value
- violation flag
- maximum violation degree

During NSW1 non-spike intervals:

- average RRP was approximately $68/MWh
- average active constraint count was approximately 19.3

During NSW1 spike intervals:

- average RRP was approximately $2,417/MWh
- average active constraint count increased to approximately 24.4
- no violation intervals were observed during NSW1 spike periods

This indicates that NSW1 spike intervals occurred during periods with higher average constraint activity.

However, maximum constraint marginal value was not concentrated in spike intervals. The largest marginal value events occurred during non-spike intervals. This means the evidence does not support a simple conclusion that NSW1 price spikes were purely constraint-driven.

## Interconnector Insights

The interconnector analysis created the following features:

- flow
- absolute flow
- flow change
- absolute flow change
- high flow flag
- high flow change flag
- congestion-style signal flag

The `VIC1-NSW1` interconnector was particularly important because it directly connects the focus regions.

For `VIC1-NSW1`:

- maximum absolute flow was approximately 1,873 MW
- maximum absolute flow change was approximately 1,447 MW
- congestion-style signal share was approximately 19.8%

During NSW1 spike intervals:

- average maximum absolute interconnector flow increased
- average maximum absolute flow change increased
- average congestion signal count increased
- 49 of 53 NSW1 spike intervals had at least one broader congestion-style signal

This suggests that broader interconnector stress was relevant during most NSW1 spike intervals.

However, direct `VIC1-NSW1` absolute flow was lower during NSW1 spike intervals than during non-spike intervals. This means the spike explanation is broader than a single direct interconnector flow signal.

## Regional Price Divergence

NSW1-VIC1 price divergence was measured using:

- NSW1 RRP
- VIC1 RRP
- signed price spread
- absolute price spread
- divergence flag above $100/MWh
- extreme divergence flag above $300/MWh

Key divergence results:

| Metric | Value |
|---|---:|
| Intervals analysed | 8063 |
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

The divergence analysis shows that NSW1 prices were higher than VIC1 on average, and that material price separation occurred in concentrated intervals.

This supports the conclusion that regional separation was an important part of the market story, particularly during high-price periods.

## Spike Classification

Price spikes were classified using a rule-based framework.

Classification categories:

- Constraint-driven
- Interconnector/congestion-driven
- Demand-driven
- Volatility-driven
- Unknown

Classification logic used:

- demand tightness signal
- volatility signal
- constraint signal
- interconnector congestion signal
- divergence signal

Key classification result:

| Region | Classification | Spike Intervals | Average RRP | Max RRP |
|---|---|---:|---:|---:|
| NSW1 | Constraint-driven | 27 | 1522.01 | 9914.02 |
| NSW1 | Interconnector/congestion-driven | 26 | 3346.61 | 20300.00 |

NSW1 spike intervals were split almost evenly between constraint-driven and interconnector/congestion-driven classifications.

The interconnector/congestion-driven group had the higher average RRP and included the maximum price event of $20,300/MWh.

This suggests that the most severe price outcomes were more strongly associated with interconnector stress and regional price divergence than with constraint signals alone.

## Decision Intelligence Recommendations

The decision intelligence layer translated spike classifications into:

- market situation
- insight
- recommendation
- risk
- confidence

For constraint-driven spike intervals, the recommendation is to review binding or high-impact constraints and monitor whether similar constraints recur during peak demand or outage conditions.

For interconnector/congestion-driven intervals, the recommendation is to monitor interconnector flow, flow changes, and regional price spreads, with particular focus on NSW1-VIC1 separation and neighbouring interconnector behaviour.

The decision recommendations are not trading instructions. They are decision-support outputs that help analysts prioritise which intervals, constraints, interconnectors, and price divergence events require deeper review.

## Visual Outputs

The project created the following interactive Plotly charts:

| Chart | Purpose |
|---|---|
| RRP by region with spike markers | Shows price outcomes and spike timing. |
| NSW1 vs VIC1 RRP comparison | Shows regional price alignment and separation. |
| NSW1-VIC1 price spread | Shows material and extreme divergence. |
| Constraint activity vs NSW1 RRP | Compares constraints with price outcomes. |
| VIC1-NSW1 interconnector flow | Shows direct NSW-VIC transfer behaviour. |
| Congestion signal count | Shows broader interconnector stress. |
| Spike classification summary | Shows likely spike drivers. |
| Demand and price overview | Compares demand and price between NSW1 and VIC1. |

These charts support both analyst review and future dashboard design.

## Key Findings

1. NSW1 was the main high-price event region, with 53 intervals above $300/MWh.

2. VIC1 did not breach the selected price spike threshold during the study period.

3. NSW1 price spikes were associated with higher average active constraint counts.

4. Constraint marginal value and violation evidence did not support a simple conclusion that all NSW1 spikes were purely constraint-driven.

5. Broader interconnector congestion-style signals were present during most NSW1 spike intervals.

6. NSW1-VIC1 price divergence was material in concentrated intervals, with 307 divergence intervals and 60 extreme divergence intervals.

7. Spike classification split NSW1 spike intervals between constraint-driven and interconnector/congestion-driven categories.

8. The most severe price outcomes were in the interconnector/congestion-driven classification group.

## Risks

Key market risks identified:

- regional price separation risk
- constraint recurrence risk
- interconnector flow stress risk
- high-price volatility risk
- classification uncertainty where multiple drivers overlap

## Limitations

This project has several limitations:

- It focuses on NSW1 and VIC1 only.
- Congestion-style signals are analytical indicators, not formal AEMO congestion classifications.
- Constraint RHS and equation data were not deeply interpreted.
- The spike classification is rule-based, not causal.
- Bidding behaviour and unit-level dispatch were not included.
- FCAS and BESS revenue logic are future enhancements.

These limitations should be clearly stated in portfolio and interview discussion.

## Next Steps

Recommended future enhancements:

1. Extend the analysis to all NEM regions.
2. Allow dynamic region pair selection.
3. Add formal interconnector limit and constraint equation analysis.
4. Use constraint RHS and equation data to explain binding constraints.
5. Add FCAS price and requirement analysis.
6. Add BESS charge/discharge and revenue simulation.
7. Convert notebook logic into reusable Python model code.
8. Build a Streamlit dashboard.
9. Create a unified trader decision signal layer.
10. Schedule ETL and analytics refresh.

## Final Conclusion

This project demonstrates a complete energy market analyst workflow: extracting dispatch data, engineering market and network features, identifying price spikes and regional divergence, classifying likely drivers, and translating results into decision intelligence.

The analysis shows that NSW1 high-price events during February 2026 were not explained by demand or constraints alone. Network conditions, interconnector stress, and regional price separation were important parts of the market story.

The project is suitable for a professional Energy Market Analyst or Energy Modeller portfolio because it combines technical Python analysis with market reasoning, business interpretation, and decision-oriented outputs.
