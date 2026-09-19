# Home Lab Server

A small home server (an old laptop repurposed as a Linux box), manually
configured and hardened, running Docker-based services with uptime
monitoring and alerting.

## How to run

```bash
docker compose up -d
```

Grafana available at `localhost:3000`, Prometheus at `localhost:9080`, cAdvisor at `localhost:8080`.

## Why this exists

Wanted to build hands-on infrastructure/networking skills beyond what I
did in my previous sysadmin role — starting with a server I fully own and
control, from OS install to hardening to monitoring.

## Current setup
Fresh Ubuntu install

│
▼
SSH hardening key-only auth, no root login, custom port

│
▼
fail2ban bans repeated failed SSH attempts

│
▼
Docker data-root moved to /home (see below)

│
▼
Uptime Kuma uptime checks + Telegram alerts on downtime

## Monitoring stack

In addition to Uptime Kuma (up/down checks), the server runs a full metrics stack via `docker-compose.yml`:

- **node-exporter** — host-level metrics (CPU, memory, disk, network)
- **cAdvisor** — per-container resource usage
- **Prometheus** — scrapes and stores metrics from both exporters
- **Grafana** — dashboards for visualizing everything above

Config: [`docker-compose.yml`](./docker-compose.yml), [`prometheus.yml`](./prometheus.yml)

## Key design decisions

- **Docker data lives on `/home`, not `/`.** The root partition on this
  hardware is small — a partitioning mistake from the initial Ubuntu
  install a year ago. Docker's `data-root` is repointed via
  `/etc/docker/daemon.json` so container images/volumes don't fill up the
  system partition. Lesson learned for next time: plan disk partitioning
  before installing anything on top.
- **SSH is key-only, no root login, no password auth.** Combined with
  fail2ban, this closes off the most common attack surface (brute-forced
  passwords).
- **Telegram alerting** is configured in the Uptime Kuma UI, so I get
  notified immediately if the server or any monitored service goes down.

## What this demonstrates

- Linux server hardening (SSH, fail2ban)
- Docker administration (custom data-root, container lifecycle)
- Monitoring & alerting (Uptime Kuma + Telegram)
- Practical troubleshooting on real (not cloud, not tutorial) hardware
- Metrics collection & visualization (Prometheus, Grafana, cAdvisor)

## Roadmap

- [ ] Automate this setup with Ansible (currently done manually — next
      step is turning it into a reproducible playbook)
- [✓] Prometheus + Grafana for resource metrics, not just up/down checks
- [ ] Reverse proxy (Nginx/Traefik) with HTTPS in front of dashboards
- [ ] Automated off-box backups
