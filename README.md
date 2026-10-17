# Bank Network Implementation Project

A hands-on bank network implementation project built and documented in **GNS3** and on **real Cisco devices**. This project focuses on **static routing and default routing** across a bank LAN and multiple upstream routers. It does not use OSPF, EIGRP, or BGP in the routing configuration.

> **Important:** The GNS3 address plan below is read from the topology screenshot. The physical-rack/whiteboard labels are not all clear, so confirm real-device interface names, cabling, and IP addresses before applying commands.

## Project objectives

- Build a small bank LAN in GNS3 and reproduce the design on physical Cisco equipment.
- Configure router interfaces and verify direct connectivity.
- Use static routes to reach specific remote networks.
- Use default routes to forward traffic toward upstream routers.
- Provide DHCP addressing from R1 if required by the intended design.
- Validate routing and end-to-end connectivity with Cisco IOS show commands and ping tests.

## Topology overview

- **R1** — bank gateway / DHCP server label in the diagram
- **R2** — upstream router (JIO label)
- **R3** — upstream router (AIRTEL label)
- **R4** — upstream router (GOOGLE label)
- **Switch1** — bank LAN access switch
- **PC1 / PC2** — bank LAN hosts

## Address plan read from the GNS3 diagram

| Link / device | Interface (as shown) | IP address | Network |
|---|---|---|---|
| Bank LAN / R1 | R1 `f0/0` | `192.168.1.100/24` | `192.168.1.0/24` |
| Bank PC1 | PC1 `e0` | `192.168.1.1/24` | `192.168.1.0/24` |
| Bank PC2 | PC2 `e0` | `192.168.1.2/24` | `192.168.1.0/24` |
| R1–R2 | R1 `g1/0` | `192.168.12.1/24` | `192.168.12.0/24` |
| R1–R2 | R2 `g2/0` | `192.168.12.2/24` | `192.168.12.0/24` |
| R1–R3 | R1 `g2/0` | `192.168.13.1/24` | `192.168.13.0/24` |
| R1–R3 | R3 `g1/0` | `192.168.13.3/24` | `192.168.13.0/24` |
| R2–R4 | R2 `g1/0` | `192.168.24.2/24` | `192.168.24.0/24` |
| R2–R4 | R4 `g2/0` | `192.168.24.4/24` | `192.168.24.0/24` |
| R3–R4 | R3 `g2/0` | `192.168.34.3/24` | `192.168.34.0/24` |
| R3–R4 | R4 `g1/0` | `192.168.34.4/24` | `192.168.34.0/24` |

The `/24` masks are inferred from the network labels in the screenshot. Confirm all interface mappings with `show ip interface brief`.

## Routing design

This repository uses **static routes and default routes**.

- **Static route:** manually defines a route to a particular destination network.
- **Default route:** sends traffic for destinations that do not match a more specific route to a next-hop router.
- The sample command file uses R1 with a primary default route through R2 and a floating backup default route through R3. R2 and R3 forward unknown destinations to R4. R4 has routes back to the bank LAN and the two R1-to-upstream transit networks.
- The R2/R3/R4 routing choices are lab examples; adjust them to match the actual intended upstream design.

## Suggested repository structure

```text
bank-network-implementation/
├── README.md
├── configs/
│   └── commands.md
├── diagrams/
│   ├── gns3-topology.png
│   ├── physical-rack.jpg
│   └── whiteboard-topology.jpg
└── evidence/
    └── verification-output.md
```

## Verification checklist

- `show ip interface brief` — interfaces are addressed and up/up.
- `show running-config` — confirm the applied configuration.
- `show ip route` — verify connected (`C`), local (`L`), static (`S`) and candidate default (`S*`) routes.
- `ping <directly-connected-neighbor>` — verify each router link.
- `traceroute <destination>` — observe the forwarding path.
- On VPCS: `show ip`, then ping `192.168.1.100`.

## Safety and implementation notes

- Commands in `configs/commands.md` are templates, not a production-ready configuration.
- Interface names vary between GNS3 images and physical Cisco models. Substitute the actual interface names.
- `8.8.8.8` is a public DNS destination, not an interface address to assign to R4. Successful Internet access requires a real upstream connection and a valid return path.
- For physical bank equipment, back up configurations, confirm cabling and addressing, use a maintenance window, and prepare a rollback plan.
- Do not publish credentials, SNMP communities, keys, public IPs, serial numbers, or customer data.

## GitHub repository description

Bank Network Implementation using GNS3 and Real Cisco Devices | Static Routing, Default Routing, Bank LAN Connectivity, DHCP, and Network Troubleshooting.
