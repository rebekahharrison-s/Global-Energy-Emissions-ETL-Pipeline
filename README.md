## Global Energy & Emissions ETL Pipeline
Source: Our World in Data — owid/energy-data (via TidyTuesday / GitHub)
Domain: Utilities / Energy / Climate Analytics

Overview
This notebook implements a production-grade ETL pipeline and interactive analytics dashboard over the Our World in Data global energy dataset — covering 200+ countries and 120+ years of annual records (focused on 15 key economies, 1990–2022).
It is explicitly designed to demonstrate domain-agnostic data engineering: the same medallion architecture, Pydantic validation, quality gates, and idempotent warehouse layer used in a companion e-commerce pipeline are applied unchanged to a completely different domain and data grain.

Architecture
The pipeline follows a classic Bronze → Silver → Gold (Staging → Curated) medallion pattern:
StageWhat happensExtractFetches CSV from GitHub (TidyTuesday), filters to 15 countries and 1990–2022ValidateEach row is parsed through a Pydantic EnergyRecord model — invalid rows are rejected and countedStagingValidated records normalised into a flat pandas DataFrameCuratedBusiness enrichments added (see below)LoadLoaded into an in-memory SQLite warehouse for SQL-based analytics

Business Enrichments (Curated Layer)

Energy transition score — renewable share minus fossil share (higher = greener)
Transition tier — categorical label: green_leader, transitioning, mixed, fossil_dependent
GDP per capita — derived from raw GDP and population
Carbon intensity tier — bucketed from gCO2/kWh into very_low → very_high
Decade bucket — for trend grouping across time


Dashboards
Static Dashboard (Matplotlib) — 6-panel dark-theme figure:

Renewable electricity share over time for 6 countries (with Paris Agreement 2015 marker)
Carbon intensity by country (2022, horizontal bar)
Electricity generation mix stacked bar (2022, Fossil / Renewables / Nuclear)
GDP per capita vs renewables share scatter (2022, colour-coded by transition tier)
UK wind & solar growth area chart (1990–2022)
Carbon intensity trend lines — UK, Germany, China

Interactive Dashboard (Plotly) — 3 fully interactive charts:

Animated scatter — GDP per capita vs renewable share, playing through 1990–2022, bubble size = electricity generation
Multi-metric line chart — dropdown selector switches between 5 metrics (renewables share, carbon intensity, fossil share, generation, energy per capita), with a range slider
Generation mix stacked bar — year scrubber slides from 1990 to 2022 across all 15 countries


Tech Stack
ComponentToolSchema validationPydantic v2 (field_validator, model_validate)Data manipulationpandas, numpyWarehouseSQLite (in-memory, SQL queries via pd.read_sql_query)Static vizmatplotlib, GridSpecInteractive vizPlotly Express + Plotly Graph Objects

Key Design Principles

Domain-agnostic infrastructure — pipeline architecture is identical to the companion e-commerce notebook; only the schema and business logic differ
Pydantic-first validation — all type coercion and null handling happens at the boundary, not scattered through transform code
SQL-queryable warehouse — curated data loaded into SQLite so all analytical queries are plain SQL, not pandas chains
Reproducible — single-cell execution, no local files required (data fetched from a stable public URL)
