# Enterprise Network: HQ and Branch

A two-site enterprise network designed, built and tested in Cisco Packet Tracer. It covers switching, routing, gateway redundancy, central services, security and wireless, and every feature was verified with a test plan and recorded in a troubleshooting log.

![Network topology diagram](<img width="1722" height="830" alt="topolog" src="https://github.com/user-attachments/assets/794ce7a5-22e9-4005-a28b-09554d67903d" />


## Quick links

| What | Where |
|---|---|
| Topology diagram | [diagrams/topology.png](diagrams/topology.png) |
| Device configurations (sanitized) | [configs/](configs/) |
| Troubleshooting log | [docs/troubleshooting-log.md](docs/troubleshooting-log.md) |
| Evidence: services | [docs/screenshots/service/](docs/screenshots/services/) |
| Evidence: routing and redundancy | [docs/screenshots/routing-and-redundancy/](docs/screenshots/routing-and-redundancy/) |
| Evidence: security and wireless | [docs/screenshots/security-and-wireless/](docs/screenshots/security-and-wireless/) |
| License | [LICENSE](LICENSE) |

## Scenario

A company runs a head office in Kumasi and a branch in Accra. Staff, servers, guests and management traffic must be kept apart, the gateway must survive the loss of a core switch, and guests must reach the internet but nothing inside the network.

## What is built

| Area | What was implemented | Why |
|---|---|---|
| Addressing | VLSM plan from 10.10.0.0/16, /30 routed links, and contiguous per-site blocks (HQ 10.10.0.0/23, Branch 10.10.2.0/24) | Right-sized subnets that can be summarized cleanly |
| VLANs | Staff, Servers, VoIP (voice-ready), Guest, Management, unused native VLAN 999 | Segmentation and a smaller attack surface |
| Layer 2 | Rapid PVST+ with Core-1 as root, LACP EtherChannel between the cores, port security on staff, server, VoIP and management ports, PortFast and BPDU guard on all edge ports | Loop-free redundancy and edge protection |
| Gateway redundancy | HSRP across two Layer 3 core switches with preemption and uplink tracking | A core failure does not take the gateway down |
| Routing | Single-area OSPF with passive interfaces, MD5 authentication on every link, and a default route from HQ-R1 | Automatic routes between sites, protected from rogue routers |
| Services | Static addressing for infrastructure and DHCP for users, a central DHCP server with relay at both sites, DNS with internal records, NAT with overload to a simulated ISP | One service point for both sites, with HQ protected from a core failure |
| Security | Guest isolation ACL, management ACL on the VTY lines, SSH only, hashed secrets, OSPF authentication, unused VLAN 1, no DTP on switch-to-switch trunks, unused ports shut down and parked | Least privilege and layered defense |
| Wireless | Guest SSID with WPA2-PSK mapped to the guest VLAN at both sites | Wireless guests get the same isolation as wired ones |

## Skills demonstrated

| Area | Skills | Where in the project |
|---|---|---|
| Design | VLSM subnetting, VLAN segmentation | Addressing plan, 10.10.0.0/16 split into right-sized subnets and /30 links |
| Switching | VLANs and trunking, Rapid PVST+, EtherChannel (LACP), port security, PortFast and BPDU guard | Core and access layers at both sites |
| Routing | Inter-VLAN routing (Layer 3 switch SVIs at HQ, router-on-a-stick at the branch), OSPF with MD5 authentication, passive interfaces, default route origination | Cores, HQ-R1, BR-R1 |
| Redundancy | HSRP with preemption and interface tracking | Core-1 and Core-2 |
| Services | DHCP relay, DNS, NAT/PAT | Central server, HQ-R1 |
| Security | Standard and extended ACLs, SSH hardening, routing protocol authentication, wireless security (WPA2-PSK) | VTY ACL, guest isolation ACL, NAT ACL, OSPF links, guest SSIDs |
| Operations | Structured troubleshooting and documentation | Test results, troubleshooting log |

## Devices and configurations

Every configuration is a sanitized `show running-config` export (password hashes and OSPF keys replaced with placeholders).

| Device | Role | Management IP (VLAN 99) | OSPF router ID | Configuration |
|---|---|---|---|---|
| HQ-R1 | HQ edge router, NAT, default route origin | n/a (router) | 1.1.1.1 | [HQ-R1.txt](configs/HQ-R1.txt) |
| Core-1 | HQ Layer 3 core, STP root, HSRP active | 10.10.1.66 | 2.2.2.2 | [Core-1.txt](configs/Core-1.txt) |
| Core-2 | HQ Layer 3 core, HSRP standby | 10.10.1.67 | 3.3.3.3 | [Core-2.txt](configs/Core-2.txt) |
| ACC-SW1 | HQ access (Staff, VoIP) | 10.10.1.68 | n/a | [ACC-SW1.txt](configs/ACC-SW1.txt) |
| ACC-SW2 | HQ access (Servers) | 10.10.1.69 | n/a | [ACC-SW2.txt](configs/ACC-SW2.txt) |
| ACC-SW3 | HQ access (Management, Guest) | 10.10.1.70 | n/a | [ACC-SW3.txt](configs/ACC-SW3.txt) |
| BR-R1 | Branch router (router-on-a-stick) | n/a (router) | 4.4.4.4 | [BR-R1.txt](configs/BR-R1.txt) |
| BR-SW1 | Branch access switch | 10.10.2.130 | n/a | [BR-SW1.txt](configs/BR-SW1.txt) |
| ISP (simulated) | Stands in for the internet, Gi0/0 203.0.113.1, Loopback0 8.8.8.8 | n/a | n/a | [ISP.txt](configs/ISP.txt) |

### Static hosts

| Host | Address | VLAN | Role |
|---|---|---|---|
| Central server | 10.10.0.130 | 20 (Servers) | DHCP and DNS for both sites |
| HQ-MGMT | 10.10.1.72 | 99 (MGMT) | HQ management workstation |
| BR-MGMT | 10.10.2.132 | 99 (MGMT) | Branch management workstation |

## Addressing plan

### VLANs and subnets

The VLAN names below are the ones configured on the switches.

| VLAN | Name | HQ subnet | HQ gateway (HSRP) | Branch subnet | Branch gateway |
|---|---|---|---|---|---|
| 10 | Staff | 10.10.0.0/25 | 10.10.0.1 | 10.10.2.0/26 | 10.10.2.1 |
| 20 | Servers | 10.10.0.128/26 | 10.10.0.129 | not used | n/a |
| 30 | Voip | 10.10.0.192/26 | 10.10.0.193 | 10.10.2.64/27 | 10.10.2.65 |
| 40 | Guest | 10.10.1.0/26 | 10.10.1.1 | 10.10.2.96/27 | 10.10.2.97 |
| 99 | MGMT | 10.10.1.64/28 | 10.10.1.65 | 10.10.2.128/28 | 10.10.2.129 |
| 999 | Native_Unused | none | n/a | none | n/a |

Core-1 uses the .2 / .131 / .194 / .2 / .66 host addresses in VLANs 10, 20, 30, 40 and 99, and Core-2 uses .3 / .132 / .195 / .3 / .67.

### Routed links

| Link | Subnet | Addresses |
|---|---|---|
| HQ-R1 to Core-1 | 10.10.255.4/30 | .5 (HQ-R1), .6 (Core-1) |
| HQ-R1 to Core-2 | 10.10.255.8/30 | .9 (HQ-R1), .10 (Core-2) |
| HQ-R1 to BR-R1 (serial WAN) | 10.10.255.0/30 | .1 (HQ-R1), .2 (BR-R1) |
| HQ-R1 to simulated ISP | 203.0.113.0/30 | .2 (HQ-R1), .1 (ISP, with 8.8.8.8/32 on Loopback0 as the "internet") |

## Design details

### Layer 2

- **Spanning tree:** Rapid PVST+ on every switch. Core-1 has priority 4096 (root) and Core-2 has 8192 (secondary), set explicitly for all VLANs.
- **EtherChannel:** the two cores are joined by an LACP bundle (Port-channel 1, Fa0/4-5, `mode active`). Access-switch uplinks are plain trunks, so Rapid PVST+ blocks the redundant path.
- **Trunks:** all trunks carry VLANs 10, 20, 30, 40 and 99 with native VLAN 999, and `switchport nonegotiate` is set on the switch-to-switch trunks.
- **Edge ports:** PortFast and BPDU guard on every access port. Port security uses sticky MAC learning, a maximum of one MAC address per port, and the shutdown violation action, and is applied on staff, server, VoIP and management ports. Guest ports are excluded by design because guest devices change constantly, and guests are controlled by the isolation ACL instead.
- **Unused ports:** shut down and parked in VLAN 999, and VLAN 1 is shut down on every device.

### Gateway redundancy

HSRP groups 10, 20, 30, 40 and 99 run across Core-1 and Core-2. Core-1 has priority 105 and Core-2 has the default of 100. Both cores preempt and track their routed uplink (Gi0/1). With the default tracking penalty of 10, a lost uplink drops Core-1 to 95, below Core-2's 100, so Core-2 takes over the virtual gateway.

### Routing

- OSPF process 1, single area 0, on HQ-R1, both cores and BR-R1.
- `passive-interface default` on the cores and BR-R1, with only the routed uplinks left active.
- HQ-R1 originates the default route (`default-information originate`) from a static route to the ISP.
- The serial WAN link has an OSPF cost of 64, against 1 on the Gigabit links.
- Inter-VLAN routing is done by the Layer 3 cores at HQ and by router-on-a-stick (BR-R1 subinterfaces) at the branch.

#### OSPF authentication

Every OSPF link uses MD5 message-digest authentication, so a router that does not know the key cannot form an adjacency or inject routes. Each link has its own key, and all links use key ID 1.

| Link | Interfaces | Authentication |
|---|---|---|
| HQ-R1 to Core-1 | HQ-R1 Gi0/1, Core-1 Gi0/1 | MD5, key ID 1, key unique to this link |
| HQ-R1 to Core-2 | HQ-R1 Gi0/2, Core-2 Gi0/1 | MD5, key ID 1, key unique to this link |
| HQ-R1 to BR-R1 (serial WAN) | HQ-R1 Se0/2/0, BR-R1 Se0/3/0 | MD5, key ID 1, key unique to this link |

The same two commands are used on both ends of each link (the key itself is a placeholder here):

```text
interface <routed-interface>
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 <OSPF-KEY>
```

Both ends must use the same key and key ID, or the adjacency does not come up. Verification is in the test results below.

### Services

- **Static for infrastructure, dynamic for users:** management and server addresses are fixed. The switch management addresses in VLAN 99, the management workstations at both sites (HQ-MGMT and BR-MGMT), the router and core interfaces, and the central DHCP and DNS server (10.10.0.130) are all statically assigned and excluded from the DHCP pools. Staff, VoIP and guest clients get their addresses from DHCP.
- **DHCP and DNS:** one central server in the Servers VLAN. `ip helper-address 10.10.0.130` is set on the Staff, VoIP and Guest gateways at both sites, and deliberately not on the Servers and MGMT gateways. Staff and VoIP clients get the internal DNS server, and guests get a public DNS server (8.8.8.8).
- **NAT:** HQ-R1 translates all of 10.10.0.0/16 to its ISP-facing address with overload.

### Security

- **Guest isolation:** the `Guest-in` ACL is applied inbound on the Guest gateway at both sites. It permits DHCP, denies the whole 10.10.0.0/16 range, and permits everything else, so guests reach the internet and nothing internal.
- **Management access:** ACL 10 on the VTY lines allows only the two management subnets (10.10.1.64/28 and 10.10.2.128/28), and `transport input ssh` with SSH version 2 is set on every device.
- **Routing protocol security:** OSPF adjacencies are authenticated with MD5 on every link (see [OSPF authentication](#ospf-authentication)), and `passive-interface default` stops hellos from leaving the core and branch VLAN interfaces.
- **Credentials:** passwords are hashed (`service password-encryption` and `secret`), with local login on the console and VTY lines and a 5-minute exec timeout.
- **Banner:** an authorized-access banner is set on every device.

### Wireless

One access point per site serves a WPA2-PSK guest SSID (AES): `HQGuestWiFi` at HQ and `BR-GuestWiFi` at the branch. Each is mapped to VLAN 40, so wireless guests are subject to the same `Guest-in` ACL as wired guests.

## Test results

Evidence is stored under [`docs/screenshots/`](docs/screenshots/) in three folders: [`service`](docs/screenshots/service/), [`routing-and-redundancy`](docs/screenshots/routing-and-redundancy/) (which also holds the Layer 2 evidence) and [`security-and-wireless`](docs/screenshots/security-and-wireless/).

### Services

| Test | Result | Evidence |
|---|---|---|
| HQ staff PC receives a DHCP lease through the relay (10.10.0.0/25, gateway 10.10.0.1, DNS 10.10.0.130) | Pass | [svc-01][View DHCP HQ Staff Screenshot](docs/screenshots/service/svc-01-dhcp-hq-staff.png) |
| HQ guest laptop on the guest SSID receives a lease (10.10.1.0/26, gateway 10.10.1.1) | Pass | [svc-02](docs/screenshots/service/svc-02-dhcp-hq-guest-wifi.png) |
| Branch staff PC receives a lease across the WAN (10.10.2.0/26, gateway 10.10.2.1) | Pass | [svc-03](docs/screenshots/service/svc-03-dhcp-br-staff.png), [svc-03b](docs/screenshots/service/svc-03b-dhcp-br-staff-second-lease.png) |
| Branch guest laptop on the guest SSID receives a lease (10.10.2.96/27, gateway 10.10.2.97) | Pass | [svc-04](docs/screenshots/service/svc-04-dhcp-br-guest-wifi.png) |
| HQ VoIP PC receives a lease (10.10.0.192/26, gateway 10.10.0.193, DNS 10.10.0.130) | Pass | [svc-09](docs/screenshots/service/svc-09-dhcp-voip-hq.png) |
| Branch VoIP PC receives a lease across the WAN (10.10.2.64/27, gateway 10.10.2.65, DNS 10.10.0.130) | Pass | [svc-09b](docs/screenshots/service/svc-09b-dhcp-voip-br.png) |
| Management workstations use static addresses inside the MGMT subnets, outside the DHCP pools (BR-MGMT 10.10.2.132, HQ-MGMT 10.10.1.72) | Pass | [svc-08](docs/screenshots/service/svc-08-static-mgmt-br.png), [svc-08b](docs/screenshots/service/svc-08b-static-mgmt-hq.png) |
| Branch staff resolves `web.chris.lab` through the central DNS server | Pass | [svc-05](docs/screenshots/service/svc-05-dns-br-staff-internal-ok.png) |
| NAT overload translates HQ staff traffic to 203.0.113.2 | Pass | [svc-06](docs/screenshots/service/svc-06-nat-hq-staff-icmp.png) |
| NAT translates branch staff traffic and guest DNS queries | Pass | [svc-07](docs/screenshots/service/svc-07-nat-br-staff-and-guest-dns.png) |
| HQ guest reaches 8.8.8.8 | Pass | [svc-10](docs/screenshots/service/svc-10-guest-internet-hq-and-ssh-blocked.png) |
| Branch guest reaches 8.8.8.8 | Pass | [svc-10b](docs/screenshots/service/svc-10b-guest-internet-br-and-ssh-blocked.png) |

### Routing

| Test | Result | Evidence |
|---|---|---|
| HQ-R1 has FULL adjacencies with both cores and BR-R1 while authentication is enabled | Pass | [rt-01](docs/screenshots/routing-and-redundancy/rt-01-ospf-neighbors-hqr1.png) |
| MD5 authentication (key ID 1) is enabled on the HQ-R1 to Core-1 and Core-2 links | Pass | [rt-02](docs/screenshots/routing-and-redundancy/rt-02-ospf-int-hqr1-gig.png), [rt-05](docs/screenshots/routing-and-redundancy/rt-05-ospf-int-core2.png) |
| MD5 authentication and cost 64 on the serial WAN, both ends | Pass | [rt-03](docs/screenshots/routing-and-redundancy/rt-03-ospf-int-hqr1-serial.png), [rt-04](docs/screenshots/routing-and-redundancy/rt-04-ospf-int-brr1-serial.png) |
| Branch staff reaches the HQ server and HQ staff through HQ-R1 (TTL 125) | Pass | [rt-06](docs/screenshots/routing-and-redundancy/rt-06-ping-br-staff-to-hq-server-and-hq-staff.png) |
| HQ staff reaches the HQ server through the core (TTL 127) and the branch staff PC across the WAN (TTL 125) | Pass | [rt-07](docs/screenshots/routing-and-redundancy/rt-07-ping-hq-staff-to-hq-server-and-br-staff.png) |
| BR-R1 learns the default route through OSPF (external type 2 via 10.10.255.1) | Pass | [rt-08](docs/screenshots/routing-and-redundancy/rt-08-default-route-brr1.png) |
| VoIP PCs at both sites reach each other, the staff PCs and the server (TTL 125 across the WAN, 127 within HQ) | Pass | [rt-09](docs/screenshots/routing-and-redundancy/rt-09-ping-br-voip-to-hq-voip-staff-server.png), [rt-10](docs/screenshots/routing-and-redundancy/rt-10-ping-hq-voip-to-br-voip-staff-and-server.png) |
| WAN link drops and the adjacency and default route recover | Pass (recovery only) | [rt-11](docs/screenshots/routing-and-redundancy/rt-11-wan-flap-brr1-log.png) |

### Gateway redundancy

| Test | Result | Evidence |
|---|---|---|
| Core-1 uplink shut down, OSPF neighbor lost | Pass | [ha-01](docs/screenshots/routing-and-redundancy/ha-01-core1-uplink-down.png) |
| Core-2 becomes Active on all five HSRP groups | Pass | [ha-02](docs/screenshots/routing-and-redundancy/ha-02-core2-active-all-groups.png) |
| Uplink restored, Core-1 preempts and is Active again at priority 105 | Pass | [ha-03](docs/screenshots/routing-and-redundancy/ha-03-core1-uplink-restored-preempt.png) |
| Core-2 returns to Standby | Pass | [ha-05](docs/screenshots/routing-and-redundancy/ha-05-core2-back-to-standby.png) |
| Staff traffic to the internet, the ISP and the branch keeps working through Core-2 while Core-1's uplink is down | Pass | [ha-06b](docs/screenshots/routing-and-redundancy/ha-06b-hq-staff-paths-core1-uplink-down.png) |
| The same paths go back through Core-1 after the uplink is restored | Pass (brief interruption, see the troubleshooting log) | [ha-06a](docs/screenshots/routing-and-redundancy/ha-06a-hq-staff-paths-core1-restored.png) |

### Layer 2

| Test | Result | Evidence |
|---|---|---|
| Access trunks are static 802.1Q with native VLAN 999 and the five allowed VLANs | Pass | [l2-01](docs/screenshots/routing-and-redundancy/l2-01-trunks-accsw1.png), [l2-02](docs/screenshots/routing-and-redundancy/l2-02-trunks-accsw2.png) |
| Port-channel 1 is up with both members bundled | Pass | [l2-03](docs/screenshots/routing-and-redundancy/l2-03-etherchannel-core1.png) |
| Core-1 is the spanning-tree root | Pass | [l2-04](docs/screenshots/routing-and-redundancy/l2-04-stp-root-core1.png) |
| Core-2 is a non-root bridge reaching the root through Port-channel 1, and CDP confirms the cabling | Pass | [l2-04b](docs/screenshots/routing-and-redundancy/l2-04b-core2-cdp-and-stp-vlan99.png) |
| An access switch forwards on one uplink and blocks the other | Pass | [l2-05](docs/screenshots/routing-and-redundancy/l2-05-stp-vlan10-accsw1.png) |
| The bundle stays up with one member down | Pass | [l2-07a](docs/screenshots/routing-and-redundancy/l2-07a-etherchannel-member-down.png) |
| Staff reachability continues while the member is down | Pass | [l2-07b](docs/screenshots/routing-and-redundancy/l2-07b-staff-pings-while-fa0-4-down.png) |
| BPDU guard disables an edge port when a switch is connected | Pass | [l2-08](docs/screenshots/routing-and-redundancy/l2-08-bpduguard-violation.png) |
| VLAN names and port assignments match the design on ACC-SW1 and ACC-SW3 | Pass | [l2-10](docs/screenshots/routing-and-redundancy/l2-10-vlan-brief-accsw1.png), [l2-10b](docs/screenshots/routing-and-redundancy/l2-10b-vlan-brief-accsw3.png) |
| BR-SW1 runs Rapid PVST+ | Pass | [l2-11](docs/screenshots/routing-and-redundancy/l2-11-rapid-pvst-brsw1.png) |

### Security

| Test | Result | Evidence |
|---|---|---|
| HQ guest cannot reach the branch, the management subnet or switch SSH | Pass | [sec-01](docs/screenshots/security-and-wireless/sec-01-guest-hq-blocked-branch-mgmt-ssh.png) |
| Branch guest cannot reach the HQ server | Pass | [sec-02](docs/screenshots/security-and-wireless/sec-02-guest-br-blocked-hq-server.png) |
| Branch guest cannot reach branch staff | Pass | [sec-03](docs/screenshots/security-and-wireless/sec-03-guest-br-blocked-br-staff.png) |
| Guests cannot resolve internal names (public DNS handed out, internal server blocked) | Pass (expected) | [sec-04](docs/screenshots/security-and-wireless/sec-04-guest-dns-hq-timeout.png), [sec-05](docs/screenshots/security-and-wireless/sec-05-guest-dns-br-timeout.png) |
| An unknown device on a staff port exceeds the one-MAC limit, triggers a violation and shuts the port | Pass | [sec-06](docs/screenshots/security-and-wireless/sec-06-portsec-violation-fa0-7.png) |
| The port is restored with shutdown and no shutdown | Pass | [sec-07](docs/screenshots/security-and-wireless/sec-07-portsec-recovery-fa0-7.png) |
| Port-security summary on ACC-SW1 (maximum 1, shutdown action) | Pass | [sec-08](docs/screenshots/security-and-wireless/sec-08-portsec-summary-accsw1.png) |
| SSH works from the HQ and branch management hosts, including across the WAN | Pass | [sec-09](docs/screenshots/security-and-wireless/sec-09-ssh-from-branch-mgmt-ok.png), [sec-09b](docs/screenshots/security-and-wireless/sec-09b-ssh-from-hq-mgmt-ok.png) |
| SSH from a staff PC is refused by the VTY ACL | Pass | [sec-10](docs/screenshots/security-and-wireless/sec-10-ssh-from-staff-refused.png) |

### Wireless

| Test | Result | Evidence |
|---|---|---|
| HQ access point and guest laptop | Pass | [wl-01](docs/screenshots/security-and-wireless/wl-01-hq-ap-and-laptop.png) |
| HQ and branch access points use WPA2-PSK with AES (passphrase redacted) | Pass | [wl-02](docs/screenshots/security-and-wireless/wl-02-hq-ap-wpa2-settings.png), [wl-02b](docs/screenshots/security-and-wireless/wl-02b-br-ap-wpa2-settings.png) |

## Troubleshooting

The full log, with symptom, cause, fix and evidence for each item, is in [`docs/troubleshooting-log.md`](docs/troubleshooting-log.md). Two entries shaped the design:

- **Layer 2 loop:** duplicate ping replies and an unreliable server were caused by both uplinks of each access switch forwarding at once. Explicit root priorities and plain (non-edge, unbundled) access uplinks let Rapid PVST+ block the redundant path. The duplicate replies were observed but not captured, so only the fixed state is evidenced.
- **HSRP would not fail over:** the default tracking penalty left the active core's priority tied with the standby's. Starting priority minus the penalty must be below the peer's priority, so Core-1 runs at 105.

## Limitations and future work

### Limitations

- **HSRP authentication:** not supported by the Packet Tracer image used. On real IOS it is enabled with `standby <group> authentication md5 key-string <key>` on every group, using the same key on both cores. Compensating controls are in place: edge ports are locked down with PortFast, BPDU guard and port security, unused ports are shut down, and guest and management traffic sit in separate VLANs.
- **Voice:** the VoIP VLAN, subnets and DHCP pools are provisioned and tested with PCs, but there is no telephony service, QoS or phones. This is a voice-ready design, not a working voice service.
- **Single point of failure at the branch:** the branch depends on the serial WAN link and HQ-R1 for DHCP and DNS, so a WAN failure leaves branch clients without new leases. The core-failure protection applies to HQ.
- **Guest DNS:** the guest ACL blocks all of 10.10.0.0/16, which includes the internal DNS server, so the guest DHCP pools hand out a public DNS server (8.8.8.8). Guests therefore cannot resolve internal names, which is intended.
- **Simulated ISP:** it runs no DNS service, so internet names do not resolve in this lab. Internet reachability is tested by IP (8.8.8.8).
- **Port security:** one MAC address per port, so a PC connected through an IP phone would need a maximum of 2 on that port. This build has no phones, so one is enough.
- **OSPF authentication:** the tests show authentication enabled and adjacencies up. A wrong-key test, showing that a mismatched key blocks an adjacency, was not run.
- **VLAN 999:** it is both the native VLAN and the parking VLAN for unused ports.
- **Password hashing:** secrets use Cisco type 5 hashing, which is the strongest option these Packet Tracer images support.

### Not built (future work)

- IPsec site-to-site VPN between HQ and the branch
- Network monitoring (SNMP, syslog and NetFlow collection)
- Configuration backup automation
- DHCP snooping and dynamic ARP inspection
- IPv6
- OSPF HMAC-SHA authentication with key chains, which newer IOS versions support

## Repository layout

```text
.
├── README.md
├── LICENSE
├── diagrams/
│   └── topology.png
├── configs/
│   ├── ACC-SW1.txt
│   ├── ACC-SW2.txt
│   ├── ACC-SW3.txt
│   ├── Core-1.txt
│   ├── Core-2.txt
│   ├── HQ-R1.txt
│   ├── BR-R1.txt
│   ├── BR-SW1.txt
│   └── ISP.txt
└── docs/
    ├── troubleshooting-log.md
    └── screenshots/
        ├── routing-and-redundancy/
        ├── security-and-wireless/
        └── service/
```

## Security note

The files in `configs/` and the screenshots have had password hashes, OSPF keys and Wi-Fi passphrases replaced or blurred. 

## License

This project is released under the license in [LICENSE](LICENSE).
