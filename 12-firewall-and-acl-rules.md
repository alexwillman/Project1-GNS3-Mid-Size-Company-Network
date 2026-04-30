# Firewall and ACL rules

Firewall rules and ACL (Access Control List) rules will work together to provide control over traffic flow in the network. Together, they will control how each VLAN in the network communicates. The firewall rules will control the traffic between the internal network and the internet, and the ACL rules will control traffic inside the internal network.

This section will cover the traffic rules, replacing the temporary firewall rules with specific rules, configuring ACLs on L3-Multilayer-SW1 and L3-Multilayer-SW2, and verifying the rules are working correctly.

<br>

## ACL and Firewall Rule Summary

This design provides isolation of department VLANs, access to the internet, control over inter-VLAN communication, access to necessary services, and control over ICMP traffic to restrict unnecessary pings from department VLANs and allow necessary troubleshooting pings for IT, Management, and servers. As this is a lab environment, this is a very simple version of the rules required. Production environments would require more complex rule sets for security.

<br>

| Source | Internet | Other Departments | Servers (DNS/NTP/HTTP) | Server SSH | Admin PC | IT/Management VLANs | ICMP |
|--------|----------|-------------------|------------------------|------------|----------|---------------|------|
| Department VLANs 10/20/30 | Allow | Deny | Allow | Deny | Deny | Deny | Echo-reply to VLAN 40,99 only |
| IT VLAN 40 | Allow | Allow | Allow | Allow | Allow | Allow | Allow all |
| Management VLAN 99 | Allow | Allow | Allow | Allow | N/A | Allow | Echo only |
| Server VLAN 50/60 | Allow | N/A | N/A | N/A | N/A | N/A | Allow all (troubleshooting) |

<br>

## Replacing Temporary Firewall Rules With Specific Rules

The temporary allow all rules that were applied in section 08 need to be updated to more specific rules so we can control traffic between the internal network and the internet.

<br>

Log into pfSense webConfigurator using the default credentials:

**Note:** You should change these credentials for security but for lab purposes leaving them default is fine.

Username: admin

Password: pfsense

### Delete temporary allow all rules

- Go to Firewall → Rules → L3MULTILAYERSW2LINK
- Click on the rule then click Delete to remove the allow all rule
- Click Apply Changes
- Go to L3MULTILAYERSW1LINK
- Click on the rule then click Delete to remove the allow all rule
- Click Apply Changes

![](images/deletetempallowrulesimg1.PNG)

![](images/deletetempallowrulesimg2.PNG)

### Add specific rules for L3MULTILAYERSW2LINK and L3MULTILAYERSW1LINK

Go to Firewall → Rules → L3MULTILAYERSW2LINK

Add these rules in order by clicking the Add button with the down arrow for each rule:

**Rule 1 - Allow OSPF:**

- Action: Pass
- Protocol: OSPF
- Source: Any
- Destination: Any
- Description: Allow OSPF
- Leave others as default
- Click Save

**Rule 2 - Allow OSPF Multicast 224.0.0.5:**
  
- Action: Pass
- Protocol: Any
- Source: Any
- Destination: Address or Alias then type 224.0.0.5 in the Destination Address box
- Description: Allow OSPF Multicast 224.0.0.5
- Leave others as default
- Click Save

**Rule 3 - Allow OSPF Multicast 224.0.0.6:**

- Action: Pass
- Protocol: Any
- Source: Any
- Destination: Address or Alias then type 224.0.0.6 in the Destination Address box
- Description: Allow OSPF Multicast 224.0.0.6
- Leave others as default
- Click Save

**Rule 4 - Allow pfSense NTP:**

- Action: Pass
- Protocol: UDP
- Source: Address or Alias then type 10.0.0.5 in the Source Address box
- Destination: Address or Alias then type 172.16.0.5 in the Destination Address box
- Destination Port Range: from NTP (123) to NTP (123)
- Description: Allow pfSense NTP to Infra Server
- Leave others as default
- Click Save

