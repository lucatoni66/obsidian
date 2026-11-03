---
title: New chat
source: https://gemini.google.com/app/468efd43c24e3741
author:
published:
created: 2026-09-23
description: Gemini conversation with 2 messages
tags:
  - clippings
  - cisco vpc
---
**You**

i have 2 cisco nexus 9k switches and 2 rhel servers with multiple cards; create cisco switch configuration for connecting them and use vpc to connect servers to both switches to achieve redundancy; provide configuraion for switch devices and rhel servers

---

**Gemini**

To configure high availability using Cisco Virtual Port Channel (vPC) across two Nexus 9000 switches and bind multiple network interfaces on Red Hat Enterprise Linux (RHEL) servers using LACP (IEEE 802.3ad), follow the reference designs below.

## 1\. Cisco Nexus 9,000 Switch Configuration

### Switch 1 (Primary vPC Peer)

```
feature lacp
feature vpc

vpc domain 1
  peer-keepalive destination 192.168.100.2 source 192.168.100.1 vrf management
  peer-gateway
  role priority 100

! vPC Peer-Link (Connected between Switch 1 and Switch 2)
interface Ethernet1/1
  switchport mode trunk
  channel-group 100 mode active

interface Ethernet1/2
  switchport mode trunk
  channel-group 100 mode active

interface port-channel100
  switchport mode trunk
  vpc peer-link

! Downstream Connection to Server 1 (Link 1 on Eth1/3)
interface Ethernet1/3
  switchport mode access
  switchport access vlan 10
  channel-group 10 mode active
  lacp fast-rate

interface port-channel10
  switchport mode access
  switchport access vlan 10
  vpc 10

! Downstream Connection to Server 2 (Link 1 on Eth1/4)
interface Ethernet1/4
  switchport mode access
  switchport access vlan 10
  channel-group 20 mode active
  lacp fast-rate

interface port-channel20
  switchport mode access
  switchport access vlan 10
  vpc 20
```

### Switch 2 (Secondary vPC Peer)

```
feature lacp
feature vpc

vpc domain 1
  peer-keepalive destination 192.168.100.1 source 192.168.100.2 vrf management
  peer-gateway
  role priority 200

! vPC Peer-Link
interface Ethernet1/1
  switchport mode trunk
  channel-group 100 mode active

interface Ethernet1/2
  switchport mode trunk
  channel-group 100 mode active

interface port-channel100
  switchport mode trunk
  vpc peer-link

! Downstream Connection to Server 1 (Link 2 on Eth1/3)
interface Ethernet1/3
  switchport mode access
  switchport access vlan 10
  channel-group 10 mode active
  lacp fast-rate

interface port-channel10
  switchport mode access
  switchport access vlan 10
  vpc 10

! Downstream Connection to Server 2 (Link 2 on Eth1/4)
interface Ethernet1/4
  switchport mode access
  switchport access vlan 10
  channel-group 20 mode active
  lacp fast-rate

interface port-channel20
  switchport mode access
  switchport access vlan 10
  vpc 20
```

## 2\. RHEL Server Network Bonding Configuration

On each RHEL server, assume physical NICs **`eth0`** and **`eth1`** are connected respectively to Switch 1 and Switch 2. Run the following NetworkManager (`nmcli`) commands to configure an 802.3ad LACP bond (`bond0`):

```
# 1. Create the bond interface with LACP (mode 4) and fast rate
sudo nmcli con add type bond con-name bond0 ifname bond0 bond.options "mode=802.3ad,miimon=100,lacp_rate=fast,xmit_hash_policy=layer2+3"

# 2. Add physical network interfaces as slaves to bond0
sudo nmcli con add type ethernet slave-type bond con-name bond0-slave-eth0 ifname eth0 master bond0
sudo nmcli con add type ethernet slave-type bond con-name bond0-slave-eth1 ifname eth1 master bond0

# 3. Configure static IP address settings on the bond interface (modify as needed)
sudo nmcli con modify bond0 ipv4.method manual ipv4.addresses 192.168.10.50/24 ipv4.gateway 192.168.10.1 ipv4.dns "8.8.8.8"

# 4. Bring up the bond and slave connections
sudo nmcli con up bond0
```

## Verification Steps

- **Verify Switch vPC Status:** Run `show vpc brief` on both switches to confirm the vPC peer-link and individual server port channels (`po10`, `po20`) are operational.
- **Verify Linux Bond Status:** Run `cat /proc/net/bonding/bond0` on the RHEL servers to check that the bonding mode is `IEEE 802.3ad` and that all member links show an `Up` status.

For a visual guide on setting up equivalent switch port channels, check out this video on

[Configuring Cisco Nexus Port Channels with LACP](https://www.youtube.com/watch?v=cCBftR8sgDE&utm_source=gemini)

.