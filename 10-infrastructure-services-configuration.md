# DNS, HTTP, NTP Infrastructure Services Configuration

DNS, HTTP, and NTP services must be configured for the network to function properly. DNS will resolve the hostnames to their IP address, HTTP will provide a internal web server that serves a simple internal site, and NTP will keep all devices synchronized to the same time so the Syslog logs will show the accurate time. the Ubuntu-Admin-PC will need to be configured with a static IP address in order to verify configuration.

This section will cover installing and configuring the bind9 DNS server, installing and configuring the chrony NTP server, updating the name server address on Ubuntu-Infra-Server, the static IP address configuration of the Ubuntu-Admin-PC, configuring NTP on each switch and pfSense, installing and configuring the apache2 web server, and verifying the configurations were successful.

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

## Installing and Configuring bind9 on Ubuntu-Infra-Server

The bind9 DNS server will be installed and configured and allow the devices to access resources using hostnames instead of IP addresses.

<br>

Log into Ubuntu-Infra-Server using the default credentials:

Username: ubuntu

Password: ubuntu

### Configure a hostname

A hostname will be configured to allow easy recognition of the device from the terminal.

<br>

Then change the hostname using the command:
```
sudo hostnamectl set-hostname infra-server
```
**Note:** The hostname will be updated after a reboot.

Then add the hostname to the hosts file using the command:
```
sudo nano /etc/hosts
```

After the 127.0.0.1 localhost line, add:
```
127.0.1.1 infra-server
```
Then save with Ctrl+X, then y, then enter.

![](images/updatehostsfileimg1.PNG)

### Install bind9

Install bind9 using the commands:
```
sudo apt update
sudo apt install bind9 bind9utils -y
```

### Configure bind9 options

By editing the bind9 options file we will restrict bind9 to listen only on the VLAN 50 interface, set bind9 to forward any queries it cannot resolve to Google's DNS servers, restrict recursive queries to only internal subnets, and restrict which clients can send queries to only internal subnets.

<br>

The command to edit the bind9 options file is:
```
sudo nano /etc/bind/named.conf.options
```

Edit the file to:
```
options {
    directory "/var/cache/bind";

    // Forward external queries to Google DNS
    forwarders {
        8.8.8.8;
        8.8.4.4;
    };

    // Listen on VLAN 50 interface only
    listen-on { 172.16.0.5; };
    listen-on-v6 { none; };

    // Allow recursion for internal subnets only
    allow-recursion {
        192.168.0.0/16;
        172.16.0.0/16;
        127.0.0.1;
    };

    // Allow queries from internal subnets
    allow-query {
        192.168.0.0/16;
        172.16.0.0/16;
        127.0.0.1;
    };

    dnssec-validation no;
    recursion yes;
};
```
Then save with Ctrl+X, then y, then enter.

![](images/bind9optionsimg1.PNG)

![](images/bind9optionsimg2.PNG)

### Configure forward and reverse zones

The forward zone will map hostnames to IPv4 addresses inside our domain ecorp.local and the reverse zone will map IPv4 addresses back to hostnames.

<br>

Edit the zones configuration file using the command:
```
sudo nano /etc/bind/named.conf.local
```

Edit the file to add the forward zone, reverse zone for 192.168.0.0/16, and reverse zone for 172.16.0.0/25.
```
// Forward zone
zone "ecorp.local" {
    type master;
    file "/etc/bind/db.ecorp.local";
};

// Reverse zone for 192.168.0.0/16
zone "168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.192.168";
};

// Reverse zone for 172.16.0.0/25
zone "16.172.in-addr.arpa" {
    type master;
    file "/etc/bind/db.172.16";
};
```
Then save with Ctrl+X, then y, then enter.

![](images/zonesconfigfileimg1.PNG)

![](images/zonesconfigfileimg2.PNG)

Then create the forward zone file using the command:
```
sudo nano /etc/bind/db.ecorp.local
```

