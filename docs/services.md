# Services

## Networking
### OPNsense
Primary router/firewall for the homelab. Handles LAN routing, VPN access, and core network control.

### OpenVPN
Remote access into the homelab. Current remote-access VPN solution, with WireGuard as a likely future migration target.

### AdGuard Home
Primary DNS filtering layer for clients.

### Unbound
Recursive/upstream DNS component behind AdGuard Home.

## Infrastructure
### Main Server (`seranogenomics`)
Primary Linux host running self-hosted services and Docker workloads.

### NAS (`nas`)
Primary storage system for archive/media/infrastructure data.

### UPS / NUT
Used for power event monitoring and controlled shutdown behavior.

## Applications
### Jellyfin / media stack
Self-hosted media services running through Docker/YAMS-style stack.

### Home Assistant
Home automation platform running as a VM.

## Documentation Notes
Each service should eventually include:
- host location
- purpose
- dependencies
- backup/recovery notes
- known issues
