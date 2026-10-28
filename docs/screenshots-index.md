# Screenshots index

This register organizes the supplied evidence into three sections. Only files present in the supplied archive are included; screenshot IDs without a matching file are omitted. Result values are carried over from the supplied test summary and were not independently re-executed.

Screenshots are stored under `docs/screenshots/` in the corresponding section folder.

## Services

|Test|Result|Evidence|
|-|-|-|
|HQ staff receives a DHCP lease via relay|Pass|[`svc-01`](screenshots/services/svc-01-dhcp-hq-staff.png)|
|HQ and branch guest clients receive DHCP leases|Pass|[`svc-02`](screenshots/services/svc-02-dhcp-hq-guest-wifi.png), [`svc-04`](screenshots/services/svc-04-dhcp-br-guest-wifi.png)|
|Branch staff receives a lease across the WAN|Pass |[`svc-03`](screenshots/services/svc-03-dhcp-br-staff.png)|
|HQ and branch VoIP VLAN PCs receive leases with internal DNS|Pass|[`svc-09`](screenshots/services/svc-09-dhcp-voip-hq.png), [`svc-09b`](screenshots/services/svc-09b-dhcp-voip-br.png)|
|Management workstations use static addresses in MGMT subnets|Pass |[`svc-08`](screenshots/services/svc-08-static-mgmt-br.png), [`svc-08b`](screenshots/services/svc-08b-static-mgmt-hq.png)|
|Internal DNS, NAT/PAT, and guest internet tests|Pass|[`svc-05 — initial`](screenshots/services/svc-05-dns-br-staff-internal.png), [`svc-05 — successful`](screenshots/services/svc-05-dns-br-staff-internal-ok.png), [`svc-06`](screenshots/services/svc-06-nat-hq-staff-icmp.png), [`svc-07`](screenshots/services/svc-07-nat-br-staff-and-guest-dns.png), [`svc-10`](screenshots/services/svc-10-guest-internet-hq-and-ssh-blocked.png)|

## Routing and redundancy

|Test|Result|Evidence|
|-|-|-|
|OSPF adjacencies and authentication verified|Pass |[`rt-01`](screenshots/routing-and-redundancy/rt-01-ospf-neighbors-hqr1.png), [`rt-02`](screenshots/routing-and-redundancy/rt-02-ospf-int-hqr1-gig.png), [`rt-03`](screenshots/routing-and-redundancy/rt-03-ospf-int-hqr1-serial.png), [`rt-04`](screenshots/routing-and-redundancy/rt-04-ospf-int-brr1-serial.png), [`rt-05`](screenshots/routing-and-redundancy/rt-05-ospf-int-core2.png)|
|Staff and VoIP connectivity between sites verified|Pass |[`rt-06`](screenshots/routing-and-redundancy/rt-06-ping-br-staff-to-hq-server-and-hq-staff.png), [`rt-07`](screenshots/routing-and-redundancy/rt-07-ping-hq-staff-to-hq-server-and-br-staff.png), [`rt-09`](screenshots/routing-and-redundancy/rt-09-ping-br-voip-to-hq-voip-staff-server.png), [`rt-10`](screenshots/routing-and-redundancy/rt-10-ping-hq-voip-to-br-voip-staff-and-server.png)|
|Branch learns default route through OSPF|Pass |[`rt-08`](screenshots/routing-and-redundancy/rt-08-default-route-brr1-and-ospf-neighbor.png)|
|HSRP failover and restoration tested|Pass |[`ha-01`](screenshots/routing-and-redundancy/ha-01-core1-uplink-down.png), [`ha-02`](screenshots/routing-and-redundancy/ha-02-core2-active-all-groups.png), [`ha-02 — additional`](screenshots/routing-and-redundancy/ha-02-core2-active-all-groups-2.png), [`ha-03`](screenshots/routing-and-redundancy/ha-03-core1-uplink-restored-preempt.png), [`Core-2 standby state`](screenshots/routing-and-redundancy/core2-back-to-standby.png), [`ha-06a`](screenshots/routing-and-redundancy/ha-06a-hq-staff-paths-core1-restored.png), [`ha-06b`](screenshots/routing-and-redundancy/ha-06b-hq-staff-paths-core1-uplink-down.png)|
|STP, EtherChannel, and edge protections tested|Pass |[`l2-01`](screenshots/routing-and-redundancy/l2-01-trunks-accsw1.png), [`l2-02`](screenshots/routing-and-redundancy/l2-02-trunks-accsw2.png), [`l2-03`](screenshots/routing-and-redundancy/l2-03-etherchannel-core1.png), [`l2-04`](screenshots/routing-and-redundancy/l2-04-stp-root-core1.png), [`l2-04b`](screenshots/routing-and-redundancy/l2-04b-core2-cdp-and-stp-vlan99.png), [`l2-04b — additional`](screenshots/routing-and-redundancy/l2-04b-core2-cdp-and-stp-vlan99-2.png), [`l2-05`](screenshots/routing-and-redundancy/l2-05-stp-vlan10-accsw1.png), [`l2-07a`](screenshots/routing-and-redundancy/l2-07a-etherchannel-member-down.png), [`l2-07b`](screenshots/routing-and-redundancy/l2-07b-staff-pings-while-fa0-4-down.png), [`l2-08`](screenshots/routing-and-redundancy/l2-08-bpduguard-violation.png), [`l2-10`](screenshots/routing-and-redundancy/l2-10-vlan-brief-accsw1.png), [`l2-10b`](screenshots/routing-and-redundancy/l2-10b-vlan-brief-accsw3.png), [`l2-11`](screenshots/routing-and-redundancy/l2-11-rapid-pvst-brsw1.png)|
|WAN link flap and OSPF recovery|Recovery observed in supplied evidence|[`rt-11`](screenshots/routing-and-redundancy/rt-11-wan-flap-brr1-log.png)|

## Security and wireless

|Test|Result|Evidence|
|-|-|-|
|Guest isolation and management ACLs verified|Pass |[`sec-03`](screenshots/security-and-wireless/sec-03-guest-br-blocked-br-staff.png), [`sec-04`](screenshots/security-and-wireless/sec-04-guest-dns-hq-timeout.png), [`sec-05`](screenshots/security-and-wireless/sec-05-guest-dns-br-timeout.png), [`sec-10`](screenshots/security-and-wireless/sec-10-ssh-from-staff-refused.png)|
|Port-security violation and recovery tested|Pass |[`sec-06`](screenshots/security-and-wireless/sec-06-portsec-violation-fa0-7.png), [`sec-06b`](screenshots/security-and-wireless/sec-06b-portsec-int-fa0-7.png), [`sec-07`](screenshots/security-and-wireless/sec-07-portsec-recovery-fa0-7.png), [`sec-08`](screenshots/security-and-wireless/sec-08-portsec-summary-accsw1.png)|
|SSH from management hosts tested|Pass |[`sec-09`](screenshots/security-and-wireless/sec-09-ssh-from-branch-mgmt-ok.png), [`sec-09b`](screenshots/security-and-wireless/sec-09b-ssh-from-hq-mgmt-ok.png)|
|Guest AP security settings checked|Pass |[`wl-02`](screenshots/security-and-wireless/wl-02-hq-ap-wpa2-settings.png), [`wl-02b`](screenshots/security-and-wireless/wl-02b-br-ap-wpa2-settings.png)|



