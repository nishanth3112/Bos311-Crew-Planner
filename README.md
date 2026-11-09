# Boston 311 Crew Planner

A month-by-month 2025 staffing plan for four City of Boston departments, built from real 311 service request data, with a GenAI intake assistant and a planner agent.

IE 5374 Hackathon, Northeastern University. Team of 3.

> **Status:** in progress.

## What's in the repo

- `notebooks/boston311_crew_planner.ipynb`: the whole project, run top to bottom
- `report.md`: a one-page analysis of our decisions, assumptions and sensitivity results
- `data/processed/`: small CSVs the notebook writes

## Departments in scope

| Department | Code | 2024 requests | On time, 2024 | Requests per crew per day | Annual cost per crew | Crews, Jan 2025 |
|---|---|---|---|---|---|---|
| Public Works | PWDx | 157,610 | 90% | 12 | $130,000 | 50 |
| Transportation | BTDT | 81,447 | 50% | 15 | $110,000 | 18 |
| Parks & Recreation | PARK | 18,736 | 93% | 6 | $120,000 | 10 |
| Inspectional Services | ISD | 18,624 | 72% | 4 | $140,000 | 16 |

Request counts and on-time rates come from the real data. Capacities, costs and starting crews are planning assumptions from the hackathon brief.

## Data

Public 311 service requests from [Analyze Boston](https://data.boston.gov/dataset/311-service-requests), the 2023 and 2024 files. The raw CSVs are too large for GitHub and are not committed; the notebook downloads them.

## Setup

1. Install [uv](https://docs.astral.sh/uv/getting-started/installation/).
2. Install the dependencies (Python 3.12 is fetched automatically):
   ```bash
   uv sync
   ```
3. Copy the environment file and add your [Groq](https://console.groq.com/keys) API key to `.env`:
   ```bash
   cp .env.example .env
   ```
4. Start Jupyter and open `notebooks/boston311_crew_planner.ipynb`:
   ```bash
   uv run jupyter lab
   ```

## Team

| Member | Notebook | Repo | Report | Video |
|---|---|---|---|---|
| Nishanth Manoharan ([@nishanth3112](https://github.com/nishanth3112)) | Setup (Section 0), intake assistant, planner agent and Gradio app (Sections 6–8), analysis and limitations (Section 10), final run | uv project, notebook skeleton, setup steps; reviewed the staffing and README PRs | Intake assistant paragraph and the "How we used AI" section | Architecture walkthrough: diagram, intake assistant, agent tools, optimizer |
| Ramgopal Reddy Alumalla ([@ramg-0903](https://github.com/ramg-0903)) | Data download and checks against the brief, four exploratory charts, the 2025 forecast with a 2024 backtest (Sections 1–3), on-time rates by neighborhood and resident versus employee requests (Section 9) | Shared CSVs (`request_types`, `monthly_demand`, `forecast_2025`, `neighborhood_stats`) and the 30-message `intake_test_set.csv`, labeled from `request_types.csv`; reviewed the setup and GenAI app PRs | Compiled `report.md`; wrote the data, forecast and equity paragraphs | The problem |
| Aryan Patil ([@Aryan16d](https://github.com/Aryan16d)) | Staffing plan (MILP) and hold-flat baseline (Section 4), sensitivity scenarios (Section 5) | `plan_2025.csv`, `scenarios.csv`, README with the Mermaid architecture diagram; reviewed the data, report and final-run PRs | Staffing plan and sensitivity paragraphs, with Transportation called out | Product demo: the plan, a sensitivity result and the Gradio app |
