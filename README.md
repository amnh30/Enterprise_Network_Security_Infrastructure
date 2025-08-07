# Enterprise Network Security Infrastructure — Final Project (NTI Network Security Internship)
<image src="https://github.com/amnh30/Enterprise_Network_Security_Infrastructure/blob/main/Redesigned%20Network%20Security%20Architecture.png"/>
## Overview

This repository contains the configuration and documentation for a simulated enterprise-grade network built as the final project for the **Network Security Internship (NTI)**. The design demonstrates layered security, routing resilience, centralized authentication, secure site-to-site tunnelling, logging, and time synchronization. The environment was implemented and tested using Cisco IOS-style configuration (Packet Tracer compatible).

---

## Topology Summary

- **9 routers (Router0 — Router8)** forming an OSPF backbone (process `1`).
- **Multiple LANs**: 192.168.0.0/24, 192.168.2.0/24, 192.168.3.0/24, 192.168.4.0/24, 192.168.5.0/24, 192.168.6.0/24, 192.168.7.0/24, 192.168.8.0/24, plus inter-router /30 links in the 10.0.0.0/30 space.
- **Zone-Based Policy Firewall (ZPF)** implemented on Router4 (DMZ / INSIDE / OUTSIDE).
- **Site-to-site VPN** between Router2 (192.168.2.0/24 side) and Router8 (192.168.8.0/24 side) using ISAKMP + IPSec.
- **AAA**: TACACS+ and RADIUS servers integrated with local fallback for user authentication and accounting.
- **Infrastructure services**: NTP server (192.168.10.21) and Syslog server (192.168.10.20).

---

## Files in this Project

- `configs/` — Individual device configuration files (Router0.cfg ... Router8.cfg, Switch\*.cfg).
- `diagrams/topology.png` — Network topology diagram.
- `notes/` — Lab notes, test plans, verification commands and output samples.
- `README.md` — This file.

---

## Objectives

1. Demonstrate OSPF design and verification across multiple routers.
2. Implement secure communications between remote sites using IPSec VPN.
3. Centralize authentication and accounting (TACACS+, RADIUS) with fallback to local accounts.
4. Design and enforce traffic policies using ZPF and ACLs.
5. Harden Layer 2 with port security, VLANs, and trunking.
6. Provide centralized logging and accurate timestamping with Syslog and NTP.
7. Produce documentation and test evidence suitable for professional handoff.

---

## Key Technologies & Protocols

- Cisco IOS CLI
- OSPF (Area 0)
- ISAKMP (IKE) & IPSec
- TACACS+, RADIUS
- Zone-Based Policy Firewall (ZPF)
- ACLs (Extended)
- NAT (PAT) where required
- VLANs, Trunking, Port Security
- NTP & Syslog

---

## Notable Configuration Snippets

> These are illustrative extracts. Full device configs are in `configs/`.

### OSPF (example)

```text
router ospf 1
 network 192.168.0.0 0.0.0.255 area 0
 network 10.0.0.0 0.0.0.3 area 0
```

### Site-to-Site VPN (example on Router2)

```text
crypto isakmp policy 10
 encr aes
 hash sha
 authentication pre-share
 group 2
 lifetime 86400

crypto isakmp key VPN123 address 10.0.0.29
crypto ipsec transform-set VPN-SET esp-aes esp-sha-hmac
crypto map VPN-MAP 10 ipsec-isakmp
 set peer 10.0.0.29
 set transform-set VPN-SET
 match address 100
interface Serial0/1/0
 crypto map VPN-MAP
access-list 100 permit ip 192.168.2.0 0.0.0.255 192.168.8.0 0.0.0.255
```

### ZPF (high-level)

```text
zone security INSIDE
zone security DMZ
zone security OUTSIDE

interface GigabitEthernet0/0/1
 zone-member security INSIDE
interface GigabitEthernet0/0/0
 zone-member security DMZ
interface Serial0/1/0
 zone-member security OUTSIDE

zone-pair security ZP-IN2OUT source INSIDE destination OUTSIDE
 service-policy type inspect PM-IN2OUT
```

### TACACS+ and RADIUS (example)

```text
aaa new-model

tacacs-server host 192.168.3.21 key cisco
radius-server host 192.168.3.20 key cisco

aaa authentication login default group tacacs+ group radius local
aaa authorization exec default group tacacs+ group radius local
aaa accounting exec default start-stop group tacacs+
username cisco privilege 15 secret cisco
line vty 0 4
 login authentication default
 transport input ssh
```

### NTP & Syslog

```text
ntp server 192.168.10.21
logging 192.168.10.20
logging trap debugging
service timestamps log datetime msec
```

---

## Access Control & Hardening

- **Extended ACLs** were used to restrict services per host/group (ICMP-only for some hosts, HTTP/HTTPS where required).
- **WAN ACLs** drop unsolicited ICMP echo from the internet-facing links.
- **Port Security** with sticky MACs on access ports to mitigate MAC spoofing.
- **uRPF-like checks** (`ip verify unicast source reachable-via rx`) enabled on sensitive LAN interfaces where applicable.
- **Logging** (buffered & remote) configured for audit and troubleshooting.

---

## Test Plan & Verification Commands

- **OSPF**: `show ip ospf neighbor`, `show ip route ospf`
- **VPN**: `show crypto isakmp sa`, `show crypto ipsec sa`, `ping` across VPN
- **AAA**: `show tacacs`, `show running-config | include tacacs`, test login via VTY
- **ZPF**: `show zone-pair`, `show policy-map type inspect` and packet tests between zones
- **ACLs**: `show access-lists`, `debug ip packet` (lab-only)
- **Syslog / NTP**: check server logs & `show ntp status`

Include test evidence (screenshots / command outputs) in `notes/tests/`.

---

## Deployment Steps (brief)

1. Load individual `configs/RouterX.cfg` into each device in Packet Tracer.
2. Verify OSPF adjacency and route propagation.
3. Configure and verify NTP & Syslog servers.
4. Turn up VPN and validate encryption & traffic flow.
5. Integrate TACACS+/RADIUS and test login flows (VTY/SSH).
6. Apply ZPF policies and test inter-zone traffic per policy map.
7. Run the test plan and collect logs/screenshots.

---

## Security Considerations & Limitations

- **Pre-shared keys** used for the lab VPN are for demonstration only — use certificates for production.
- **Device credentials** in configs are sample values. Replace with strong secrets and secure vaults in real deployments.
- **Packet Tracer limitations**: Some advanced features or exact diagnostics may differ from real hardware behavior.
- **Logging & retention**: In production, forward logs to a hardened SIEM with proper retention and access control.

---

## Future Enhancements

- Migrate VPN to certificate-based authentication (IKEv2 + PKI).
- Implement SROS2 or signed OTA updates for device firmware where applicable.
- Centralized configuration management (Ansible, NetBox) and automated compliance checks.
- Integrate IDS/IPS and a SIEM for proactive threat detection.

---

## Credits

- **Instructor:** Eng. Salam — guidance and review.
- **Tools:** Cisco Packet Tracer, Cisco IOS.

---

## License

This project documentation is released under the MIT License — adapt as needed for school or employer rules.

---

If you want a version tailored for GitHub (with badges, smaller sections, or fewer technical details) or a `CHANGELOG.md`, tell me which format.