Edit the file to:
```
$TTL    86400
@       IN      SOA     ubuntu-infra-server.ecorp.local. admin.ecorp.local. (
                        2024010101  ; Serial
                        3600        ; Refresh
                        1800        ; Retry
                        604800      ; Expire
                        86400 )     ; Negative TTL

; Name servers
@       IN      NS      ubuntu-infra-server.ecorp.local.

; Ubuntu Servers
ubuntu-infra-server     IN      A       172.16.0.5
ubuntu-mon-server       IN      A       172.16.0.135
ubuntu-admin-pc         IN      A       192.168.99.10

; Core Switches
l3-multilayer-sw1       IN      A       192.168.99.2
l3-multilayer-sw2       IN      A       192.168.99.3

; Access Switches
l2-sw1                  IN      A       192.168.99.4
l2-sw2                  IN      A       192.168.99.5
l2-sw3                  IN      A       192.168.99.6

; pfSense Firewall
pfsense                 IN      A       192.168.245.2

; Web server alias
www                     IN      CNAME   ubuntu-infra-server.ecorp.local.
```
Then save with Ctrl+X, then y, then enter.

**Note:** This file ensures that a query to the device name (e.g ubuntu-infra-server) returns the correct IP address.

![](images/forwardzonefileimg1.PNG)

![](images/forwardzonefileimg2.PNG)

Now create the reverse zone file for the 192.168.0.0/16 addresses.
```
sudo nano /etc/bind/db.192.168
```

And edit the file to:
```
$TTL    86400
@       IN      SOA     ubuntu-infra-server.ecorp.local. admin.ecorp.local. (
                        2024010101  ; Serial
                        3600        ; Refresh
                        1800        ; Retry
                        604800      ; Expire
                        86400 )     ; Negative TTL

; Name servers
@       IN      NS      ubuntu-infra-server.ecorp.local.

; PTR Records
10.99   IN      PTR     ubuntu-admin-pc.ecorp.local.
2.99    IN      PTR     l3-multilayer-sw1.ecorp.local.
3.99    IN      PTR     l3-multilayer-sw2.ecorp.local.
4.99    IN      PTR     l2-sw1.ecorp.local.
5.99    IN      PTR     l2-sw2.ecorp.local.
6.99    IN      PTR     l2-sw3.ecorp.local.
```
Then save with Ctrl+X, then y, then enter.

**Note:** This file ensures that reverse lookups on 192.168 IP addresses resolve to the correct hostname.

![](images/reversezonefileimg.PNG)

Then create the reverse zone file for the 172.16.0.0/25 addresses.
```
sudo nano /etc/bind/db.172.16
```

And edit the file to:
```
$TTL    86400
@       IN      SOA     ubuntu-infra-server.ecorp.local. admin.ecorp.local. (
                        2024010101  ; Serial
                        3600        ; Refresh
                        1800        ; Retry
                        604800      ; Expire
                        86400 )     ; Negative TTL

; Name servers
@       IN      NS      ubuntu-infra-server.ecorp.local.

; PTR Records
5.0     IN      PTR     ubuntu-infra-server.ecorp.local.
135.0   IN      PTR     ubuntu-mon-server.ecorp.local.
```
Then save with Ctrl+X, then y, then enter.

**Note:** This file ensures that reverse lookups on the 172.16.0.0/25 IP addresses resolve to the correct hostname.

![](images/reversezonefileimg2.PNG)

### Check for configuration errors

Before starting the service, we should check for configuration errors using the commands:
```
sudo named-checkconf /etc/bind/named.conf
sudo named-checkzone ecorp.local /etc/bind/db.ecorp.local
sudo named-checkzone 168.192.in-addr.arpa /etc/bind/db.192.168
sudo named-checkzone 16.172.in-addr.arpa /etc/bind/db.172.16
```
**Note:** Make sure to fix any configuration errors before starting the service.

![](images/checkconfigerror.PNG)

### Start and enable the bind9 service

bind9 is technically an alias name and might not work with all systemctl commands. We will be using the actual daemon name "named" to start the service.

<br>

To start bind9 and enable on startup, use the commands:
```
sudo systemctl enable named
sudo systemctl restart named
```

Then we can verify it is running using the command:
```
sudo systemctl status named --no-pager -l
```

![](images/enablebind9img.PNG)

<br>

## Installing and Configuring chrony on Ubuntu-Infra-Server

chrony is the NTP software we will be using to sync time across devices.

