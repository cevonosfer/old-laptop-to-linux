# PHASE2

# Homepage Deployment

As i added more services, remembering individual IP addresses and ports became increasingly inconvenient.

Homepage was deployed to act as a central dashboard for the homelab.

The dashboard provides quick access to:

- Proxmox
- OpenMediaVault
- Portainer
- Nextcloud
- Uptime Kuma
- Future services

During deployment, Homepage initially failed to load and reported a host validation error.

This was caused by Homepage's host validation feature, which rejects requests from unapproved hosts.

The issue was solved by adding the allowed hosts setting in the yaml file and recreating the container.

---

# Uptime Kuma Deployment

After Homepage was operational, I deployed Uptime Kuma to introduce monitoring capabilities.

The purpose of Uptime Kuma is to monitor both infrastructure and services.

Monitors were configured for:

## Infrastructure

- Proxmox Host
- OpenMediaVault VM
- Ubuntu Docker VM

## Applications

- Homepage
- Nextcloud
- Portainer
- OpenMediaVault Web Interface

For services using self-signed certificates, TLS verification was disabled within the monitor configuration.

This allowed internal services to be monitored successfully without requiring certificates.

The monitoring setup now provides visibility into service availability and server status.

---

# Current Status

Completed components:

- Proxmox VE
- OpenMediaVault
- Ubuntu Server
- Nextcloud
- Homepage
- Uptime Kuma
- Portainer

Current architecture:

```text
Proxmox VE
├── OpenMediaVault
│
└── Ubuntu Server
    ├── Docker
    ├── Portainer
    ├── Nextcloud
    ├── Homepage
    └── Uptime Kuma
```

---

# Next Planned Services

Future additions currently being considered:

- Jellyfin
- Immich
- Reverse Proxy
- Grafana
- VPN Access

The homelab has now progressed beyond initial infrastructure setup and into the application and services phase.