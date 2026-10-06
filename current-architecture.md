## Current Architecture

```text
                         ┌──────────────┐
                         │    Router    │
                         └──────┬───────┘
                                │
                                ▼
                         ┌──────────────┐
                         │  Proxmox VE  │
                         └──────┬───────┘
                                │
                    ┌───────────┴───────────┐
                    │                       │
                    ▼                       ▼
             ┌─────────────┐         ┌─────────────┐
             │ OpenMediaVault│         │ Ubuntu VM   │
             └─────────────┘         └──────┬──────┘
                                            │
                                         Docker
                                            │
              ┌─────────────┬─────────────┬─┴─────────────┬─────────────┐
              │             │             │               │             │
              ▼             ▼             ▼               ▼             ▼
         Portainer      Nextcloud      Homepage       Prometheus      Grafana
                                            │               │
                                            ▼               │
                                       Uptime Kuma          │
                                                            │
                              ┌─────────────────────────────┘
                              │
                ┌─────────────┼──────────────┐
                │             │              │
                ▼             ▼              ▼
         Node Exporter   PVE Exporter    cAdvisor
                │             │              │
                └─────────────┼──────────────┘
                              │
                              ▼
                         Prometheus
                              │
                              ▼
                           Grafana
