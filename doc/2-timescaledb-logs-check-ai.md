# Context: Logs checks of the running `timescaledb` container as the production ready service

In this `crypto-scout-db` project we have configured the `timescaledb` server in a container for messaging between
services and to collect crypto data. So you will need to check logs and make sure that the service is running correctly.

## Roles

Take the following roles:

- Expert dev-opts engineer.

## Conditions

- Use the best practices and design patterns to fix any issues found in the logs.
- Do not hallucinate.
- Use the latest technology stack: `timescale/timescaledb:latest-pg13`.

## Tasks

- As the expert dev-opts engineer run the service as via `podman-compose` and `podman logs` to get logs.
- Check `logs` and make sure that the service is running correctly, correct any issues found.
- Recheck your proposal and make sure that they are correct and haven't missed any important points.
- Write a report with your proposal and implementation into `doc/timescaledb-production-setup.md`.