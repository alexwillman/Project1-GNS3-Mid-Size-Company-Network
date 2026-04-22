# pfSense Configuration

In this network, pfSense will act as the edge firewall and default gateway for internet traffic. It connects to both layer 3 core switches via point-to-point links and will run OSPF to advertise a default route into the network so devices can reach the internet.
This section will include an addition to the topology and cover the set up of a cloud node for browser access to pfSense, configuring pfSense interfaces, installing the FRR package for OSPF, configuring OSPF on pfSense, verifying NAT, verifying OSPF adjacency, and verifying internet access from end devices.

pfSense firewall rules will be configured in a later section after all internal services are configured. This ensures these services can be configured and verified without firewall rules potentially blocking the traffic from these services.

<br>


## Setting up a Cloud Node for webConfigurator Access

The GNS3 cloud node will allow your host machine to directly access the GNS3 network to allow you to access the pfSense webConfigurator from a browser on your host machine.

### Step 1: Check Your Host Machines VMware adapter IP Address

The pfSense interface connecting to the cloud node needs to be configured on the same subnet as the host machines VMware adapter. First we need to see what subnet that adapter is on.

On your host machine open the Command Prompt and run:
```
ipconfig
```
Look for the IP address that is on **VMware Network Adapter VMnet8**. Note this IPv4 address down. The interface to the cloud node (we will be using em1) must be configured with an IP address on this same subnet. **Example: My VMware Network Adapter VMnet8 shows an IPv4 address of 192.168.245.1. This means the em1 interface to the cloud node must be configured with an IP address of 192.168.245.2.**

### Step 2: Add a Cloud Node to the GNS3 Topology

In GNS3:

- Drag a Cloud Node above the pfSense-Firewall
- Choose the GNS3 VM server
- Right click the cloud node and click configure
- Select eth1 from the available interfaces and click Add then OK
- Connect a cable from the **eth1** interface of the Cloud Node to the **em1** interface of the pfSense-Firewall

The new topology should look like this:

![](images/addcloudnodetopology.PNG)

### Step 3: Assign pfSense Interfaces in the Console Menu

After loading up pfSense, select option 1: Assign Interfaces
```
Should VLANs be set up now?: n
Enter WAN interface name: em0
Enter LAN interface name: em1
Enter optional interface 1: em2
Enter optional interface 2: em3
Enter optional interface 3: (skip by pressing enter)
Do you want to proceed?: y
```

### Step 4: Set WAN and LAN IP addresses in the Console Menu

**WAN interface (em0) connecting to NAT node:**

From the console menu, select Option 2: Set interface(s) IP address
```
Enter the number of the interface you wish to configure: 1 
Configure IPv4 address WAN interface via DHCP?: y
Configure IPv6 address WAN interface via DHCP6?: n
Enter the new WAN IPv6 address: (press enter for none)
```
The WAN interface will now have an IP address recieved from DHCP via the NAT node

**LAN interface (em1) connecting to Cloud node:**

From the console menu, select option 2: Set interface(s) IP address

!!!**THE IP ADDRESS CONFIGURED SHOULD BE ON THE SAME SUBNET AS THE ADDRESS FOUND IN STEP 1**!!!
```
Enter the number of the interface you wish to configure: 2
Configure IPv4 address LAN interface via DHCP?: n
Enter the new LAN IPv4 address: 192.168.245.2
Enter the new LAN IPv4 subnet bit count: 24
Enter the new LAN IPv4 upstream gateway address: (press enter for none)
Configure IPv6 address LAN interface via DHCP6?: n
Enter the new LAN IPv6 address: (press enter for none)
Do you want to enable the DHCP server on LAN?: n
```

<br>

You can now open a browser on your host machine and enter the em1 IPv4 address to open the pfSense webConfigurator.

<br>

## Configure OPT1 (em2) and OPT2 (em3) Interfaces in the webConfigurator

On your host machine, open a browser and enter the em1 IP address you configured.
```
http://192.168.245.2
```
Log in to pfSense with these default credentials:

- Username: admin
- Password: pfSense

### Assign OPT1 (em2) and OPT2 (em3) Interfaces

- Go to Interfaces → Assignments
- em2 and em3 should appear next to available network ports
- Click Add next to em2 and it should become OPT1
- Click Add next to em3 and it should become OPT2
- Click Save

### Configure OPT1 (em2)

This is the link connecting to L3-Multilayer-SW2. Configure it with the point-to-point IP chosen in section 02.

- Go to Interfaces → OPT1
- Check Enable Interface
- Set the description to L3-Multilayer-SW2 P2P Link
- Set IPv4 Configuration Type to Static IPv4
- Set IPv4 Address to 10.0.0.5 with subnet /30
- Leave IPv4 Upstream Gateway blank
- Click Save then Apply Changes

  




