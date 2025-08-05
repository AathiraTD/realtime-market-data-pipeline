# realtime-market-data-pipeline

*A 45-minute refresh cycle for live FTSE equities and FX rates*

---

## 1 · Why this project exists

Most trading teams still wake up to yesterday’s prices. This repo turns **24-hour lag into sub-hour insight**, so risk, P\&L, and exposure dashboards stay current without blowing up cloud spend.

---

## 2 · What it delivers

| Capability             | In plain English                                                             |
| ---------------------- | ---------------------------------------------------------------------------- |
| **Live data feed**     | Pulls UK stock (FTSE) and major currency prices every hour.                  |
| **Fast transform**     | Cleans and reshapes raw ticks in Snowflake in minutes, not hours.            |
| **Instant visibility** | Updates Superset dashboards automatically; no manual refresh needed.         |
| **Heads-up alerts**    | Sends a Slack ping if a job is late or data looks off.                       |
| **Zero waste**         | Auto-scales Snowflake compute only when work is running, so no idle charges. |

---

## 3 · How it works (high-level)

```
Market APIs  ─▶  Airflow (hourly DAGs)  ─▶  Snowflake (raw ➜ models)  ─▶  Superset dashboards
                                    │
                                    └──▶  Slack alerts (failures, anomalies)
```

*18 k rows ingested per run, \~100 k processed daily.*

---

## 4 · Quick start

> **Prerequisites** – Docker & Docker Compose, a Snowflake account, and API keys for your data provider.

```bash
# 1. Clone the repo
git clone https://github.com/your-org/realtime-market-data-pipeline.git
cd realtime-market-data-pipeline

# 2. Set secrets (edit .env with your creds)
cp .env.example .env

# 3. Fire it up
docker compose up -d airflow superset

# 4. Run the bootstrap DAG to create tables & models
docker compose exec airflow airflow dags trigger bootstrap_init
```

* Airflow UI → `http://localhost:8080` (`airflow / airflow`)
* Superset UI → `http://localhost:8088` (`admin / admin`)

---

## 5 · Repository layout

```
.
├─ dags/              # Airflow DAG definitions
├─ dbt/               # Data models and tests
├─ superset/          # Dashboard JSON exports
├─ docker-compose.yml # One-command local stack
└─ docs/              # Architecture diagrams & ADRs
```

---

## 6 · Configuration

| Variable                                | Purpose                              |
| --------------------------------------- | ------------------------------------ |
| `SNOWFLAKE_ACCOUNT`                     | Your Snowflake account identifier    |
| `SNOWFLAKE_USER` / `SNOWFLAKE_PASSWORD` | Warehouse login                      |
| `FTSE_API_KEY` / `FX_API_KEY`           | Market-data provider creds           |
| `SLACK_WEBHOOK_URL`                     | Channel for success / failure alerts |

Edit `.env` (never commit real secrets).

---

## 7 · Daily schedule

| Time (UTC)               | Task                                               |
| ------------------------ | -------------------------------------------------- |
| **00, 15, 30, 45 min**   | `ingest_ftse`, `ingest_fx` pull fresh prices       |
| **15 min past the hour** | `dbt_run` transforms & tests data                  |
| **On-event**             | `notify_slack` fires if any SLA or data test fails |

---

## 8 · Extending the pipeline

| Need                              | Where to start                                                 |
| --------------------------------- | -------------------------------------------------------------- |
| Add another market (e.g., NASDAQ) | Duplicate a DAG in `dags/` and adjust API client               |
| More frequent loads               | Change the `schedule_interval` cron in the DAG                 |
| Deploy to AWS                     | Use the `deploy/` compose file → ECS Fargate or EKS            |
| Cost breakdown                    | Enable `WAREHOUSE_USAGE_HISTORY` view and add a Superset chart |

---

## 9 · Troubleshooting

| Symptom                        | Likely cause             | Fix                                                     |
| ------------------------------ | ------------------------ | ------------------------------------------------------- |
| Airflow task stuck in *queued* | Docker memory low        | Allocate ≥4 GB RAM                                      |
| Dashboards stale               | `dbt_run` failed QC test | Check Slack alert, rerun DAG                            |
| Snowflake bill spikes          | Warehouse left running   | Confirm `auto-suspend=60s` in `snowflake/warehouse.sql` |

---

## 10 · Contributing

1. Fork & create a feature branch.
2. Run `pre-commit install` (black, flake8, sqlfluff).
3. Open a PR with a concise description—screenshots welcome.

---

## 11 · License

MIT. See `LICENSE`.

---

## 12 · Maintainers

| Name      | Role     | Contact                 |
| --------- | -------- | ----------------------- |
| Your Name | Lead Dev | `@your-handle` on Slack |
| …         | …        | …                       |

*Got questions?* Open an issue—no PR is too small.
