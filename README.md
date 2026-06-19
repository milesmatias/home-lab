# CCNA Packet Tracer Lab: PC to Simulated Internet

![Lab Type](https://img.shields.io/badge/Lab-CCNA-blue)
![Tool](https://img.shields.io/badge/Tool-Cisco%20Packet%20Tracer-green)
![Level](https://img.shields.io/badge/Level-Beginner-orange)
![Topic](https://img.shields.io/badge/Topic-NAT%20%7C%20Default%20Route%20%7C%20IPv4-purple)

## Overview

In this beginner-friendly CCNA lab, you will build a simple **LAN-to-Internet** topology in Cisco Packet Tracer.

The goal is to configure a PC, switch, router, and simulated ISP router so that **PC1 can ping 8.8.8.8**.

> In this lab, `8.8.8.8` is not the real Google DNS server. It is a loopback interface created on the ISP router to simulate an internet destination inside Packet Tracer.

## What You Will Learn

By completing this lab, you will practice:

- IPv4 addressing
- Subnet masks
- Default gateway configuration
- Cisco router interface configuration
- Static default routes
- NAT overload / PAT
- Basic Layer 2 switching
- End-to-end ping testing
- Structured troubleshooting
- Verification with Cisco show commands

## Lab Goal

At the end of the lab, PC1 should successfully ping `8.8.8.8`.

```text
PC1> ping 8.8.8.8

Reply from 8.8.8.8: bytes=32 time<1ms TTL=255
```

## Topology

```text
PC1  --->  SW1  --->  R1  --->  ISP  --->  Simulated Internet

PC1: 192.168.1.10
R1 LAN: 192.168.1.1
R1 WAN: 203.0.113.2
ISP WAN: 203.0.113.1
ISP Loopback: 8.8.8.8
```

## Devices Needed

| Device | Name | Notes |
|---|---|---|
| PC | PC1 | End-user device |
| Switch | SW1 | Cisco 2960 switch |
| Router | R1 | Main LAN router and NAT device |
| Router | ISP | Simulated ISP / internet router |
| Cables | Copper straight-through | Used to connect all devices |

## Physical Connections

| From Device | From Port | To Device | To Port |
|---|---|---|---|
| PC1 | FastEthernet0 | SW1 | FastEthernet0/1 |
| SW1 | FastEthernet0/24 | R1 | GigabitEthernet0/0 |
| R1 | GigabitEthernet0/1 | ISP | GigabitEthernet0/0 |

## IP Addressing Table

| Device | Interface | IP Address | Subnet Mask | Purpose |
|---|---|---|---|---|
| PC1 | NIC | 192.168.1.10 | 255.255.255.0 | End host |
| PC1 | Gateway | 192.168.1.1 | N/A | Default gateway |
| R1 | G0/0 | 192.168.1.1 | 255.255.255.0 | LAN side |
| R1 | G0/1 | 203.0.113.2 | 255.255.255.252 | WAN side |
| ISP | G0/0 | 203.0.113.1 | 255.255.255.252 | R1-facing side |
| ISP | Loopback0 | 8.8.8.8 | 255.255.255.255 | Simulated internet |

---

# Step 1: Build the Packet Tracer Topology

Open Cisco Packet Tracer and add the following devices:

- 1 PC
- 1 Cisco 2960 switch
- 2 Cisco routers, such as 2911 or 1941 routers

Rename the devices:

```text
PC  -> PC1
Switch -> SW1
Router 1 -> R1
Router 2 -> ISP
```

Connect the devices using copper straight-through cables:

```text
PC1 FastEthernet0       -> SW1 FastEthernet0/1
SW1 FastEthernet0/24    -> R1 GigabitEthernet0/0
R1 GigabitEthernet0/1   -> ISP GigabitEthernet0/0
```

## Why the Switch Needs No Configuration

For this beginner version, the switch is only acting as a basic Layer 2 access switch.

PC1 and R1 are in the same default VLAN, so the switch can forward frames between them without extra configuration.

---

# Step 2: Configure PC1

Click **PC1**.

Go to:

```text
Desktop > IP Configuration
```

Enter the following settings:

```text
IP Address:      192.168.1.10
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.1.1
DNS Server:      8.8.8.8
```

## Why This Matters

The default gateway is one of the most important settings on an end device.

PC1 uses `192.168.1.1` when it needs to reach anything outside its local network of `192.168.1.0/24`.

---

# Step 3: Configure R1

Click **R1** and open the CLI.

Enter the following configuration:

```cisco
enable
configure terminal

hostname R1

interface gigabitEthernet0/0
 description LAN to SW1
 ip address 192.168.1.1 255.255.255.0
 ip nat inside
 no shutdown
exit

interface gigabitEthernet0/1
 description WAN to ISP
 ip address 203.0.113.2 255.255.255.252
 ip nat outside
 no shutdown
exit

access-list 1 permit 192.168.1.0 0.0.0.255

ip nat inside source list 1 interface gigabitEthernet0/1 overload

ip route 0.0.0.0 0.0.0.0 203.0.113.1

end
write memory
```

## R1 Command Explanation

| Command | Meaning |
|---|---|
| `ip address 192.168.1.1 255.255.255.0` | Makes R1 the default gateway for the LAN |
| `ip nat inside` | Marks the LAN interface as the inside/private side |
| `ip address 203.0.113.2 255.255.255.252` | Gives R1 a WAN IP toward the ISP router |
| `ip nat outside` | Marks the WAN interface as the outside/public side |
| `access-list 1 permit 192.168.1.0 0.0.0.255` | Identifies the LAN subnet that is allowed to be translated |
| `ip nat inside source list 1 interface gigabitEthernet0/1 overload` | Enables NAT overload/PAT |
| `ip route 0.0.0.0 0.0.0.0 203.0.113.1` | Sends unknown traffic toward the ISP router |

## CCNA Note

The default route is often called the **gateway of last resort**.

R1 uses this route when the destination network is not already in its routing table.

---

# Step 4: Configure the ISP Router

Click **ISP** and open the CLI.

Enter the following configuration:

```cisco
enable
configure terminal

hostname ISP

interface gigabitEthernet0/0
 description WAN to R1
 ip address 203.0.113.1 255.255.255.252
 no shutdown
exit

interface loopback0
 description Simulated Internet 8.8.8.8
 ip address 8.8.8.8 255.255.255.255
exit

end
write memory
```

## Why Use a Loopback for 8.8.8.8?

A loopback interface is a virtual router interface.

It stays up as long as the router is working, which makes it a stable test target for a simulated internet destination.

## Do We Need a Route Back to PC1?

For this NAT version, no extra route back to PC1 is needed on the ISP router.

R1 translates PC1's source IP from `192.168.1.10` to `203.0.113.2`. The ISP router only sees traffic coming from R1's WAN IP, so replies go back to `203.0.113.2`. R1 then reverses the NAT translation and sends the reply back to PC1.

---

# Step 5: Test the Lab

## Test 1: Ping the Default Gateway

On PC1, open the Command Prompt and run:

```text
ping 192.168.1.1
```

If this works, PC1 can reach R1's LAN interface.

## Test 2: Ping the ISP Router

On PC1, run:

```text
ping 203.0.113.1
```

If this works, PC1 can leave its local network and reach the R1-to-ISP link.

## Test 3: Ping the Simulated Internet

On PC1, run:

```text
ping 8.8.8.8
```

This is the final proof that the lab works.

If PC1 can ping `8.8.8.8`, your addressing, routing, default gateway, and NAT configuration are working.

---

# Verification Commands

Run these commands on R1 to verify your configuration:

```cisco
show ip interface brief
show ip route
show running-config | section interface
show running-config | include ip route
show access-lists
show ip nat translations
show ip nat statistics
```

## What to Look For

| Command | Good Sign | Problem Sign |
|---|---|---|
| `show ip interface brief` | G0/0 and G0/1 are up/up | Interface is administratively down or down/down |
| `show ip route` | Default route exists | No gateway of last resort |
| `show ip nat translations` | NAT translation appears after ping | No NAT entries |
| `show access-lists` | ACL 1 shows matches | ACL has zero matches |

---

# Troubleshooting Guide

## If PC1 Cannot Ping 192.168.1.1

Check the basics first:

- PC1 IP address
- PC1 subnet mask
- PC1 default gateway
- Cable from PC1 to SW1
- Cable from SW1 to R1
- R1 G0/0 interface status

On R1, run:

```cisco
show ip interface brief
```

Make sure G0/0 is `up/up`.

Also confirm that `no shutdown` was entered under R1 G0/0.

## If PC1 Can Ping 192.168.1.1 But Not 203.0.113.1

Check the WAN link:

- R1 G0/1 should be `203.0.113.2/30`
- ISP G0/0 should be `203.0.113.1/30`
- Both WAN interfaces should be `up/up`
- The cable between R1 and ISP should be connected

Run this command on both routers:

```cisco
show ip interface brief
```

## If PC1 Can Ping 203.0.113.1 But Not 8.8.8.8

Check the simulated internet target:

- ISP Loopback0 should exist
- ISP Loopback0 should use `8.8.8.8/32`
- R1 should have a default route to `203.0.113.1`

On R1, run:

```cisco
show ip route
```

Look for a default route like this:

```text
S* 0.0.0.0/0 [1/0] via 203.0.113.1
```

## If Ping Works But NAT Translations Do Not Show

Check NAT configuration on R1:

- G0/0 should have `ip nat inside`
- G0/1 should have `ip nat outside`
- ACL 1 should permit `192.168.1.0 0.0.0.255`
- NAT overload should use the correct outside interface

You can clear old NAT translations and test again:

```cisco
clear ip nat translation *
```

Then ping again from PC1:

```text
ping 8.8.8.8
```

Check NAT again:

```cisco
show ip nat translations
```

---

# Final Configurations

## R1 Final Configuration

```cisco
hostname R1
!
interface GigabitEthernet0/0
 description LAN to SW1
 ip address 192.168.1.1 255.255.255.0
 ip nat inside
 no shutdown
!
interface GigabitEthernet0/1
 description WAN to ISP
 ip address 203.0.113.2 255.255.255.252
 ip nat outside
 no shutdown
!
access-list 1 permit 192.168.1.0 0.0.0.255
!
ip nat inside source list 1 interface GigabitEthernet0/1 overload
!
ip route 0.0.0.0 0.0.0.0 203.0.113.1
```

## ISP Final Configuration

```cisco
hostname ISP
!
interface GigabitEthernet0/0
 description WAN to R1
 ip address 203.0.113.1 255.255.255.252
 no shutdown
!
interface Loopback0
 description Simulated Internet 8.8.8.8
 ip address 8.8.8.8 255.255.255.255
```

## PC1 Final Settings

```text
IP Address:      192.168.1.10
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.1.1
DNS Server:      8.8.8.8
```

---

# Student Checklist

Before calling the lab complete, verify each item:

- [ ] PC1 has the correct IP address
- [ ] PC1 has the correct subnet mask
- [ ] PC1 has the correct default gateway
- [ ] R1 G0/0 is configured and up/up
- [ ] R1 G0/1 is configured and up/up
- [ ] ISP G0/0 is configured and up/up
- [ ] ISP Loopback0 is configured as 8.8.8.8/32
- [ ] R1 has a default route to 203.0.113.1
- [ ] R1 has NAT inside on G0/0
- [ ] R1 has NAT outside on G0/1
- [ ] R1 has NAT overload configured
- [ ] PC1 can ping 192.168.1.1
- [ ] PC1 can ping 203.0.113.1
- [ ] PC1 can ping 8.8.8.8
- [ ] R1 shows NAT translations after the ping

---

# Skills Demonstrated

This lab demonstrates beginner CCNA-level skills including:

- IPv4 addressing
- Default gateways
- Static default routes
- Router interface configuration
- NAT overload/PAT
- Simulated ISP connectivity
- Packet Tracer topology building
- CLI verification
- Troubleshooting methodology

---

# Suggested YouTube Lab Flow

If you are following along with the video, use this flow:

| Section | What You Should Do |
|---|---|
| Intro | Understand the topology and goal |
| Build | Add devices and cable them |
| Addressing | Configure PC1 and router interfaces |
| Routing | Add the default route on R1 |
| NAT | Configure NAT overload on R1 |
| ISP | Configure the ISP router and loopback |
| Testing | Ping gateway, ISP, and 8.8.8.8 |
| Verification | Use show commands to prove the lab works |
| Troubleshooting | Fix issues in order from local to remote |

---

# Optional Next Labs

After completing this lab, try improving it with one of these upgrades:

- Add DHCP on R1 so PC1 receives an IP address automatically
- Add a second PC and verify both clients NAT through one public IP
- Add DNS simulation so PC1 can ping a hostname instead of only 8.8.8.8
- Add VLANs and router-on-a-stick
- Add an ACL to restrict which LAN hosts can reach the internet

---

# GitHub / Video Versioning

This guide is designed to match the YouTube walkthrough for this lab.

When the video is published, use a Git tag so viewers can follow the exact same version used in the recording.

Example:

```bash
git add README.md
git commit -m "Add CCNA PC to Internet Packet Tracer lab guide"
git tag -a ccna-pc-to-internet-v1 -m "YouTube version: CCNA PC to Internet Packet Tracer Lab"
git push origin main --tags
```

Then viewers can download the same version shown in the video:

```bash
git clone <your-repo-url>
git checkout ccna-pc-to-internet-v1
```

---

# Portfolio Summary

Built a Cisco Packet Tracer lab demonstrating LAN-to-WAN connectivity. Configured PC addressing, a Layer 2 switch path, Cisco router interfaces, NAT overload/PAT, a static default route, and a simulated internet destination using an ISP router loopback address of `8.8.8.8`. Verified the lab with ping tests, routing table checks, interface status, ACL matches, and NAT translation commands.


