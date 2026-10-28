# Troubleshooting log

\# Troubleshooting log



This log records the faults and inconsistencies found while building and testing the network, with the symptom, cause, fix and evidence for each. Screenshots are linked from \[`docs/screenshots/`](screenshots/). ## Summary



| # | Problem | Area | Status |

|---|---|---|---|

| 1 | Layer 2 loop: duplicate ping replies and an unreliable server | Layer 2 | Fixed (symptom observed, not captured) |

| 2 | HSRP would not fail over | Gateway redundancy | Fixed |

| 3 | Guest `nslookup` times out | Services | Expected behavior |

| 4 | Brief interruption when the gateway fails back to Core-1 | Gateway redundancy | Open improvement |

| 5 | Core-2 Fa0/2 shown as a blocked port | Layer 2 | Cleared by itself |

| 6 | BR-SW1 ran plain PVST | Layer 2 | Fixed |

| 7 | ACC-SW3 guest ports had port-security lines but no port security | Security | Fixed (design decision) |

| 8 | Trunks could negotiate DTP | Layer 2 | Fixed |

| 9 | Unused ports not parked | Security | Fixed |

| 10 | Leftover HSRP line and unused SVIs | Cleanup | Fixed |

| 11 | One SSH attempt succeeded | Security | Produced |

| 12 | Port-security maximum inconsistent | Security | Fixed |

| 13 | Redundant OSPF interface command on HQ-R1 | Routing | Fixed |

| 14 | Staff PCs changed address between screenshots | Services | Expected behavior |



\## Entries



\### 1. Layer 2 loop: duplicate ping replies and an unreliable server



\- \*\*Symptom:\*\* on the network, pings returned duplicate replies and a server was unreliable. \*\*This was observed during testing, but no screenshot of the duplicate replies was captured\*\*, so there is no "before" evidence for this entry.

\- \*\*Cause:\*\* both uplinks of each access switch were forwarding at once, which is a Layer 2 loop.

\- \*\*Fix:\*\* explicit root priorities (Core-1 4096, Core-2 8192 on all VLANs) and plain (non-edge, unbundled) access uplinks, so Rapid PVST+ blocks the redundant path.

\- \*\*Evidence (after the fix only):\*\* \[Core-1 is the root](screenshots/routing-and-redundancy/l2-04-stp-root-core1.png), \[Core-2 reaches the root through Port-channel 1](screenshots/routing-and-redundancy/l2-04b-core2-cdp-and-stp-vlan99.png), and \[ACC-SW1 forwards on one uplink and blocks the other](screenshots/routing-and-redundancy/l2-05-stp-vlan10-accsw1.png).



\### 2. HSRP would not fail over



\- \*\*Symptom:\*\* when the active core lost its uplink, the standby did not take over.

\- \*\*Cause:\*\* the default tracking penalty (10) left the active core's priority tied with the standby's. Starting priority minus the penalty must be below the peer's priority.

\- \*\*Fix:\*\* Core-1 is set to priority 105, so a lost uplink drops it to 95, below Core-2's 100. Both cores preempt and track Gi0/1 on all five groups.

\- \*\*Evidence:\*\* \[Core-1 uplink down](screenshots/routing-and-redundancy/ha-01-core1-uplink-down.png), \[Core-2 Active on all groups](screenshots/routing-and-redundancy/ha-02-core2-active-all-groups.png), and \[Core-1 Active again at 105](screenshots/routing-and-redundancy/ha-03-core1-uplink-restored-preempt.png).



\### 3. Guest `nslookup` times out



\- \*\*Symptom:\*\* `nslookup web.chris.lab` from a guest device times out against 8.8.8.8.

\- \*\*Cause:\*\* the guest ACL blocks 10.10.0.0/16, which includes the internal DNS server, so guests are given a public DNS server, and the simulated ISP runs no DNS service.

\- \*\*Outcome:\*\* expected behavior and not a fault. Guests still reach 8.8.8.8, and internet access is tested by IP.

\- \*\*Evidence:\*\* \[HQ guest](screenshots/security-and-wireless/sec-04-guest-dns-hq-timeout.png), \[branch guest](screenshots/security-and-wireless/sec-05-guest-dns-br-timeout.png), and \[guest internet access](screenshots/service/svc-10-guest-internet-hq-and-ssh-blocked.png).



\### 4. Brief interruption when the gateway fails back to Core-1



\- \*\*Symptom:\*\* after Core-1's uplink was restored, the first traceroute from a staff PC timed out and later replies took 4 to 5 seconds.

\- \*\*Probable cause:\*\* Core-1 preempts as soon as tracking restores its priority, before its OSPF adjacency with HQ-R1 is fully up. This was not confirmed with debugging.

\- \*\*Recommended fix:\*\* add `standby <group> preempt delay minimum 60` under each Core-1 VLAN interface, so it takes over only after OSPF has converged. \*\*This has not been applied or verified.\*\*

\- \*\*Evidence:\*\* \[paths after the uplink was restored](screenshots/routing-and-redundancy/ha-06a-hq-staff-paths-core1-restored.png).



\### 5. Core-2 Fa0/2 shown as a blocked port



\- \*\*Symptom:\*\* `show spanning-tree vlan 99` on Core-2 briefly showed Fa0/2 as an alternate (blocked) port.

\- \*\*Probable cause:\*\* spanning tree still settling after the earlier link tests. This was not confirmed.

\- \*\*Outcome:\*\* cleared without any change. A later check showed Fa0/2 as Desg FWD on all five VLANs.

\- \*\*Evidence:\*\* \[Core-2 CDP and spanning tree after it cleared](screenshots/routing-and-redundancy/l2-04b-core2-cdp-and-stp-vlan99.png).



