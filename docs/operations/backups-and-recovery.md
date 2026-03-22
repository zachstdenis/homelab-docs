# Backups and Recovery

This page documents the current backup and recovery posture of the homelab, including what is important to protect, what recovery concerns exist, and what still needs improvement.

## Purpose

The goal of backups and recovery planning is to reduce the impact of:
- hardware failure
- power events
- bad updates
- configuration mistakes
- accidental data loss

This page is not meant to imply that every system already has a perfect backup strategy. It exists to document the current state and identify where recovery risk still exists.

## Recovery Priorities

The most important things to recover in this homelab are:

### 1. Network Access
If the network is not functioning, everything else becomes harder to diagnose or recover.

**Critical components:**
- OPNsense
- DNS path
- VPN access where relevant

### 2. Core Compute
The main server is central to a large portion of the lab.

**Critical component:**
- `seranogenomics`

### 3. Core Storage
The NAS is a key storage dependency for the environment.

**Critical component:**
- `nas`

### 4. Core Services
Important services should be recoverable even if they are not equally critical.

**Examples:**
- AdGuard Home
- Unbound
- Dockerized application stack
- Home Assistant

## Current Recovery Concerns

### Main Server Remote Unlock
The main server can stall during initramfs/dropbear remote unlock, which makes reboot recovery less predictable than it should be.

### NAS Power-Loss / Boot Behavior
The NAS has shown unreliable behavior after power loss and may require full power drain/manual intervention before booting normally again.

### Service Dependency Visibility
Some services may technically be recoverable, but the exact dependency chain is not yet fully documented.

## What Should Be Protected

Over time, the following should be clearly backed up or exportable:

### Configuration
- OPNsense configuration
- DNS-related configuration
- Docker Compose files / service definitions
- important host configuration notes
- VM/service-specific configuration where relevant

### Data
- NAS-hosted important data
- Home Assistant configuration/state as appropriate
- critical application data for self-hosted services

### Documentation
- runbooks
- addressing notes
- service inventory
- troubleshooting writeups
- recovery procedures

## Recovery Principles

The homelab is being documented with a few recovery principles in mind:

- prefer recoverable systems over fragile ones
- document weak points honestly
- make manual recovery steps explicit
- reduce reliance on memory
- improve startup/recovery reliability before adding complexity

## Current Practical Recovery Approach

At a high level, recovery currently depends on:

- OPNsense returning cleanly
- DNS functioning correctly
- `seranogenomics` completing boot successfully
- `nas` recovering normally after restart/power event
- validating key services after infrastructure recovery

## Validation After a Recovery Event

After a power event, restart, or major change, validate:

### Network
- router/firewall is reachable
- internet connectivity works
- LAN clients have access
- VPN behaves as expected

### DNS
- AdGuard Home is functioning
- Unbound is functioning
- clients can resolve expected domains

### Core Hosts
- `seranogenomics` is online
- `nas` is online
- Home Assistant is reachable

### Services
- critical Dockerized services are reachable
- media stack behaves normally
- expected internal services are available

## Improvement Areas

This area still needs more work. High-value future improvements include:

- clearer documentation of what is backed up and how
- export/backup notes for important configs
- better runbooks for partial failure scenarios
- improved NAS recovery confidence
- improved remote unlock reliability on the main server

## Status

Recovery documentation is in progress and will improve as the lab becomes more standardized.
