# Addressing and VLAN Inventory

## Network IP Plan

| VLAN | Purpose | Network | Mask | Gateway | DHCP Range |
|---|---|---|---|---|---|
| 10 | Employee | 10.10.10.0 | 255.255.255.0 | 10.10.10.254 | 10.10.10.100–.200 |
| 20 | Voice | 10.10.20.0 | 255.255.255.0 | 10.10.20.254 | 10.10.20.100–.200 |
| 30 | Guest | 10.10.30.0 | 255.255.255.0 | 10.10.30.254 | 10.10.30.100–.200 |
| 40 | Servers | 10.10.40.0 | 255.255.255.0 | 10.10.40.254 | Static only |
| 99 | Management | 10.10.99.0 | 255.255.255.0 | 10.10.99.254 | Static only |
| 100 | WAN | 10.10.100.0 | 255.255.255.252 | .1 | Point-to-point |

## Infrastructure

| Hostname | Role | Interface/VLAN | Address |
|---|---|---|---|
| CORE-1 | Core | VLAN 10 | 10.10.10.252 |
| CORE-1 | Core | VLAN 20 | 10.10.20.252 |
| CORE-1 | Core | VLAN 30 | 10.10.30.252 |
| CORE-1 | Core | VLAN 40 | 10.10.40.252 |
| CORE-1 | Core | VLAN 99 | 10.10.99.252 |
| CORE-2 | Core | VLAN 10 | 10.10.10.253 |
| CORE-2 | Core | VLAN 20 | 10.10.20.253 |
| CORE-2 | Core | VLAN 30 | 10.10.30.253 |
| CORE-2 | Core | VLAN 40 | 10.10.40.253 |
| CORE-2 | Core | VLAN 99 | 10.10.99.253 |
| ACCESS-1 | Access | VLAN 99 | 10.10.99.11 |
| ACCESS-2 | Access | VLAN 99 | 10.10.99.12 |
| SRV-DNS-DHCP | Server | VLAN 40 | 10.10.40.10 |
| SRV-WEB | Server | VLAN 40 | 10.10.40.11 |
| EDGE-R1 | WAN | gi0/0/0 | 10.10.100.1 |
| EDGE-R1 | Internet | gi0/0/1 | 203.0.113.2 |

## Reservation Guidelines

- `.1–.99`: infrastructure and static servers
- `.100–.200`: dynamic endpoint pool
- `.201–.254`: spare/reserved pool

## Public Address Notes

The `203.0.113.0/24` range is reserved for documentation and can be used in Packet Tracer. In a real deployment, replace it with the provider-assigned public subnet or implement a private lab with a routed external subnet.
