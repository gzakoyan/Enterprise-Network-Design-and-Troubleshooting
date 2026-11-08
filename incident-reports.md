# Network Troubleshooting — Incident Reports

## Overview

This document covers five network troubleshooting scenarios for the Enterprise Network Design & Troubleshooting project built using Cisco Packet Tracer.

The scenarios focus on identifying network failures, analyzing root causes, applying corrective configurations, and verifying network connectivity.

**Lab Status:** The scenarios are documented as troubleshooting exercises.

---

## Incident 1 — VLAN Misconfiguration

### Problem Description

IT-PC1 cannot communicate with its default gateway (192.168.20.1).

### Network Information

- Device: SW-IT
- Interface: FastEthernet0/1
- Expected VLAN: 20
- Incorrect VLAN: 30
- Affected Device: IT-PC1

### Root Cause

The switch port connected to IT-PC1 is incorrectly assigned to VLAN 30 instead of VLAN 20.

As a result, IT-PC1 cannot communicate with its default gateway.

### Diagnostic Commands

```cisco
show vlan brief
show interfaces status
show running-config
```

### Resolution

Restore the correct VLAN assignment:

```cisco
enable
configure terminal

interface fastEthernet 0/1
switchport mode access
switchport access vlan 20

end
```

### Verification

From IT-PC1:

```cmd
ping 192.168.20.1
```

**Expected Result:** Successful connectivity to the default gateway.

---

## Incident 2 — DHCP Service Failure

### Problem Description

A client computer cannot obtain an IP address automatically through DHCP.

### Network Information

- DHCP Server: R1
- Affected Network: Internal VLANs
- Protocol: DHCP
- Transport: UDP 67/68

### Root Cause

The DHCP service on R1 has been disabled.

As a result, new DHCP requests cannot be processed.

### Diagnostic Commands

```cisco
show running-config
show ip dhcp pool
show ip dhcp binding
```

### Resolution

Enable the DHCP service:

```cisco
enable
configure terminal

service dhcp

end
```

### Verification

On the affected PC:

1. Open Desktop.
2. Select IP Configuration.
3. Select DHCP.
4. Verify the assigned IP address and default gateway.

Example for IT-PC1:

```text
IP Address:      192.168.20.x
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.20.1
```

**Expected Result:** The client receives a valid DHCP configuration.

---

## Incident 3 — OSPF Neighbor Failure

### Problem Description

R1 loses its OSPF adjacency with R2 and can no longer learn the simulated ISP network through OSPF.

### Network Information

- Router 1: R1
- Router 2: R2
- Routing Protocol: OSPF
- OSPF Area: 0
- Transit Network: 10.0.0.0/30

### Root Cause

The R2 interface connected to R1 is configured as a passive interface.

This prevents OSPF Hello packets from being transmitted through that interface.

Consequently, the OSPF neighbor relationship fails after the dead timer expires.

### Diagnostic Commands

```cisco
show ip ospf neighbor
show ip ospf interface
show ip route ospf
show ip interface brief
```

### Resolution

Remove the passive-interface configuration from R2:

```cisco
enable
configure terminal

router ospf 1
no passive-interface gigabitEthernet 0/0

end
```

### Verification

On R1:

```cisco
show ip ospf neighbor
```

Expected neighbor state:

```text
Neighbor ID     State
2.2.2.2         FULL
```

Verify the routing table:

```cisco
show ip route ospf
```

Expected route:

```text
O 198.51.100.0/24 via 10.0.0.2
```

**Expected Result:** OSPF adjacency is restored and the remote network is learned.

---

## Incident 4 — NAT/PAT Misconfiguration

### Problem Description

Internal client traffic does not create the expected NAT translation entries.

### Network Information

- Router: R1
- Inside Networks: 192.168.10.0/24, 192.168.20.0/24, 192.168.30.0/24, 192.168.40.0/24
- Outside Interface: GigabitEthernet0/1
- Translation Method: PAT (Overload)

### Root Cause

The NAT overload configuration has been removed from R1.

Consequently, internal source IP addresses are not translated through the configured PAT rule.

### Diagnostic Commands

```cisco
show ip nat translations
show ip nat statistics
show running-config
show access-lists 1
```

### Resolution

Restore NAT/PAT configuration:

