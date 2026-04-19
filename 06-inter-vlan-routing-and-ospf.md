# Inter-VLAN Routing and OSPF

This section covers the layer 3 configuration in the network and how communication is provided between VLANs and devices in the network. This includes enabling IP routing on the core switches to let them operate as layer 3 devices, setting interfaces for the uplinks to the pfSense firewall, configuring SVIs for each VLAN, configuring management SVIs, and configuring OSPF for routing across the network. Basic connectivity between devices will be verified in this section.

<br>

## Enable IP Routing

IP routing must be enabled on both layer 3 core switches to allow them to operate at layer 3 and route traffic between VLANs.

Apply this to both L3-Multilayer-SW1 and L3-Multilayer-SW2:
```
enable
configure terminal
ip routing
exit
write
```

<br>

## Configuring Interfaces for pfSense Uplinks

Before setting IP addresses on the interfaces, you must first use the 'no switchport' command on the interface to turn the layer 2 switchports into layer 3 routed interfaces, even after using the 'ip routing' command.
After that you can assign them the IP addresses from the point-to-point section in 02-ip-addressing-subnetting-and-vlans.

### L3-Multilayer-SW1 Gi3/3

```
enable
configure terminal
interface Gi3/3
no switchport
ip address 10.0.0.2 255.255.255.252
description Link to pfSense-Firewall em3
no shutdown
exit
do write
```

![](images/ConfigInterfacepfsenseuplinkimg1.PNG)

### L3-Multilayer-SW2 Gi3/2

```
enable
configure terminal
interface Gi3/2
no switchport
ip address 10.0.0.6 255.255.255.252
description Link to pfSense-Firewall em2
no shutdown
exit
do write
```

![](images/ConfigInterfacepfsenseuplinkimg2.PNG)

### Verify

To confirm this worked, run:
```
show ip interface brief
```
And confirm that the correct interface shows the correct ip address. Also confirm that the status and protocol both show 'up'.

<br>

## Configuring SVIs on the Layer 3 Switches

The Switched Virtual Interfaces (SVIs) are the layer 3 gateway interfaces for each VLAN. Each SVI will be assigned the switch's IP address for each VLAN. Use the address table from Section 02 for the ip address.

Before you configure the SVIs power on each end device/server. This ensures the SVIs come up when you verify.











