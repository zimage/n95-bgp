# Welcome to the BGP for DC Workshop at NANOG 95

This README is your starting point into the hands on section.

Please contact [**Reda Laichi**](https://www.linkedin.com/in/reda-l-5b28292) or [**Saju Salahudeen**](https://www.linkedin.com/in/saju-salahudeen) if you have any questions.

Pre-requisite: A laptop with SSH client

Shortcut links to major sections in this README:

|   |   |
|---|---|
| [Lab Topology](#lab-topology) | [Deploying the lab](#deploying-the-lab) |
| [CLI Quick Reference](#sr-linux-configuration-mode) |

## Lab Environment

A Nokia team member will provide you with a card that contains:
- your VM hostname
- SSH credentials to the VM instance
- URL of this repo

> <p style="color:red">!!! Make sure to backup any code, config, ... <u> offline (e.g on your laptop)</u>. 
> The VM instances will be destroyed once the Workshop is concluded.</p>

---
<div align=center>
<a href="https://codespaces.new/srlinuxamericas/n95-bgp?quickstart=1">
<img src="https://gitlab.com/rdodin/pics/-/wikis/uploads/d78a6f9f6869b3ac3c286928dd52fa08/run_in_codespaces-v1.svg?sanitize=true" style="width:50%"/></a>

**[Run](https://codespaces.new/srlinuxamericas/n95-bgp?quickstart=1) this lab in GitHub Codespaces for free**.  
[Learn more](https://containerlab.dev/manual/codespaces/) about Containerlab for Codespaces.

</div>

---

## Workshop
The objective of the hands on section of this workshop is the following:
- Configure BGP peering
- Configure BGP attributes
- Configure Route Policies
- Configure EVPN

## Lab Topology

Each workshop participant will be provided with the below topology consisting of 2 leaf and 2 spine nodes along with 4 clients.

![image](images/lab-topology.jpg)

## NOS (Network Operating System)

All leafs and Spines will be running the latest Nokia [SR Linux](https://www.nokia.com/networks/ip-networks/service-router-linux-NOS/) release 25.7.2.

All 4 clients will be running [Alpine Linux](https://alpinelinux.org/)

## Deploying the lab

Install Containerlab on your VM.

```bash
curl -sL https://containerlab.dev/setup | sudo -E bash -s "all"
```

Logout and login for the sudo privileges to take effect.

Use the below command to clone this repo to your VM.

```bash
sudo git clone https://github.com/srlinuxamericas/n95-bgp.git
```

Verify that the git repo files are available on your VM.

```bash
ls -lrt n95-bgp
```

To deploy the lab, run the following:

```bash
cd n95-bgp
sudo clab deploy -t srl-bgp.clab.yml
```

[Containerlab](https://containerlab.dev/) will deploy the lab and display a table with the list of nodes and their IPs.

```bash
14:29:46 INFO Adding host entries path=/etc/hosts
14:29:46 INFO Adding SSH config for nodes path=/etc/ssh/ssh_config.d/clab-srl-evpn.conf
╭─────────┬──────────────────────────────┬─────────┬────────────────────╮
│   Name  │          Kind/Image          │  State  │   IPv4/6 Address   │
├─────────┼──────────────────────────────┼─────────┼────────────────────┤
│ client1 │ linux                        │ running │ 172.20.20.10       │
│         │ ghcr.io/srl-labs/alpine      │         │ 2001:172:20:20::10 │
├─────────┼──────────────────────────────┼─────────┼────────────────────┤
│ client2 │ linux                        │ running │ 172.20.20.11       │
│         │ ghcr.io/srl-labs/alpine      │         │ 2001:172:20:20::11 │
├─────────┼──────────────────────────────┼─────────┼────────────────────┤
│ client3 │ linux                        │ running │ 172.20.20.12       │
│         │ ghcr.io/srl-labs/alpine      │         │ 2001:172:20:20::12 │
├─────────┼──────────────────────────────┼─────────┼────────────────────┤
│ client4 │ linux                        │ running │ 172.20.20.13       │
│         │ ghcr.io/srl-labs/alpine      │         │ 2001:172:20:20::13 │
├─────────┼──────────────────────────────┼─────────┼────────────────────┤
│ leaf1   │ nokia_srlinux                │ running │ 172.20.20.2        │
│         │ ghcr.io/nokia/srlinux:25.3.2 │         │ 2001:172:20:20::2  │
├─────────┼──────────────────────────────┼─────────┼────────────────────┤
│ leaf2   │ nokia_srlinux                │ running │ 172.20.20.4        │
│         │ ghcr.io/nokia/srlinux:25.3.2 │         │ 2001:172:20:20::4  │
├─────────┼──────────────────────────────┼─────────┼────────────────────┤
│ spine   │ nokia_srlinux                │ running │ 172.20.20.3        │
│         │ ghcr.io/nokia/srlinux:25.3.2 │         │ 2001:172:20:20::3  │
╰─────────┴──────────────────────────────┴─────────┴────────────────────╯
```

To display all deployed labs on your VM at any time, use:

```bash
sudo clab inspect --all
```

## Connecting to the devices

Find the nodename or IP address of the device from the above output and then use SSH.

Username: `admin`

Password: Refer to the provided card

```bash
ssh admin@leaf1
```

To login to the client, identify the client hostname using the `sudo clab inspect --all` command above and then:

```bash
sudo docker exec –it client3 sh
```

## Physical link connectivity

When the lab is deployed with the default startup config, all the links between leafs and clients are configured by containerlab.

This allows to start configuring bgp right away.

Here's a summary of what is included in the startup config:

- Configure interface between Leaf & Client
- Configure system loopback
- Configure default Network Instance (VRF) and add system loopback and Leaf/Client interfaces to this VRF
- Configure IPs and static routes on Clients

Check the [startup config](n95-bgp/configs/fabric/startup) files to see how these objects are configured in SR Linux.

To view Interface status on SR Linux use:

```srl
show interface
```

### IPv4 Link Addressing

![image](images/lab-ipv4-2.jpg)

### Verify reachability between leaf and client

After the lab is deployed, check reachability between leaf and client devices using ping.

Example on Leaf1 to Client1:

```srl
ping -c 3 172.16.10.50 network-instance default
```

## SR Linux Configuration Mode

To enter candidate configuration edit mode in SR Linux, use:

```srl
enter candidate
```

To commit the configuration in SR Linux, use:

```srl
commit stay
```

Here's a reference table with some commonly used commands.

| Action | Command |
| --- | --- |
| Enter Candidate mode | `enter candidate {private}` |
| Commit configuration changes | `commit {now\|stay}` |
| | `now` – commits and exits from candidate mode |
| | `stay` – commits and stays in candidate mode |
| Delete configuration elements | `delete` |
| | Eg: `delete interface ethernet-1/5` |
| Discard configuration changes | `discard {now\|stay}` |
| Compare candidate to running | `diff running /` |
| View configuration in current mode & context | `info {flat}` |
| View configuration in another mode & context | `info {flat} from state /interface ethernet-1/1` |
| Output modifiers | `<command> \| as {table\|json\|yaml}` |
| Access Linux shell | `bash` |
| Find a command | `tree flat detail \| grep <keyword>` |

## Configure BGP peering

We are now ready to start configuring BGP.

The first step is to establish BGP peering sessions between the devices.

### Autonomous System (AS) number

For our lab topology:
- leaf1, client1 and client2 will be part of AS 64500
- leaf2, client3 and client4 will be part of AS 64600
- spine1 and spine2 will be part of AS 65500

### BGP peering between Leafs and Clients

We will use the dynamic peering method using IPv4 interface address to establish peering between leafs and clients.

All leaf interfaces towards the clients are configured with IPv4 address as part of startup config.

Client1 and Client3 are Layer2 and connects to the network using the `irb0` interface on each leaf.

To view Interface status on SR Linux use:

```srl
show interface
```

Dynamic BGP peering configuration on Leaf1 for clients:

```srl
set / network-instance default protocols bgp autonomous-system 64500
set / network-instance default protocols bgp router-id 1.1.1.1
set / network-instance default protocols bgp afi-safi ipv4-unicast admin-state enable
set / network-instance default protocols bgp dynamic-neighbors accept match 172.16.10.0/24 peer-group servers
set / network-instance default protocols bgp dynamic-neighbors accept match 172.16.10.0/24 allowed-peer-as [ 64500 ]
set / network-instance default protocols bgp dynamic-neighbors accept match 10.80.1.0/24 peer-group servers
set / network-instance default protocols bgp dynamic-neighbors accept match 10.80.1.0/24 allowed-peer-as [ 64500 ]
set / network-instance default protocols bgp group servers peer-as 64500
set / network-instance default protocols bgp group servers send-default-route ipv4-unicast true
```

Dynamic BGP peering configuration on Leaf2 for clients:

```srl
set / network-instance default protocols bgp autonomous-system 64600
set / network-instance default protocols bgp router-id 2.2.2.2
set / network-instance default protocols bgp afi-safi ipv4-unicast admin-state enable
set / network-instance default protocols bgp dynamic-neighbors accept match 10.90.1.0/24 peer-group servers
set / network-instance default protocols bgp dynamic-neighbors accept match 10.90.1.0/24 allowed-peer-as [ 64600 ]
set / network-instance default protocols bgp dynamic-neighbors accept match 172.17.10.0/24 peer-group servers
set / network-instance default protocols bgp dynamic-neighbors accept match 172.17.10.0/24 allowed-peer-as [ 64600 ]
set / network-instance default protocols bgp group servers peer-as 64600
set / network-instance default protocols bgp group servers send-default-route ipv4-unicast true
```

Verify that BGP peering sessions are 'established' between leafs and clients.

```srl
show network-instance default protocols bgp neighbor
```

Expected output on leaf1:

```srl
A:admin@leaf1# show network-instance default protocols bgp neighbor
-----------------------------------------------------------------------------------------------------------------------------------------------------
BGP neighbor summary for network-instance "default"
Flags: S static, D dynamic, L discovered by LLDP, B BFD enabled, - disabled, * slow
-----------------------------------------------------------------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------------------------------------------------------------
+----------------+------------------------+----------------+------+---------+-------------+-------------+------------+------------------------+
|    Net-Inst    |          Peer          |     Group      | Flag | Peer-AS |    State    |   Uptime    |  AFI/SAFI  |     [Rx/Active/Tx]     |
|                |                        |                |  s   |         |             |             |            |                        |
+================+========================+================+======+=========+=============+=============+============+========================+
| default        | 10.80.1.1              | servers        | D    | 64500   | established | 0d:1h:34m:9 | ipv4-      | [0/0/1]                |
|                |                        |                |      |         |             | s           | unicast    |                        |
| default        | 172.16.10.50           | servers        | D    | 64500   | established | 0d:1h:32m:4 | ipv4-      | [0/0/1]                |
|                |                        |                |      |         |             | 6s          | unicast    |                        |
+----------------+------------------------+----------------+------+---------+-------------+-------------+------------+------------------------+
-----------------------------------------------------------------------------------------------------------------------------------------------------
Summary:
0 configured neighbors, 0 configured sessions are established, 0 disabled peers
2 dynamic peers
```

### BGP peering between Leafs and Spines

Between leafs and spines, we will use the IPv6 Link Local Address (LLA) to form dynamic BGP peering sessions.

No manual IP configuration is required. However, the interfaces should be enabled for IPv6 and IPv6 RA (Router Advertisement) should be enabled.

Enabling IPv6 on Leaf1 and Leaf2 interfaces to Spine1 and Spine2:
(Copy and paste to both leafs)

```srl
set / interface ethernet-1/1 admin-state enable
set / interface ethernet-1/1 subinterface 0 ipv6 admin-state enable
set / interface ethernet-1/1 subinterface 0 ipv6 router-advertisement router-role admin-state enable
set / interface ethernet-1/2 admin-state enable
set / interface ethernet-1/2 subinterface 0 ipv6 admin-state enable
set / interface ethernet-1/2 subinterface 0 ipv6 router-advertisement router-role admin-state enable
set /network-instance default interface ethernet-1/1.0
set /network-instance default interface ethernet-1/2.0
```

Enabling IPv6 on Spine1 and Spine2 interfaces to Leaf1 and Leaf2:
(Copy and paste to both spines)

```srl
set / interface ethernet-1/1 admin-state enable
set / interface ethernet-1/1 subinterface 0 ipv6 admin-state enable
set / interface ethernet-1/1 subinterface 0 ipv6 router-advertisement router-role admin-state enable
set / interface ethernet-1/2 admin-state enable
set / interface ethernet-1/2 subinterface 0 ipv6 admin-state enable
set / interface ethernet-1/2 subinterface 0 ipv6 router-advertisement router-role admin-state enable
set /network-instance default interface ethernet-1/1.0
set /network-instance default interface ethernet-1/2.0
```

Verify interface status and check IPv6 Link Local Address (LLA).

```srl
show interface
```

Expected output on leaf1 (LLA address may be different on your setup):


```srl
A:admin@leaf1# show interface
=====================================================================================================================================================
ethernet-1/1 is up, speed 25G, type None
  ethernet-1/1.0 is up
    Network-instances:
    Encapsulation   : null
    Type            : routed
    IPv6 addr    : fe80::1844:4ff:feff:1/64 (link-layer, unknown)
-----------------------------------------------------------------------------------------------------------------------------------------------------
ethernet-1/2 is up, speed 25G, type None
  ethernet-1/2.0 is up
    Network-instances:
    Encapsulation   : null
    Type            : routed
    IPv6 addr    : fe80::1844:4ff:feff:2/64 (link-layer, unknown)
-----------------------------------------------------------------------------------------------------------------------------------------------------
```

Next we will configure BGP dynamic peering between leafs and spines.

Dynamic BGP peering configuration on leaf1 and leaf2 towards spines:
(Copy and paste to both leafs)

```srl
set / network-instance default protocols bgp group spines
set / network-instance default protocols bgp dynamic-neighbors interface ethernet-1/1.0 peer-group spines
set / network-instance default protocols bgp dynamic-neighbors interface ethernet-1/1.0 allowed-peer-as [ 65500 ]
set / network-instance default protocols bgp dynamic-neighbors interface ethernet-1/2.0 peer-group spines
set / network-instance default protocols bgp dynamic-neighbors interface ethernet-1/2.0 allowed-peer-as [ 65500 ]
```


Dynamic BGP peering configuration on spine1 towards leafs:

```srl
set / network-instance default protocols bgp autonomous-system 65500
set / network-instance default protocols bgp router-id 10.10.10.10
set / network-instance default protocols bgp dynamic-neighbors interface ethernet-1/1.0 peer-group leafs
set / network-instance default protocols bgp dynamic-neighbors interface ethernet-1/1.0 allowed-peer-as [ 64500 64600 ]
set / network-instance default protocols bgp dynamic-neighbors interface ethernet-1/2.0 peer-group leafs
set / network-instance default protocols bgp dynamic-neighbors interface ethernet-1/2.0 allowed-peer-as [ 64500 64600 ]
set / network-instance default protocols bgp afi-safi ipv4-unicast admin-state enable
set / network-instance default protocols bgp group leafs
```

Dynamic BGP peering configuration on spine2 towards leafs:

```srl
set / network-instance default protocols bgp autonomous-system 65500
set / network-instance default protocols bgp router-id 20.20.20.20
set / network-instance default protocols bgp dynamic-neighbors interface ethernet-1/1.0 peer-group leafs
set / network-instance default protocols bgp dynamic-neighbors interface ethernet-1/1.0 allowed-peer-as [ 64500 64600 ]
set / network-instance default protocols bgp dynamic-neighbors interface ethernet-1/2.0 peer-group leafs
set / network-instance default protocols bgp dynamic-neighbors interface ethernet-1/2.0 allowed-peer-as [ 64500 64600 ]
set / network-instance default protocols bgp afi-safi ipv4-unicast admin-state enable
set / network-instance default protocols bgp group leafs
```

Verify that BGP peering sessions are 'established' between leafs and spines.

```srl
show network-instance default protocols bgp neighbor
```

Expected output on spine1:

```srl
A:admin@spine1# show network-instance default protocols bgp neighbor
-----------------------------------------------------------------------------------------------------------------------------------------------------
BGP neighbor summary for network-instance "default"
Flags: S static, D dynamic, L discovered by LLDP, B BFD enabled, - disabled, * slow
-----------------------------------------------------------------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------------------------------------------------------------
+----------------+------------------------+----------------+------+---------+-------------+-------------+------------+------------------------+
|    Net-Inst    |          Peer          |     Group      | Flag | Peer-AS |    State    |   Uptime    |  AFI/SAFI  |     [Rx/Active/Tx]     |
|                |                        |                |  s   |         |             |             |            |                        |
+================+========================+================+======+=========+=============+=============+============+========================+
| default        | fe80::1844:4ff:feff:1% | leafs          | D    | 64500   | established | 0d:0h:2m:34 | ipv4-      | [0/0/0]                |
|                | ethernet-1/1.0         |                |      |         |             | s           | unicast    |                        |
| default        | fe80::18c3:5ff:feff:2% | leafs          | D    | 64600   | established | 0d:0h:2m:35 | ipv4-      | [0/0/0]                |
|                | ethernet-1/2.0         |                |      |         |             | s           | unicast    |                        |
+----------------+------------------------+----------------+------+---------+-------------+-------------+------------+------------------------+
-----------------------------------------------------------------------------------------------------------------------------------------------------
Summary:
0 configured neighbors, 0 configured sessions are established, 0 disabled peers
2 dynamic peers
```
