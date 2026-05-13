<!-- Audience: Humans. Single source of truth for project overview, setup, and contribution. -->

# Nordic Statistics Comparison Dashboard

A Streamlit dashboard for exploring and comparing key socioeconomic and demographic indicators across the five Nordic countries (Denmark, Finland, Iceland, Norway, Sweden) using data from Eurostat.

## Features

- **Overview**: Population metrics for each Nordic country
- **Inflation & Unemployment**: Monthly trends for HICP inflation and unemployment rates
- **Economic Growth**: Quarterly GDP trends in chain-linked volumes
- **Marriage & Divorce**: Annual crude marriage and divorce rates
- **Deaths & Fertility**: Annual crude death rates and total fertility rates

All charts are interactive with hover tooltips and support multi-country comparison through the sidebar filter.

## Installation

### Using uv (recommended)

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
uv sync
```

### Using pip

```bash
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

## Environment Variables

No environment variables are required. The [Eurostat API](https://ec.europa.eu/eurostat) is public and does not require authentication.

## Running the Dashboard

```bash
streamlit run app.py
```

The dashboard will open in your default web browser at `http://localhost:8501`.

## Project Structure

```
.
├── app.py                   # Main Streamlit application
├── config/
│   └── design.toml         # Styling configuration (colors, fonts, layout)
├── src/
│   ├── __init__.py
│   ├── eurostat_api.py     # Eurostat API client
│   ├── data_cleaning.py    # Data cleaning and transformation
│   └── graphs.py           # Plotly chart factories
├── tests/                  # Test suite
├── .claude/agents/         # Agent definitions for Claude Code
├── SPEC.md                 # Product specification
├── DATA_CONTRACT.md        # Data schemas and contracts
├── UI_PLAN.md              # Dashboard layout specification
├── CHART_SPECS.md          # Chart specifications
├── tasks.json              # Task breakdown
└── pyproject.toml          # Project dependencies
```

## Data Sources

All data is fetched in real-time from the [Eurostat API](https://ec.europa.eu/eurostat). See the Datasets table in `CLAUDE.md` for the full list of dataset codes and their frequencies.

## Notes

- First load may take a few seconds as data is fetched from the Eurostat API
- Data is cached for the session to improve performance
- To refresh data, reload the page in your browser
- The dashboard is optimized for desktop viewing

## Contributing

To extend the dashboard with new tabs or datasets:

1. Add a cleaning function in `src/data_cleaning.py`
2. Add a chart factory in `src/graphs.py`
3. Wire the new tab in `app.py`
4. Update `config/design.toml` with any new styling

Agent-driven development follows the pipeline defined in `CLAUDE.md`. See `.claude/agents/` for agent definitions.

## License

This project is licensed under the MIT License.
