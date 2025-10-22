# Cisco Configuration

version v1.1

## Basics

0. Connection

1. Hostname
 - `hostname <hostname>`

2. Turn off ip domain-lookup
 - `no ip domain-lookup`


## Security

1. Enable password encryption service
 - `service password-encryption`

2. Secure the priviliged mode
 - `enable password <password>`

3. (optional) Enforce password requirements
 - `security passwords min-length <min-length>`

4. Secure the console *(the physical console connection)*
 - `line console 0` 
 - `password <password>`
 - `login` *enables the password prompt*


## IP Configuration

1. Switch to the correct interface
 - `interface g0/0`

*in `interface <interface>` context*
2. Set the IP address and the subnet mask
 - `ip address <ip-address> <subnet-mask>`

*in `interface <interface>` context*
3. Enable the interface
 - `no shutdown`


## Accessing a different network

When accessing a remote network through another router, you have to configure network's router address, so the router knows how to access it.
1. Add it to the routing table
 - `ip route <network-address> <subnet-mask> <local-router-address>`


## SSH

1. Set a user's password
 - `username <username> password <password>`

2. Set the domain name
 - `ip domain-name <domain-name>`

3. Generate SSH RSA encryption keys
 - `crypto key generate rsa`
 You will be prompted for bit count, set __1024__ (required for SSH v2).

4. Enable SSH ftor virtual terminals.
 - `line vty 0 15`
 - `transport input ssh`
 - `login local`

5. Set the SSH version to v2.
 - `ip ssh version 2`

*in `line vty 0 15` context*
5. (optional) enable auto-logout on timeout
 - `exec-timeout <minutes> <seconds>`


## VLAN & VLANs across multiple switches

1. vlan creation
 - `1`
 - `name VLAN10`
 - `exit`

2. Open the interface range (or a single one)
 - `int range f0/1-10`
 - `switchport mode access`
 - `switchport access vlan 10`
 - `exit`

3. Inter-VLAN
 - `int g0/1` - the interface between the switches
 - `shutdown` - temporarily shut down the interface to avoid errors
 - `switchport trunk native vlan 99`
 - `exit`
 
 Configure both switches, then use `no shutdown` to enable the interfaces.

## Multi-layer switching & Inter-VLAN routing
  Before doing this guide, be sure to create the VLANs.
  To achieve multilayer switching, we have to set switch's ip address on each vlan first.

1. Set the IP and enable the interface
  - `int vlan 10`
  - `ip address 192.168.10.1 255.255.255.0`
  - `exit`

2. Enable inter-VLAN routing
  - `ip routing`
  
3. using a port as a "router" port, not switchport
  - `int g0/1`
  - `no switchport`
  *dont forget to set the ip on this port!*

## Access Control List (ACL)

1. set the acl rule
   - `access-list <acl> <permit|deny> <service> <src-network-addr> <src-mask> <dst-network-addr> <dst-mask>`
   - `access-list 110    deny          ip        192.168.10.0       0.0.0.255  192.168.20.0       0.0.0.255`
   - `access-list 110    permit        ip        any                any        any                any`

2. apply the rule to set vlan
   - `int vlan 10`
   - `int access-group 110 in`
   - `exit`
