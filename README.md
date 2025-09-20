# Secure Smart Home Networking (Cisco Packet Tracer) — IT 491 Capstone

A secure, segmented smart home network built and validated in Cisco Packet Tracer.  
It demonstrates VLAN isolation, router-on-a-stick (ROAS) inter-VLAN routing, DHCP pools and exclusions, DHCP Snooping, Port Security (sticky MAC), ACLs (least-privilege), Cisco ASA with NAT and Clientless WebVPN, WPA3 Wi-Fi, centralized Syslog and Email alerts, and scenario-based testing with Wireshark.

> Main topology: `Secure Smart Home Networking alteredJC-5-01-25.pkt`  
> Design and test documentation: `IT 491 Final Report.pdf`

---

## Highlights

- Segmentation by role: Admin (VLAN 10), IoT (VLAN 20), Guest (VLAN 30), Users (VLAN 40)
- Inter-VLAN routing via ROAS with subinterfaces and trunking
- Per-VLAN DHCP pools with address exclusions and correct default gateways
- Access control policies:
  - Guest (VLAN 30): allow HTTP/HTTPS and DNS to the Internet; ICMP blocked; no access to Admin or IoT
  - IoT (VLAN 20): isolated from Admin (VLAN 10); limited egress as required
- Edge security with Cisco ASA: NAT and Clientless WebVPN portal for external access to internal web apps
- LAN hardening with DHCP Snooping (trusted uplink) and Port Security (sticky MAC, violation shutdown)
- Supporting services: DNS/HTTP/NTP server for the smart-home control plane, centralized Syslog and Email alerts
- Wireless security: WPA3 SSID for protected Wi-Fi access
- Validation: Wireshark captures, negative tests (denied pings/SSH), and portal login flow verification

---

## Topology and Roles

```
[ Outside Client ] --HTTPS--> [ ASA Firewall ] --NAT/ACL--> [ Router (ROAS) ] --trunk--> [ Switch ]
                                                                                         |-- VLAN 10: Admin PCs
                                                                                         |-- VLAN 20: IoT + Server (DNS/HTTP/NTP, Syslog/Email)
                                                                                         |-- VLAN 30: Guest (internet-only; ICMP blocked)
                                                                                         |-- VLAN 40: Users
```


- ROAS handles inter-VLAN routing on the router using subinterfaces.
- The switch carries VLAN trunks and access ports with port-security applied on edge ports.
- The server in VLAN 20 hosts DNS, HTTP control page, NTP, and receives Syslog and Email alerts.

---

## Network Design

### VLANs and Subnets

| VLAN | Role   | Example Subnet   | Notes                                                     |
|-----:|--------|------------------|-----------------------------------------------------------|
| 10   | Admin  | 192.168.10.0/24  | Admin PCs; management access to infrastructure            |
| 20   | IoT    | 192.168.20.0/24  | Smart-home devices; server at 192.168.20.100             |
| 30   | Guest  | 192.168.30.0/24  | Internet-only; HTTP/HTTPS/DNS allowed; ICMP blocked      |
| 40   | Users  | 192.168.40.0/24  | Standard user access                                     |

### Inter-VLAN Routing (ROAS)

- The router provides subinterfaces per VLAN (e.g., G0/0.10, G0/0.20, G0/0.30, G0/0.40).
- The switch uplink is a trunk carrying 802.1Q tags for these VLANs.
- Default route from the router to the ASA outside as applicable.

### Access Control

- Extended ACLs enforce least-privilege at VLAN boundaries and at the ASA edge.
- Guest VLAN is restricted to HTTP/HTTPS/DNS only and cannot reach Admin or IoT subnets.
- ICMP from Guest is explicitly denied to demonstrate policy effectiveness.

### LAN Hardening

- DHCP Snooping is enabled with the router uplink marked trusted; builds binding table to prevent rogue DHCP.
- Port Security uses sticky MAC on access ports; violations trigger interface shutdown or restrict as configured.

### Edge Security and Remote Access

- Cisco ASA performs NAT for inside-to-outside traffic.
- Clientless WebVPN provides a portal login; authenticated users launch a bookmark to reach the internal web app.

### Wireless

- WPA3-protected SSID is provided for secure wireless access aligned with the VLAN policy model.

---

## How to Open and Run

1. Open the `.pkt` file in Cisco Packet Tracer 8.x or newer and allow links to converge.
2. Verify addressing via DHCP on sample hosts in each VLAN; ensure default gateway assignment is correct.
3. ACL verification from the Guest VLAN:
   - Ping to Admin/IoT gateways should fail.
   - HTTP/HTTPS browsing to approved destinations should work.
4. DHCP Snooping test:
   - Connect a simulated rogue DHCP server on an untrusted access port and confirm clients do not receive leases.
   - Inspect the binding table to verify legitimate bindings.
5. Port Security test:
   - Swap a device on a sticky port and confirm violation behavior (err-disable/shutdown or restrict).
6. Remote access via ASA:
   - From an outside client, open the Clientless WebVPN portal, authenticate, and use the provided bookmark to reach the internal web application.
7. Logging and alerts:
   - Trigger a violation (for example, port-security event) and confirm Syslog entry and email alert generation.
8. Wireless WPA3:
   - Connect a wireless client to the WPA3 SSID and verify expected access limitations per VLAN policy.

---

## Test Commands (quick reference)

```
# On router/switch
show vlan brief
show interfaces trunk
show ip interface brief
show running-config
show access-lists
show ip dhcp binding
show ip dhcp snooping
show ip dhcp snooping binding
show port-security interface fa0/x
show mac address-table

# On ASA
show running-config
show access-list
show nat
show vpn-sessiondb detail webvpn
```

---

## Repository Structure

```
.
├─ Secure Smart Home Networking alteredJC-5-01-25.pkt
├─ IT 491 Final Report.pdf
├─ README.md

```

---

## License

This repository is intended for educational and portfolio use. You may fork and adapt the topology and documentation.
