# Bank Network Implementation — Static and Default Routing Commands

These are **lab templates** based on the GNS3 topology screenshot. Confirm interface names and IP addresses before applying. This version uses **static routes and default routes only**; it does not configure OSPF, EIGRP, or BGP.

## 1. Check the router interfaces first

Run on each router:

```text
enable
show version
show ip interface brief
show cdp neighbors
show running-config
```

Use the interface names shown by your actual router. The names below follow the GNS3 diagram labels.

## 2. R1 — Bank gateway

```text
enable
configure terminal
hostname R1
!
interface FastEthernet0/0
 description BANK-LAN-to-Switch1
 ip address 192.168.1.100 255.255.255.0
 no shutdown
!
interface GigabitEthernet1/0
 description LINK-to-R2
 ip address 192.168.12.1 255.255.255.0
 no shutdown
!
interface GigabitEthernet2/0
 description LINK-to-R3
 ip address 192.168.13.1 255.255.255.0
 no shutdown
!
! Primary default route via R2
ip route 0.0.0.0 0.0.0.0 192.168.12.2
! Floating backup default route via R3 (higher administrative distance)
ip route 0.0.0.0 0.0.0.0 192.168.13.3 10
end
```

The floating static route is used only if the primary default route is removed from the routing table. This does not automatically verify that the Internet behind R2 is working; additional tracking may be needed for link/path failure detection.

## 3. R2 — Upstream router

```text
enable
configure terminal
hostname R2
!
interface GigabitEthernet2/0
 description LINK-to-R1
 ip address 192.168.12.2 255.255.255.0
 no shutdown
!
interface GigabitEthernet1/0
 description LINK-to-R4
 ip address 192.168.24.2 255.255.255.0
 no shutdown
!
! Return route to bank LAN via R1
ip route 192.168.1.0 255.255.255.0 192.168.12.1
! Forward unknown destinations toward R4
ip route 0.0.0.0 0.0.0.0 192.168.24.4
end
```

## 4. R3 — Upstream router

```text
enable
configure terminal
hostname R3
!
interface GigabitEthernet1/0
 description LINK-to-R1
 ip address 192.168.13.3 255.255.255.0
 no shutdown
!
interface GigabitEthernet2/0
 description LINK-to-R4
 ip address 192.168.34.3 255.255.255.0
 no shutdown
!
! Return route to bank LAN via R1
ip route 192.168.1.0 255.255.255.0 192.168.13.1
! Forward unknown destinations toward R4
ip route 0.0.0.0 0.0.0.0 192.168.34.4
end
```

## 5. R4 — Upstream router

```text
enable
configure terminal
hostname R4
!
interface GigabitEthernet2/0
 description LINK-to-R2
 ip address 192.168.24.4 255.255.255.0
 no shutdown
!
interface GigabitEthernet1/0
 description LINK-to-R3
 ip address 192.168.34.4 255.255.255.0
 no shutdown
!
! Routes back toward R1 networks through R2
ip route 192.168.1.0 255.255.255.0 192.168.24.2
ip route 192.168.12.0 255.255.255.0 192.168.24.2
ip route 192.168.13.0 255.255.255.0 192.168.24.2
end
```

**Important routing note:** These R4 routes prefer R2 as the return path. If R2 fails, return traffic through R3 will need floating static routes or another explicitly designed failover method. Add a second route with a higher administrative distance if you want a backup path, for example:

```text
ip route 192.168.1.0 255.255.255.0 192.168.34.3 10
ip route 192.168.12.0 255.255.255.0 192.168.34.3 10
ip route 192.168.13.0 255.255.255.0 192.168.34.3 10
```

## 6. Optional DHCP service on R1

Use this only if R1 is intended to hand out addresses to bank PCs. The diagram labels PC1 and PC2 with static addresses `.1` and `.2`; those addresses should not be leased to other devices.

```text
configure terminal
ip dhcp excluded-address 192.168.1.1 192.168.1.20
ip dhcp excluded-address 192.168.1.100 192.168.1.110
ip dhcp pool BANK-LAN
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.100
 dns-server 8.8.8.8
end
```

## 7. Bank switch — simple access ports

The screenshot does not specify VLANs or exact switch ports. Confirm the ports before using this template.

```text
enable
configure terminal
hostname Switch1
interface range FastEthernet0/1 - 2
 description BANK-PC-ACCESS-PORTS
 switchport mode access
 spanning-tree portfast
 no shutdown
end
```

## 8. VPCS commands

Static settings shown in the GNS3 topology:

PC1:
```text
ip 192.168.1.1/24 192.168.1.100
save
show ip
ping 192.168.1.100
```

PC2:
```text
ip 192.168.1.2/24 192.168.1.100
save
show ip
ping 192.168.1.100
```

To use DHCP instead, run `ip dhcp` on each VPCS after enabling the pool on R1.

## 9. Verification and troubleshooting

On routers:

```text
show ip interface brief
show ip route
show running-config
ping 192.168.12.2
ping 192.168.13.3
traceroute 192.168.24.4
```

Check:
1. Both ends of every link are in the same subnet.
2. Interfaces are `up/up` and not administratively down.
3. Each router can ping its directly connected neighbor.
4. Static routes point to a reachable next-hop address.
5. The return path exists on remote routers.
6. Bank PCs use subnet mask `255.255.255.0` and default gateway `192.168.1.100`.

Useful route codes:
- `C` — connected network
- `L` — local interface address
- `S` — static route
- `S*` — candidate default static route

Save after verification:

```text
copy running-config startup-config
```

## 10. Real Cisco devices

Do not paste this directly onto physical devices without mapping the cables and ports. First gather:

```text
show version
show inventory
show ip interface brief
show cdp neighbors detail
show running-config
```

The physical rack and whiteboard photos do not clearly expose every model, port, and address. Back up the current configuration, adapt the templates to the actual hardware, and use an approved maintenance window and rollback plan.
