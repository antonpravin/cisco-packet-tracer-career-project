# Secure Branch Office Network

A senior-network-engineer-style Cisco Packet Tracer lab that demonstrates a resilient, segmented branch network for a small company.

## Scenario

The organization operates a single branch office with 45 users. The network separates users, management, servers, and guest traffic while providing resilient switching, controlled internet access, centralized addressing, encrypted administration, and documented troubleshooting.

## Technologies

- VLAN and 802.1Q trunking
- Inter-VLAN routing on multilayer switches
- HSRP gateway redundancy
- EtherChannel (LACP)
- Rapid Per-VLAN Spanning Tree
- DHCP relay
- Centralized DNS and web services
- NAT on the edge router
- Standard and extended ACLs
- SSH-only device administration
- Syslog/NTP-ready architecture
- Point-to-point WAN addressing
- Guest isolation

## Logical Addressing

| VLAN | Purpose | Subnet | Gateway | DHCP Pool |
|---:|---|---|---|---|
| 10 | Employee | 10.10.10.0/24 | 10.10.10.254 | .100–.200 |
| 20 | Voice | 10.10.20.0/24 | 10.10.20.254 | .100–.200 |
| 30 | Guest | 10.10.30.0/0? 10.10.30.0/24 | 10.10.30.254 | .100–.200 |
| 40 | Servers | 10.10.40.0/24 | 10.10.40.254 | Static |
| 99 | Management | 10.10.99.0/24 | 10.10.99.254 | Static |
| 100 | WAN/Router | 10.10.100.0/30 | 10.10.100.1 | N/A |

> Production implementation: never use a **/0** subnet for a user VLAN. The guest subnet is explicitly `10.10.30.0/24`.

## Recommended Topology

- Internet router: `ISR4331` or compatible Packet Tracer router
- Internet: external router plus cloud/server
- Edge: `ISR4331` router
- Core switches: 2 × `3560-24PS` or `3650`
- Access switches: 2 × `2960X` or `2960`
- Endpoints: PCs, laptops, IP phones, printer, server

```text
Internet --- Edge Router --- Core1 === Core2
                              /  
                         Access1 Access2
```

## Equipment and Port Plan

| Device | Interface | Connection | Configuration |
|---|---|---|---|
| Core1 | Gi0/1–0/2 | Core2 | LACP EtherChannel |
| Core1 | Gi0/3 | Edge Router Gi0/0/0 | Routed VLAN 100 |
| Core1 | Fa0/1 | Access1 | 802.1Q trunk |
| Core1 | Fa0/2 | Access2 | 802.1Q trunk |
| Core2 | Gi0/1–0/2 | Core1 | LACP EtherChannel |
| Core2 | Gi0/3 | Edge Router Gi0/0/1 | Routed VLAN 100 |
| Core2 | Fa0/1 | Access1 | 802.1Q trunk |
| Core2 | Fa0/2 | Access2 | 802.1Q trunk |

Adjust interface names to match the devices available in your Packet Tracer version.

## VLAN Plan

| VLAN | Name | Native? | DHCP |
|---|---|---|---|
| 10 | EMPLOYEE | No | Yes |
| 20 | VOICE | No | Yes |
| 30 | GUEST | No | Yes |
| 40 | SERVERS | No | No |
| 99 | MANAGEMENT | No | No |
| 100 | WAN | No | No |

Reserve the first 99 addresses for infrastructure and DHCP starts at `.100`.

## Senior Engineer Baseline

Apply these settings consistently:

- Use strong enable passwords and local SSH users.
- Disable unused ports and assign them to VLAN 999.
- Enable port security on access ports.
- Enable root guard and BPDU guard on access ports.
- Add LoopGuard and spanning-tree protection where supported.
- Permit SSH v2 only; disable Telnet.
- Synchronize logging and NTP; DNS is `10.10.40.10`.
- Avoid directly connected server DHCP unless required by lab size.
- Use separate VLANs for user, voice, guest, server, and management traffic.
- Restrict management using ACLs and dedicated management VLAN.

## Core Switch Configuration

### Core1

