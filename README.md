# Coursera IBM Data Science — SpaceX Capstone

Labs for the IBM Applied Data Science Capstone (SpaceX Falcon 9 first-stage landing).

## Plotly Dash dashboard (TASK 1–4)

Files:

- `spacex_dash_app.py` — completed Dash app
- `spacex_dash_app.ipynb` — same app as a notebook
- `spacex_launch_dash.csv` — launch records used by the dashboard

Run locally:

```bash
source .venv/bin/activate
python spacex_dash_app.py
```

Open http://127.0.0.1:8050

| Task | What it does |
|---|---|
| 1 | Launch-site dropdown (`site-dropdown`, default ALL) |
| 2 | Pie chart of successes by site, or success vs failure for one site |
| 3 | Payload range slider 0–10000 kg |
| 4 | Scatter of payload vs class, colored by booster version |

## Other labs

- Data collection, wrangling, SQL, EDA visualization, Folium launch-site maps
- Data dictionary: `spacex-falcon9-data-dictionary.md`
