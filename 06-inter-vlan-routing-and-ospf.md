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

## Configuring Interfaces for pfSense Uplinks

Before setting IP addresses on the interfaces, you must first use the 'no switchport' command on the interface to turn the layer 2 switchports into layer 3 routed interfaces, even after using the 'ip routing' command.
After that you can assign them the IP addresses from section 2, 02-ip-addressing-subnetting-and-vlans.

### L3-Multilayer-SW1 Gi3/3

