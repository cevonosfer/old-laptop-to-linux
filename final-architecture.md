                                         INTERNET
                                             │
                                  Public IP / Dynamic DNS
                                             │
                                          openVPN
                                             │
                                     Router w/OPNsense
                                             │
                                      Gigabit Switch
                                             │
 ┌───────────────────────────────────────────┼────────────────────────┐
 │                       |                   │                        │
 │                       |                   │                        │
NAS w/trueNAS          Backup              Main PC                 Admin PC
                                             │
                                             │
                                          Proxmox 
                                             │
 ┌───────────────────────────────────────────┼──────────────────────────────────────────────┐
 │                                                                                             
 │  General                                                                           
 │  ├── Portainer                                                                              
 │  ├── Vaultwarden                                                                               
 |                                                                         
 │
 │  Self Hosting 
 │  ├── Seafile
 │  ├── Jellyfin
 |  ├── Wikipedia
 │  ├── ...
 │
 |
 │  Security
 │  ├── Prometheus
 │  ├── openVAS
 │  ├── Suricata
 │  └── Grafana
 │
 |
 │  Networking
 │  ├── openVPN
 │  ├── Nginx Proxy Manager
 │  ├── Internal DNS
 │  ├── DHCP
 │
 └────────────────────────────────────────────────────────────────────────────────────────────┘

                                    CYBERSECURITY LAB

                Kali Linux ─────────────┐
                                        │
           Metasploitable ──────────────┼────────► Isolated Lab Network
                                        │
                   DVWA ────────────────┤
                                        │
              Windows 11 VM ────────────┤
                                        │
             Vulnerable Ubuntu ─────────┘

                          │
                          ▼

                  Wazuh SIEM
                      │
          ┌───────────┼────────────┐
          │           │            │
      OpenVAS     Suricata      Zeek
          │           │            │
          └───────────┼────────────┘
                      │
                  Grafana Dashboards