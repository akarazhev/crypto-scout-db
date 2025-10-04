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

Notes

- Never commit real secrets. Only commit example files.
