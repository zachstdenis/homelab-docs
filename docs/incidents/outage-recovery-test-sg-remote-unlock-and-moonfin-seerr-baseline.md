# Outage Recovery Test, SG Remote Unlock, and Moonfin/Seerr Baseline

This page documents a homelab stabilization phase focused on validating the current baseline after recent changes, learning from a real outage recovery event, improving preboot remote unlock on the main server, and settling on practical expectations for Moonfin and Seerr behavior.

## Summary

The goal of this phase was to stop chasing side quests and instead confirm what was actually working, what had improved, and what remained the most important unsolved problems.

The work centered on:

- validating homelab behavior after a real outage
- reframing the main server issue as a preboot remote unlock problem rather than a simple boot failure
- cleaning up Dropbear and initramfs config drift on `seranogenomics`
- confirming the real active initramfs config paths and baked key state
- clarifying expected Moonfin and Seerr behavior for remote users
- capturing a cleaner baseline of the current homelab state and backlog

## Main Goal

Stabilize the homelab after recent changes by:

- validating outage recovery behavior
- improving understanding of SG preboot remote unlock
- confirming the new baseline for networking, services, and DNS
- deciding which remaining issues actually mattered most
- avoiding wasted time on low-value rabbit holes

## Environment

### Core Baseline
- main server: `seranogenomics` at `192.168.5.7`
- NAS: `nas` at `192.168.5.8`
- OPNsense: `texhnolyze.home.arpa`
  - LAN: `192.168.5.1/24`
  - WAN: `10.0.0.8/24`
- OpenVPN subnet: `192.168.6.0/24`
- Home Assistant VM: `192.168.5.14`
- gaming PC: `192.168.5.15`

### General State at Start
- old `192.168.7.x` and `192.168.8.x` treated as stale
- OPNsense, YAMS, Jellyfin, Seerr, Radarr/Sonarr, Bazarr, Recyclarr, Home Assistant, and UPS/NUT generally working
- Moonfin preferred over the stock Jellyfin Android TV app
- Watchtower assumed fixed and working again
- DNS path understood as:
  - clients -> AdGuard Home -> upstream
  - Unbound on `5353`

## Starting State

At the beginning of this phase, the homelab was mostly working from a day-to-day usability perspective, but some important recovery questions remained unresolved.

Key uncertainty areas included:

- whether the main server could be cleanly remotely unlocked after reboot or outage
- whether the NAS would recover predictably after power loss
- how much more time Moonfin and Seerr proxy/sync behavior was worth debugging
- whether qBittorrent's reported firewalled status represented a real problem

## Problems Addressed

This phase touched several related reliability and usability issues:

- outage recovery revealed that SG still stalled at encrypted preboot and needed manual unlock
- NAS still had unresolved power-loss recovery weirdness
- Dropbear and initramfs configuration on SG had drifted and become confusing
- old unlock workflow assumptions no longer matched the current network design
- Moonfin and Seerr proxy behavior was inconsistent for remote users
- Moonfin settings sync did not behave like a universal admin push
- qBittorrent showed a firewalled status
- domain ideas and hostname/DDNS possibilities were discussed, but only at a practical low-risk level

## Changes Made

## 1. Reframed the Main SG Problem

Instead of treating SG as simply "not booting after outage," the issue was reframed more accurately.

### Reframed Problem
The main problem was:

- preboot remote unlock networking and authentication were still unreliable

### Result
This made troubleshooting much more focused and realistic.

## 2. Confirmed SG Preboot Networking Details

Worked out the actual facts for SG's early-boot network path.

### Confirmed Facts
- active physical NIC for preboot networking: `enp3s0`
- live host IP remained on `br0`: `192.168.5.7/24`
- chosen preboot static IP: `192.168.5.17`

### Initramfs Early Network Config

```bash
DEVICE=enp3s0
IP=192.168.5.17::192.168.5.1:255.255.255.0:sg-preboot:enp3s0:none
```

### Result
Preboot networking became much more understandable and predictable.

## 3. Pinned the Required NIC Module into Initramfs

Ensured the expected NIC driver would be present during early boot.

### Pinned Module

```text
r8169
```

### Result
This improved confidence that early networking had the right driver available.

## 4. Found the Real Active Dropbear/Initramfs Paths

One of the biggest sources of confusion turned out to be editing the wrong path.

### Critical Discovery
The active install was actually using:

- `/etc/dropbear/initramfs/authorized_keys`
- `/etc/dropbear/initramfs/dropbear.conf`

