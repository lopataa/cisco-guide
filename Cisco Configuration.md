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
 - `vlan`
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
 - (only on L3 switches) `switchport trunk encapsulation dot1q` - enabling encapsulation is required
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

## Spanning-tree protocol (STP)

### Costs (short path cost)

Cost is used to calculate the optimal path when building the spanning tree map.

*in a interface configuration ctx*
- `spanning tree cost 10`

| speed     | cost |
|-----------|------|
| 10 mbps   | 100  |
| 100 mbps  | 19   |
| 1 gbps    | 4    |
| 10 gbps   | 2    |
| 100 gbps  | 1    |


### Priority




### Portfast

On a interface, where no switch or other network device (switch, router etc.) will be connected, you can set the interface to portfast mode, where any STP discovery is not needed and takes a long time.

*in a interface configuration ctx*
- `spanning-tree portfast`

#### BDPU Guard

This protects a port by checking for bpdu packets, and automatically administratively disables the port, if a network device (switch, router etc.) is connected. 

*in a interface configuration ctx*
- `spanning-tree bpduguard enable`

## DHCP Server

- `ip dhcp excluded-address <range-from> [range-to]`
- `ip dhcp <pool-name>`
- 

## Access Control List (ACL)

1. set the acl rule
   - `access-list <acl> <permit|deny> <service> <src-network-addr> <src-mask> <dst-network-addr> <dst-mask>`
   - `access-list 110    deny          ip        192.168.10.0       0.0.0.255  192.168.20.0       0.0.0.255`
   - `access-list 110    permit        ip        any                any        any                any`

2. apply the rule to set vlan
   - `int vlan 10`
   - `int access-group 110 in`
   - `exit`


## HSRP (Cisco Hot Standby Router Protocol)

1. specify the HSRP version
    - `standby version 2` (two is the latest)

2. set the virtual gateway
    - `standby <group-number> ip <ip>`
    - `standby 1 ip 192.168.1.254`

3. set the router's priority, 100's default
    - `standby 1 priority 150`

4. If it's desirable that the active router resumes its role, when it becomes available again
    - `standby 1 preempt`

After configuring, make sure devices' default gateway is set to the HSRP ip.