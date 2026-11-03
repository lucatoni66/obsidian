---
title: Updating Cisco Nexus 9K firmware via SSH
source: https://claude.ai/chat/a58e0c77-ee04-4cc7-985d-778bbb434965
author:
published: 2026-10-21
created: 2026-10-21
description: Claude conversation with 1 messages
tags:
  - clippings cisco
  - nexus
---
## Upgrading Cisco Nexus 9000 NX-OS over SSH

### 1\. Plan before you touch the switch

- Download the correct image from Cisco Software Download. Use the 64-bit image (`nxos64-cs.<version>.bin`) for most 9K models. Older 7.x images are `nxos.<version>.bin`.
- Read the release notes and the **Cisco Nexus 9000 Upgrade/Downgrade Matrix** to confirm your supported upgrade path. You may need an intermediate hop.
- Confirm the image supports your hardware (9300, 9500, -EX/-FX/-GX, etc.).
- Schedule a maintenance window. A non-ISSU upgrade reloads the switch.

### 2\. Connect and capture the current state



```
ssh admin@<switch-ip>
show version
show module
show running-config > bootflash:pre-upgrade-config.txt
copy running-config startup-config
copy running-config scp://user@backup-host/n9k-pre-upgrade.cfg vrf management
```

### 3\. Check free space



```
dir bootflash:
```

You need room for the new image (often 1.5 GB or more). Delete old images if needed:



```
delete bootflash:nxos64-cs.<old-version>.bin
```

### 4\. Copy the image to the switch

Over SCP (or SFTP/TFTP), using the management VRF:



```
copy scp://user@<server-ip>/nxos64-cs.10.x.x.bin bootflash: vrf management
```

Optionally use the compact option to shrink the image during transfer (SCP/SFTP only, on supported platforms):



```
copy scp://user@<server-ip>/nxos64-cs.10.x.x.bin bootflash: compact vrf management
```

### 5\. Verify the image integrity



```
show file bootflash:nxos64-cs.10.x.x.bin sha512sum
```

Compare the result against the checksum on Cisco’s download page. Do not continue if it doesn’t match.

### 6\. Run the impact and compatibility checks



```
show incompatibility-all nxos bootflash:nxos64-cs.10.x.x.bin
show install all impact nxos bootflash:nxos64-cs.10.x.x.bin
```

This shows whether the upgrade is disruptive, whether it needs a reload, and any feature or config incompatibilities. Fix anything flagged first.

### 7\. Perform the upgrade



```
install all nxos bootflash:nxos64-cs.10.x.x.bin
```

The switch runs its checks, asks you to confirm, installs the image, and reloads. Your SSH session will drop. Wait for the reload, which usually takes 5–15 minutes, then reconnect.

For a supported non-disruptive ISSU on a dual-supervisor chassis, the same command applies, but check the impact output and release notes first.

### 8\. Verify after the reload



```
show version
show install all status
show module
show interface status
show running-config diff
show ip route summary
```

Confirm the version is correct, all modules are `ok`, and your interfaces, routing and vPC peers are back to normal. For vPC pairs, upgrade one peer at a time and confirm the first is healthy before starting the second.

### 9\. Clean up and save



```
copy running-config startup-config
delete bootflash:<old-image>.bin
```

### Extras

- **EPLD upgrades:** some releases need separate firmware for line cards or FPGAs, done with `install epld bootflash:<epld-image> module <n>`. Check the release notes.
- **Rollback:** keep the old image on bootflash until you’re sure the upgrade is stable. You can boot it again with `install all nxos bootflash:<old-image>.bin`.
- **Out-of-band access:** have console access ready in case the switch doesn’t come back on SSH.

If you tell me your exact model and current and target NX-OS versions, I can help check the upgrade path and any version-specific caveats.

48 minutes ago



Write a message…

  









[Claude is AI and can make mistakes. Please double-check responses.](https://support.anthropic.com/en/articles/8525154-claude-is-providing-incorrect-or-misleading-responses-what-s-going-on)

Sonnet 5.5 Medium