```cisco
enable
configure terminal

ip nat inside source list 1 interface gigabitEthernet 0/1 overload

end
```

### Verification

From IT-PC1:

```cmd
ping 198.51.100.10
```

On R1:

```cisco
show ip nat translations
```

Expected translation behavior:

```text
Inside Local:  192.168.20.x
Inside Global: 10.0.0.1
```

**Important:** Ping may still work without NAT because R2 learns the internal routes through OSPF. Successful connectivity alone does not prove that NAT is functioning.

**Expected Result:** New traffic generates NAT translation entries.

---

## Incident 5 — ACL Blocking Legitimate Traffic

### Problem Description

IT-PC1 cannot communicate with the internal server (192.168.40.10).

### Network Information

- Router: R1
- Source Network: 192.168.20.0/24
- Destination Server: 192.168.40.10
- Affected Interface: GigabitEthernet0/0.20
- Security Mechanism: Extended ACL

### Root Cause

An incorrectly configured Extended ACL blocks traffic from the IT network to the internal server.

### Diagnostic Commands

```cisco
show ip access-lists
show running-config
show ip interface gigabitEthernet 0/0.20
```

### Resolution

Remove the temporary ACL from the interface:

```cisco
enable
configure terminal

interface gigabitEthernet 0/0.20
no ip access-group IT_TEST_BLOCK in
exit

no ip access-list extended IT_TEST_BLOCK

end
```

### Verification

From IT-PC1:

```cmd
ping 192.168.40.10
```

**Expected Result:** Connectivity to the internal server is restored.


---

# Final Network Verification

The following tests should be performed after completing all troubleshooting exercises.

| Test | Source | Destination | Expected Result |
|------|--------|-------------|-----------------|
| VLAN Connectivity | IT-PC1 | IT-PC2 | Success |
| DHCP | IT-PC1 | R1 DHCP | IP Assigned |
| Inter-VLAN Routing | IT-PC1 | SRV1 | Success |
| OSPF | R1 | R2 | FULL |
| NAT/PAT | IT-PC1 | ISP-SERVER | Translation Created |
| HR Security ACL | HR-PC1 | IT-PC1 | Blocked |
| Sales HTTP Access | SALES-PC1 | SRV1 HTTP | Allowed |
| Sales ICMP Restriction | SALES-PC1 | SRV1 ICMP | Blocked |
| SSH Management | IT-PC1 | R1 SSH | Allowed |
| SSH Restriction | SALES-PC1 | R1 SSH | Blocked |

## Troubleshooting Methodology

The following methodology is used during network troubleshooting:

1. **Identify the Problem**
   - Determine the affected devices and services.
   - Collect information about the failure.

2. **Analyze the Network**
   - Check interface status.
   - Verify IP addressing.
   - Examine VLAN assignments and routing tables.

3. **Determine the Root Cause**
   - Review device configurations.
   - Compare expected and actual behavior.
   - Identify misconfigured interfaces or protocols.

4. **Implement a Solution**
   - Apply the necessary configuration changes.
   - Avoid modifying unrelated settings.

5. **Verify Connectivity**
   - Repeat the original connectivity test.
   - Confirm that the issue is resolved.
   - Check that other network services remain operational.

6. **Document the Incident**
   - Record the symptoms.
   - Describe the root cause.
   - Document the solution and verification results.

---

# Skills Demonstrated

- Network Troubleshooting
- Cisco IOS CLI
- VLAN Configuration
- DHCP Troubleshooting
- OSPF Routing Diagnostics
- NAT/PAT Troubleshooting
- Access Control Lists
- Network Security
- Root Cause Analysis
- Network Connectivity Testing
- Technical Documentation

---

# Conclusion

This project provides hands-on practice in enterprise network troubleshooting using Cisco Packet Tracer.

The scenarios demonstrate common networking issues involving VLANs, DHCP, OSPF, NAT/PAT, and Access Control Lists.

The objective is to develop practical troubleshooting, network analysis, and problem-solving skills relevant to Junior Network Engineer and NOC Engineer positions.

Actual incident outcomes will be documented after completing the corresponding Packet Tracer tests.

---

**Author:** Gagik Zakoyan

**Project:** Enterprise Network Design & Troubleshooting

**Platform:** Cisco Packet Tracer