<br>

### Install chrony

Install chrony using the command:
```
sudo apt install chrony -y
```

### Configure chrony

Edit the configuration file using the command:
```
sudo nano /etc/chrony/chrony.conf
```
Replace all 4 pool lines with:
```
pool pool.ntp.org iburst minpoll 4 maxpoll 6
```
Then directly under that line, we allow internal subnets and pfSense to use this server using:
```
allow 192.168.0.0/16
allow 172.16.0.0/16
allow 10.0.0.0/24
```

Then change the makestep 1 3 line to:
```
makestep 1 -1
```
Then save with Ctrl+X, then y, then enter.

![](images/chronyconfigimg.PNG)

![](images/chronyconfigimg1.PNG)

### Configure a timezone (EDT/EST)

To set the timezone to US Eastern time, use the command:
```
sudo timedatectl set-timezone America/New_York
```
You can verify this change was applied using the command:
```
timedatectl
```

![](images/timezoneconfigimg.PNG)

### Restart, enable, and verify chrony

Restart and enable the service using:
```
sudo systemctl restart chrony
sudo systemctl enable chrony
```

Verify chrony is syncing time using the command:
```
chronyc tracking
```
**Note:** This command will still show the UTC time even with a timezone configured.

![](images/chronyverifyimg.PNG)

<br>

## Update Nameserver in Ubuntu-Infra-Server Network Configuration

Now that the bind9 DNS server is running, we can change the nameservers in the network configuration on Ubuntu-Infra-Server to point to itself as the nameserver. We set bind9 to forward external queries to Google's DNS so that we no longer need them in the network configuration.

<br>

### Edit network configuration

To edit the network configuration use the command:
```
sudo nano /etc/netplan/50-cloud-init.yaml
```

Change the address under nameservers to:
```
- 172.16.0.5
```
Then save with Ctrl+X, then y, then enter.

![](images/updatenameserverimg.PNG)

### Apply the configuration

Apply the new config using the command:
```
sudo netplan apply
```

### Bypass systemd-resolved

We need to bypass systemd-resolved so DNS queries get forwarded to the DNS server directly.

<br>

To bypass systemd-resolved, use the commands:
```
sudo unlink /etc/resolv.conf
sudo nano /etc/resolv.conf
```
In this file, add:
```
nameserver 172.16.0.5
```
Save with Ctrl+X, then y, then enter.

**Note:** This forces queries to be forwarded to the nameserver at 172.16.0.5 directly.

![](images/bypassresolvedimg.PNG)

### Verify DNS resolution

DNS resolution should be working correctly on Ubuntu-Infra-Server.

<br>

To verify, run the commands:
```
nslookup ubuntu-infra-server.ecorp.local
nslookup google.com
nslookup l3-multilayer-sw1.ecorp.local
```

![](images/verifydnsresolutionimg.PNG)

<br>

## Configuring Ubuntu-Admin-PC

The Ubuntu-Admin-PC will need to be configured with a permanent static IP address to verify the services configured in this section and have chrony NTP installed.

Log into Ubuntu-Admin-PC using the default credentials:

Username: ubuntu

Password: ubuntu

### Configure a hostname

Change the hostname using the command:
```
sudo hostnamectl set-hostname admin-pc
```
**Note:** The hostname will be updated after a reboot.

Update the hosts file using the command:
```
sudo nano /etc/hosts
```
After the 127.0.0.1 localhost line, add:
```
127.0.1.1 admin-pc
```
Then save with Ctrl+X, then y, then enter.

![](images/updatehostsfileimg2.PNG)

### Disable cloud-init network management

Like before, we need to disable cloud-init to prevent the system from overwriting the IP.

Commands:
```
sudo nano /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
```
Then add:
```
network: {config: disabled}
```
Save the file with Ctrl+X, then Y, then enter.

### Configure a static IP

We will use the IP address of 192.168.99.10 and a default gateway of 192.168.99.1, as written in section 02.

<br>

To configure a static IP, use the command:
```
sudo nano /etc/netplan/50-cloud-init.yaml
```

