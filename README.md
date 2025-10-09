# crypto-scout-db

Production-ready TimescaleDB 17 + backup stack for crypto time-series (Bybit, CoinMarketCap). Podman Compose deploy with
pre-tuned PostgreSQL, schema bootstrap, compression, retention, and daily backups.

---

## Features

- **TimescaleDB 17 container** (`timescale/timescaledb:latest-pg17`) with tuned PostgreSQL params.
- **Schema bootstrap** via `script/init.sql`:
    - Hypertables with 1-day chunks for: `cmc_fgi`, `bybit_spot_tickers_btc_usdt`, `bybit_spot_tickers_eth_usdt`,
      `bybit_lpl`.
    - Compression policies (compress after 7 days) and reorder policies on time indexes.
    - Retention policies (Bybit tickers: 180 days; FGI/LPL: ~730 days).
    - Extensions: `timescaledb`, `pg_stat_statements`.
- **Backups sidecar** (`prodrigestivill/postgres-backup-local:latest`) with schedule and retention to `./backups/`.
- **Secrets management** via env files in `secrets/` (examples provided, real files are git ignored).
- **Healthcheck** using `pg_isready` and observability settings enabled.

---

## Architecture

- **Service `postgres`** (container name `crypto-scout-db`)
    - Ports: `5432:5432`
    - Volumes:
        - Data: `./data/postgresql -> /var/lib/postgresql/data`
        - Init SQL (read-only): `./script/init.sql -> /docker-entrypoint-initdb.d/init.sql:ro`
    - Env file: `./secrets/timescaledb.env`
    - Healthcheck: `pg_isready -U $POSTGRES_USER -d $POSTGRES_DB`
    - Notable parameters (see `podman-compose.yml`): `shared_preload_libraries=timescaledb,pg_stat_statements`, WAL and
      memory tuning, telemetry off, slow query logging.

- **Service `pgbackups`** (backup sidecar)
    - Image: `prodrigestivill/postgres-backup-local:latest`
    - Env file: `./secrets/postgres-backup.env`
    - Volume: `./backups -> /backups`
    - Runs on schedule (e.g., `@daily`) with configurable retention.

---

## Quick start

1. Prepare directories and secrets (see `secrets/README.md` for detailed steps):

```bash
mkdir -p ./data/postgresql ./backups ./secrets
cp ./secrets/timescaledb.env.example ./secrets/timescaledb.env
cp ./secrets/postgres-backup.env.example ./secrets/postgres-backup.env
chmod 600 ./secrets/*.env
```

2. Start the stack with Podman Compose:

```bash
podman compose up -d
```

3. Verify health and logs:

```bash
podman ps
podman logs crypto-scout-db --tail=200
```

4. Connect with psql:

```bash
psql "host=localhost port=5432 dbname=crypto_scout user=crypto_scout_db"
```

---

## Secrets

Real env files live in `secrets/` and are git ignored. Examples:

- `secrets/timescaledb.env.example` → copy to `secrets/timescaledb.env` and adjust.
- `secrets/postgres-backup.env.example` → copy to `secrets/postgres-backup.env` and adjust.

Important keys:

- TimescaleDB: `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`, optional `POSTGRES_INITDB_ARGS`, `TZ`.
- Backup sidecar: `POSTGRES_HOST`, `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`, `SCHEDULE`, `BACKUP_KEEP_*`,
  `POSTGRES_EXTRA_OPTS`, `TZ`.

Security tip: to enable SCRAM auth on fresh clusters, set `POSTGRES_INITDB_ARGS=--auth=scram-sha-256` in
`secrets/timescaledb.env` before the first startup (re-init data dir to apply later).

---

## Schema and policies

Defined in `script/init.sql`:

- Schema: `crypto_scout` and persisted `search_path`.
- Hypertables (1-day chunks):
    - `crypto_scout.cmc_fgi (timestamp)`
    - `crypto_scout.bybit_spot_tickers_btc_usdt (timestamp)`
    - `crypto_scout.bybit_spot_tickers_eth_usdt (timestamp)`
    - `crypto_scout.bybit_lpl (stake_begin_time)`
- Indexes on time columns for efficient range scans.
- Compression policies: compress chunks older than 7 days.
- Reorder policies on the time indexes of all hypertables.
- Retention policies:
    - Bybit tickers: 180 days
    - FGI, LPL: ~730 days

---

## Operations

- Start/stop: `podman compose up -d` / `podman compose down`
- Health: the `postgres` container reports healthy when `pg_isready` succeeds.
- Backups: daily dumps stored under `./backups/` per `postgres-backup.env`.
- Restore: use `psql`/`pg_restore` matching dump format and `POSTGRES_EXTRA_OPTS` used.

---

## Directory layout

- `podman-compose.yml` – services, volumes, healthchecks, tuning
- `script/init.sql` – schema, hypertables, policies, privileges
- `secrets/` – env files (examples provided; real files git ignored)
- `backups/` – backup output directory
- `doc/` – in-depth operations: `doc/timescaledb-production-setup.md`

---

## License

Unlicense. See `LICENSE`.

---

## Short description for GitHub

Production-ready TimescaleDB 17 + backup stack for crypto time-series (Bybit, CoinMarketCap). Podman Compose deploy,
schema bootstrap, compression, retention, and daily backups.