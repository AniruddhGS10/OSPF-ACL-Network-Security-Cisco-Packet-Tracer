# OSPF Routing and ACL Based Network Security using Cisco Packet Tracer

## Project Overview

This project demonstrates the configuration of a multi-router network using Cisco Packet Tracer, combining OSPF dynamic routing with Standard and Extended Access Control Lists to implement specific network security policies.

The topology consists of five routers, a switch, a DHCP server, and three PCs. R1, R2, and R3 provide the main routed path between the different networks, while R4 is configured as SERV-1 and R5 is configured as SERV-2.

The project was built around two practical ACL scenarios. The first scenario uses a Standard ACL to restrict traffic from a particular source host. The second scenario uses an Extended ACL to restrict a particular protocol from a particular source to a particular destination while allowing other traffic.

The complete configuration was tested using ping, Telnet, and SSH to verify that the routing and security requirements were working as intended.

## Network Addressing

| Device | Interface | IP Address |
|---|---|---|
| R1 | G0/0 | 10.1.1.10/24 |
| R1 | G0/1 | 192.168.12.1/24 |
| R2 | G0/0 | 20.1.1.10/24 |
| R2 | G0/1 | 192.168.12.2/24 |
| R2 | G0/2 | 192.168.23.1/24 |
| R3 | G0/0 | 30.1.1.10/24 |
| R3 | G0/1 | 192.168.23.2/24 |
| R4 / SERV-1 | G0/0 | 20.1.1.2/24 |
| R5 / SERV-2 | G0/0 | 30.1.1.2/24 |
| PC1 | NIC | 10.1.1.1/24 |
| PC2 | NIC | 10.1.1.2/24 |
| PC3 | NIC | 10.1.1.3/24 |
| DHCP Server | NIC | 10.1.1.100/24 |

## OSPF Dynamic Routing

OSPF process ID 12345 was configured on R1, R2, R3, R4, and R5 using Area 0.

The 10.1.1.0/24, 20.1.1.0/24, 30.1.1.0/24, 192.168.12.0/24, and 192.168.23.0/24 networks were advertised through OSPF.

The purpose of this configuration was to establish dynamic routing between the different network segments before applying the ACL security policies.

## Standard ACL Security Scenario

The first requirement was to block all traffic from PC1, using source IP address 10.1.1.1, from reaching SERV-1 at 20.1.1.2.

A Standard ACL was configured on R2:

access-list 1 deny host 10.1.1.1
access-list 1 permit any

interface g0/1
ip access-group 1 out

The ACL was applied outbound on R2 G0/1 because this interface leads toward the 20.1.1.0/24 network containing SERV-1.

The first ACL statement blocks traffic originating from 10.1.1.1, while the second statement permits all other sources.

The configuration was verified by attempting to ping 20.1.1.2 from PC1. The ping failed and the response showed that the destination host was unreachable.

PC1 was then tested against SERV-2 at 30.1.1.2. The ping was successful, confirming that the ACL was restricting the intended traffic rather than completely preventing PC1 from communicating with the routed network.

## SSH and Telnet Configuration

SERV-2, configured as R5, was prepared for remote access using SSH and Telnet.

Local authentication was configured using:

username Ani privilege 15 password Ani

The domain name was configured and RSA keys were generated for SSH:

ip domain name google.com
crypto key generate rsa general-keys modulus 1234

The VTY lines were configured as:

line vty 0 3
transport input all
login local

This allowed SSH and Telnet to be tested against the ACL policy.

## Extended ACL Security Scenario

The second requirement was to block only Telnet traffic from PC2, using source IP address 10.1.1.2, when accessing SERV-2 at 30.1.1.2.

SSH access from the same PC should remain available, and other traffic should continue to work.

An Extended ACL was configured on R1:

access-list 100 tcp host 10.1.1.2 host 30.1.1.2 eq 23
access-list 100 permit any any

interface g0/0
ip access-group 100 in