It was **not** using the initially assumed:

- `/etc/dropbear-initramfs/...`

### Result
This explained why earlier edits did not seem to take effect.

## 5. Fixed Stale Baked Key and Dropbear Port Config

Multiple rebuilds eventually isolated stale baked auth and config problems.

### Key Fixes
- corrected the baked `authorized_keys`
- ensured effective Dropbear config was written to:
  - `/etc/dropbear/initramfs/dropbear.conf`
- confirmed:

```text
DROPBEAR_OPTIONS="-p 2222"
```

### Result
The initrd eventually contained the expected Windows unlock key and the intended port configuration.

## 6. Validated Baked Initrd State on SG

Confirmed what was actually baked into the initramfs rather than guessing.

### Earlier Evidence of Stale Key

```text
/tmp/initrd-check/main/root-.../.ssh/authorized_keys
ssh-ed25519 ... zach@gaming-pc
```

### Later Corrected Baked Key

```text
/tmp/initrd-check/main/root-.../.ssh/authorized_keys
ssh-ed25519 ... zach-gamingpc-sg-unlock
```

### Final Baked Validation

```text
authorized_keys inside initrd:
ssh-ed25519 ... zach-gamingpc-sg-unlock

dropbear.conf inside initrd:
DROPBEAR_OPTIONS="-p 2222"
```

### Result
The baked initrd state was finally aligned with the intended remote unlock design.

## 7. Validated Moonfin and Seerr Reality

Tested assumptions around Moonfin and Seerr behavior and stopped expecting them to behave like a perfect centralized sync model.

### Observations
- both `jellyfin` and `jellyseerr` were confirmed on `yams_network`
- using the Docker service name (`http://jellyseerr:5055`) for Moonfin did not materially improve behavior
- Moonfin settings sync behaved more like a per-user default than a global enforced admin push
- remote users on different Jellyfin accounts still often needed manual proxy disconnect and login steps

### Practical Conclusion
- keep the Tailscale Seerr URL as the default for most users/devices
- accept that remote users may still need to disconnect proxy and log in once manually
- stop treating Moonfin sync as a true globally authoritative push model

## 8. Explained qBittorrent Firewalled Status

The reported qBittorrent state was checked in context.

### Conclusion
The "firewalled" state is expected with Mullvad because Mullvad no longer supports port forwarding.

### Result
This was treated as an expected provider limitation, not a new homelab breakage.

## What Worked

The following improvements or conclusions were successful:

- UPS/NUT shutdown behavior worked correctly during a real outage
- router recovered successfully after power restoration
- SG auto-power-on after outage appeared to work
- SG preboot networking improved enough that `192.168.5.17` became reachable during initramfs
- the correct Windows key was eventually verified inside the baked initrd
- the intended Dropbear port `2222` was eventually verified inside the baked initrd
- qBittorrent firewalled status was correctly explained rather than over-troubleshot
- Moonfin and Seerr expectations became more realistic and operationally useful
- Tailscale Seerr URL remained the practical default for remote clients
- Moonfin settings behavior was better understood as per-user rather than globally authoritative

## What Broke or Regressed

A few important weaknesses were confirmed rather than resolved:

### SG Remote Unlock Still Not Fully Proven
A real outage confirmed that SG remote unlock was still not dependable enough end-to-end, even though the preboot networking situation improved substantially.

### NAS Still Has Boot-Recovery Weirdness
The NAS still failed to boot cleanly after power restoration and required PSU drain / rear switch power cycle behavior.

### Dropbear Troubleshooting Was Messy
The investigation was slowed by:

- stale key sources
- conflicting config paths
- outdated assumptions about port and IP
- old unlock workflow assumptions that no longer matched current reality

### Moonfin / Seerr Proxy Mode Still Not Clean
Proxy mode still did not reliably populate content for remote users in the way originally hoped.

### Time Was Lost on Low-Value Behavior
Moonfin proxy and sync debugging continued longer than it was worth, given the ultimately user/session-specific nature of the issue.

## Validation

### SG NIC and Routing Checks

```bash
ip -br link
ip -br addr
ip route
sudo lspci -nnk | grep -A3 -Ei 'ethernet|network'
```

### Successful Preboot Reachability

```text
Reply from 192.168.5.17: bytes=32 ...
```

This confirmed that SG preboot networking had at least become reachable.

### Earlier Authentication Failure

```text
root@192.168.5.17: Permission denied (publickey).
```

