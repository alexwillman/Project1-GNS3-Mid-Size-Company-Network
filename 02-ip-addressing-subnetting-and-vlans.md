# IP Addressing, Subnetting, and VLANs for the Network

This section covers the complete addressing scheme and VLAN assignments for the network. The network uses three separate RFC 1918 private address ranges to easily identify where the traffic is coming from and what device it belongs to, which simplifies troubleshooting. The end devices use the 192.168.0.0/16 range with /24 subnets, the servers 
use the 172.16.0.0/12 range with /25 subnets, and the point-to-point links between pfSense and the Layer 3 Switches use the 10.0.0.0/8 range with /30 subnets. The peer link between both core switches will not use point-to-point addressing because it is an EtherChannel trunk. OSPF adjacency for those devices will form over the VLAN 99 SVI.

This network will have devices that are configured statically and dynamically. End device workstations will be configured dynamically and receive addresses from DHCP. Servers, network devices, and the management workstation will be configured statically. These devices are static because other devices must reach them
at a consistent IP address. 

Each VLAN has three addresses at the core layer. One SVI on L3-Multilayer-SW1, one SVI on L3-Multilayer-SW2, and one HSRP virtual IP that acts as the default gateway for all devices on that VLAN. End devices and servers must use the HSRP virtual IP as their default gateway, not the
switch SVI addresses. If the active switch fails, HSRP automatically changes the virtual IP to the standby switch and traffic continues.

<br>

## IP Addressing Table

| VLAN | Name | Network ID | Subnet Mask | HSRP Virtual IP | L3-Multilayer-SW1 SVI | L3-Multilayer-SW2 SVI | Assignment |
|------|------|------------|-------------|-----------------|-----------------------|-----------------------|------------|
| 10 | HR | 192.168.0.0 | 255.255.255.0 | 192.168.0.1 | 192.168.0.2 | 192.168.0.3 | DHCP |
| 20 | Sales | 192.168.1.0 | 255.255.255.0 | 192.168.1.1 | 192.168.1.2 | 192.168.1.3 | DHCP |
| 30 | Finance | 192.168.2.0 | 255.255.255.0 | 192.168.2.1 | 192.168.2.2 | 192.168.2.3 | DHCP |
| 40 | IT | 192.168.3.0 | 255.255.255.0 | 192.168.3.1 | 192.168.3.2 | 192.168.3.3 | DHCP |
| 50 | Infrastructure | 172.16.0.0 | 255.255.255.128 | 172.16.0.1 | 172.16.0.2 | 172.16.0.3 | Static |
| 60 | Monitoring | 172.16.0.128 | 255.255.255.128 | 172.16.0.129 | 172.16.0.130 | 172.16.0.131 | Static |
| 99 | Management | 192.168.99.0 | 255.255.255.0 | 192.168.99.1 | 192.168.99.2 | 192.168.99.3 | Static |

**Note:** The HSRP Virtual IP is the default gateway for all devices on that VLAN. The L3-Multilayer-SW1 SVI and L3-Multilayer-SW2 SVI are the actual interface addresses configured on each core switch.

## HSRP Active Gateway Assignments

| Device | HSRP Active VLANs | HSRP Standby VLANs |
|--------|-------------------|--------------------|
| L3-Multilayer-SW1 | 10,20,30,99 | 40,50,60 |
| L3-Multilayer-SW2 | 40,50,60 | 10,20,30,99 |

## Static Device IP Adressing

| Device | Interface | VLAN | IP Address | Subnet Mask | Default Gateway |
|--------|-----------|------|------------|-------------|-----------------|
| Ubuntu-Infra-Server | e0 | 50 | 172.16.0.5 | 255.255.255.128 | 172.16.0.1 |
| Ubuntu-Mon-Server | e0 | 60 | 172.16.0.135 | 255.255.255.128 | 172.16.0.129 |
| Ubuntu-Admin-PC | e0 | 99 | 192.168.99.10 | 255.255.255.0 | 192.168.99.1 |
| L2-SW1 | Vlan99 | 99 | 192.168.99.4 | 255.255.255.0 | 192.168.99.1 |
| L2-SW2 | Vlan99 | 99 | 192.168.99.5 | 255.255.255.0 | 192.168.99.1 |
| L2-SW3 | Vlan99 | 99 | 192.168.99.6 | 255.255.255.0 | 192.168.99.1 |

**Note:** L2 switch management addresses are configured on a VLAN 99 SVI using interface Vlan99. The default gateway for each L2 Switch points to the HSRP virtual IP for VLAN 99 so the management traffic can be routed.

## Point-to-Point Link IP Addressing

In this network, the point-to-point links use /30 subnets which provide two usable host addresses. This provides exactly one address for each side of the link.

| Device | Interface | IP Address | Subnet Mask | Connected To | Interface | Network |
|--------|-----------|------------|-------------|--------------|-----------|---------|
| pfSense-Firewall | em3 | 10.0.0.1 | 255.255.255.252 | L3-Multilayer-SW1 | Gi3/3 | 10.0.0.0/30 |
| L3-Multilayer-SW1 | Gi3/3 | 10.0.0.2 | 255.255.255.252 | pfSense-Firewall | em3 | 10.0.0.0/30 |
| pfSense-Firewall | em2 | 10.0.0.5 | 255.255.255.252 | L3-Multilayer-SW2 | Gi3/2 | 10.0.0.4/30 |
| L3-Multilayer-SW2 | Gi3/2 | 10.0.0.6 | 255.255.255.252 | pfSense-Firewall | em2 | 10.0.0.4/30 |

**Note:** The peer link between L3-Multilayer-SW1 and L3-Multilayer-SW2 is a Layer 2 EtherChannel trunk that carries all VLANs. OSPF adjacency between the two Layer 3 switches forms over the VLAN 99 SVI and therefore does not use a point-to-point address.


