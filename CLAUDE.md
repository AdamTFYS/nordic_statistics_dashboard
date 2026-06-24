# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Context

A Streamlit dashboard comparing statistics across Nordic countries (Denmark, Finland, Iceland, Norway, Sweden) using Eurostat data. Tab-based, configured via Streamlit session state. No authentication required for the Eurostat API.

## Stack

- Python 3.14+, managed with `uv` (astral.sh)
- Streamlit (UI), Plotly (charts), Pandas (data), Seaborn (statistical viz)
- Eurostat API via `src/eurostat_api.py` — base URL: `https://ec.europa.eu/eurostat/api/dissemination/statistics/1.0/data`

## Commands

```bash
# Install dependencies
uv sync

# Run the dashboard
streamlit run app.py

# Smoke-test all imports
python -c "from src.data_cleaning import *; from src.graphs import *; print('All imports OK')"

# Run tests
python -m pytest tests/
python -m pytest tests/test_data_cleaning.py    # single test file
```

## Architecture

### Data Flow

```
eurostat_api.py → data_cleaning.py → graphs.py → app.py
```

- **`src/eurostat_api.py`** — Eurostat API integration. **Read-only, never modify.**
- **`src/data_cleaning.py`** — Pure cleaning functions. Every function returns a DataFrame with `country` (full name), `date` (datetime), and value columns. No Streamlit imports.
- **`src/graphs.py`** — Plotly figure factories. Returns `go.Figure` objects, no Streamlit calls.
- **`app.py`** — Streamlit orchestration only. No data wrangling or chart construction here.
- **`config/design.toml`** — Centralized Plotly styling (colors, fonts).

### Planning Artifacts

These files are produced by agents and serve as immutable truth for downstream agents:

| File | Producer | Purpose |
|------|----------|---------|
| `SPEC.md` | pm-spec-generator | Product specification |
| `tasks.json` | pm-spec-generator | Atomic task breakdown |
| `DATA_CONTRACT.md` | data-engineer | Data schemas and contracts |
| `UI_PLAN.md` | viz-ux-designer | Layout and controls |
| `CHART_SPECS.md` | viz-ux-designer | Chart specifications |

Never instruct an agent to contradict these files. If a change is needed, re-run the originating agent first.

## Orchestrator Role

You are the orchestrator. You do not write code yourself. When given a task:

1. Restate the user's intent in one sentence to confirm understanding
2. Ask clarifying questions if anything is ambiguous — never make assumptions
3. Break the task into ordered steps
4. Spawn subagents sequentially using the Task tool
5. Summarize the result when all subagents have completed

## Agent Pipeline

Always invoke agents in this exact order. Never skip a step. `pm-spec-generator` is always the entry point.

| Step | Agent | Reads | Writes |
|------|-------|-------|--------|
| 1 | **pm-spec-generator** | `CLAUDE.md` | `SPEC.md`, `tasks.json` |
| 2 | **data-engineer** | `SPEC.md`, `tasks.json`, `src/eurostat_api.py` | `DATA_CONTRACT.md`, `src/data_cleaning.py` |
| 3 | **viz-ux-designer** | `SPEC.md`, `DATA_CONTRACT.md`, `src/data_cleaning.py`, `config/design.toml` | `UI_PLAN.md`, `CHART_SPECS.md` |
| 4 | **streamlit-builder** | `SPEC.md`, `DATA_CONTRACT.md`, `UI_PLAN.md`, `CHART_SPECS.md`, `src/data_cleaning.py` | `app.py`, `src/graphs.py`, `config/design.toml` |
| 5 | **verifier** | All output files | PASS/FAIL report |

If verifier returns FAIL: re-run streamlit-builder with the failure report as context, then re-run verifier. Repeat until PASS.

Agent definitions live in `.claude/agents/`. When spawning a subagent, pass the relevant `.md` file as system prompt with a precise task description and any upstream context. Never spawn the next agent before the current one has completed and produced its required output files.

## Datasets

| Code | Title | Frequency |
|------|-------|-----------|
| `demo_pjan` | Population on 1 January | Annual |
| `prc_hicp_manr` | HICP inflation rate | Monthly |
| `une_rt_m` | Unemployment rate | Monthly |
| `namq_10_gdp` | GDP and main components | Quarterly |
| `tps00206` | Crude marriage rate | Annual |
| `tps00216` | Crude divorce rate | Annual |
| `tps00029` | Deaths and crude death rate | Annual |
| `tgs00100` | Total fertility rate (NUTS 2 → country) | Annual |

## Nordic Countries

```python
NORDIC_COUNTRIES = {"DK": "Denmark", "FI": "Finland", "IS": "Iceland", "NO": "Norway", "SE": "Sweden"}
```

## Coding Conventions

- All cleaned DataFrames must have `country` (full name, not ISO code) and `date` (datetime) columns
- Column names: lowercase with underscores, no spaces
- All data-fetching functions use `@st.cache_data` to avoid redundant API calls
- All visual parameters defined in `config/design.toml` — no hardcoded colors or fonts
- Mixed data frequencies: monthly (inflation, unemployment), quarterly (GDP), annual (others)
- NUTS 2 regional data (fertility rate `tgs00100`) must be aggregated to country level

## Key Rules

- Never modify `src/eurostat_api.py`
- Always invoke agents in the defined pipeline order — no exceptions
- Never skip `pm-spec-generator`, even for small changes
