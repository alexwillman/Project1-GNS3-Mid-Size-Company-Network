# pfSense Configuration

In this network, pfSense will act as the edge firewall and default gateway for internet traffic. It connects to both layer 3 core switches via point-to-point links and will run OSPF to advertise a default route into the network so devices can reach the internet.
This section will include an addition to the topology and cover the set up of a cloud node for browser access to pfSense, configuring pfSense interfaces, installing the FRR package for OSPF, configuring OSPF on pfSense, verifying NAT, verifying OSPF adjacency, and verifying internet access from end devices.

pfSense firewall rules will be configured in a later section after all internal services are configured. This ensures these services can be configured and verified without firewall rules potentially blocking the traffic from these services.

<br>