Then edit the file to:
```
network:
  version: 2
  ethernets:
    ens3:
      addresses:
        - 192.168.99.10/24
      routes:
        - to: default
          via: 192.168.99.1
      nameservers:
        addresses:
          - 172.16.0.5
```
Then save with Ctrl+X, then y, then enter.

![](images/adminpcstaticipimg.PNG)

Then apply the configuration with the command:
```
sudo netplan apply
```

### Bypass systemd-resolved

Just like Ubuntu-Infra-Server, we need to bypass systemd-resolved so DNS queries will get forwarded to the DNS server directly.

<br>

Use the commands:
```
sudo unlink /etc/resolv.conf
sudo nano /etc/resolv.conf
```

In this file, add:
```
nameserver 172.16.0.5
```
Then save with Ctrl+X, then y, then enter.

![](images/bypassresolvedimg2.PNG)

Then verify the new IP address using the command:
```
ip addr show
```

![](images/verifyipadminpc.PNG)

### Install and configure chrony

To install chrony, use the command:
```
sudo apt update
sudo apt install chrony -y
```

Then configure chrony to use Ubuntu-Infra-Server as the NTP source with the command:
```
sudo nano /etc/chrony/chrony.conf
```

Replace all 4 of the pool lines with:
```
server 172.16.0.5 iburst minpoll 4 maxpoll 6
```

Then change the makestep 1 3 line to:
```
makestep 1 -1
```
Then save with Ctrl+X, then y, then enter.

![](images/chronyconfigadminpcimg.PNG)

Restart and enable chrony to run on startup with the commands:
```
sudo systemctl restart chrony
sudo systemctl enable chrony
```

Set the timezone to US Eastern time with the command:
```
sudo timedatectl set-timezone America/New_York
```

To verify the timezone is set and chrony is syncing, use the commands:
```
timedatectl
chronyc tracking
```

![](images/verifychronyadminpc.PNG)

<br>

## Configuring NTP on the Switches

All five switches need to be configured to sync time from Ubuntu-Infra-Server. We are setting the minpoll and maxpoll options to poll more frequently to help reduce the clock offset in the lab environment.

**Note:** Virtual Cisco IOSvL2 switches in GNS3 can experience clock drift issues. If the switch shows as unsynchronized after configuring NTP try restarting the chrony service on Ubuntu-Infra-Server using 'sudo systemctl restart chrony'. Then wait a few minutes for it to sync to the switches. If that does not work then try saving configuration on switches using 'do write' and safely shut down ubuntu servers using 'sudo shutdown now'. Wait a few minutes before turning them back on. Then wait a few minutes for everything to load correctly before verifying NTP with 'show ntp status'. **RAM and CPU pressure on the host machine can also cause inconsistencies with the synchronization of NTP.** On real devices NTP will work more consistently than in a virtual environment. A full restart of the host machine helped resolve NTP synchronization issues during troubleshooting of this issue.

### L3-Multilayer-SW1
```
enable
configure terminal

ntp server 172.16.0.5 minpoll 4 maxpoll 6 prefer
do write
```

![](images/ntpswitchconfig.PNG)

**Note:** I am only showing the configuration of L3-Multilayer-SW1, as the commands are the same on each switch.

### L3-Multilayer-SW2
```
enable
configure terminal

ntp server 172.16.0.5 minpoll 4 maxpoll 6 prefer
do write
```

### L2-SW1
```
enable
configure terminal

ntp server 172.16.0.5 minpoll 4 maxpoll 6 prefer
do write
```

### L2-SW2
```
enable
configure terminal

ntp server 172.16.0.5 minpoll 4 maxpoll 6 prefer
do write
```

### L2-SW3
```
enable
configure terminal

ntp server 172.16.0.5 minpoll 4 maxpoll 6 prefer
do write
```

### Verify NTP on each switch

Verify NTP is syncing and the reference clock shows the IP of Ubuntu-Infra-Server (172.16.0.5).

Use the commands:
```
show ntp status
show ntp associations
```
**L3-Multilayer-SW1:**

![](images/verifyntpimg1.PNG)

**L2-SW1:**

![](images/verifyntpimg2.PNG)

<br>

## Configuring NTP on pfSense

We will configure pfSense to use Ubuntu-Infra-Server as the NTP source.

<br>

