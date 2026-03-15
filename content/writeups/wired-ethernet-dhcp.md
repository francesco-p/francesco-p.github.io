---
title: "Fixing a Wired Ethernet Connection Stuck on Link-Local (169.254.x.x)"
date: 2026-02-25
summary: "Quick fix for Ubuntu Ethernet stuck on 169.254.x.x due to link-local-only configuration."
tags: ["linux", "ubuntu", "networking", "dhcp", "nmcli", "writeup"]
---

## Problem

`eno1` had `169.254.x.x`, no gateway, and no internet access.

## Resolution

1. Optional: disable Wi-Fi while testing.

```bash
nmcli radio wifi off
```

2. Switch the Ethernet profile to DHCP:

```bash
sudo nmcli connection modify netplan-eno1 ipv4.method auto
sudo nmcli connection modify netplan-eno1 ipv4.never-default no
sudo nmcli connection modify netplan-eno1 autoconnect yes
```

3. Restart the connection:

```bash
sudo nmcli connection down netplan-eno1
sudo nmcli connection up netplan-eno1
```

4. Confirm `eno1` now has a router-range IP:

```bash
ip a | grep eno1 -A3
```

5. Optional: make future Ethernet connections default to DHCP:

```bash
sudo nmcli connection add type ethernet ifname "*" con-name "auto-eth" ipv4.method auto autoconnect yes
```

6. Quick connectivity check:

```bash
ping -c 3 8.8.8.8    # internet reachability
ping -c 3 google.com # DNS resolution
```

## Outcome

Wired networking works again, and `169.254.x.x` fallback is avoided.
