# VLAN Configuration and Port Security

This section covers the VLAN configuration for all five switches. This includes creating VLANs, configuring trunk ports, configuring access ports, and enabling port security on access ports. VLANs must be created and configured before any routing or switching protocols are configured in later sections.

<br>

## Creating the VLANs

VLANs must be created on every switch in the network. Even though the L3 switches handle routing, they still need VLANs created to pass traffic across trunk links. 

Apply to each switch.

**Commands:**
```
configure terminal
vlan 10
name HR
vlan 20
name SALES
vlan 30
name FINANCE
vlan 40
name IT
vlan 50
name INFRASTRUCTURE
vlan 60
name MONITORING
vlan 99
name MANAGEMENT
vlan 666
name NATIVE
exit
do write
```
**Note:** If you did not configure the black hole vlan from section 03, create VLAN 999 and name it BLACK-HOLE

**Verify the VLANs were created using:**
```
show vlan brief
```
Each VLAN should be listed with the correct number and name. If it does not include one of the VLANs, just create it again. The status of each VLAN should be active. The unused ports should be listed under BLACK-HOLE as configured in section 03.

![](images/vlancreationimg.PNG)
