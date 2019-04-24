# Hacking Networking

  - Overview
  - Basic Vs Advanced Zone
  - Network Models: L2, Shared, Isolated, VPC
  - Isolation: SG, VLAN, VXLAN etc.
  - Virtual Router
  - Network tools, usage and debugging

Reading list:
- Switching and Routing: https://www.practicalnetworking.net/series/packet-traveling/packet-traveling/
- ARP: https://www.practicalnetworking.net/series/arp/address-resolution-protocol/
- NAT, DNAT, SNAT: https://www.practicalnetworking.net/series/nat/nat/
- VLAN: https://www.practicalnetworking.net/stand-alone/vlans/
- Linux Virtual Networking 101: https://developers.redhat.com/blog/2018/10/22/introduction-to-linux-interfaces-for-virtual-networking/

Video List:
- CloudStack Networking series by Chiradeep: https://www.youtube.com/playlist?list=PLDF5F6BBB1C3A7CDD

Slides list:
- Inside CloudStack VR by Rohit: https://docs.google.com/presentation/d/1fTfOaur4BNTStd_NCuwNEGGJ3E4xbf-aTFIP0nqGCiY/edit#slide=id.g38be540065_6_22
- Networking Refresh by Dag: https://shapeblue.sharepoint.com/_layouts/15/Doc.aspx?sourcedoc=%7B25FBCF52-BC9D-4907-9432-5DEAB4A570CD%7D&file=Networking%20refresh%20April%202018.pptx&action=edit&mobileredirect=true&DefaultItemOpen=1

### Sessions

Session 1:
- Basic terms, terminologies
- OSI layer, L1-3, L4-7 layers
- Basic devices (switch, router, bridge, tap, tun, etc.)
- Basic protocols and addressing (arp, dhcp, dns, tcp, udp, icmp, igmp, ipv4, ipv6, optional: ospf, bgp)
- Isolation: vlan, vxlan, sg
- Bridge networking
- CloudStack network models basics
- Practical demo using monkeybox

Session 2:
- Revisit layers
- CloudStack network models adv. with diagrams for each type
- Linux nf framework (netfilters)
- Network tools (iproute2, iptables/ebtables/nftables, tcpdump, ping/arping, netstat, nslookup, arp, traceroute etc.)
- Debugging network stack across machines

Session 3:
- CloudStack SystemVM building, patching, init
- CloudStack agent framework, CPVM/SSVM use-cases
- Practical network part 1
- CloudStack VR programming part 1

Session 4:
- Practical network part 2
- CloudStack VR programming part 2
- Implement X: Wrap up, end to end demo and walkthrough
- Debug, find/extend feature etc.

### Untracked:

Tools: ping, traceroute, telnet, iptables, iproute2, ebtables, netstat

Case study:
SDN implementation for isolated network with KVM, VR with vlan+bridges.

References:

https://tcpdump101.com/

Misc topics/concepts:
router, switch, lb, access point, firewall, VPN (ipsec, gre, ssl/vpn, ptp/pptp)a

network service/application: dhcp, dns, proxy/reverse-proxy, NAT (pat, snat,
dnat), port forwarding

WAN tech - fiber, frame relay, satellite, dsl/adsl/vdsl, atm, ppp, mpls,
gsm/cdma (lte/4g, 3g, hspa+, edge), dialup, wimax, leased (t1, t3, e1, e3, oc3,
oc12),circuit switch, packet switch.

Popular cable/connectors: rj45, cat5, cat5e, cat6, cat6a, cat7, FC

Topologies: mesh (partial/full), bus, ring, star, hybrid, point-to-point,
point-to-multipoint, client-server, peer-peer

network infra: wan, man, lan, wlan/hotspot, pan (bluetooth, ir, nfc), scada/ics,
medianets (isdn, sip/ip)

addressing:
ipv4 (address structure, subnet, apipa, classful a,b,c,d, classless)
ipv6 (auto-config:eui64, slaac,, dhcp6, link local, address structure, address compression, tunnel 6to4,4to6, teredo/miredo)
private vs public, nat/pat, mac addressing, multicast, unicast, broadcast, broadcast domain vs collision domains

Routing concepts/protocols: loopback, routing loops, routing tables, static vs
dynamic routing, default route, RIPv2 (distance vector routing protocol), hybrid
routing BGP, link-state routing protocols (OSPF,IS-IS), interior vs exterior
gateway routing protocols, autonomous system numbers (ASN), route distribution,
high availability (VRRP, virtual IP, HSRP), route aggregation,
routing metrics (hop counts, mtu/bandwidth, costs, latency, administrative distance, spb)

Misc: VoIP, QoS (dscp, cos), multicast vs unicast

SDN/Virtualization: virtual switches/routers/firewall, virtual vs physical nic,
SDN (bridge, ovs etc), storage area network (SAN: iSCSI, jumbo frame, fiber
channel, network attached storage)

Network operations: packet/network analyzer (tcpdump/wireshark), interface
monitoring, port scanner (nmap), snmp, packet flow monitoring, link status,
upgrade/patching

Network security: disaster recovery (DR), single point of failure (SPOF),
pen-testing (kali linux?), attacks (DoS, ARP cache poisoning, spoofing, session
hijack, man-in-the-middle, vlan hopping, zero-day), unsecure ports/access, open
ports

tools: iproute2, netstat, ifconfig, ping/ping6, traceroute/6, nslookup, arp,
tcpdump, telnet, iptables, ebtables, nftables

https://cwiki.apache.org/confluence/display/CLOUDSTACK/CloudStack+Advanced+Network+Tutorial+-+Step+by+Step
