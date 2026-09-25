# Homelab Progress

## Prometheus and Grafana

Next, I moved on to monitoring.

I installed Prometheus and Grafana using Docker Compose. Prometheus was configured to collect metrics from the Ubuntu Docker VM, while Grafana was used to visualize the collected metrics.

Initially, Prometheus would not start because of YAML formatting and indentation problems in the `prometheus.yml` file.

One of the errors was:

    yaml: line 10: did not find expected '-' indicator

The problem was caused by incorrect indentation in the `scrape_configs` section.

After correcting the YAML structure, Prometheus started successfully.

### Node Exporter

I added Node Exporter so Prometheus could monitor the Ubuntu VM itself.

The first problem was that Prometheus was showing the target as `UP`, but the `up` metric was returning:

    0

Prometheus was also reporting an invalid hostname because the target address had accidentally been entered with formatting that was not valid YAML.

After correcting the target to the proper IP address, Node Exporter started working and Prometheus was able to collect the Ubuntu VM metrics.

---

## Proxmox Monitoring

After getting Node Exporter working, I added the Prometheus PVE Exporter to monitor the Proxmox host and its virtual machines.

The exporter was running correctly and was able to communicate with Proxmox.

I verified that it was producing metrics such as:

    pve_up{id="node/monolith"} 1.0
    pve_up{id="qemu/100"} 0.0
    pve_up{id="qemu/101"} 1.0

This showed that the exporter was successfully retrieving information from Proxmox.

### Prometheus PVE Exporter Problem

The confusing part was that Prometheus was reporting:

    Error scraping target:
    server returned HTTP status 500 INTERNAL SERVER ERROR

At first, it looked like the exporter itself was broken. However, testing the exporter directly showed that it was working.

The important discovery was that the PVE exporter uses the `/pve` endpoint together with a target parameter.

The following request returned the actual Proxmox metrics:

    http://192.168.1.110:9221/pve?target=192.168.1.137

The problem was therefore that Prometheus was scraping `/pve` without specifying the Proxmox target.

I changed the Prometheus configuration so that the Proxmox job passed the correct target:

    - job_name: "proxmox"
      metrics_path: /pve
      params:
        target:
          - "192.168.1.137"
      static_configs:
        - targets:
            - "192.168.1.110:9221"

After correcting the configuration, Prometheus successfully scraped the Proxmox metrics.

---

## cAdvisor

After getting Proxmox monitoring working, I added cAdvisor to monitor the Docker containers running on the Ubuntu VM.

cAdvisor provides container-level metrics such as:

- CPU usage
- Memory usage
- Network traffic
- Disk I/O
- Container activity

I also checked whether Redis was necessary for cAdvisor.

It was not. cAdvisor collects Docker container metrics directly, so Redis was not required for the monitoring setup.

---

## Grafana Metrics Dashboard

Once Node Exporter, PVE Exporter and cAdvisor were working, I started building dashboards for all of them.

The idea was to have dashboards containing the important parts of the entire homelab for every monitoring source.

The dashboards i used are:

### Proxmox (10347)
### Ubuntu (1860)
### Docker (14282)

## Services

Uptime Kuma remains responsible for monitoring whether the actual services are reachable.

The monitoring stack is therefore split between:

    PVE Exporter  → Proxmox and VMs
    Node Exporter → Ubuntu VM
    cAdvisor      → Docker containers
    Uptime Kuma   → Service availability
    Prometheus    → Collects the metrics
    Grafana       → Visualizes the metrics

---
Right now i think i am done with services and apps although i will continue adding them in the future, 
The next phase will be about networking and opening up to the internet.


