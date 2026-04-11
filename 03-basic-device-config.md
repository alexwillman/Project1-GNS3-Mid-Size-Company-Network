# Basic Device Configuration

This section covers the base configuration that must be applied to all five switches before any other configuration takes place. The basic configuration includes setting the hostname, setting up passwords, enabling SSH, configuring a login banner, and shutting down unused ports.
The devices these are configured on are L3-Multilayer-SW1, L3-Multilayer-SW2, L2-SW1, L2-SW2, and L2-SW3.

## Credentials in the lab

You can use any password you would like for this lab. Common passwords are not recommended for security best practices in real environments, but for test environment purposes they are acceptable. However, it is recommended you use secure passwords to familiarize yourself with these practices. 

| Setting | Credential |
|---------|------------|
| Enable Secret | (your password) |
| Console Password | (your password) |
| VTY Password | (your password) |
| SSH Username | ecorpadmin | 
| SSH Password | (your password) |
| SSH Domain Name | ecorp.local |

**Note:** I will be using the same password across all devices for simplicity of the lab. A real environment would require unique passwords for each device, or a centralized authentication system like TACACS+ or RADIUS.

## Setting the Hostname 

The hostname is configured on each device to ensure you know which device you are currently configuring. The hostname should be configured on each device and match the topology.

### Commands:
```
enable
configure terminal
hostname L3-Multilayer-SW1
exit
write memory

```
**Note:** Write Memory can be shortened to write or wr. If you are in any configuration mode, the command must be 'do write'. This saves the configuration so it will stay after the device reloads.

![](images/hostnamecommandsimg.PNG)


| Device | Hostname |
|--------|----------|
| Layer 3 Core Switch 1 | L3-Multilayer-SW1 |
| Layer 3 Core Switch 2 | L3-Multilayer-SW2 |
| Layer 2 Access Switch 1 | L2-SW1 |
| Layer 2 Access Switch 2 | L2-SW2 |
| Layer 2 Access Switch 3 | L2-SW3 |

## Setting the Enable Secret

The enable secret command sets and encrypts the privileged EXEC password for the device. The console line password (next part) will not be encrypted so we will use service password-encryption to encrypt that. Apply to each switch.

You must be in global configuration mode to set this. (configure terminal)

### Commands: 

```
enable secret P@ssW0rd!
service password-encryption
exit
write memory
```
**Note:** "P@ssw0rd!" is the password of your choosing.

![](images/enablesecretimg.PNG)

Password is covered for security purposes.

## Configuring the Console Line

We will secure the console port using a password, set the session timeout to 10 minutes of inactivity, and disable syslog messages to avoid typing disruption. Apply this to each switch.

### Commands:

```
line console 0
password P@ssw0rd!
login
exec-timeout 10 0
logging synchronous
```
**Note:** "P@ssw0rd!" is the password of your choosing.

![](images/consolelineimg.PNG)

## Configuring SSH

SSH allows the secure remote access from the Ubuntu-Admin-PC. Privilege 15 grants full administrative access and secret encrypts the password so it is not stored in plaintext. Apply this to each switch to allow SSH access.

### Commands: 

```
ip domain-name ecorp.local
username ecorpadmin privilege 15 secret P@ssw0rd!
crypto key generate rsa (choose 2048 bits when prompted)
ip ssh version 2
```
**Then configure the VTY lines for SSH:**

```
line vty 0 4
login local
transport input ssh
exit
do write
```

![](images/sshimg.PNG)

## Configuring the Login Banner

The banner is displayed before you login to the device and is a legal warning to display that access is for authorized users only. You should never use welcoming language. Apply this to each switch.

### Commands: 

```
banner motd # AUTHORIZED USERS ONLY. UNAUTHORIZED ACCESS IS PROHIBITED! #
```
**Note:** The message should begin and end with a special character

To confirm that this worked, log out of the switch and before you enter your password you should see the banner.

![](images/bannermotdimg.PNG)

## Shutting down unused ports

For security purposes, unused ports are administratively shut down and put into an unused VLAN so unauthorized devices cannot gain access from that unused port. This is applied to each unused port on each switch.

**Create the black hole VLAN:**
```
vlan 999
name BLACK-HOLE
exit
```
**Putting the unused ports into the black hole VLAN and shut them down:**

Identify which ports are not being used and include them. [x/x - x/x] is the port range. Do not include the brackets.
```
interface range GigabitEthernet[x/x - x/x]
switchport mode access
switchport access vlan 999
shutdown
exit
write
```
**Note:** We will remove vlan 999 from the allowed vlan list in section 04.

