---
description: How to start and verify the full ticker-nerve data engineering stack
---

# Start the Full Stack

## Prerequisites
- Docker Desktop running (with WSL2 backend enabled)

## 1. Start the Compose Stack (Airflow + Kafka + Postgres)

// turbo
```bash
docker compose up -d
```

This starts 8 services: `postgres`, `redis`, `airflow-init`, `airflow-webserver`, `airflow-scheduler`, `airflow-worker`, `zookeeper`, `kafka`.

First run pulls images (~5-10 min depending on bandwidth).

## 2. Verify Compose Services

// turbo
```bash
docker compose ps
```

All services should show `Up (healthy)` except `airflow-init` which exits after setup.

## 3. Start Airbyte (separate kind cluster via abctl)

Airbyte runs on its own Kubernetes cluster managed by `abctl`. It is NOT part of docker-compose.

```bash
abctl local install
```

If already installed, just make sure Docker Desktop is running — the `airbyte-abctl-control-plane` container auto-starts.

## 4. Verify Airbyte

```bash
abctl local status
```

Or check the container directly:
// turbo
```bash
docker ps --filter "name=airbyte" --format "table {{.Names}}\t{{.Status}}"
```

## 5. Get Airbyte Credentials

```bash
abctl local credentials
```

---

## Access Points

| Service         | URL                        | Credentials              |
|-----------------|----------------------------|--------------------------|
| Airflow UI      | http://localhost:8080       | airflow / airflow        |
| Airbyte UI      | http://localhost:8000       | Run `abctl local credentials` |
| PostgreSQL      | localhost:5432              | airflow / airflow        |
| Kafka Broker    | localhost:9092 (external)   | —                        |
| Kafka Internal  | kafka:29092 (from containers) | —                     |
| Redis           | localhost:6379              | —                        |
| Zookeeper       | localhost:2181              | —                        |

## Stop Stack

```bash
# Stop compose services (keep data)
docker compose down

# Stop Airbyte (keep data)
abctl local uninstall

# Stop Airbyte AND delete data
abctl local uninstall --persisted
```

## Port Conflict Notes

- Airflow uses **8080**, Airbyte uses **8000** — no conflict by default.
- If port 8000 is taken, reinstall Airbyte with: `abctl local install --port <other-port>`
