# Realtime Market Data Pipeline ⚡️📈

[![Build & Test](https://img.shields.io/github/actions/workflow/status/your-org/realtime-market-data-pipeline/ci.yml?branch=main\&style=flat-square)](../../actions)
[![Docker Pulls](https://img.shields.io/docker/pulls/your-org/realtime-market-data-pipeline?style=flat-square)](https://hub.docker.com/r/your-org/realtime-market-data-pipeline)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](CONTRIBUTING.md)

> **Turn *yesterday’s* prices into *sub‑hour* insights — without torching your Snowflake bill.**

---

## 📑 Table of Contents

1. [Why it exists](#-why-it-exists)
2. [What it delivers](#-what-it-delivers)
3. [How it works](#-how-it-works)
4. [Quick start](#-quick-start)
5. [Repository layout](#-repository-layout)
6. [Configuration](#-configuration)
7. [Daily schedule](#-daily-schedule)
8. [Extending the pipeline](#-extending-the-pipeline)
9. [Troubleshooting](#-troubleshooting)
10. [Contributing](#-contributing)
11. [License](#-license)
12. [Maintainers](#-maintainers)

---

## 🧐 Why it exists

Most trading teams still wake up to **yesterday’s** prices. This repo shrinks the *24‑hour lag* to **<45 minutes**, so your risk, P\&L, and exposure dashboards stay current *and* cost‑efficient.

---

## 🚀 What it delivers

|  Capability            |  Plain English                                             |  Tech Highlights              |
| ---------------------- | ---------------------------------------------------------- | ----------------------------- |
| **Live data feed**     | Pulls UK stock (FTSE) & major FX rates every *15 minutes*. | Airflow + Python operators    |
| **Fast transform**     | Cleans & reshapes raw ticks in minutes, not hours.         | dbt models on Snowflake       |
| **Instant visibility** | Dashboards auto‑refresh — no manual clicks.                | Superset auto‑update          |
| **Heads‑up alerts**    | Slack pings if a job is late or data looks off.            | Airflow SLA & dbt tests       |
| **Zero waste**         | Snowflake compute auto‑scales *only* when needed.          | Warehouse auto‑suspend (60 s) |

*\~18 k rows ingested per run → ≈100 k processed daily.*

---

## 🛠️ How it works

```mermaid
flowchart LR
  subgraph Extract
    A[Market APIs]
  end
  subgraph Orchestrate
    B[Airflow
(hourly DAGs)]
  end
  subgraph Transform
    C[Snowflake
(raw to models)]
  end
  subgraph Visualise
    D[Superset Dashboards]
  end
  A --> B --> C --> D
  B --> E[Slack Alerts]
```

> **Tip /** Add your own PNG/SVG architecture diagram to [`docs/`](docs/) and embed it here for extra clarity.

---

## ⚡ Quick start

> **Prerequisites** — Docker & Docker Compose, a Snowflake account, and API keys from your market‑data provider.

```bash
# 1. Clone the repo
$ git clone https://github.com/your-org/realtime-market-data-pipeline.git
$ cd realtime-market-data-pipeline

# 2. Configure secrets (edit .env with your creds)
$ cp .env.example .env
$ ${EDITOR:-nano} .env

# 3. Spin up local stack (Airflow + Superset)
$ docker compose up -d airflow superset

# 4. Bootstrap Snowflake tables & models
$ docker compose exec airflow airflow dags trigger bootstrap_init
```

|  Service   |  URL                                           |  Login              |
| ---------- | ---------------------------------------------- | ------------------- |
|  Airflow   | [http://localhost:8080](http://localhost:8080) | `airflow / airflow` |
|  Superset  | [http://localhost:8088](http://localhost:8088) | `admin / admin`     |

---

## 📂 Repository layout

```text
.
├─ dags/              # Airflow DAG definitions
├─ dbt/               # Data models & tests
├─ superset/          # Dashboard JSON exports
├─ docker-compose.yml # One‑command local stack
└─ docs/              # Architecture diagrams, ADRs & slides
```

---

## 🔧 Configuration

Edit `.env` (never commit real secrets).

|  Variable                               |  Purpose                             |
| --------------------------------------- | ------------------------------------ |
| `SNOWFLAKE_ACCOUNT`                     | Snowflake account identifier         |
| `SNOWFLAKE_USER` / `SNOWFLAKE_PASSWORD` | Warehouse login                      |
| `FTSE_API_KEY` / `FX_API_KEY`           | Market‑data provider creds           |
| `SLACK_WEBHOOK_URL`                     | Channel for success / failure alerts |

---

## ⏰ Daily schedule

|  UTC Time              |  Task                                                   |
| ---------------------- | ------------------------------------------------------- |
| **00, 15, 30, 45 min** | `ingest_ftse`, `ingest_fx` pull fresh prices            |
| **+15 min**            | `dbt_run` transforms & tests data                       |
| **On event**           | `notify_slack` fires on any SLA or data‑quality failure |

> Change the cron in `dags/` if you need a different cadence.

---

## ➕ Extending the pipeline

|  Need                             |  Where to start                                             |
| --------------------------------- | ----------------------------------------------------------- |
| Add another market (e.g., NASDAQ) | Duplicate & tweak a DAG in `dags/`                          |
| Increase load frequency           | Update `schedule_interval` cron                             |
| Deploy to AWS                     | Use `deploy/docker-compose.aws.yml` with ECS Fargate or EKS |
| Cost breakdown                    | Enable `WAREHOUSE_USAGE_HISTORY` view + Superset chart      |

---

## 🛠️ Troubleshooting

<details>
  <summary>Click to expand common issues</summary>

|  Symptom                         |  Likely Cause            |  Fix                                                    |
| -------------------------------- | ------------------------ | ------------------------------------------------------- |
| Airflow task stuck in **queued** | Docker memory low        | Allocate ≥4 GB RAM                                      |
| Dashboards stale                 | `dbt_run` failed QC test | Check Slack alert, rerun DAG                            |
| Snowflake bill spikes            | Warehouse left running   | Confirm `auto-suspend=60s` in `snowflake/warehouse.sql` |

</details>

---

## 🤝 Contributing

1. **Fork** the repo & create a feature branch.
2. Run `pre-commit install` (black, flake8, sqlfluff).
3. Open a PR with a concise description — screenshots encouraged.

*Any improvement — docs, tests, typofix — is welcome!* 💚

---

## 📝 License

This project is licensed under the [MIT License](LICENSE).

---

## 👥 Maintainers

|  Name       |  Role      |  Contact                 |
| ----------- | ---------- | ------------------------ |
|  Your Name  |  Lead Dev  |  `@your-handle` on Slack |
|  …          |  …         |  …                       |

Have a question? [Open an issue](../../issues) — no PR is too small.
