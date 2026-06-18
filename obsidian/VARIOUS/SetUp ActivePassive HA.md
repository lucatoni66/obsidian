---
title: "SetUp Active/Passive HA"
source: "https://docs.paloaltonetworks.com/ngfw/administration/high-availability/set-up-activepassive-ha"
author:
published:
created: 2026-04-23
description: "Ensure both firewalls have the same model, PAN-OS version, multi virtual system        capability, and type of interfaces."
tags:
  - "clippings palo alto"
---
Next-Generation Firewall

Table of Contents

Ensure both firewalls have the same model, PAN-OS version, multi virtual system capability, and type of interfaces.

| Where Can I Use This? | What Do I Need? |
| --- | --- |
| - NGFW | For Strata Cloud Manager managed NGFWs:  - Strata Cloud Manager Pro - Strata Cloud Manager Essentials |

To set up high availability on your Palo Alto Networks firewalls, you need a pair of firewalls that meet the following requirements:

- **The same model** —Both the firewalls in the pair must be of the same hardware model or virtual machine model. (Verify that by viewing Dashboard, General Information, Model.)
- **The same PAN-OS version** —Both the firewalls should be running the same PAN-OS version and must each be up-to-date on the application, URL, and threat databases. (Verify that by viewing Dashboard, General Information, Software Version.)
- The same multi virtual system capability—Both firewalls must have Multi Virtual System Capability either enabled or not enabled. When enabled, each firewall requires its own multiple virtual systems licenses. (Verify that by viewing Device > Setup > Management, General Settings, Multi Virtual System Capability enabled or disabled.)
	**(Cloud Managed NGFWs Only)** —Both firewalls must have the multi-vsys capability disabled.
- **The same type of interfaces** —Dedicated HA links, or a combination of the management port and in-band ports that are set to [interface type](#) HA. (Verify the following on Device > High Availability > HA Communications.)
	- **(Cloud Managed NGFWs Only)** —Strata Cloud Manager supports IPv4 addresses only.
		- Determine the IP address for the HA1 (control) connection between the HA peers. The HA1 IP address for both peers must be on the same subnet if they are directly connected or are connected to the same switch.
		For firewalls without dedicated HA ports, you can use the management port for the control connection. Using the management port provides a direct communication link between the management planes on both firewalls. However, because the management ports will not be directly cabled between the peers, make sure that you have a route that connects these two interfaces across your network.
		- If you use Layer 3 as the transport method for the HA2 (data) connection, determine the IP address for the HA2 link. Use Layer 3 only if the HA2 connection must communicate over a routed network. The IP subnet for the HA2 links must not overlap with that of the HA1 links or with any other subnet assigned to the data ports on the firewall.
- **The same set of licenses** —Licenses are unique to each firewall and cannot be shared between the firewalls. Therefore, you must license both firewalls identically. If both firewalls do not have an identical set of licenses, they cannot synchronize configuration information and maintain parity for a seamless failover. (Verify that the licenses match by comparing Device > Licenses.)
	As a best practice, if you have an existing firewall and you want to add a new firewall for HA purposes and the new firewall has an existing configuration [Reset the Firewall to Factory Default Settings](https://docs.paloaltonetworks.com/content/techdocs/en_US/ngfw/administration/firewall-administration/reset-the-firewall-to-factory-default-settings.html#ide7aa6f33-1993-4057-86cf-c50105678cb8) on the new firewall. This ensures that the new firewall has a clean configuration. After HA is configured, you will then sync the configuration on the primary firewall to the newly introduced firewall with the clean configuration.
- **(Cloud Managed NGFWs Only)** —Both firewalls in the HA pair must be added to the same folder.
	Firewalls in an HA pair cannot be moved to a new folder. To move them, you must first break the HA configuration, move both firewalls to the new folder, and then reconfigure HA

### LACP and LLDP Pre-Negotiation for Active/Passive HA

If a firewall uses LACP or LLDP, negotiation of those protocols upon failover prevents sub-second failover. However, you can enable an interface on a passive firewall to negotiate LACP and LLDP prior to failover. Thus, a firewall in [Passive](https://docs.paloaltonetworks.com/content/techdocs/en_US/ngfw/administration/high-availability/ha-firewall-states.html#id2ee5562a-9bc8-42b6-9c77-2cc496a112fc_id6f374cb0-ff4b-4693-87dd-c3b83a4e43fd) or [Non-functional](https://docs.paloaltonetworks.com/content/techdocs/en_US/ngfw/administration/high-availability/ha-firewall-states.html#id2ee5562a-9bc8-42b6-9c77-2cc496a112fc_idcab71177-4774-4c00-817d-dcc267e7d88a) HA state can communicate with neighboring devices using LACP or LLDP. Such pre-negotiation speeds up failover.

All firewall models except VM-Series firewalls support a pre-negotiation configuration, which depends on whether the Ethernet or AE interface is in a Layer 2, Layer 3, or virtual wire deployment. An HA passive firewall handles LACP and LLDP packets in one of two ways:

- **Active** —The firewall has LACP or LLDP configured on the interface and actively participates in LACP or LLDP pre-negotiation, respectively.
- **Passive** —LACP or LLDP is not configured on the interface and the firewall does not participate in the protocol, but allows the peers on either side of the firewall to pre-negotiate LACP or LLDP, respectively.

The following table displays which deployments are supported on Aggregate Ethernet (AE) and Ethernet interfaces.

| Interface Deployment | AE Interface | Ethernet Interface |
| --- | --- | --- |
| LACP in Layer 2 | Active | Not supported |
| LACP in Layer 3 | Active | Not supported |
| LACP in Virtual Wire | Not supported | Passive |
| LLDP in Layer 2 | Active | Active |
| LLDP in Layer 3 | Active | Active |
| LLDP in Virtual Wire | Active | - Active if LLDP itself is configured. - Passive if LLDP itself is not configured. |

Pre-negotiation is not supported on subinterfaces or tunnel interfaces.

To configure LACP or LLDP pre-negotiation, see the step [(Optional) Enable LACP and LLDP Pre-Negotiation for Active/Passive HA for faster failover in your network uses LACP or LLAP](https://docs.paloaltonetworks.com/content/techdocs/en_US/ngfw/administration/high-availability/set-up-activepassive-ha/configure-activepassive-ha/configure-activepassive-ha-pan-os.html#configure-activepassive-ha-pan-os_ida427d371-2878-49b1-9f61-d89517833033).