\### 6. BR-SW1 ran plain PVST



\- \*\*Symptom:\*\* BR-SW1 ran `pvst`, with no PortFast or BPDU guard on its edge ports.

\- \*\*Cause:\*\* configuration drift from the other switches, found during the configuration review.

\- \*\*Fix:\*\* changed to Rapid PVST+, and PortFast and BPDU guard enabled on all edge ports.

\- \*\*Evidence:\*\* \[BR-SW1 in rapid-pvst mode](screenshots/routing-and-redundancy/l2-11-rapid-pvst-brsw1.png).



\### 7. ACC-SW3 guest ports had port-security lines but no port security



\- \*\*Symptom:\*\* ACC-SW3 Fa0/16-24 carried sticky-learning lines but no `switchport port-security`, so nothing was enforced.

\- \*\*Cause:\*\* port security was never actually enabled on those ports.

\- \*\*Fix (design decision):\*\* guest ports have no port security because guest devices change constantly. The leftover lines were removed, and guests are controlled by the isolation ACL instead.

\- \*\*Evidence:\*\* \[ACC-SW3 VLAN assignments](screenshots/routing-and-redundancy/l2-10b-vlan-brief-accsw3.png).



\### 8. Trunks could negotiate DTP



\- \*\*Symptom:\*\* some trunk ports had no `switchport nonegotiate`.

\- \*\*Fix:\*\* added on every switch-to-switch trunk, including the Core-1 and Core-2 port-channel. BR-SW1's trunk to the router is a static trunk, so it was set to match.

\- \*\*Evidence:\*\* \[ACC-SW1 trunks](screenshots/routing-and-redundancy/l2-01-trunks-accsw1.png) and \[ACC-SW2 trunks](screenshots/routing-and-redundancy/l2-02-trunks-accsw2.png).



\### 9. Unused ports not parked



\- \*\*Symptom:\*\* ACC-SW2's unused ports were set to trunk mode, and unused Gigabit ports on the access switches were up in VLAN 1.

\- \*\*Fix:\*\* unused ports set to access mode in VLAN 999 and shut down, and the unused Core Gi0/2 ports shut down.



\### 10. Leftover HSRP line and unused SVIs



\- \*\*Symptom:\*\* a stray `standby 0 preempt` on Core-2 Vlan99, and unused Vlan10, Vlan30 and Vlan40 interfaces on HQ-R1.

\- \*\*Cause:\*\* leftover configuration.

\- \*\*Fix:\*\* removed.



\### 11. One SSH attempt refused, retry succeeded



\- \*\*Symptom:\*\* from BR-MGMT, one SSH attempt to ACC-SW2 was refused, and the retry logged in.

\- \*\*Outcome:\*\* not reproduced, and treated as transient Packet Tracer behavior. No change was needed.

\- \*\*Evidence:\*\* \[SSH from the branch management host](screenshots/security-and-wireless/sec-09-ssh-from-branch-mgmt-ok.png).



\### 12. Port-security maximum inconsistent



\- \*\*Symptom:\*\* the maximum differed between switches, and two test runs on ACC-SW1 Fa0/7 used different maximums.

\- \*\*Cause:\*\* inconsistent configuration.

\- \*\*Fix:\*\* standardized on a maximum of 1 on every non-guest edge port, and the Fa0/7 violation test was repeated on that setting.

\- \*\*Evidence:\*\* \[violation on Fa0/7](screenshots/security-and-wireless/sec-06-portsec-violation-fa0-7.png), \[recovery](screenshots/security-and-wireless/sec-07-portsec-recovery-fa0-7.png), and \[summary showing a maximum of 1](screenshots/security-and-wireless/sec-08-portsec-summary-accsw1.png).



\### 13. Redundant OSPF interface command on HQ-R1



\- \*\*Symptom:\*\* HQ-R1 Serial0/2/0 carried `ip ospf 1 area 0`, which the `network 10.10.255.0 0.0.0.3 area 0` statement already covers.

\- \*\*Fix:\*\* removed the interface command. The OSPF adjacency with BR-R1 dropped and re-formed within the same second, and all three neighbors were FULL afterwards.

\- \*\*Evidence:\*\* \[HQ-R1 neighbors](screenshots/routing-and-redundancy/rt-01-ospf-neighbors-hqr1.png).



\### 14. Staff PCs changed address between screenshots



\- \*\*Symptom:\*\* HQ-Staff 10 appears as 10.10.0.11 in some screenshots and 10.10.0.18 in others, and BR-Staff appears as 10.10.2.16 and 10.10.2.11.

\- \*\*Cause:\*\* the PCs renewed their DHCP leases and received different addresses. The MAC addresses are identical.

\- \*\*Outcome:\*\* expected DHCP behavior, with no change needed.

\- \*\*Evidence:\*\* \[BR-Staff, first lease](screenshots/service/svc-03-dhcp-br-staff.png) and \[BR-Staff, second lease](screenshots/service/svc-03b-dhcp-br-staff-second-lease.png).



\## Evidence notes



Some problems and tests have no matching screenshot, and none of them is presented as evidenced:



\- \*\*Layer 2 loop (entry 1):\*\* the duplicate replies were not captured, so only the fixed state is shown.

\- \*\*Gateway failback (entry 4):\*\* the interruption is visible in one traceroute, but there is no continuous ping through the failover, and the preempt delay was not tested.

\- \*\*WAN failure:\*\* only the recovery of the serial link was captured, not the failure itself, so the branch's DHCP dependency is documented as a limitation in the README and not as a tested result.

\- \*\*BPDU guard:\*\* the violation was captured, and the port reset was done but not screenshotted.