```cisco
hostname CORE-1
username admin privilege 15 secret <CHANGE-ME>
enable secret <CHANGE-ME>
ip domain-name corp.local
crypto key generate rsa modulus 2048
ip ssh version 2

vtp mode transparent
vlan 10
 name EMPLOYEE
vlan 20
 name VOICE
vlan 30
 name GUEST
vlan 40
 name SERVERS
vlan 99
 name MANAGEMENT
vlan 100
 name WAN
vlan 999
 name UNUSED

spanning-tree mode rapid-pvst
spanning-tree vlan 10,20,30,40,99 root primary
spanning-tree vlan 100 root primary

interface range gigabitEthernet 0/1-2
 channel-group 1 mode active
 no shutdown
!
interface Port-channel1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,30,40,99,100
 no shutdown

interface range fastEthernet 0/1-2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,30,40,99
 spanning-tree portfast trunk
 no shutdown

interface vlan 99
 ip address 10.10.99.252 255.255.255.0
 no shutdown
!
ip default-gateway 10.10.99.254

interface vlan 10
 ip address 10.10.10.252 255.255.255.0
 standby 10 ip 10.10.10.254
 standby 10 priority 110
 standby 10 preempt
 no shutdown
!
interface vlan 20
 ip address 10.10.20.252 255.255.255.0
 standby 20 ip 10.10.20.254
 standby 20 priority 110
 standby 20 preempt
 no shutdown
!
interface vlan 30
 ip address 10.10.30.252 255.255.255.0
 standby 30 ip 10.10.30.254
 standby 30 priority 110
 standby 30 preempt
 no shutdown
!
interface vlan 40
 ip address 10.10.40.252 255.255.255.0
 standby 40 ip 10.10.40.254
 standby 40 priority 110
 standby 40 preempt
 no shutdown
!
interface vlan 99
 standby 99 ip 10.10.99.254
 standby 99 priority 110
 standby 99 preempt

ip routing
ip route 0.0.0.0 0.0.0.0 10.10.100.1

ip helper-address 10.10.40.10

line vty 0 15
 login local
 transport input ssh
 exec-timeout 10 0

logging buffered 16384
service timestamps log datetime msec
ntp server 10.10.40.10
ip name-server 10.10.40.10
```

Repeat VLAN and SVI configuration on Core2 with `.253` interface addresses and HSRP priority `100`.

## Core2 Baseline

Use the same VLANs, trunks, EtherChannel, STP, routing, DHCP relay, SSH, and NTP. Change:

```cisco
hostname CORE-2
interface vlan 10
 ip address 10.10.10.253 255.255.255.0
 standby 10 ip 10.10.10.254
 standby 10 priority 100
 standby 10 preempt
!
interface vlan 20
 ip address 10.10.20.253 255.255.255.0
 standby 20 ip 10.10.20.254
 standby 20 priority 100
 standby 20 preempt
!
interface vlan 30
 ip address 10.10.30.253 255.255.255.0
 standby 30 ip 10.10.30.254
 standby 30 priority 100
 standby 30 preempt
!
interface vlan 40
 ip address 10.10.40.253 255.255.255.0
 standby 40 ip 10.10.40.254
 standby 40 priority 100
 standby 40 preempt
!
interface vlan 99
 ip address 10.10.99.253 255.255.255.0
 standby 99 ip 10.10.99.254
 standby 99 priority 100
 standby 99 preempt
```

Add the outbound route `ip route 0.0.0.0 0.0.0.0 10.10.100.2` on Core2 if edge connectivity differs.

## Access Switches

```cisco
hostname ACCESS-1
username admin privilege 15 secret <CHANGE-ME>
vtp mode transparent

interface range fastEthernet 0/1-2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,30,40,99
 spanning-tree portfast trunk
 no shutdown

interface range fastEthernet 0/3-24
 switchport mode access
 switchport access vlan 10
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 switchport port-security mac-address sticky
 spanning-tree portfast
 spanning-tree bpduguard enable
 no shutdown

interface range gigabitEthernet 0/1-2
 shutdown

interface vlan 99
 ip address 10.10.99.11 255.255.255.0
 no shutdown
ip default-gateway 10.10.99.254

line vty 0 15
 login local
 transport input ssh
```

Access2 uses the management address `10.10.99.12` and a similar baseline.

## Edge Router

```cisco
hostname EDGE-R1
username admin privilege 15 secret <CHANGE-ME>
enable secret <CHANGE-ME>
ip domain-name corp.local
crypto key generate rsa modulus 2048
ip ssh version 2

interface gigabitEthernet0/0/0
 ip address 10.10.100.1 255.255.255.252
 ip nat inside
 no shutdown
!
interface gigabitEthernet0/0/1
 ip address 203.0.113.2 255.255.255.252
 ip nat outside
 no shutdown

ip route 10.10.0.0 255.255.0.0 10.10.100.1
ip route 0.0.0.0 0.0.0.0 203.0.113.1

ip nat inside source list NAT_TRAFFIC interface gigabitEthernet0/0/1 overload

ip access-list extended NAT_TRAFFIC
 permit ip 10.10.0.0 0.0.255.255 any

line vty 0 15
 login local
 transport input ssh
 exec-timeout 10 0

router? Keep-this-router-config-simple
```

