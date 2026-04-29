# SNMP and Syslog Configuration

SNMP (Simple Network Management Protocol) allows a server to collect data from the devices in our network, such as device performance and health. The data it collects includes things like CPU usage, memory usage, uptime, and interface status. Syslog allows our network devices and servers to send log information to a server for troubleshooting and monitoring. The syslog can log warning messages to a central server so device messages are easier to keep track of.

This section will cover configuring Ubuntu-Mon-Server with a static IP, hostname, and base services, installing and configuring snmpd on Ubuntu-Mon-Server, configuring SNMP on all switches, installing and configuring rsyslog on Ubuntu-Mon-Server, configuring Syslog on all switches, servers, and pfSense, and finally verify these services are functioning correctly. 

<br>

## Configuring Ubuntu-Mon-Server

Before other configuration, the monitoring server needs a static IP configured and other base services installed.

<br>

Log into Ubuntu-Mon-Server with the default credentials:

Username: ubuntu

Password: ubuntu

### Change the hostname

To change the hostname of Ubuntu-Mon-Server, use the command:
```
sudo hostnamectl set-hostname mon-server
```
**Note:** The hostname will update after the device is rebooted.

Then update the hosts file using the command:
```
sudo nano /etc/hosts
```

After 127.0.0.1 localhost, add the line:
```
127.0.1.1 mon-server
```
Then save with Ctrl+X, then y, then enter.

![](images/changehostnameimg.PNG)

### Disable cloud-init network management

To disable cloud-init and prevent the system from overwriting the IP on boot, use the command:
```
sudo nano /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
```

Then add this in the file:
```
network: {config: disabled}
```
Then save with Ctrl+x, then y, then enter.

![](images/disablecloudinitmg.PNG)

### Configure a Static IP

Configure a static IP address using the IP address from section 02.

Use the command:
```
sudo nano /etc/netplan/50-cloud-init.yaml
```

Then edit the file to this:
```
network:
  version: 2
  ethernets:
    ens3:
      addresses:
        - 172.16.0.135/25
      routes:
        - to: default
          via: 172.16.0.129
      nameservers:
        addresses:
          - 172.16.0.5
```
Then save with Ctrl+X, then y, then enter.

![](images/staticipconfigimg.PNG)

Apply the configuration using the command:
```
sudo netplan apply
```

### Bypass systemd-resolved





