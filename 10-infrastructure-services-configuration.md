# DNS, HTTP, NTP Infrastructure Services Configuration

DNS, HTTP, and NTP services must be configured for the network to function properly. DNS will resolve the hostnames to their IP address, HTTP will provide a internal web server that serves a simple internal site, and NTP will keep all devices synchronized to the same time so the Syslog logs will show the accurate time. the Ubuntu-Admin-PC will need to be configured with a static IP address in order to verify configuration.

This section will cover the static IP address configuration of the Ubuntu-Admin-PC, installing and configuring the bind9 DNS server, installing and configuring the chrony NTP server, configuring NTP on each switch and pfSense, installing and configuring the apache2 web server, updating the name server address on Ubuntu-Infra-Server, and verifying the configurations were successful.

<br>

## DNS Records Table

| Device | Hostname | IP Address |
|--------|----------|------------|
| Ubuntu-Infra-Server | ubuntu-infra-server.ecorp.local | 172.16.0.5 |
| Ubuntu-Mon-Server | ubuntu-mon-server.ecorp.local | 172.16.0.135 |
| Ubuntu-Admin-PC | ubuntu-admin-pc.ecorp.local | 192.168.99.10 |
| L3-Multilayer-SW1 | l3-multilayer-sw1.ecorp.local | 192.168.99.2 |
| L3-Multilayer-SW2 | l3-multilayer-sw2.ecorp.local | 192.168.99.3 |
| L2-SW1 | l2-sw1.ecorp.local | 192.168.99.4 |
| L2-SW2 | l2-sw2.ecorp.local | 192.168.99.5 |
| L2-SW3 | l2-sw3.ecorp.local | 192.168.99.6 |
| pfSense | pfsense.ecorp.local | 192.168.245.2 |

<br>

## Configuring Ubuntu-Admin-PC

The Ubuntu-Admin-PC will need to be configured with a permanent static IP address to verify the services configured in this section and chrony NTP installed.

### Configure a static IP address

Log into Ubuntu-Admin-PC using the default credentials:

Username: ubuntu

Password: ubuntu

Change the hostname using the command:
```
sudo hostnamectl set-hostname admin-pc
```
**Note:** The hostname will change after a reboot.
