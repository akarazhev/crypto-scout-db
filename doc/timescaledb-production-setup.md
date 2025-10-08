# TimescaleDB Production Setup for crypto-scout-db

This document describes the production-ready TimescaleDB configuration used by `crypto-scout-db` to store high-volume
time-series data from Bybit and CoinMarketCap.

## Objectives

- **Reliability**: durable storage, predictable memory usage, safe restarts.
- **Performance**: optimized ingestion and queries over time ranges.
- **Data lifecycle**: automatic compression and retention at scale.
- **Operations**: health checks, backups, and clear runbooks.

---

## Summary of changes

- **Compose**: `podman-compose.yml`
    - Init SQL mounted read-only, added `shm_size`, `ulimits`, env, and improved healthcheck.
    - Added PostgreSQL tuning via `command: [postgres, -c ...]`.
    - Added daily backup sidecar service with retention.
    - Increased resource limits to align with tuning.
- **SQL init**: `script/init.sql`
    - Idempotent hypertables with 1-day chunking.
    - Reorder policies on timestamp indexes.
    - Retention policies for Bybit/CMC data.

---

## Log review and remediations

- **[initialization-success]** Database initialized, TimescaleDB extension installed, tuning applied. Service restarts
  once to apply tuned `postgresql.conf` and then becomes ready: `database system is ready to accept connections`.
- **[locale-warning]** `sh: locale: not found` and `no usable system locales were found` during init on Alpine base.
  These are benign and do not affect operation; no action taken.
- **[bgworker-template]** `TimescaleDB background worker connected to template database, exiting` occurs briefly during
  extension installation. This is expected and harmless in init phase.
- **[compression-warnings]** We updated compression ordering to stabilize blocks and address notices:
    - Tickers (`bybit_spot_tickers_*`): `compress_orderby = 'timestamp DESC, id DESC'` (no `segment_by`).
    - Launch Pool (`bybit_lpl`): `compress_segmentby = 'return_coin'`, `compress_orderby = 'stake_begin_time DESC, id DESC'`.
    - FGI (`cmc_fgi`): `compress_segmentby = 'name'`, `compress_orderby = 'timestamp DESC, id DESC'`.
      Informational messages during init are acceptable; future compression will use the updated order-by.
- **[schema-type-warning]** `VARCHAR(50)` for `return_coin` changed to `TEXT` as suggested by Timescale during init.
- **[backup-sidecar]** Backup sidecar started with `@daily` cron and exposes port `8080` for health checking. Ensure
  `secrets/postgres-backup.env` exists and credentials match `timescaledb.env`.

---

## Container services (podman-compose.yml)

- **Image**: `timescale/timescaledb:latest-pg17`
- **Service**: `postgres`
    - **Persistence**
        - Data: `./data/postgresql -> /var/lib/postgresql/data`
        - Init: `./script/init.sql -> /docker-entrypoint-initdb.d/init.sql:ro`
    - **Environment**
        - `env_file: ./secrets/timescaledb.env`
        - `POSTGRES_DB=crypto_scout`, `POSTGRES_USER=crypto_scout_db`
    - **Healthcheck**
        - `pg_isready -U $POSTGRES_USER -d $POSTGRES_DB` with `start_period: 30s`
    - **Resources**
        - `shm_size: 1g`
        - `ulimits: nofile soft=262144 hard=262144`
        - `deploy.resources.limits: cpus=2, memory=8G`
    - **PostgreSQL tuning (command)**
        - `shared_preload_libraries=timescaledb`
        - `max_connections=200`
        - `shared_buffers=2GB`, `effective_cache_size=6GB`
        - `maintenance_work_mem=1GB`, `work_mem=16MB`, `autovacuum_work_mem=256MB`
        - `wal_level=replica`, `wal_compression=on`
        - `max_wal_size=8GB`, `min_wal_size=2GB`
        - `checkpoint_timeout=15min`, `checkpoint_completion_target=0.9`
        - `timezone=UTC`, `log_min_duration_statement=500ms`, `log_checkpoints=on`
        - `timescaledb.telemetry_level=off`

- **Backup sidecar**: `pgbackups` (`prodrigestivill/postgres-backup-local:latest`)
    - Schedule: `@daily`
    - Retention: `BACKUP_KEEP_DAYS=7`, `BACKUP_KEEP_WEEKS=4`, `BACKUP_KEEP_MONTHS=6`
    - Output: `./backups -> /backups`
    - Env file: `./secrets/postgres-backup.env` (single source for backup config and credentials)
    - Required keys: `POSTGRES_HOST`, `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`, `SCHEDULE`,
      `BACKUP_KEEP_DAYS`, `BACKUP_KEEP_WEEKS`, `BACKUP_KEEP_MONTHS`, `POSTGRES_EXTRA_OPTS`
    - Extra opts: `--schema=crypto_scout --blobs`

### Required secrets

Create `secrets/timescaledb.env` (do not commit):

```
POSTGRES_PASSWORD=change_me
POSTGRES_USER=crypto_scout_db
POSTGRES_DB=crypto_scout
```

Create `secrets/postgres-backup.env` (do not commit):

```
POSTGRES_HOST=postgres
POSTGRES_DB=crypto_scout
POSTGRES_USER=crypto_scout_db
POSTGRES_PASSWORD=change_me
SCHEDULE=@daily
BACKUP_KEEP_DAYS=7
BACKUP_KEEP_WEEKS=4
BACKUP_KEEP_MONTHS=6
POSTGRES_EXTRA_OPTS=--schema=crypto_scout --blobs
```

