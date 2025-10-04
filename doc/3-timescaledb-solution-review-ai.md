# Context: Solution review of the `timescaledb` production ready service

In this `crypto-scout-db` project the `timescaledb` server has been configured in a container for messaging between
services and to collect crypto data. So you will need to review and update the configuration to be sure that it is ready
for production.

## Roles

Take the following roles:

- Expert dev-opts engineer.

## Conditions

- Use the best practices and design patterns.
- Do not hallucinate.
- Use the latest technology stack: `timescale/timescaledb:latest-pg13`.

## Tasks

- As the expert dev-opts engineer review the current `timescaledb` service and verify that it has to be ready for
  production.
- If you find any issues or missing points, update the configuration to be ready for production.
- Recheck your proposal and make sure that they are correct and haven't missed any important points.
- Write a report with your proposal and implementation into `doc/timescaledb-production-setup.md`.