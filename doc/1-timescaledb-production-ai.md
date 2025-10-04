# Context: Development of the `timescaledb` production ready service

In this `crypto-scout-db` project we are going to use the `timescaledb` server in a container for messaging between
services and to collect crypto data. So you will need to review and update the `podman-compose.yml` file for the
production.

## Roles

Take the following roles:

- Expert dev-opts engineer.

## Conditions

- Use the best practices and design patterns.
- Do not hallucinate.
- Use the latest technology stack: `timescale/timescaledb:latest-pg13`.

## Tasks

- As the expert dev-opts engineer review the current TimescaleDB service implementation in `podman-compose.yml` and
  update it to be ready for production.
- Use streams to collect crypto data: `crypto-bybit-stream`.
- Use a common queue for incoming events and for messaging between services: `crypto-scout-collector-queue`.
- Use streams to collect metrics from `Bybit` and `CoinMarketCap`: `metrics-bybit-stream`, `metrics-cmc-stream`.
- Recheck your proposal and make sure that they are correct and haven't missed any important points.
- Write a report with your proposal and implementation into `doc/timescaledb-production-setup.md`.