---

## Security: Harden local auth on init

- To enable SCRAM for local connections during cluster bootstrap, set `POSTGRES_INITDB_ARGS=--auth=scram-sha-256` in `secrets/timescaledb.env`.
- This only applies when the data directory is empty. To apply later, re-initialize the data volume.

## Database schema and policies (script/init.sql)

- **Extension**: installed by image init scripts; `script/init.sql` does not create the extension.
- **Schema**: `CREATE SCHEMA IF NOT EXISTS crypto_scout;` and `SET search_path TO public, crypto_scout;`

### Tables and hypertables

- `crypto_scout.cmc_fgi (timestamp)` → 1-day chunks
- `crypto_scout.bybit_spot_tickers_btc_usdt (timestamp)` → 1-day chunks
- `crypto_scout.bybit_spot_tickers_eth_usdt (timestamp)` → 1-day chunks
- `crypto_scout.bybit_lpl (stake_begin_time)` → 1-day chunks
- Hypertable creation is idempotent (`if_not_exists => TRUE`).

### Indexes

- Time indexes for ranges:
    - `idx_cmc_fgi_timestamp`, `idx_bybit_spot_tickers_btc_usdt_timestamp`, `idx_bybit_spot_tickers_eth_usdt_timestamp`,
      `idx_bybit_lpl_stake_begin_time`
- Selectivity helpers: `idx_bybit_lpl_return_coin`.

### Compression

- Enabled on all hypertables with time-order:
    - `bybit_spot_tickers_btc_usdt`: `compress_orderby = 'timestamp DESC, id DESC'`
    - `bybit_spot_tickers_eth_usdt`: `compress_orderby = 'timestamp DESC, id DESC'`
    - `bybit_lpl`: `compress_segmentby = 'return_coin'`, `compress_orderby = 'stake_begin_time DESC, id DESC'`
    - `cmc_fgi`: `compress_segmentby = 'name'`, `compress_orderby = 'timestamp DESC, id DESC'`
- **Compression policy**: compress chunks older than 7 days for all hypertables.

#### Apply to an existing initialized cluster

Run these once to update compression settings without rebuilding the cluster:

```sql
-- BTC/USDT
ALTER TABLE crypto_scout.bybit_spot_tickers_btc_usdt SET (
  timescaledb.compress,
  timescaledb.compress_orderby = 'timestamp DESC, id DESC'
);

-- ETH/USDT
ALTER TABLE crypto_scout.bybit_spot_tickers_eth_usdt SET (
  timescaledb.compress,
  timescaledb.compress_orderby = 'timestamp DESC, id DESC'
);

-- Launch Pool
ALTER TABLE crypto_scout.bybit_lpl SET (
  timescaledb.compress,
  timescaledb.compress_segmentby = 'return_coin',
  timescaledb.compress_orderby = 'stake_begin_time DESC, id DESC'
);

-- Fear & Greed Index
ALTER TABLE crypto_scout.cmc_fgi SET (
  timescaledb.compress,
  timescaledb.compress_segmentby = 'name',
  timescaledb.compress_orderby = 'timestamp DESC, id DESC'
);
```

### Reorder policies

- Reorder open chunks by their time index for better locality:
    - BTC/USDT, ETH/USDT, FGI by timestamp index; LPL by `stake_begin_time` index.

### Retention policies

- High-volume ticker data (Bybit): keep 180 days.
- Lower-volume reference (FGI, LPL): keep ~730 days.

### Permissions

- Grants to `crypto_scout_db` on schema, tables, and sequences.

---

## Operations

- **Start**
    - Ensure directories exist: `./data/postgresql`, `./backups`, and secrets populated (`./secrets/timescaledb.env`,
      `./secrets/postgres-backup.env`).
    - Bring up with Podman Compose (example):

```sh
podman compose up -d
```

- **Health**
    - Check container health status via compose; service is healthy when `pg_isready` succeeds.

- **Connect**

```sh
psql "host=localhost port=5432 dbname=crypto_scout user=crypto_scout_db"
```

- **Backups**
    - Daily dumps saved under `./backups/` with rotation.
    - Restore with matching client tools (`psql`/`pg_restore`) as appropriate for dump format.

- **Vacuum/Analyze**
    - Defaults plus increased `autovacuum_work_mem` target large tables. Monitor bloat and adjust if needed.

---

## Sizing and rationale

- Memory: `shared_buffers=2GB` (~25% of 8GB), `effective_cache_size=6GB` (~75%). Adjust proportionally to host capacity.
- WAL/checkpoints: larger `max_wal_size` and higher `checkpoint_completion_target` smooth I/O for steady ingestion.
- Compression after 7 days reduces storage and speeds long-range scans.
- Retention prevents unbounded growth; tune windows to match analytics needs and storage budget.

---

## Validation checklist

- **Security**: credentials in `secrets/timescaledb.env`; init SQL mounted read-only.
- **Persistence**: data and backups directories exist with correct permissions.
- **Observability**: slow statements and checkpoints logged.
- **Capacity**: container limits align with PostgreSQL memory settings.
- **Policies**: compression, reorder, and retention present for all hypertables.

---

## Recommended next steps

- Add monitoring (e.g., postgres_exporter + Prometheus/Grafana).
- Consider WAL archiving to object storage for PITR if required.
- Periodically review retention and compression windows based on usage and cost.

---

## File references

- Compose: `podman-compose.yml`
- Schema/Policies: `script/init.sql`
- This document: `doc/timescaledb-production-setup.md`
