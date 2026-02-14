# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

JC Farm App is an R/Quarto analytics application for financial tracking and accountability at JC Poultry Farms (Uganda). It pulls transaction data from KoboToolbox (a remote data collection platform) via API and generates interactive HTML dashboards showing KPIs, P&L statements, and statistical process control (SPC) charts.

## Tech Stack

- **Language**: R
- **Document/Dashboard Framework**: Quarto Dashboards (`.qmd` files rendered to HTML)
- **IDE**: RStudio (project file: `jc-farm-app.Rproj`)
- **Data Source**: KoboToolbox API via the `robotoolbox` R package
- **Currency**: UGX (Ugandan Shillings)

## Build / Render Commands

```bash
# Render all Quarto documents to HTML
quarto render

# Render a specific document
quarto render index.qmd
quarto render trends_jc_farms.qmd

# Preview with live reload (opens browser)
quarto preview index.qmd
```

Rendering can also be done via the RStudio "Render" button.

## Architecture

### Dashboards (Quarto `.qmd` files)
- **`index.qmd`** → `index.html`: Main dashboard. Pulls live data from KoboToolbox API, displays per-batch KPIs (revenue, cost, net profit, cost/bird, profit/bird) as value boxes, and renders P&L statement tables.
- **`trends_jc_farms.qmd`** → `trends_jc_farms.html`: Trends report with SPC charts (control limits at mean ± 2σ/3σ), profit margin trends, cost-per-bird trends, mortality rates, and Feed Conversion Ratio (FCR).
- **`trends.qmd`** → `trends.html`: Alternative trends report.

### Core R Code
- **`R-scripts/00-functions.R`**: Shared helper functions sourced by the dashboards:
  - `jc_dashboard_kpis(data, batch_num, num_birds)` — calculates 5 KPIs for a given batch
  - `jc_pnl_by_batch(data, batch_num)` — generates a detailed P&L statement dataframe
- **`R-scripts/01-data-acquisition.R`** and **`R-scripts/02-nice-shot-charts.R`**: Legacy/unrelated scripts (basketball data), not used by the farm dashboards.

### Data Pipeline
```
KoboToolbox API (kobo_setup → kobo_data)
  → R dataframe (dplyr transformations, column renames, filtering)
  → Dashboard output (value boxes, gt/DT tables, plotly/highcharter charts)
  → Archived CSV snapshots in data/
```

### Key Directories
- **`data/`**: 40+ timestamped CSV/Excel transaction snapshots and business operating model files
- **`sandbox/`**: Experimental R scripts (`pnl_sandbox.R`, `spc_sandbox.R`) for prototyping
- **`assets/`**: Screenshots of KoboToolbox forms and documentation images

## Data Schema

Transaction records have these key fields:
- `Batch_Number` — production batch identifier (batches 4–16 tracked)
- `account_type` — classification: `revenues`, `cogs`, `opex`
- `category_cogs` — COGS subcategory: chicks, feeds, vet
- `category_opex` — OPEX subcategory: transport, utilities, salaries
- `category_revenues` — revenue type
- `Amount_UGX` — transaction amount in Ugandan Shillings
- `transaction_date`, `quantity`, `unit_price`, `description`

## Key R Libraries

Data: tidyverse, lubridate, readxl, robotoolbox, janitor
Visualization: plotly, highcharter, ggplot2, ggthemes
Tables: gt, gtExtras, DT, kableExtra, formattable
Dashboard UI: bslib, bsicons, shiny
Analysis: qcc (SPC charts), scales, glue

## Styling

Custom CSS in `style.css` applies Bootstrap overrides (teal accent color `#69b3a2`, heading spacing, TOC styling). Quarto Dashboards use Bootstrap 5 by default.

## Notes

- KoboToolbox API token is currently hardcoded in `.qmd` files — should be moved to environment variables.
- Batch numbers for KPI calculations are hardcoded in the dashboard; new batches require manual updates to `index.qmd`.
- No formal test suite exists; `sandbox/` scripts serve as manual experimentation space.
- No linting configuration; code style follows RStudio defaults (2-space indentation, UTF-8).
