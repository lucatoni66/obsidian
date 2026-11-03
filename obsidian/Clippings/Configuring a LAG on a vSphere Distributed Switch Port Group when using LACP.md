---
title: "Configuring a LAG on a vSphere Distributed Switch Port Group when using LACP"
source: "https://knowledge.broadcom.com/external/article/312554/configuring-lacp-on-a-vsphere-distribute.html"
author:
  - "[[Broadcom Inc]]"
published: 2026-08-13
created: 2026-11-02
description: "Configuring a LAG on a vSphere Distributed Switch Port Group when using LACP"
tags:
  - "clippings vmware vsan"
---
Configuring a LAG on a vSphere Distributed Switch Port Group when using LACP<iframe title="https://cdn.wolkenservicedesk.com/Broadcom_%20Header.html" src="https://cdn.wolkenservicedesk.com/Broadcom_%20Header.html" height="90px"></iframe>

search cancel

book

#### Article ID: 312554

calendar\_today

#### Updated On: 08-13-2026

#### Products

VMware vCenter Server VMware vSphere ESXi

#### Issue/Introduction

This article provides information on configuring a LAG within the vSphere Distributed Switch while using Link Aggregation Control Protocol (LACP) on the physical switch.

#### Environment

VMware vCenter Server

VMware vSphere ESXi

#### Resolution

**Before Configuration Occurs:**

