---
title: "Configure Virtual Switch with an EtherChannel (or port channel)"
source: "https://knowledge.broadcom.com/external/article?legacyId=1004048"
author:
  - "[[Broadcom Inc]]"
published: 2026-09-23
created: 2026-11-02
description: "Configure Virtual Switch with an EtherChannel (or port channel)"
tags:
  - "clippings vmware vsan"
---
Configure Virtual Switch with an EtherChannel (or port channel)<iframe title="https://cdn.wolkenservicedesk.com/Broadcom_%20Header.html" src="https://cdn.wolkenservicedesk.com/Broadcom_%20Header.html" height="90px"></iframe>

search cancel

book

#### Article ID: 321425

calendar\_today

#### Updated On: 09-23-2026

#### Products

VMware vCenter Server VMware vSphere ESXi

#### Issue/Introduction

This article provides configuration information when using an EtherChannel (or port channel) for connectivity between ESXi hosts and Physical Network Switches on a vSphere Standard Switch (vSS) or vSphere Distributed Switch (vDS).  

#### Environment

VMware vSphere ESXi  
VMware vCenter Server

#### Resolution

### Standard Switch Configuration:

To configure the ESXi portion of a port channel, the aggregated NICs need to be placed into a load balancing configuration of " **Route based on IP hash** ". This can be done at a virtual switch level or at an individual portgroup level, however the only use case for modifying only certain portgroups would be if both standalone and port channeled NICs are on the same vSwitch, which is an unsupported configuration: [Route Based on IP Hash](https://techdocs.broadcom.com/us/en/vmware-cis/vsphere/vsphere/7-0/vsphere-networking-7-0/networking-policies/teaming-and-failover-policy/load-balancing-algorithms-for-virtual-switches/route-based-on-ip-hash.html)

1. Navigate to the host with the standard switch to modify, then select Virtual Switches from Configure -> Networking
	- ![](https://api-broadcomcms-software.wolkenservicedesk.com/es/attachments/get_attachment_content?uniqueFileId=1793651481105)
2. Before changing the setting at the virtual switch, ensure that every portgroup that will be using the port channel is set to inherit the configuration from the switch, and any portgroup that will not be using the port channel is set to Override
	1. Click the three dots next to each portgroup name, e.g. "VM Network", and select "Edit Settings"
		2. Under "Teaming and Failover", if the portgroup is expected to use the port channel confirm the "Override" checkbox is not selected:  
		- ![](https://api-broadcomcms-software.wolkenservicedesk.com/es/attachments/get_attachment_content?uniqueFileId=1793655705203)
		3. Repeat for the remaining portgroups
		- Note: The importance of verifying that the vSwitch setting is not being overridden is also to avoid a bug impacting certain ESXi versions where, if "Override" is selected then the load balancing policy may change on boot, even if the portgroup is still set to the same configuration as the vSwitch. See [vSwitch (Standard Switches) teaming Load balancing change to port id after ESXi 7.0.3l update](https://knowledge.broadcom.com/external/article/318490)
3. Once all portgroups have been validated, click "Edit" next to the virtual Standard Switch that is being modified
4. Under "Teaming and Failover", change the **Load Balancing** configuration to " **Route Based on IP Hash** "
5. Ensure all of the adapters are listed as **Active** and are the physical NICs corresponding to the physical port channel configuration
	- **Note:** All NICs on the vSwitch at this point should be configured to be in a port channel on the physical switch, and there should be no NICs in **Standby** or **Unused**
6. Click "OK" to complete the change

### Virtual Distributed Switch Configuration:

*Note: If the physical network is configured with an EtherChannel (port channel) do not create a LAG within the virtual Distributed Switch. A LAG is only needed when the physical network is using LACP. Please see* [Configuring LACP on a vSphere Distributed Switch Port Group](https://knowledge.broadcom.com/external/article/312554/configuring-lacp-on-a-vsphere-distribute.html)

1. Click on the Network symbol within vSphere
2. The portgroups on the virtual Distributed Switch need to be changed to have the **Load Balancing Policy** set to **Route based on** **IP hash**
	- To set this for all portgroups on a vDS, right click the virtual Distributed Switch and select the option **Manage Distributed Port Groups  
		![](https://api-broadcomcms-software.wolkenservicedesk.com/es/attachments/get_attachment_content?uniqueFileId=1848924606558)  
		**
3. Select the check box next to **Teaming and Failover** and click **Next**
4. Select the check box next to the left of the **Name** column to select all portgroups on the virtual switch and click **Next**
5. Click the drop down for the load balancing policy and select **Route based on IP hash** (red box in the example below) and click **Next**
	- **Note:** When using **Route based on IP Hash** all uplinks on the virtual switch must be in the **Active uplinks** state (blue box in the example below)  
		![](https://api-broadcomcms-software.wolkenservicedesk.com/es/attachments/get_attachment_content?uniqueFileId=1793612425972)
6. Click **Finish** to end the Wizard and apply the change to all selected portgroups.

#### Additional Information

For more information about EtherChannel (port channel) configurations and other link aggregation concepts please reference the articles below:

- **EtherChannel**: This is a link aggregation (port trunking) method used to provide fault-tolerance and high-speed links between switches, routers, and servers by grouping two to eight physical Ethernet links to create a logical Ethernet link with additional failover links. For additional information on Cisco EtherChannel, see the [EtherChannel Introduction](http://www.cisco.com/en/US/tech/tk389/tk213/tsd_technology_support_protocol_home.html) by Cisco.
- **LACP or IEEE 802.3ad**: The Link Aggregation Control Protocol (LACP) is included in IEEE specification as a method to control the bundling of several physical ports together to form a single logical channel. LACP allows a network device to negotiate an automatic bundling of links by sending LACP packets to the peer (directly connected device that also implements LACP).
	- See [Configuring a LAG on a vSphere Distributed Switch Port Group when using LACP](https://knowledge.broadcom.com/external/article/312554) for VMware by Broadcom configuration assistance specifically for LACP
		- See [Converting to Enhanced LACP Support on a vSphere Distributed Switch- "Source vCenter Server has instance(s) of Distributed Virtual Switch at unsupported lacpApiVersion"](https://knowledge.broadcom.com/external/article/318706/converting-to-enhanced-lacp-support-on-a.html)
- **EtherChannel vs. 802.3ad**: EtherChannel and IEEE 802.3ad standards are very similar and accomplish the same goal. There are a few differences between the two, other than EtherChannel is Cisco proprietary and 802.3ad is an open standard.
	- For more information on EtherChannel implementation, see the [Understanding EtherChannel Load Balancing and Redundancy on Catalyst Switches](http://www.cisco.com/en/US/tech/tk389/tk213/technologies_tech_note09186a0080094714.shtml) article from Cisco.

#### Feedback

Was this article helpful?

thumb\_up Yes

thumb\_down No

Powered by

<iframe frameborder="0" title="https://cdn.wolkenservicedesk.com/Broadcom_%20Footer.html" src="https://cdn.wolkenservicedesk.com/Broadcom_%20Footer.html" height="118px"></iframe>