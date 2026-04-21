# HSRP Configuration

HSRP (Hot Standby Router Protocol) is a protocol that provides redundancy for the network by creating a virtual IP address that is shared between the two core switches with one acting as the active and the other as the standby. If the active core switch goes down the standby takes over as the gateway. The end devices use the virtual IP as their default gateway so no changes will be needed on devices if one core switch goes down. HSRP will switch the virtual IP address automatically. In this topology, we will be dedicating one of the core switches to be active for VLANs 10, 20, 30, and 99, and the other switch active for VLANs 40, 50, and 60, to align with the STP root bridge assignments from section 05. This ensures that both switches are handling traffic rather than one sitting idle as a backup.

In this section we will cover the configuration of HSRP, verification of HSRP, ping tests, and failover tests to confirm connectivity.

<br>

## HSRP Active Assignments

| VLAN | Virtual IP | Active Switch | Standby Switch |
|------|------------|---------------|----------------|
| 10 | 192.168.0.1 | L3-Multilayer-SW1 | L3-Multilayer-SW2 |
| 20 | 192.168.1.1 | L3-Multilayer-SW1 | L3-Multilayer-SW2 |
| 30 | 192.168.2.1 | L3-Multilayer-SW1 | L3-Multilayer-SW2 |
| 40 | 192.168.3.1 | L3-Multilayer-SW2 | L3-Multilayer-SW1 |
| 50 | 172.16.0.1 | L3-Multilayer-SW2 | L3-Multilayer-SW1 |
| 60 | 172.16.0.129 | L3-Multilayer-SW2 | L3-Multilayer-SW1 |
| 99 | 192.168.99.1 | L3-Multilayer-SW1 | L3-Multilayer-SW2 |


### Active Switch Priority

HSRP uses a higher priority to win the role of active switch, so the active switch for a particular VLAN will be configured with priority 110 and it will always win the election. The standby switch will keep the default priority of 100 so it only wins after the active switch goes down. We will be enabling preemption on the active switch so it will automatically recover the active role after the switch comes back up. Preemption is only configured on the switch that is active for that VLAN.

<br>

## Configuring HSRP

To keep the configuration easy to read, the HSRP group number for each VLAN will match the VLAN ID.

### L3-Multilayer-SW1

L3-Multilayer-SW1 is the active switch for VLANs 10, 20, 30, and 99 with priority 110 and preemption enabled. It is the standby switch for VLANs 40, 50, and 60 so those VLANs will have the default priority 100 and no preemption.

```
enable
configure terminal

interface Vlan10
standby version 2
standby 10 ip 192.168.0.1
standby 10 priority 110
standby 10 preempt
exit




