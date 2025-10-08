# Secrets for TimescaleDB (production)

This folder holds local secrets used by `podman-compose.yml` for TimescaleDB. Files here are ignored by git (see project
`.gitignore`).

Required files:

- `timescaledb.env` — env file providing:
    - `POSTGRES_DB`
    - `POSTGRES_USER`
    - `POSTGRES_PASSWORD`
    - `TIMESCALEDB_TELEMETRY`
    - `TIMESCALEDB_TUNE_MAX_CONNECTIONS`
    - `TIMESCALEDB_TUNE_MAX_BACKGROUND_WORKERS`
    - `TIMESCALEDB_TUNE_MAX_PARALLEL_WORKERS`
    - `TIMESCALEDB_TUNE_MEMORY`

- `postgres-backup.env` — env file for the backup sidecar providing:
    - `POSTGRES_HOST`, `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`
    - `SCHEDULE`, `BACKUP_KEEP_DAYS`, `BACKUP_KEEP_WEEKS`, `BACKUP_KEEP_MONTHS`
    - `POSTGRES_EXTRA_OPTS`

Quick start

1) Create the directory if missing:
   mkdir -p ./secrets

2) Create the env file from example:
   cp ./secrets/timescaledb.env.example ./secrets/timescaledb.env
   # Or generate strong values:
   ```bash
   POSTGRES_DB=crypto_scout
   POSTGRES_USER=$(openssl rand -base64 48 | tr -dc 'A-Za-z0-9' | head -c 48)
   POSTGRES_PASSWORD=$(openssl rand -base64 48 | tr -dc 'A-Za-z0-9' | head -c 48)
   TIMESCALEDB_TELEMETRY=off
   TIMESCALEDB_TUNE_MAX_CONNECTIONS=100
   TIMESCALEDB_TUNE_MAX_BACKGROUND_WORKERS=8
   TIMESCALEDB_TUNE_MAX_PARALLEL_WORKERS=4
   TIMESCALEDB_TUNE_MEMORY=512MB
   printf "POSTGRES_DB=%s\nPOSTGRES_USER=%s\nPOSTGRES_PASSWORD=%s\nTIMESCALEDB_TELEMETRY=%s\nTIMESCALEDB_TUNE_MAX_CONNECTIONS=%s\nTIMESCALEDB_TUNE_MAX_BACKGROUND_WORKERS=%s\nTIMESCALEDB_TUNE_MAX_PARALLEL_WORKERS=%s\nTIMESCALEDB_TUNE_MEMORY=%s\n" "$POSTGRES_DB" "$POSTGRES_USER" "$POSTGRES_PASSWORD" "$TIMESCALEDB_TELEMETRY" "$TIMESCALEDB_TUNE_MAX_CONNECTIONS" "$TIMESCALEDB_TUNE_MAX_BACKGROUND_WORKERS" "$TIMESCALEDB_TUNE_MAX_PARALLEL_WORKERS" "$TIMESCALEDB_TUNE_MEMORY" > ./secrets/timescaledb.env
   ```
3) Restrict permissions (recommended on Unix/macOS):
   chmod 600 ./secrets/timescaledb.env

4) Create backup env file from example (this is the ONLY env file loaded by the backup sidecar):
   cp ./secrets/postgres-backup.env.example ./secrets/postgres-backup.env
   # Required minimal content (edit as needed):
   # POSTGRES_HOST=postgres
   # POSTGRES_DB=crypto_scout
   # POSTGRES_USER=crypto_scout_db
   # POSTGRES_PASSWORD=...strong secret...
   # SCHEDULE=@daily
   # BACKUP_KEEP_DAYS=7
   # BACKUP_KEEP_WEEKS=4
   # BACKUP_KEEP_MONTHS=6
   # POSTGRES_EXTRA_OPTS=--schema=crypto_scout --blobs
   chmod 600 ./secrets/postgres-backup.env

Notes

- Never commit real secrets. Only commit example files.
- For the backup sidecar, `podman-compose.yml` loads only `postgres-backup.env`.
