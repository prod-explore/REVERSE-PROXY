# Reverse Proxy

Centralized Nginx reverse proxy for routing and SSL termination across multiple containerized services. One entry point, automated HTTPS, isolated Docker networks per service.

## What it does

- Terminates TLS for every service behind a single Nginx instance
- Automates certificate issuance and renewal with **Certbot** (Let's Encrypt)
- Routes each subdomain to its own backend container over an isolated Docker network
- Handles SSE/streaming connections (buffering disabled, extended read/send timeouts) for long-lived API responses

## Architecture

```
Internet → Nginx (80/443) → Docker network per service → backend container
                ↓
            Certbot (auto-renewal)
```

Each service lives on its own external Docker network (e.g. `mindgate_net`, `futumore-network`), so the proxy only talks to containers it's explicitly connected to — no shared network sprawl between unrelated projects.

## Structure

```
conf.d/           # one server block per service (HTTP→HTTPS redirect + SSL vhost)
docker-compose.yml # nginx-proxy + certbot services, network definitions
```

## Services routed

| Config | Purpose |
|---|---|
| `mindgate.conf` | Local LLM API gateway |
| `futumore.conf` | Agency site |
| `skarpa.conf` | Sports registration platform |
| `nextouch.conf` | Cross-platform touchpad backend |
| `hypnagogia.conf` | Digital e-commerce platform |

## Setup

```bash
docker compose up -d
```

Certificates are requested/renewed automatically via the `certbot` container against the paths mounted in `certs/`. SSL keys, certs, and `.env` files are gitignored — nothing sensitive is committed.

## Notes

Config pattern per service:
1. Port 80 block — ACME challenge passthrough + redirect to HTTPS
2. Port 443 block — SSL config, proxy headers, upstream resolution via Docker's internal DNS resolver (`127.0.0.11`) so backend containers can restart without breaking routing