**Rule 5 - Allow pfSense DNS:**

- Action: Pass
- Protocol: TCP/UDP
- Source: Address or Alias then type 10.0.0.5 in the Source Address box
- Destination: Address or Alias then type 172.16.0.5 in the Destination Address box
- Destination Port Range: from DNS (53) to DNS (53)
- Description: Allow pfSense DNS to Infra Server
- Leave others as default
- Click Save

**Rule 6 - Allow pfSense Syslog:**

- Action: Pass
- Protocol: UDP
- Source: Address or Alias then type 10.0.0.5 in the Source Address box
- Destination: Address or Alias then type 172.16.0.135 in the Destination Address box
- Destination Port Range: from Syslog (514) to Syslog (514)
- Description: Allow pfSense Syslog to Mon server
- Leave others as default
- Click Save

**Rule 7 - Allow Internal Subnets to Internet:**

- Action: Pass
- Protocol: Any
- Source: Network then type 192.168.0.0 in the Source Address box and choose /16
- Destination: Any
- Description: Allow Internal Subnet Internet Access
- Leave others as default
- Click Save

**Rule 8 - Allow Servers to Internet (for future package installation):**

- Action: Pass
- Protocol: Any
- Source: Network then type 172.16.0.0 in the Source Address box and choose /16
- Destination: Any
- Description: Allow Servers Internet Access
- Leave others as default
- Click Save

After adding all rules in order, click Apply Changes.

![](images/firewallrulesl3multilayersw2linkimg.PNG)

Then go to Firewall → Rules → L3MULTILAYERSW1LINK and add these rules in order:

**Rule 1 - Allow OSPF:**

- Same values as the previous Rule 1.

**Rule 2 - Allow OSPF Multicast 224.0.0.5:**
  
- Same values as the previous Rule 2.

**Rule 3 - Allow OSPF Multicast 224.0.0.6:**

- Same values as the previous Rule 3.

**Rule 4 - Allow pfSense NTP:**

- Action: Pass
- Protocol: UDP
- Source: Address or Alias then type 10.0.0.1 in the Source Address box
- Destination: Address or Alias then type 172.16.0.5 in the Destination Address box
- Destination Port Range: from NTP (123) to NTP (123)
- Description: Allow pfSense NTP to Infra Server
- Leave others as default
- Click Save

**Rule 5 - Allow pfSense DNS:**

- Action: Pass
- Protocol: TCP/UDP
- Source: Address or Alias then type 10.0.0.1 in the Source Address box
- Destination: Address or Alias then type 172.16.0.5 in the Destination Address box
- Destination Port Range: from DNS (53) to DNS (53)
- Description: Allow pfSense DNS to Infra Server
- Leave others as default
- Click Save

**Rule 6 - Allow pfSense Syslog:**

- Action: Pass
- Protocol: UDP
- Source: Address or Alias then type 10.0.0.1 in the Source Address box
- Destination: Address or Alias then type 172.16.0.135 in the Destination Address box
- Destination Port Range: from Syslog (514) to Syslog (514)
- Description: Allow pfSense Syslog to Mon server
- Leave others as default
- Click Save

**Rule 7 - Allow Internal Subnets to Internet:**

- Same values as the previous Rule 7.

**Rule 8 - Allow Servers to Internet (for future package installation):**

- Same values as the previous Rule 8.

Then Apply Changes.

![](images/firewallrulesl3multilayersw1linkimg.PNG)

### Verify OSPF after changing rules

On both L3-Multilayer-SW1 and L3-Multilayer-SW2, verify OSPF is still forming an adjacency with pfSense.

Use the command:
```
enable

show ip ospf neighbor
```
pfSense should still apear as a FULL neighbor.

**L3-Multilayer-SW1:**

![](images/verifyospfafterfirewallimg1.PNG)

**L3-Multilayer-SW2:**

![](images/verifyospfafterfirewallimg2.PNG)

<br>

## Configuring ACLs on L3-Multilayer-SW1


