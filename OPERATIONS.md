# Monitoring startup and triage checklist

Use this checklist when the containers are running but dashboards are empty.
It separates Compose configuration, scrape health and dashboard problems.

## Before starting

- Read [docker-compose.yml](docker-compose.yml) and [Caddyfile](caddy/Caddyfile).
  Caddy publishes ports 3000, 8080, 9090, 9093 and 9091 on the host.
  Restrict access to a trusted network before starting this stack.
- Replace the default admin credentials and the Caddy password hash together.
  Grafana has its own login; Caddy protects the other published endpoints.
  Changing an environment variable does not necessarily reset an existing
  Grafana user's password in the persistent database.
- This stack mounts host filesystems and runs cAdvisor privileged. Use a
  dedicated lab host; Docker Desktop metrics may describe its Linux VM.
- Do not commit environment files, password hashes or notification secrets.

## Configuration-only checks

Run from the repository root with Docker Compose v2 installed:

```sh
docker compose config --quiet
docker compose config --services
```

These commands do not start containers. A zero exit code validates Compose
configuration, not connectivity or correct Prometheus queries. Avoid publishing
full rendered configuration because interpolated values can contain credentials.

## Triage an already-running stack

```sh
docker compose ps
docker compose logs --tail=100 prometheus
docker compose logs --tail=100 caddy
docker compose logs --tail=100 grafana
```

| Symptom | Check next |
| --- | --- |
| Unauthorized response on port 9090 | Caddy user/hash and browser credentials |
| Grafana opens but has no series | Prometheus target health, then datasource configuration |
| One exporter is down | Exporter logs, service DNS and its configured scrape target |
| Host metrics differ from the physical machine | Docker VM and mounted filesystem boundaries |
| Old dashboards or users persist | Existing Grafana volume and provisioning settings |

Scrub logs before sharing. Do not delete volumes to resolve login or empty-chart
issues: the Grafana and Prometheus volumes contain persistent state.

## Development note

This troubleshooting guide was added with AI assistance. Upstream code,
licenses and contributor attribution remain unchanged.
