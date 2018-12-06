# Hacking Networking

  - Overview
  - Basic Vs Advanced Zone
  - Network Models: L2, Shared, Isolated, VPC
  - Isolation: SG, VLAN, VXLAN etc.
  - Virtual Router
  - Network tools, usage and debugging

Tools: ping, traceroute, telnet, iptables, iproute2, ebtables, netstat

Case study:
SDN implementation for isolated network with KVM, VR with vlan+bridges.

References:
- Linux Virtual Networking 101: https://developers.redhat.com/blog/2018/10/22/introduction-to-linux-interfaces-for-virtual-networking/

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

## Network Provider Plugin

Dummy provider plugin?