Access the pfSense webConfigurator on your browser with the default credentials.

Username: admin

Password: pfsense

### Configure NTP

- Go to Services → NTP
- Leave Enable NTP Server checked
- Under Time Servers, clear the existing 2.pfsense.pool.ntp.org entry
- Enter 172.16.0.5 in the Time Servers field
- Set Type to Server
- Check Prefer
- Leave all other settings as default
- Click Save

![](images/pfsensentpconfigimg.PNG)

### Verify NTP on pfSense

Wait a few minutes after configuring for everything to sync, then:

- Go to Status → NTP

The status should show Active Peer and the Server should show 172.16.0.5.

![](images/verifypfsensentpimg.PNG)

<br>

## Installing and Configuring apache2 web server on Ubuntu-Infra-Server

We will install apache2 and configure a simple webpage to test connectivity to the http server.

<br>

### Install apache2

On Ubuntu-Infra-Server, install apache2 using the command:
```
sudo apt update
sudo apt install apache2 -y
```
**Note:** If no disk space is left run the command 'sudo apt clean' to free up some space.

### Local test

Since we do not have access to a browser on any device in GNS3, we will be using 'curl' to test if we can access the web page.

<br>

Check if apache2 is running using the commands
```
sudo systemctl status apache2 --no-pager -l
sudo systemctl is-enabled apache2
```

![](images/localtestapache2img1.PNG)

On Ubuntu-Infra-Server, run the command:
```
curl http://172.16.0.5
```

The command should return the HTML content of the apache2 default page, and confirm apache2 is serving the page correctly.

![](images/localtestapache2img2.PNG)

### Create a simple webpage to replace the default page

Replacing the default page will make it easy to know if we are reaching the correct server.

Create the webpage using the command:
```
sudo nano /var/www/html/index.html
```

Then replace the contents of that file with:
```
<!DOCTYPE html>
<html>
<head>
    <title>ECorp Web Page</title>
</head>
<body>
    <h1>Project GNS3 Mid-Size Company Network</h1>
    <p>This page is served from Ubuntu-Infra-Server at 172.16.0.5</p>
</body>
</html>
```
Then save with Ctrl+X, then y, then enter.

![](images/apache2newwebpageimg.PNG)

### Start and enable apache2

Start apache2 and enable it to run on startup using the commands:
```
sudo systemctl start apache2
sudo systemctl enable apache2
```

Verify it is running using:
```
sudo systemctl status apache2 --no-pager -l
```

Then we can verify the web page was changed to the new simple page using the command:
```
curl http://172.16.0.5
```

![](images/verifyapache2newpageimg.PNG)

<br>

## Verification

After DNS, NTP, and HTTP are configured, we can verify that they work using simple verification commands. We can also verify that the department workstations can access Ubuntu-Infra-Server using a ping test. Save and restart all devices if CPU usage is high to reset CPU usage and allow traffic to flow smoother.

### Verify DNS from Ubuntu-Admin-PC

On Ubuntu-Admin-PC we can verify forward DNS lookup is working by running the commands:
```
nslookup ubuntu-infra-server.ecorp.local
nslookup l3-multilayer-sw1.ecorp.local
nslookup google.com
```
The hostnames should resolve to the correct IP address.

Then we can verify reverse DNS lookup is working by running the commands:
```
nslookup 172.16.0.5
nslookup 192.168.99.2
```
The IP addresses should resolve back to the correct hostname.

![](images/verifydnsimg.PNG)

### Verify HTTP from Ubuntu-Admin-PC

Verify Ubuntu-Admin-PC can reach the HTTP server by running the command:
```
curl http://www.ecorp.local
```
The output should show the HTML of our new simple web page. This confirms that the HTTP server is reachable, that DNS resolution is working, and the CNAME record we set earlier is resolving.

![](images/verifyhttpserver.PNG)

### Verify Connectivity from department workstations

From any VPCS workstation run the commands:
```
ip dhcp
ping 172.16.0.5
```
A successful ping confirms department workstations can reach the Infrastructure Server and therefore access the services on it.

**Note:** Since VPCS have limited functionality we cannot directly confirm the functionality of the services from them.

![](images/verifyconnectivitypc1.PNG)