- If the environment has multiple NICs that will be used in an LACP configuration, VMware by Broadcom recommends first creating the LAG on the Distributed Switch. After that, coordinate with the hardware vendor, configuring LACP on the physical switch to migrate the NICs gradually, moving a few at a time. This helps prevent total connectivity loss. If all NICs are placed into the LACP configuration on the physical switch before configuring the Distributed Switch, connectivity will be lost, and the LACP configuration will need to be reverted to restore network access.  
	- If connectivity is lost and the vCenter VM is not on an ephemeral portgroup on the Distributed Switch, see [vCenter network connectivity lost - Recover vCenter network when connected to a Distributed Switch](https://knowledge.broadcom.com/external/article/318719/vcenter-network-connectivity-lost-recov.html) and [Static (non-ephemeral) or ephemeral port binding on a vSphere Distributed Switch](https://knowledge.broadcom.com/external/article/324492/static-nonephemeral-or-ephemeral-port-bi.html)
- For Support Case situations, when LACP is configured, any network troubleshooting will typically require not only support from VMware by Broadcom, but also collaboration on the same call with the customer personnel who manages and can access and modify physical switch configurations. Because both these groups are typically in high demand, this tends to elongate the length of time for support calls considerably.

## Configuring LACP on an Uplink Port Group

**An abbreviated summary of this process can be accessed by clicking the Migrating Network Traffic to LAGs button in the LACP configuration panel.**

![](https://api-broadcomcms-software.wolkenservicedesk.com/es/attachments/get_attachment_content?uniqueFileId=2124550383992)

**General Procedure:**

1. Create and configure a Link Aggregation Group (LAG).
2. Connect the physical adapters to LAG that just got created
3. Configure teaming and failover specifically for LACP.
4. Activate and Migrate LAG to desired host(s).

**Create the LAG:**

1. Open the vSphere Web client.
2. Click the **Networking** icon:
3. Expand the Datacenter and select the Distributed Switch that the LAG is being added to.
4. Click on the **Configuration** tab and select **LACP** click **NEW**.
5. Begin Configuring the LAG
	1. Name the LAG
		2. Number of ports (set to the number of NICs that are going to be in the LAG)
		3. Mode
		1. Either the Distributed Switch LAG or the physical switch (or both) need to be set to **Active**
		4. Load balancing mode will need to be set to match the physical switches' configuration for the LACP
		5. Timeout (configure this setting based on your physical switch vendor recommendation)
		1. Slow: will send out a heartbeat packet every 30 seconds
				2. Fast: will send out a heartbeat packet every second
		6. VLAN trunk range (configure this setting based on your physical switch vendor recommendation)
		7. NetFlow (if necessary, ensure NetFlow is also enabled on the specific Distributed Switch portgroups that are to be monitored)

**Configure teaming and failover for LACP:**

You will need to edit the teaming and failover policies for all port groups that will be connecting via the LAG.

1. Use the *Manage Distributed Port Groups* wizard (shown below) and select *Teaming and failover* to adjust the settings for multiple port groups simultaneously.  
	Or, you can edit the settings of each port group individually by selecting *Edit Settings* on that port group and then *Teaming and failover* settings  
	  
	![](https://api-broadcomcms-software.wolkenservicedesk.com/es/attachments/get_attachment_content?uniqueFileId=2124590992280)
2. Highlight the LAG and move it into Standby Uplinks.  
	![](https://api-broadcomcms-software.wolkenservicedesk.com/es/attachments/get_attachment_content?uniqueFileId=2124598041461)
3. You will receive a caution message. This is expected, and you can select OK.  
	![](https://api-broadcomcms-software.wolkenservicedesk.com/es/attachments/get_attachment_content?uniqueFileId=2124560114359)

After you have successfully completed all steps on this page to establish the LAG, you can move the LAG uplink to Active on each port group and move all individual uplinks to Unused to match the recommended configuration.

**Assign NICs to the LAG:**

The next step is to safely move NIC(s) over to the LAG and ensure all portgroups are using the LAG as the **Active** uplink. To ensure connectivity remains up VMware by Broadcom recommends performing the following steps with your network vendor and/or local networking team that has access to make changes to the physical switch during this process.

Example:

**NOTE:** This is a sample and actual adapters and VMNIC numbers may be different for each environment and the steps should be applied to all Hosts that will be using the LAG

1. 1. Identify the NICs (VMNICs) that need to be moved to the LAG adapter group. For this example, the NICs are vmnic0 and vmnic1.
		2. Ensure that all the NICs (VMNICs) are tagged with the correct VLAN, if used.
		3. On the physical switch, down the port connected to vmnic0.
		4. On the physical switch, place the port connected to vmnic0 in the LACP/Etherchannel configuration.
		5. In the *Add and Manage Hosts* wizard, select the host and assign vmnic0 to one of the LAG uplinks by selecting vmnic0 and clicking "Assign uplink". The LAG uplinks will be indented under the LAG name; refer to the screenshot below. Finish the wizard.
		6. On the physical switch, up the port connected to vmnic0.
		7. In the teaming policy for the port groups on the Distributed Switch, you may now move the LAG uplink group to *Active* and all other uplinks to *Unused* (see notes below), or you can repeat the process to add additional NICs to the LAG. Ensure the host stays connected during these changes before moving onto the next NIC.

![](https://api-broadcomcms-software.wolkenservicedesk.com/es/attachments/get_attachment_content?uniqueFileId=2124571435190)

**NOTE:** If at this point, the management vmkernel (generally vmk0) is currently on a Standard vSwitch and will be moved to a vDS portgroup that will have the LAG applied, the management vmk will need to be migrated to the appropriate management vDS portgroup by using the **Add and Manage Hosts** wizard for all Hosts that will be using the LAG. Then continue with steps below:

1. 1. On the physical switch, down the port connected to vmnic1.
		2. Add the vmnic1 to the LAG uplink group on the Distributed Switch through the **Add and Manage Hosts** wizard and ensure that the any VMKernel adapters that use the VMNICs that will be within the LAG are also. Also add the switch port connected to vmnic1 to the LACP/Etherchannel in the physical switch.
		3. On the physical switch, up the vmnic1 port again.
		4. Now the LAG should be set to be the **Active** uplink for all portgroups

**NOTE:** On all the DVS port group you need to use default load balancing "Route based on the originating virtual port"

#### Additional Information

**Note**: For more information, see [vSphere Networking](https://techdocs.broadcom.com/us/en/vmware-cis/vsphere/vsphere/8-0/vsphere-networking-8-0.html) This guide contains definitive information. If there is a discrepancy between the guide and the article, assume the guide is correct.

  
For more information on LACP support on a vSphere Distributed Switch, see:
- [LACP Support on a vSphere Distributed Switch](https://techdocs.broadcom.com/us/en/vmware-cis/vsphere/vsphere/8-0/vsphere-networking-8-0/configuring-lacp-on-a-vsphere-distrubuted-switch-in-the-vsphere-web-client.html)

**Note**: Changing the vDS LACP to Active mode from Passive mode creates a bilateral negotiation where both the virtual and physical switches actively maintain the health of the LAG. Because the protocol transitions from unilateral to bilateral negotiation, the state change is generally transparent to existing traffic.  
  
Japanese Version:  
[LACP を使用する場合の vSphere 分散スイッチ ポートグループでの LACP の構成](https://knowledge.broadcom.com/external/article/322017)

#### Feedback

Was this article helpful?

thumb\_up Yes

thumb\_down No

Powered by

<iframe frameborder="0" title="https://cdn.wolkenservicedesk.com/Broadcom_%20Footer.html" src="https://cdn.wolkenservicedesk.com/Broadcom_%20Footer.html" height="118px"></iframe>