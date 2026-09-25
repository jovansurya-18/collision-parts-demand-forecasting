# Collision Parts Demand Forecasting

A machine learning and time series forecasting project focused on predicting monthly collision-part demand at SKU level using demand segmentation, multiple forecasting models, rolling backtesting, and automated model selection.

## Overview

The objective of this project was to develop a repeatable forecasting workflow for collision spare-parts demand.

Collision-part demand is challenging to forecast because it is sparse, volatile, and event-driven. Many SKUs experience long periods of zero demand followed by occasional demand spikes.

The project therefore uses a multi-model approach rather than relying on a single forecasting technique.

The workflow:

- Segments SKUs by demand behaviour
- Separates commercially important high-volume SKUs
- Tests multiple forecasting approaches
- Evaluates models using rolling-origin backtesting
- Selects models based on WMAPE
- Generates 18-month forecasts
- Retains diagnostics and outputs for reproducibility

## Dataset

The source dataset contained **952,504 monthly SKU records**.

After filtering for collision parts, the analysis identified **8,097 collision SKUs**.

A complete monthly panel was created, with unobserved months explicitly assigned zero demand. This was important because zero demand represents genuine intermittent demand rather than missing data.

The available history ranged from **1 to 64 months**, with a median history of approximately 58 months.

## Demand Segmentation

SKUs were classified using:

- Average Demand Interval (ADI)
- Squared Coefficient of Variation (CV²)

This resulted in five demand categories:

| Demand Type | SKUs | % of SKUs | % of Demand |
|---|---:|---:|---:|
| Intermittent | 6,593 | 81.4% | 22.2% |
| Smooth | 972 | 12.0% | 65.8% |
| Lumpy | 381 | 4.7% | 5.2% |
| Erratic | 124 | 1.5% | 6.8% |
| No demand observed | 27 | 0.3% | 0% |

Demand was highly concentrated:

- The top **5% of SKUs generated approximately 80% of total demand**
- The top **20% generated approximately 95% of total demand**

## Methodology

The project followed a structured forecasting workflow:

- Data validation
- Monthly panel construction
- Demand classification
- Feature engineering
- Model development
- Rolling-origin backtesting
- WMAPE evaluation
- Demand-type comparison
- Volume-tier comparison
- Model selection
- 18-month forecasting
- Forecast diagnostics
- Performance monitoring

A three-month information lag was used to ensure that models only used information available at the forecast origin.

## Forecasting Models

A portfolio of statistical, machine learning, neural network, foundation, and ensemble models was evaluated.

### Baseline Models

- Naive
- Three-month moving average

### Statistical Models

- ARIMA / SARIMA
- Theta
- Simple Exponential Smoothing
- Croston / SBA

### Machine Learning Models

- XGBoost
- LightGBM

### Neural Network Models

- LSTM
- Residual LSTM

### Foundation Models

- Amazon Chronos
- Chronos-2 Small
- Salesforce Moirai

### Ensemble Models

- XGBoost + LightGBM
- XGBoost + Croston residual ensemble
- XGBoost + LSTM
- XGBoost + Residual LSTM
- LSTM + XGBoost residual ensemble

## Feature Engineering

Feature engineering focused on historical demand, recency, sparsity, and seasonality.

### Demand Lags

- `lag_1`
- `lag_2`
- `lag_3`
- `lag_6`
- `lag_9`
- `lag_12`
- `lag_15`

### Rolling Statistics

- Rolling mean
- Rolling standard deviation
- Rolling maximum

Rolling windows included 3, 6, and 12 months.

### Sparsity and Recency

- Zero-demand share over 6 and 12 months
- Months since last demand
- Known demand history

### Calendar Features

- Month of year
- Quarter
- Year

## Backtesting

Models were evaluated using standardised rolling-origin backtesting.

Key safeguards included:

- Identical forecast origins
- 12-month backtest horizon
- Three-month information lag
- Consistent aggregation logic
- No future observations used as features
- Inner training slices for ensemble weight optimisation
- Repeated origins to assess temporal stability

## Evaluation Metric

**WMAPE (Weighted Mean Absolute Percentage Error)** was used as the primary evaluation metric because it weights forecast errors by actual demand.

Performance was evaluated across:

- All eligible SKUs
- Top 20% by demand
- Top 5% by demand
- Demand type
- Forecast horizon
- Backtest origin
- SKU-level performance

A **70% WMAPE benchmark** was also used to assess forecast coverage.

## Results

Across all eligible SKUs, **Chronos-2 Small** achieved approximately **61% WMAPE**.

For the top 20% of SKUs by demand, **Chronos-2 Small** achieved approximately **56% WMAPE**.

For the top 5% of SKUs by demand, **Croston** achieved approximately **48% WMAPE**.