This helped confirm that early failures were authentication-related rather than pure connectivity failure.

### Docker Network Confirmation for Jellyfin / Jellyseerr

```bash
docker ps --format 'table {{.Names}}\t{{.Image}}\t{{.Networks}}\t{{.Ports}}' | egrep 'jellyfin|seerr|jellyseerr'
```

Observed result pattern:

```text
jellyfin       ... yams_network ... 8096
jellyseerr     ... yams_network ... 5055
```

### Power Outage Findings
The outage test established that:

- UPS/NUT shutdown worked
- router recovered
- SG powered on but still required manual dm-crypt unlock
- NAS needed PSU drain / rear switch power cycle before booting

## Ending State

By the end of this phase:

### Core Homelab
- the overall homelab baseline remained largely healthy
- core services and network design were still in a good place overall
- the main weak points were now much more clearly identified

### SG
- preboot IP was now predictable: `192.168.5.17`
- correct key was baked into initramfs
- correct Dropbear port config was baked into initramfs
- remote unlock was much closer to working cleanly
- end-to-end remote unlock still was not considered fully solved or proven

### NAS
- hardware and firmware recovery behavior after power loss remained unresolved
- NAS power-loss boot issue stayed open

### Moonfin / Seerr
- use Tailscale Seerr URL as the default for most users and devices
- accept one-time per-user manual login/disconnect workflow as the practical model
- no further debugging of Moonfin proxy or sync behavior was considered worth the time for now

## Lessons Learned

- a real outage is often the fastest way to reveal what still is not truly resilient
- preboot networking problems and post-boot service problems are different classes of failure
- if initramfs troubleshooting feels inconsistent, verify the actual active paths instead of assuming package defaults
- stale baked keys can make authentication troubleshooting look much more mysterious than it really is
- a predictable preboot IP is a major step forward even if the unlock workflow is not yet fully complete
- not every awkward app-integration behavior is worth solving perfectly if a practical operating model already exists
- a provider-side limitation like Mullvad dropping port forwarding should not be mistaken for a local infrastructure regression

## Follow-Up Items

- revisit SG preboot and Dropbear remote unlock later and finish proving it works cleanly after reboot or outage
- investigate NAS power-loss boot problem, likely starting with CMOS battery, BIOS persistence, or old hardware behavior
- update old Windows unlock scripts if remote unlock work resumes:
  - preboot IP: `192.168.5.17`
  - preboot port: `2222`
  - normal SG IP: `192.168.5.7`
- keep Tailscale Seerr URL as default for most users and devices
- accept one-time per-user Seerr login in Moonfin as the current operating model
- revisit monthly maintenance cadence later
- return to backlog items such as VPN client internet behavior, WireGuard migration, and other deferred infrastructure work later

## Commands Worth Preserving

### SG NIC / Network Facts

```bash
ip -br link
ip -br addr
ip route
sudo lspci -nnk | grep -A3 -Ei 'ethernet|network'
```

### Relevant SG NICs

```text
enp3s0 = active physical uplink used for preboot networking
live host IP is on br0 = 192.168.5.7/24
preboot static IP chosen: 192.168.5.17
```

### Initramfs Early Network Config

```bash
DEVICE=enp3s0
IP=192.168.5.17::192.168.5.1:255.255.255.0:sg-preboot:enp3s0:none
```

### Pinned Module

```text
r8169
```

### Actual Effective Dropbear Paths

```text
/etc/dropbear/initramfs/authorized_keys
/etc/dropbear/initramfs/dropbear.conf
```

### Final Baked Initrd Validation

```text
authorized_keys inside initrd:
ssh-ed25519 ... zach-gamingpc-sg-unlock

dropbear.conf inside initrd:
DROPBEAR_OPTIONS="-p 2222"
```

### Docker Network Check for Jellyfin / Jellyseerr

```bash
docker ps --format 'table {{.Names}}\t{{.Image}}\t{{.Networks}}\t{{.Ports}}' | egrep 'jellyfin|seerr|jellyseerr'
```

### Important Operational Conclusions

```text
- qBittorrent firewalled state is expected with Mullvad because Mullvad no longer supports port forwarding
- synced/default Seerr URL can be Tailscale-based
- Moonfin settings sync is effectively per-user, not a universal global admin push
- remote users may still need one-time manual login after disconnecting proxy
```

## Status

Completed as a stabilization and baseline-clarification phase, with SG remote unlock and NAS post-power-loss recovery still remaining as the two main unresolved issues.
