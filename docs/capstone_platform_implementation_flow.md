# Capstone Platform Implementation Flow

# NEM Market Intelligence & Trader Decision Platform

This document stores the agreed implementation direction for later build stages.

The current work is focused on learning each business case through notebooks. Later, the notebook logic will be converted into reusable model code and combined into a unified dashboard.

## Core Flow

```text
1. Learn each business case in notebooks
        ↓
2. Convert final notebook logic into reusable model code
        ↓
3. Create one pipeline per business case
        ↓
4. Save standard output tables
        ↓
5. Combine all output tables into trader signal layer
        ↓
6. Build dashboard pages for each case
        ↓
7. Build final trader decision dashboard
```

## Full Architecture

```text
AEMO NEMWeb Data
        ↓
Existing ETL Pipeline
        ↓
PostgreSQL raw schema
        ↓
Business Case Models
        ↓
Business Case Pipelines
        ↓
Standard Signal Tables
        ↓
Trader Signal Engine
        ↓
Streamlit Dashboard
```

## Recommended Folder Structure

```text
notebooks/
    project_01_price_drivers/
    project_02_network_divergence/
    project_03_fcas_intelligence/
    project_04_bess_intelligence/

src/
    core/
        db_connection.py
        config.py
        validation.py

    models/
        price_drivers.py
        network_divergence.py
        rooftop_pv.py
        ramp_volatility.py
        fcas_intelligence.py
        bess_dispatch.py
        trader_signal_engine.py

    pipelines/
        run_price_drivers.py
        run_network_divergence.py
        run_fcas_intelligence.py
        run_bess_dispatch.py
        run_all.py

    dashboard/
        data_loader.py
        charts.py
        filters.py
        metrics.py

app.py
outputs/
README.md
requirements.txt
```

## Notebook Flow

For every business case:

```text
Notebook 01: extract data
Notebook 02: clean data
Notebook 03: create features
Notebook 04: classify / model events
Notebook 05: generate recommendations
Notebook 06: create visuals
```

Once the notebook logic is stable:

```text
Move reusable logic into src/models/
```

## Model Flow

Each model file should represent one business case or analytical module.

```text
price_drivers.py
    → price spikes, demand, rooftop PV, volatility

network_divergence.py
    → constraints, interconnector flow, regional spread

fcas_intelligence.py
    → FCAS prices, scarcity, co-optimisation signals

bess_dispatch.py
    → charge/discharge/hold signals, revenue estimate

trader_signal_engine.py
    → combines all business-case signals into final trader view
```

## Pipeline Flow

Each pipeline runs one full business case.

```text
run_price_drivers.py
    reads PostgreSQL
    builds price driver features
    saves outputs

run_network_divergence.py
    reads PostgreSQL
    builds network features
    saves outputs

run_all.py
    runs all business cases
    combines recommendations
    creates trader signal table
```

## Dashboard Flow

```text
app.py
    loads final output tables
    shows sidebar filters
    routes user to selected dashboard page
```

Recommended dashboard pages:

```text
Executive Overview
Price Drivers
Network & Divergence
FCAS Intelligence
BESS Intelligence
Trader Decision Console
Recommendations
```

## Standard Signal Table

Each business case should eventually produce a standard signal table.

Recommended columns:

```text
module_name
settlementdate
regionid
event_type
signal
risk_level
confidence
market_situation
insight
recommendation
watch_items
```

Example rows:

```text
Network & Divergence | NSW1 | Price spike | Network Risk | High | High
Price Drivers | VIC1 | Low price | Rooftop PV Risk | Medium | Medium
FCAS Intelligence | SA1 | FCAS spike | FCAS Opportunity | High | High
BESS Intelligence | VIC1 | Negative price | Charge Opportunity | Medium | High
```

## Final Trader Signal Layer

All modules produce business-case signals:

```text
price_driver_signals
network_signals
fcas_signals
bess_signals
```

Then:

```text
trader_signal_engine.py
```

combines them into:

```text
final_trader_signals.csv
```

The dashboard shows:

```text
Trader Signal
Primary Driver
Secondary Driver
Risk Level
Confidence
Recommendation
Supporting Evidence
```

## Practical Build Order

Current practical path:

```text
Step 1: Finish Project 2 notebooks
Step 2: Create src/models/network_divergence.py
Step 3: Move Project 2 logic into functions
Step 4: Create run_network_divergence.py
Step 5: Build Streamlit page for Project 2
Step 6: Repeat for Project 1
Step 7: Combine both into final trader dashboard
```

## Key Design Principle

```text
Notebook = understand
Model code = reuse
Pipeline = run automatically
Dashboard = show insights
```

This structure keeps the learning work separate from the production-style capstone code.
