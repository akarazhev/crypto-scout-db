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
  - `POSTGRES_INITDB_ARGS` (optional; applies at initdb only)

- `postgres-backup.env` — env file for the backup sidecar providing:
   - `POSTGRES_HOST`
   - `POSTGRES_DB`
   - `POSTGRES_USER`
   - `POSTGRES_PASSWORD`
   - `SCHEDULE`
   - `BACKUP_KEEP_DAYS`
   - `BACKUP_KEEP_WEEKS`
   - `BACKUP_KEEP_MONTHS`
   - `POSTGRES_EXTRA_OPTS`
   - `POSTGRES_PASSWORD`
      
Quick start

1. Create the directory if missing:
   ```bash 
   mkdir -p ./secrets
   ```

2. Create the env file from example:
   ```bash 
   cp ./secrets/timescaledb.env.example ./secrets/timescaledb.env
   ```
   Or generate strong values:
   ```bash
   POSTGRES_DB=crypto_scout
   POSTGRES_USER=crypto_scout_db
   POSTGRES_PASSWORD=$(openssl rand -base64 48 | tr -dc 'A-Za-z0-9' | head -c 48)
   TIMESCALEDB_TELEMETRY=off
   TIMESCALEDB_TUNE_MAX_CONNECTIONS=100
   TIMESCALEDB_TUNE_MAX_BACKGROUND_WORKERS=8
   TIMESCALEDB_TUNE_MAX_PARALLEL_WORKERS=4
   TIMESCALEDB_TUNE_MEMORY=512MB
   POSTGRES_INITDB_ARGS=--auth=scram-sha-256
   
   printf "POSTGRES_DB=%s\nPOSTGRES_USER=%s\nPOSTGRES_PASSWORD=%s\nTIMESCALEDB_TELEMETRY=%s\nTIMESCALEDB_TUNE_MAX_CONNECTIONS=%s\nTIMESCALEDB_TUNE_MAX_BACKGROUND_WORKERS=%s\nTIMESCALEDB_TUNE_MAX_PARALLEL_WORKERS=%s\nTIMESCALEDB_TUNE_MEMORY=%s\nPOSTGRES_INITDB_ARGS=%s\n" \
     "$POSTGRES_DB" "$POSTGRES_USER" "$POSTGRES_PASSWORD" "$TIMESCALEDB_TELEMETRY" "$TIMESCALEDB_TUNE_MAX_CONNECTIONS" \
     "$TIMESCALEDB_TUNE_MAX_BACKGROUND_WORKERS" "$TIMESCALEDB_TUNE_MAX_PARALLEL_WORKERS" "$TIMESCALEDB_TUNE_MEMORY" \ 
     "$POSTGRES_INITDB_ARGS" > ./secrets/timescaledb.env
   
   chmod 600 ./secrets/timescaledb.env
   ```

4. Create the backup env file (this is the ONLY env file loaded by the backup sidecar) using values from
   `timescaledb.env` to ensure credentials match:
   ```bash
   source ./secrets/timescaledb.env

   POSTGRES_HOST=postgres
   SCHEDULE=@daily
   BACKUP_KEEP_DAYS=7
   BACKUP_KEEP_WEEKS=4
   BACKUP_KEEP_MONTHS=6
   POSTGRES_EXTRA_OPTS=--schema=crypto_scout --blobs

   printf "POSTGRES_HOST=%s\nPOSTGRES_DB=%s\nPOSTGRES_USER=%s\nPOSTGRES_PASSWORD=%s\nSCHEDULE=%s\nBACKUP_KEEP_DAYS=%s\nBACKUP_KEEP_WEEKS=%s\nBACKUP_KEEP_MONTHS=%s\nPOSTGRES_EXTRA_OPTS=%s\n" \
     "$POSTGRES_HOST" "$POSTGRES_DB" "$POSTGRES_USER" "$POSTGRES_PASSWORD" "$SCHEDULE" "$BACKUP_KEEP_DAYS" \
     "$BACKUP_KEEP_WEEKS" "$BACKUP_KEEP_MONTHS" "$POSTGRES_EXTRA_OPTS" \
     > ./secrets/postgres-backup.env

   chmod 600 ./secrets/postgres-backup.env
   ```

Notes

- Never commit real secrets. Only commit example files.
- For the backup sidecar, `podman-compose.yml` loads only `postgres-backup.env`.
- Ensure `POSTGRES_DB`, `POSTGRES_USER`, and `POSTGRES_PASSWORD` in `postgres-backup.env` match the values in
  `timescaledb.env`. Mismatched credentials will cause backup failures.
- `POSTGRES_HOST` in `postgres-backup.env` should be the compose service name of the database: `postgres`.
- Files matching `secrets/*.env` are gitignored (see project `.gitignore`). Keep permissions restrictive (`chmod 600`).
- To harden local auth during bootstrap, add `POSTGRES_INITDB_ARGS=--auth=scram-sha-256` to `secrets/timescaledb.env`.
  This only takes effect when creating a new data directory; to apply later, re-initialize `./data/postgresql`.
