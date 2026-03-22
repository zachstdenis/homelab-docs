# Homelab Documentation

This repository documents my homelab architecture, services, troubleshooting, deployments, and operational notes. The lab is focused on practical networking, self-hosting, Linux administration, reliability, and public-facing documentation of real infrastructure work.

## Topology

> Topology diagram placeholder. A cleaned-up visual diagram will be added later.

<!-- Uncomment once you add the image -->
<!-- ![Homelab topology](images/topology.png) -->

## Environment Summary

Current environment includes:

- OPNsense router/firewall
- Main server: `seranogenomics`
- NAS: `nas`
- LAN: `192.168.5.0/24`
- OpenVPN subnet: `192.168.6.0/24`
- 2.5 Gbps internet service
- 2.5 GbE switching on the core wired path
- Wi-Fi provided by the ISP modem/router
- AdGuard Home + Unbound for DNS
- Docker/YAMS media stack
- Home Assistant
- UPS/NUT shutdown and recovery work
- additional standalone self-hosted services such as:
  - reading stack (`Komga`, `Suwayomi`, `Calibre-Web`)
  - `Karakeep`
  - `Tailscale`

## Goals

- build practical NetOps, sysadmin, and self-hosting skills
- improve reliability and recovery workflows
- document troubleshooting and design decisions clearly
- maintain a public portfolio of real homelab work

## Core Documentation

- [Network Topology](docs/network-topology.md)
- [Services](docs/services.md)
- [Hardware](docs/hardware.md)
- [DNS](docs/dns.md)
- [Runbooks](docs/runbooks.md)
- [Backups and Recovery](docs/backups-and-recovery.md)
- [Maintenance and Updates](docs/maintenance-and-updates.md)
- [Roadmap](docs/roadmap.md)

## Incident and Recovery Writeups

- [Incidents and Fixes](docs/incidents.md)

## Service Deployment Writeups

- [Service Deployments](docs/service-deployments.md)

## Current Focus

- improving SG preboot/dropbear remote unlock reliability
- continuing NAS power-loss / boot behavior investigation
- keeping VPN behavior clearly documented
- cleaning up stale DHCP/DNS/network artifacts
- improving long-term maintainability and recovery documentation

## Why This Repo Exists

This repo serves two purposes:

1. personal infrastructure documentation for operating and maintaining the lab
2. a public showcase of practical networking, Linux administration, self-hosting, and troubleshooting work

## Notes

All public documentation is sanitized. Sensitive information such as credentials, private keys, tokens, and other secrets is excluded from this repository.