| Evaluation Tier | Model | WMAPE |
|---|---|---:|
| All eligible SKUs | Chronos-2 Small | ≈61% |
| Top 20% by demand | Chronos-2 Small | ≈56% |
| Top 5% by demand | Croston | ≈48% |

## Demand-Type Findings

Model performance varied considerably across demand segments.

- **Smooth demand** generally produced the lowest WMAPE.
- **Intermittent demand** remained more difficult to forecast.
- **Lumpy demand** also showed relatively high forecast error.
- **Erratic demand** presented additional forecasting challenges.
- Foundation models performed strongly across several segments.
- Specialist intermittent-demand methods such as Croston remained valuable for sparse demand.

This supports using a segment-aware model selection strategy rather than deploying one universal forecasting model.

## Commercial Volume Findings

Forecast accuracy improved when evaluation focused on commercially important SKUs.

```text
All eligible SKUs  →  ≈61%
Top 20%            →  ≈56%
Top 5%             →  ≈48%
```

The top 5% of SKUs generated approximately 80% of total demand, making these parts particularly important for inventory planning.

## Forecast Horizon

Forecast performance was also analysed across the 12-month backtest horizon.

Smooth and erratic segments experienced approximately 10 percentage points of WMAPE degradation over the forecast horizon.

Intermittent and lumpy segments were comparatively stable.

## Business Implications

The forecasting workflow can support:

- Inventory planning
- Spare-parts availability
- Reordering decisions
- Stockout reduction
- Overstock reduction
- Replenishment planning
- Prioritisation of high-volume SKUs

For intermittent, lumpy, and short-history SKUs, forecasts should be combined with appropriate inventory safeguards rather than being treated as precise point estimates.

## Recommended Forecasting Strategy

The analysis supports a configurable **champion-by-segment** approach.

Rather than permanently selecting one model, the workflow can:

1. Rerun the standardised backtest during each retraining cycle.
2. Select the strongest validated model within each demand type and volume tier.
3. Prioritise the top 5% and top 20% of SKUs for planner review.
4. Apply additional safeguards to intermittent, lumpy, and short-history SKUs.
5. Combine forecasts with lead times, service-level targets, inventory costs, and business knowledge.
6. Trigger model reselection when monitored forecasting accuracy deteriorates.

## Limitations

Key limitations include:

- Available history was limited to approximately 64 months.
- Short-history SKUs were less likely to qualify for backtesting.
- Collision demand can be affected by external events not represented in historical sales.
- WMAPE does not directly optimise stockout costs, holding costs, or service levels.
- Segment-level winners are not necessarily optimal for every individual SKU.
- External variables require further validation before being incorporated into the forecasting pipeline.

Future development could incorporate:

- Lead times
- Inventory costs
- Service-level targets
- External demand drivers
- Additional vehicle and market information
- More sophisticated uncertainty estimation

## Key Findings

- Collision-part demand was highly sparse, with **81.4% of SKUs classified as intermittent**.
- Smooth SKUs represented only **12.0% of SKUs but 65.8% of total demand**.
- The top 5% of SKUs generated approximately **80% of demand**.
- Forecast performance varied substantially across demand types.
- Chronos-2 Small achieved approximately **61% WMAPE** across all eligible SKUs.
- Chronos-2 Small achieved approximately **56% WMAPE** for the top 20% of SKUs by demand.
- Croston achieved approximately **48% WMAPE** for the top 5% of SKUs.
- A single universal forecasting model was not consistently optimal across all demand segments and volume tiers.
- A monitored multi-model approach provides a more flexible forecasting framework.
- Forecast outputs should be combined with inventory policies and business knowledge before operational deployment.

## Technologies & Techniques

### Programming

- Python
- Pandas
- NumPy

### Forecasting & Statistics

- ARIMA
- SARIMA
- Theta
- Exponential Smoothing
- Croston / SBA

### Machine Learning

- XGBoost
- LightGBM
- Scikit-learn

### Deep Learning

- LSTM
- Residual LSTM

### Foundation Models

- Amazon Chronos
- Chronos-2
- Salesforce Moirai

### Evaluation

- WMAPE
- Rolling-origin backtesting
- Forecast horizon analysis
- Demand segmentation
- Volume-tier analysis


## Project Files

**Notebook:** Contains the forecasting analysis, data preparation, demand segmentation, feature engineering, model development, backtesting, model comparison, and forecast generation.

**Report:** Provides the full project analysis, methodology, model portfolio, automated pipeline design, results, limitations, business recommendations, and supporting analysis.

## Project Structure

```text
collision-parts-demand-forecasting/
│
├── README.md
├── collision_parts_demand_forecasting.ipynb
└── collision_parts_demand_forecasting_report.pdf
```

## Author

**Jovan Surya**

Data Science, Machine Learning & AI

> **Collaboration Project:** This project was developed collaboratively with a group of peers, combining our individual strengths across data preparation, forecasting, machine learning, model evaluation, and analysis.
