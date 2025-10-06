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

When the lab is deployed with the default startup config, all the links are created with IPv4 and IPv6 addresses.

This allows to start configuring bgp right away.

Here's a summary of what is included in the startup config:

- Configure interfaces between Leaf & Spine
- Configure interface between Leaf & Client
- Configure system loopback
- Configure default Network Instance (VRF) and add system loopback and Leaf/Spine interfaces to this VRF
- Configure IPs and static routes on Clients

Check the [startup config](n95-bgp/configs/fabric/startup) files to see how these objects are configured in SR Linux.

To view Interface status on SR Linux use:

```srl
show interface
```

### IPv4 Link Addressing

![image](images/lab-ipv4-2.jpg)

### Verify reachability between devices

After the lab is deployed, check reachability between leaf and spine devices using ping.

Example on spine to Leaf1 using IPv4:

```srl
ping -c 3 192.168.10.2 network-instance default
```

Example on spine to Leaf1 using IPv6:

```srl
ping6 -c 3 192:168:10::2 network-instance default
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