Remove the placeholder text above before saving the device configuration.

**Important:** The edge router should use PAT. Do not configure a server's private address as a public address unless demonstrating 1:1 NAT.

## Security Policy

### Guest isolation

Guests may access DNS and internet only. They must not reach employee, server, voice, or management VLANs.

Apply on each core switch:

```cisco
ip access-list extended GUEST-IN
 permit udp any host 10.10.40.10 eq 53
 permit tcp any host 10.10.40.10 eq 53
 permit tcp any 203.0.113.0 0.0.0.3 eq 80
 permit tcp any 203.0.113.0 0.0.0.3 eq 443
 deny ip 10.10.30.0 0.0.0.255 10.10.0.0 0.0.255.255
 permit ip any any

interface vlan 30
 ip access-group GUEST-IN in
```

In Packet Tracer, internet simulation should use a server or router rather than the documentation-only `203.0.113.0/30` address. Adjust the permit rules to match the exact test environment.

### Management ACL

```cisco
ip access-list standard MGMT-SSH
 permit 10.10.99.0 0.0.0.255
 deny   any log

line vty 0 15
 access-class MGMT-SSH in
 login local
 transport input ssh
```

## Server Configuration

Server `10.10.40.10`:

- DNS: enable and map `corp.local`, `www.corp.local`, `files.corp.local`
- DHCP: configure pools for VLAN 10, 20, 30
- HTTP/HTTPS: enable for the internal portal
- FTP or file services: optional
- Syslog: optional depending on Packet Tracer version
- Gateway: `10.10.40.254`

Suggested employee DHCP:

- Pool: `10.10.10.100–10.10.10.200`
- Mask: `255.255.255.0`
- Default gateway: `10.10.10.254`
- DNS: `10.10.40.10`

Repeat for VLAN 20 and VLAN 30.

## Validation Checklist

### Connectivity

```text
ping 10.10.10.254
ping 10.10.40.10
ping 203.0.113.2
ping 203.0.113.1
show ip route
show ip interface brief
```

### Switching

```text
show interfaces trunk
show etherchannel summary
show spanning-tree summary
show spanning-tree vlan 10
show port-security interface fastEthernet0/3
show cdp neighbors
show lldp neighbors
```

### Redundancy

```text
show standby brief
show standby vlan 10
```

Then shut down Core1 uplink or SVI and confirm endpoints continue using HSRP gateway `10.10.10.254`.

### Security

```text
show ip access-lists
show ip interface vlan 30
show line
show users
```

Confirm:

- Guest cannot reach management VLAN 99
- Guest cannot reach employee VLAN 10
- Guest cannot reach server VLAN 40 except DNS
- Telnet fails
- SSH succeeds from management VLAN
- Port-security restricts unauthorized MAC addresses

## Test Cases

| ID | Test | Expected Result |
|---|---|---|
| TC-01 | Employee gets DHCP address | Address belongs to VLAN 10 |
| TC-02 | Employee reaches server | ICMP and HTTP succeed |
| TC-03 | Guest reaches server | DNS only; no management or employee traffic |
| TC-04 | Core1 shutdown | HSRP moves to Core2 |
| TC-05 | EtherChannel failure | Alternative uplink remains operational |
| TC-06 | Unauthorized workstation | Port security rejects or restricts it |
| TC-07 | Internet access | NAT/PAT translation succeeds |
| TC-08 | SSH from VLAN 99 | Administrator can connect |
| TC-09 | Telnet attempt | Connection is refused |

## Career Presentation Notes

- Explain the fault domains and blast radius for each VLAN.
- Explain why HSRP is used at the access-to-core boundary.
- Explain why the LACP EtherChannel sits in a separate redundant core layer.
- Explain the default route and DHCP helper-address design.
- Explain why guest ACLs are applied inbound to the guest SVI.
- Mention that this lab does not replace a real firewall, NAC, or enterprise routing stack.
- Record a short packet capture while testing and explain why packets are forwarded or denied.

## Known Packet Tracer Limitations

- HSRP support varies by switch model and IOS feature set.
- Interface names differ between 2960, 3650, 3560, and ISR images.
- `switchport trunk allowed vlan` and LACP syntax depend on model support.
- Packet Tracer generally has limited NetFlow, NTP, syslog, certificate, and IPSec behavior.
- Not every command runs on every simulated device.
- 802.1X and advanced NAC are better represented in a full lab platform.

## Deliverables

1. `.pkt` Packet Tracer topology
2. Addressing worksheet
3. Device configuration backup
4. Test report
5. Packet capture screenshots
6. Architecture diagram
7. Short interview walkthrough
