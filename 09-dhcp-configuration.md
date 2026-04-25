# DHCP Configuration

DHCP (Dynamic Host Configuration Protocol) automatically assigns IP addresses to devices when they connect to the network. Without it, all of the department workstations would need to be manually configured with ip addresses, subnet masks, default gateways, and other network settings like DNS and NTP servers. The Ubuntu-Infra-Server will be acting as the DHCP server for each department VLAN and be assigning available addresses from the DHCP pool once the device connects to the network. Since the DHCP server is on a separate VLAN, the layer 3 core switches will relay the DHCP requests to the server.

This section will cover configuring a static IP on Ubuntu-Infra-Server, installing isc-dhcp-server on Ubuntu-Infra-Server, configuring the DHCP pools for the department VLANs, configuring DHCP relay using 'ip helper-address' on the layer 3 core switches, verifying the workstations receive addresses through DHCP, and confirming network connectivity through ping testing.

<br>

## Configuring a Static IP on Ubuntu-Infra-Server

Before configuring DHCP, we will need to set a static IP address on the Ubuntu-Infra-Server. The IP address we configured in section 07 is only temporary and will be removed once the device reboots. We will now configure the IP address to stay even after a reboot.

### Log in to Ubuntu-Infra-Server using default credentials

Username: ubuntu

Password: ubuntu

### Disable cloud-init network management

Disabling cloud-init network management prevents the system from overwriting the static IP on reboot.

Type:
```
sudo nano /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
```
In this file, add the line:
```
network: {config: disabled}
```
Then save the file with Ctrl+X, then Y, then press enter.

![](images/disablecloudinitimg.PNG)

### Edit netplan configuration file

To set the new IP address, we must edit the netplan configuration file.

Type:
```
sudo nano /etc/netplan/50-cloud-init.yaml
```
Replace the file contents with this:
```
network:
  version: 2
  ethernets:
    ens3:
      addresses:
        - 172.16.0.5/25
      routes:
        - to: default
          via: 172.16.0.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
```
**Note:** A temporary public DNS server is required in order to update the package list.

Then save the file with Ctrl+X, then Y, then enter.


![](images/netplanconfigimg.PNG)

Apply the new configuration using:
```
sudo netplan apply
```

### Verify new IP address

Verify the newly configured IP address with:
```
ip addr show
```

![](images/verifynewIPimg.PNG)

<br>

## Installing isc-dhcp-server on Ubuntu-Infra-Server

First we update then install isc-dhcp-server using:
```
sudo apt update
sudo apt install isc-dhcp-server -y
```
**Note:** The service will fail to start after installing because we have not set up the configuration file yet.

<br>

## Configuring DHCP Interface Binding
