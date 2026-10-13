                         INTERNET
                            │
                  ┌─────────┴──────────┐
                  |     deSEC DNS      │
                  └─────────┬──────────┘
                            │
                 ┌──────────┴───────────┐
                 │                      │
                 ▼                      ▼
           Remote Devices             Router
                 │                      │
              Tailscale                LAN
                 │                      |
                 │                      │
                 |                      |
                 |                      │                      
                 |                      ▼                      
                 └──────────────>  Raspberry Pi             
                 |                      |
                 |                      ▼
                 └──────────────>   Proxmox VE         
                 │                      |                    
                 │                      |                      
                 │                      |                      
                 |               ┌──────┴──────┐
                 |               │             │
                 |               ▼             ▼
                 └───────>   Ubuntu VM        OMV
                                 |    
                                 |
                                 │
                                 ▼
                            Docker Host
                                 │
                                 ▼
                              Traefik
                                 |      
                                 |     
                                 |      
                                 |
                                 ▼
                            Docker proxy
                              network
                                 │
                      ┌──────────┼──────────┐─────────┐───────┐─────────┐────────┐
                      │          │          │         |       |         |        |
                      ▼          ▼          ▼         ▼       ▼         ▼        ▼
                   Nextcloud  Grafana   Portainer  Homepage  Kuma  Prometheus Exporters  
                      │          
                      ▼          
                   MariaDB                     