The ACL was applied inbound on R1 G0/0 because traffic from the 10.1.1.0/24 LAN enters R1 through this interface.

The first ACL statement specifically matches TCP traffic from 10.1.1.2 to 30.1.1.2 using destination port 23. TCP port 23 is used by Telnet.

The second statement permits all other traffic.

Because SSH uses TCP port 22, SSH traffic from PC2 does not match the Telnet restriction and remains permitted.

## Verification

The configured ACL policies were verified using actual traffic from the PCs.

PC1 was tested against SERV-1 using ping. The connection failed, confirming that traffic from 10.1.1.1 toward 20.1.1.2 was being blocked by the Standard ACL.

PC1 was then tested against SERV-2 using ping. The connection was successful, confirming that the Standard ACL was not blocking PC1 from all remote networks.

PC2 was tested against SERV-2 using Telnet. The Telnet connection timed out, confirming that the Extended ACL was blocking TCP port 23 specifically from 10.1.1.2 to 30.1.1.2.

SSH was then tested from PC2 using:

ssh -l Ani 30.1.1.2

After authentication, the SERV-2 command prompt was successfully reached. This confirmed that SSH traffic was still permitted from PC2.

Finally, Telnet was tested from PC1 to SERV-2. The Telnet connection was successfully established and the local authentication prompt appeared. After authentication, the SERV-2 command prompt was reached.

This final test confirmed that Telnet was not globally disabled. The Extended ACL was specifically restricting Telnet traffic from PC2 while allowing Telnet from another permitted host.

## Final Traffic Policy

| Source | Destination | Traffic | Result |
|---|---|---|---|
| PC1 10.1.1.1 | SERV-1 20.1.1.2 | IP traffic | Blocked |
| PC1 10.1.1.1 | SERV-2 30.1.1.2 | Ping | Allowed |
| PC2 10.1.1.2 | SERV-2 30.1.1.2 | Telnet | Blocked |
| PC2 10.1.1.2 | SERV-2 30.1.1.2 | SSH | Allowed |
| PC1 10.1.1.1 | SERV-2 30.1.1.2 | Telnet | Allowed |
| Other permitted traffic | Other destinations | Other traffic | Allowed |

## Key Concepts Demonstrated

This project provided hands-on experience with IPv4 addressing, OSPF dynamic routing, OSPF Area 0, Standard ACLs, Extended ACLs, ACL placement and direction, source and destination based traffic filtering, TCP port based filtering, SSH, Telnet, local authentication, RSA key generation, VTY configuration, connectivity testing, and network troubleshooting.

A key part of the project was understanding where an ACL should be applied. The Standard ACL was applied outbound on R2 G0/1 because that interface leads toward the destination network containing SERV-1. The Extended ACL was applied inbound on R1 G0/0 because the traffic from the 10.1.1.0/24 LAN enters R1 through that interface.

## Verification Evidence

Screenshots included with this project show the configured topology and the actual verification performed in Cisco Packet Tracer.

The screenshots demonstrate the Standard ACL blocking PC1 from reaching SERV-1, successful connectivity from PC1 to SERV-2, Telnet being blocked from PC2, successful SSH access from PC2, and successful Telnet access from PC1.

These tests provide practical evidence that the configured ACL policies were working according to the intended requirements.

## Technologies Used

Cisco Packet Tracer
Cisco IOS
OSPF
Standard ACL
Extended ACL
SSH
Telnet
IPv4
TCP/IP

## Conclusion

This project demonstrates how OSPF can provide dynamic routing while ACLs are used to enforce specific network traffic policies.

The Standard ACL provides source based filtering, while the Extended ACL provides more granular control using the source IP, destination IP, protocol, and destination port.

The verification process confirmed that the required traffic was blocked while permitted traffic continued to function. The project therefore provided practical experience in configuring, applying, testing, and troubleshooting routing and network security policies in a Cisco Packet Tracer environment.
